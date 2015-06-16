# Chapter 4 クラスとオブジェクト

---
## クラス、フィールド、メソッド

* 基本的にJavaと同じ
```scala
class ScalaClass {
  val myField = "hoge"
  private var myStatus = "01"

  def run(): String = {
    setStatus("02")
    myStatus
  }

  def end(): String = {
    setStatus("03")
    myStatus
  }

  private def setStatus(status: String) {
    myStatus = status
  }
}
```

* メソッド引数は"val"


* returnは省略できる
    * 早期returnは使わない
    * メソッドは1つの値を生成する式→設計レベルで意識する必要あり


* メソッドのシグネチャと実装の間の"="に注意
    * "="が無いと(手続き型記述)返り値がすべてUnitになる
```scala
def a() {
  "hoge"
}
def b(): Unit = "hoge"
```

---
## セミコロンの扱い

* セミコロンは基本的に不要
    * 1行に複数文記載する場合に使用する
    * 復数文またがる場合もScalaが分割

```scala
// OK
if (flag)
  println("true")
else
  println("false")

// NG
val sum = x
+ y
```


* セミコロン推論の規則
次の条件に当てはまる場合は、行末セミコロンとして扱われない
    1. 該当行の末尾が、ピリオドや中置演算子などの文の末尾として文法的に認められていない単語になっている
    1. 次の行の先頭が、文の先頭として認められない単語になっている
    1. 括弧()や角括弧[]の中にいる状態で文末になっている

---
## シングルトンオブジェクト

* Scalaはstaticメンバを持てない
    * 代わりにシングルトンオブジェクト(Singleton Object)を利用できる


* obectキーワードで指定する
```scala
object ScalaStatic {
  val value = "hoge"
}
println(ScalaStatic.value)
```


* staticと同じようなものだが、フィールドやメソッドは同名のクラスに属していない
    * Javaはクラスの共通領域で管理される
    * シングルトンオブジェクトはそれが唯一のインスタンスとして扱われる


* シングルトンオブジェクトと同じ名前のクラスがある場合、下記のような呼び方をされる
    * シングルトンオブジェクト→コンパニオンオブジェクト
    * 同名のクラス→コンパニオンクラス
    * 同一のソースファイルに記載される必要がある
    * コンパニオン同士はお互いの非公開メンバにアクセスできる
    * 逆にコンパニオンクラスのないシングルトンオブジェクトは「スタンドアロンオブジェクト」と呼ばれる

```scala
class Sample {
  private val value = "foo"
}
object Sample {
  private val value = "bar"
}
// それぞれのvalueは別物
```

* コンパニオンオブジェクトはクラスとは別のクラスやトレイトを継承・ミックスインできる


* Javaのバイトコード的にはstatic領域に無名内部クラスのインスタンスがセットされる
    * 初期化セマンティクスがJavaのstaticメンバと同じ
    * 最初にアクセスされた時に初期化される

---
## Scalaアプリケーションのエントリーポイント

* Scalaのエントリーポイント(プログラム実行開始)は、スタンドアロンオブジェクトのmainメソッド
```scala
object HogeFuga {
  def main(args: String): Unit = {
    // run program
  }
}
```


* Scalaはソースファイルに好きな名前をつけることができるが慣習的にJavaと同じ命名規則が用いられる


* Scalaのソースは暗黙的に下記のパッケージやオブジェクトがインポートされる
    * java.lang._
    * scala._
    * scala.Predef._


---
# Chapter 5 基本型と演算子

---
## 基本型

---
## リテラル

---
## シンボル(Symbol)

---
