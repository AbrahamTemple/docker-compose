# Docker Compose进阶之路

## 官方简单引导

> https://docs.docker.com/compose/compose-file/compose-file-v3/

## 简单案例快速入门
``` yml
# 声明compose版本
version: '3'

# 声明启动的服务
services:
  # 定义redis服务
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
    # 要加入的docker网络
    networks:
      - sc-net
    # 暴露的端口(本地端口:容器端口)
    ports:
      - 6379:6379
  
  zipkin-server:
    # dockerhub社区镜像
    image: openzipkin/zipkin
    container_name: sc-zipkin-server
    restart: always
    volumes:
      - ./data/zipkin-server/logs:/var/logs
    networks:
      - sc-net
    ports:
      - 9411:9411
    # es和rabbitmq结合使用,促进es异步搜索
    environment:
      - RABBIT_ADDRESSES=rabbitmq:5672
      - RABBIT_MQ_PORT=5672
      - RABBIT_PASSWORD=guest
      - RABBIT_USER=guest
      - STORAGE_TYPE=elasticsearch
      - ES_HOSTS=http://elasticsearch:9200
    # 1、docker-compose up:在zipkin启动之前启动rabbitmq
    # 2、docker-compose stop:在zipkin停止之前停止rabbitmq
    depends_on:
      - rabbitmq

  elasticsearch:
    image: elasticsearch:alpine
    container_name: sc-elasticsearch
    restart: always
    # 1、节点与其它节点共享了cluster.name就能加入集群当中
    # 2、锁定物理内存地址，防止es内存被交换出去，也就是避免es使用swap交换分区，频繁的交换，会导致IOPS变高。
    # 3、限制es进程启动占用的大小
    # 4、换个节点名字，用处不大
    environment:
      - cluster.name=elasticsearch
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - node.name=elasticsearch_node_1
    # 限制每个用户可使用的资源
    ulimits:
      memlock:
        soft: -1 # 软限制:当超过限制值只会报警
        hard: -1 # 硬限制:必定不能超过限制值
    # 挂载数据库、配置和日志    
    volumes:
      - ./data/elasticsearch/data:/usr/share/elasticsearch/data
      - ./data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./data/elasticsearch/logs:/usr/share/elasticsearch/logs
    networks:
      - sc-net
    ports:
      - 9200:9200
      - 9300:9300
    
networks:
  sc-net:
    external: false # true:指定此卷是在Compose之外创建的,docker-compose up不会尝试创建它,如果它不存在,则会引发错误
```

## 复杂案例进阶深化

``` yml
version: '3.9'
services:
    db:
        container_name: db
        image: mysql8.0.22
        # dockerfile的CMD就是这个command
        command: [--default-authentication-plugin=mysql_native_password]
        ports:
            - ${MYSQL_PORT}3306
        environment:
            MYSQL_ROOT_PASSWORD ${MYSQL_ROOT_PASSWORD}
            MYSQL_DATABASE ${MYSQL_DATABASE}
            MYSQL_USER ${MYSQL_USER}
            MYSQL_PASSWORD ${MYSQL_PASSWORD}
        volumes:
            - .dbvarlibmysql

    nginx:
        container_name: nginx
        # 在构建时应用的配置选项
        build: 
            # 指定Dockerfile的目录(nginx)的路径
            context: .nginx 
            # 添加构建参数，这是只能在构建过程中访问的环境变量
            args:
                NGINX_SYMFONY_SERVER_NAME ${NGINX_SYMFONY_SERVER_NAME}
                KIBANA_PORT ${KIBANA_PORT}
        ports:
            - ${NGINX_PORT}80
        depends_on:
            - php
        environment:
            - NGINX_ENVSUBST_OUTPUT_DIR=etcnginxconf.d
            - NGINX_ENVSUBST_TEMPLATE_DIR=etcnginxtemplates
            - NGINX_ENVSUBST_TEMPLATE_SUFFIX=.template
            - NGINX_SYMFONY_SERVER_NAME=${NGINX_SYMFONY_SERVER_NAME}
        # 匿名挂载
        volumes:
            - .logsnginxvarlognginxcached
            - .symfonyvarwwwsymfonycached
```
