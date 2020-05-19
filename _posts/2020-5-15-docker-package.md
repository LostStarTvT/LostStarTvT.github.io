---
layout: post
title: Docker部署项目
tags: java
---


> 记录在springBoot项目中使用maven将项目打包并且部署到docker服务器上。

##  目录
* 目录
{:toc}
[参考链接](http://www.macrozheng.com/#/reference/docker_maven)

# 一、Docker Registry

## Docker Registry 2.0搭建

```shell
docker run -d -p 5000:5000 --restart=always --name registry2 registry:2
```

如果遇到镜像下载不下来的情况，需要修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值，然后重启docker服务：

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

## Docker开启远程API

### 用vim编辑器修改docker.service文件

```
vi /usr/lib/systemd/system/docker.service
```

需要修改的部分：

```shell
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

修改后的部分：

```shell
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

### 让Docker支持http上传镜像

其中地址就是直接写服务器自己的地址。

```shell
echo '{ "insecure-registries":["192.168.3.130:5000"] }' > /etc/docker/daemon.json
```

### 修改配置后需要使用如下命令使配置生效

```shell
systemctl daemon-reloadCopy
```

### 重新启动Docker服务

```shell
systemctl stop docker
systemctl start docker
```

### 开启防火墙的Docker构建端口

```shell
firewall-cmd --zone=public --add-port=2375/tcp --permanent
firewall-cmd --reload
```

## 使用Maven构建Docker镜像

> 该代码是在mall-tiny-02的基础上修改的。

### 在应用的pom.xml文件中添加docker-maven-plugin的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.macro.mall.tiny</groupId>
    <artifactId>mall_study</artifactId>
    <!--这里配置docker里面具体的安装images名称-->
    <version>0.0.2-SNAPSHOT</version>
    <name>mall_study</name>
    <description>Demo project for Spring Boot</description>
	

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!--需要加的插件在这里，也就是将项目build 然后上传到服务器。-->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.1.0</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <imageName>mall-tiny/${project.artifactId}:${project.version}</imageName>
                    <!--配置服务器ip地址-->
                    <dockerHost>http://192.168.60.130:2375</dockerHost>
                    <baseImage>java:8</baseImage>
                    <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]
                    </entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

相关配置说明：

- executions.execution.phase:此处配置了在maven打包应用时构建docker镜像；
- imageName：用于指定镜像名称，mall-tiny是仓库名称，`${project.artifactId}`为镜像名称，`${project.version}`为仓库名称；
- dockerHost：打包后上传到的docker服务器地址；
- baseImage：该应用所依赖的基础镜像，此处为java；
- entryPoint：docker容器启动时执行的命令；
- resources.resource.targetPath：将打包后的资源文件复制到该目录；
- resources.resource.directory：需要复制的文件所在目录，maven打包的应用jar包保存在target目录下面；
- resources.resource.include：需要复制的文件，打包好的应用jar包。

### 修改application.yml，将localhost改为db

> 可以把docker中的容器看作独立的虚拟机，mall-tiny-docker访问localhost自然会访问不到mysql，docker容器之间可以通过指定好的服务名称db进行访问，至于db这个名称可以在运行mall-tiny-docker容器的时候指定。

```yml
spring:
  datasource:
    url: jdbc:mysql://db:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
```

### 使用IDEA打包项目并构建镜像

直接双击maven中的package按钮便可以直接的进行打包，如果是第一次打包需要等待一段时间，因为要下载对应的依赖，最终的结果出现如下：

![dockerSuccess.png](https://pic.tyzhang.top/images/2020/05/15/dockerSuccess.png)

此时在docker服务器上，输入docker images便可以发现已经上传了：

![docker-image.png](https://pic.tyzhang.top/images/2020/05/15/docker-image.png)

# 二、运行项目

等到上传到docker服务器以后，便可以通过配置进行上传和运行项目：

- 使用docker命令启动（--link表示应用可以用db这个域名访问mysql服务）：

```shell
docker run -p 8080:8080 --name mall-tiny-docker \
--link mysql:db \
-v /etc/localtime:/etc/localtime \
-v /mydata/app/mall-tiny-docker/logs:/var/logs \
-d mall-tiny/mall-study:0.0.2-SNAPSHOT
```

其中-d即为上图中的image的名称， --link表示引用本地的mysql数据，且别名为db。对应与上面的**修改application.yml，将localhost改为db** 此时便可以直接的访问项目，进行上线。