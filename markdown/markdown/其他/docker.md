# 概述（overview）
## 什么是容器（container）？
- 是镜像的可运行实例。可以使用 DockerAPI 或 CLI 创建、启动、停止、移动或删除容器
- 可以在本地计算机、虚拟机上运行或部署到云中
- 是可移植的（可以在任何操作系统上运行）
- 与其他容器隔离，并运行自己的软件、二进制文件和配置
## 什么是容器镜像（image）？
启动一个容器时，会启用一个独立的（isolated）文件系统。这个自定义的文件系统由一个容器镜像提供，此文件系统包含容器运行的所有必要元素，包括所有依赖项、配置、脚本、二进制文件等。该镜像还包含容器的其他配置，例如环境变量、要运行的默认命令和其他元数据
# 如何使用docker?
## 镜像
```shell
#搜索镜像，以ubuntu为例
docker search ubuntu

#从dockerhub拉取ubuntu镜像
docker pull ubuntu

#前台运行一个ubuntu容器，并输出hello world
docker run -i -t ubuntu /bin/echo "Hello world"
#-t: 在新容器内指定一个伪终端或终端
#-i: 允许你对容器内的标准输入 (STDIN) 进行交互

#运行上面的命令后，一个ubuntu容器就已经启动了

#查看已经在运行的容器
docker ps
#-a参数可以查看所有的容器状态

#有时需要后台运行一个容器，就要用到-d参数
docker run -d ubuntu
#会输出一个容器id

#可以使用以下命令停止容器
docker stop x
#x可以使容器名，也可以是容器id

#停止的容器可以通过以下命令启动
docker start x
#x可以使容器名，也可以是容器id

#可以使用以下命令重启容器
docker restart x
#x可以使容器名，也可以是容器id

#在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
docker attach
docker exec
#推荐 docker exec 命令，因为此命令会退出容器终端，但不会导致容器的停止
docker exec -it 243c32535da7 /bin/bash
```