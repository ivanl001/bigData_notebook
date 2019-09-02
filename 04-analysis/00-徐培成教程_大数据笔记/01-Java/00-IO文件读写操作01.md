---
title: 180916-IO文件读写操作01
tags: java
---

# IO文件读写操作01: 字节输入输出流

> 本文涉及的代码源文件地址：https://github.com/ivanl001/java_train



## -1, StringBuffer和StringBuilder

> StringBuffer是线程安全的可变字符串，继承AbstractStringBuilder，默认创建是16位长度的字符串

> StringBuiler的话不能保证线程同步，也是可变字符串，但是相对StringBuffer而言速度更快，因为不需要每次进行更改的时候都进行加锁，所以如果是同一线程，显然StringBuilder更推荐使用

## 0, File

`File是一个文件处理的类，直接继承于Object，构造File后，可通过其特定方法进行创建文件：createNewFile，创建文件夹：mkdir，创建多级文件夹mkdirs, 删除文件或者文件夹:delete,`

*下面这个例子演示了File类的基本操作*

<!-- more -->


```java
import java.io.File;
import java.io.FileFilter;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 12:24
 * #description : 通过File来遍历文件夹，然后计算文件夹中所有拍过的照片的数量
 **/
public class FileAbout {

    public static void main(String[] args) {

        File theRootD3100 = new File("/Volumes/张不二-2T/02-Hobbies/02-Photograph/03-D3100");
        File theRoot5D4 = new File("/Volumes/张不二-2T/02-Hobbies/02-Photograph/05-EOS 5D mark4");

        //先计算3100的所有的JPG格式照片的个数
        if (theRootD3100.isDirectory()) {
            theJpgAmtInFile(theRootD3100);
        }else{
            System.out.println("theRootD3100 is not directory, and the amt is 0 by default");
        }

        //先计算5D4的所有的JPG格式照片的个数
        if (theRoot5D4.isDirectory()) {
            theJpgAmtInFile(theRoot5D4);
        }else{
            System.out.println("theRoot5D4 is not directory, and the amt is 0 by default");
        }
    }

    static void theJpgAmtInFile(File input) {

        File[] files = input.listFiles(new FileFilter() {

            @Override
            public boolean accept(File pathname) {
                if (pathname.isDirectory()) {
                    return true;
                }else {
                    //是文件，那么需要选择正确后缀名的文件进行返回
                    if (pathname.toString().endsWith(".jpg") || pathname.toString().endsWith(".JPG") ) {
                        return true;
                    }else {
                        return false;
                    }
                }
            }
        });

        for (File file : files) {

            if (file.isDirectory()) {
                theJpgAmtInFile(file);
            }else{
                System.out.println(file);
                //这里写入到文件中，然后从文件的行数读取文件内容
                //这里是文件
            }
        }
    }
}
```

## 1, OutputStream

`这个类是一个超类，抽象类，可以理解为主要有四个方法：close方法，和三个write方法，分别是：write(int b),写出字节， write(byte[] b),写出字节数组，write（byte[] b, int off, int len), 通过一定的偏移写出字节数组 `

### 1.1, FileOutputStream

`本类是OutputStream的继承类，可以直接创建写出到文件中`

*下面的这个演示该类的基本使用*

```java
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 13:41
 * #description : 字节输出流相关内容
 **/
public class FileOutputStreamAbout {

    public static void main(String[] args) {

        byte[] bytes01 = "ABC".getBytes();
        System.out.println(bytes01);

        FileOutputStream fileOutputStream = null;
        try {
            //第二个参数表示追加
            fileOutputStream = new FileOutputStream("test01.txt", true);

            //第一种方法直接写入字节流，这个是根据ASCII进行转码的
            fileOutputStream.write(100);

            //第二种方式
            byte[] bytes  = {99, 98, 97};
            fileOutputStream.write(bytes);

            //第三种方式，这种适合的比较长的内容逐步通过位移进行数据写入
            byte[] bytes1 = {90, 89, 88, 87, 86, 85, 84, 83, 82, 81, 80};
            fileOutputStream.write(bytes1, 1, 5);

            //当然，以码表写入的方式非常不直观，所以我们可以通过把字符串转成码表，然后在写出比较正常
            String str01 = "ivanl001 is the king of world!";
            fileOutputStream.write(str01.getBytes());

            //写入换行符
            fileOutputStream.write("\n".getBytes());

            fileOutputStream.close();

        } catch (IOException e) {
            e.printStackTrace();
        }finally {
          if (fileOutputStream != null) {
              try {
                  fileOutputStream.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
        }
    }
}
```

### 1.2, BufferedOutputStream(推荐)

```java
package InputStreamAndOutputStream;

import java.io.BufferedOutputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 22:10
 * #description : 字节输出流的高效能的一个子类
 **/
public class BufferedOutputStreamAbout {

    public static void main(String[] args) {
        BufferedOutputStream bufferedOutputStream = null;

        try {
            FileOutputStream fileOutputStream = new FileOutputStream("test04.txt", true);
            bufferedOutputStream = new BufferedOutputStream(fileOutputStream);

            bufferedOutputStream.write(100);

            byte[] bytes  = {'1','2','a','b','/'};
            bufferedOutputStream.write(bytes);

            bufferedOutputStream.write(bytes, 4, 1);
            bufferedOutputStream.write("\n".getBytes());

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bufferedOutputStream != null) {
                try {
                    bufferedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 2, InputStream

`这个类是一个超类，抽象类，可以理解为主要有四个方法：close方法，和三个read方法，分别是：read(int b),写出字节， read(byte[] b),写出字节数组，read(byte[] b, int off, int len), 通过一定的偏移写出字节数组 `

### 2.1, FileInputStream

> 本类是InputStream的继承类，可以直接读取文件中
> 读取文件的时候需要注意一个点， 就是如果读取字节数组的话，最后的字节数组转成字符串的时候，最好需要按照偏移量进行转，不然的话， 有可能造成最后的一个字符数组出现小问题，具体看下面代码

```java

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 14:09
 * #description :
 **/
public class FileInputStreamAbout {

    public static void main(String[] args) {
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream("test01.txt");

            //这种方式是一个字节一个字节读取，但是这种方式每个字节读一次相对来讲不是很划算
            /*int b = 0;
            while ((b = fileInputStream.read()) != -1) {
                //这里是直接输出ascii值
                //System.out.println(b);
                //如果输出ascii值，太不值观，所以下面可以输出字符串
                System.out.println((char)b);
            }*/


            //一次读取1024字节，⚠️这种方式是有问题的需要注意
            //这种方式会好一点， 但是有一个问题，就是最后一次打印字节组的时候可能会因为最后一组的字节个数小于设定的字节个数,这样子最后一组数据就会有问题，导致读取的数据最有一组会有问题
            /*byte[] bytes = new byte[4];
            int b = 0;
            while ((b = fileInputStream.read(bytes)) != -1) {
                System.out.print(new String(bytes));
                //System.out.println(b);
            }*/


            //那么就引出了最后一种方式
            byte[] bytes01 = new byte[5];
            int b = 0;
            while ((b = fileInputStream.read(bytes01)) != -1) {
                System.out.print(new String(bytes01, 0, b));//这个按照偏移量进行打印的
                //System.out.println(b);
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 2.2, BufferedInputStream**(推荐)**

> 这个是对于FileInputStream的高效能

```java
package InputStreamAndOutputStream;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 22:18
 * #description :
 **/
public class BufferedInputStreamAbout {

    public static void main(String[] args) {
        BufferedInputStream bufferedInputStream = null;

        try {
            FileInputStream fileInputStream = new FileInputStream("test04.txt");
            bufferedInputStream = new BufferedInputStream(fileInputStream);

            //这里就不再一个字节一个字节读取了， 没啥意义
            int offset = 0;
            byte[] bytes = new byte[3];
            while ((offset = bufferedInputStream.read(bytes)) != -1) {
                System.out.print(new String(bytes, 0, offset));
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bufferedInputStream != null) {
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



## 3, copy案例01,02: 单字节拷贝和字节数组拷贝

> 利用FileInputStream和FileOutputStream进行文件的copy

### 1,第一种文件拷贝，一个字节一个字节进行拷贝

```java
import java.awt.*;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Date;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 15:11
 * #description : 读一个字节，写入一个字节的方式来进行复制
 *
 * //小型文件用这个没问题，但是一旦文件比较大，可以尝试一下十几兆的文件，估计要拷贝n多时间,我尝试了一下， 20M左右的视频，拷贝了200秒左右
 *
 **/
public class Copy01 {

    public static void main(String[] args) {

        //1，首先因为变量作用域的原因进行null初始化
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;

        Long i = 0L;

        try {
            //2，对输入输出流进行赋值
            fileInputStream = new FileInputStream("/Users/ivanl001/Desktop/a/01.mp4");
            fileOutputStream = new FileOutputStream("/Users/ivanl001/Desktop/b/01.mp4", true);

            Long startTime = System.currentTimeMillis();

            //3，先一个字节一个字节对方式读到文件
            int b = 0;
            while ((b = fileInputStream.read()) != -1) {
                i+=1;
                System.out.println("写入文件中---" + i);
                //4,把读到对字节写入到目标文件中
                fileOutputStream.write(b);
            }

            Long endTime = System.currentTimeMillis();

            System.out.println(endTime-startTime);
            Long cost = (endTime-startTime)/1000;
            System.out.println("主要消耗时长是:"+cost);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fileOutputStream != null) {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("copy finished");
        }
    }
}
```

### 2,上面那种一个字节一个字节拷贝实在是太慢，效率低，下面这种是可以设定缓存，然后一次性读取存储制定大小的容量进行copy，当然这种copy也还可以，但是后面还有更好到buffer类的copy，效率会更高

```java
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-16 15:44
 * #description : 1024字节缓存复制，加快效率
 *
 * 同样的， 如果是1024字节组进行读取和存储的话，20M左右的视频文件大概需要耗时480毫秒左右
 *
 **/
public class Copy02 {

    public static void main(String[] args) {

        //1，首先因为变量作用域的原因进行null初始化
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;

        Long i = 0L;

        try {

            //2，对输入输出流进行赋值
            fileInputStream = new FileInputStream("/Users/ivanl001/Desktop/a/01.mp4");
            fileOutputStream = new FileOutputStream("/Users/ivanl001/Desktop/c/01.mp4", true);

            Long startTime = System.currentTimeMillis();

            //3，先一个字节一个字节对方式读到文件
            int b = 0;
            byte[] bytes = new byte[1024];
            while ((b = fileInputStream.read(bytes)) != -1) {
                i+=1;
                System.out.println("写入文件中---" + i);
                //4,把读到对字节写入到目标文件中

                //b是偏移量，byte是1024个字节数组，最后一组不一定全部是需要的
                fileOutputStream.write(bytes, 0, b);
            }

            Long endTime = System.currentTimeMillis();

            System.out.println(endTime-startTime);
            Long cost = (endTime-startTime)/1000;
            System.out.println("主要消耗时长是:"+cost);
          
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fileOutputStream != null) {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("copy finished");
        }
    }
}
```





## 4,copy案例03: Buffered拷贝

> 利用BufferedOutputStream和BufferedInputStream实现高效的copy

```java
package InputStreamAndOutputStream;

import java.io.*;

/**
 * #author      : ivanl001
 * #creator     : 2018-09-17 08:56
 * #description : 通过高效率的方式实现复制
 *
 * 同样的， 如果是1024字节组缓存方式进行读取和存储的话，20M左右的视频文件大概需要耗时80毫秒左右，比直接用字节组要快十倍左右
 **/
public class Copy04_buffered {

    public static void main(String[] args) {
        BufferedInputStream bufferedInputStream = null;
        BufferedOutputStream bufferedOutputStream  = null;
        try {
            //1，首先创建对象和设置追加等
            FileInputStream fileInputStream = new FileInputStream("/Users/ivanl001/Desktop/a/01.mp4");
            bufferedInputStream = new BufferedInputStream(fileInputStream);
            FileOutputStream fileOutputStream = new FileOutputStream("/Users/ivanl001/Desktop/e/01.mp4", true);
            bufferedOutputStream = new BufferedOutputStream(fileOutputStream);

            Long startTime = System.currentTimeMillis();

            //2,1024字节缓存读取
            int offset = 0;
            byte[] bytes = new byte[1024];
            while ((offset = bufferedInputStream.read(bytes)) != -1) {
                bufferedOutputStream.write(bytes, 0, offset);
            }

            Long endTime = System.currentTimeMillis();

            System.out.println(endTime-startTime);
            Long cost = (endTime-startTime)/1000;
            System.out.println("主要消耗时长是:"+cost);
            System.out.println("copied finished!");

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bufferedInputStream != null) {
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bufferedOutputStream != null) {
                try {
                    bufferedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```