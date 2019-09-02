⚠️：代码中直接指定协议为hdfs://协议，如果不想在代码中指定，可以在resources目录下建立一个core-site.xml文件，具体内容如下：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://centos02:8020/</value>
        <description>Deprecated. Use (fs.defaultFS) property
            instead</description>
    </property>
</configuration>
```



## 1, core-site.xml中指定后读写如下

### 1.1, fs_read

```java
//这里是读取,在A01_HadoopFS文件中已经有了
@Test
public void fs_read(){

  Configuration configuration = new Configuration();
  //配置文件core-site.xml中指定了hdfs协议和服务器地址之后，这里就不需要重复写了
  //<name>fs.default.name</name>
  //<value>hdfs://centos02:8020/</value>
  Path path = new Path("/user/ivanl001/ivanl001.txt");

  try {
    FileSystem fs = FileSystem.get(configuration);

    //第一种方式
    /*InputStream in = fs.open(path);
            IOUtils.copyBytes(in, System.out, 1024);*/

    //第二种方式
    FSDataInputStream fsDataInputStream = fs.open(path);
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    IOUtils.copyBytes(fsDataInputStream, byteArrayOutputStream, 1024);

    System.out.println(new String(byteArrayOutputStream.toByteArray()));
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

### 1.2, fs_write

```java
//这里是写入,在A01_HadoopFS文件中已经有了
@Test
public void fs_write(){

  Configuration configuration = new Configuration();
  Path path = new Path("/user/ivanl001/test02.txt");

  try {
    FileSystem fs = FileSystem.get(configuration);

    FSDataOutputStream fsDataOutputStream = fs.create(path);
    fsDataOutputStream.write("ivanl001 tesst02 is the king?".getBytes());

    fsDataOutputStream.close();

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```





## 2, 指定副本数

### 2.1, replicationCount

```java
//指定副本数为2
@Test
public void replicationCount(){

  Configuration configuration = new Configuration();
  Path path = new Path("/user/ivanl001/test03.txt");

  try {
    FileSystem fileSystem = FileSystem.get(configuration);
    //这里指定按照指定的副本数进行创建，不指定，按照默认的个数来创建
    FSDataOutputStream fsDataOutputStream = fileSystem.create(path, (short) 2);
    fsDataOutputStream.write("iiiiiiiiiiii".getBytes());
    fsDataOutputStream.close();

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

### 2.2, replicationAndBlockSize

* 注意：块大小配置文件中默认是1M，如果要更改这个值，这个值必须是512的倍数

```java
//指定副本数和块大小
//注意：如果什么都不做，直接运行，报错：Specified block size is less than configured minimum value (dfs.namenode.fs-limits.min-block-size): 5 < 1048576，因为要有最小的分块限制：必须要大于1M，如果要更改这个值，这个值必须是512的倍数，因为这是校验和的限制
@Test
public void replicationAndBlockSize(){

  File file = new File("/Users/ivanl001/Desktop/00-bigData/00-data/input/ivanl001copy.txt");

  Configuration configuration = new Configuration();
  Path path = new Path("/user/ivanl001/test04.txt");

  try {

    FileInputStream fileInputStream = new FileInputStream(file);

    FileSystem fileSystem = FileSystem.get(configuration);
    //20k块大小，备份块数是4，但是实际上我只有三个数据节点，所以最后还是只会存储3个备份数据, 20k
    FSDataOutputStream fsDataOutputStream = fileSystem.create(path, true, 1024, (short) 2, 1024*20);

    byte[] bytes = new byte[1024];
    int len = 0;
    while ((len = fileInputStream.read(bytes)) != -1) {
      fsDataOutputStream.write(bytes, 0, len);
    }

    fsDataOutputStream.close();
    fileInputStream.close();

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



