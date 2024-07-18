# 环境配置

## 配置静态 IP

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
# 重启网络
service network restart
```

```md
# 修改为静态IP
BOOTPROTO=static
# 开机启用
ONBOOT=yes
# 配置DNS
DNS2=114.114.114.114
# 静态IP
IPADDR=192.168.3.131
# 设置子网掩码
NETMASK=255.255.255.0
# 网关，根据具体网络情况填写
GATEWAY=192.168.3.x
```

## 替换 yum 源为国内源

```shell
# 查看 yum 源
yum info yum

# 备份原来的 CentOS-Base.repo，方便回滚
cd /etc/yum.repos.d
cp CentOS-Base.repo CentOS-Base.repo.bak

# 覆盖 CentOS-Base.repo 里的 yum 源
```

**阿里**  

```shell
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

## 安装常用的命令

```shell
# ifconfig
yum install net-tools.x86_64

# telnet
yum install telnet.x86_64

# wget
yum install wget.x86_64

# vim
yum install vim

# unzip
yum install unzip.x86_64

# git
yum install git
```



## 开机启动

```shell
# 编辑
vim /etc/rc.d/rc.local 

# 添加启动脚本
/opt/server/xxx/restart.sh

# 授予执行权限
chmod +x /etc/rc.d/rc.local 
```



# 常用命令

## nohup

远程连接会话关闭后，避免程序自动退出了。



## firewalld

```shell
# 重启防火墙
firewall-cmd --reload

# 列出开放的端口
firewall-cmd --zone=public --list-ports

# 开放 3306 端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```