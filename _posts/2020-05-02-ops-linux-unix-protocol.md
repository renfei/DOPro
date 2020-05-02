---
layout: post
title:  Ops运维 - Linux/Unix 规约
date:   2020-05-02 00:00:00 +0800
categories: Ops运维篇
tag: Linux/Unix
---

* content
{:toc}

在 Linux/Unix 环境中的规约。本篇文章以CentOS(RedHat)为案例，数据库以MySQL(MariaDB)为例，其他系统例如Ubuntu、Debian等可能命令有区别，请自行Google。


Linux/Unix 通用规约			{#general}
====================================

- 在服务器上不允许使用```rm```命令，遇到确实不用的文件请使用```mv filename /tmp```的方式，将文件放入```/tmp```目录下，由系统自动删除。
- 对于没有经过验证的命令绝不允许在服务器上运行，必须完全理解该命令执行后的结果方可执行。执行命令时必须确定当前的工作目录。
- 修改配置文件必须备份，如```cp filename filename.20101010```，然后进行修改。
- 系统在投入使用前，版本应当保持最新，拿到机器先```yum update```。如果已经投入生产环境，软件包更新需要谨慎操作，将生产流量切换到其他机器，做好备份回滚的准备。部分软件包更新可能会导致不兼容的情况出现。


防火墙/端口规约			{#firewall}
====================================

【强制】防火墙必须开启。

### iptable 防火墙

```bash
# 查看防火墙状态
service iptables status
# 停止防火墙
service iptables stop
# 启动防火墙
service iptables start
# 重启防火墙
service iptables restart
# 开启80端口为例
vim /etc/sysconfig/iptables
# 加入如下代码
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
# 保存退出后重启防火墙
service iptables restart
```

### firewall 防火墙

```bash
# 查看firewall服务状态
systemctl status firewalld
# 查看firewall的状态
firewall-cmd --state
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
# 查看防火墙规则
firewall-cmd --list-all
# 查询端口是否开放
firewall-cmd --query-port=80/tcp
# 开放80端口为例
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=80/tcp
# 重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
```

### 端口最小开放原则
端口开放规则采取最小化的原则，能不开就不开，例如Web服务器对外提供HTTP服务，那应该只开放80、443端口和用于管理的22端口，其他端口应当默认关闭。

数据库服务、配置服务端口禁止公网开放。

数据库服务、配置服务的端口只允许内网访问，在任何时候禁止向公网开放端口，部分数据库默认端口如下：
> Oracle 1521  
> SQL Server 1433  
> MySQL/MariaDB 3306  
> DB2 5000  
> MongoDB 27017  
> Redis 6379  
> Memcached 11211  
> ZooKeeper 2181


账户规约			{#account}
====================================
### root账户禁用
root账户禁止远程登陆，并且在日常使用中禁止直接使用 root 账户进行操作，应当使用自建的账号登陆和操作，在必须使用高权限的时候使用 ```sudo su``` 命令使用 root，在使用完成之后应当立即执行 ```exit``` 退出 root，禁止长时间使用 root 账户操作。

添加一个用户并授予```sudo```权限，例如：
```bash
# 添加自定义用户 work
adduser work
# 为 work 指定密码
passwd test
# 安装 sudo
yum install sudo
# 编辑 sudoers 文件
vim /etc/sudoers
# 添加一行，以 work 用户为例
work ALL=(ALL)   ALL
# 保存并退出
# 禁用 root 登陆
vim  /etc/ssh/ssh_config
# 把 PermitRootLogin yes 改为 PermitRootLogin no  保存并退出
# 重启 sshd
sshd  service sshd restart
```

### SSH Key 登陆
（推荐但不强制）使用 Key 可以免去密码，使系统更加安全，但操作步奏较多，切非强制性，请自行Google。

推荐但不强制的规约			{#recommend}
====================================
- 高并发服务器建议调小 TCP 协议的 time_wait 超时时间。
> 在 linux 服务器上请通过变更/etc/sysctl.conf 文件去修改该缺省值（秒）：net.ipv4.tcp_fin_timeout = 30

- 调大服务器所支持的最大文件句柄数（File Descriptor，简写为 fd）。
> 主流操作系统的设计是将 TCP/UDP 连接采用与文件一样的方式去管理，即一个连接对应于一个 fd。主流的 linux 服务器默认所支持最大 fd 数量为 1024，当并发连接数很大时很容易因为 fd 不足而出现“open too many files”错误，导致新的连接无法建立。 建议将 linux服务器所支持的最大句柄数调高数倍（与服务器的内存数量相关）。

- 给 JVM 设置-XX:+HeapDumpOnOutOfMemoryError 参数，让 JVM 碰到 OOM 场景时输出 dump 信息。
> OOM 的发生是有概率的，甚至有规律地相隔数月才出现一例，出现时的现场信息对查错非常有价值。

- 在线上生产环境，JVM 的 Xms 和 Xmx 设置一样大小的内存容量，避免在 GC 后调整堆大小带来的压力。