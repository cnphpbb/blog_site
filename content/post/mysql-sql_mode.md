---
title: "Mysql 的 sql_mode 配置"
date: 2018-11-05T02:31:05+08:00
draft: true
menu: "main"
Categories: ["mysql", "Guide"]
Description: "Mysql 的 sql_mode 配置"
Tags: ["mysql", "guide","sql_mode"]
---

在 `oracle` 或 `sqlserver` 或 `PostgreSQL` 中，如果某个表的字段设置成 `not null`， `insert` 或 `update` 时不给这个字段赋值，必然报错。

比如下面这样：

表 `t_test(id,name)`中 `id`, `name` 都不允许为空，

`insert into t_test(name) values('xxx')` 必然报错，这是天经地义的事情，但是在mysql中这是有可能成功，具体取决于 `sql_mode` 的设置。

`sql_mode` 可以分为二大类：

1. 所谓的宽松无敌模式

    `my.cnf` 中 `sql_mode` 设置为空或仅 `NO_ENGINE_SUBSTITUTION`.  

    这种模式下，`not null` 的字段，在 `insert` 或 `update` 时不设置值也能成功，db在插入时，会自动给默认值。  
    比如: `int` 会给 `0` 值，甚至可以把 `abc` 赋值给 `int型` 的字段`（`当然，db会自动忽略该值，变成默认值 **0** `）`

1. 所谓的严格模式

    具体有很多可选值，设置成严格模式后，`mysql` 就跟传统的 `oracle`、`sqlserver`、 `PostgreSQL`, 表现一致了，这也是我个人强烈推荐的模式。


最后，无耻的从网上抄一段贴在这里备份：

如果使用 `mysql`，为了继续保留大家使用 `oracle` 的习惯，可以对 `mysql` 的`sql_mode` 设置如下：

在my.cnf添加如下配置

```vim
[mysqld]
sql_mode='ONLY_FULL_GROUP_BY,NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,
ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,PIPES_AS_CONCAT,ANSI_QUOTES'
```

补充两句：

在 `mysql 5.7` 版本以后 默认是开启了 **严格模式** 的。 这点我也比较赞成的。

但在开发过程中，因为种种原因（工期缩短，开发人员的水平，等等）需要使用非严格模式开发。

1. 在连接 `mysql` 后, 使用 `set` 来关闭严格模式
   
    ```
    set global sql_mode='';
    ```

2. 修改 `my.cnf`, 简单

    ```vim
    [mysqld]
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
    ```

参考网址：
1. https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html#sql-mode-setting
2. https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sql-mode-setting
3. https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sql-mode-setting

