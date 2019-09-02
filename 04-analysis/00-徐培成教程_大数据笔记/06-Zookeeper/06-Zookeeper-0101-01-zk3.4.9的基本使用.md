## 1, zk的shell基本使用

*每一个目录都是一个节点*

### 1.1, 增：创建节点,创建节点必须要给节点赋值

```shell
# 创建节点必须要赋值，不然创建失败
create /ivanl001 ivanl001

# 创建节点不可以递归，也就是说，多层节点，上层节点必须要先创建
create /ivanl001/ivanl002 ivanl002
```

### 1.2, 查：显示目录文件和节点内容

* 显示节点目录文件

```shell
ls /
ls /zookeeper
ls /zookeeper/quota
```



* 获取节点的信息和值

```shell
# 获取某个节点
get /ivanl001
```



### 1.3, 改：更改某个节点的值

```shell
# 更改某个节点
set /ivanl001/ivanl002 ivanl00200
```



### 1.4, 删：删除节点

* 直接删除某个最里层节点

```shell
delete /a
```



* 递归删除，如果/a下面有其他的内容，delete删不掉的，需要递归删除

```shell
# 递归删除节点/a和该节点下的所有内容
rmr /a
```



## 2, zk的java API

> 全部代码参考：im.ivanl001.bigData.Zookeeper.A01_Zookeeper



### 2.0, 创建zk连接

```java
//依赖jar包
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>2.4.9</version>
</dependency>

//代码中使用的zk是静态代码中直接生成的
private static ZooKeeper zk = new ZooKeeper("centos01:2181,centos02:2181,centos03:2181", 5000, null);
```





### 2.1, 增

```java
//创建
@Test
public void zk_createNode_ephemeral_test() throws Exception{
  //注意：这里创建的是临时节点
  ///String result = zk.create("/ivanl001", "not leader".getBytes(), ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.EPHEMERAL);
  //一般我们创建都不是临时节点吧，比如高可用监控，因为临时节点一旦断掉，连接就会断开，就不能再监听了
  String result = zk.create("/ivanl002", "leader".getBytes(), ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.PERSISTENT);
  System.out.println(result);
  System.out.println("finieshed");
}
```

### 2.2, 查

* 查内容

```java
//查询某个节点
@Test
public void zk_getData_test() throws Exception {
  Stat stat = new Stat();
  String path = "/ivanl002";
  byte[] data = zk.getData(path, null, stat);
  System.out.println("result data:" + new String(data) + ":" + stat.getVersion());
}
```

* 查子节点

```java
//这个是迭代的查询根目录/下的所有节点
@Test
public void zk_lsAll_test() throws Exception {
  ls_all("/");
}

//迭代输出所有的路径
public void ls_all(String path) throws Exception{
  List<String> results = zk.getChildren(path, null);
  if (results.isEmpty() || results == null) {
    return;
  }else {
    for (String str : results) {
      //System.out.println(str);

      if ("/".equals(path)) {
        System.out.println(path + str);
        //这里需要注意的是：如果是最上一级，其实就是一个 "/", 需要特殊处理
        //System.out.println("---" + path  + str);
        ls_all(path + str);
      }else {
        System.out.println(path + "/" + str);
        //System.out.println("---" + path + "/" + str);
        ls_all(path + "/" + str);
      }
    }
  }
}

//查某一个目录下有哪些节点
@Test
public void zk_ls_test() throws IOException, KeeperException, InterruptedException, Exception {

  ZooKeeper zk = new ZooKeeper("centos01:2181,centos02:2181,centos03:2181", 5000, null);
  List<String> results = zk.getChildren("/", null);

  for (String str : results) {
    System.out.println(str);
  }
}
```

### 2.3, 改
```java
//更新某个节点
@Test
public void zk_setData_test() throws Exception {
  //如果想要更改，那么需要要拿到节点的版本号，所以就需要先查询，后用版本号更改
  //默认状态的版本号是0
  Stat stat = new Stat();
  String path = "/ivanl002";

  //System.out.println(stat.getVersion());

  byte[] data = zk.getData(path, null, stat);
  System.out.println(stat.getVersion());

  Stat stat1 = zk.setData(path, "ivanl002".getBytes(), stat.getVersion());
  System.out.println("new version: " + stat1.getVersion());
}
```

### 2.4, 删
```java
//删除某个节点
@Test
public void zk_delNode_test() throws Exception {
  //如果想要删除，那么需要要拿到节点的版本号，所以就需要先查询，后用版本号删除
  Stat stat = new Stat();
  String path = "/ivanl002";

  byte[] data = zk.getData(path, null, stat);
  System.out.println(stat.getVersion());

  //这里没有返回值
  zk.delete("/ivanl002", stat.getVersion());
}
```



### 2.5, 判断是否存在及获取子节点个数

```java
//判断是否存在
@Test
public void zk_exists() throws KeeperException, InterruptedException {
  /*Stat stat = zk.exists("/ivanl001", null);
        System.out.println(stat);
        if (stat == null) {
            System.out.println("节点不存在");
        } else {
            System.out.println(stat.getVersion());
        }*/

  //如果没有字节点，个数是0
  List<String> children = zk.getChildren("/ivanl001", false);
  System.out.println(children.size());
}
```



### 2.6, 监听器

```java
@Test
public void zk_watch_test() throws Exception {

  final Stat stat = new Stat();
  final String path = "/ivanl002";

  //注意：如果是更新，会接受到通知，如果是删除了， 就会断开连接不能再继续监听了
  final Watcher watcher = new Watcher() {

    public void process(WatchedEvent event) {

      System.out.println("接收到通知---01---，数据有变动-----");
      try {

        byte[] data = zk.getData(path, this, stat);
        System.out.println("数据是:" + new String(data) + ", 版本号是:" + stat.getVersion());

        System.out.println("接收到通知---02---，数据有变动-----");

      } catch (KeeperException e) {
        e.printStackTrace();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  };
```