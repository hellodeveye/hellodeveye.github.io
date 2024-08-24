+++
title = "Docker安装软件集合"
date = 2021-03-16
tags = ["docker"]
categories = ["docker"]
+++


### Docker安装Nginx

```
docker run --name nginx -p 8000:80 -d --privileged=true \ 
-v $PWD/log:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
-v $PWD/conf/conf.d:/etc/nginx/conf.d \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf --restart=always nginx
```

### Docker安装FTP
1. docker search vsftpd #寻找vsftpd的镜像
#假如我们找到一个最多引用的，叫fauria/vsftpd
2. docker pull fauria/vsftpd #把镜像pull到本地

```
docker run -d -p 21:21 -p 20:20 -p 21100-21110:21100-21110 --name vsftpd \
-v /opt/ftp_file:/home/vsftpd \
-e FTP_USER=username \
-e FTP_PASS=password \
-e PASV_ADDRESS=xxx.xxx.xxx.xxx \
-e PASV_MIN_PORT=21100 \ 
-e PASV_MAX_PORT=21110  --restart=always fauria/vsftpd
``` 
 
`-p`进行端口绑定映射
`-v`进行文件目录的映射 FTP_UESR 和FTP_PASS如果设定了会在container里面的 
/etc/vsftpd/virtual_users.txt
`PASV_MIN_PORT`和`PASV_MAX_PORT`映射的是被动模式下端口使用范围 
`PASV_ADDRESS`指的的宿主机地址


1、我们先进入container里面
 
```
docker exec -i -t vsftpd bash 
```
 
2、修改并生成虚拟用户模式下的用户db文件
 
```
vi /etc/vsftpd/virtual_users.txt #编辑配置文件写入用户跟密码
```
 
假如我们添加了user用户
 
```
mkdir /home/vsftpd/user #建立新用户文件夹 
/usr/bin/db_load -T -t hash
 -f /etc/vsftpd/virtual_users.txt /etc/vsftpd/virtual_users.db
```

### Docker 配置阿里镜像 

```
sudo mkdir -p /etc/docker 
sudo tee /etc/docker/daemon.json <<-'EOF' { "registry-mirrors": ["https://2g2ux2e3.mirror.aliyuncs.com"] } EOF 
sudo systemctl daemon-reload 
sudo systemctl restart docker
```
