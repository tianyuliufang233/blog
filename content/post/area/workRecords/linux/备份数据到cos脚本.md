---
title: "备份数据到cos脚本"
date: 2023-10-24T10:23:01+08:00
categories:
- 收件箱
- linxu相关
tags:
- shell
- 案例
keywords:
- 脚本
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
执行之前需要预先配置好coscli登陆认证。
```
#!/bin/bash
set -u
export PATH=$PATH:/usr/sbin/
CONTAINER_ARRAY=(docker-name docker-name-1)
DATATIME=`date -d "1 day ago" +"%Y-%m-%d"`
BACKUP_AUDIT_LOCAL_IP=`ip addr|grep eth0|grep inet|awk -F ' ' '{print $2}'|cut -d '/' -f 1`
BACKUP_SHELL_LOG="/data/apps/backup/backup.log"
LOGDIR=/data/logs/docker

function log(){
echo "[$(date +"%Y/%m/%dT%H:%M:%S")] $*" >> ${BACKUP_SHELL_LOG}
}


# backup all log for audit, include audit dir
log "============= starting to backup all data for datacenter  ====================="
if [ ! -d "${LOGDIR}/logs" ];
then
mkdir -p ${LOGDIR}/logs/
fi
if [ ! -d "${LOGDIR}/upload" ];
then
mkdir -p ${LOGDIR}/upload/
fi
for e in ${CONTAINER_ARRAY[@]} ;
do
log "start dump container ${e} logs to local"
docker logs --since="${DATATIME}T00:00:00" --until "${DATATIME}T23:59:59" ${e} >  ${LOGDIR}/logs/${e}.log 2>&1
zip ${LOGDIR}/upload/${e}${DATATIME}.zip ${LOGDIR}/logs/${e}.log
done
if [ -d "${LOGDIR}/logs" ];
then
rm -rf ${LOGDIR}/logs
fi
log "============= finished to backup all data for datacenter  ====================="

# upload the backup data of auditd to COS storage by coscli command
log "============= starting to upload all backup data to COS on qcloud  ====================="
/usr/local/bin/coscli sync -r --rate-limiting=4 --storage-class ARCHIVE ${LOGDIR}/upload cos://cosname/linux/${BACKUP_AUDIT_LOCAL_IP}/datacenter-master/  >> ${BACKUP_SHELL_LOG}
if [ -d "${LOGDIR}/upload" ];
then
rm -rf ${LOGDIR}/upload
fi
log "============= finished to upload all backup data to COS on qcloud  ====================="
```