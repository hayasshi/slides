# Chapter 2

## Scalaプログラミングの第一歩
* Scalaインタープリタ(REPL)
* 変数、定数
* 関数(メソッド)
* whileループ
* ifループ
* インクリメンタル演算子、デクリメント演算子はない
* foreachとforによる反復実行
* Scalaスクリプト

# Chapter 3

## Scalaプログラミングの次の一歩
* 配列と型パラメータ
```scala
val greetString = new Array[String](3)
greetString(0) = "Hello"
greetString(1) = ", "
greetString(2) = "world!"
for (i <- 0 to 2) {
  println(greetString(i))
}

greetString(2) = "hayasshi!"
for (i <- 0 to 2) {
  println(greetString(i))
}
```

* Scalaのメソッドについて
```scala
greetString(0) == greetString.apply(0)
(greetString(0) = "foobar") == greetString.update(0, "foobar")
```

* リスト
```scala
val list1 = List(1, 2, 3) // List.apply[Int](1, 2, 3)
val list2 = 1 :: 2 :: 3 :: Nil
println(list1 == list2)
println(list1 ::: list2) // list1 ++ list2
```

* リストメソッド
    * Scalaのリストメソッドは豊富
    * http://www.ne.jp/asahi/hishidama/home/tech/scala/collection/method.html

* タプル
```scala
val t1 = (1, "two") // Tuple2.apply[Int, String](1, "two")
println(t1._1)
println(t1._2)
val (a, b) = t1 // t1.unapply
val t23 = (1, "two", '3', ..., 23)
```

* 集合とマップ
    * scala.collection.Set
        * scala.collection.immutable.Set
        * scala.collection.mutable.Set
        ```scala
        val mSet = scala.collection.mutable.Set(1, 2, 3)
        val imSet = scala.collection.immutable.Set(1, 2, 3)
        println(mSet += 4)  // Set(1, 2, 3, 4)
        println(imSet += 4) // Set(1, 2, 3, 4)
        println(mSet)  // Set(1, 2, 3, 4)
        println(imSet) // Set(1, 2, 3)
        ```
    * scala.collection.Map
        * scala.collection.immutable.Map
        * scala.collection.mutable.Map
        ```scala
        val mMap = scala.collection.mutable.Map(1 -> "one", 2 -> "two")
        val imSet = scala.collection.immutable.Map(1 -> "one", 2 -> "two")
        println(mMap += (3 -> "three"))  // Map(1 -> "one", 2 -> "two", 3 -> "three")
        println(imMap += (3 -> "three")) // Map(1 -> "one", 2 -> "two", 3 -> "three")
        println(mMap)  // Map(1 -> "one", 2 -> "two", 3 -> "three")
        println(imMap) // Map(1 -> "one", 2 -> "two")
        ```

* List, Set, Mapについては少々情報が古くなっているので下記資料を参考
    * http://www.ne.jp/asahi/hishidama/home/tech/scala/collection/index.html
    * http://www.scala-lang.org/api/2.11.6/#package

* 関数型のスタイルを見つける
    * varとval
    * 副作用のない関数、メソッド
        * 副作用を局所化しテストしやすい設計を
    * まずはval、イミュータブルオブジェクト、副作用のないメソッドを使うことを心がける
        * どう書いてよいかわからない時は相談しよう
        * 明確な理由がある場合のみ、var、ミュータブルオブジェクト、副作用のあるメソッドをつかう

* ファイルから行を読み出す
    * scala.io.Sourceについて書いてあるが色々腐ってるので基本的につかわない
    * java.ioパッケージを使う
    * Scalaっぽく変換もできる(http://kmizu.hatenablog.com/entry/20121128/1354115941)