---
title: "Meltanoとtap-jquantsを用いたELT"
emoji: "🐉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [meltano,j-quants,elt,etl,finance]
published: false
---
このドキュメントでは、Meltanoを用いて、J-Quants APIから[tap-jquants](https://github.com/stn/tap-jquants)を用いてデータを取得する手順を説明します。

[J-Quants API](https://jpx.gitbook.io/j-quants-ja/)が正式リリースされ、個人投資家にはこれまで取得が難しかったデータへのアクセスが容易になりました。
公式にも[jquants-api-client-python](https://github.com/J-Quants/jquants-api-client-python)をはじめとした[さまざまな言語のクライアント](https://github.com/J-Quants)が公開されています。

J-Quants APIを用いると市場全体の分析のために必要なさまざまなデータを取得できます。
しかし、APIは日々、追加されるデータを含み、毎回、すべてを取得しては時間がかかってしまいます。
このようなケースではETL/ELTパイプラインを構築し、過去のデータをキャッシュし、新たなデータをインクリメンタルに取得し、分析するという手法が有効です。

とはいえ、ETL/ELTパイプラインを構築するためには、前回はどこまでのデータが取得されたのか、次回はどこから取得すればよいのかといった情報を管理し、
取得したデータをデータベースに保存し、更新されたデータに基づいて分析を行うためのコードを書く必要があります。

このようなELTパイプラインを簡単に構築できる[Meltano](https://meltano.com/)というツールがあります。
Meltanoはさまざまなプラグイン([Meltano Hub](https://hub.meltano.com/))を組み合わせることで、データの抽出 (Extract)、ロード (Load)、変換 (Transform)を行うことができます。



## 前提条件

このドキュメントでは、以下の前提条件を想定しています。

- [J-Quants API](https://jpx-jquants.com/)のアカウントを持っていること。
- Python 3.7, 3.8, 3.9, 3.10, 3.11のいずれかがインストールされていること。
  - このドキュメントでは、Python 3.9で動作を確認しています。

## Meltanoのインストール

Meltanoは[meltano/Installation](https://docs.meltano.com/getting-started/installation)にもある通り[pipx](https://pypa.github.io/pipx/)で簡単にインストールすることができます。

```shell
$ pipx install meltano
$ meltano --version
meltano, version 2.18.0
```

## Meltanoプロジェクトの作成

Meltanoプロジェクトを作成します。

```bash
$ meltano init meltano-jquants
Creating .meltano folder
created .meltano in /Users/hoge/jquants/meltano-jquants/.meltano
Creating project files...
  meltano-jquants/
   |-- meltano.yml
   |-- README.md
   |-- requirements.txt
   |-- output/.gitignore
   |-- .gitignore
   |-- extract/.gitkeep
   |-- load/.gitkeep
   |-- transform/.gitkeep
   |-- analyze/.gitkeep
   |-- notebook/.gitkeep
   |-- orchestrate/.gitkeep
Creating system database...  Done!



                          ████   █████
                         ░░███  ░░███
 █████████████    ██████  ░███  ███████    ██████   ████████    ██████
░░███░░███░░███  ███░░███ ░███ ░░░███░    ░░░░░███ ░░███░░███  ███░░███
 ░███ ░███ ░███ ░███████  ░███   ░███      ███████  ░███ ░███ ░███ ░███
 ░███ ░███ ░███ ░███░░░   ░███   ░███ ███ ███░░███  ░███ ░███ ░███ ░███
 █████░███ █████░░██████  █████  ░░█████ ░░████████ ████ █████░░██████
░░░░░ ░░░ ░░░░░  ░░░░░░  ░░░░░    ░░░░░   ░░░░░░░░ ░░░░ ░░░░░  ░░░░░░



Your project has been created!

Meltano Environments initialized with dev, staging, and prod.
To learn more about Environments visit: https://docs.meltano.com/concepts/environments

Next steps:
  cd meltano-jquants
  Visit https://docs.meltano.com/getting-started/part1 to learn where to go from here
```

## Extractor

ExtractorはELTパイプラインの入り口となる、データを取得するためのプラグインです。
[Meltano Hub](https://hub.meltano.com/extractors)を見ると、様々なExtractorが公開されていることが分かります。

### tap-jquantsのインストール

[tap-jquants](https://github.com/stn/tap-jquants)は現在、開発中でMeltano Hubに登録できていませんので、custom extractorとして設定します。
将来的には以下のインストールは `meltano add extractor tap-jquants` でインストールできるようになると思います。

```bash
$ cd meltano-jquants
$ meltano add extractor tap-jquants --custom
Adding new custom extractor with name 'tap-jquants'...

Specify the plugin's namespace, which will serve as the:
- identifier to find related/compatible plugins
- default database schema (`load_schema` extra),  for use by loaders that support a target schema

Hit Return to accept the default: plugin name with underscores instead of dashes

(namespace) [tap_jquants]:
```

ここはデフォルトのままで構いませんので、そのままenterを押します。

```bash
Specify the plugin's `pip install` argument, for example:- PyPI package name:
	tap-jquants
- Git repository URL:
	git+https://<PLUGIN REPO URL>.git
- local directory, in editable/development mode:
	-e extract/tap-jquants
- 'n' if using a local executable (nothing to install)

Default: plugin name as PyPI package name

(pip_url) [tap-jquants]:
```

ここは、gitリポジトリを `git+https://github.com/stn/tap-jquants.git` と指定します。

```bash
(pip_url) [tap-jquants]: git+https://github.com/stn/tap-jquants.git

Specify the plugin's executable name

Default: name derived from `pip_url`

(executable) [tap-jquants]:
```

ここは、そのままenter

```bash
Specify the tap's supported Singer features (executable flags), for example:
	`catalog`: supports the `--catalog` flag
	`discover`: supports the `--discover` flag
	`properties`: supports the `--properties` flag
	`state`: supports the `--state` flag

To find out what features a tap supports, reference its documentation or try one of the tricks underhttps://docs.meltano.com/guide/integration#troubleshooting.

Multiple capabilities can be separated using commas.

Default: no capabilities

(capabilities) [[]]:
```

`state,catalog,discover,about,stream-maps` と指定します。

```bash
(capabilities) [[]]: state,catalog,discover,about,stream-maps

Specify the extractor's supported settings
Multiple setting names (keys) can be separated using commas.

A setting kind can be specified alongside the name (key) by using the `:` delimiter,
e.g. `port:integer` to set the kind `integer` for the name `port`

Supported setting kinds:
string | integer | boolean | date_iso8601 | email | password | oauth | options | file | array | object | hidden

- Credentials and other sensitive setting types should use the password kind.
- If not specified, setting kind defaults to string.
- Nested properties can be represented using the `.` separator, e.g. `auth.username` for `{ "auth": { "username": value } }`.
- To find out what settings a extractor supports, reference its documentation.

Default: no settings

(settings) [[]]:
```

`mail_address,password:password,start_date:date_iso8601` と指定します。

```bash
(settings) [[]]: mail_address,password:password,start_date:date_iso8601
Added extractor 'tap-jquants' to your Meltano project

Installing extractor 'tap-jquants'...
Installed extractor 'tap-jquants'
```

以上で、tap-jquantsがインストールされました。

### tap-jquantsのconfig

tap-jquantsの必須の設定は以下の3つです。

- mail_address: J-Quants APIに登録したメールアドレス
- password: J-Quants APIに登録したパスワード
- start_date: 取得開始日 YYYY-MM-DD

これらの設定は、`meltano config tap-jquants set --interactive` でCUIで設定することもできますし、
以下のようにコマンドラインから設定することもできます。

（コマンドラインで設定する場合には、パスワードがコマンド履歴などに残らないように注意してください）

```bash
$ meltano config tap-jquants set mail_address 'taro.yamada@example.com'
$  meltano config tap-jquants set password 'abcdefg1234!@#'
```

start_dateは、今日の10日程度前の日を設定しておくとデータの取得が早く終わります。

```bash
$ meltano config tap-jquants set start_date '2023-05-10'
```

Meltanoの設定は、`meltano.yml` に保存されます。
また、パスワードは `.env` に保存されます。

### tap-jquantsのストリーム

次に、J-Quants APIからどのデータを取得するかを設定します。

tap-jquantsがサポートしている項目は次のようにして確認できます。

```bash
$ meltano select tap-jquants --list --all
Legend:
	selected
	excluded
	automatic

Enabled patterns:
	*.*

Selected attributes:
	[selected ] daily_quotes.adjustment_close
	[selected ] daily_quotes.adjustment_factor
	[selected ] daily_quotes.adjustment_high
...
```

デフォルトでは「すべて」選択されていますので、 必要な項目だけ選択するようにします。
ここでは、[株価四本値](https://jpx.gitbook.io/j-quants-ja/api-reference/daily_quotes)のみを取得するようにします。

```bash
$ meltano select tap-jquants daily_quotes '*'
```

指定後、再度 `meltano select tap-jquants --list --all` を実行すると `daily_quotes.*` だけが選択されていることが確認できます。

これで、tap-jquantsの設定は完了です。

## Loader

取得したデータを格納するのがLoaderです。

[Meltano HubのLoaders](https://hub.meltano.com/loaders/)ページでは、さまざまなLoaderが紹介されています。
有名なデータベースはほぼ揃っていますし、ファイルに出力するものであったり、S3やGCSに直接データをロードするものもあります。

ここまでの設定が正しいかを確認するために、[target-csv](https://hub.meltano.com/loaders/target-csv) を用いてテストしてみましょう。

```bash
$ meltano add loader target-csv
$ meltano run tap-jquants target-csv
2023-05-19T06:06:04.789340Z [info     ] Environment 'dev' is active
2023-05-19T06:06:07.095374Z [info     ] 2023-05-19 15:06:07,095 | INFO     | tap-jquants          | Skipping deselected stream 'announcement'. cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T06:06:07.095651Z [info     ] 2023-05-19 15:06:07,095 | INFO     | tap-jquants          | Skipping deselected stream 'breakdown'. cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T06:06:07.095721Z [info     ] 2023-05-19 15:06:07,095 | INFO     | tap-jquants          | Beginning incremental sync of 'daily_quotes'... cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T06:06:07.095874Z [info     ] 2023-05-19 15:06:07,095 | INFO     | tap-jquants          | Tap has custom mapper. Using 1 provided map(s). cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T06:06:07.096207Z [info     ] 2023-05-19 15:06:07,095 | INFO     | tap-jquants          | URL params: {'date': '2023-05-10'} cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T06:06:08.019936Z [info     ] INFO Sending version information to singer.io. To disable sending anonymous usage data, set the config parameter "disable_collection" to true cmd_type=elb consumer=True name=target-csv producer=False stdio=stderr string_id=target-csv
2023-05-19T06:06:12.653796Z [info     ] 2023-05-19 15:06:12,652 | INFO     | singer_sdk.metrics   | METRIC: {"type": "timer", "metric": "http_request_duration", "value": 3.987396, "tags": {"stream": "daily_quotes", "endpoint": "/prices/daily_quotes", "http_status_code": 200, "status": "succeeded"}} cmd_type=elb consume
...
2023-05-19T06:06:43.255001Z [info     ] Incremental state has been updated at 2023-05-19 06:06:43.254932.
2023-05-19T06:06:43.259509Z [info     ] Block run completed.           block_type=ExtractLoadBlocks err=None set_number=0 success=True
```

1行目で`target-csv`をインストールし、2行目で`tap-jquants`から`target-csv`へのEL(T)パイプラインを起動しています。

出力は、`output/daily_quotes-20230519T150608.csv`のように実行時の日時がついたファイルに保存されます。

みんな大好きCSVですが、`daily-quotes`は`date`と`code`をキー属性として持つにも関わらず、CSVではキー属性が使われず、また、型の情報も失わてしまうなど、データベースに比べて扱いにくいフォーマットです。

### target-sqlite

そこで、データベースに直接データをロードするLoaderを使ってみましょう。
ここでは、ローカルでも簡単に使える[SQLite](https://hub.meltano.com/loaders/target-sqlite)を例に解説します。

インストールは簡単で次の1行だけです。

```bash
$ meltano add loader target-sqlite
Added loader 'target-sqlite' to your Meltano project
Variant:	meltanolabs (default)
Repository:	https://github.com/MeltanoLabs/target-sqlite
Documentation:	https://hub.meltano.com/loaders/target-sqlite--meltanolabs

Installing loader 'target-sqlite'...
Installed loader 'target-sqlite'

To learn more about loader 'target-sqlite', visit https://hub.meltano.com/loaders/target-sqlite--meltanolabs
```

設定はデフォルトのままでも動きますので、早速 `target-csv` と同様に実行してみましょう。

```bash
$ meltano run tap-jquants target-sqlite
2023-05-19T08:59:20.348241Z [info     ] Environment 'dev' is active
2023-05-19T08:59:22.122964Z [warning  ] No state was found, complete import.
2023-05-19T08:59:25.209922Z [info     ] 2023-05-19 17:59:25,209 | INFO     | tap-jquants          | Skipping deselected stream 'announcement'. cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:25.210136Z [info     ] 2023-05-19 17:59:25,209 | INFO     | tap-jquants          | Skipping deselected stream 'breakdown'. cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:25.210236Z [info     ] 2023-05-19 17:59:25,209 | INFO     | tap-jquants          | Beginning incremental sync of 'daily_quotes'... cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:25.210300Z [info     ] 2023-05-19 17:59:25,209 | INFO     | tap-jquants          | Tap has custom mapper. Using 1 provided map(s). cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:25.210491Z [info     ] 2023-05-19 17:59:25,210 | INFO     | tap-jquants          | URL params: {'date': '2023-05-10'} cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:30.272166Z [info     ] 2023-05-19 17:59:30,271 | INFO     | singer_sdk.metrics   | METRIC: {"type": "timer", "metric": "http_request_duration", "value": 3.955928, "tags": {"stream": "daily_quotes", "endpoint": "/prices/daily_quotes", "http_status_code": 200, "status": "succeeded"}} cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:31.031070Z [info     ] 2023-05-19 17:59:31,030 | INFO     | tap-jquants          | URL params: {'date': '2023-05-11'} cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
2023-05-19T08:59:35.071721Z [info     ] 2023-05-19 17:59:35,071 | INFO     | singer_sdk.metrics   | METRIC: {"type": "timer", "metric": "http_request_duration", "value": 3.8846, "tags": {"stream": "daily_quotes", "endpoint": "/prices/daily_quotes", "http_status_code": 200, "status": "succeeded"}} cmd_type=elb consumer=False name=tap-jquants producer=True stdio=stderr string_id=tap-jquants
...
2023-05-19T09:00:06.441495Z [info     ] Incremental state has been updated at 2023-05-19 09:00:06.441438.
2023-05-19T09:00:06.452339Z [info     ] Block run completed.           block_type=ExtractLoadBlocks err=None set_number=0 success=True
```

無事に終了すると `warehouse.db` が作成されます。

sqlite3で確認してみます。

```bash
❯ sqlite3 warehouse.db
SQLite version 3.39.5 2022-10-14 20:58:05
Enter ".help" for usage hints.
sqlite> .table
daily_quotes
sqlite> SELECT date, COUNT(*) FROM daily_quotes GROUP BY date;
2023-05-10|4267
2023-05-11|4267
2023-05-12|4270
2023-05-15|4270
2023-05-16|4270
2023-05-17|4270
2023-05-18|4270
2023-05-19|4269
sqlite> .quit
```

## ステート管理

Meltanoは、データの取得状況をステート(state)として管理します。

```bash
$ meltano state list
dev:tap-jquants-to-target-csv
dev:tap-jquants-to-target-sqlite
```

さきほど実行した `meltano run tap-jquants target-sqlite` のステートを確認してみます。

```bash
$ meltano state get dev:tap-jquants-to-target-sqlite
{"singer_state": {"bookmarks": {"daily_quotes": {"replication_key": "date", "replication_key_value": "2023-05-19"}}}}
```

最後に取得したデータの`date`の値がステートとして保存されています。
これによって、再度、`meltano run tap-jquants target-sqlite`を実行しても、最後に取得したデータ以降を取得するようになっています。


## まとめ

Meltanoを使うと、J-Quants APIからのデータの取得に1行のコードを書くことなくデータを取得し、データベースへと格納し、ステート管理まで行うことができました。

しかも、データの格納先に関しては、データベースだけでなく、CSVやS3、BigQueryなど、様々なデータストアに対応しています。

データの変換(Transform)に関しては述べませんでしたので、ELTではなくELでしかなくタイトル詐欺ですが、Transormに関しては、また別の機会に紹介したいと思います。
