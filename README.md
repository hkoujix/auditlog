# 監査ログ検索ハンズオン
![Alt text](img/1565917987115.png)


## 1. 監査ログCloudTrail をAthenaで検索
![Alt text](img/1565934365602.png)

### 1.1 CloudTrail設定
CloudTrail > 証跡情報

##### 証跡の作成　をクリック
![Alt text](img/1565751102321.png)

##### 証跡名に　`cloudtrail-handson` 　を入力
![Alt text](img/1565751632364.png)

##### ストレージの場所で新しいバケットを作成しますか　 `はい`
S3バケット　`cloudtrail-handson-<アカウントID>`　を入れて作成をクリック
![Alt text](img/1565751570233.png)

##### 証跡情報に作成した証跡のS3バケットが表示されることを確認
![Alt text](img/1565751714390.png)

### 1.2 テーブルの作成
 CloudTrail > イベント履歴

##### 「Amazon Athena で高度なクエリを実行します」　をクリック 
![Alt text](img/1565751825760.png)

##### 「Amazon Athenaでのテーブルを作成」ダイアログが表示され、
保存場所に `cloudtrail-handson-<アカウントID>` を選択し、テーブル作成をクリック
![Alt text](img/1565752028642.png)

##### 「・・・作成されました」が表示されたら、キャンセルをクリック
![Alt text](img/1565752189150.png)

### 1.3 Athenaで検索
最初のアクセスでは　Get Startedをクリック
![Alt text](img/1565755240865.png)

##### Database に `default` を選択、Tablesに `cloudtra_logs_ cloudtrail_handson-<アカウントID>`があることを確認
![Alt text](img/1565756569728.png)

#### イベント別件数の集計
FROMに `cloudtra_logs_ cloudtrail_handson-<アカウントID>`　を指定し、
検索条件に今日の日付を含むように BETWEEN句を修正する。
```
SELECT
 date(from_iso8601_timestamp(eventtime)) as date,
 count(*) as count,
 eventsource,
 eventtype,
 eventname,
 awsregion,
 sourceipaddress
FROM
  cloudtrail_logs_cloudtrail_handson_925889618331
WHERE
  awsregion = 'ap-northeast-1'
  AND date_format(from_iso8601_timestamp(eventtime),'%Y/%m/%d %H:%i')
  BETWEEN '2019/08/01 00:00'
      AND '2019/08/19 00:00'
GROUP BY
  date(from_iso8601_timestamp(eventtime)),
  eventsource,
  eventtype,
  eventname,
  awsregion,
  sourceipaddress
ORDER BY
 date(from_iso8601_timestamp(eventtime)) DESC
```
##### Run queryを実行
![Alt text](img/1565756803320.png)

##### 実行結果
![Alt text](img/1565756983936.png)


####  詳細一覧のクエリ
同様に
FROMに `cloudtra_logs_ cloudtrail_handson-<アカウントID>`　を指定し、
検索条件に今日の日付を含むように BETWEEN句を修正する。
```
SELECT
 date_format(from_iso8601_timestamp(eventtime),'%Y/%m/%d %H:%i') AS datetime,
 eventsource,
 eventtype,
 eventname,
 awsregion,
 sourceipaddress,
 vpcendpointid,
 errorcode,
 useridentity,
 resources,
 serviceeventdetails
FROM
  cloudtrail_logs_cloudtrail_handson_925889618331
WHERE
  awsregion = 'ap-northeast-1'
  AND date_format(from_iso8601_timestamp(eventtime),'%Y/%m/%d %H:%i')
  BETWEEN '2019/08/01 00:00'
      AND '2019/08/19 00:00'
ORDER BY datetime DESC
```

##### 実行結果
![Alt text](img/1565757077389.png)


####参考
[CloudTrail レコードの内容](https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html)

##### eventsource 一覧

| サービス名 | イベントソース |
|:---------------|---------------------|
|ログインなど |	signin.amazonaws.com||CloudFormation |	
|S3 |	s3.amazonaws.com|
|RDS |	rds.amazonaws.com|
|EC2 |	ec2.amazonaws.com|
|IAM |	iam.amazonaws.com|
|Lambda |	lambda.amazonaws.com|
|Redshift |	redshift.amazonaws.com|
|Quicksig |	quicksight.amazonaws.com|
|SNS |	sns.amazonaws.com|
|STS |	sts.amazonaws.com|
|SQS |	sqs.amazonaws.com|
|CloudTrail |	cloudtrail.amazonaws.com|
|CloudWatch |	monitoring.amazonaws.com|
|CloudWatchLogs |	log.amazonaws.com|
|DynamoDB |	dynamodb.amazonaws.com|


## 2. Redshift 監査ログ
###  2.1 監査ログ設定

##### Redshift > クラスター
クラスターを選択し、データベースをクリック
![Alt text](img/1565762518567.png)

##### 監査ログ記録の設定をクリック
![Alt text](img/1565762679728.png)

* 監査ログ記録の有効化　`はい`
* S3バケット　新規作成を選択
* 新規バケット名　`redshift-audit-<アカウントID>`
* S3キープレフィックス `dwhaudit`　を入力して保存
![Alt text](img/1565762805461.png)

### 2.2 Amazon S3を確認
S3のバケットができたことを確認
![Alt text](img/1565763059679.png)

### 2.3 Redshiftにアクセスしてログを残す
Redshift > クラスター
![Alt text](img/1565763199902.png)

##### エンドポイント名をコピー
5439  はポート番号になるので `:` の前までがホスト名
![Alt text](img/1565763245644.png)

[DbVisualizeを導入 https://www.dbvis.com/](https://www.dbvis.com/)

##### DbVisualizerを起動
Database Server に　エンドポイントのホスト名をペースト
Database Port に `5439`
Database user id `awsuser`
Database Password  `指定されたパスワード` 
を入力して、Connect
![Alt text](img/1565763423702.png)

##### 接続が成功したら、クエリーを実行
```
SELECT
  prod_id,
  count(1) as deal_count,
  avg(quantity_sold) as average_sold_num,
  sum(quantity_sold*amount_sold) as total_sales
FROM
  sh10.sales
GROUP BY
  prod_id
ORDER BY
  total_sales DESC;
```
![Alt text](img/1565763814934.png)

##### ディスク容量の確認クエリ
```
SELECT
  trim(pgdb.datname) AS Database,
  trim(pgn.nspname) AS Schema,
  trim(a.name) AS Table,
  b.mbytes,
  a.rows
FROM (
  SELECT db_id, id, name, sum(rows) AS rows
  FROM stv_tbl_perm a
  GROUP BY db_id, id, name
) AS a
JOIN pg_class AS pgc on pgc.oid = a.id
JOIN pg_namespace AS pgn on pgn.oid = pgc.relnamespace
JOIN pg_database AS pgdb on pgdb.oid = a.db_id
JOIN (
  SELECT tbl, count(*) AS mbytes
  FROM stv_blocklist
  GROUP BY tbl
) b on a.id = b.tbl;
```
![Alt text](img/1565764105027.png)


### 2.4 Athenaで検索

##### データベースの作成
```
CREATE  DATABASE auditlogdb
```
Athenaで create database を実行
![Alt text](img/1565769926506.png)

##### 外部テーブルの作成
`<アカウントID>` にご自身のアカウントIDを入れる
* userlog
```
CREATE external TABLE auditlogdb.userlog
(
userid INTEGER,
username CHAR(50),
oldusername CHAR(50),
action CHAR(10),
usecreatedb INTEGER,
usesuper INTEGER,
usecatupd INTEGER,
valuntil TIMESTAMP,
pid INTEGER,
xid BIGINT,
recordtime VARCHAR(200)
)
PARTITIONED by(
year char(4),
month char(2),
day char(2)
)
row format delimited
lines terminated by '\n'
stored as textfile
location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1';
```
* connection ログ
```
CREATE external TABLE auditlogdb.connectionlog
(
event CHAR(50) ,
recordtime VARCHAR(200) ,
remotehost CHAR(32) ,
remoteport CHAR(32) ,
pid INTEGER ,
dbname CHAR(50) ,
username CHAR(50) ,
authmethod CHAR(32) ,
duration BIGINT ,
sslversion CHAR(50) ,
sslcipher CHAR(128) ,
mtu INTEGER ,
sslcompression CHAR(64) ,
sslexpansion CHAR(64) ,
iamauthguid CHAR(36)
)
PARTITIONED by(
year char(4),
month char(2),
day char(2)
)
row format delimited
fields terminated by '|'
stored as textfile
location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1';
```
![Alt text](img/1565771739925.png)

* activity ログ
```
CREATE external TABLE auditlogdb.activitylog
(
logtext VARCHAR(20000)
)
PARTITIONED by(
year char(4),
month char(2),
day char(2)
)
row format delimited
lines terminated by '\n'
stored as textfile
location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1';
```

#### S3の YYYY/MM/DD に分割されたデータをパーティションとして登録
S3にはデータが YYYY/MM/DD に分割されている。これをパーティションとして登録しないと検索されない。
```
ALTER TABLE auditlogdb.connectionlog ADD PARTITION (year='2019',month='08',day='14') location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1/2019/08/14/';


ALTER TABLE auditlogdb.activitylog ADD PARTITION (year='2019',month='08',day='14') location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1/2019/08/14/';

ALTER TABLE auditlogdb.userlog ADD PARTITION (year='2019',month='08',day='14') location 's3://redshift-audit-<アカウントID>/dwhaudit/AWSLogs/<アカウントID>/redshift/ap-northeast-1/2019/08/14/';
```

![Alt text](img/1565771780085.png)


#### 検索の実行
例
```
SELECT * FROM "auditlogdb"."connectionlog" limit 10;
```

![Alt text](img/1565771938898.png)


## 3. CloudTrail ログをElasticsearchで検索
![Alt text](img/1565934400798.png)

### 3.1 CloudTrail 証跡を CloudWatch Logsへ配信する
CloudTrail > 証跡情報　
##### CloudWatch Logsの設定を押す
 ![Alt text](img/1565773976177.png)

##### 新規または既存のロググループに　`CloudTrail/AuditLogGroup` を入力して次へ
![Alt text](img/1565774182620.png)

##### 許可を押す
![Alt text](img/1565774306331.png)

##### 次へ
![Alt text](img/1565774361064.png)

##### 成功するとロググループが設定される
![Alt text](img/1565774418265.png)


### 3.2 CloudWatch Logsの確認
CloudWatch > ログ　
![Alt text](img/1565774544254.png)

##### CloudTrail/AuditLogGroup をクリックし
ログストリームが出来ていることを確認する
![Alt text](img/1565774585462.png)


### 3.3 Elasticsearch Service の構成

##### 自分のIPアドレスをメモしておく
[使用中のIPアドレスを調べる http://www.cman.jp/network/support/go_access.cgi](http://www.cman.jp/network/support/go_access.cgi)

##### Elasticsearch Service を選択
![Alt text](img/1565774885968.png)

##### 新しいドメインの作成
![Alt text](img/1565774953689.png)

##### 開発およびテストを選択
バージョンは最新を選択して次へ
![Alt text](img/1565774990515.png)

* ドメイン名`handson`
* インスタンスタイプ　`r5.large.elasticsearch`  
次へ
![Alt text](img/1565775059906.png)

##### `パブリックアクセス` を選択し、
![Alt text](img/1565848878375.png)

##### テンプレートを選択から `特定のIPからのドメインアクセスを許可`　を選択
![Alt text](img/1565849014845.png)

##### メモしてある自分のIPアドレスを登録
![Alt text](img/1565849896999.png)

##### 次へ
![Alt text](img/1565849972971.png)

##### 確認をクリック
![Alt text](img/1565850021945.png)

##### ドメインが作成され、`読み込み中`となります。
![Alt text](img/1565850056727.png)

##### `アクティブ` になったら　Kibanraをクリック
![Alt text](img/1565870212042.png)

##### KibanaにアクセスできればOK
`Explore on my own` をクリック
![Alt text](img/1565851888083.png)

##### Kibanaの管理画面が表示される
![Alt text](img/1565851992411.png)


### 3.4 CloudWatch LogsからLamndaの構成
CloudWatch > ログ　
`CloudTrail/AuditLogGroup` を選択
![Alt text](img/1565852132696.png)

##### アクションから　Amazon Elasticsearch Serviceへのストリーム　をクリック
![Alt text](img/1565852187171.png)

##### Amazon ES クラスター `handson` を指定
![Alt text](img/1565870341388.png)

##### Lambda IAM  実行ロール　`新しいIAMロールの作成`を選択
![Alt text](img/1565852441886.png)

##### IAMロールの作成で、許可　をクリック
![Alt text](img/1565852513573.png)

##### 次へ
![Alt text](img/1565870445109.png)

##### ログの形式　で　`AWS CloudTrail` を選択
![Alt text](img/1565852663492.png)

##### 次へ
![Alt text](img/1565852700720.png)

##### 次へ
![Alt text](img/1565870572728.png)

##### ストリーミングの開始
![Alt text](img/1565852746005.png)

##### サブスクリプションフィルターが作成される
![Alt text](img/1565870686011.png)

### 3.5 Lambdaの確認
Lambda ＞関数
関数をクリック
![Alt text](img/1565870774459.png)

##### `LogsToElasticsearch_handson`が表示される
![Alt text](img/1565870835104.png)

##### 関数コードを開き
![Alt text](img/1565853037392.png)

63行目　`cwl-` 　が Elasticsearchのindex pattern になるため、書き変える
![Alt text](img/1565853007548.png)

`cwl-` を `ctl-`  に書き変える
```
        // index name format: ctl-YYYY.MM.DD
        var indexName = [
            'ctl-' + timestamp.getUTCFullYear(),              // year
            ('0' + (timestamp.getUTCMonth() + 1)).slice(-2),  // month
            ('0' + timestamp.getUTCDate()).slice(-2)          // day
        ].join('.');
```

##### 保存する
![Alt text](img/1565871173922.png)


##### また、ここのコードをコピーして、メモ帳に保存しておく

### 3.6 KibanaでIndexを作成する
##### Kibana  > 左端メニューの ` Management` をクリック
![Alt text](img/1565853732831.png)

##### Index Patterns >  Create index
![Alt text](img/1565853837443.png)

##### Index patternに　`ctl-` を入力し、Next Step

![Alt text](img/1565853898812.png)

##### Time Filter field name に `@timestamp` を選択し Create index pattern
![Alt text](img/1565854016929.png)

##### フィールドが表示されればOK
![Alt text](img/1565854047242.png)

##### Kibana  > 左端メニューの ` Discover` をクリック
`ctl-*` のIndexが表示されることを確認する
![Alt text](img/1565854128387.png)

##### 再び　Kibana  > 左端メニューの `Management` をクリック
![Alt text](img/1565854294217.png)

##### Saved Objects を選択し、import をクリック
![Alt text](img/1565854364711.png)

##### Import saved objects　にファイル
`cloudtrail-kibana-dashboard.json` を指定して、import
![Alt text](img/1565854516901.png)

##### New index に `ctl` を選択して　Confirm all changes 
![Alt text](img/1565854775537.png)

##### Done をクリック
![Alt text](img/1565854847327.png)

##### Main-Dashboard および　複数のVisualizeがインポートされる
![Alt text](img/1565854889719.png)

##### Kibana  > 左端メニューの `Dashboard` をクリック
![Alt text](img/1565855396302.png)

##### Dashboardが表示される
![Alt text](img/1565871865559.png)

##### filter 条件を ✕ で削除
![Alt text](img/1565871910103.png)

##### Dashboardに表示される
Indexを作成した直後のため、まだログがなく、フィルターを外さないとほとんど出て来ないため、フィルター条件を外している
![Alt text](img/1565872040289.png)

## 4. Flowlogを Elasticsearchで検索

### 4.1 Flowlogの作成
VPC > VPCを選択 > アクション > フローログの作成
![Alt text](img/1565875124031.png)

##### 送信先ロググループ　`/aws/flowlog` 
![Alt text](img/1565875341369.png)

##### 権限の設定をクリック
![Alt text](img/1565875470544.png)

* IAMロール　`新しいIAMロールの作成` 
* ロール名  `HandonFlowlogsRole`
* を入力し、許可
![Alt text](img/1565875539416.png)

##### IAMロール名  `HandonFlowlogsRole` を選択し、作成
![Alt text](img/1565875790329.png)

##### 作成されたら　閉じるをクリック
![Alt text](img/1565875865356.png)

### 4.2  CLoudWatchLogからElasticsarchに配信するLambdaの作成

 Lambda > 関数 > 関数の作成
![Alt text](img/1565876230658.png)

##### 一から作成を選択
 ![Alt text](img/1565876409596.png)

* 関数名 `FlowLogsToElasticsearch_handson`
* ランタイム　`Node.js 8.10` 
* 実行ロール　`既存のロールを使用する`
* 既存のロール　`lambda_elasticsearch_execution`
を設定し、関数の作成
![Alt text](img/1565876455251.png)

##### トリガーを追加をクリック
![Alt text](img/1565876661540.png)

##### トリガーの設定に `CloudWatch Logs`を選択
![Alt text](img/1565876810945.png)

##### 追加をクリック
* ロググループ　`/aws/flowlog`
* フィルターの名前 `flowlogES`
* フィルターパターン `[version, account_id, interface_id, srcaddr != "-", dstaddr != "-", srcport != "-", dstport != "-", protocol, packets, bytes, start, end, action, log_status]`
![Alt text](img/1565877039021.png)

##### CloudWatch LogsのトリガーができればOK
![Alt text](img/1565877293854.png)

##### FlowLogsToElasticsearch_handsonを選択し、開発コードを表示
![Alt text](img/1565877393123.png)

##### 前章でメモ帳に保存したLogsToElasticsearch_handson の関数コードをここの開発コードにペーストする
63行目を `flowlog-` に書き変えて保存する
![Alt text](img/1565877789475.png)

### 4.3  CloudWath log の確認
CloudWatch > ログ
サブスクリプションが設定されていることを確認する
![Alt text](img/1565877913148.png)

### 4.4 Kibana
Kibana > Management ＞　Index patterns > Create index pattern
Index pattern に `flowlog-`を入れて　Next Step
![Alt text](img/1565878058410.png)

##### Time Filter field name　に `@timestamp` を選択し、Create index patternを押す
![Alt text](img/1565878295538.png)

##### dashboardのimport
Management  > Saved Objects > Import
![Alt text](img/1565881358872.png)

##### flow-dashboard.json をimport
![Alt text](img/1565881421214.png)

##### import成功したら　done
![Alt text](img/1565881490639.png)

##### Dashboard　>  FlowDashboard
![Alt text](img/1565881590873.png)

##### 以上、dashboardを自由にカスタマイズしてください。
