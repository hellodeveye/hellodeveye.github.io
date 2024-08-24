+++
title = "MySQL常用命令"
date = 2021-03-16
tags = ["MySQL"]
categories = ["MySQL"]
+++


### 创建表空间
```
./mysql -uroot -p -hlocalhost -P3306

mysql> CREATE DATABASE cicro DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> CREATE USER 'cicrouser'@'%' IDENTIFIED BY 'cicropassword';
mysql> GRANT ALL PRIVILEGES ON cicro.* TO 'cicrouser'@'%';
mysql> flush privileges;

```
