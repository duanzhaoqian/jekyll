---
layout: post
title:  "linux syslog"
date:   2016-09-11
desc: "linux syslog"
keywords: "linux syslog"
categories: [Linux]
tags: [linux,syslog]
icon: fa-linux
---


# 2016-09-11-linux-syslog

<!--
create time: 2016-09-11 09:54:44
Author: 段朝骞
Email: duanzhaoqian@duanzhaoqian.com

categories:[Linux] [Database]  [Java] [HTML]  [Mac] [Life]
icon:fa-linux fa-database icon-java fa-apple

icon http://fizzed.com/oss/font-mfizz
icon http://fontawesome.io/icons/
-->

[http://ant595.blog.51cto.com/5074217/1080922](http://ant595.blog.51cto.com/5074217/1080922)

## syslog简介

syslog 系统日志,记录linux系统启动及运行的过程中产生的信息,rhel5.x系统上默认自带了syslog 其配置文件是/etc/syslog.conf

syslog 默认有两个守护进程,klogd,syslogd,

klogd 进程是记录系统运行的过程中内核生成的日志,而在系统启动的过程中内核初始化过程中 生成的信息记录到控制台(/dev/console）当系统启动完成之后会把此信息存放到/var/log/dmesg文件中,我可以通过cat /var/log/dmesg查看这个文件,也可以通过dmesg命令来查看

syslogd 进程是记录非内核以外的信息
而为什么需要两个守护进程呢?是因为内核跟其他信息需要记录的详细程度及格式的不同
我们使用ps命令可以看到syslog的两个守护进程

```
ps -ef | grep klogd | grep -v grep 
root      3308     1  0 Nov26 ?        00:00:00 klogd -x 
 
ps -ef | grep syslogd | grep -v grep  
root      3288     1  0 Nov26 ?        00:00:00 syslogd -m 0 
```

上面通过ps命令可以看到syslog的两个守护进程,而这两个守护进程是共用一个配置文件/etc/syslog.conf,下面介绍下其配置文件

## syslog配置文件详解

```
配置文件定义格式为 facility.priority   action 
 facility,可以理解为日志的来源或设备目前常用的facility有以下及中 
    1,auth      # 认证相关的 
    2,authpriv  # 权限,授权相关的 
    3,cron      # 任务计划相关的 
    4,daemon    # 守护进程相关的 
    5,kern      # 内核相关的 
    6,lpr       # 打印相关的 
    7,mail      # 邮件相关的 
    8,mark      # 标记相关的 
    9,news      # 新闻相关的 
    10,security # 安全相关的,与auth 类似  
    11,syslog   # syslog自己的 
    12,user     # 用户相关的 
    13,uucp     # unix to unix cp 相关的 
    14,local0 到 local7 # 用户自定义使用 
    15,*        # *表示所有的facility 
    等..... 
 
 priority(log level)日志的级别,一般有以下几种级别(从低到高) 
    debug           # 程序或系统的调试信息 
    info            # 一般信息, 
    notice          # 不影响正常功能,需要注意的消息 
    warning/warn    # 可能影响系统功能,需要提醒用户的重要事件 
    err/error       # 错误信息 
    crit            # 比较严重的 
    alert           # 必须马上处理的 
    emerg/oanic     # 会导致系统不可用的 
    *               # 表示所有的日志级别 
    none            # 跟* 相反,表示啥也没有 
     
 action(动作)日志记录的位置 
    系统上的绝对路径    # 普通文件 如： /var/log/xxx 
    |                   # 管道  通过管道送给其他的命令处理 
    终端              # 终端   如：/dev/console 
    @HOST               # 远程主机 如： @10.0.0.1      
    用户              # 系统用户 如： root 
    *                   # 登录到系统上的所有用户，一般emerg级别的日志是这样定义的 
```

```
定义格式例子： 
mail.info   /var/log/mail.log # 表示将mail相关的,级别为info及 
                              # info以上级别的信息记录到/var/log/mail.log文件中 
auth.=info  @10.0.0.1         # 表示将auth相关的,基本为info的信息记录到10.0.0.1主机上去 
                              # 前提是10.0.0.1要能接收其他主机发来的日志信息 
user.!=error                  # 表示记录user相关的,不包括error级别的信息 
user.!error                   # 与user.error相反 
*.info                        # 表示记录所有的日志信息的info级别 
mail.*                        # 表示记录mail相关的所有级别的信息 
*.*                           # 你懂的. 
cron.info;mail.info           # 多个日志来源可以用";" 隔开 
cron,mail.info                # 与cron.info;mail.info 是一个意思 
mail.*;mail.!=info            # 表示记录mail相关的所有级别的信息,但是不包括info级别的 
```

```
接下来去翻译下rhel5.x系统上自带的syslog的配置文件/etc/syslog.conf 
# 表示将所有facility的info级别,但不包括mail,authpriv,cron相关的信息,记录到 /var/log/messages文件 
*.info;mail.none;authpriv.none;cron.none                /var/log/messages 
 
# 表示将权限,授权相关的所有基本的信息,记录到/var/log/secure文件中.这个文件的权限是600 
authpriv.*                                              /var/log/secure 
 
# 表示将mail相关的所有基本的信息记录到/var/log/maillog文件中,可以看到路径前面有一个"-" 
# "-" 表示异步写入磁盘, 
mail.*                                                  -/var/log/maillog 
 
# 表示将任务计划相关的所有级别的信息记录到/var/log/cron文件中 
cron.*                                                  /var/log/cron 
 
# 表示将所有facility的emerg级别的信息,发送给登录到系统上的所有用户 
*.emerg                                                 * 
 
# 表示将uucp及news的crit级别的信息记录到/var/log/spooler文件中 
uucp,news.crit                                          /var/log/spooler 
 
# 表示将local7的所有级别的信息记录到/var/log/boot.log文件中, 
# 上面说过local0 到local7这8个是用户自定义使用的,这里的local7记录的是系统启动相关的信息 
local7.*                                                /var/log/boot.log 
```

syslog默认记录的日志格式有四个字段,时间标签 主机 子系统名称 消息

可以使用tail  /var/log/messages 看下

## syslog-ng简介

syslog-ng (syslog-Next generation) 是syslog的升级版,syslog-ng有两个版本,一个是收费的,一个是开源的，那么作为syslog的下一代产品,功能是可想而知,肯定比syslog的功能强大的多,如

>高性能		
>
>可靠的传输 	
>
>支持多平台		
>
>高可靠性	
>
>众多的用户群体		
>
>强大的日志过滤及排序		
>
>事件标签和关联性		
>
>支持最新的IETF标准		
>
>等....

开源版本的主页 http://www.balabit.com/network-security/syslog-ng/opensource-logging-system/overview

## syslog-ng的安装

rhel5.x的系统上默认没有使用syslog-ng来记录日志的,需要使用的话,需要自己编译安装,安装方法如下

```
yum install gcc*  
cd /usr/src 
wget http://www.balabit.com/downloads/files/syslog-ng/sources/3.2.4/source/eventlog_0.2.12.tar.gz 
wget http://www.balabit.com/downloads/files/syslog-ng/open-source-edition/3.3.5/source/syslog-ng_3.3.5.tar.gz 
tar xvf eventlog_0.2.12.tar.gz 
cd eventlog-0.2.12 
./configure --prefix=/usr/local/eventlog 
make 
make install 
 
cd /usr/src 
tar xvf syslog-ng_3.3.5.tar.gz 
cd syslog-ng-3.3.5 
export PKG_CONFIG_PATH=/usr/local/eventlog/lib/pkgconfig 
./configure --prefix=/usr/local/syslog-ng 
make 
make install 
 
 
将syslog-ng添加为系统服务, 
vim /etc/init.d/syslog-ng  #内容如下 
#!/bin/bash 
#  
# chkconfig: -  60 27 
# description: syslog-ng SysV script.  
. /etc/rc.d/init.d/functions 
 
syslog_ng=/usr/local/syslog-ng/sbin/syslog-ng 
prog=syslog-ng 
pidfile=/usr/local/syslog-ng/var/syslog-ng.pid 
lockfile=/usr/local/syslog-ng/var/syslog-ng.lock 
RETVAL=0 
STOP_TIMEOUT=${STOP_TIMEOUT-10} 
 
start() { 
        echo -n $"Starting $prog: " 
        daemon --pidfile=$pidfile $syslog_ng $OPTIONS 
        RETVAL=$? 
        echo 
        [ $RETVAL = 0 ] && touch ${lockfile} 
        return $RETVAL 
} 
 
stop() { 
    echo -n $"Stopping $prog: " 
    killproc -p $pidfile -d $STOP_TIMEOUT $syslog_ng 
    RETVAL=$? 
    echo 
    [ $RETVAL = 0 ] && rm -f $lockfile $pidfile 
} 
 
case "$1" in 
  start) 
    start 
    ;; 
  stop) 
    stop 
    ;; 
  status) 
        status -p $pidfile $syslog_ng 
    RETVAL=$? 
    ;; 
  restart) 
    stop 
    start 
    ;; 
  *) 
    echo $"Usage: $prog {start|stop|restart|status}" 
    RETVAL=2 
esac 
exit $RETVAL 
------------------------------------------------------------ 
chmod a+x /etc/init.d/syslog-ng 
killall syslogd 
chkconfig --add syslog-ng 
chkconfig syslog-ng on 
service syslog-ng start 
```

## syslog-ng配置文件详解

此时syslog-ng服务已经启动起来了,配置文件的位置在安装目录下的etc/syslog-ng.conf

```
syslog-ng.conf文件里的内容有以下几个部分组成, 
# 全局选项,多个选项时用分好";"隔开 
options { .... }; 
# 定义日志源, 
source s_name { ... }; 
# 定义过滤规则，规则可以使用正则表达式来定义,这里是可选的,不定义也没关系 
filter f_name { ... }; 
# 定义目标 
destination d_name { ... }; 
# 定义消息链可以将多个源,多个过滤规则及多个目标定义为一条链 
log { ... }; 
详解如下 
---------------------------------------------------------------- 
options { long_hostnames(off); sync(0); perm(0640); stats(3600); }; 
    更多选项如下 
    chain_hostnames(yes|no)     # 是否打开主机名链功能，打开后可在多网络段转发日志时有效 
    long_hostnames(yes|no)      # 是chain_hostnames的别名，已不建议使用 
    keep_hostname(yes|no)       # 是否保留日志消息中保存的主机名称 
    use_dns(yes|no)             # 是否打开DNS查询功能， 
    use_fqdn(yes|no)            # 是否使用完整的域名 
    check_hostname(yes|no)      # 是否检查主机名有没有包含不合法的字符 
    bad_hostname(regexp)        # 可通过正规表达式指定某主机的信息不被接受 
    dns_cache(yes|no)           # 是否打开DNS缓存功能 
    dns_cache_expire(n)         # DNS缓存功能打开时，一个成功缓存的过期时间 
    dns_cache_expire_failed(n)  # DNS缓存功能打开时，一个失败缓存的过期时间 
    dns_cache_size(n)           # DNS缓存保留的主机名数量 
    create_dirs(yes|no)         # 当指定的目标目录不存在时，是否创建该目录 
    dir_owner(uid)              # 目录的UID 
    dir_group(gid)              # 目录的GID 
    dir_perm(perm)              # 目录的权限，使用八进制方式标注，例如0644 
    owner(uid)                  # 文件的UID 
    group(gid)                  # 文件的GID 
    perm(perm)                  # 文件的权限，同样，使用八进制方式标注 
    gc_busy_threshold(n)        # 当syslog-ng忙时，其进入垃圾信息收集状态的时间一旦分派的对象达到这个数字，syslog-ng就启动垃圾信息收集状态。默认值是：3000。 
    gc_idle_threshold(n)        # 当syslog-ng空闲时，其进入垃圾信息收集状态的时间一旦被分派的对象到达这个数字，syslog-ng就会启动垃圾信息收集状态，默认值是：100 
    log_fifo_size(n)            # 输出队列的行数 
    log_msg_size(n)             # 消息日志的最大值（bytes） 
    mark(n)                     # 多少时间（秒）写入两行MARK信息供参考，目前没有实现 
    stats(n)                    # 多少时间（秒）写入两行STATUS信息,默认值是：600 
    sync(n)                     # 缓存多少行的信息再写入文件中，0为不缓存，局部参数可以覆盖该值。 
    time_reap(n)                # 在没有消息前，到达多少秒，即关闭该文件的连接 
    time_reopen(n)              # 对于死连接，到达多少秒，会重新连接 
    use_time_recvd(yes|no)      # 宏产生的时间是使用接受到的时间，还是日志中记录的时间；建议使用R_的宏代替接收时间，S_的宏代替日志记录的时间，而不要依靠该值定义。 
 
source s_name { internal(); unix-dgram("/dev/log"); udp(ip("0.0.0.0") port(514)); }; 
 
    file (filename)                 # 从指定的文件读取日志信息 
    unix-dgram  (filename)          # 打开指定的SOCK_DGRAM模式的unix套接字，接收日志消息 
    unix-stream (filename)          # 打开指定的SOCK_STREAM模式的unix套接字，接收日志消息 
    udp ( (ip),(port) )             # 在指定的UDP端口接收日志消息 
    tcp ( (ip),(port) )             # 在指定的TCP端口接收日志消息 
    sun-streams (filename)          # 在solaris系统中，打开一个（多个）指定的STREAM设备，从其中读取日志消息 
    internal()                      # syslog-ng内部产生的消息 
    pipe(filename),fifo(filename)   # 从指定的管道或者FIFO设备，读取日志信息 
 
filter f_name   { not facility(news, mail) and not filter(f_iptables); }; 
    更多规则函数如下 
    facility(..)    # 根据facility（设备）选择日志消息，使用逗号分割多个facility 
    level(..)       # 根据level（优先级）选择日志消息，使用逗号分割多个level，或使用“..”表示一个范围 
    program(表达式)    # 日志消息的程序名是否匹配一个正则表达式 
    host(表达式)   # 日志消息的主机名是否和一个正则表达式匹配 
    match(表达式)  # 对日志消息的内容进行正则匹配 
    filter()        # 调用另一条过滤规则并判断它的值 
    定义规则的时候也可以使用逻辑运算符and or not 
 
destination d_name { file("/var/log/messages"); }; 
    更多动作如下 
    file (filename)                 # 把日志消息写入指定的文件 
    unix-dgram  (filename)          # 把日志消息写入指定的SOCK_DGRAM模式的unix套接字 
    unix-stream (filename)          # 把日志消息写入指定的SOCK_STREAM模式的unix套接字 
    udp (ip),(port)                 # 把日志消息发送到指定的UDP端口 
    tcp (ip),(port)                 # 把日志消息发送到指定的TCP端口 
    usertty(username)               # 把日志消息发送到已经登陆的指定用户终端窗口 
    pipe(filename),fifo(filename)   # 把日志消息发送到指定的管道或者FIFO设备 
    program(parm)                   # 启动指定的程序，并把日志消息发送到该进程的标准输入 
 
log { source(s_name); filter(f_name); destination(d_name) }; 
```

一条日志的处理流程大概是这样的,如下

首先是	 "日志的来源 	source s_name { ... };"

然后是	 "过滤规则 	filter f_name { ... };"

再然后是 "消息链 	log { source(s_name); filter(f_name); 
destination(d_name) };"

最后是	 "目标动作 	destination d_name { ... };"

这样以来一条日志就根据你的意思来处理了,需要注意的是一条日志消息过了之后,会匹配定义的所有配置,并不是匹配到以后就不再往下匹配了.

## syslog-ng配置文件例子

```
$syslog-ng_path/etc/syslog-ng.conf 内容如下 
 
options { long_hostnames(off); sync(0); perm(0640); stats(3600); }; 
 
source src {  
            internal();  
            unix-dgram("/dev/log");  
            # 表示日志来源为本机udp的514端口, 
            udp(ip("0.0.0.0") port(514));  
}; 
 
filter f_iptables   { facility(kern) and match("IN=") and match("OUT="); }; 
 
filter f_console    { level(warn) and facility(kern) and not filter(f_iptables) 
                      or level(err) and not facility(authpriv); }; 
 
filter f_newsnotice { level(notice) and facility(news); }; 
filter f_newscrit   { level(crit)   and facility(news); }; 
filter f_newserr    { level(err)    and facility(news); }; 
filter f_news       { facility(news); }; 
 
filter f_mailinfo   { level(info)      and facility(mail); }; 
filter f_mailwarn   { level(warn)      and facility(mail); }; 
filter f_mailerr    { level(err, crit) and facility(mail); }; 
filter f_mail       { facility(mail); }; 
 
filter f_cron       { facility(cron); }; 
 
filter f_local      { facility(local0, local1, local2, local3, 
                               local4,  local6, local7); }; 
 
filter f_acpid_full { match('^acpid:'); }; 
filter f_acpid      { level(emerg..notice) and match('^acpid:'); }; 
 
filter f_acpid_old  { match('^\[acpid\]:'); }; 
 
filter f_netmgm     { match('^NetworkManager:'); }; 
 
filter f_messages   { not facility(news, mail) and not filter(f_iptables); }; 
filter f_warn       { level(warn, err, crit) and not filter(f_iptables); }; 
filter f_alert      { level(alert); }; 
 
destination console  { pipe("/dev/tty10"    owner(-1) group(-1) perm(-1)); }; 
log { source(src); filter(f_console); destination(console); }; 
 
destination xconsole { pipe("/dev/xconsole" owner(-1) group(-1) perm(-1)); }; 
log { source(src); filter(f_console); destination(xconsole); }; 
 
destination newscrit   { file("/var/log/news/news.crit" 
                              owner(news) group(news)); }; 
log { source(src); filter(f_newscrit); destination(newscrit); }; 
 
destination newserr    { file("/var/log/news/news.err" 
                              owner(news) group(news)); }; 
log { source(src); filter(f_newserr); destination(newserr); }; 
 
destination newsnotice { file("/var/log/news/news.notice" 
                              owner(news) group(news)); }; 
log { source(src); filter(f_newsnotice); destination(newsnotice); }; 
 
destination mailinfo { file("/var/log/mail.info"); }; 
log { source(src); filter(f_mailinfo); destination(mailinfo); }; 
 
destination mailwarn { file("/var/log/mail.warn"); }; 
log { source(src); filter(f_mailwarn); destination(mailwarn); }; 
 
destination mailerr  { file("/var/log/mail.err" fsync(yes)); }; 
log { source(src); filter(f_mailerr);  destination(mailerr); }; 
 
destination mail { file("/var/log/mail"); }; 
log { source(src); filter(f_mail); destination(mail); }; 
 
destination acpid { file("/var/log/acpid"); }; 
destination null { }; 
log { source(src); filter(f_acpid); destination(acpid); flags(final); }; 
 
log { source(src); filter(f_acpid_full); destination(null); flags(final); }; 
 
log { source(src); filter(f_acpid_old); destination(acpid); flags(final); }; 
 
destination netmgm { file("/var/log/NetworkManager"); }; 
log { source(src); filter(f_netmgm); destination(netmgm); flags(final); }; 
 
destination localmessages { file("/var/log/localmessages"); }; 
log { source(src); filter(f_local); destination(localmessages); }; 
 
destination messages { file("/var/log/messages"); }; 
log { source(src); filter(f_messages); destination(messages); }; 
 
destination firewall { file("/var/log/firewall"); }; 
log { source(src); filter(f_iptables); destination(firewall); }; 
 
destination warn { file("/var/log/warn" fsync(yes)); }; 
log { source(src); filter(f_warn); destination(warn); }; 
 
filter f_ha         { facility(local5); }; 
destination hamessages { file(/var/log/ha); }; 
log { source(src); filter(f_ha); destination(hamessages); }; 
```