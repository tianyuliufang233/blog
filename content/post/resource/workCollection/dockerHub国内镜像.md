---
title: "DockerHub国内镜像"
date: 2024-08-07T14:23:55+08:00
categories:
- 资源
- 工作技巧及思路
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

# Docker Hub 国内镜像源配置
Docker Hub 国内镜像源是指在国内境内提供 Docker 镜像服务的镜像源。由于国际网络带宽等问题，国内用户下载 Docker 镜像通常速度较慢。因此，为了解决这个问题，一些国内的公司和组织提供了 Docker 镜像的国内镜像源，例如阿里云、网易云、百度云等。使用这些镜像源，国内用户可以更快地下载 Docker 镜像，提高使用体验。

## 国内有效的镜像源
以下是一些常见的 Docker 国内镜像源：  
- 网易云 Docker 镜像：http://hub-mirror.c.163.com  
- 百度云 Docker 镜像：https://mirror.baidubce.com  
- 腾讯云 Docker 镜像：https://ccr.ccs.tencentyun.com  
- Docker Proxy 镜像：https://dockerproxy.com  
- 阿里云 Docker 镜像（需要使用阿里账号自行创建专属镜像仓库）：https://cr.console.aliyun.com/  
- DaoCloud Docker 镜像（配置文档）：http://f1361db2.m.daocloud.io  

使用这些镜像源，可以加速 Docker 镜像的下载，提高使用体验。根据自己的需求和使用情况，可以选择其中一个或多个镜像源。需要注意的是，不同的镜像源可能包含的 Docker 镜像不同，因此在使用时需要注意确认所需的 Docker 镜像是否在镜像源中存在。

### 测试镜像源是否有效
docker pull hub-mirror.c.163.com/library/nginx:latest
docker pull mirror.baidubce.com/library/nginx:latest
docker pull ccr.ccs.tencentyun.com/library/nginx:latest
docker pull dockerproxy.com/library/nginx:latest
## 无效的镜像源
- Docker 官方国内镜像站：https://registry.docker-cn.com、https://reg-mirror.qiniu.com、https://dockerhub.azk8s.cn已转为私有
- 腾讯云 Docker 镜像：https://mirror.ccs.tencentyun.com/
- 中国科学技术大学镜像站 Docker 镜像源（配置文档）：https://docker.mirrors.ustc.edu.cn/
### 国内镜像站配置
#### 命令行
在使用 Docker 时，可以通过将 Docker 镜像源设置为国内镜像源来加速镜像下载。例如，在使用 Docker 命令拉取镜像时，可以使用以下命令：  
```
   docker pull 镜像名称 -–registry-mirror=国内镜像源地址
```  
其中，镜像名称 是要下载的 Docker 镜像的名称，国内镜像源地址 是要使用的国内镜像源的地址。通过这种方式，可以更快地下载 Docker 镜像。

#### 配置文件
##### Linux 
要配置 Docker 国内镜像源，可以按照以下步骤进行：  
打开 Docker 配置文件 /etc/docker/daemon.json，如果该文件不存在，则可以创建该文件。在该配置文件中添加以下内容：
```
{
    "registry-mirrors": ["https://hub-mirror.c.163.com"]
}
```
如果要使用多个镜像源，可以在 "registry-mirrors" 数组中添加多个镜像源地址，以英文逗号分隔。
保存配置文件，并重启 Docker 服务，以使配置生效。可以使用以下命令重启 Docker 服务：
```
sudo systemctl restart docker
```
如果使用的是 Ubuntu 14.04 等旧版系统，可以使用以下命令重启 Docker 服务：
```
sudo service docker restart
```
配置完成后，可以使用 docker pull 命令测试是否成功使用了国内镜像源。例如，可以使用以下命令拉取官方的 Ubuntu 镜像：
```
docker pull ubuntu
```
如果使用了正确的国内镜像源地址，镜像的下载速度应该比官方源快很多。  
注意：如果在 Docker Desktop for Mac 或 Docker Desktop for Windows 中使用 Docker，可以在 Docker Desktop 的设置中进行镜像加速器的配置，不需要手动编辑配置文件。  
##### Windows/Mac  
以在 Windows 上配置 Docker 国内镜像为例，按照以下步骤进行：  
打开 Docker 设置。可以在任务栏右下角找到 Docker 图标，右键单击该图标，然后选择“Settings”打开设置。在设置界面中，选择“Docker Engine”选项卡，在该选项卡中找到“registry-mirrors”一栏。在“registry-mirrors”一栏中，输入要使用的国内镜像源的地址，例如：
```
    https://registry.docker-cn.com
```
点击“Apply & restart”保存设置，并等待 Docker 服务重启。配置完成后，可以使用 docker pull 命令测试是否成功使用了国内镜像源。例如，可以使用以下命令拉取官方的 Ubuntu 镜像：
```
docker pull ubuntu
```
如果使用了正确的国内镜像源地址，镜像的下载速度应该比官方源快很多。  
在 Mac 上的配置类似。  
##### 注意：
在 Docker Desktop for Windows 中，还需要确保 Docker Daemon 正在运行，并已经启用了“Expose daemon on tcp://localhost:2375 without TLS”选项。可以在 Docker 设置的“General”选项卡中勾选该选项。


## 参考

[Docker国内镜像源列表](https://blog.csdn.net/sxf1061700625/article/details/140895299)