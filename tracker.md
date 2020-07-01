### 安装依赖

下载并解压go
```
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
tar -xzvf go1.14.1.linux-amd64.tar.gz -C /usr/bin
```

添加到环境
```
nano ~/.bash_profile
export PATH=$PATH:/usr/bin/go/bin
```

生效
```
source ~/.bash_profile
```

### 下载安装程序

下载从程序
```
git clone https://github.com/chihaya/chihaya.git
```

编译并安装
```
cd chihaya
go build ./cmd/chihaya
```
```
chmod +x chihaya
cp chihaya /usr/bin
```

检查可用性
```
chihaya --help
```

### 配置程序

下载配置文件
```
wget https://raw.githubusercontent.com/chihaya/chihaya/master/dist/example_config.yaml -O /etc/chihaya.yaml
```

修改配置文件
```
nano /etc/chihaya.yaml
```
```
汇报间隔设置为15分钟，最小间隔5分钟

https_addr: "0.0.0.0:8443"

tls_cert_path: "/etc/ssl/ansopen.org.crt"
tls_key_path: "/etc/ssl/ansopen.org.key"

修改
real_ip_header: "CF-Connecting-IP"

将所有有关UDP的行注释掉
```

创建服务
```
cat > /lib/systemd/system/chihaya.service <<EOF
[Unit]
Description=chihaya tracker server
    
[Service]
User=root
ExecStart=/usr/bin/chihaya --config /etc/chihaya.yaml
Restart=on-abort
LimitCORE=infinity
LimitNOFILE=infinity
LimitNPROC=infinity
    
[Install]
WantedBy=multi-user.target
EOF
```

启用
```
systemctl enable chihaya
systemctl start chihaya
systemctl status chihaya
systemctl stop chihaya
systemctl restart chihaya
```

### CDN配置

防火墙规则，允许主机名=tracker.ansopen.org并且URL查询字符串包含info_hash
(http.host eq "tracker.ansopen.org" and http.request.uri.query contains "info_hash")

页面规则，tracker.ansopen.org/* 缓存级别为绕过 (8443端口可忽略)

### 网页配置

下载nginx vhost配置
```
wget https://cdn.jsdelivr.net/gh/ansopen/tracker@master/tracker.ansopen.org.conf -O /usr/local/nginx/conf/vhost/tracker.ansopen.org.conf
```

创建目录
```
mkdir /home/www
mkdir /home/www/status
mkdir /home/www/probe
```

下载文件
```
wget https://cdn.jsdelivr.net/gh/ansopen/tracker@master/index.html -O /home/www/index.html
wget https://cdn.jsdelivr.net/gh/ansopen/tracker@master/status/index.html -O /home/www/status/index.html
wget https://cdn.jsdelivr.net/gh/ansopen/tracker@master/probe/index.php -O /home/www/probe/index.php
```