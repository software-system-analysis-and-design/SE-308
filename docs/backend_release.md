# 后端安装部署说明

> 主要负责人：邱晓裕
>
> 审核人：邱奕浩

## 1. 部署环境说明

安装环境：Centos 7.2 64 位

数据库：PostgreSQL 10.0

后端代码：Golang1.12.6

## 2. 数据库安装

本项目后台使用PostgreSQL保存数据，因此，在部署项目之前，需要先安装PostgreSQL。

### （1）Centos 安装PostgreSQL

安装官网：https://www.postgresql.org/download/linux/redhat/

* 安装RPM

```bash
$ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

* 安装PostgreSQL10

```bash
$ yum install postgresql10
```

* 初始化数据库

```bash
$ /usr/pgsql-10/bin/postgresql10-setup initdb
```

* 设置开机自启动

```bash
$ systemctl enable postgresql-10.service
```

* 启动服务

```bash
$ systemctl start postgresql-10.service
```

### （2）PostgreSQL安装配置

* 修改用户密码：

```bash
$ su - postgres
$ psql -U postgres
$ ALTER USER postgres WITH PASSWORD '123456'
$ \q
```

* 开启远程访问

```bash
$ vi /var/lib/pgsql/9.5/data/postgresql.conf
修改#listen_addresses = 'localhost'  为  listen_addresses='*'
```

* 信任远程连接

```bash
$ vi /var/lib/pgsql/9.5/data/pg_hba.conf

# 修改如下内容，信任指定服务器连接
# IPv4 local connections:
host    all            all      127.0.0.1/32      ident
host    all            all      192.168.137.1/32（需要连接的服务器IP）  trust
```

* 重启服务

```bash
$ systemctl restart postgresql-9.5.service
```

### （3）PostgreSQL 

* 创建数据库：

```bash
postgres=# create database zhengqianbaodb;
```

* 创建表：

```sql
create table if not exists t_user
(
    phone text PRIMARY KEY NOT NULL,
    remain integer NOT NULL,
    iscow boolean NOT NULL,
    name text NOT NULL,
    password text NOT NULL,
    gender text,
    age integer,
    university text,
    company text,
    description text,
    class text
);

create table if not exists t_qformat
(
	taskID text PRIMARY KEY NOT NULL,
	taskName text NOT NULL,
	InTrash int NOT NULL,
	taskType text NOT NULL,
	creator text NOT NULL,
	description text NOT NULL,
	money integer NOT NULL,
	number integer NOT NULL,
	finishedNumber integer NOT NULL,
	publishTime text NOT NULL,
	endTime text NOT NULL,
	chooseData text
);

create table if not exists t_record
(
    taskID text NOT NULL,
    userID text NOT NULL,
    data text NOT NULL,
    CONSTRAINT pk_t_record_taskID_userID PRIMARY KEY (taskID, userID)
);

create table if not exists t_msg
(
	msgID text NOT NULL,
	state int NOT NULL,
	receiver text NOT NULL,
	time text NOT NULL,
	title text NOT NULL,
	content text NOT NULL
);

```

## 2. docker安装

### （1）下载安装

根据自己的系统在官网下载docker：

https://www.docker.com/products/docker-desktop

下载之后安装并运行docker，这个过程可能需要重启电脑

### （2）配置镜像加速器

在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
  ]
}
```

之后重新启动服务。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### （3）拉取并运行镜像

```bash
$ docker pull qiuxy23/zhengqianbaogo:version1.0
```

```bash
$ docker run -d -p 443:443 qiuxy23/zhengqianbaogo:version1.0
```