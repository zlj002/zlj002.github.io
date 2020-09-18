---
layout: post
title: 'docker-nginx-php'
subtitle: 'docker-nginx-php'
date: 2019-01-12
author: 'Lorin'
header-img: 'img/post-bg-unix-linux.jpg'
tags:
    - 前端开发
---

>  
1.安装php
- docker pull php:7.1-fpm

2.启动php
- docker run --name  aha-php-fpm -v ~/nginx/www:/www  --privileged=true -d php:7.1-fpm
命令说明：

--name aha-php-fpm : 将容器命名为 aha-php-fpm。

-v ~/nginx/www:/www : 将主机中项目的目录 www 挂载到容器的 /www
3.在该目录下添加 ~/nginx/conf/conf.d/test-php.conf 文件，内容如下：
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm index.php;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /www/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
配置文件说明：

php:9000: 表示 php-fpm 服务的 URL，下面我们会具体说明。
/www/: 是 myphp-fpm 中 php 文件的存储路径，映射到本地的 ~/nginx/www 目录。
启动 nginx：

docker run --name aha-php-nginx -p 80:80 -d \
    -v ~/nginx/www:/usr/share/nginx/html:ro \
    -v ~/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
    --link aha-php-fpm:php \
    nginx
配置说明：

-p 80:80: 端口映射，把 nginx 中的 80 映射到本地的 80 端口。
~/nginx/www: 是本地 html 文件的存储目录，/usr/share/nginx/html 是容器内 html 文件的存储目录。
~/nginx/conf/conf.d: 是本地 nginx 配置文件的存储目录，/etc/nginx/conf.d 是容器内 nginx 配置文件的存储目录。
--link myphp-fpm:php: 把 myphp-fpm 的网络并入 nginx，并通过修改 nginx 的 /etc/hosts，把域名 php 映射成 127.0.0.1，让 nginx 通过 php:9000 访问 php-fpm。


接下来我们在 ~/nginx/www 目录下创建 index.php，代码如下
<?php
echo phpinfo();
?>


创建测试容器结束



开始创建项目容器
将上面的容器生成镜像
docker run --name  aha-php-fpm -v $PWD:/www  --privileged=true -d php:7.1-fpm

docker run --name aha-php-nginx -p 80:80 -d \
    --privileged=true \
    -v ~/workspace/hjm_wx/src:/usr/share/nginx/html/aha-wx:ro \
    -v ~/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
    --link aha-php-fpm:php \
    nginx


开发使用：
多个项目统一的目录  ~/workspace
创建 aha-php-fpm
docker run --name  aha-php-fpm -v ~/workspace:/www  --privileged=true -d php:7.1-fpm
创建 aha-php-nginx
    docker run --name aha-php-nginx -p 80:80 -d \
    --privileged=true \
    -v ~/workspace:/www:ro \
    -v ~/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
    --link aha-php-fpm:php \
    nginx
新增站点配置
server {
        listen 80;
        server_name localhost;
        root /www/项目名称/发布后的目录;
        index index.php index.html;
        #access_log /usr/local/var/log/nginx/access.log;
        #error_log /usr/local/var/log/nginx/error.log;

        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';

        location / {
                try_files $uri $uri/ /index.php?$query_string;
                #if (!-e $request_filename) {
                #       rewrite ^/(.*)$ /index.php?/$1 last;
                #        break;
                #}
        }
        location /ip {
                add_header Content-Type html/text;
                return 200 $request_uri;
        }

        location ~ '\.php$' {
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass php:9000;
                fastcgi_index index.php;
                fastcgi_read_timeout 60;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}


=============

创建apache
docker pull php:7.0-apache

docker run  -it \
--name  aha-php-apache \
--privileged=true \
-u root \
-d \
-v $PWD:/var/www/html \
-p 8001:80  \
php:7.0-apache

docker run  -it \
--name  aha-php-apache \
--privileged=true \
-u root \
-d \
-v $PWD/src:/var/www/html \
sh -c echo "ServerName localhost" >> /etc/apache2/apache2.conf \
-p 80:80  \
php:7.0-apache




docker run  -it \
--privileged=true \
-u root \
-d \
-v $PWD:src:/var/www/html \
echo "ServerName localhost" >> /etc/httpd/conf/httpd.conf \
-p 8100:80  \
php:7.0-apache


//文件
FROM php:7.0-apache
COPY . /var/www/html/
EXPOSE 80
CMD echo "ServerName localhost" >> /etc/apache2/apache2.conf
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]


> 结束语
> 某些看起来想魔法一样的盒子，当你打开他的时候，会有很多意外收获。再补一句汤，如果没有感觉今天的自己比昨天有进步那么就是在倒退
