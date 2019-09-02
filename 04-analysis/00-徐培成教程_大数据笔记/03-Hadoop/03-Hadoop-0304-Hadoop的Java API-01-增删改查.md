## 01, Hadoop_01_URL


```java
//通过FsUrlStreamHandlerFactory这种方式不推荐，因为setURLStreamHandlerFactory这个方法只能调用一次，如果需要再设置其他的处理器就歇菜了
@Test
public void Hadoop_01_URL(){

  //注意：默认情况下是没有hdfs协议的，需要写如下一行代码添加hdfs协议解析
  //这一行可以加到静态代码块中
  URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());

  InputStream in;
  try {

    //1，第一种方式
    //in = new URL("hdfs://centos02:8020/user/ivanl001test/zhang.txt").openConnection().getInputStream();

    //2, 第二种方式
    in = new URL("hdfs://centos02:8020/user/ivanl001/ivanl001.txt").openStream();

    //1，第一种方式
    //IOUtils.copyBytes(in, System.out, 4096, false);

    //2, 第二种方式
    byte[] bytes = new byte[in.available()];
    in.read(bytes);
    in.close();
    String resultStr = new String(bytes);

    System.out.println("-------------------------------------Hadoop_01_URL--------------------------------------------");
    System.out.println(resultStr);

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## 02, Hadoop_02_fs_read

```java
//读取文件00
@Test
public void Hadoop_02_fs_read(){

  try {

    //0, 首先是配置对象和uri
    Configuration configuration = new Configuration();
    configuration.set("fs.defaultFS", "hdfs://centos02:8020");

    //1, 创建fs对象
    FileSystem fs = FileSystem.get(configuration);
    InputStream in = fs.open(new Path("/user/ivanl001/ivanl001.txt"));

    //1，第一种方式
    //IOUtils.copyBytes(in, System.out, 4096, false);

    //2, 第二种方式
    byte[] bytes = new byte[in.available()];
    in.read(bytes);
    in.close();
    String resultStr = new String(bytes);

    System.out.println("-------------------------------------Hadoop_02_fs_read--------------------------------------------");

    System.out.println(resultStr);

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## 03, Hadoop_03_fs_read01


```java
//读取文件01，推荐用这个吧，比较直观
@Test
public void Hadoop_03_fs_read01(){

  try {
    //0, 首先创建uri
    URI uri = URI.create("hdfs://centos02:8020/user/ivanl001/ivanl001.txt");

    //1, 创建fs对象
    //如果在这里直接把uri给fs，那么fs获取链接的时候会自动设置成hdfs，也就不需要设置中再设置了
    FileSystem fs = FileSystem.get(uri, new Configuration());
    InputStream in = fs.open(new Path(uri));

    //1，第一种方式
    //IOUtils.copyBytes(in, System.out, 4096, false);

    //2, 第二种方式
    byte[] bytes = new byte[in.available()];
    in.read(bytes);
    in.close();
    String resultStr = new String(bytes);

    System.out.println("-------------------------------------Hadoop_02_fs_read01--------------------------------------------");

    System.out.println(resultStr);

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## 04, Hadoop_04_fs_mkdir


```java
//创建文件夹
@Test
public void Hadoop_04_fs_mkdir(){

  try {
    //1, 创建uri，知道是什么协议，然后给出路径，方便后续使用
    URI uri = URI.create("hdfs://centos02:8020/");
    Path path = new Path("/user/ivanl001/ivanl000");

    //2，得到文件系统
    FileSystem fs = FileSystem.get(uri, new Configuration());

    //3，mkdirs创建文件夹
    boolean yes = fs.mkdirs(path);
    System.out.println(yes);

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## 05, Hadoop_05_fs_ls

```java
//显示文件夹下文件列表
    @Test
    public void Hadoop_05_fs_ls(){

        try {
            //1, 创建uri，知道是什么协议，然后给出路径，方便后续使用
            URI uri = URI.create("hdfs://centos02:8020/");
            Path path = new Path("/user/ivanl001");

            //2，得到文件系统
            FileSystem fs = FileSystem.get(uri, new Configuration());

            //3，listFiles得到目录下文件列表
            RemoteIterator<LocatedFileStatus> iterator = fs.listFiles(path, true);
            while (iterator.hasNext()) {
                LocatedFileStatus fileStatus = iterator.next();
                System.out.println(fileStatus.getPath());
            }

        } catch (IOException e) {
            e.printStackTrace();
        }


    }
```



## 06, Hadoop_06_fs_put

```java
//写入文件
//这个也就是写入,这个后续会有解析一下，方便学习代码
@Test
public void Hadoop_06_fs_put() throws InterruptedException {

  try {
    //1, 创建uri，知道是什么协议，然后给出路径，方便后续使用
    URI uri = URI.create("hdfs://centos02:8020");
    Path path = new Path("/user/ivanl001/wahaha_new01.txt");

    //2，得到文件系统
    FileSystem fs = FileSystem.get(uri, new Configuration());

    //3, 写入文件
    // 如果是从本地上传文件：fs.copyFromLocalFile(new Path(""), path);
    //这里是直接覆盖哦
    //FSDataOutputStream fsDataOutputStream = fs.create(path);
    //这里是不覆盖,如果文件已经存在会报错：org.apache.hadoop.fs.FileAlreadyExistsException
    FSDataOutputStream fsDataOutputStream = fs.create(path, false);

    //⚠️：这里默认是没有换行符的哦
    fsDataOutputStream.write("orld!ivanl0001 is the king of world!\n".getBytes());
    fsDataOutputStream.write("ivanl0002 is the king of world!".getBytes());

    //注意：如果这里不关闭是不会写入的，当然在主程序结束的时候，流会自动关闭写出，如果写了下面的循环，这里是必须要有下面一句，要不然是没法输出的。
    //fsDataOutputStream.close();

  } catch (IOException e) {
    e.printStackTrace();
  }

  /*while (true) {
            Thread.sleep(1000);
        }*/

}
```



## 07, Hadoop_07_fs_append

```java
//追加文件
@Test
public void Hadoop_07_fs_append() throws InterruptedException {

  try {
    //1, 创建uri，知道是什么协议，然后给出路径，方便后续使用
    URI uri = URI.create("hdfs://centos02:8020");
    Path path = new Path("/user/ivanl001/wahaha_new01.txt");

    //2，得到文件系统
    FileSystem fs = FileSystem.get(uri, new Configuration());

    //3, 追加写入文件
    FSDataOutputStream fsDataOutputStream = fs.append(path);

    //⚠️：这里默认是没有换行符的哦
    fsDataOutputStream.write("hhhhhhhhhhhhh\n".getBytes());
    fsDataOutputStream.write("ivanl0002 is the king of world!".getBytes());

    //注意：如果这里不关闭是不会写入的，当然在主程序结束的时候，流会自动关闭写出，如果写了下面的循环，这里是必须要有下面一句，要不然是没法输出的。
    //fsDataOutputStream.close();

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## 08, Hadoop_08_fs_delete


```java
//删除文件
@Test
public void Hadoop_08_fs_delete(){

  try {
    //1, 创建uri，知道是什么协议，然后给出路径，方便后续使用
    URI uri = URI.create("hdfs://centos02:8020");
    Path path = new Path("/user/ivanl001/wahaha_new01.txt");

    //2，得到文件系统
    FileSystem fs = FileSystem.get(uri, new Configuration());
    boolean yes = fs.delete(path, false);
    System.out.println(yes);

  } catch (IOException e) {
    e.printStackTrace();
  }
}
```