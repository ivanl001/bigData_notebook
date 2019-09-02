*这个输入方式主要是可以直接用来连接数据库进行数据处理。但是感觉用处不大吧*

## 1, App
*这里可以从数据库输入，也可以一并输出到数据库中，见代码中注释*
```java
package im.ivanl001.bigData.Hadoop.A15_MR_database;


import im.ivanl001.bigData.model.IMWCDBWritable;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.db.DBConfiguration;
import org.apache.hadoop.mapreduce.lib.db.DBInputFormat;
import org.apache.hadoop.mapreduce.lib.db.DBOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-30 16:29
 * #description :
 **/
public class IMDBApp {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {


        Path outPath = new Path("/Users/ivanl001/Desktop/bigData/output012");


        //0, 准备配置文件
        Configuration configuration = new Configuration();

        //这里因为文件存在，总是需要删除，麻烦，所以直接程序自动删除
        FileSystem.get(configuration).delete(outPath);


        //1, 创建作业，并设置基本内容
        Job job = Job.getInstance(configuration);
        job.setJobName("databaseJob");
        job.setJarByClass(IMDBApp.class);


        //2, 设置作业输入并配置数据库信息
        job.setInputFormatClass(DBInputFormat.class);

        //2.1, 配置数据库的相关配置
        String driverClass = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/test";
        String username = "root";
        String password = "ivanl48265";
        //这里需要指定config，就是要把设置信息配置到config里面去
        //注意：这里的config不能直接设置成上面创建的那个，而需要从job中获取
        DBConfiguration.configureDB(job.getConfiguration(), driverClass, url, username, password);
        DBInputFormat.setInput(job, IMWCDBWritable.class, "select id,words from tb_wc", "select count(*) from tb_wc");


        //可以通过这种方式直接往数据库中写入，不过我这里为了简便，也就没有输出出去，直接输出文本了，请看下面步骤5
        //DBOutputFormat.setOutput(job,"stats","word","c");


        //3, 设置mapper
        job.setMapperClass(IMDBMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        //4，设置reducer
        job.setReducerClass(IMDBReducer.class);
        job.setNumReduceTasks(2);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //5, 设置输出
        FileOutputFormat.setOutputPath(job, outPath);

        //6,提交，开始处理
        job.waitForCompletion(false);

    }
}
```

## 2, Mapper

```java
package im.ivanl001.bigData.Hadoop.A15_MR_database;


import im.ivanl001.bigData.model.IMWCDBWritable;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-30 16:20
 * #description :
 **/
public class IMDBMapper extends Mapper<LongWritable, IMWCDBWritable, Text, IntWritable> {

    @Override
    protected void map(LongWritable key, IMWCDBWritable value, Context context) throws IOException, InterruptedException {

        String str = value.getWords();
        System.out.println("key:" + key + ", value:" + str);

        String[] splits = str.split(" ");

        Text text = new Text();
        IntWritable count = new IntWritable();

        for (String word : splits) {
            text.set(word);
            count.set(1);

            context.write(text, count);
        }
    }
}
```

## 3, Reducer

```java
package im.ivanl001.bigData.Hadoop.A15_MR_database;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-30 16:27
 * #description :
 **/
public class IMDBReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for (IntWritable once : values) {
            count += 1;
        }
        context.write(key, new IntWritable(count));
    }
}
```



