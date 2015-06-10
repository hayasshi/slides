name: inverse
layout: true
class: center, middle, inverse

---
# 第0回　Scala社内勉強会  (キックオフ)

---
layout: false
## 目次
* Scalaとは、メリット、デメリット

* Scalaの構文紹介

* Scalaのエコシステム

* 読書会の進め方

---
## Scalaとは
* マルチパラダイム言語(オブジェクト指向言語＋関数型言語)

* 「Scala」は英語の「scalable language」に由来

* JVM上で動作し既存のJavaのプログラムと容易に連携できる

* Typesafe社が管理

* スイス連邦工科大学 (EPFL) の Martin Odersky 教授によって設計
  (マーティン・オーダスキー、小田好先生)

---
## Scalaのメリット
* Javaに比べコードを簡潔に記述できる

* FPの習慣を用いて不具合の少ないコードを書きやすい

* JVM上で動作するため既存の運用ノウハウが活かしやすい

* Javaの資産(ライブラリ等)を活用できる

* 「Scala使いです」っていうのがカッコイイ

---
## Scalaのデメリット
* 言語仕様が複雑(習得に時間がかかる)

* 糖衣構文が多く多彩な書き方ができる

上記はコードレビューを促進させる要因にもなる

* 後方バイナリ互換がない(ソースコード互換はある程度あり)

* コンパイルが遅い(笑)

---
# Scalaの構文紹介

---
### class
Javaとだいたい同じ
```java
public class JavaClass {
    // code here
}
```
```scala
class ScalaClass {
  // code here
}
```

---
### object
そのJVMプロセスで唯一のインスタンス(シングルトン)を生成
(Javaのstaticみたいなもの)
```java
public class JavaClass {
    public static final String value = "hoge";
}

// access
System.out.println(JavaClass.value);
```
```scala
object ScalaClass {
  val value = "foo"
}

// access
println(ScalaClass.value)
```

---
### case class
このキーワードで定義されたクラスは下記のメソッドがオーバーライドされた状態で生成される
* setter/getter
* toString
* equals
* hashCode

JavaBeanを簡単に作ることができる
```java
public class JavaBean {
    private String value;
    public JavaBean(String value) {
        this.value = value;
    }
    public String getValue() {
        return value;
    }
    // override toString, equals, hashCode...
}
```
```scala
case class ScalaBean(val value)
```

---
### trait
メンバ変数と実装を持てるインタフェースのようなもの
```java
public interface JavaInterface {
    default public String formatValue() {
        return String.format("[%s]", value());
    }
    public String value();
}
```
```scala
trait ScalaTrait {
  val formatString = "[%s]"
  def formatValue: String = formatString.format(value)
  abstract def value
}
```

---
### var, val
```java
String value1 = "one";
value1 = "oneone";
final String value2 = "two";
value2 = "twotwo"; // コンパイルエラー
```
```scala
var value1 = "one"
value1 = "oneone"
val value2 = "two"
value2 = "twotwo" // コンパイルエラー
```

---
### method
```java
public String addPrefix(String value) {
    return "pre" + value;
}
```
```scala
def addPrefix(value: String): String = {
  "pre" + value
}
def addPrefix(value: String) = "pre" + value
```

---
### if式
if文ではなくif式で、値を返却する
Javaの三項演算子のほうが挙動が近い
```java
String value = null;
if (flag) {
    value = "flag on";
} else {
    value = "flag off";
}
```
```scala
val value = if (flag) {
  "flag on"
} else {
  "flag off"
}

val value = if (flag) "flag on" else "flag off"
```

---
### while
Javaのwhileと同じだが基本的に使用しない
詳しくはコップ本で

---
### for式
Javaと同じように利用できるが別用途で用いることのほうが多い
yieldキーワードで値をリストで返却することもできる
```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```
```scala
for (i <- 0 to 9) {
  println(i)
}
```

---
### match式
switchみたいなものだが柔軟に色々なケースを書ける
```java
String s = null;
switch (x) {
    case 0:
        s = "zero";
        break;
    case 1:
        s = "one";
        break;
    default:
        s = "other";
        break;
}
```
```scala
val s = x match {
  case 0 => "zero"
  case 1 => "one"
  case _ => "other"
}

val i = scala.util.Random.nextInt(15) + 1
(i % 3, i % 5) match {
  case (0, 0) => println("FizzBuzz")
  case (0, _) => println("Fizz")
  case (_, 0) => println("Buzz")
  case (_, _) => println(i.toString)
}
```

---
### Function
Scalaは関数をオブジェクトとして扱える
その実態はFunctionNトレイトの無名クラス
関数リテラルで意識せずに書ける
```java
Function1<Integer, String> intToString = new Function1() {
    public String apply(Integer arg) {
        return "string: " + arg.toString();
    }
};
System.out.println(intToString.apply(1)); // string: 1
System.out.println(intToString.apply(25)); // string: 25
```
```scala
val intToString: Int => String = (i: Int) => "string: " + i.toString
println(intToString(1)) // string: 1
println(intToString(25)) // string: 25
```

---
# Scalaのエコシステム

---
### ビルドツール
* sbt

* (Gradle)

---
### IDE
* Eclipse (Scala IDE for Eclipseプラグイン)

* Intellij IDEA (Scalaプラグイン)

* Vim (Scalaプラグインいくつか)

* Emacs (プラグインあるらしいけどシラン)

* 好きなテキストエディタ＋コマンドライン

---
### web framework
* Play! Framework

* Scalatra

* spray

* skinny

---
### DBアクセスライブラリ
* slick

* scalikejdbc

---
### testing
* scalatest

* scalacheck

* scalaprops

---
### 分散処理
* akka

* Spark

---
### その他
* scalaz (関数型ライブラリ)

* scala.js (AltJS)

* GitBucket (GitHubクローン)

* Gatling (負荷ツール)

---
## 勉強会の進め方
* コップ本の目次・区切り方

* 担当決め

---
## 1クール目の目次
* 第01章(スケーラブルな言語)

* 第02章(Scalaプログラミングの第一歩)
  第03章(次の一歩)

* 第04章(クラスとオブジェクト)
  第05章(基本型と演算子)

* 第06章(関数型スタイルのオブジェクト)
  第07章(組み込みの制御構造)

* 第08章(関数とクロージャ)
  第09章(制御の抽象化)

* 第10章(合成と継承)
  第11章(Scalaの階層構造)

* 第12章(トレイト)
  第13章(パッケージとインポート)

* 第14章(表明と単体テスト)
  第15章(ケースクラスとパターンマッチ)

* 第16章(リストの操作)
  第17章(コレクション)

---
## 6月17日(水)
* 第01章(スケーラブルな言語)

* 第02章(Scalaプログラミングの第一歩)
  第03章(次の一歩)

* 第04章(クラスとオブジェクト)
  第05章(基本型と演算子)

---
## 6月24日(水)
* 第06章(関数型スタイルのオブジェクト)
  第07章(組み込みの制御構造)

* 第08章(関数とクロージャ)
  第09章(制御の抽象化)

---
## 7月1日(水)
* 第10章(合成と継承)
  第11章(Scalaの階層構造)

* 第12章(トレイト)
  第13章(パッケージとインポート)

---
## 7月8日(水)
* 第14章(表明と単体テスト)
  第15章(ケースクラスとパターンマッチ)

* 第16章(リストの操作)
  第17章(コレクション)
