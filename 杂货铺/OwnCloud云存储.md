







# 1 介绍

OwnCloud 是一个开源免费专业的私有云存储项目，它能帮你快速在个人电脑或服务器上架设一套专属的私有云文件同步网盘，可以像百度云那样实现文件跨平台同步、共享、版本控制、团队协作等等。OwnCloud 能让你将所有的文件掌握在自己的手中，只要你的设备性能和空间充足，那么用起来几乎没有任何限制。OwnCloud　使用AGPLv3协议发布。本项目是基于PHP和SQLite，MySQL，Oracle或PostgreSQL数据库，所以它可以运行在所有的平台上。

OwnCloud 支持 Windows/Mac桌面端，IOS/Android手机端。基本可以替代在线网盘如百度网盘等。



# 2 安装

https://doc.owncloud.com/server/10.9/

```bash
# Create a new project directory
mkdir owncloud-docker-server

cd owncloud-docker-server

# Copy docker-compose.yml from the GitHub repository
wget https://raw.githubusercontent.com/owncloud/docs/master/modules/admin_manual/examples/installation/docker/docker-compose.yml

# Create the environment configuration file
cat << EOF > .env
OWNCLOUD_VERSION=10.9
OWNCLOUD_DOMAIN=localhost:8080
OWNCLOUD_ADMIN_USERNAME=admin
OWNCLOUD_ADMIN_PASSWORD=admin@123
HTTP_PORT=8080
EOF

# restart docker
systemctl restart docker

# Build and start the container
docker-compose up -d
```





修改docker-compose.yml将本地目录映射到容器内：

```yaml
version: "3"

#volumes:
#  files:
#    driver: local
#  mysql:
#    driver: local
#  redis:
#    driver: local

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    container_name: owncloud_server
    restart: always
    ports:
      - ${HTTP_PORT}:8080
    depends_on:
      - mariadb
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=mariadb
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - "/data0/owncloud/files:/mnt/data"

  mariadb:
    image: mariadb:10.5
    container_name: owncloud_mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=owncloud
      - MYSQL_DATABASE=owncloud
    command: ["--max-allowed-packet=128M", "--innodb-log-file-size=64M"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-u", "root", "--password=owncloud"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - "/data0/owncloud/mysql:/var/lib/mysql"

  redis:
    image: redis:6
    container_name: owncloud_redis
    restart: always
    command: ["--databases", "1"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - "/data0/owncloud/redis:/data"
```



安装客户端后配置同步目录：

![owncloud客户端同步本地文件夹](assets/owncloud%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%90%8C%E6%AD%A5%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%A4%B9.gif)





服务端页面查看上传情况和基本功能：

![owncloud服务端登录查看上传情况和基本功能](assets/owncloud%E6%9C%8D%E5%8A%A1%E7%AB%AF%E7%99%BB%E5%BD%95%E6%9F%A5%E7%9C%8B%E4%B8%8A%E4%BC%A0%E6%83%85%E5%86%B5%E5%92%8C%E5%9F%BA%E6%9C%AC%E5%8A%9F%E8%83%BD.gif)