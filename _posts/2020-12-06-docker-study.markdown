---
layout: post
title:  "Docker使用"
date:   2020-12-06 13:10:45 +0800
lang: zh-cmn-Hans
categories: docker docker-compose
---
Docker使用部署手册。

> 使用 docker 来部署相关系统，其中包含一个Tomcat服务、一个 Nginx 作为前台服务，一个Mysql服务、一个Redis服务，使用 docker-compose 来部署。

<figure>
<a><img src="{{site.url}}/asstes/images/docker-path.png"></a>
</figure>

docker-compose.yml 的内容以及相关说明

```yaml
version: '2.0'

services: 
#   nginx 配置相关信息，其中服务名称为 nginx，直接使用 docker 官方默认的 nginx 镜像来配置，
  nginx:
    container_name: nginx_server
    image: nginx
    # build: ./nginx
    # 配置 Nginx 配置文件位置为当前目录下文件
    # 配置前端代码默认位置
    # 配置端口信息，其中映射到宿主机两个端口
    # 80 端口为MUISE X.SYSTEM管理系统
    # 81 端口为MUISE X.SYSTEM大屏展示系统
    # volumes 下的配置将宿主机的目录映射到容器里对应的目录
    volumes: 
      - ./nginx/conf:/etc/nginx/conf.d
      - ./nginx/dist:/dist
      - ./nginx/data_show:/data_show
    #   ports 下将宿主机的端口号映射到容器中对应的端口号
    ports:
      - "80:8081"
      - "81:8082"
    #   在 nginx 服务里面使用 web 服务时，使用web-host代替，比如 http://web-host:8081/api
    links:
      - web:web-host
    #   nginx 服务依赖于 web 服务，在 web服务启动后再启动 nginx 服务
    depends_on: 
      - web
#  web 服务使用了 server 目录作为镜像，后面会有说明
  web:
    container_name: web_server
    build: ./server
    # ports: 
    #   - "8081:8080"
    # volumes:
      # - ./server/logs:/mnt/apache-tomcat-8.5.56/logs
      # - ./server/webapps:/mnt/apache-tomcat-8.5.56/webapps
    links:
      - redis:redis-host
      - mysql:mysql-host
    depends_on: 
      - redis
      - mysql
# redis 使用默认的官方 redis 镜像
  redis:
    image: redis
    container_name: redis_server
    # ports:
    #   - "8084:6379"
# mysql 使用默认的官方 mysql 镜像
  mysql:
    container_name: mysql_server
    image: mysql:5.7
    command: 
      --character-set-server=utf8mb4 
      --collation-server=utf8mb4_unicode_ci
    volumes:
    # 初始化mysql，执行mysql/init目录下所有的sql语句，按照 01、02、03 的顺序执行
      - ./mysql/init:/docker-entrypoint-initdb.d/
    #   存放mysql数据到宿主机硬盘上，否则docker重启后数据会丢失
      - ./mysql/data:/var/lib/mysql
    restart: always
    environment:
    # 配置MYSQL的账号、密码、使用的数据库
      TZ: Asia/Shanghai
      MYSQL_DATABASE: test_db
      MYSQL_USER: db_user
      MYSQL_PASSWORD: db_user@pwd
      MYSQL_ROOT_HOST: 0.0.0.0
      MYSQL_ROOT_PASSWORD: root@pwd
    # ports:
    #   - "13306:3306"

# volumes:
#   logvolume1:
#     driver: local


```

server/Dockerfile 的内容说明

```docker
FROM  centos:7
LABEL authors=msmsz
LABEL description=这是JDK、Tomcat配置环境
# 拷贝文件，将宿主机相关的文件拷贝到容器中
ADD jdk-8u231-linux-x64.tar.gz /mnt/
ADD apache-tomcat-8.5.56.tar /mnt/
ADD web.tar /mnt/apache-tomcat-8.5.56/webapps/
ADD context.xml /mnt/apache-tomcat-8.5.56/conf/
# 配置环境变量
ENV WORK_PATH /mnt/
ENV TOMCAT_HOME /mnt/apache-tomcat-8.5.56
# ADD web.war ${TOMCAT_HOME}/webapps/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
WORKDIR ${WORK_PATH}

#配置java与tomcat环境变量
ENV JAVA_HOME /mnt/jdk1.8.0_231
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /mnt/apache-tomcat-8.5.56
ENV CATALINA_BASE /mnt/apache-tomcat-8.5.56
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080

#启动时运行tomcat
# ENTRYPOINT ["/mnt/apache-tomcat-8.5.56/bin/startup.sh" ]
CMD ["/mnt/apache-tomcat-8.5.56/bin/catalina.sh","run"]
# CMD /mnt/apache-tomcat-8.5.56/bin/startup.sh 

```

常用docker命令
```bash
# docker 镜像、容器列表
docker image ls
docker ps 

# 进入 docker 容器，可以执行各种命令
docker exec -it fa538e3105bd /bin/bash

# 保存、导入 docker 镜像，可以使用的命令：
#  docker save 保存的是镜像（image），docker export 保存的是容器（container）；
#  docker load 用来载入镜像包，docker import 用来载入容器包，但两者都会恢复为镜像；
#  docker load 不能对载入的镜像重命名，而 docker import 可以为镜像指定新名称。
docker export 98ca36 > muise_web_server.tar
docker save ubuntu:load > muise_web_server.tar
docker load < muise_web_server.tar

# 删除所有已经不使用的docker镜像，清除不必要的空间占用，慎用
docker system prune --all
```

---参考资料---
* [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)
* [https://hub.docker.com/\_/nginx](https://hub.docker.com/_/nginx)
* [https://hub.docker.com/\_/redis](https://hub.docker.com/_/redis)
* [https://hub.docker.com/\_/mysql](https://hub.docker.com/_/mysql)
* [https://github.com/docker-library/mysql/blob/ee33a2144a0effe9459abf02f20a6202ae645e94/5.7/Dockerfile.debian](https://github.com/docker-library/mysql/blob/ee33a2144a0effe9459abf02f20a6202ae645e94/5.7/Dockerfile.debian)