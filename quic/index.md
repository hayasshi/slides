# QUIC (Quick UDP Internet Connections)

---
## 自己紹介

![hayasshi](/slides/quic/prof.png)

- GitHub: [hayasshi](https://github.com/hayasshi)
- Twitter: [@hayasshi_](https://twitter.com/hayasshi_)
- Chatwork株式会社 サーバーサイド開発部 Scala MGR
- Scala関西スタッフ

---
## サーバーサイド開発部 Scala 部署紹介

- メンバー9名
    - PHPからの移籍メンバー
    - 20卒新卒一名
    - 21卒新卒内定者もおり今後教育にも注力
- ミッション
    - 既存のScalaアプリケーションの運用保守
    - サービス機能の新規開発をScalaでおこなえる体制の整備

---
## QUICとは

QUICの現状確認をしたい
https://qiita.com/flano_yuki/items/251a350b4f8a31de47f5
https://asnokaze.hatenablog.com/entry/2019/11/25/021908

Googleが考案した**UDP上でHTTPを実現する**ためのプロトコル。  
UDP上で信頼性のある安全な通信をおこなうことで、オーバーヘッドの大きいTCP依存を減らし、Webを改善することが目的。

GoogleがChromeや自社プロダクトで使用しているGoogle QUIC(gQUIC)と、  
IETFで標準化を行っているIETF QUIC(iQUIC)がある。

互換性はないが、お互いに影響しながら改善が進められている。

---
## QUICによって改善が期待されるもの

- HTTP
    - [Hypertext Transfer Protocol Version 3 (HTTP/3)](https://tools.ietf.org/html/draft-ietf-quic-http-27)
- DNS
    - [Specification of DNS over Dedicated QUIC Connections](https://tools.ietf.org/html/draft-huitema-quic-dnsoquic-07)
- WebRTC
    - [QUIC API for Peer-to-peer Connections](https://w3c.github.io/webrtc-quic/)

その他、TCPで実現されていた様々なWebプロトコル

---
## なぜ QUIC に注目しているか

#### UDP ベースであること

すでに世界中に普及しているNW機器で対応可能で、UDP上でのプロトコル定義なので**ユーザーランド**(ソフトウェア)のみでのアップデートが可能。

#### HTTP/3 のベース技術であること

TLSによるE2Eの暗号化もあり、HTTPサーバー、クライアントソフトウェアを入れ替えるだけで、アプリケーションはこれまで通りHTTP通信を行える。

---
## なぜ QUIC に注目しているか

- 既存技術を応用して、アプリケーションの変更無しで、Web(インターネット)が早くなる
    - 既存技術を応用したスマートな改善
- システムにおける、例えばマイクロサービス間のレイテンシ低下、スループットの向上が見込める
- `fire and forget`の上でどのように信頼性を確保していくかの参考
    - アクタープログラミング
    - Akkaが選択した`Aeron`との比較
    - ゼロトラスト

---
## 今後のやっていき

- UDP上でどのようにTCPと同レベルの信頼性を確保しているか
- プロダクションで使っていくとすればどのようにしていけるか
- できれば実装していきたい
