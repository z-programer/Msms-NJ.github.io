---
layout: post
title:  "PostgreSQL安装手册"
date:   2020-11-14 11:22:31 +0800
lang: zh-cmn-Hans
categories: postgresql
---
在 Centos 7 上安装 PostgreSQL12 的手册。

1、安装PostgreSQL

```shell
# 安装Centos版本的源
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
# 安装PostgreSQL 12
sudo yum install -y postgresql12-server
# 安装扩展
sudo yum install postgresql12-contrib
# 启动服务
sudo /usr/pgsql-12/bin/postgresql-12-setup initdb
sudo systemctl enable postgresql-12
sudo systemctl start postgresql-12

```

2、配置PostgreSQL远程访问，编辑文件 ```/var/lib/pgsql/12/data/postgresql.conf```，修正以下内容，使数据库可以远程访问。
```conf
listen_addresses = '*'
```

3、编辑文件```/var/lib/pgsql/12/data/pg_hba.conf```，修正以下内容，配置相关账户的登录方式。
```conf
# 所有账户可以在本地使用密码登录
host    all             all             127.0.0.1/32            md5
# db_user 可以远程使用密码登录
host    all             db_user         0.0.0.0/0               md5
```

4、启动数据库UUID扩展的命令
```sql
CREATE EXTENSION pgcrypto;
CREATE EXTENSION "uuid-ossp";
```

---参考资料---
* [https://yum.postgresql.org](https://yum.postgresql.org)
* [https://www.postgresql.org/download/linux/redhat/](https://www.postgresql.org/download/linux/redhat/)
