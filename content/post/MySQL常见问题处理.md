+++
title = "MySQL常见问题处理"
date = 2021-09-26
tags = ["MySQL"]
categories = ["MySQL"]
+++

* 关于MySQL出现```lock wait timeout exceeded; try restarting transaction``` 的解决方案。

```
select * from information_schema.innodb_trx;

kill $trx_mysql_thread_id;
```
