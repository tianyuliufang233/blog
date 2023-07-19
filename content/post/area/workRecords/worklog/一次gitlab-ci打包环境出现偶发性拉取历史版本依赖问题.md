---
title: "一次gitlab Ci打包maven环境出现偶发性拉取历史版本依赖问题"
date: 2024-07-29T17:06:33+08:00
categories:
- 收件箱
- 工作记录
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## 问题现象： gitlab-ci打包maven环境偶尔拉取历史版本依赖

## 解题思路：
1. 线上服务无法启动，但本地服务能够正常启动，开发根据日志判断为部分依赖未更新导致。
2. 导出容器镜像，导出生成的jar文件后解压，发现依赖未更新，通过文心一言对maven没有更新依赖的可能逻辑进行列举，判断缓存的可能性极大。
3. 分析打包命令后，发现mvn命令参数中有“-U”参数，这是一个强制更新参数，此时问题导向其他方向，经过搜索，发现这个参数只针对快照强制更新。
4. 再次分析各个可能性，判断仍然是缓存可能性更大，搜索“maven 的缓存机制”，首次的“[日常遇到maven出现依赖版本/缓存问题通用思路](https://blog.csdn.net/acwing/article/details/136199811)”的有提到settings.xml，但因为不熟悉，只有另外搜索。
5. 按照表象“upload后maven-release仓库不会及时更新”进行搜索，发现“[maven release版本不自动更新的原因](https://www.cnblogs.com/lnlvinso/p/10046763.html)”有提到更新策略，其中daily为默认项，即每天更新，也就会导致有时候会发现无法更新，除非删除本地仓库中的版本。

## 反思：
是否应该先分析成体系的可能性？然后直接找到更系统的信息源？

## 参考：
[maven release版本不自动更新的原因](https://www.cnblogs.com/lnlvinso/p/10046763.html)
