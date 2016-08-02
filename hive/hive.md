class: center, middle

# Hive触ってみた

---

# Hiveってなに

- Hadoop(HDFS+MapReduce)をベースにしたデータウェアハウス基盤
- HiveQLと言われるSQLライクな問い合わせ言語でデータの問い合わせ、分析を行う
  - HiveQLはMapReduceジョブに変換されHadoop上で実行される
  - HiveQLはSQLに基いているがSQL-92をフルサポートはしていない
- Facebookが開発してOSS化し、現在はApache Software Foundationが管理
- AWSのEMR(Elastic MapReduce)にも含まれる


- [Apache Hive](https://cwiki.apache.org/confluence/display/Hive/Home)
- [Wikipedia Apache_Hive](https://ja.wikipedia.org/wiki/Apache_Hive)

---

# Hiveのアーキテクチャ

![Hive Architecture](hive/hive-architecture.jpg)

---

# Hiveのできる

- 様々なインタフェースから実行できる
  - Command Line
  - JDBC
  - ODBC
  - PHP
  - Python
  - Thrift
- テーブルでのデータ型がある
- SQLの演算子をサポートしている
- 組み込み関数もある

---

# Hiveのちがう

- オンラインプロセスで使っちゃう


- [Apache Hive Tutorial](https://cwiki.apache.org/confluence/display/Hive/Tutorial)

---

# Hiveのモチベーション

## たぶんMapReduceを毎回書きたくなかったんだと思う

---

# なんか似たようなのと比較

|名前   |概要                             |速度|扱えるデータ容量|
|:-----|:--------------------------------|:---:|:---:|
|Hive  |SQLを話してMapReduceを実施する蜂    |△|◯|
|Pig   |Pig語を話してMapReduceを実施するブタ |×|◯|
|Impala|SQLを話してメモリ上でMRを実施する速い鹿|◯|△|
|Presto|Impalaと大体同じで生き物ですら無い    |◯|△|
|Spark|Scalaしゃべってたのに最近はSQLも話す   |◯|△|

---

# まとめると

- アドホックな使い捨てレポートや中量のデータのリアルタイム処理
  - Impala
  - Presto
  - Spark
- 大量のデータをバッチ処理での分析、解析
  - Hive
  - Pig

---

# 動かしてみる

- [Hive GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)
  - これをもとにhiveをスタンドアロンで動かしました
- [Hue](http://jp.gethue.com/start-developing-hue-on-a-mac-in-a-few-minutes-2/)
- [HueDocker](https://github.com/cloudera/hue/tree/master/tools/docker)
  - Clouderaが公開しているQuickstartVM(Hadoopとか入ってる)と
  - Hueと呼ばれるWebアプリを接続して
  - WebUI上でHiveとかImpalaとか動かせる

---

# 思ったこと

- いずれにせよHiveのテーブル形式で保存するために事前に変換処理とか必要な気がする
  - Presto→Hive→...とかなってたのはそのせい？
- コーディング、コンパイルなしで簡単な集計等が行えるのは嬉しいかも
  - 複雑な解析系のものは扱えない印象
- 処理するデータ量によっては選択肢としてありえる
  - 色々なインタフェースを持っているのがいいと思った

---
class: center, middle

# おしまい

---
