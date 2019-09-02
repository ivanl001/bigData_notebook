*有一点需要注意：在链条化中，可以有很多很多的Mapper，因为Mapper是在一个节点上进行的。但是Reducer一个Job中只能有一个，因为reducer涉及到shuffle，也就是很多节点上的数据，所以不能有多个reducer*

* 我们这里链条化的案例是这样的：
* IMWordCountMapper负责map成key value对
* IMWordCount_M_chain01负责删除某些不符合要求的kv对
* IMWordCountReducer负责进行加和
* IMWordCount_R_chain01负责对结果再进行加上某些值。这里这个链条依然是Mapper哈，具体看代码：

## 1，IMWordCountApp
```java
package im.ivanl001.bigData.Hadoop.A14_MR_chain;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.chain.ChainMapper;
import org.apache.hadoop.mapreduce.lib.chain.ChainReducer;
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
//            configuration.set("fs.defaultFS", "file:///");
            //configuration.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
            //configuration.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());

            //这里因为文件存在，总是需要删除，麻烦，所以直接程序自动删除
            FileSystem.get(configuration).delete(new Path(outputFolderStr));

            //1，创建作业
            Job wordcountJob = Job.getInstance(configuration);
            wordcountJob.setJobName("wordcountApp");
            //之前这句没写，就会一直报错，什么mapper类找不到，这里需要注意一下
            wordcountJob.setJarByClass(IMWordCountApp.class);

            //2,设置作业输入
            //这句话可以不加，因为默认就是文本输入格式
            wordcountJob.setInputFormatClass(TextInputFormat.class);
            FileInputFormat.addInputPath(wordcountJob, new Path(inputFileStr));

            //这个链条主要做了两件事情：------------------下面是链条-----------------------
            //1，在M的链条中，删除了hellohello字段
            //2, 在R的链条中，对于超过100的字段进行直接+1000
            //因为我们这里使用的链条，所以就不需要设置mapper和reducer了
            ChainMapper.addMapper(wordcountJob, IMWordCountMapper.class, LongWritable.class, Text.class, Text.class, IntWritable.class, configuration);
            ChainMapper.addMapper(wordcountJob, IMWordCount_M_chain01.class, Text.class, IntWritable.class, Text.class, IntWritable.class, configuration);

            ChainReducer.setReducer(wordcountJob, IMWordCountReducer.class, Text.class, IntWritable.class, Text.class, IntWritable.class, configuration);
            ChainReducer.addMapper(wordcountJob, IMWordCount_R_chain01.class, Text.class, IntWritable.class, Text.class, IntWritable.class, configuration);

            //不过还是需要设置一下reducer的个数
            wordcountJob.setNumReduceTasks(1);


            /*//3，设置mapper
            wordcountJob.setMapperClass(IMWordCountMapper.class);
            wordcountJob.setMapOutputKeyClass(Text.class);
            wordcountJob.setMapOutputValueClass(IntWritable.class);


            //4, 设置reducer
            wordcountJob.setReducerClass(IMWordCountReducer.class);
            //每个reduce会产生一个输出结果或者输出文件，这里设置一个reduce
            wordcountJob.setNumReduceTasks(1);
            //设置输出的key和value的类型
            wordcountJob.setOutputKeyClass(Text.class);
            wordcountJob.setOutputValueClass(IntWritable.class);*/

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

## 2，IMWordCountMapper
```java
package im.ivanl001.bigData.Hadoop.A14_MR_chain;

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

## 3,IMWordCount_M_chain01
```java
package im.ivanl001.bigData.Hadoop.A14_MR_chain;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-25 17:50
 * #description : mapper端的链条1
 **/
public class IMWordCount_M_chain01 extends Mapper<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void map(Text key, IntWritable value, Context context) throws IOException, InterruptedException {

        if (!key.toString().equals("hellohello")) {
            context.write(key, value);
        }
    }
}
```

## 4, IMWordCountReducer

```java
package im.ivanl001.bigData.Hadoop.A14_MR_chain;

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

## 5, IMWordCount_R_chain01

```java
package im.ivanl001.bigData.Hadoop.A14_MR_chain;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-25 17:52
 * #description : Reducer端的链条1
 **/
public class IMWordCount_R_chain01 extends Mapper<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void map(Text key, IntWritable value, Context context) throws IOException, InterruptedException {
        if (value.get() > 100) {
            context.write(key, new IntWritable(value.get() + 1000));
        }else{
            context.write(key, value);
        }
    }
}
```