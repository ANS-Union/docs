### Oracle机器重装

##### 使用Oracle官方Ubuntu18.04 Minimal 先进行如下操作
```
mkdir /etc/network/interfaces.d
echo "# This file describes the network interfaces available on your system
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback
allow-hotplug ens3
iface ens3 inet dhcp
" > /etc/network/interfaces
```

##### 脚本DD安装Debian10
```
bash <(wget --no-check-certificate -qO- 'https://cdn.jsdelivr.net/gh/ansopen/static/fixinstall.sh') -d 10 -v 64 -a -firmware -p OracleDefaultPasswd
```
https://moeclub.org/attachment/LinuxShell/InstallNET.sh


##### 初始登录信息
```root
MoeClub.org
```

##### 修改服务器root密码
```
passwd
```

##### 修改软件源(CN)
```
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
```

##### 修改软件源(KR)
```
sed -i 's/deb.debian.org/ftp.kr.debian.org/g' /etc/apt/sources.list
sed -i 's|security.debian.org/debian-security|ftp.kr.debian.org/debian-security|g' /etc/apt/sources.list
```

##### 修改软件源(JP)
```
sed -i 's/deb.debian.org/ftp.jp.debian.org/g' /etc/apt/sources.list
sed -i 's|security.debian.org/debian-security|ftp.jp.debian.org/debian-security|g' /etc/apt/sources.list
```

##### 修改软件源(US)
```
sed -i 's/deb.debian.org/ftp.us.debian.org/g' /etc/apt/sources.list
sed -i 's|security.debian.org/debian-security|ftp.us.debian.org/debian-security|g' /etc/apt/sources.list
```

```
apt update
```

##### 配置网络
```
apt install net-tools
```

bbr
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
echo "fs.file-max = 65536" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr
```

### OVH机器重装
登录web面板，选择debian10

bbr
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
echo "fs.file-max = 65536" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr
```

升级
```
apt update && apt upgrade
```

### ARM Storage重装

##### 选择系统
重装选择Debian9 English
勾选Custom installation

##### 分区
将swap分区调整为256MB
删除默认的分区2
将分区1调整为Use the remaining space

##### 补充信息
输入自定义hostname
勾选Use the distribution kernel