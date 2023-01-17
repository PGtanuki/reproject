# 不動産新着物件情報のサーバレス巡回通知システム

AWS各サービスを連携させ、SUUMOの検索結果を巡回して新着物件情報を通知します。
https://suumo.jp/
※requestsを用いたクローリングは直接的に利用規約違反にはなりませんが、負荷をかけることは禁止されています。新着物件情報は頻繁には更新されないので、1時間に1回以上か、朝昼晩くらいの感じにしましょう。AWS無料枠の範囲に確実に抑える観点からもそのほうが適しています。

## 使ったAWSサービス
+ EventBridge Scheduler
+ Lambda
+ SNS
+ DynamoDB(SUUMOのみならRDSでOK)
  
## EventBridge Scheduler
2022年末にリリースされた新しいSchedulerの機能を用いています。cron記述が不要で、シンプルに「2分ごと」など指定できます。

## Lambda
コードはlambda.pyを参照。BeautifulSoupなど外部ライブラリはpipインストールなどが使えないので、Dockerを使わないでベタ書き実装する場合はレイヤー追加する必要があります。
しかし、なぜかうまく行かなかったので、arnを提供してくださっている方のarnを追加すれば問題なくいけました。
巡回するURLと、DynamoDBのパーティションキーとなる「PK」は、環境変数で指定する必要があります。

## SNS
メール送信を用いています。

## DynamoDB 
SUUMOのように単一サイトの場合はBeautifulSoupで整えた形が均一なので問題ありませんが、学習のためにもNoSQLを採用しました。サイトごとにDBを変更すれば尚更RDSで問題ありません。
設計として、パーティションキーには検索条件のタイトルをLambdaの環境変数に入れています。例えば、渋谷区10万円なら['shibuya10']といった具合です。
ほか、パーサーした各種データを放り込んでいます。
ソートキーには家賃を採用しています。
パーティションキーおよびソートキーを考えればあとは何も考えなくていいというのは、NoSQLのメリットかもしれません。たとえば、駅徒歩何分の情報は3駅まで掲載されるので、それがリストになることでリスト型を格納する必要がありますが、NoSQLでしたら考える必要はありません。



