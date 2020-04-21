---
layout: post
title: Docker使用记录
tags: java  
---


> 之前苦于环境的搭建，总是很麻烦，突然发现现在都有docker这种工具了，很方便也很简单，记录一下

##  目录
* 目录
{:toc}
## 0.简介

**Docker**是一个开源的应用容器引擎；是一个轻量级容器技术；他的功能与传统的虚拟机出发点不同，其对比图如下所示：

  [![docker.png](https://pic.tyzhang.top/images/2020/04/09/docker.png)](https://pic.tyzhang.top/image/dPQ1)

从上图可以看出，传统的虚拟机的出现是为了让程序运行互不干扰，即每个系统之间没有联系，所以虚拟出来多个系统进行实现，但是解决的痛点可以概括为：**为了程序运行的互不干扰而构建多个系统**。这样来看其实花费是很大的，其一，**不同的软件在不同的系统中都要进行安装配置**，其二，**两个系统的运行需要耗费更多的资源**。基于以上问题，docker解决的思路是，既然是为了独立的运行程序互不干扰，那么我就直接提供一个引擎，可以**直接的运行多个应用，并且互不干扰，互相隔离，即做到应用层的隔离**。即采用类与对象的概念。而且针对第一个痛点，docker将每个应用程序的安装打包，使得安装变得非常的简单，类似于提供了一个应用商场，简化安装和配置。对于第二个痛点，**因为是只有一个引擎层，所有使得占用的资源更少，而且运行的更快**。    

实现的方法，就是将软件编译成一个镜像；镜像中已经做好了软件的配置。将镜像发布出去以后，其他使用者可以直接使用这个镜像，而不需要在此进行繁杂的配置，相当于对于软件的二次打包。  

运行中的这个镜像称为容器，容器启动是非常快速的。而且一个镜像可以开启多个容器，例如，下载了一个mysql镜像，然后可以同时运行好几个mysql，并且是各自独立的，相当于利用虚拟机的概念，将程序虚拟出来。很是方便。  其图标如下，也显示了其概念：

![20180303145450.png](https://pic.tyzhang.top/images/2020/04/08/20180303145450.png)

原理图：就相当于之前的盗版系统，系统给你预安装好很多的东西，在次安装的时候一些常用的配置就不需要设置。这里系统镜像类比于程序，比如mysql，自己安装的时候很多步骤，但是在docker中，别人设置好的镜像直接下载，然后便能直接的运行，对应的设置也能更改，即将核心的抽取出来，一些繁琐的安装直接屏蔽，非常的简单。

[![20180303145531.png](https://pic.tyzhang.top/images/2020/04/08/20180303145531.png)](https://pic.tyzhang.top/image/dNUo)

## 1.核心概念

docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）；  

docker客户端(Client)：连接docker主机进行操作；  

docker仓库(Registry)：用来保存各种打包好的软件镜像；  

docker镜像(Images)：软件打包好的镜像；放在docker仓库中；  

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用；  

**docker镜像与容器的关系，类似与类与对象的关系。即镜像为类，容器为对象，即容器为具体镜像的运行实例。**

使用Docker的步骤：  

1）、安装Docker；  

2）、去Docker仓库找到这个软件对应的镜像；  

3）、使用Docker运行这个镜像，这个镜像就会生成一个Docker容器；  

4）、对容器的启动停止就是对软件的启动停止；  

## 2.安装Docker

安装docker的前提是有linux虚拟机，不建议在window上装docker，虚拟机的话失败了还能删除..  

**centOS上安装：**

```shell
CentOS 安转docker
1、检查内核版本，必须是3.10及以上
uname -r
2、安装docker
yum install docker
3、输入y确认安装
4、启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker -v
Docker version 1.12.6, build 3e8e77d/1.12.6
5、开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
6、停止docker
systemctl stop docker

```

**linux安装：（测试的是Ubuntu 14.0.98?）**

```shell
//参考链接 https://www.runoob.com/docker/ubuntu-docker-install.html
设置阿里云为安转服务器。
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
//安装需要的包
sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
//添加使用 HTTPS 传输的软件包以及 CA 证书
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
//.添加GPG密钥
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
// 添加软件源
echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
// 更新软件包缓存
sudo apt-get update
//安装。
sudo apt-get install -y docker.io
```

## 3.Docker常用命令&操作

### 1）、镜像操作

| 操作 | 命令                                            | 说明                                                     |
| ---- | ----------------------------------------------- | -------------------------------------------------------- |
| 检索 | docker  search 关键字  eg：docker  search redis | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 | docker pull 镜像名:tag                          | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 | docker images                                   | 查看所有本地镜像                                         |
| 删除 | docker rmi image-id                             | 删除指定的本地镜像                                       |

[官方网站](https://hub.docker.com/) 

在进行下载的时候可以现在官网上搜索，然后在使用命令下载，一般不要下载最新版。可以进行指定版本。eg docker pull mysql:5.5   下载mysql5.5版本。

### 2）、容器操作

软件镜像（QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，运行的QQ）；

每run一次就是新安转了一个程序，下次在使用start命令进行启动。

步骤：

```shell
1、搜索镜像
[root@localhost ~]# docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
3、根据镜像启动容器：这个也相当于安装， name 就是为这个应用自定义的名字，如果关闭，下次使用start命令进行启动，见命令7
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙

ubunte 
关闭防火墙 ufw disable  
开启 ufw enable
查看状态 ufw status


11、查看容器的日志
docker logs container-name/container-id

更多命令参看
https://docs.docker.com/engine/reference/commandline/docker/
可以参考每一个镜像的文档

```

### 3）、安装MySQL示例

```shell
docker pull mysql:5.5   尽量不要下载最新版，5.5版本测试成功。
```

错误的启动  

```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

mysql退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01
538bde63e500        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       compassionate_
goldstine
c4f1ac60b3fc        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       lonely_fermi
81ec743a5271        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       sick_ramanujan


//错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；这个三个参数必须指定一个
```

**正确的安转启动** ：但是没有做端口映射，外网访问不进来：

```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

**做了端口映射安装启动**  ：最终可以进行测试的。

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```

几个其他的高级操作  

```shell
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
把主机的/conf/mysql文件夹挂载到 mysqldocker容器的/etc/mysql/conf.d文件夹里面
改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）

docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
指定mysql的一些配置参数
```

