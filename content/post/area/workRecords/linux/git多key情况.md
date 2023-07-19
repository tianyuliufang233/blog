---
title: "Git多key情况"
date: 2024-07-25T00:04:42+08:00
categories:
- 收件箱
- linxu相关
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## ~/.ssh/config 文件

配置如下：
```
Host 192.168.1.4                     # 指定主机名，域名，ip不能加端口
    HostName 192.168.1.4             # 指定主机名，域名，ip不能加端口
    User git                         # 指定用户
    Port 2222                        # 指定端口
    IdentityFile ~/.ssh/id_rsa_home  # 此语句指定key位置
    PubkeyAcceptedKeyTypes +ssh-rsa  # 此语句解决因加密方式不支持导致的ssh连接问题，指定ssh-rsa方式加密
```

注意：

文件名为config，权限为644及更低。