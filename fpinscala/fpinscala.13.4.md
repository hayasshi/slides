# 13.4 より高機能なIO

---
スタックオーバーフローに関する問題は解決したが、モナドに関する問題が2つ残っている。

- 発生する作用が曖昧(IOでまるめてる)
- 実行スレッドをブロックせずにIOを実行するための並列化の仕組みと手段がない

例えば、`FlatMap(Suspend(s), k)`をrunすると、s()が実行されるが、それがもたらす作用がどのようなものか、インタープリタ側はわからない。
つまり、*作用が不透明* である。

これによって、インタープリタが非同期呼び出しをサポートしたくても、可能にする方法がない。

---
これらは、Suspendの引数resumeがFunction0であることに起因する。(完了を待つことしかできない)
Function0の代わりに、一時的な停止にParを使用したらどうなるか考える。
インタープリタが非同期実行をサポートできるようになるため、この方をAsyncと名付けて定義する。

```scala
sealed trait Async[A] {
  def flatMap[B](f: A => Async[B]): Async[B] = FlatMap(this, f)
  def map[B](f: A => B): Async[B] = flatMap(f andThen (Return(_)))
}

case class Return[A](a: A) extends Async[A]
case class Suspend[A](resume: Par[A]) extends Async[A]
case class FlatMap[A, B](sub: Async[A], k: A => Async[B]) extends Async[B]
```

---
Suspendの引数がFunction0[A]からPar[A]に変更されたことにより、runの実装が変更される。
runは、Aではなく、Par[A]を返すようになる。そして、FlatMapのコンストラクタ再結合には別の末尾再帰関数stepを定義して利用する。

```scala
@annotation.tailrec
def step[A](async: Async[A]): Async[A] = async match {
  case FlatMap(FlatMap(x, f), g) => step(x flatMap (xa => f(xa) flatMap g)) // FlatMap(x, fg)
  case FlatMap(Return(x), f)     => step(f(x))
  case _                         => async
}

def run[A](async: Async[A]): Par[A] = step(async) match {
  case Return(a)     => Par.unit(a)
  case Suspend(r)    => r
  case FlatMap(x, f) => x match {
    case Suspend(r) => Par.flatMap(r)(a => run(f(a)))
    case _          => sys.error("Impossible; `step` eliminates these cases") // stepを経由しているのでこのケースには来ないはず
  }
}
```

任意のParを受け取るSuspendを利用することで、非同期処理をサポートすることができるようになった。

---
さらにこれを一歩進めて、Suspendで使用される型コンストラクタの選択と抽象化を行う。
そのためには、TailRecとAsyncを一般化し、Function0やParを明示的に利用するのではなく、型コンストラクタFでパラメータ化する。(外部から与えられるようにする。)
このより抽象的なデータ型をFreeと呼ぶことにする。

```scala
seald trait Free[F[_], A]
case class Return[F[_], A](a: A) extends Free[F, A]
case class Suspend[F[_], A](resume: F[A]) extends Free[F, A]
case class FlatMap[F[_], A, B](sub: Free[F, A], k: A => F[B]) extends Free[F, B]

type TailRec[A] = Free[Function0, A]
type Async[A] = Free[Par, A]
```

---
## 13.4.1 フリーモナド

ReturnコンストラクタとFlatMapコンストラクタは、Fのあらゆる選択肢に対するモナドであることを裏付けている。
(11.2 flatMapと単位関数の一般化参照)
またモナドを生成するにはこれらの演算(ReturnコンストラクタとFlatMapコンストラクタ)が必要であるので、これはフリーモナド(FreeMonad)であるといえる。

※10 ここで言う「フリー」とは、F自体がモナド構造を持つ必要がないという点で、*自由に生成される* ことを意味する。
「フリー」の意味のより正式な定義については、チャプターノートを参照。
https://github.com/fpinscala/fpinscala/wiki

---
### exercise13.1
### exercise13.2
### exercise13.3
確認する。

---
### Free[F, A]の意味

Free[F, A]は、基本的には型Aの値が、0個以上のFのレイヤに包含された再帰構造である。
これがモナドであるのは、flatMapを通じてAを受け取ることができ、そこからFのレイヤをさらに生成できるためである。
結果を取得するためには、この構造のインタープリタがそうしたFレイヤを全て処理できなければならない。

この構造とインタープリタを、相互にやり取りするコルーチンとみなすことができる。
そして、Fはこのやり取りの手順を定義する。Fを慎重に選択すれば、どのような種類のやり取りを可能にするかを正確に制御できる。

---
## 13.4.2 コンソールI/Oだけをサポートするモナド

ここまでコンソールI/OにFunction0を利用してきたが、これは作用に制限がなく推論できない。
コンソールのやり取りをモデリングする代数データ型を考える。

```scala
seald trait Console[A] {
  def toPar: Par[A]
  def toThunk: () => A
}
case object ReadLine extends Console[Option[String]] {
  def toPar = Par.lazyUnit(run)
  def toThunk = () => run
  def run: Option[String] = try { Some(readLine()) } catch { case e: Exception => None }
}
case class PrintLine(line: String) extends Console[Unit] {
  def toPar = Par.lazyUnit(println(line))
  def toThunk = () => println(line)
}
```

---
この代数データ型をFreeに埋め込むと、コンソールIOのみを許可する制限されたIO型が得られる。

```scala
object Console {
  type ConsoleIO[A] = Free[Console, A]
  def readLn: ConsoleIO[Option[String]] = Suspend(ReadLine)
  def printLn(line: String): ConsoleIO[Unit] = Suspend(PrintLine(line))
}
```

---
これを使って、コンソールとやり取りするプログラムを記述できる。
API的に、他の種類のI/Oは実行されないことがわかる。

```scala
val f1: Free[Console, Option[String]] = for {
  _ <- printLn("I can only interact with the console.")
  ln <- readLn
} yield ln
```

---
ConsoleIOを実行したいが、いまはできない。
runのシグネチャにあるF(ConsoleIO)のモナドインスタンスを定義できていないためである。(runのシグネチャ唐突に出てきた)
(少し前にあった、F自体はモナド構造を持たなくても良いとあったのに、ここでMonadインスタンスを求めている？)
(F自体はモナド構造を持たなくても良いが、その場合は専用のインタープリタが必要という意味？)

```scala
def run[F[_], A](a: Free[F, A])(implicit F: Monad[F]): F[A]
```

しかし、Consoleはモナドを形成できない。(ConsoleのためのflatMapを実装することは不可能)

```scala
seald trait Console[A] {
  def flatMap[B](f: A => Console[B]): Console[B] = this match {
    case ReadLine => ???
    case PrintLine => ???
  }
}
```

---
このモナドを形成しないConsoleを、Function0やParなどモナドを形成する他の方に変換しなければならない。
それ用の型を定義する。

```scala
trait Translate[F[_], G[_]] {
  def apply[A](f: F[A]): G[A]
}
type ~>[F[_], G[_]] = Translate[F, G]

val consoleToFunction0 = new (Console ~> Function0) {
  def apply[A](a: Console[A]) = a.toThunk
}
val consoleToPar = new (Console ~> Par) {
  def apply[A](a: Console[A]) = a.toPar
}
```

---
この型をつかって、runの実装を一般化する。

```scala
def runFree[F[_], G[_], A](free: Free[F, A])(t: F ~> G)(implicit G: Monad[G]): G[A] =
  step(free) match {
    case Return(a) => G.unit(a)
    case Suspend(r) => t(r)
    case FlatMap(Suspend(r), f) => G.flatMap(t(r))(a => runFree(f(a))(t))
    case _ => sys.error("...")
  }
```

---
F ~> Gの値を受け取り、Free[F, A]の解釈時に変換を実行する。
これにより、Free[Console, A]をFunction0[A]またはPar[A]に変換する便利な関数を定義できる。
ただし、Monad[Function0]とMonad[Par]のインスタンスが必要。

```scala
implicit val function0Monad = new Monad[Function0] {
  def unit[A](a: A) = () => a
  def flatMap[A, B](a: Function0[A])(f: A => Function0[B]) = () => f(a())()
}
def runConsoleFunction0[A](a: Free[Console, A]): () => A =
  runFree[Console, Function0, A](a)(consoleToFunction0)

implicit val parMonad = new Monad[Par] {
  def unit[A](a: A) = Par.unit(a)
  def flatMap[A, B](a: Par[A])(f: A => Par[B]) = Par.fork { Par.flatMap(a)(f) }
}
def runConsolePar[A](a: Free[Console, A]): Par[A] =
  runFree[Console, Par, A](a)(consoleToPar)
```

---
### exercise13.4
確認する

---
Free[F, A]の値(インスタンス)は、Fが提供する命令セットで記述されたプログラムのようなものである。
例えば、FがConsoleのケースでは、PrintLineとReadLineの2つの命令で作られた命令セットになる。
再起の足場となるSuspendと、モナディックな変数置換であるFlatMapとReturnは、Free自体が提供する。

異なる命令セットごとに、Fの他の選択肢を導入できる。
たとえばファイルシステムFでは、ファイルシステムでの読み書きアクセス、または読み取りアクセスのいをサポートできるし、
ネットワークFでは、ネットワーク接続を開いて読み取りを開始すると言った機能をサポートできる。

---
## 13.4.3 純粋なインタープリタ
