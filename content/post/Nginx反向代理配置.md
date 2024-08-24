+++
title = "Nginx反向代理配置"
date = 2020-08-19
tags = ["Nginx"]
categories = ["Nginx"]
+++

使用nginx做反向代理的时候，可以简单的直接把请求原封不动的转发给下一个服务。设置`proxy_pass`请求只会替换域名，如果要根据不同的url后缀来访问不同的服务，则需要通过如下方法：

### 方法一：加"/"

```
server {
    listen              8000;
    server_name         abc.com;
    access_log  "pipe:rollback /data/log/nginx/access.log interval=1d baknum=7 maxsize=1G"  main;

    location ^~/user/ {
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header  X-NginX-Proxy true;

        proxy_pass http://user/;
    }

    location ^~/order/ {
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header  X-NginX-Proxy true;

        proxy_pass http://order/;
    }
}
```
^~/user/表示匹配前缀是user的请求，proxy_pass的结尾有/， 则会把/user/*后面的路径直接拼接到后面，即移除user。

### 方法二：rewrite

```
location ^~/user/ {
        proxy_set_header Host $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;

        rewrite ^/user/(.*)$ /$1 break;
        proxy_pass http://user;
    }

    location ^~/order/ {
        proxy_set_header Host $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;

        rewrite ^/order/(.*)$ /$1 break;
        proxy_pass http://order;
    }
}
```
proxy_pass结尾没有/， rewrite重写了url。


### location匹配命令

```
 ~     #波浪线表示执行一个正则匹配，区分大小写
 ~*    #表示执行一个正则匹配，不区分大小写
 ^~    #^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
 =     #进行普通字符精确匹配
 @     #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
```

###  location 匹配的优先级(与location在配置文件中的顺序无关)

***=*** 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。

普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。

***^~*** 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。

最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。
