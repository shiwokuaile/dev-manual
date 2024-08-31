# docker

## 常用命令

```shell
# 查看容器日志
docker logs <容器 id>

# 查看镜像
docker images

# 移除容器
docker rm <容器 id>

# 移除镜像
docker rmi <镜像 id>

# 进入容器
docker exec -it <容器 id> /bin/bash
```

## Dockerfile

```dockerfile
# 构建 xxx.jar 的 docker 镜像
FROM openjdk:8
# docker 容器内的工作目录
WORKDIR /opt/app
# 将本地当前目录的 xxx.jar 拷贝到 docker 容器的 /opt/app 目录
COPY xxx.jar /opt/app/xxx.jar
CMD ["java", "-jar", "xxx.jar"]
```

```shell
# 在 Dockerfile 当前目录下构建镜像，镜像名为 xxx
docker build -t xxx .
# 创建容器并运行，端口 9528，容器名 yyy，镜像 xxx
docker run -d -p 9528:9528 --name yyy xxx
```

## docker-compose 部署自定义镜像

```yaml
version : '3'
services:
  xlh:
    image: xxx-server:latest
    container_name: xxx
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - "./xxx/files:/opt/server/xxx/files"
    networks:
      - node-network
networks:
  node-network:
```
