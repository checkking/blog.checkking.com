<!--
author: checkking
date: 2016-12-17
title: Markdown 博客搭建
tags: GitBlog,markdown
category:markdown
status: publish
summary: 搭建属于自己的markdown博客
-->
## nginx+php环境安装
### php 安装
``` bash
wget http://cn2.php.net/distributions/php-5.6.2.tar.gz
tar -zxvf php-5.6.2.tar.gz
cd php-5.6.2
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=php-fpm --with-fpm-group=php-fpm --with-mysql=mysqlnd --with-mysql-sock=/tmp/mysql.sock --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-zlib-dir --with-mcrypt --enable-soap --enable-gd-native-ttf --enable-ftp --enable-mbstring --enable-exif --disable-ipv6 --with-pear --with-curl --with-openssl
```
如果报错缺少包，则apt-get install 对应的包，已经发现需要安装的有：
``` bash
tar zxvf libxml2-2.6.20.tar.gz
cd libxml2-2.6.20
./configure
make
make install
tar zxvf freetype-2.4.0.tar.bz2
cd freetype-2.4.0
./configure
make
make install
apt-get install 
apt-get install libjpeg-dev
apt-get install libpng-dev
apt-get install libmcrypt-dev
```
安装好这些依赖包之后，到php目录下
``` bash
make
make install
```
> **注意：**编译php的时候make不要多线程，不然可能内存不够用，如果机器内存小的话。

安装好之后就可以调用php了，如果在直接php不行，建立一个软连接
```bash
ln -s /usr/local/bin/php /usr/local/php/bin/php
```
### nginx安装
```bash
wget http://nginx.org/download/nginx-1.6.2.tar.gz
tar -xvf nginx-1.6.2.tar.gz
cd nginx-1.6.2
cd nginx-1.6.2
./configure \
--prefix=/usr/local/nginx \
--with-http_realip_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_stub_status_module  \
--with-pcre
```
这一步可能会报错，报缺少pcre，安装：
```bash
 wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
 tar -xvf pcre-8.38.tar.gz
 cd pcre-8.38
 ./configure
 make
 make install
```
建立软连接：
```bash
ln -sf /usr/local/nginx/sbin/nginx  /usr/sbin/nginx
```
启动nginx
```bash
nginx -t
```
如果报错：nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory
```bash
root@instance-8alx1qc6-1:~/soft/lnmp/nginx-1.6.2# ldd $(which /usr/local/nginx/sbin/nginx)
        linux-vdso.so.1 =>  (0x00007fff965fe000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007feb440e4000)
        libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1 (0x00007feb43eab000)
        libpcre.so.1 => not found
        libcrypto.so.1.0.0 => /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 (0x00007feb43ace000)
        libz.so.1 => /usr/local/lib/libz.so.1 (0x00007feb438b4000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007feb434ee000)
        /lib64/ld-linux-x86-64.so.2 (0x00007feb4430d000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007feb432ea000)
```
找到libpcre.so.1所在目录为/usr/local/lib64/
建立软连接:
```bash
ln -s /usr/local/lib/libpcre.so.1 /lib/libpcre.so.1 
```
测试成功输出：
```bash
root@instance-8alx1qc6-1:~/soft/lnmp/nginx-1.6.2# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
###启动php-fpm
```bash
cd /usr/local/php/etc
cp php-fpm.conf.default php-fpm.conf
修改php-fpm.conf 将user和group改成root(我是用root)
/usr/local/php/sbin/php-fpm -R
root用户启动必须加-R
```
###配置防火墙
配置防火墙开启80端口，不开启的话，有时防火墙会不让外网访问80端口我们就无法访问nginx配置的网站了。
修改防火墙配置：
输入以下命令：
```bash
vi /etc/sysconfig/iptables, 添加:
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```
重启防火墙：
```bash
systemctl restart iptables
```
启动nginx，从浏览器访问验证。

### 安装gitblog
```bash
wget https://github.com/jockchou/gitblog/archive/v2.2.1.tar.gz
tar -xvf v2.2.1.tar.gz
将目录下的内容复制到网站目录下
```
安装mbstring扩展，php.ini中加入:
```bash
extension=mbstring.so
short_open_tag=On
```
###修改nginx配置
```bash
user  root; // 不然会出现js/css文件forbiden的问题
server {
        listen       80;
        server_name  localhost;
        **root /root/github/blog.checkking.com;
        index index.html index.htm index.php;
        location ~ \.(jpg|png|gif|js|css|swf|flv|ico)$ {
            expires 12h;
        }**

        #charset koi8-r;

        access_log  logs/host.access.log;

        **location / {
            if (!-e $request_filename) {
                rewrite ^(.*)$ /index.php?$1 last ;
                break;
            }
        }
        location ~* ^/(doc|logs|app|sys)/ {
            return 403;
        }**

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;

        **location ~ .*\.(php|php5)?$ {
            fastcgi_connect_timeout 300;
            fastcgi_send_timeout 300;
            fastcgi_read_timeout 300;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }**

    }
```
然后reload nginx，即可访问。

###修改php.ini后重启php-fpm
找到php-fpm对应pid，可以在php-fpm.log中找到，然后：
```bash
kill USR2 pid
```

---------
[1] http://gitblogdoc.gitblog.cn/
