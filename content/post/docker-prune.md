+++
title = "Docker：清理Docker占用的磁盘空间"
date = 2021-07-02T09:38:48+08:00
draft = false
menu = "main"
Categories = ["Docker", "Guide"]
Description = "常用的docker操作, Docker：清理Docker占用的磁盘空间"
Tags = ["docker","guide"]
+++

现整理出一些常用的 **清理Docker占用的磁盘空间** 指令，方便查找。   

**官方CLI网址 - [Docker CLI][1]**

> **Docker** 好， **Docker化** 好！ 好！ 好！

### 容器清理

- **`docker container prune`** : 仅删除停止运行的容器。
- **`docker rm -f $(docker ps -aq)`** : 删除所有容器（包括停止的、正在运行的）。
- **`docker container rm -f $(docker container ls -aq)`**:  同上。

### 镜像清理

**`docker rmi <image id>`** ：通过镜像的id来删除指定镜像。    

有一些镜像是隐形的：

- 子镜像，就是被其他镜像引用的中间镜像，不能被删除。
- 悬挂状态的镜像，就是不会再被使用的镜像，可以被删除。

其他命令：

- **`docker image ls -f dangling=true`**:  可以列出所有悬挂状态的镜像.  
并使用命令 `docker image rm $(docker image ls -f dangling=true -q)` 或 `docker image prune` 进行删除。
- **`docker image rm $(docker image ls -q)`** ：删除所有镜像。但正在被容器使用的镜像无法删除。

### 数据卷清理
- **`docker volume rm $(docker volume ls -q)`** ：删除不再使用的数据卷。
- **`docker volume prune`** ：同上。

### 缓存清理
**Docker 18.09** 引入了 **BuildKit** ，提升了构建过程的性能、安全、存储管理等能力。   
**`docker builder prune`** ：删除 build cache。

###  一键清理

**`docker system df`** 命令，类似于 **Linux**上的 **df** 命令，用于查看 **Docker** 的磁盘使用情况：

**TYPE**列出了 **Docker** 使用磁盘的 4 种类型：

- **Images** ：所有镜像占用的空间，包括拉取下来的镜像，和本地构建的。
- **Containers** ：运行的容器占用的空间，表示每个容器的读写层的空间。
- **Local Volumes** ：容器挂载本地数据卷的空间。
- **Build Cache** ：镜像构建过程中产生的缓存空间（只有在使用 BuildKit 时才有，Docker 18.09 以后可用）。

最后的 RECLAIMABLE 是可回收大小。

- **`docker system prune`** : 可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及 dangling 镜像（即无 tag 的镜像）。
- **`docker system prune -a`** : 清理得更加彻底，可以将没有容器使用 **Docker** 镜像都删掉。

> 注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的 Docker 镜像都删掉了。


[1]: https://docs.docker.com/engine/reference/commandline/cli/