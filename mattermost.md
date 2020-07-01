https://github.com/mattermost/mattermost-server

### 安装服务器

安装sudo
```
apt install sudo
```

下载解压最新版服务端
```
wget https://releases.mattermost.com/5.24.1/mattermost-5.24.1-linux-amd64.tar.gz
tar -xvzf mattermost*.gz
```

https://mattermost.com/download/

位移并创建数据目录
```
mv mattermost /opt
mkdir /opt/mattermost/data
```

创建用户并赋予权限
```
useradd --system --user-group mattermost
chown -R mattermost:mattermost /opt/mattermost
chmod -R g+w /opt/mattermost
```

创建数据库

修改配置文件
```
nano /opt/mattermost/config/config.json
```
```
    "SqlSettings": {
        "DriverName": "mysql",
        "DataSource": "dbusername:dbpasswd@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4,utf8&readTimeout=30s&writeTimeout=30s",

    "FileSettings": {
        "EnableFileAttachments": true,
        "EnableMobileUpload": true,
        "EnableMobileDownload": true,
        "MaxFileSize": 52428800,
        "DriverName": "local",
        "Directory": "/opt/mattermost/data/",
```

启动并检查是否可以访问
```
cd /opt/mattermost
sudo -u mattermost ./bin/mattermost
```

创建进程
```
nano /etc/systemd/system/mattermost.service
```
```
[Unit]
Description=Mattermost
After=syslog.target network.target mysqld.service

[Service]
Type=notify
WorkingDirectory=/opt/mattermost
User=mattermost
ExecStart=/opt/mattermost/bin/mattermost
PIDFile=/var/spool/mattermost/pid/master.pid
TimeoutStartSec=3600
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target
```

配置自启
```
chmod 664 /etc/systemd/system/mattermost.service
systemctl daemon-reload
systemctl enable mattermost
systemctl start mattermost
```

创建nginx vhost配置
```
nano /usr/local/nginx/conf/vhost/im.ans-union.com.conf
```
```
server {
    listen 80;
    server_name im.ans-union.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name im.ans-union.com;

    ssl_certificate           /etc/ssl/ans-union.com.crt;
    ssl_certificate_key       /etc/ssl/ans-union.com.key;
    ssl_protocols             TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers               'TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384:!TLS_AES_128_GCM_SHA256:!ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-CCM8:ECDHE-ECDSA-AES256-CCM:!ECDHE-ECDSA-AES128-CCM:!ECDHE-ECDSA-AES128-CCM8:!ECDHE-ECDSA-ARIA128-GCM-SHA256:ECDHE-ECDSA-ARIA256-GCM-SHA384';
    ssl_ecdh_curve            secp384r1;
    add_header                Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
    ssl_session_cache         builtin:1000 shared:SSL:10m;
    ssl_session_timeout       6h;
    ssl_dhparam               dhparam.pem;

   location ~ /api/v[0-9]+/(users/)?websocket$ {
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       client_max_body_size 50M;
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       client_body_timeout 60;
       send_timeout 300;
       lingering_timeout 5;
       proxy_connect_timeout 90;
       proxy_send_timeout 300;
       proxy_read_timeout 90s;
       proxy_pass http://backend;
   }

   location / {
       client_max_body_size 50M;
       proxy_set_header Connection "";
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       proxy_read_timeout 600s;
       proxy_cache mattermost_cache;
       proxy_cache_revalidate on;
       proxy_cache_min_uses 2;
       proxy_cache_use_stale timeout;
       proxy_cache_lock on;
       proxy_http_version 1.1;
       proxy_pass http://backend;
   }
}

upstream backend {
   server 127.0.0.1:8065;
   keepalive 32;
}

proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mattermost_cache:10m max_size=3g inactive=120m use_temp_path=off;
```

配置生效
```
nginx -s reload
```

### 配置服务器

登录web面板，创建管理员用户

根据域名设置Site URL

设置smtp发信