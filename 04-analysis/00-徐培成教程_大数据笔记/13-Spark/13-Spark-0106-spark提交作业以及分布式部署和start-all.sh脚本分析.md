spark作业提交,如果想要提交分布式作业，看##3

  > spark-submit --master local --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.WordCountApp_scala Spark_Test.jar /root/ivanl001/test/zhang.txt



## 1，local本地模式(standalone模式)

*安装好之后默认是本地模式，本地启动也可以不加--master参数*
```shell
spark-shell --master local
```



## 2, standalone

> 这个属于独立模式，spark不依赖其他(如yarn)单独运行集群

### 01, 配置slave文件

```shell
vim /usr/local/spark/conf/slaves

# 增加如下内容
centos01
centos02
centos03
```



### 02，设置jdk

```shell
/sbin/spark-config.sh

# 增加如下一句
export JAVA_HOME=/usr/local/jdk
```



### 03, 启动spark

```shell
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/start-all.sh 
```



### 04, 页面验证

> 页面上会有如下信息：Spark Master at spark://master:7077，提交分布式作业的时候需要用到spark://master:7077

* http://centos01:8080/



### 05, 关闭spark

```shell
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/stop-all.sh
```



## 3，提交作业到分布式spark上

*注意：代码里要指定是集群模式才行哦，至少不能指定成本地模式哈*



### 01, 启动hadoop集群

* 至少也要开启hdfs

```shell
start-dfs.sh
```



### 02, spark-submit提交

* 选取hdfs上的文件比如说这个文件/user/root/test/zhang.txt
* ⚠️注意：在代码中要指定集群模式哈，如果重新修改了scala代码，重新打包前最好重新编译后再打包

```shell
spark-submit --master spark://master:7077 --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.WordCountApp_scala Spark_Test.jar hdfs://master01:8020/user/root/test/zhang.txt
```





## 4，start-all.sh脚本分析

* 01，脚本主要内容如下，比较简单，这里看看
  ```shell
  # Load the Spark configuration
  . "${SPARK_HOME}/sbin/spark-config.sh"
  
  # Start Master
  "${SPARK_HOME}/sbin"/start-master.sh
  
  # Start Workers
  "${SPARK_HOME}/sbin"/start-slaves.sh
  ```
  
* 02，接下来看一下start-master.sh的主要内容
*差不多就是这个：spark-daemon.sh start org.apache.spark.deploy.master.Master --host --port --webui-port ...*

  ```shell
  # Starts the master on the machine this script is executed on.
  # 先是判断是否需要加载帮助文档啥的
  . "${SPARK_HOME}/sbin/spark-config.sh"
  . "${SPARK_HOME}/bin/load-spark-env.sh"
  
  # 判断端口，主机名，ui端口等等
  if [ "$SPARK_MASTER_PORT" = "" ]; then
    SPARK_MASTER_PORT=7077
  fi
  
  if [ "$SPARK_MASTER_HOST" = "" ]; then
    case `uname` in
        (SunOS)
            SPARK_MASTER_HOST="`/usr/sbin/check-hostname | awk '{print $NF}'`"
            ;;
        (*)
            SPARK_MASTER_HOST="`hostname -f`"
            ;;
    esac
  fi
  
  if [ "$SPARK_MASTER_WEBUI_PORT" = "" ]; then
    SPARK_MASTER_WEBUI_PORT=8080
  fi
  
  CLASS="org.apache.spark.deploy.master.Master"
  
  # 根据参数用spark-daemon.sh脚本启动Master这个类，大概就是这样：spark-daemon.sh start org.apache.spark.deploy.master.Master --host --port --webui-port ...
  
  "${SPARK_HOME}/sbin"/spark-daemon.sh start $CLASS 1 \
    --host $SPARK_MASTER_HOST --port $SPARK_MASTER_PORT --webui-port $SPARK_MASTER_WEBUI_PORT \
    $ORIGINAL_ARGS
  ```
  
* 03，接下来看一下start-slaves.sh的主要内容

  ```shell
  # Starts a slave instance on each machine specified in the conf/slaves file.
  . "${SPARK_HOME}/sbin/spark-config.sh"
  . "${SPARK_HOME}/bin/load-spark-env.sh"
  
  # Find the port number for the master
  if [ "$SPARK_MASTER_PORT" = "" ]; then
    SPARK_MASTER_PORT=7077
  fi
  
  if [ "$SPARK_MASTER_HOST" = "" ]; then
    case `uname` in
        (SunOS)
            SPARK_MASTER_HOST="`/usr/sbin/check-hostname | awk '{print $NF}'`"
            ;;
        (*)
            SPARK_MASTER_HOST="`hostname -f`"
            ;;
    esac
  fi
  
  # Launch the slaves
  # 通过调用slaves.sh循环conf/slaves文件，然后通过start-slave.sh启动每个slave节点
  "${SPARK_HOME}/sbin/slaves.sh" cd "${SPARK_HOME}" \; "${SPARK_HOME}/sbin/start-slave.sh" "spark://$SPARK_MASTER_HOST:$SPARK_MASTER_PORT"
  
  ```

* 04，start-slaves.sh使用了slaves.sh脚本，那再看看slaves.sh的主要内容
  ```shell
  # 这里进行for循环进行slave文件的每个slave
  
  for slave in `echo "$HOSTLIST"|sed  "s/#.*$//;/^$/d"`; do
    if [ -n "${SPARK_SSH_FOREGROUND}" ]; then
      ssh $SPARK_SSH_OPTS "$slave" $"${@// /\\ }" \
        2>&1 | sed "s/^/$slave: /"
    else
      ssh $SPARK_SSH_OPTS "$slave" $"${@// /\\ }" \
        2>&1 | sed "s/^/$slave: /" &
    fi
    if [ "$SPARK_SLAVE_SLEEP" != "" ]; then
      sleep $SPARK_SLAVE_SLEEP
    fi
  done
  ```
  
* 05，start-slave.sh

  ```shell
  
  . "${SPARK_HOME}/sbin/spark-config.sh"
  
  . "${SPARK_HOME}/bin/load-spark-env.sh"
  
  # First argument should be the master; we need to store it aside because we may
  # need to insert arguments between it and the other arguments
  MASTER=$1
  shift
  
  # Determine desired worker port
  if [ "$SPARK_WORKER_WEBUI_PORT" = "" ]; then
    SPARK_WORKER_WEBUI_PORT=8081
  fi
  
  # Start up the appropriate number of workers on this machine.
  # quick local function to start a worker
  function start_instance {
    WORKER_NUM=$1
    shift
  
    if [ "$SPARK_WORKER_PORT" = "" ]; then
      PORT_FLAG=
      PORT_NUM=
    else
      PORT_FLAG="--port"
      PORT_NUM=$(( $SPARK_WORKER_PORT + $WORKER_NUM - 1 ))
    fi
    WEBUI_PORT=$(( $SPARK_WORKER_WEBUI_PORT + $WORKER_NUM - 1 ))
  
    "${SPARK_HOME}/sbin"/spark-daemon.sh start $CLASS $WORKER_NUM \
       --webui-port "$WEBUI_PORT" $PORT_FLAG $PORT_NUM $MASTER "$@"
  }
  
  if [ "$SPARK_WORKER_INSTANCES" = "" ]; then
    start_instance 1 "$@"
  else
    for ((i=0; i<$SPARK_WORKER_INSTANCES; i++)); do
      start_instance $(( 1 + $i )) "$@"
    done
  fi
  ```