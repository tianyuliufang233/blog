---
title: "Java排查cpu过高问题"
date: 2024-08-27T19:16:59+08:00
categories:
- 收件箱
- 记录
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

## 发现有进程长期99%的cpu占用
通过top命令查看 发现是java进程  
通过top -H -p <pid>  查看发现XNIO-1 I/O-1占用过高  
通过搜索引擎搜索相关问题，发现配置undertow线程数解决。  

## 通过jstack 排查cpu占用过高问题
在容器中执行ps -aux命令查看进程id
再使用top -H -p <pid> 查看线程cpu占用最高的TID
printf "0x%x\n" <线程TID>  转换为十六进制 
通过jstack <pid> |grep 上一步转换的十六进制TID 得到线程情况
使用jmap -dump:live,format=b,file=heap,hprof 进程号
使用memoryAnalyzer对jmap dump出来的文件进行分析，找到XNIO-1 I/O-1相关线程
通过搜索引擎搜索到的信息进行比对，加入新配置进行尝试。


## 参考
https://blog.csdn.net/yinni11/article/details/89395768  
https://www.cnblogs.com/xiedy001/p/16825964.html  
https://cn.bing.com/search?q=XNIO-1+I%2FO-1+%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98&qs=n&form=QBRE&sp=-1&lq=0&pq=xnio-1+i%2Fo-1+%E5%8D%A0%E7%94%A8guo%27gao&sc=10-22&sk=&cvid=471B7F1893CF420A9178DF6E803E976D&ghsh=0&ghacc=0&ghpl=  
https://www.cnblogs.com/rinack/p/12969851.html  
https://blog.csdn.net/u014163312/article/details/124248558  
https://cloud.tencent.com/developer/article/2351448  
https://cn.bing.com/search?q=java+cpu+%E5%8D%A0%E7%94%A8%E9%AB%98+%E6%8E%92%E6%9F%A5&form=ANNTH1&refig=66cd9eee62aa43cc87806a4ab15e8963&pc=NMTS&pqlth=5&assgl=15&sgcn=java+cpu+%E5%8D%A0%E7%94%A8%E9%AB%98+%E6%8E%92%E6%9F%A5&smvpcn=0&swbcn=10&cvid=66cd9eee62aa43cc87806a4ab15e8963&kpratsg=1&hsmssg=0  
https://blog.csdn.net/u014163312/article/details/124248558  
https://www.cnblogs.com/nuccch/p/18246905  

https://blog.csdn.net/csdn_tiger1993/article/details/127493812  

