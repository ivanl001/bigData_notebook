## 1, dockerui

参照官网教程：

https://hub.docker.com/r/abh1nav/dockerui

⚠️注意：仅供学习，实际用到的不多，功能有限，貌似还有bug

### Quickstart

#### Step 1

Pull the latest image:

```shell
docker pull abh1nav/dockerui:latest
```

#### Step 2

If you're running Docker using a unix socket (default):

```shell
docker run -d -p 9000:9000 -v /var/run/docker.sock:/docker.sock \
--name dockerui abh1nav/dockerui:latest -e="/docker.sock"
```

If you're running Docker over tcp:

```shell
docker run -d -p 9000:9000 --name dockerui \
abh1nav/dockerui:latest -e="http://<dockerd>:4243"
```

#### Step 3

Open your browser to  http://192.168.147.101:9000



## 2, Shipyard(重点)

* 据说已经停止维护，暂时不看啦

