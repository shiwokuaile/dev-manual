# 环境搭建

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

**Dockerfile**  

```dockerfile
# 构建 xxx.jar 的 docker 镜像
FROM openjdk:8
WORKDIR /opt/app
COPY xxx.jar /opt/app/xxx.jar
CMD ["java", "-jar", "xxx.jar"]
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
