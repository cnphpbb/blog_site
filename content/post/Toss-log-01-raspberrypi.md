---
title: "折腾日志(1) -- 自用树莓派内网穿透搭建GITEA"
date: 2020-10-14T12:25:15+08:00
draft: false
menu: "main"
Categories: ["Guide", "solutions"]
Description: "折腾 -- 自用树莓派内网穿透搭建GITEA"
Tags: ["linux", "raspberrypi", "debian", "solutions", "Guide"]
---

###  自备树莓派4B
> 这台自行组装的树莓派4B，主要用途: 存储，应用服务。

1. 安装树莓派系统, 并且安装到树莓派到硬盘中     
    
    这里不多说安装树莓派的方法，可以自行搜索 `度娘` 或 `谷歌`
    
2. 我的树莓派是啥样子的？？   (最近搭建了个图床可以上图了) **无图无真相**。    
 **主板** 1 块：    
 树莓派4B 8G     
 
 **SSD硬盘** 2块：   
   一块 256G(上面红色的),   
  另一块 512G(下面银色的)。    
  全部是 `浦科特（Plextor）` M8VC。    
  
 **扩展板** 2块：    
 - X829 扩展板:: 一款2.5' 双硬盘 NAS扩展板 [树莓派4B--树莓派NAS集群 X829 扩展板](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.6d7c2e8dbVKdC6&id=618003080805&_u=e1n4bqb3463)  
  
 - X735扩展板:: 电源管理和温控风扇扩展板 [树莓派4B--X735电源管理和温控风扇扩展板](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.6d7c2e8dbVKdC6&id=592730621196&_u=e1n4bqbbd1f) 


  俯视图 
![我的树莓派](https://gitee.com/kevinGunn/myimgs/raw/master/20201014085439.jpg)
右侧接口
![我的树莓派](https://gitee.com/kevinGunn/myimgs/raw/master/20201014085457.jpg)
左侧HHD指示灯
![我的树莓派](https://gitee.com/kevinGunn/myimgs/raw/master/20201014085508.jpg)


3. 安装好docker，frpc 这2个服务应用

> 以上都安装好了，就可以部署自己的服务器啦。 

### 开始折腾部署 [Gitea](https://gitea.io/zh-cn/)
> 因为树莓派是ARM架构的, 所以原来保存的docker镜像都要重新下载

1. Pull docker images    

    >  `mysql` & `percona` 的 `Docker Official Images` 中都没有ARM & ARM64 的 image    
    >  如果使用`mysql` 做数据库请选择 `mariadb`
    
    我本来也想使用 `gitea + redis + mysql` 的.    
    但是因为`mysql` 没有ARM64 的 image,     
    所以采用了  `gitea + redis + postgres` 
    
    - docker pull gitea/gitea:1.12.5
    - docker pull redis:alpine
    - docker pull postgres:13-alpine

    这里我折腾了近半天, 为啥呢?  
    
    1.  `mysql` & `percona` 的 `Docker Official Images` 中都没有ARM & ARM64 的 image 
    1.  `postgres` 的启动参数中不能像 `mysql` 那样, 直接使用 `environment` 来完成默认用户的权限, 需要分步走才能完成, 麻烦~
    1.  这里采用了创建好 `postgres` 容器后, 直接在数据库管理工具中创建 `gitea` 的 `用户，密码, 数据库`
    
    #### docker run
    
    1. 建立一个网络
    
        ```shell
        docker network create -d bridge --subnet=172.20.0.0/16 myGitNet
        ```
    2. 创建 `Redis` 容器
    
        ```shell
        docker run -d --name gitea-redis \
        --restart always \
        -v redis_data:/data \
        --net myGitNet --ip 172.20.0.201 \
        redis:6-alpine \
        --requirepass gitea
        ```
    
    3. 创建 `postgres` 容器
    
        ```shell
        docker run -d --name gitea-pgsql \
        --restart always \
        -v postgres_data:/var/lib/postgresql/data \
        -e POSTGRES_PASSWORD=hub#postgres \
        --net myGitNet --ip 172.20.0.229 \
        # 这里开放端口是要进行全程管理, 后期可以关闭哦.
        -p 5432:5432 \
        postgres:13-alpine
        ```
        
        使用数据库管理工具 `Navicat `,  建立`Postgres` 用户，密码, 数据库
        
        ```sql
        -- 
        -- 创建用户、密码
        --
        
        CREATE ROLE "kevinGit" LOGIN PASSWORD 'gY#0412#Pgsql';
        
        -- 
        -- 创建数据库
        -- 
        
        CREATE DATABASE "myGiteaDB"
        WITH
          OWNER = "kevinGit"
          TABLESPACE = "pg_default"
        ;
        ```
        
    4. 创建 `gitea` 容器
    
        ```shell
        docker run -d --name my-gitea \
        --restart always -v gitea_data:/data \
        --net myGitNet --ip 172.20.0.37 \
        -p 3022:22 -p 3000:3000 \
        gitea/gitea:1.12.5
       ```
    5. 配置 `gittea`,  访问 http://localhost:3000/
        
        安装上面的UI界面, 进行逐步的配置.
    
    6. 修改 gitea的配置文件, 把 cache & session 部分修改为 redis
    
    
    **到这里 gitea的docker部署 就完成了**  
    
    ### 部署 [FRP](https://github.com/fatedier/frp)
    上面部署好的 `gitea` 在内网的呀, 怎么才能在公网上访问到呢? 
    
    现在使用 `frp` 来完成 内网中部署好的gitea, 进行内网穿透.
    
    1. 在已有的云平台 或 有固定公网IP的服务器上部署好 frps服务
    
        - 配置文件。 
        
        ```ini
        [common]
        bind_port = 61007
        vhost_http_port = 8080
        token = myToken

        dashboard_port = 17500
        dashboard_user = kevinGunn
        dashboard_pwd = kevin#0412
 
 
        tcp_mux = true
        max_pool_count = 10
        ```
        
    2. 在树莓派上部署 frpc 服务
    
        - 配置文件。  
        
        ```ini
        [common]
        server_addr = **.**.**.**
        server_port = 61007
        token = myToken
        # ...... //这里有别的配置省略了
        [gov-gitea-web]
        type = http
        local_ip = 127.0.0.1
        local_port = 3000
        custom_domains = git.tp229.com
        
        [gitea-ssh]
        type = tcp
        local_ip = 127.0.0.1
        local_port = 3022
        remote_port = 3022
        ```
            
        > 不能把 `22` 端口, 直接映射为原创的`22` 端口
        > 所以就使用 `3022` 这个端口
    
    3. 验证, 一切正常啦~~!!!!!!
    4. 我采用了`caddy` 做了proxy, 最终https访问.
    
    ### 最后放置一张截图, 看下折腾到最后的成果.
    ![git.tp229.com](https://gitee.com/kevinGunn/myimgs/raw/master/20201015174822.png)    
    
    ### 总结一下, 我折腾过程遇到的坑
    > 1.  树莓派系统, 最后选择了 **OPENFANS开源社区 & 树莓派爱好者社区** 联合出品的 [Debian-Pi-Aarch64](https://github.com/openfans-community-offical/Debian-Pi-Aarch64)     
            "最初直接使用的树莓派官方的系统在安装 docker 及后期的docker 部署上 各种各样的问题. "    
            "还烧了一块 512GB的SSD. 还好在保,  漂亮温和的客服MM、直接就给我换了一块. 赞👍👍👍"
    2.  mysql 没有 docker的ARM镜像. 
    3.  postgres 提供的变量, 应该带 子数据库和子数据库账号&子数据库账号密码的变量. 免得多次部署
    4.  gitea 中 cache & seesion, 默认走的是内存模式, 应该在配置UI中增加此处的配置选择.
    5.  frp 不能 22 - 22 有点小遗憾. 我抽空看看代码为啥会这样.(或许是安全方面的考虑吧)
    6.  caddy2 配置上没有 caddy1 那样简单明了. 我还要看看是不是替换到v1版本. 越来越像NGINX的配置啦. 遗憾    
    