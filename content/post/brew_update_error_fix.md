+++
title = "Error: /usr/local must be writable! 解决方法"
draft = false
date = "2016-12-26T12:51:15+08:00"
menu = "main"
Categories = ["mac", "brew"]
Description = "Mac下执行`brew update`时出现以下错误提示：Error: /usr/local must be writable! 解决方法。"
Tags = ["mac", "brew", "guide"]
+++

今天在Mac下执行`brew update`时出现以下错误提示：
  
> $ brew update  
> Error: /usr/local must be writable!   

以前一直默认用的好好的,突然出现这种情况导致不能正常使用了。
按我习惯性的方法，修改下目录的权限吧

> $ sudo chmod -R g+w /usr/local  
  $ brew update  
还是出现错误提示
Error: /usr/local must be writable!  

有点晕啦，怎么还是有问题呢？ 仔细想了下，自检以下吧。

> $ brew doctor

上面是修改目录的写权限，现在修改目录的用户权限吧！！

> $ sudo chown -R $(whoami) /usr/local

再次执行`brew doctor`, 正常啦！！！  
再次执行`brew update`, 终于正常，正常啦！！！

**总结一下：** 
 
* 升级了macos后没执行`brew doctor`，导致`brew` 出现问题。    
* 升级系统，一定要检测下第三方软件是否能正常工作。     
* 检测工作没执行到底呀。

