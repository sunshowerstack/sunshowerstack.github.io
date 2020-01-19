---
title: docker方式安装airflow
date: 2019-08-10 13:30:09
categories:
- schedule
tags:
- airflow
- docker
---



# 1.安装方式



## 1.1 直接现成镜像pull方式。

~~~
$sudo docker pull puckel/docker-airflow
~~~



~~~
$ sudo docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
puckel/docker-airflow   latest              6bf739d22d98        2 days ago          744MB
~~~



## 1.2 自己build生成镜像方式。

~~~shell
# 安装git
$yum -y install git
$git clone https://github.com/puckel/docker-airflow.git

$cd docker-airflow
~~~



~~~shell
docker build --rm --build-arg AIRFLOW_DEPS="datadog,dask" -t puckel/docker-airflow .
docker build --rm --build-arg PYTHON_DEPS="flask_oauthlib>=0.9" -t puckel/docker-airflow .
~~~



# 2.运行方式

> 执行器-Executor

执行器有 SequentialExecutor, LocalExecutor, CeleryExecutor

SequentialExecutor 为顺序执行器，默认使用 sqlite 作为知识库，由于 sqlite 数据库的原因，任务之间不支持并发执行，常用于测试环境，无需要额外配置。

LocalExecutor 为本执行器，不能使用 sqlite 作为知识库，可以使用 mysql,postgress,db2,oracle 等各种主流数据库，任务之间支持并发执行，常用于生产环境，需要配置数据库连接 url。

CeleryExecutor 为 Celery 执行器，需要安装 Celery ,Celery 是基于消息队列的分布式异步任务调度工具。需要额外启动工作节点-worker。使用 CeleryExecutor 可将作业运行在远程节点上。



## SequentialExecutor方式

~~~shell
sudo docker run -d -p 8082:8080 puckel/docker-airflow webserver

# 工作目录挂载host方式 -v位置不能乱放 宿主机:container 只要修改宿主机的文件即可
sudo docker run -d -p 8082:8080 \
-v /usr/local/airflow/dags:/usr/local/airflow/dags \
-v /usr/local/airflow/airflow.cfg:/usr/local/airflow/airflow.cfg \
--name airflow puckel/docker-airflow webserver 

# airflow.cfg也外挂的原因是要修改时区
#(修改前)default_timezone = utc
#(修改后)default_timezone = Asia/Shanghai
~~~

运行后可以

登录ui

~~~
http://xxx:8082/admin      
~~~

`注意::  小机上内存太小很快就会被杀掉进程,根本跑不起来`



虚拟机上可以看到用掉357.8M内存

~~~shell
# docker stats
CONTAINER ID        NAME                  CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
73ab2ccadcaf        xenodochial_taussig   2.59%               357.8MiB / 3.686GiB   9.48%               274kB / 2.95MB      2.24MB / 16.6MB     19

~~~





build方式安装的镜像会有以下错误

~~~
[centos@ip-172-26-6-221 docker-airflow-master]$ sudo docker run -d -p 8082:8080 puckel/docker-airflow webserver
dbb065881f8c5859245f28f18bd78c00da55459d2276c2af545a78fb2964a50c
docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "exec: \"/entrypoint.sh\": permission denied": unknown.

~~~

~~~

sudo chmod +x script/entrypoint.sh
~~~





## LocalExecutor方式



安装compose

```
$sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$sudo chmod +x /usr/local/bin/docker-compose
#查看安装版本
$ docker-compose --version
docker-compose version 1.24.0, build 0aa59064
```



~~~
$ sudo /usr/local/bin/docker-compose  -f docker-compose-LocalExecutor.yml up -d
~~~





# 国内虚拟机centos安装



~~~
[root@localhost docker-airflow-master]# docker build --rm --build-arg AIRFLOW_DEPS="datadog,dask" -t puckel2/docker-airflow .
Sending build context to Docker daemon  70.14kB
Step 1/23 : FROM python:3.7-slim-stretch
Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on 192.168.23.1:53: read udp 192.168.23.130:40278->192.168.23.1:53: i/o timeout

~~~



## 国内环境必须要设置镜像加速,否则根本安装不上



~~~shell
cd /etc/docker/
touch daemon.json
vi daemon.json
#增加
{
"registry-mirrors": [
    "https://bhu1x6ya.mirror.aliyuncs.com",
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://registry.docker-cn.com"
  ]
}
[root@localhost docker]# systemctl daemon-reload
[root@localhost docker]# systemctl restart docker
~~~





安装dig 

~~~
yum install bind-utils
~~~

~~~
[root@localhost docker-airflow-master]# dig @114.114.114.114 registry-1.docker.io

; <<>> DiG 9.9.4-RedHat-9.9.4-74.el7_6.2 <<>> @114.114.114.114 registry-1.docker.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41740
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;registry-1.docker.io.          IN      A

;; ANSWER SECTION:
registry-1.docker.io.   349     IN      A       54.175.43.85

;; Query time: 20 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: 一 8月 12 14:38:25 CST 2019
;; MSG SIZE  rcvd: 65

~~~





~~~shell
vi /etc/hosts
# add
54.175.43.85 registry-1.docker.io
~~~

