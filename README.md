# 説明
MLBのオープンデータ「Retorosheet」を使うためのコンテナです。
* Retorosheet http://www.retrosheet.org/ → MLBのオープンデータ
* Chadwick http://chadwick.sourceforge.net/doc/index.html → Retrosheetのデータをパースするライブラリ
* py-retrosheet https://github.com/wellsoliver/py-retrosheet → Chadwickのコマンドを利用して、指定した年のデータを自データベースに格納してくれるライブラリ

を利用しています。

# 使い方
※「docker compose」をインストールしておく必要があります。

1. Retrosheetデータを保存するvolume作成
`docker volume create retrosheet_sample`

2. イメージの取得
`docker pull mysql:latest`
`docker pull centos:latest`
`docker pull shigechiitech:retrosheet-use`

3. 必要ファイル作成
`vi (your_local_path)/docker-compose.yml`
> db:
> &nbsp;&nbsp;&nbsp;&nbsp;image: mysql:latest
> &nbsp;&nbsp;&nbsp;&nbsp;environment:
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_ROOT_PASSWORD: password
> &nbsp;&nbsp;&nbsp;&nbsp;volumes:
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- (your_local_path)/db:/docker-entrypoint-initdb.d
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- retrosheet_sample:/var/lib/mysql
> app:
> &nbsp;&nbsp;&nbsp;&nbsp;image: shigechiitech/retrosheet_use:1.0
> &nbsp;&nbsp;&nbsp;&nbsp;volumes:
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- (your_local_path)/app/config.ini:/retrosheet/py-retrosheet/scripts/config.ini
> &nbsp;&nbsp;&nbsp;&nbsp;links:
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- db:mysql
> &nbsp;&nbsp;&nbsp;&nbsp;privileged: true

    `vi (your_local_path)/app/config.ini`
> [database]
> engine = mysql+pymysql
> host = 172.17.0.2
> database = appdb
> schema = retrosheet
> user = appusr
> password = password
> 
> [download]
> directory = files
> 
> #This seems like a safe value for retrosheet.org
> num_threads = 20
> 
> #Currently, only eventfiles are processed by parse.py, but gamelogs can still be downloaded.
> dl_eventfiles = True
> dl_gamelogs = False
> 
> [chadwick]
> directory = /usr/local/bin/
> 
> #Don't change this unless you know what you're doing
> [retrosheet]
> eventfiles_url = http://www.retrosheet.org/game.htm
> gamelogs_url = http://www.retrosheet.org/gamelogs/index.html
> 
> [debug]
> verbose = True

    `vi (your_local_path)/db/createdb.sql`
> CREATE USER appusr IDENTIFIED BY 'password';
> CREATE DATABASE appdb CHARACTER SET utf8;
> GRANT ALL PRIVILEGES ON appdb.* TO appusr;
> GRANT ALL PRIVILEGES ON appdb_test.* TO appusr;
> FLUSH PRIVILEGES;

4. docker-composeを利用して、コンテナ起動
`/usr/local/bin/docker-compose up -d`

5. appコンテナへアクセス
`docker exec -it retrosheet_app_1 /bin/bash`

6. schema流す
`mysql -u appusr -p -h 172.17.0.2 appdb < /retrosheet/py-retrosheet/sql/schema.sql `
`Enter password: password`

7. 年指定してデータダウンロード
`cd /retrosheet/py-retrosheet/scripts/`
`python download.py -y 2016`

8. 年指定してデータをデータベースに格納
`python parse.py -y 2016`
