---
title: "Prometheusalert规则编辑"
date: 2023-09-26T15:57:26+08:00
categories:
- 收件箱
- linxu相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

## prometheus-alert 模板
```
{{ $var := .externalURL}}{{ range $k,$v:=.alerts }}{{if eq $v.status "resolved"}}[PROMETHEUS-恢复信息]({{$v.generatorURL}})
> **[{{$v.labels.alertname}}]({{$var}})**
> <font color="info">告警级别:</font> {{$v.labels.severity}}
> <font color="info">开始时间:</font> {{$v.startsAt}}
> <font color="info">结束时间:</font> {{$v.endsAt}}
> <font color="info">故障服务:</font> {{$v.labels.job}}
> <font color="info">告警状态:</font> {{$v.status}}
> <font color="info">**{{$v.annotations.description}}**</font>{{else}}[PROMETHEUS-告警信息]({{$v.generatorURL}})
> **[{{$v.labels.alertname}}]({{$var}})**
> <font color="warning">告警级别:</font> {{$v.labels.severity}}
> <font color="warning">开始时间:</font> {{$v.startsAt}}
> <font color="warning">故障服务:</font> {{$v.labels.job}}
> <font color="warning">告警状态:</font> {{$v.status}}
> <font color="warning">**{{$v.annotations.description}}**</font>{{end}}{{ end }}
{{ $urimsg:=""}}{{ range $key,$value:=.commonLabels }}{{$urimsg =  print $urimsg $key "%3D%22" $value "%22%2C" }}{{end}}[*** 点我屏蔽该告警]({{$var}}/#/silences/new?filter=%7B{{SplitString $urimsg 0 -3}}%7D)
```