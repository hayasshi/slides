# 13.5 ノンブロッキングの非同期I/O

---
## 結論

- 「ノンブロッキングまたは非同期I/O」を実行するという問題の回答
- 「ノンブロッキングまたは非同期I/O」を扱うJavaAPIはコールバックを渡して結果を扱う
- 「ノンブロッキングまたは非同期I/O」を扱うJavaAPIを並列処理を扱うモナドでラップすることでFreeでも扱えるようにする

---
## では

13.4の最初に上げた問題の、残っているほう、「ノンブロッキングまたは非同期I/O」を実行するという問題に目を向ける。
具体的にはFreeインタープリタの実装で、I/Oを実行する際に、完了に時間がかかるものの、CPUは占有しない演算をどうあつかうか。

13.4であつかったI/OはParで実行することはできるが、ブロッキングI/Oであるので、「ノンブロッキングまたは非同期I/O」ではない。
「ノンブロッキングまたは非同期I/O」を扱うI/Oライブラリが用意されていれば、Parとバインドさせることで扱うことができる。

---
### 例題

下記のようなインターフェイスのノンブロッキングI/OのAPIがある。

```scala
trait Source {
  // readBytesはノンブロッキングで、制御がすぐに戻る
  def readBytes(numBytes: Int, callback: Either[Throwable, Array[Byte]] => Unit): Unit
}
```

これはそのままでは取り扱うのは難しいが、Par型を利用すれば、これらのコールバックをラッピングできる。

---
```scala
// 7.4.4で定義されている`parallelism/Nonblocking.scala`
trait Futre[+A] {
  private[parallelism] def apply(k: A => Unit): Unit
}
type Par[+A] = ExecutorService => Future[A]
```

このFutureの内部表現は、さきほどのSourceのものに似ている。
Futureの内部表現はすぐに制御を戻す単一のメソッドで、型Aの値が利用可能になった時点で呼び出されるコールバック(継続k)が指定されている。

---
Source.readBytesをFutureでラッピングするのが手っ取り早い方法だが、そのためにParにプリミティブを追加する必要がある。

```scala
def async[A](f: (A => Unit) => Unit): Par[A] = es => new Future[A] {
  def apply(k: A => Unit) = f(k)
}
```

これを利用して、非同期関数readBytesをParの便利なモナドインターフェイスにラッピングできる。

---
```scala
def nonblockingRead(source: Source, numBytes: Int): Par[Either[Throwable, Array[Byte]]] =
  async { (cb: Eithre[Throwable, Array[Byte]] => Unit) => source.readBytes(numBytes, cb) }

def readPar(source: Source, numBytes: Int): Free[Par, Either[Throwable, Array[Byte]]] =
  Suspend(nonblockingRead(source, numBytes))
```

---
for内包表記をつかって、チェーンを生成することもできる。

```scala
val src: Source = ???
val prog: Free[Par, Unit] = for {
  chunk1 <- readPar(src, 1024)
  chunk2 <- readPar(src, 1024)
  ...
} yield ()
```
