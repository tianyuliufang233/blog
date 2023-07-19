---
title: "Ab测试命令详解"
date: 2024-11-26T21:09:37+08:00
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

## apache  ab测试
|系统|安装命令|
|---|---|
|centos|yum install -y httpd-tools| 
|Debian/Ubuntu|apt install -y apache2-utils|
## 参数介绍
ab --help
```
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -q              Do not show progress when doing more than 150 requests
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don't exit on socket receive errors.
    -h              Display usage information (this message)
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol
                    (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```
翻译
```
    -n：在测试会话中所执行的请求个数。默认时，仅执行一个请求。
    -c：一次产生的请求个数。默认是一次一个。
    -t：测试所进行的最大秒数。其内部隐含值是-n 50000，它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。
    -p：包含了需要POST的数据的文件。
    -P：对一个中转代理提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。
    -T：POST数据所使用的Content-type头信息。
    -v：设置显示信息的详细程度-4或更大值会显示头信息，3或更大值可以显示响应代码(404,200等),2或更大值可以显示警告和其他信息。
    -V：显示版本号并退出。
    -w：以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。
    -i：执行HEAD请求，而不是GET。
    -x：设置<table>属性的字符串。
    -X：对请求使用代理服务器。
    -y：设置<tr>属性的字符串。
    -z：设置<td>属性的字符串。
    -C：对请求附加一个Cookie:行。其典型形式是name=value的一个参数对，此参数可以重复。
    -H：对请求附加额外的头信息。此参数的典型形式是一个有效的头信息行，其中包含了以冒号分隔的字段和值的对(如,"Accept-Encoding:zip/zop;8bit")。
    -A：对服务器提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即,是否发送了401认证需求代码)，此字符串都会被发送。
    -h：显示使用方法。
    -d：不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)。
    -e：产生一个以逗号分隔的(CSV)文件，其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用。
    -g：把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件。此文件可以方便地导入到Gnuplot,IDL,Mathematica,Igor甚至Excel中。其中的第一行为标题。
    -i：执行HEAD请求，而不是GET。
    -k：启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求。默认时，不启用KeepAlive功能。
    -q：如果处理的请求数大于150，ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。此-q标记可以抑制这些信息。
```
## 案例：  

```
# abs.exe -n 5000 -c 5000 http://localhost:1313/blog/

This is ApacheBench, Version 2.3 <$Revision: 1913912 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 500 requests
Completed 1000 requests
Completed 1500 requests
Completed 2000 requests
Completed 2500 requests
Completed 3000 requests
Completed 3500 requests
Completed 4000 requests
Completed 4500 requests
Completed 5000 requests
Finished 5000 requests


Server Software:
Server Hostname:        localhost
Server Port:            1313

Document Path:          /blog/                           #测试路由
Document Length:        22595 bytes                      #页面大小

Concurrency Level:      5000                             #测试的并发数
Time taken for tests:   4.459 seconds                    #整个测试持续的时间   
Complete requests:      5000                             #完成的请求数量
Failed requests:        0                                #失败的请求数量
Total transferred:      114760000 bytes                  #整个过程中，网络的传输量
HTML transferred:       112975000 bytes                  #html的总量
Requests per second:    1121.23 [#/sec] (mean)           #吞吐率，大家最关心的指标之一，相当于 LR 中的每秒事务数，后面括号中的 mean 表示这是一个平均值
Time per request:       4459.406 [ms] (mean)             #用户平均请求等待时间，大家最关心的指标之二，相当于 LR 中的平均事务响应时间，后面括号中的 mean 表示这是一个平均值
Time per request:       0.892 [ms] (mean, across all concurrent requests)  #每个连接请求实际运行时间的平均值
Transfer rate:          25131.22 [Kbytes/sec] received   #平均每秒网络上的流量

Connection Times (ms)                                     #络上消耗的时间的分解
              min  mean[+/-sd] median   max               #最小     平均误差     中位数    最大
Connect:        0    0   0.3      0       3               #连接时间
Processing:  1186 1534 133.6   1587    1665               #处理时间
Waiting:       14  777 300.4    805    1243               #等待时间
Total:       1186 1534 133.6   1587    1665               #总时间

Percentage of the requests served within a certain time (ms)   #在一定时间内的请求响应时间占比
  50%   1587                                                   #请求数完成一半后，统计的平均响应时间，下面的以此类推
  66%   1625
  75%   1639
  80%   1649
  90%   1659
  95%   1663
  98%   1664
  99%   1664
 100%   1665 (longest request)
```
## 测试案例
```
abs.exe -n 1 -c 1 -T "application/json" -H 'TOKEN: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9' -p post.txt https://datacenter-gateway-api.a.com/cloud-clinic/v1.0/api/noAccredit/common-home-page'
```

## 参考连接：

[ab（http）与abs（https）压测工具](https://www.cnblogs.com/weizhxa/p/8427708.html)
[ab（Apache Bench）测试工具安装与使用](https://www.cnblogs.com/hejianliang/p/13957870.html)
[Apache Bench(ab)压力测试概述-从0到1涵盖各大使用场景](https://cloud.tencent.com/developer/article/2052542)
[网站性能压力测试工具：Apache ab使用详解](https://cloud.tencent.com/developer/article/2048055)


