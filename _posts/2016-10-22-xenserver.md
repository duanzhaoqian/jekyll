---
layout: post
title:  "xenserver 6.5"
date:   2016-10-22 09:30:41
desc: "xenserver 6.5"
keywords: "xenserver 6.5"
categories: [Linux]
tags: [linux]
icon: fa-linux
---


# 2016-10-22-xenserver

<!--
create time: 2016-10-22 09:30:41
Author: 段朝骞
Email: duanzhaoqian@duanzhaoqian.com

categories:[Linux] [Database]  [Java] [HTML]  [Mac] [Life]
icon:fa-linux fa-database icon-java fa-apple

icon http://fizzed.com/oss/font-mfizz
icon http://fontawesome.io/icons/
-->

## 设置VM通电自动启动

[http://support.citrix.com/article/CTX133910](http://support.citrix.com/article/CTX133910)

Setting the XenServer to allow Auto-Start

```
xe pool-list
xe pool-param-set uuid=UUID other-config:auto_poweron=true
```

Setting the Virtual Machines to Auto-Start

```
xe vm-list
xe vm-param-set uuid=UUID other-config:auto_poweron=true
```

设置所有的虚拟机开机自动启动

```
for i in `xe vm-list params=uuid --minimal | sed 's/,/ /g'`;do xe vm-param-set uuid=$i other-config:auto_poweron=true;done
```