# Maven

## 部署

**1、下载**

[Maven – Download Apache Maven](https://maven.apache.org/download.cgi)

**2、配置**

```shell
 vim /etc/profile

// 配置环境变量
MAVEN_HOME=/opt/bin/apache-maven-3.9.6
PATH=$MAVEN_HOME/bin:$PATH
export MAVEN_HOME

// 生效
source /etc/profile
```



## 执行顺序

> maven 的第三方插件的执行顺序与在 pom 文件声明的顺序一致，先声明先执行。

## 版本说明

> 一般来讲，依赖的版本有两种。
> 
> 1、SNAPSHOT：开发版本，不稳定
> 
> 2、RELEASE：正式版本
> 
> 3、ALPHA：内部测试版本
> 
> 4、BEAT：公测版本，近似于 RELEASE 版本
> 
> 5、RC：正式版本的候选版本

## 问题列表

> 问题：SpringBoot 打包插件，Unable to find main class
> 
> 解决：在 pom 文件找到插件的声明位置，然后添加 mainclass 位置
> 
> ```xml
> <plugin>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-maven-plugin</artifactId>
>     <version>2.5.14</version>
>     <executions>
>         <execution>
>             <goals>
>                 <goal>repackage</goal>
>             </goals>
>         </execution>
>     </executions>
>     <configuration>
>         <mainClass>com.xxx.yyyy.zzz.Application</mainClass>
>     </configuration>
> </plugin>
> ```
> 
> 问题：错误提示：maven 项目 deploy 报错：Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin。
> 
> 分析：没有在工程 pom 文件声明 distributionManagement（jar 上传的路径）
> 
> 解决：maven 的配置声明远程仓库，参考：https://blog.51cto.com/u_15072903/4217929
