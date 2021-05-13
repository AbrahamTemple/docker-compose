# Docker Compose理解之路

官方简单引导

> https://docs.docker.com/compose/compose-file/compose-file-v3/

``` yml
# 声明compose版本
version: '3'

# 启动的服务
services:
  # 定义redis
  redis: 
    # 镜像名(dockerhub拉取的带版本号)
    image: redis:alpine
    # 容器名(一个镜像可以启动多个容器)
    container_name: sc-redis
    # 允许容器重启(不允许为no)
    restart: always
    # 卷挂载(本地目录:容器目录)
    volumes:
      - ./data/redis:/data
    # 添加环境变量(一般是些连接参数)
    environment:
      - REDIS_PASSWORD=123456
    # 要加入的网络
    networks:
      - sc-net
    # 暴露的端口(本地端口:容器端口)
    ports:
      - 6379:6379

networks:
  sc-net:
    external: false # true:指定此卷是在Compose之外创建的,docker-compose up不会尝试创建它,如果它不存在,则会引发错误
```
