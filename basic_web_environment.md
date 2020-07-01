### 安装nginx

##### 安装依赖
更新软件源
```
apt update
```

安装所需的依赖
```
apt install -y build-essential libpcre3 libpcre3-dev zlib1g-dev unzip git libssl-dev wget curl nano
```

##### 创建用户与目录
```
groupadd -r www && useradd -r -g www -s /sbin/nologin -d /usr/local/nginx -M www
mkdir -p /usr/local/nginx
rm -rf /usr/sbin/nginx /sbin/nginx
```

##### 下载nginx源码
```
wget --no-check-certificate http://nginx.org/download/nginx-1.16.1.tar.gz && tar xzf nginx-1.16.1.tar.gz && rm -rf nginx-1.16.1.tar.gz
cd nginx-1.16.1/
git clone https://github.com/google/ngx_brotli.git
cd ngx_brotli && git submodule update --init && cd ../
git clone https://github.com/aperezdc/ngx-fancyindex.git
cd ngx-fancyindex && git submodule update --init && cd ../
```

##### 编译安装
```
./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-http_sub_module --with-stream --with-stream_ssl_module --add-module=./ngx_brotli --add-module=./ngx-fancyindex
make -j$(nproc) && make install
ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx
ln -s /usr/local/nginx/sbin/nginx /sbin/nginx
cd
```

##### 创建目录并整理
```
mkdir -p /usr/local/nginx/conf/vhost
mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
rm -rf /usr/local/nginx/conf/nginx.conf /root/nginx-1.16.1
```

##### 创建配置文件与进程
配置文件
```
nano /usr/local/nginx/conf/nginx.conf
```
```
user  www www;
worker_processes  auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 65536;

error_log  logs/error.log crit;
pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    server_tokens off;
    include       mime.types;
    default_type  application/octet-stream;
    
    log_format  cdn  '$http_x_forwarded_for - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" ';

    sendfile       on;
    sendfile_max_chunk 512k;
    tcp_nopush     on;
    tcp_nodelay    on;

    keepalive_timeout  60;

    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 256k;

    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
    gzip_vary on;
    gzip_proxied   expired no-cache no-store private auth;
    gzip_disable   "MSIE [1-6]\.";

    brotli on;
    brotli_comp_level 6;
    brotli_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;

include vhost/*.conf;
}
```

进程
```
nano /lib/systemd/system/nginx.service
```
```
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
ExecStartPost=/bin/sleep 0.1
ExecStartPre=/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /usr/local/nginx/logs/nginx.pid
TimeoutStopSec=5
KillMode=mixed

PIDFile=/usr/local/nginx/logs/nginx.pid
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

##### 收尾工作
```
openssl dhparam -dsaparam -out /usr/local/nginx/conf/dhparam.pem 4096
```
```
systemctl enable nginx.service
systemctl start nginx.service
systemctl restart nginx.service
systemctl status nginx.service --no-pager
```
```
echo "/usr/local/nginx/logs/*.log {
    monthly
    rotate 2
    size 1m
    compress
    delaycompress
    missingok
    postrotate
        /sbin/nginx -s reload
    create 644 root staff
}" > /etc/logrotate.d/nginx
```

### 安装证书

```
nano /etc/ssl/域名.crt
nano /etc/ssl/域名.key
```

### 安装php7.3
安装程序(包含composer)
```
apt install -y php-fpm php-common php-curl php-mbstring php-xml php-dom php-zip php-mysql php-gd composer
```

调整配置
```
sed -i "s/;cgi.fix_pathinfo=0/cgi.fix_pathinfo=1/g" /etc/php/7.3/fpm/php.ini
sed -i "s/user = www-data/user = www/g" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/group = www-data/group = www/g" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/listen.owner = www-data/listen.owner = www/g" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/listen.group = www-data/listen.group = www/g" /etc/php/7.3/fpm/pool.d/www.conf
service php7.3-fpm restart
```

### 安装MariaDB

##### 安装程序
```
apt install mariadb-server
```

##### 配置密码
```
mysql_secure_installation
```

##### 数据库操作
以root身份登录
```
mysql -u root -p
```

创建数据库
```
CREATE DATABASE dbname;
GRANT ALL ON dbname.* TO 'dbusername'@'localhost' IDENTIFIED BY 'dbpasswd' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

删除数据库
```
DROP DATABASE dbname;
```

查看数据库
```
SHOW DATABASES;
```

退出数据库控制台
```
exit;
```

```
mysqldump -u root -p dbname > dbname.sql    //数据库导出
mysql -u root -p dbname < dbname.sql    //数据库导入
```

### 安装redis
```
apt install redis
```
