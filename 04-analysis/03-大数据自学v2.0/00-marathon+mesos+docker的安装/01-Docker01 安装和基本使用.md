dcentos7: https://docs.docker.com/install/linux/docker-ce/centos/

### Docker的安装使用
#### 1， 卸载旧版本
```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 2, 先安装依赖
*我在安装的时候，第一次报错，重新执行了一次正常了*
```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

#### 3, 设置安装仓库
*我这里报错*

```shell
# 执行前先把yum更新到最新，要不然可能会报错啥到
$ sudo yum update

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

#### 4, 安装docker
```shell
$ sudo yum install docker-ce -y

# 如果需要指定安装版本，使用如下命令
$ sudo yum install docker-ce-<VERSION STRING>
```

#### 5，启动docker
```shell
sudo systemctl start docker

systemctl stop docker
```

#### 6, 启动hello-world镜像检查是否安装正常
```shell
sudo docker run hello-world
```

#### 7, 设置阿里云镜像加速
* 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://08jkonzq.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Dcoker的基本命令
#### 1，帮助命令

```shell
docker version

docker info

docker --help
```

#### 2, 镜像信息查看
```shell
$> docker images

# 列出本地所有镜像的详细信息(含中间映像层)
$> docker images -a

# 查询当前镜像的id值
$> docker images -q

# 查询当前所有镜像的id
$> docker images -qa

# 显示镜像的摘要信息
$> docker images --digests

```

#### 3，镜像操作

```shell
# 具体参考2
$> docker images

$> docker search tomcat


# 查找star大于100的
$> docker search tomcat -s 100

# 下载镜像
$> docker pull tomcat

# 删除指定镜像，空格隔开可以删除多个
# rmi: revmove images
$> docker rmi hello-world
# 强制删除
$> docker rmi -f hello-world
# 删除所有的
$> docker rmi -f $(docker images -qa)
```

#### 4，容器命令01

```shell
# 我们这里通过centos镜像创建一个centos容器然后演示docker的容器命令
$> docker pull centos

# 启动并创建容器, i代表interactive， t代表terminal
# -it也可以写成 -i -t
$> docker run -it 1e1148e4cc2c

# 从容器中退出，这个是在容器中运行哈
# exit是代表容器关闭，然后退出，ctrl代表容器不关闭，然后退出界面
$> exit
$> ctrl + P + Q

$> docker run -it --name centos01 1e1148e4cc2c
$> docker run -it centos # 后面也可以直接跟名字就可以



# 列出所有正在运行的docker容器,这个是在宿主机上运行哈
$> docker ps
$> docker ps -a # 显示所有的，包括正在运行的和历史运行的
$> docker ps -l # 显示最近创建的容器
$> docker ps -n 3 # 显示最近创建的3个容器
$> docker ps -q # 静默模式，只显示当前正在运行的容器编号
$> docker ps --no-trunc # 不截断输出，显示信息的全部


# 从容器中退出，这个是在容器中运行哈
# exit是代表容器关闭，然后退出，ctrl代表容器不关闭，然后退出界面
$> exit
$> ctrl + P + Q


# 删除容器, 注意：只能删除已经停止的容器
$> docker rm 8dad94645d45
$> docker rm $(docker ps -qa)
$> docker rm -f $(docker ps -qa)

# 退出的容器重新启动
$> docker start 5455e2a3e728

# 重启容器
$> docker restart 5455e2a3e728

# 停止容器
$> docker stop 5455e2a3e728

# 强制停止容器
$> docker kill 5455e2a3e728
```

#### 5, 容器命令02
```shell
# 守护进程式启动, 这个基本启动之后就会退出
$> docker run -d centos

# 守护线程， 并一直循环打印log，启动后ID是251fd9453a78
$> docker run -d centos /bin/sh -c "while true; do echo hello; sleep 2; done"

# 查看上面启动的那个守护线程的log
$> docker logs 251fd9453a78
$> docker logs -t 251fd9453a78 # 这个可以显示日志打印的时间
$> docker logs -t -f 251fd9453a78 # 显示时间并追加
$> docker logs -t -f -tail 10 251fd9453a78 # 从最后10行开始

# 查看top, 进程
$> docker top 251fd9453a78

# 查看docker内部细节
$> docker inspect 251fd9453a78

# 重新进入退出未关闭的容器01
$> docker attach f9e146bc4fd

# 重新进入退出未关闭的容器02,这里试了一下好像可以进入，但是无法执行内部命令，后面再看看#是因为之前没有加i，不能交互
$> docker exec -it f9e146bc4fd8 /bin/bash
# exec还有一个功能就是可以在不进入内部的时候执行容器命令
$> docker exec -t f9e146bc4fd8 ls /
# 直接在外部执行内部centos的ls命令
$> docker exec -t 35dc9c72f95f ls -l /tmp

# 容器内数据拷贝到主机
# 容器内数据拷贝到主机
# 容器内数据拷贝到主机
$> docker cp f9e146bc4fd8:/tmp/yum.log /root/
```

#### 6, 镜像commit

```shell
# 运行tomcat并制定端口号, -p制定端口号, -P是随机分配端口
# 8888是docker暴露给外界的端口，必须要映射出来，要不然外界没法访问
$> docker run -it -p 8888:8080 tomcat
# P大写的话是随机分配，分配的端口可以通过docker ps查看
$> docker run -it -P tomcat


# 进入到tomcat根目录下并删除掉文档目录
$> docker exec -it 35dc9c72f95f /bin/bash


# 提交没有文档的这个容器, a:author, m:message
$> rm -rf webapps/docs/
$> docker commit -a 'ivanl001' -m 'without docs' 35dc9c72f95f tomcat_without_docs:1.0

# 先删除当前运行的所有容器
$> docker rm -f $(docker ps -q)

# 运行刚才提交的那个image,注意：需要带版本号， 这个启动的容器中就没有文档
$>  docker run -it -p 8080:8080 tomcat_without_docs:1.0

# tomcat也可以后台守护进程运行 
$> docker run -d -p 8888:8080 tomcat_without_docs:1.0

```

#### 7, Docker容器数据卷

##### 7.1， 容器卷用-v， --volume list

```shell
# 启动容器，并把容器中的data目录和宿主机中的/data/docker/centos文件共享，双向同步哈
$> docker run -it -v /data/docker/centos:/data centos
# 注意：这里-v必须要在镜像前
$> docker run -d  -v /root/marathon/:/root/marathon/ b3f33a92ad10
# 通过如下方式进行查看具体的配置json格式
$> docker inspect c87d8c174f49

# read only:ro, 只读， 在容器中共享目录下文件是只读的，不能修改
$> docker run -it -v /data/docker/centos:/data:ro centos
# 通过如下方式进行查看具体的配置json格式
$> docker inspect c87d8c174f49
```

##### 7.2, 容器卷用dockerfile

* 参考Dockerfile内容：

  * tomcat的Dockerfile：https://github.com/docker-library/tomcat/blob/1c76426151f1d88bc3dc042d5b45174f9f9ae3d0/9.0/jdk13/openjdk-oracle/Dockerfile
  * redis的Dockerfile：https://github.com/docker-library/redis/blob/0b2910f292fa6ac32318cb2acc84355b11aa8a7a/5.0/Dockerfile

  

* 创建Dockerfile，内容如下：

```shell
# volume test
FROM centos
VOLUME ["/data1", "data2"]
CMD echo "finished, success----------------"
CMD /bin/bash
```

* 使用Dockerfile构建镜像

  > docker build -f /data/docker/dockerfile/Dockerfile -t dockerfile_image .

*  查看镜像发现已经存在dockerfile_image 

  > docker images

* 运行镜像, 可以在其根目录下发现自动生成了两个文件夹data1，data2

  > docker run -it dockerfile_image

* 这次虽然我们没有通过-v进行镜像指定，但是docker默认也会进行一个默认的指定位置，通过inspect命令可以发现如下内容：data1指定给/var/lib/docker/volumes/30a6c5496d2aa7b095c14d0df685fe6b7d71914a08900c5deb681f1d8bf05c67/_data

  ```json
  "Mounts": [
              {
                  "Type": "volume",
                  "Name": "30a6c5496d2aa7b095c14d0df685fe6b7d71914a08900c5deb681f1d8bf05c67",
                  "Source": "/var/lib/docker/volumes/30a6c5496d2aa7b095c14d0df685fe6b7d71914a08900c5deb681f1d8bf05c67/_data",
                  "Destination": "/data1",
                  "Driver": "local",
                  "Mode": "",
                  "RW": true,
                  "Propagation": ""
              },
              {
                  "Type": "volume",
                  "Name": "f0b31077bf25aa9b92142d3c688a1994732e5977c74e3c67095a107a30a75825",
                  "Source": "/var/lib/docker/volumes/f0b31077bf25aa9b92142d3c688a1994732e5977c74e3c67095a107a30a75825/_data",
                  "Destination": "data2",
                  "Driver": "local",
                  "Mode": "",
                  "RW": true,
                  "Propagation": ""
              }
          ]
  ```

  

### 数据卷容器

#### 1,  父容器c00

*  先创建父容器

> docker run -it --name c00  dockerfile_image

#### 2， 子容器c10

* 然后从父容器创建子容器, 这样， c00  和 c10 的data1和data2就是共享的，双向的

> docker run -it --name c10 --volumes-from c00 dockerfile_image

#### 3， 子容器c20

* 同2一样，再创建c00的子容器c20, 这样，三个容器的data1和data2都是共享的

> docker run -it --name c20 --volumes-from c00 dockerfile_image

#### 4,  关闭c00

* 如何关掉父容器，其余两个容器的共享功能并不会被影响

> docker kill bdd11378491d

### Dockerfile语法

```
Dockerfile是软件的原材料
Docker镜像是软件的交付品
Docker容器是软件的运行态
```

#### 1， dockerfile的保留字

```dockerfile
FROM        基础镜像，当前新镜像是基于哪个镜像的
MAINTAINER  镜像作者
RUN         容器构建时候需要运行的命令
EXPOSE      容器暴露的端口号
WORKDIR     指定在创建容器后， 终端默认登录进来的工作目录，比如centos的/目录
ENV         用来在构建镜像过程中设置环境变量
ADD         将宿主机目录下的文件拷贝进镜像且ADD命令或自动处理uRL和解压tar压缩包
COPY        将宿主机目录下的文件拷贝进镜像
			COPY src dest
			COPY ['src', 'dest']
VOLUME    	容器数据卷，用于数据保存和持久化工作
CMD 		指定一个容器启动时候所需要执行的命令
			Dockerfile中可以有多个CMD命令，但是只有最后一个会生效，CMD会被docker 			  run之后的参数替换
ENTRYPOINT  指定一个容器启动时要运行的命令
		    ENTRYPOINT的目的和CMD一样， 都是在指定容器启动程序及参数
ONBUILD    	
```

|     build     |  both   |    run     |
| :-----------: | :-----: | :--------: |
|     FROM      | WORKDIR |    CMD     |
|  MAINTAINER   |  USER   |    ENV     |
|     COPY      |         |   EXPOSE   |
|      ADD      |         |   VOLUME   |
|      RUN      |         | ENTRYPOINT |
|    ONBUILD    |         |            |
| .dockerignore |         |            |

#### 2, Dockerfile自定义镜像

* centos

* 更改登录后默认目录是/data
* 登录后默认会支持vim和ifconfig命令



```dockerfile
from centos

ENV mypath /data
WORKDIR $mypath

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80
CMD /bin/bash
```



##### 2.1, 构建镜像

* 使用如上的镜像dockerfile构建成镜像, -t是tag，target string,最后一个点是当前文件夹

> docker build -f /data/docker/dockerfile/dockerfile01 -t cus_centos:1.0.0 . 

##### 2.2， 运行容器

* 进入之后检查一下， 默认目录是不是/data, 并且可以执行vim和ifconfig命令 

> docker build -f /data/docker/dockerfile/dockerfile01 -t cus_centos:1.0.0 . 

##### 2.3, 查看镜像历史

> docker history a35c41a18525

#### 3, CMD和ENTRYPOINT

##### 3.1， CMD

* CMD会被覆盖，如下命令, 因为 ls /覆盖了dockerfile中的CMD ["catalina.sh", "run"]， 所以最后并不会启动tomcat，只会打印出ls的结果

  > docker run -it tomcat ls /

##### 3.2， ENTRYPOINT（先不练习了）

* ENTRYPOINT不会被覆盖，后面的ENTRYPOINT命令只会以追加的方式执行

  > 



