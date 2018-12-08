---
title: Linux最大打开文件描述符数
date: 2018-07-26 15:11:13
tags: 
    - Linux
    - MongoDB
---
最近线上DB到下午就突然崩溃，报错: 
``` 
Too many files
```
<!--more-->
经定位，发现是由于一台机器上数据库太多，每个collection上的索引都占用open files资源个数，这些占用的连接数加起来超过系统单个进程允许的最大open files数量65535，超过的就会被系统杀掉。  
为此对open files进行一番简单的总结。  
在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。如传输控制协议 (TCP) 和用户数据报协议 (UDP)套接字等，系统在后台都为该应用程序分配了一个文件描述符，该文件描述符提供了大量关于这个应用程序本身的信息。
在进行高并发TCP连接处理时，最高的并发数量都要受到系统对用户单一进程同时可打开的文件描述符数量的限制。可使用ulimit命令查看系统允许当前用户进程打开的文件数限制：

``` bash
$ ulimit -n
```
### 系统最大打开文件描述符数：/proc/sys/fs/file-max
查看： 
``` 
$ cat /proc/sys/fs/file-max
```
临时设置：
``` 
$ echo 10000000 > /proc/sys/fs/file-max
```
永久设置: 在/etc/sysctl.conf 中设置
``` 
$ fs.file-max = 1000000
```

### 进程最大打开文件描述符数：user limit中的nofile 的soft limit
设置：  
临时性：  
通过ulimit -Sn设置最大打开文件描述符数的soft limit，注意soft limit不能大于hard limit（ulimit -Hn可查看hard limit），另外ulimit -n默认查看的是soft limit，但是ulimit -n 1800000则是同时设置soft limit和hard limit。对于非root用户只能设置比原来小的hard limit。  
查看hard limit: ulimit -Hn  
设置soft limit,必须小于hard limit:  
ulimit -Sn 1000000  
同时设置soft limit 和 hard limit  
ulimit -n 1000000  
永久性：  
上面的方法只是临时设置，注销重新登录就失效了。而且不能增大hard limit,只能在hard limit范围内修改 soft limit。若要使修改永久有效，则需要在 /etc/security/limits.conf中进行设置 [需要root权限]
可添加如下两行，表示用户chanon最大打开文件描述符的soft limit 为180000，hard limit为20000，以下设置需要注销之后重新登录才能生效：
``` 
chanon           soft    nofile          1800000
chanon           hard    nofile          2000000
```

设置nofile的hard limit还有一点要注意的就是hard limit不能大于/proc/sys/fs/nr_open，假如hard limit大于nr_open，注销后无法正常登录。可以修改nr_open的值：echo 2000000 > /proc/sys/fs/nr_open

### 查看当前系统使用的打开文件描述符数 
``` 
$ cat /proc/sys/fs/file-nr
  7968    0   1620266
```
第一个数：当前系统已分配使用的打开文件描述符数;  
第二个数：分后后已释放的 （目前不再使用的);  
第三个数：系统最大打开文件描述符数：file-max
### 总结：
a. 所有进程打开的文件描述符数不能超过 /proc/sys/fs/file-max  
b. 单个进程打开的文件描述符数不能超过user limit 中nofile的soft limit  
c. nofile的soft limit不能超过其hard limit  
d. nofile的hard limit不能超过 /proc/sys/fs/nr_open

