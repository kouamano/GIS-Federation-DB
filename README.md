# DB system
```
$ dnf list installed | grep postgresql
postgresql18.aarch64                                 18.6-1.0.1.el10_2                    @ol10_appstream    
postgresql18-contrib.aarch64                         18.6-1.0.1.el10_2                    @ol10_appstream    
postgresql18-postgis.aarch64                         3.5.3-1.el10_2.1                     @ol10_appstream    
postgresql18-postgis-client.aarch64                  3.5.3-1.el10_2.1                     @ol10_appstream    
postgresql18-private-libs.aarch64                    18.6-1.0.1.el10_2                    @ol10_appstream    
postgresql18-server.aarch64                          18.6-1.0.1.el10_2                    @ol10_appstream  
```

# User
- postgres : super user
- gisuser : developer
- dbviewer : read-only user

## create
ここで、データベースユーザーのパスワードログインを可能にする。

ファイル: /var/lib/pgsql/data/pg_hba.conf を以下の様に書き換え
```
# IPv4 local connections:
#host    all             all             127.0.0.1/32            ident
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
#host    all             all             ::1/128                 ident
host    all             all             ::1/128                 scram-sha-256
```

ユーザー作成
```
CREATE USER gisuser WITH PASSWORD '<password>' LOGIN CREATEDB;
```
```
CREATE ROLE dbviewer WITH LOGIN PASSWORD <osjimの例のパスワード>;
```

確認
```
psql -U gisuser -h localhost -d postgres
```
```
psql -U dbviewer -h localhost -d postgres
```

# DB
## create
全体管理用DB作成（全体でひとつだけ、DB名/テーブル名がUUIDになるので補助情報として）
```
=> CREATE DATABASE gisfederation;
```

テスト用DB作成
```
/opt/util-linux-2.41/bin/uuidgen --time-v7
01a1d2e2-5476-74e6-982b-6158fd9d9006

=> CREATE DATABASE "01a1d2e2-5476-74e6-982b-6158fd9d9006";
```

DBのPostGis有効化 ← これはsuper userで
```
01a1d2e2-5476-74e6-982b-6158fd9d9006=# CREATE EXTENSION IF NOT EXISTS postgis;
```

# Table
## create
GISテーブル作成
```
/opt/util-linux-2.41/bin/uuidgen --time-v7
01a1d331-836d-7551-8f15-bda257c9273c #テーブル名

psql -U gisuser -h localhost -d 01a1d2e2-5476-74e6-982b-6158fd9d9006
01a1d2e2-5476-74e6-982b-6158fd9d9006=>　CREATE TABLE "01a1d331-836d-7551-8f15-bda257c9273c" (tuple_id UUID, item_key JSONB, item_val JSONB, gis GEOMETRY, gis_annotation JSONB);
```

# Test
1) GISのインサート

```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> INSERT INTO "01a1d331-836d-7551-8f15-bda257c9273c" (tuple_id,gis) VALUES ('019a0000-0000-7000-8000-000000000001',ST_GeomFromText('POINT(139.7 35.6)'));
INSERT 0 1
```

OK↓ ただし、バイナリ形式で表示される。
```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> select * from "01a1d331-836d-7551-8f15-bda257c9273c";
```

座標変換して表示
```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> select tuple_id,ST_AsText(gis) AS coordinates from "01a1d331-836d-7551-8f15-bda257c9273c" where ST_DWithin(gis,ST_GeomFromText('POINT(139 35)'),1);
               tuple_id               |    coordinates
--------------------------------------+-------------------
 019a0000-0000-7000-8000-000000000001 | POINT(139.7 35.6)
(1 行)
```

2) JSONのアップデート
```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> UPDATE "01a1d331-836d-7551-8f15-bda257c9273c" SET item_key='{"testkey1":"testke1-1"}' WHERE tuple_id='019a0000-0000-7000-8000-000000000001';
UPDATE 1
```

間違えてみる
```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> UPDATE "01a1d331-836d-7551-8f15-bda257c9273c" SET item_val='{"testkey1","testke1-1"}' WHERE tuple_id='019a0000-0000-7000-8000-000000000001';
ERROR:  json型に対する不正な入力構文
```
ちゃんとエラーになる。

3) copy

まず、postgres ユーザーで、当該DBユーザーのファイルreadをgrantする：
```
postgres=# GRANT pg_read_server_files TO gisuser;
```

DBへのログインと'\copy'コマンドを、ローカルユーザーから実行する
```
$ psql -h localhost -U gisuser -d 01a1d2e2-5476-74e6-982b-6158fd9d9006  -c "\copy \"public\".\"01a1d331-836d-7551-8f15-bda257c9273c\" FROM '/var/tmp/GIS-Federation-DB/postgres_copy_sample_10rows.csv' WITH (FORMAT CSV, HEADER TRUE, ENCODING 'UTF8')"
ユーザー gisuser のパスワード:
COPY 10
```
実行後はログアウト状態になる。

4) 検索

```
01a1d2e2-5476-74e6-982b-6158fd9d9006=> select * from "01a1d331-836d-7551-8f15-bda257c9273c" WHERE (item_val ->> 'ph')::numeric >= 7.0;
```
