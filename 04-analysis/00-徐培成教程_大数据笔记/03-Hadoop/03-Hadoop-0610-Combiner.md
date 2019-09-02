input ->  reader  ->  map  ->  partition  ->   combiner  -> reduce

reader这里是负责如果切片没有到行尾，则判断读取到行尾

先分区，后combiner，因为combiner是map端在分区上的聚合。所以肯定是先分区后在combiner的





* *Combiner其实就是Reducer，只不过是map端的聚合，可以在shuffle之前直接聚合*
* *但是也不是所有的reduce可以做成combiner的，比如求最大和最小的差值或者求所有值的平均值，就不能用combiner等*
* 这个是我们自定义的Combiner

```java
package im.ivanl001.bigData.Hadoop.A07_combiner;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-24 15:33
 * #description : 本地聚合
 **/
public class IMCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        System.out.println("--------------------------------combiner--------------------------------------"+ Thread.currentThread().getName());
        System.out.println();

        int count = 0;
        for (IntWritable intWritable : values) {
            count = count + intWritable.get();
        }
        context.write(key, new IntWritable(count));
    }


}
```


* 然后把Combiner设置到job中去。

```java
package im.ivanl001.bigData.Hadoop.A07_combiner;

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

    /*
     * mapper过程之后产生的文件的命名中是***-m-00000*什么的，m代表是mapper，后面的数字代表是分区
     * reducer过程之后产生的文件的命名中是***-r-00000*什么的，r代表的是reducer，后面的数字代表的也是分区
     * 如果设置三个reducer，在没有重写分区函数的情况下，会有三个r，也就会有三个输出文件，因为一个reducer会有一个输出文件
     * 如果重写了分区函数，其实也会生成三个文件，但是只有算法中有指向的才会有内容，其他的就是空文件了
     * */
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

            //--------------------------------这里设置输出格式类------------------------------------
            //这个是设置输出格式为------------序列文件输出格式-------------，我这里并不想保存序列文件，所以这里就不设置
            //wordcountJob.setOutputFormatClass(SequenceFileOutputFormat.class);


            //2,设置作业输入
            //这句话可以不加，因为默认就是文本输入格式
            wordcountJob.setInputFormatClass(TextInputFormat.class);
            FileInputFormat.addInputPath(wordcountJob, new Path(inputFileStr));

            //3，设置mapper
            wordcountJob.setMapperClass(IMWordCountMapper.class);
            wordcountJob.setMapOutputKeyClass(Text.class);
            wordcountJob.setMapOutputValueClass(IntWritable.class);



            //----中间设置一下分区函数
            wordcountJob.setPartitionerClass(IMPartitioner.class);

            //combiner是在分区之后进行的，所以在这里设置combiner
            //其实这个combiner的内容和reducer是一样的，这里直接设置成IMWordCountReducer这个类也是可以的，为了区分明显，我这里才重新复制改名字使用
            wordcountJob.setCombinerClass(IMCombiner.class);



            //4, 设置reducer
            wordcountJob.setReducerClass(IMWordCountReducer.class);
            //每个reduce会产生一个输出结果或者输出文件，这里设置一个reduce
            wordcountJob.setNumReduceTasks(2);//设置reducer的个数，如果是0就是不需要r
            //设置输出的key和value的类型
            wordcountJob.setOutputKeyClass(Text.class);
            wordcountJob.setOutputValueClass(IntWritable.class);

            //5, 设置输出
            wordcountJob.setOutputValueClass(FileOutputFormat.class);
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