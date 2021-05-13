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
