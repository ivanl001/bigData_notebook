#### 一，私服registry的搭建和使用

##### 1， 拉取registry镜像

> docker pull registry

##### 2， 从镜像启动容器

*需要把容器内的/var/lib/registry地址映射到宿主机上保存*

> docker run -d -p 5000:5000 -v /data/docker/repo:/var/lib/registry --name private_registry registry

* 查看该私有镜像

> http://node101:5000/v2/_catalog

```json
{"repositories":[]}
```

* ok，私服搭建成功了！

##### 3， push镜像到registry私服上去

* 首先默认情况下仓库是dockerhub那个，但是我们需要传到私服上，所以需要先打tag

> docker tag ivanl0001:latest 192.168.147.103:5000/ivanl0001

* 尝试提交

> docker push 192.168.147.103:5000/ivanl0001

* 失败，大致报错如下：

> docker server gave HTTP response to HTTPS client

* 查询资料发现是高版本只支持https请求，需要添加指定网址的http支持

* 修改文件/etc/docker/daemon.json，增加字段“insecure-registries”

```json
{
  "registry-mirrors": ["https://08jkonzq.mirror.aliyuncs.com"],
  "insecure-registries":["192.168.147.103:5000"]
}
```

* 重新提交，成功!

> docker push 192.168.147.103:5000/ivanl0001

##### 4，从私服上拉取

> docker pull 192.168.147.103:5000/ivanl0001

##### 5，然后可以正常使用了

* 8080已经被marathon占用了

> docker run -d -p 9090:8080 --name spring-test 192.168.147.103:5000/ivanl0001



#### 二，Harbar私服的搭建

https://www.cnblogs.com/anxminise/p/9764221.html

https://www.cnblogs.com/anxminise/p/9776043.html

https://www.cnblogs.com/anxminise/p/9776038.html



##### 1,  安装docker-compose

*Harbor好像依赖docker-compose，所以先安装docker-compose*

###### 1.1, 下载docker-compose的最新版本

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

###### 1.2, 为docker-compose添加可执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

###### 1.3, 验证查看docker-compose版本

```shell
docker-compose --version
```



##### 2,  安装Harbor

###### 2.1, 下载Harbor安装包

*如果下载不下来也可以通过网页直接下载：http://harbor.orientsoft.cn/*

```shell
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.3.tgz
```

###### 2.2, 解压离线安装包

```shell
tar -zxvf harbor-offline-installer-v1.5.3.tgz
mv harbor /usr/local/
```

###### 2.3, 编辑配置文件

```shell
vim /usr/local/harbor/harbor.cfg
```

* 修改如下内容

```properties
hostname = 192.168.147.103 #修改harbor的启动ip，这里需要依据系统ip设置
harbor_admin_password = Harbor12345 #修改harbor的admin用户的密码
```

###### 2.4, 安装运行harbor

* 在harbor根目录下执行如下：

```shell
./prepare #配置Harbor
/usr/local/harbor/install.sh #安装启动Harbor

# 如果需要关闭Harbor
# cd 到 harbor目录下
docker-compose stop
```

###### 2.5, 查看相关进程

```shell
docker-compose ps
```

###### 2.6, 安装之后会自动启动,直接查看即可

http://192.168.147.103

默认账户名：admin

默认密码：Harbor12345

##### 3, Harbor命令行登陆

###### 3.1, 增加http支持

* 在需要登陆的主机上修改配置文件：/etc/docker/daemon.json, 增加如下内容：

```json
"insecure-registries":["192.168.147.103"]
```

###### 3.2, 重启docker

```shell
systemctl restart docker
```

###### 3.3, 登陆

```shell
docker login 192.168.147.103

admin
Harbor12345
```

##### 4, Harbor的使用

###### 4.0, 页面创建仓库

> ivanl001

###### 4.1, tag

> docker tag nginx:latest 192.168.147.103/ivanl001/nginx-test

###### 4.2, images

> docker images

###### 4.2, push

> docker push 192.168.147.103/ivanl001/nginx-test

###### 4.3, pull

> docker pull 192.168.147.103/ivanl001/nginx-test:latest





#### 三，dockerhub的上传和拉取镜像

##### 1， 首先到dockerhub上注册

* https://hub.docker.com/

##### 2， 登陆并创建需要的仓库名

* 我这里创建了ivanl001仓库

##### 3， 通过Dockerfile创建本地镜像

* 具体参考“01-Docker03 运行jar包.md”文件

##### 4， 重新tag成需要的镜像

* "01-Docker03 运行jar包"中最后生成的镜像REPOSITORY是“spring/test”，但是我们刚才创建的仓库是ivanl001， 所以需要重新进行tag成指定仓库ivanl001下的test，不然上传不能成功。

> docker tag 239d336e1fcc:latest ivanl001/test

##### 5，上传镜像

*感觉一个小程序，上传的镜像要有600多M有点大，后续需要看一下有没有优化手段，要不然公服的镜像空间占用还是蛮大的*

> docker push ivanl001/test

