---
layout: post
title: 利用Docker在ubuntu上安装Nginx
date: 2018-07-14 17:50:24.000000000 +08:00
---

对于许多常用的开源工具，docker hub里已经有做好的镜像，可以直接拿来用，对于新手尤其方便有效。本文以在ubuntu上采用docker安装nginx为例进行说明。

**1.在主目录下创建nginx目录及3个子目录：**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ mkdir -p ./www ./conf ./log
ryan@ryan-VirtualBox:~/nginx$ ls
conf  log  www
'''
{% endhighlight %}

其中，www目录将映射为nginx容器配置的虚拟目录，conf目录里的配置文件将映射为nginx容器的配置文件，log目录将映射为nginx容器的日志目录。

**2.在conf目录下添加nginx.conf文件，填入如下内容：**

{% highlight ruby %}
'''
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /opt/nginx/log/access.log;
    error_log /opt/nginx/log/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;
    gzip_disable "msie6";

    ##
    # Virtual Host Configs
    ##

    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /opt/nginx/www;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
        }

    }
}
'''
{% endhighlight %}

**3.在www目录下添加index.html文件，填入以下内容：**

{% highlight ruby %}
'''
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body>
<h2>Hello docker nginx</h1>
</body>
</html>
'''
{% endhighlight %}

**4.在log目录下创建两个空日志文件access.log和error.log。这时的nginx目录应该是这样：**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ tree
.
├── conf
│   └── nginx.conf
├── log
│   ├── access.log
│   └── error.log
└── www
    └── index.html
'''
{% endhighlight %}

**5.查找Docker Hub上的nginx镜像：**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ docker search nginx
NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                                                  Official build of Nginx.                        9017                [OK]                
jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker con…   1359                                    [OK]
richarvey/nginx-php-fpm                                Container running Nginx + PHP-FPM capable of…   588                                     [OK]
jrcs/letsencrypt-nginx-proxy-companion                 LetsEncrypt container to use with nginx as p…   389                                     [OK]
kong                                                   Open-source Microservice & API Management la…   204                 [OK]                
webdevops/php-nginx                                    Nginx with PHP-FPM                              106                                     [OK]
kitematic/hello-world-nginx                            A light-weight nginx container that demonstr…   102                                     
zabbix/zabbix-web-nginx-mysql                          Zabbix frontend based on Nginx web-server wi…   58                                      [OK]
bitnami/nginx                                          Bitnami nginx Docker Image                      54                                      [OK]
linuxserver/nginx                                      An Nginx container, brought to you by LinuxS…   37                                      
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5   ubuntu-16-nginx-php-phpmyadmin-mysql-5          36                                      [OK]
tobi312/rpi-nginx                                      NGINX on Raspberry Pi / armhf                   20                                      [OK]
nginxdemos/nginx-ingress                               NGINX Ingress Controller for Kubernetes . Th…   11                                      
wodby/drupal-nginx                                     Nginx for Drupal container image                9                                       [OK]
blacklabelops/nginx                                    Dockerized Nginx Reverse Proxy Server.          9                                       [OK]
webdevops/nginx                                        Nginx container                                 8                                       [OK]
nginxdemos/hello                                       NGINX webserver that serves a simple page co…   7                                       [OK]
centos/nginx-18-centos7                                Platform for running nginx 1.8 or building n…   6                                       
1science/nginx                                         Nginx Docker images that include Consul Temp…   4                                       [OK]
centos/nginx-112-centos7                               Platform for running nginx 1.12 or building …   3                                       
pebbletech/nginx-proxy                                 nginx-proxy sets up a container running ngin…   2                                       [OK]
toccoag/openshift-nginx                                Nginx reverse proxy for Nice running on same…   1                                       [OK]
travix/nginx                                           NGinx reverse proxy                             1                                       [OK]
ansibleplaybookbundle/nginx-apb                        An APB to deploy NGINX                          0                                       [OK]
mailu/nginx                                            Mailu nginx frontend                            0                                       [OK]
'''
{% endhighlight %}

**6.直接下载官方镜像：**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ docker pull nginx
'''
{% endhighlight %}

下载完成后，可以通过命令查看名为nginx的镜像：

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              3c5a05123222        7 days ago          109MB
ubuntu              latest              113a43faa138        5 weeks ago         81.2MB
hello-world         latest              e38bc07ac18e        3 months ago        1.85kB
'''
{% endhighlight %}

**7.运行容器**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ docker run -p 8093:80 --name mnginx  -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/www:/opt/nginx/www -v $PWD/log:/opt/nginx/log  -d nginx
aaed799efdadecb81d21079dc05b8b8a03d8bf8c38ab532e4886a872f6692e94
'''
{% endhighlight %}

命令说明：

- -p 8093:80  将容器的80端口映射到主机的8093端口
- --name mnginx  将容器命名为mnginx
- -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf  将主机目录下的nginx.conf挂载到容器中的/etc/nginx/nginx.conf
- -v $PWD/www:/opt/nginx/www  将主机目录下的www目录挂载到容器的/opt/nginx/www
- -v $PWD/log:/opt/nginx/log  将主机目录下的log目录挂载到容器的/opt/nginx/log

**注意，这里选择的挂载到的容器中的目录，要与nginx.conf中指定的access.log、error.log和root目录对应，否则会出现运行容器后，容器找不到相应目录的情况，导致运行失败**

**8.查看容器运行情况**

{% highlight ruby %}
'''
ryan@ryan-VirtualBox:~/nginx$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
aaed799efdad        nginx               "nginx -g 'daemon of…"   4 seconds ago       Up 3 seconds        0.0.0.0:8093->80/tcp   mnginx
'''
{% endhighlight %}

9.通过浏览器访问nginx

![Alt text](https://github.com/GitCatRyan/gitcatryan.github.io/raw/master/assets/images/nginx.png)
