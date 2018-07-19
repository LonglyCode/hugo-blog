
---
title: Linux使用的点点滴滴
date: 2018-07-17 11:48:14
lastmod: 2018-07-19 17:06:11
draft: false
tags: ["linux", "tutorial"]

---

> 完全Linux下工作已经接近一年半的时间，中间零零碎碎记录一些好的东西和遇到的坑。有些只是为了记录而记录，网上都提到就不提了。持续更新...

<!--more-->

## 基础
1. 查看具体某个进程TCP端口占用情况， `lsof -p <pid> -i "TCP"`
2. `tail -f file.log` 尾部跟踪log文件（主要是-f参数--follow），可以实时输出最后的结果，对于实时请求可以看到信息。可以监控多个文件 `tail -f /path/*/file.log*`
3. source + sh 可以重启执行sh文件
4. `grep -A -B -C` 参数可以指定行数，找到结果之前、之后、上下几行的信息。
5. awk 可以用于 按分割符打印输入的字符串, $1 代表参数 1, $2代表参数2，默认按空格打印，如果是其他分隔符用参数 -F
6. scp 可用于将本地代码通过ssh拷贝到目标机器(sshcopy简称)
7. 增量更新两个相同的目录： `rsync -avz src dst`
8. 找到对应的文件进行删除： `find . -name '*.bill' | xargs rm -f`
9. `ls -li` 其中i用于显示列出项的inode，对于查看链接的文件或者目录有用
10. 设置代理其实就是修改https_proxy变量，`export https_proxy=socks5://127.0.0.1:1080`

## 有用
1. pip可以直接从github上安装，方式是 `pip install git+https://.....git`
2. zsh插件叫autojump，自动跳转目录，非常好用。
3. 使用`sshfs`可以用来将远程路径映射到本地，所有本地操作都会自动同步到远程去，当然是ssh安全的。命令如右边 `sshfs /remote/path/ /local/path/ -o follow_symlinks -o sshfs_sync -o reconnect`
4. **mycli** mysql 的命令行工具，也有对应的postgresql工具
5. ssh免密码登录需要最后设置：`ssh-copy-id <remoteIP>`
6. lsyncd 是有用的同步目录的工具，可以设置的同步延时时间： `lsyncd -delay 2 -rsync src dst`
7. mysqldump 同的mysql命令几乎一样，前者重定向输出，后者重定向输入： `mysqldump -u <user> -h <host> <dbname> -p  --skip-lock-tables > xxx.sql`
8. strace 用于监控某个进程的读写情况：`strace -p <pid> -e trace=open,write,read,close`
9. redshift 用于护眼：`redshift -c /etc/redshift.conf &`
10. vimdiff 对于的熟悉vim的人进行两个文件临时diff简直是神器
11. tcpdump
12. dig
13. 用sed命令批量替换某个路径下的字符， `grep 原字符串 -rl 所在目录` | xargs sed -i "s/原字符串/新字符串/g"`
14. 使用`httpie`替代curl或者postman，可以将request data 通过管道导入， `cat xxxx.json | sudo http -b POST :8080/xxxxx/xxxx\?token=kjl\&uid=uid2`


## 备忘
1. emacs直接安装需要安装依赖: `sudo apt-get install build-essential texinfo libx11-dev libxpm-dev libjpeg-dev libpng-dev libgif-dev libtiff-dev libgtk2.0-dev libncurses-dev`
2. svn 进行某一部分的目录单独更新：一，对父目录进行` svn checkout svn://svn.oa.com/SrcCodes/trunk/ --depth 'immediates'`，二，到对应的想更新的路径操作`svn up --set-depth 'infinity'`
3. 查看mysql数据库对应的用户和ip的权限: `select DISTINCT(`GRANTEE`) from `information_schema`.`USER_PRIVILEGES`;`
4. 设置用户mysql权限 `GRANT insert,update,select,delect PRIVILEGES ON <dbname>.* TO 'user@'192.168.%.%' identified by 'password';`
5. mysql 安装和使用：
    - apt-get install mysql-server
    - apt-get install mysql-client
    - apt-get install python-mysqldb
    接着使用`mysql -u root -p`进入客户端，然后输入密码，记得安装过程会提示你输入密码；`create database [databasename]`，注意mysql需要设置为中文，加后缀`create database [databasename] character set gbk` 
6. 在新的设备上把SSH密钥添加到个人github账号的方式：`ssh-keygen -t rsa -C your@email.com`
7. 设置autojump，一般从github上下载源码，然后在`.zshrc`文件下添加autojump插件，最重要的是要添加下面两句:
``` shell
[[ -s /home/xxxx/.autojump/etc/profile.d/autojump.sh ]] && source /home/xxxx/.autojump/etc/profile.d/autojump.sh
```
最后运行 `source .zshrc`。
