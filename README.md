# user
- postgres : super user
- dbadmin : usual administrator
- gisuser : developer
- dbviewer : read-only user

## create
```
$ sudo su - postgres
$ psql
=# CREATE ROLE dbadmin WITH LOGIN CREATEDB CREATEROLE;
=# \password dbadmin
=# exit
```

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

一般ユーザーのアカウントから、DBの管理用ユーザーでログイン -> OK
```
$ psql -h localhost -U dbadmin -d postgres
```
