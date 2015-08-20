## nullとの戦い
- 初のオブジェクト指向言語の設計者曰く「null参照の考案は過ちであった。10億ドル単位の損害や苦労を引き起こしてきたのである。」
    - http://developers.srad.jp/story/09/03/05/0937219/
- nullはドキュメンテーションに乏しい
```java
String message = config.getMessage();
return replaceMessage(message, "${shop_key}", shopKey);
```
- 突然のNPE！
- Javadocを書く？毎回実装を確認する？

## 救世主Optional
- java.util.Optional<T>
    - https://docs.oracle.com/javase/jp/8/api/java/util/Optional.html
- 値をラッピングする
- 型に「この値はあるかもしれないし、無いかもしれない」という文脈をもたらす
    - Listも実は文脈をもたらしているがまた別のお話
- HaskellのMaybeやScalaのOptionなど、モダンな言語には採用されている

## どういう効果があるの？
- 値がないかもしれない型なのでチェックを明示的に行う
```java
Optional<String> messgeOpt = config.getMessage();
return replaceMessage(messageOpt.orElse("${shop_key}: 初期文言"), "${shop_key}", shopKey);
```
```java
Optional<String> messgeOpt = config.getMessage();
return replaceMessage(messageOpt.orElseThrow(() -> new WrongCodingError(..))), "${shop_key}", shopKey);
```

## 基本的な使い方(生成編)
- Optional.empty()
    - 空のOptionalを返す
    - いままでのnullの代わり
- Optional.of(T value)
    - 値のあるOptionalを返す
    - valueがnullの場合はNPE
    - 絶対にnullがこないことが想定されている箇所で使用
```java
public Optional<User> findById(String userId) {
	... // DBアクセスなど
	if (es.moreEntity()) {
		return Optional.of(es.getNextEntity());
	} else {
		return Optional.empty();
	}
}
```
- Optional.ofNullable(T value)
    - valueがnullの場合は空の、nullではない場合は値のあるOptionalを返す

## 基本的な使い方(値取得編)
- Optional.isPresent()
    - 値のある場合はtrue
    - 空の場合はfalse
    - そのまま利用するとただのnullチェックになってしまう

- Optional.get()
    - 値のある場合はその値を返却する
    - 空の場合はNoSuchElementExceptionをスローする
    - あまり使わない(使わないほうが良い)

- Optional.orElseThrow(Supplier<? extends X> exceptionSupplier)
    - 値のある場合はその値を返却する
    - 空の場合はexceptionSupplierを評価した結果の例外をスローする

- Optional.orElse(T other)
    - 値のある場合はその値を返却する
    - 空の場合はotherを返却する
    - otherが計算量のかからない結果ならこちら

- Optional.orElseGet(Supplier<? extends T> other)
    - 値のある場合はその値を返却する
    - 空の場合はotherを評価し、その結果を返却する
    - otherが計算量のかかる結果ならこちら

```java
User user = User.findById("1").orElseGet(() -> User.insertAndGet("1"));
```

## 基本的な使い方(値操作編)
- Optional.ifPresent(Consumer<? super T> consumer)
    - 値がある場合はその値を引数にconsumerを評価(実行)する
    - 空の場合はなにもしない
    - Stream.forEachみたいな
```java
userOpt.ifPresent(user -> System.out.println(user.getName()))
```

- Optional.filter(Predicate<? super T> predicate)
    - 値がある場合はその値を引数にpredicateを評価し、trueならその値の、falseなら空のOptionalを返却する
    - 空の場合は空のOptionalを返却する
```java
userOpt.filter(user -> user.getName().startsWith("A"))
       .ifPresent(user -> System.out.println(user.name))
```

- Optional.map(Function<? super T,? extends U> mapper)
    - 値がある場合はその値を引数にmapperを評価し、結果のOptionalを返却する
    - 空の場合は空のOptionalを返却する
```java
userOpt.map(user -> "ユーザ名: %s".format(user.getName()))
       .ifPresent(message -> System.out.println(message))
```

- Optional.flatMap(Function<? super T,Optional<U>> mapper)
    - 値がある場合はその値を引数にmapperを評価し、その結果を返却する
    - 空の場合は空のOptionalを返却する
```java
Optinal<String> ageOpt = userOpt.flatMap(user -> user.getAgeOpt()).map(Integer::toString)
System.out.println("ユーザの年齢: " + ageOpt.orElse("設定されていません"))
```

## ダメな使い方
- nullを扱う現行システムの一部に導入
    - あっちではOptionalを使う、こっちではnullチェックをする
    - カオス
- nullを返す
```java
public Optional<User> findById(String userId) {
	if (userId == null) {
		return null; ←★
	}
	...
}
```
    - 死刑
    - コードレビューをしっかりやろう

## まとめ
- Java8以降はnullに意味を持たせない

- Optionalは想定外のNPEを発生させないための良いツールなので積極的に使おう
    - ただし現行システムとの混合は危険
    - nullが消えるわけではないので注意

## 参考引用元
[Javadoc](https://docs.oracle.com/javase/jp/8/api/java/util/Optional.html)
[Java 8 "Optional" ～ これからのnullとの付き合い方 ～](http://qiita.com/shindooo/items/815d651a72f568112910)
[Java8でのプログラムの構造を変えるOptional、ただしモナドではない](http://d.hatena.ne.jp/nowokay/20130524)
[Optionalの取り扱いかた](http://irof.hateblo.jp/entry/2015/05/05/071450)
