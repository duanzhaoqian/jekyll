---
layout: post
title:  "Docker Hadoop自动化构建分析"
date:   2016-04-11
desc: "数据库原理"
keywords: "Docker Hadoop自动化构建分析"
categories: [Database]
tags: [hadoop,database,docker]
icon: icon-hadoop
---

# Docker Hadoop自动化构建分析

docker hadoop 自动化

## 使用软件 

serf dnsmasq

## 说明

### serf (注册中心)

所有slave join master机器，处理 serf members 返回值到 `/etc/dnsmasq.d/0hosts`文件

```
root@master:~# serf members
slave1.kiwenlau.com  172.17.0.3:7946   alive  
slave6.kiwenlau.com  172.17.0.8:7946   alive  
slave4.kiwenlau.com  172.17.0.6:7946   alive  
slave9.kiwenlau.com  172.17.0.11:7946  alive  
slave5.kiwenlau.com  172.17.0.7:7946   alive  
slave8.kiwenlau.com  172.17.0.10:7946  alive  
master.kiwenlau.com  172.17.0.2:7946   alive  
slave2.kiwenlau.com  172.17.0.4:7946   alive  
slave3.kiwenlau.com  172.17.0.5:7946   alive  
slave7.kiwenlau.com  172.17.0.9:7946   alive
```



### dnsmasq (dns服务器)

1. 配置/etc/resolv.conf 指向dns服务器127.0.0.1
2. 利用serf解析members数据添加address 解析 hostname和ip到 /etc/dnsmasq.d/0hosts文件
3. 使用service dnsmqsq restart 重启dnsmasq使其生效

#### kiwenlau/hadoop-master各命令位置及启动顺序

1. 使用docker -w 控制工作目录在/root --dns 127.0.0.1 修改/etc/resolv.conf文件nameserver 127.0.0.1 

2. /root/start-ssh-serf.sh 启动ssh服务（所有镜像使用同一个rsa key实现免密码登录）

	```
	service ssh start
	```
	
3. 启动start-serf-agent.sh 并输出日志到serf_log

	```
	/etc/serf/start-serf-agent.sh > serf_log &
	```
	
4. 启动 dnsmqsq服务

	```
	service dnsmasq start
	```
	
5. 输出$JOIN_IP（docker 启动使用 -e JOIN_IP=  设置环境变量）到join.json
6. 启动 serf
（master机器无JOIN_IP变量，各slave机器有JOIN_IP变量）



## Docker Hadoop启动脚本备份


```bash



#!/bin/bash

# run N slave containers
N=$1

# the defaut node number is 3
if [ $# = 0 ]
then
        N=3
fi


# delete old master container and start new master container
sudo docker rm -f master &> /dev/null
echo "start master container..."
sudo docker run --net=none -p 50070:50070 -p 50010:50010 -p 9000:9000 -d -t --dns 127.0.0.1 -P --name master -h master.kiwenlau.com -w /root kiwenlau/hadoop-master:0.1.0 /bin/bash &> /dev/null
MASTER_ID=$(docker inspect -f {{.Id}} master)
MASTER_IP=192.168.88.180
pipework br0 $MASTER_ID $MASTER_IP/24
sudo docker exec master /root/start-ssh-serf.sh
# get the IP address of master container
# FIRST_IP=$(docker inspect --format="{{.NetworkSettings.IPAddress}}" master)

# delete old slave containers and start new slave containers
i=1
while [ $i -lt $N ]
do
        sudo docker rm -f slave$i &> /dev/null
        echo "start slave$i container..."
        sudo docker run --net=none -d -t --dns 127.0.0.1 -P --name slave$i -h slave$i.kiwenlau.com -e JOIN_IP=$MASTER_IP kiwenlau/hadoop-slave:0.1.0 /bin/bash &> /dev/null
        TEMP_NAME=slave$i
        echo $TEMP_NAME
        SLAVE_TMP_ID=$(docker inspect -f {{.Id}} $TEMP_NAME)
        TEMP_IP=192.168.88.18$i
        echo $TEMP_IP
        pipework br0 $SLAVE_TMP_ID $TEMP_IP/24
        sudo docker exec $TEMP_NAME /root/start-ssh-serf.sh
        ((i++))
done


# create a new Bash session in the master container
sudo docker exec -it master bash


```


