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

確認
```
psql -U gisuser -h localhost -d postgres
```

# DB
## create
DB作成：各DBの管理情報を入れるDB
```
=> CREATE DATABASE gisfederation;
```

テスト用DB作成
```
/opt/util-linux-2.41/bin/uuidgen --time-v7
01a1d2e2-5476-74e6-982b-6158fd9d9006

=> CREATE DATABASE "01a1d2e2-5476-74e6-982b-6158fd9d9006";
```

DBのPostGis有効化
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
01a1d2e2-5476-74e6-982b-6158fd9d9006=>　CREATE TABLE "01a1d331-836d-7551-8f15-bda257c9273c" (tuple_id UUID PRIMARY KEY, item_key JSONB, item_val JSONB, gis GEOMETRY, gis_annotation JSONB);
```
