# 环境搭建

## Linux 镜像

[官方 Download](https://www.centos.org/download/)

[CentOS 7.x Mirror](http://mirror.centos.org/altarch/)

[centos安装包下载_开源镜像站-阿里云](https://mirrors.aliyun.com/centos/?spm=a2c6h.13651104.0.0.6b8e12b2D09VJO)

**镜像分类**

1、DVD ISO：标准安装版，推荐

2、Everything ISO：完整版，集成很多软件（常用命令）

3、Minimal ISO：迷你版，自带软件较少

4、NetInstall：网络安装版，最小，但需要联网下载必要的程序和数据

（从 ISO 的大小也可以知道这四个版本的丰富程度）

![](https://assets.shiwokuaile.top/pic/202408/06729f6003aeeaacfa9a17252061bd091dfc85.png)



## JDK

```shell
vim /etc/profile

# 环境变量
JAVA_HOME=<具体路径>
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

# 生效
source /etc/profile
```

## Maven

[下载地址](https://maven.apache.org/download.cgi)  

**配置**  

```shell
# 编辑
vim /etc/profile


# 配置环境变量
MAVEN_HOME=<具体路径>
PATH=$MAVEN_HOME/bin:$PATH
export MAVEN_HOME

# 生效
source /etc/profile
```

## Docker

**设置 repository**  

```shell
# 检查
uname -m

yum install -y yum-utils
# 使用阿里云的 docker 源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 官方源似乎已经不可用了
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# repository 文件存放在 /etc/yum.repos.d/ 
# 修改 /etc/yum.repos.d/docker-ce.repo 文件，指定 docker 引擎的版本
```

**安装**  

```shell
# 安装最新版本
yum install docker-ce docker-ce-cli containerd.io

# 安装制定版本
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<版本> containerd.io

# 启动 docker
systemctl start docker

# 开机启动
systemctl enable docker.service
systemctl enable containerd.service

# 禁用
systemctl disable docker.service
systemctl disable containerd.service
```

**设置国内镜像源**

2024 年 6 月，docker 官方源凉了，指引：[docker凉了，国内镜像站全军覆没！-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2428707)

```shell
 # 编辑或创建 daemon.json
vim /etc/docker/daemon.json

 # 以下是可用的 docker 镜像源
 { 
  "registry-mirrors" : 
    [ 
      "https://docker.m.daocloud.io", 
      "https://noohub.ru", 
      "https://huecker.io",
      "https://dockerhub.timeweb.cloud" 
    ] 
}
```

**docker-compose**  

```shell
# 使用以下命令，下载稳定版本的 docker compose
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 赋予执行权限
chmod +x /usr/local/bin/docker-compose

# 注意非 root 用户需要 sudo

# 验证
[root@localhost images]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

## Python

Python2 和 Python3 差别挺大，而且 Python3 无法兼容 Python2。  

**下载**

[Window install 版本](https://www.python.org/downloads/windows/)

**pip3**   

```shell
# pip3 是 python3 用于引入依赖的工具
pip3 install numpy
pip3 install matplotlib
pip3 install opencv-python
```
