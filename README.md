# user
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

