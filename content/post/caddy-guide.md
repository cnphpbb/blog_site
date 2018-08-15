+++
title = "CaddyServer使用记录"
draft = false
date = "2016-12-22T14:37:00+08:00"
menu = "main"
Categories = ["Golang", "Caddy", "Guide"]
Description = "CaddyServer是一个基于Golang开发，易于配置和使用的Web服务器架设系统，支持HTTP/2、IPV6、WebSockets、FastCGI、Markdown等，无论你是设计师还是开发者都可以通过该系统轻松的架设自己的站点，传统的Web服务器架设很复杂，而CaddyServer更注重简化的操控。"
Tags = ["caddy", "guide","solutions","golang","开源"]
+++

Caddy是一个Golang编写的服务器软件，官方的宣传语“The HTTP/2 web server with automatic HTTPS”以及“Serve The Web Like It's 2016”简明表达了这个软件的优点和趋势，它拥有Apache或者Nginx所具有的web server模块，同时还有一些很有特色的功能，比如:

* HTTP/2
* Automatic HTTPS
* Multi-core
* Websockets
* Markdown
* IPv6
* Git
* FastCGI
* ...

在去年就开始使用Caddy，它提供的便捷配置，部署了内部测试服务的Web服务器、个人网站的Web服务器。配置项的语法简单粗暴，非常简洁明了。比nginx简单多了。  

**下面举几个例子：**  

* 简单的测试网站  

```  
:8530 {
    root /var/www/test_site
}
```  

* 使用git同步测试环境  

```  
:8530 {
    root /var/www/test_site/public
    git git.int5262.com/user_name/project_name {
        path ../
        interval 60
    }
    fastcgi / 127.0.0.1:9001 php
}
```  

* 站点跳转  

```  
http://48474.com, http://www.48474.com {
    redir https://48474.com{uri}
}
```  

看看吧，是不是太简单了？ 配置起来能具体到简洁到哭的指令。 详细文档请看官方的[User Guide](https://caddyserver.com/docs)。

**启动caddy也很简单**
``` 
/path/to/caddy -conf="/path/to/caddy/caddyfile" 
``` 


* Caddy官网 https://caddyserver.com  
	官方的下载页面提供了定制并编译的功能，很好用哦。不用下载源码编译。国内还要翻（GFW）封锁墙的。  
* Caddy一键安装脚本 https://getcaddy.com  

```  
curl https://getcaddy.com | bash
```  


