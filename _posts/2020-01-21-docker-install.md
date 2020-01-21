---
title: docker工具安装篇
date: 2020-01-21 13:30:09
categories:
- docker
tags:
- docker
---

docker安装，基础篇。

# 查看硬件版本

```shell
$uname -a
Linux ip-172-26-5-137.ap-northeast-1.compute.internal 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

#看centos版本
$cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
```



# （可选）移除旧的版本：

~~~shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
~~~



# （必须1）安装一些必要的系统工具：



~~~shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
~~~





# 添加软件源信息：

（可选）国内用阿里云的资源

~~~shell
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
~~~



（可选）更新 yum 缓存：

~~~shell
$ sudo yum makecache fast
~~~





（必须2）国外服务器就用官方的

~~~shell
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
~~~



（必须3）安装 Docker-ce：

~~~shell
$ sudo yum install docker-ce docker-ce-cli containerd.io

##也可以安装指定版本
yum list docker-ce --showduplicates | sort -r
~~~



（必须3）启动 Docker 后台服务

~~~shell
$ sudo systemctl start docker   
~~~



(必须4) 开机自启动 docker

~~~shell
$ sudo systemctl enable docker 
~~~



```shell
$ docker --version
Docker version 18.09.6, build 481bc77156
```





测试运行 hello-world



~~~shell
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

~~~

