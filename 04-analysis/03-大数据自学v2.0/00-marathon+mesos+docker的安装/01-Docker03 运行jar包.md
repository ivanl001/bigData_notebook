##### 1, 项目打jar包，传到服务器

* 包名：docker_test.jar
* 服务器上位置： /root/docker_test/docker_test.jar

##### 2, 编写Dockerfile

* 在jar包相同的位置下创建v文件，并编写如下内容，简单说就是在java环境下运行java -jar容器

```dockerfile
FROM java:8
VOLUME /tmp
ADD docker_test.jar docker_test.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","docker_test.jar"]
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
EXPOSE 808
```



##### 3, 从Dockerfile创建镜像

* 注意：最后一个点不能省略，是指build当前目录

> docker build -t spring/test .

##### 4, 查看镜像

> docker images

##### 5, 运行镜像

> docker run -d -p 8090:8080 --name spring-test spring/test

##### 6, 查看运行容器

> docker ps 

##### 7, 验证

*我项目中写了一个接口：/ivanl001/test01, 服务器映射地址是node101*

http://node101:8080/ivanl001/test01











docker build -t test/docker_test .