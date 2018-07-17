---

title: 初识postgresql以及在Flask里面的关联使用
date: 2016-04-19 00:50:21
tags: ["postgresql", "Flask", "python", "db"] 
author: "longlycode"

---
<!-- toc -->

> 起因发现同一个服务器上面有人安装了postgresql用来跑一个大型的应用，postgresql如雷贯耳，一直没机会接触，于是好奇折腾了起来。因为牵扯众多只挑重点来讲。

<!--more-->

## 1.安装
1. 在Ubuntu系统以及其延伸系统安装特别方便。

    ``` sql
    sudo apt-get postgresql
    sudo apt-get postgresql-client
    sudo apt-get pgadmin3
    ```

    分别为本体、客户端和一个管理postgresql的图形界面软件，非常好用，建议不熟悉命令行的新手使用。
2. 在window下直接下载exe文件安装。
   安装完成后将postgresql的**bin**目录加入到系统环境变量`Path`,譬如`;C:\Program Files (x86)\postgres\9.3\bin`加入到计算机->属性->高级环境变量->环境变量->系统变量->Path最末端。之后也可以在cmd下面操作。


## 2. 初始化--添加用户和数据库
初始化的时候让人很疑惑，postgresql的用户和系统用户不关联，它本身自己拥有一套用户管理系统，必须新建一个用户和一个和用户名同名的数据才能使用（有疑问?）。

    ``` sql
    # 所有的命令都可以加 --help来查看用法，下面用-s参数新建一个superuser账号 hehe。
    createuser -s hehe
    # 新建一个同名的数据库hehe并用参数-O指定owner（所有者）。
    createdb -O hehe hehe    # createdb -O username databasename
    ```

值得一提的是，它自带一个postgres用户和postgres数据库。

## 3. 登陆和使用
`psql` 是连接postgresql的命令控制台工具。常用的几个参数选项和MySQL很相似。

    ``` sql
    # 用上节新建的账号登陆
    psql -h 127.0.0.1 -p 5442 -U hehe -d hehe
    ```

最常用的几个参数`-h`指定服务器，`-p`指定端口，缺省默认的为5432，`-U` 指定登陆的用户，`-d`指定数据库，缺省默认和用户名相同的数据库。如果此用户需要密码接下来还需要输入口令。

## 4. 登陆之后
> 登陆之后就进入命令控制台，几乎所有操作都是在这个环境下进行的。

### 基本操作

    1. \h:查看sql命令的解释
    2. \?:所有psql命令列表
    3. \l:现有所有数据库列表
    4. \c [databasename]:直接跳转连接到其他数据库
    5. \d:列出当前数据库的表格
    6. \d [tablename]:列出表格的结构，非常有用的命令
    7. \du:列出所有用户
    8. \conninfo:当前连接信息
    9. \password:为当前用户指定一个登陆口令
    10. \q:退出当前控制台

### sql操作
* sql命令很多而且所有数据库大多都雷同。

    ``` sql
    # 新建表
    CREATE TABLE books(book_no integer,name text,price numberic,UNIQUE (book_no));
    # 插入数据
    INSERT INTO books(book_no,name,price) VALUES(2333,'万历十五年',18.5);
    # 查询
    SELECT * FROM books
    # 等等...
    ```

## 5. 启动、备份和恢复
### 启动中止状态操作
    1. Ubuntu: /etc/init.d/postgresql start (stop/status/restart)
    2. Windows: /PostgreSQL/9.3/bin/pg_ctl.exe (stop/status/restart/reload)
### 备份和恢复
    1. pg_dump [databasename]:将postgresql数据导出到一个脚本文件
    2. pg_dumpall:将所有的postgresql数据库导出到一个脚本文件
    3. pg_restore:从以上两个命令导出脚本文件恢复
    4. psql [databasename] < backup.sql恢复外部数据

## 6. 连接Flask的依赖
    1. Psycopg2: 一个适配连接postgres数据库的python包，windows下请选择 win-Psycopg不然会提示缺少DLL的error
    2. Flask-SQLAlchemy(2.1):Flask下的SQLAlchemy拓展，以更加python的形式来写数据库结构，更容易迁移数据库
    3. Flask-Migrate:Flask数据库的初始化、迁移脚本和版本管理的利器
    4. Flask-Script:命令行的拓展
    几乎使用Flask的用户都默认安装后三者，真正依赖是Psycopg2。

## 7. 最小使用
1. 在Flask项目的config.py文件里面最主要设置`SQLALCHEMY_DATABASE_URI`变量就可以了

    ``` python
    import os
    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
    'postgresql://username:password@localhost:5432/databasename
    DEBUG =True

    app=Flask(__name__)
    app.config.from_object(__name__)
    db=SQLAlchemy(app)

    ```
    比如用之前建好的数据库`hehe`就把最后那行改成 `postgresql://hehe:@localhost:5432/hehe`

2. 用SQLAlchemy来建一个名为User的测试model,保存为model.py文件

    ``` python
    from config import db

    class User(db.Model):
        __tablename__ = 'users'

        id = db.Column(db.Integer, primary_key=True)
        email = db.Column(db.String(64), unique=True, index=True)
        username = db.Column(db.String(64), unique=True, index=True)
        password_hash = db.Column(db.String(128))
        name = db.Column(db.String(64))

        def __repr__(self):
            return '<User %r>' % self.username
    ```

3. 建立一个名为manage.py文件，最小化使用Flask框架。

    ``` python
    from config import app,db
    from flask.ext.script import Manager
    from flask.ext.migrate import Migrate,MigrateCommand
    from model import User

    migrate=Migrate(app,db)
    manager=Manager(app)

    manager.add_command('db',MigrateCommand)

    @app.route('/')
    def hello():
        return 'Hello'

    @app.route('/<name>')
    def hello_user(name):
        return "hello {} !".format(name)

    if __name__=='__main__':
        manager.run()

    ```
4. 命令行下初始化数据库，先来到项目目录下，按顺序输入下面三个命令

    ``` python
    # 初始化
    python manage.py db init
    # 建立迁移脚本
    python manage.py db migrate -m "init databasename for postgres"
    # 更新
    python manage.py db upgrade
    ```
    可以看见当前目录下面生成了一个名为`migrations`文件夹，这个文件夹是保存迁移脚本和数据库版本控制的地方，下次如果添加了新的model只要使用上面最后两条命令即可。

5. 在psql下查看

    ``` sql
    $ psql -U hehe
    # 查看当前数据库的表
    hehe=# \d
    Schema   |      Name       |  Type | Owner
    ---------|-----------------|-------|--------
    public   | alembic_version | table | hehe
    public   | users           | table | hehe
    public   | users_id_seq    | ??????| hehe
    # 查看users表结构
    hehe=# \d users
                                        Table "public.users"
        Column    |         Type           |                 Modifiers
    --------------|------------------------|---------------------------------------------
    id            | integer                | not null nextval('users_id_seq'::regclass)
    email         | character varying(64)  |
    username      | character varying(64)  |
    password_hash | character varying(128) |
    name          | character varying(64)  |
    Indexes:
        "users_pkey" PRIMARY KEY, btree (id)
        "ix_users_email" UNIQUE, btree (email)
        "ix_users_username" UNIQUE, btree (username)
    ```
    说明成功了

## 8. 添加数据和显示
还是用命令行操作，需要在manage.py文件下面添加:

``` python
from flask.ext.script import Shell
def make_shell_context():
    return dict(app=app,db=db,User=User)
manager.add_command('shell',Shell(make_context=make_shell_context))
```
然后再命令行下面`python manage.py shell`进入带有app、db和User上下文环境的shell控制台。

``` sql
u = User(username="xibao")
# 添加一个名为xibao的用户
db.session.add(u)
db.session.commit()
# 退出当前控制台
exit
```
启动服务`python manage runserver`，在浏览器上面输入`127.0.0.1:5000\xibao`会返回"hello xibao !"。

## 9. 从postgresql里面备份

``` sql
# 使用之前pg_dump命令备份
pg_dump  -f backup.sql hehe
```
当前项目目录下面会多出一个`backup.sql`文件。
