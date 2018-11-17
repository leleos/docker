配置php7.2.4 + nginx 1.15.6 环境
====
镜像nginx:1.15.6-alpine + php:7.2.4-fpm-alpine

## 1.下载镜像
### ① 运行docker，如果已经运行请略过，以下均在root权限下操作
```shell
systemctl start docker
```
### ② 镜像加速器
配置阿里云加速，这一步可以略过
请注意https://????.mirror.aliyuncs.com 这个换成自己的，地址获取方式在阿里云的控制台->容器镜像服务->镜像加速器
```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://????.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
```
### ③ 拉取镜像
```shell
docker pull nginx:1.15.6-alpine

docker pull php:7.2.4-fpm-alpine
```
### ④ 查看镜像
```shell
docker images
```
应该列出刚刚拉取得镜像

## 2.基础配置

### ① 建立目录
```shell
cd /
mkdir -p docker/web0/{www,conf}
```
在根目录上建立docker目录，在docker目录下建立目录web0，在web0中建立目录www和conf

### ② 设置nginx.conf
运行nginx
```shell
docker run --name nginx --rm -d  nginx:1.15.6-alpine
```
把nginx的配置文件复制到本地
```shell
docker cp nginx:/etc/nginx/nginx.conf /docker/web0/conf 
```
修改nginx.conf为如下：
```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

server{
      listen 80;
      location / {
      root /usr/share/nginx/html;
        index  index.html index.htm index.php;
      }
      location ~ \.php$ {
        root /var/www;
        fastcgi_pass 192.100.0.3:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
      }
}
    # include /etc/nginx/conf.d/*.conf;
}
```
其实只是添加了server{}节点的内容

### ③ 新建测试文件
```shell
vim /docker/web0/www/index.html 
```
写入以下代码
```shell
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>docker html</title>
</head>
<body>
	<div>this is ok</div>
</body>
</html>
```

```shell
vim /docker/web0/www/index.php 
```
写入以下代码
```PHP
<?php
   phpinfo();
?>
```


## 运行docker
### ① 添加docker网络
```shell
docker network create --driver=bridge --subnet=192.100.0.0/16 myweb
```
以上建立了一个192.168.100.0 的网络，用于nginx和php的通信

### ② 运行nginx 
进入到web0目录
```shell
cd /docker/web0
```
启动nginx
```shell
docker run --name nginx -d --rm -v $(pwd)/www:/usr/share/nginx/html \
-v $(pwd)/conf/nginx.conf:/etc/nginx/nginx.conf \
--network myweb -p 80:80 nginx:1.15.6-alpine 
```
参数说明：
--name 指定名称，这里指定名称为nginx
-d 后台运行
--rm 停止的时候删除容器
-v 存储映射，这里：映射了www目录，映射了配置目录
--network 指定容器网络，这里指定为myweb(上一步建立的)
-p 端口映射，这里将容器的80映射到本地80

查看状态
```shell
docker ps
```
显示如下信息
```shell
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                NAMES
b4961df0f0c8        nginx:1.15.6-alpine    "nginx -g 'daemon of…"   5 seconds ago       Up 2 hours          0.0.0.0:80->80/tcp   nginx
```
表示运行了，测试下是否能访问index.html

```shell
curl 127.0.0.1
```
如果显示出index的代码表示OK

### ③ 运行php-fpm
```shell
cd /docker/web0

docker run --name fpm -d --rm -v $(pwd)/www:/var/www \
--network myweb --ip 192.100.0.3 php:7.2.4-fpm-alpine
```
参数说明：
--ip 指定容器的IP地址

查看状态
```shell
docker ps
```
显示如下信息
```shell
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                NAMES
1c71b188db4e        php:7.2.4-fpm-alpine   "docker-php-entrypoi…"   5 seconds ago       Up 4 seconds        9000/tcp             fpm
b4961df0f0c8        nginx:1.15.6-alpine    "nginx -g 'daemon of…"   41 seconds ago      Up 40 seconds       0.0.0.0:80->80/tcp   nginx
```
这里可以看到有两个容器在运行

使用curl查看，如果显示则表示正常
