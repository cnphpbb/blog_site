+++
title = "Docker 常用命令（一）"
date = 2018-11-05T00:08:48+08:00
draft = false
menu = "main"
Categories = ["Docker", "Guide"]
Description = "常用的docker操作"
Tags = ["docker","guide"]
+++

使用了一段时间 `docker` 了。现整理出一些常用的 `docker cli` 指令，方便查找。

官方CLI网址 - [Docker CLI][1]

* 对开发，测试和运维人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。
* 确保了应用运行环境一致性，从而不会再出现 “这段代码在我机器上没问题啊？！” 这类问题。
* 由于 `Docker` 确保了执行环境的一致性，使得应用的迁移更加容易。

> Docker 好， Docker化 好！ 好！ 好！

### 开发常用的指令

```sh
# docker search debian  
# docker pull debian:<tag>
# docker run -it --rm debian // 启动一个容器，并且在运行结束后删除容器
# docker run --name mydebian -it --rm debian // 启动一个容器并命名为mydebian
# docker run -it --rm debian sh -c "cmd1 && cmd2" //启动容器并执行多个命令
# docker run -it --rm --mount type=bind,source=/<host-path>,target=/<container-path> debian // 启动容器并挂载本机目录
# docker run -it --rm -v <host-path>:<container-path> debian //上面的简化
```

### 创建新的网络,并在启动本地容器的时候 `指定ip`

```sh
# docker network create --subnet=172.20.0.0/16 mynet
# docker run --net mynet --ip 172.20.0.22 -it alpine:3 sh
```

### 清理操作

```sh
# docker image prune  // 删除临时image
# docker container prune // 删除已经停止的container
```

### 导入导出image操作

```sh
# docker save debian > debian.tar
# docker load < debian.tar
```

### 新建和删除image

```sh
# docker build -t cnphpbb/php7.1.23-alpine:latest .
# docker commit -a "cnphpbb" -m "php7.1.23" a404c6c174a2  cnphpbb/php7.1.23-alpine:v1
# docker rmi cnphpbb/php7.1.23-alpine
```

### 启动mysql

```sh
# docker run -d --restart=always \
  --name mysql \
  --net mynet \
  --ip 172.20.0.10 \
  -p 3306:3306 \
  -v D:\var\mysql:/var/lib/mysql \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
  mysql:5.6
```

### 启动redis

```sh
# docker run -d --restart=always \
  --name redis \
  --net mynet --ip 172.20.0.18 \
  -p 6379:6379 \
  redis:alpine
```

### 启动一个swoole服务

```sh
# docker run -d --restart=always \
  --name Bases \
  --net mynet \
  --ip 172.20.0.30 \
  -p 9901:9701 \
  -v D:\devlodGit\Bases:/data/Bases \
  --workdir=/data/Bases
  cnphpbb/php7.1.23-alpine:1.2.3 php start.php
```

> 使用docker作为开发环境，特别的方便。  
本地系统没有任何污染，拯救系统洁癖专用。  
尤其是在windows下面，省去了很多麻烦。

[1]: https://docs.docker.com/engine/reference/commandline/cli/