---
layout: post
title: "DVWA 安装"
date: 2020-06-14 16:01:00 +0800
description: DVWA 安装中遇到的坑 # Add post description (optional)
img:  dvwa.png # Add image post (optional)
tags: [Technical] # add tag
---


## 契机

在一个课程上看到了DVWA的使用，想要好好尝试一下，安装过程中遇到了下面的一些问题，记录下来，一边以后忘记。

## 安装XAMPP

DVWA的安装包中附着一个README文件
打开一看，我去还真长...
挑重点一看，官方推荐是哟功能XAMPP来作为网站的服务器。
遂下载
该服务器发布的版本中有Linux的deb安装包，下载之后，直接dkpg -i到Kali Linux上。
呐，Kali Linux呢，可能会报以些依赖问题，就按照报错中的解决依赖语句，直接贴在上一次运行的结果后面回车运行，应该就没问题了。

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

To set up the database, simply click on the Setup DVWA button in the main menu, then click on the `Create / Reset Database` button. This will create / reset the database for you with some data in.

If you receive an error while trying to create your database, make sure your database credentials are correct within `./config/config.inc.php`. *This differs from `config.inc.php.dist`, which is an example file.*

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

#登录页面

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

