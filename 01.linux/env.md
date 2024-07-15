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
