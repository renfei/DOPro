---
layout: post
title:  Ops运维 - 数据库规约
date:   2020-05-02 00:00:00 +0800
categories: Ops运维篇
tag: 数据库规约
---

* content
{:toc}

这里约定与数据库相关的规定


MySQL/MariaDB 规约			{#mysql}
====================================
### 账号规约
- 禁止直接使用root账号

> root账号只用于本机(localhost)管理，在程序配置连接中也禁止直接使用root账号。应当为每个角色创建它自己的账号和密码，例如程序配置中使用的是账户A，运维人员使用的是账户B。

```bash
# 连接本机数据库
mysql -uroot -p
# 查看访问权限列表，%是通配符代表任意内容
use mysql;
select user,host from user;
# 关闭 root 远程访问权限
use mysql;
update user set host = "localhost" where user = "root" and host = "%";
# 刷新
flush privileges;
```

- 账号最小权限原则

新建的账号授权时应当遵照最小权限原则，只授予他需要访问的数据库，如果应用只读取不写入，那分配的账号应该是只取权限；同理如果应用只写数据不查询，那不应该分配查询权限。

### 删除操作规约
尽量不使用硬删除，最好使用逻辑软删除。

删除命令```必须带 where 条件```，哪怕是 where 1=1。

执行delete命令前，```必须先执行select命令```，查看将要删除的数据是否是你期望删除的。
