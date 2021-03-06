---
title: Docker 
date: 2018-05-08 12:52:08
tags: Docker
categories: Docker
---
# 操作虚拟机

列出虚拟机

    $docker-machine ls

启动虚拟机

    $ docker-machine start default

停止虚拟机

    $ docker-machine stop default

查看虚拟机IP

    $ docker-machine ip

查看虚拟机环境

    $ docker-machine env

# 操作容器
---
### 启动容器

启动容器并启动bash（交互方式）:

    $docker run -i -t <image_name/continar_id> /bin/bash

启动容器以后台方式运行(更通用的方式）：

    $docker run -d -it  image_name

ps：这里的 image_name 包含了tag：hello.demo.kdemo:v1.0
附着到容器

### 附着到正在运行的容器

    docker attach <id/container_name>

进入正在运行的容器内部，同时运行bash(比attach更好用)

    docker exec -t -i <id/container_name>  /bin/bash

ps：docker exec是如此的有用，以至于我们通常是将其封装为一个脚本，放到全局可调用的地方，比如，可以写成一个indocker.sh：

    $cat indocker.sh 
    docker exec -t -i $1 /bin/bash
    
    # 查看需要附着的容器id
    $docker ps | less -S
    CONTAINER ID        IMAGE                                                 
    9cf7b563f689        hello.demo.kdemo:v160525.202747

    $./indocker.sh 9cf7b563f689 

### 查看容器日志

    docker logs <id/container_name>

实时查看日志输出

    docker logs -f <id/container_name> (类似 tail -f) (带上时间戳-t）

### 查看容器

列出当前所有正在运行的container

    $docker ps

用一行列出所有正在运行的container（容器多的时候非常清晰）

    $docker ps | less -S

列出所有的container

    $docker ps -a  

列出最近一次启动的container

    $docker ps -l 

显示一个运行的容器里面的进程信息

    $docker top Name/ID  

查看容器内部详情细节：

    $docker inspect <id/container_name>

在容器中安装新的程序

    $docker run image_name apt-get install -y app_name  

    Note： 在执行apt-get 命令的时候，要带上-y参数。如果不指定-y参数的话，apt-get命令会进入交互模式，需要用户输入命令来进行确认，但在docker环境中是无法响应这种交互的。apt-get 命令执行完毕之后，容器就会停止，但对容器的改动不会丢失。

从容器里面拷贝文件/目录到本地一个路径

    $docker cp Name:/container_path to_path  
    $docker cp ID:/container_path to_path

保存对容器的修改（commit） 当你对某一个容器做了修改之后（通过在容器中运行某一个命令），可以把对容器的修改保存下来，这样下次可以从保存后的最新状态运行该容器。

    $docker commit ID new_image_name  

    Note： image相当于类，container相当于实例，不过可以动态给实例安装新软件，然后把这个container用commit命令固化成一个image。

删除单个容器

    $docker rm Name/ID 

    -f, –force=false; -l, –link=false Remove the specified link and not the underlying container; -v, –volumes=false Remove the volumes associated to the container

删除所有容器

    $docker rm `docker ps -a -q`  

停止、启动、杀死、重启一个容器

    $docker stop Name/ID  
    $docker start Name/ID  
    $docker kill Name/ID  
    $docker restart name/ID

# 操作Image
---
### 列出镜像

    $sudo docker images

    -a, –all=false Show all images; –no-trunc=false Don’t truncate output; -q, –quiet=false Only show numeric IDs

### 从dockerhub检索image

    $docker search image_name

### 下载image

    $docker pull image_name

### 删除一个或者多个镜像;

    $docker rmi image_name  

    -f, –force=false Force; –no-prune=false Do not delete untagged parents

### 显示一个镜像的历史;

    $docker history image_name

### 发布docker镜像

    $docker push new_image_name

ps:要发布到私有Registry中的镜像，在镜像命名中需要带上Registry的域名（如果非80端口，同时需要带上端口号）比如：

$docker push dockerhub.yourdomain.com:443/hello.demo.kdemo:v1.0

### 拉取docker镜像

    $docker pull image_name

# 网络操作
---
### 查看docker0的网络(宿主机上操作)

    $ip a show docker0

### 查看容器的IP地址

    $docker inspect -f '{{ .NetworkSettings.IPAddress }}' <id、container_name>

### 附着到容器内部查看其内部ip：

    $ip a show eth0

# 查看docker基础信息
---
### 查看docker版本

    $docker version
### 查看docker系统的信息

    $docker info

# 从容器内拷贝文件到主机
首先要知道virtualbox与windows的共享目录例如/c/Users。

    $ docker cp <containerId>:/容器内文件path 共享目录  
启动或未启动的容器都可以拷贝。

# Docker Compose
Compose is a tool for defining and running multi-container Docker applications. 

查看命令详细参数：例如docker-compose run -h

     -d, --detach              Detached mode: Run containers in the background,
                               print new container names. Incompatible with
                               --abort-on-container-exit.
    --no-color                 Produce monochrome output.
    --quiet-pull               Pull without printing progress information
    --no-deps                  Don't start linked services.
    --force-recreate           Recreate containers even if their configuration
                               and image haven't changed.
    --always-recreate-deps     Recreate dependent containers.
                               Incompatible with --no-recreate.
    --no-recreate              If containers already exist, don't recreate
                               them. Incompatible with --force-recreate and -V.
    --no-build                 Don't build an image, even if it's missing.
    --no-start                 Don't start the services after creating them.
    --build                    Build images before starting containers.
    --abort-on-container-exit  Stops all containers if any container was
                               stopped. Incompatible with -d.
    -t, --timeout TIMEOUT      Use this timeout in seconds for container
                               shutdown when attached or when containers are
                               already running. (default: 10)
    -V, --renew-anon-volumes   Recreate anonymous volumes instead of retrieving
                               data from the previous containers.
    --remove-orphans           Remove containers for services not defined
                               in the Compose file.
    --exit-code-from SERVICE   Return the exit code of the selected service
                               container. Implies --abort-on-container-exit.
    --scale SERVICE=NUM        Scale SERVICE to NUM instances. Overrides the
                               `scale` setting in the Compose file if present.


### 启动所有容器
  
1.  进入docker-compose.yml所在目录
2.  docker-compose up

### 停止所有容器
1.  进入docker-compose.yml所在目录
2.  docker-compose stop

### 启动或停止指定容器
    docker-compose [up|stop]  docker-compose.yml中设定的容器名
### Build

当修改dockerfile或者docker-compose时，运行docker-compose build 重建镜像。
 
### 重新部署容器
    1. 修改代码，在本地build，生成镜像
    2. docker-compose up -d --force-recreate --no-deps --build  discovery


# 主机访问虚拟机中容器
  
  本机运行服务直接访问docker容器是访问不到的，需要虚拟机中转。

  假设本机虚拟网卡地址192.168.152.1，虚拟机运行后对外地址192.168.152.131，其中的各docker容器网段为172.17。

  route add -p 172.17.0.0 mask 255.255.255.0 192.168.152.131    