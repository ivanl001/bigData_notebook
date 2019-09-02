## 0, 基本概念
### 01，Map
* map中的输入的key是这一行的偏移量，所以总是LongWritable

### 02，Reduce
* reduce的个数决定map分区的个数

### 03，Shuffle
* 这个是map和reduce之间的数据分发的过程

## 1，wordCount案例

### 01，需要处理的文本
```text
ivanl is the king of world!
Do you know why?
Because he is awesome and 
he can do any crazy thing in the world!
```

### 02, mapper

```java
package im.ivanl001.bigData.Hadoop.WordCount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-20 19:23
 * #description : mapper
 **/
public class IMWordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        System.out.println("key:" + key + ",value:" + value);
        String[] splitStr = value.toString().split(" ");
        Text outText = new Text();
        IntWritable outInt = new IntWritable();
        for (String str : splitStr) {
            outText.set(str);
            outInt.set(1);
            //这里是意思就是把每个单词拼成(zhang,1), (li, 1), (dan, 1)类似的格式传给reduce
            context.write(outText, outInt);
        }
    }
}
```

### 03，reducer

```java
package im.ivanl001.bigData.Hadoop.WordCount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-20 19:30
 * #description : reducer
 **/
public class IMWordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {


    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for (IntWritable intWritable : values) {
            count = count + intWritable.get();
        }
        context.write(key, new IntWritable(count));
    }
}
```

### 04，app

```java
package im.ivanl001.bigData.Hadoop.WordCount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-20 19:35
 * #description : wordcount
 **/
public class IMWordCountApp {

    public static void main(String[] args) {

        try {

            if (args.length != 2) {
                System.out.println("参数个数有误！");
                return;
            }

            //"/users/ivanl001/Desktop/bigData/input/zhang.txt"
            String inputFileStr = args[0];
            String outputFolderStr = args[1];

            //0，创建配置对象，以修正某些配置文件中的配置
            Configuration configuration = new Configuration();
            //这里一旦设置单机版就会出错，而且不能有core-default.xml文件，这个文件中一旦配置也会有问题，不知道为啥，先过
            //configuration.set("fs.defaultFS", "file:///");
            //configuration.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
            //configuration.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());

            //这里因为文件存在，总是需要删除，麻烦，所以直接程序自动删除
            FileSystem.get(configuration).delete(new Path(outputFolderStr));

            //1，创建作业
            Job wordcountJob = Job.getInstance(configuration);
            wordcountJob.setJobName("wordcountApp");

            //2,设置作业输入
            //这句话可以不加，因为默认就是文本输入格式
            wordcountJob.setInputFormatClass(TextInputFormat.class);
            FileInputFormat.addInputPath(wordcountJob, new Path(inputFileStr));

            //3，设置mapper
            wordcountJob.setMapperClass(IMWordCountMapper.class);
            wordcountJob.setMapOutputKeyClass(Text.class);
            wordcountJob.setMapOutputValueClass(IntWritable.class);

            //4, 设置reducer
            wordcountJob.setReducerClass(IMWordCountReducer.class);
            //每个reduce会产生一个输出结果或者输出文件，这里设置一个reduce
            wordcountJob.setNumReduceTasks(1);
            //设置输出的key和value的类型
            wordcountJob.setOutputKeyClass(Text.class);
            wordcountJob.setOutputValueClass(IntWritable.class);

            //5, 设置输出
            //wordcountJob.setOutputValueClass(FileOutputFormat.class);
            FileOutputFormat.setOutputPath(wordcountJob, new Path(outputFolderStr));

            //6，提交，开始处理
            wordcountJob.waitForCompletion(false);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 05, 最终的处理的结果如下

```
Because	1
Do	1
and	1
any	1
awesome	1
can	1
crazy	1
do	1
he	2
in	1
is	2
ivanl	1
king	1
know	1
of	1
the	2
thing	1
why?	1
world!	2
you	1
```

