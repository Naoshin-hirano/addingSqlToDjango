<h1>djangoチュートリアル</h1>
データベースとの接続

今回はメジャーな3つのデータベース管理システムであるSqlite3、MySQL、Postgresとの接続方法を説明します。

<h2>完成版プロジェクト</h2>
https://github.com/shun-rec/django-website-05

<h2>Youtube</h2>
https://www.youtube.com/watch?v=k3feQcc6bFw&t=64s
  
<h2> 目次</h2>
事前準備
Sqlite3と接続しよう
MySQLと接続しよう
Posgresと接続しよう
  
  
<h2> 事前準備</h2>
Paiza Cloudで以下３つを選択して新規サーバーを作成して下さい 。

django
MySQL
Postgres
  

<h2>Sqlite3と接続しよう</h2>
<h3>接続設定</h3>
実は設定不要！
デフォルトでSqlite3と接続するようになっています。
メインのデータベースとしては力不足。
本番で重要なデータの保存先として使われることはまずありません。
社内限定や内輪限定など使用ユーザーが少ない、かつデータが重要ではないサービスならOK。

  <h2>接続確認</h2>
プロジェクトのデータベース内にテーブル作成
15行ほどログが出力されて成功すればOKです。

```
python manage.py migrate
```
  
試しにスーパーユーザー（管理者）を作成してみよう
ユーザー名、メール、パスワードを聞かれるので答えると作成されます。
```
python manage.py createsuperuser
```
/admin の管理画面URLから今のユーザーでログイン出来るようになります。
管理画面の詳細はまた後ほど。

  <h2>MySQLと接続しよう</h2>
  <h3>共通ライブラリのインストール</h3>
MySQLとPostgres共通で必要になるライブラリを２つインストールします。

dj-database-url: データベースの接続設定が１行で楽に書けるライブラリ
python-dotenv: .envという環境設定ファイルを使ってプロジェクトの設定が出来るライブラリ
  ```
pip install dj-database-url
pip install python-dotenv
  ```
  <h2>全体設定ファイルの編集</h2>
もともとあるDATABASESの欄を削除して以下を追記。

```
import dj_database_url
from dotenv import (
    find_dotenv,
    load_dotenv,
)
load_dotenv(find_dotenv())
DATABASES = {
    'default': dj_database_url.config(conn_max_age=600),
}
```
  
load_dotenvで環境設定ファイルを読み込み、
dj_database_url.configで自動的に設定されます。
conn_max_age=600というのは今は理解不要ですが、高速化の設定です。
  
  <h2>MySQL用ライブラリのインストール</h2>
  
```
pip install mysqlclient
```
  
<h3>MySQL上にプロジェクト用データベースを作成</h3>
MySQLにログイン
rootというユーザー名でログインするという意味。

```
mysql -u root
```
  
  <h3>djangoからMySQLに接続するユーザーのパスワードを設定</h3>
rootというユーザーのパスワードをpasswordに変更するという意味。

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```
  
<h3>プロジェクト用のデータベース作成</h3>
pj_dbという新しいデータベースを作成するという意味。

  ```
create database pj_db;
  ```
  
  <h2>環境設定ファイルを設置</h2>
プロジェクト直下に```.env```というファイル名でファイルを作り、以下の内容を入力。

※ドットから始まるファイル名は隠しファイルと言って、デフォルトでは非表示です。Paizaではプロジェクトフォルダで右クリックをすると、「隠しファイルを表示」することが出来ます。

mysqlのpj_dbというデータベースにrootユーザーでpasswordパスワードでアクセスするという意味。

  ```
DATABASE_URL=mysql://root:password@localhost/pj_db
  ```
  
  <h2>接続確認</h2>
Sqlite3の場合と同様のため省略します。

  <h2>Postgresと接続しよう</h2>
共通ライブラリのインストール
MySQLと同じため省略します。

<h3>Postgres用ライブラリのインストール</h3>
  
  ```
pip install psycopg2-binary
  ```
  
<h3>プロジェクト用のデータベースを作成</h3>
Postgresにログイン
  
  ```
sudo -u postgres psql postgres
  ```
  
  <h3>ユーザーにパスワード設定</h3>
postgresというユーザーにパスワードを設定するという意味。

  ```
\password postgres
  ```
  
  <h3>データベース作成</h3>
  
  ```
CREATE DATABASE pj_db;
  ```
  
<h3>Postgresからログアウト</h3>
\q
<h3>環境設定ファイルを設置</h3>
  
```
DATABASE_URL=postgres://postgres:password@localhost/pj_db
```
  <h3>接続確認</h3>
Sqlite3の場合と同様のため省略します。

<h2>Herokuの本番サーバーで接続できるか確認しよう</h2>
特別な手順はありません。
前回と同様に、

1.静的ファイルの設定
2.requirements.txtを作成
3.Procfileを作成
してHerokuサーバーにアップロードするだけです。
  
  <h3>requirements.txt</h3>
  
```
django
gunicorn
whitenoise
dj-database-url
python-dotenv
psycopg2-binary
mysqlclient
```
  <h3>Procfile</h3>
  
```
web: gunicorn pj_db.wsgi
```
  <h3>静的ファイルの設定</h3>
whitenoiseの設定
全体設定ファイルの編集

MIDDLEWAREの最後にwhitenoiseを追加
  
```
MIDDLEWARE = [
    ...
    'whitenoise.middleware.WhiteNoiseMiddleware',
]
```
最後に以下を追記

```
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```
〜詳細説明〜

手元のPostgresには当然Herokuからはアクセス出来ません。
実はHerokuにはデフォルトで無料のPostgresがインストールされています。
環境設定ファイルはSettingsタブのConfig Varsから自動で作成されます。
