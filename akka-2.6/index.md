# Akka 2.6.0 リリースノートを読む

---
## Akka 2.6.0 Released 🎉

Akka が二年半ぶりに大型のアップデートをおこないました。
コンセプト自体に変更はなく、しかしより堅牢に Akka を利用するための重要な変更が含まれています。

Akka 2.6.0 になって新しく追加された機能ハイライトを、[リリースノート](https://akka.io/blog/news/2019/11/06/akka-2.6.0-released)をもとに見ていきたいと思います。

---
## ハイライト

- Akka Typed が安定版へ
- Akka Remote におけるメッセージ用トランスポート周りが刷新
- メッセージシリアライゼーションが`Jackson`ベースのものに刷新
- Akka 内部(≒`ActorSystem`内?)で使われる`dispatcher`をユーザー領域のものと分離
- Akka Streams 実行時に`Materializer`生成不要に変更(`ActorSystem`のみ。暗黙に生成され渡される)
- Akka Streams における上流キャンセルの原因を伝搬できるようにし、例えば Akka HTTP のエラーハンドリングを改善などができるように修正
- Akka Cluster Sharding のエンティティのパッシベーションをデフォルトで有効に変更
- ドキュメントとサンプルプロジェクトの改善
- Akka Typed の`SLF4J logging API`を追加
- 変更可能性のあるAPIへ`may change`の付与と非推奨機能の削除

私の興味のあるものを抜粋して少し深ぼります。

---
## Akka Typed が安定版へ

型安全にメッセージパッシングができるよ！ やったね[Akkaちゃん](https://twitter.com/akkachanjp)！

すでにプロダクション利用可能と宣言されています。他のモジュール向けの統合用APIもすでに提供されています。

Untyped なアクターと、スーパービジョンやライフサイクルフックの周りの挙動が少し変わっているので、注意が必要です。
詳細は下記あたりにまとめています。
(`2.5.17`のときのものですが、簡単にドキュメントを確認した感じ変わっていなさそうでした。また時間があれば検証します。)

[スーパービジョン](https://speakerdeck.com/hayasshi/akka-typed-typesafe-messaging?slide=39) [ドキュメント](https://doc.akka.io/docs/akka/2.6/typed/fault-tolerance.html)
[ライフサイクルフック](https://speakerdeck.com/hayasshi/akka-typed-typesafe-messaging?slide=50) [ドキュメント](https://doc.akka.io/docs/akka/2.6/typed/actor-lifecycle.html)

---
## Akka Remote におけるメッセージ用トランスポート周りが刷新

Akka Remote のTCP通信周りが、[Artery](https://doc.akka.io/docs/akka/current/remoting-artery.html#what-is-new-in-artery)(PJ名)のものに置き換え。
TCPが`Netty`から`Akka Streams TCP/TLS`ベースのものへ、UDPが[`Aeron`](https://github.com/real-logic/Aeron)という方式のものへ変更。

これにより、`high-throughput, low-latency communication`を達成(?)


---
## Akka Streams TCP/TLS

TODO:調べる

`SourceRef`, `SinkRef`などの Distributed Akka Streams で使われている通信を使っているのではないかと予想しています。
また時間のあるときに見たいとおいます。

https://doc.akka.io/docs/akka/current/stream/stream-io.html#streaming-tcp
https://doc.akka.io/docs/akka/2.6/project/migration-guide-2.5.x-2.6.x.html#remoting
https://doc.akka.io/docs/akka/current/stream/stream-refs.html


---
## メッセージシリアライゼーションが`Jackson`ベースのものに刷新

https://doc.akka.io/docs/akka/2.6/serialization-jackson.html

いままで Java シリアライゼーション形式でしたが、Akka Remoteの刷新に伴い、デフォルトのSerDesが`Jackson`ベースのものに置き換わりました。
これにより、オブジェクトのバージョンアップに伴う変更が行いやすくなりました。

`Jackson`の依存が入ってしまうので、そこは注意が必要です。[Jackson 2.10.0 に依存](https://github.com/akka/akka/blob/v2.6.0/project/Dependencies.scala#L24)
もちろんこれまで通り、自分で実装したSerDesを利用することも可能です。


---
## Akka 内部(≒`ActorSystem`内?)で使われる`dispatcher`をユーザー領域のものと分離

これまで Akka のシステム内で使われている`dispatcher`は、`default-dispatcher`が使われていましたが、新しく定義された[`internal-dispatcher`](https://github.com/akka/akka/blob/v2.6.0/akka-actor/src/main/resources/reference.conf#L546-L558)が使われるようになります。

これにより、意図せずユーザー側で`default-dispatcher`のスレッドをブロックしてしまっても Akka 内部は守られた状態になります。


---
## Akka Streams 実行時に`Materializer`生成不要に変更(`ActorSystem`のみ。暗黙に生成され渡される)

```scala
implicit val system       = ActorSystem("akka-sample")
implicit val materializer = ActorMaterializer()
Source.single("Hello World!").runForeach(println)
```
↓
```scala
implicit val system       = ActorSystem("akka-sample")
Source.single("Hello World!").runForeach(println)
```

少しコードを見た感じ`akka.stream.Materializer`内で`ActorSystem`からの暗黙の変換がおこなわれている。
作られる`Materiazlier`は Akka Extentions を使って管理され、`ActorSystem`毎に一つだけ作られるようになっている。

https://github.com/akka/akka/blob/v2.6.0/akka-stream/src/main/scala/akka/stream/Materializer.scala#L193-L197
https://github.com/akka/akka/blob/v2.6.0/akka-stream/src/main/scala/akka/stream/SystemMaterializer.scala


---
## まとめ

- かなり内部的にも大きな変更があるので、自分たちのユースケースにあったパフォーマンス評価とかはやっておいたほうが良さそう
- 手を動かして諸々確認します
