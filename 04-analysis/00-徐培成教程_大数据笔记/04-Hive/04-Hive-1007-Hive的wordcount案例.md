## 1, Hive函数：wordcount的应用

### 01，hive的wordcount算法

* 首先创建表，存储需要计算的数据，注意：这里指定的是字段的分隔符
  
```mysql
create table word_count (line string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
```



* 加载数据
  
```mysql
# 加载本地数据
load data local inpath '/root/share/word_count.txt' into table word_count;
# 加载hdfs数据
load data inpath '/root/share/word_count.txt' into table word_count;
```



* 使用内置函数进行计算
  * 首先把每一行切成数组
    
  ```mysql
  select split(line, ' ') from word_count;
  ```
  
  
  
  * 再通过explode函数把数组炸开
    
  ```mysql
  select  explode(split(line, ' ')) from word_count;
  ```
  
  
  
  * 之后通过子查询最终确定个数, 这个逻辑有点绕，其实简单来讲，就是把单个单词的集合当作t, 每个单词就是t中的word，我们通过对word分组，算出这组的总数就是了，对吧
    
  ```mysql
  select t.word, count(1) from ((select  explode(split(line, ' ')) as word from word_count) as t) group by t.word;
  ```
  
  
  
  * 最后还需要把结果记录下来，只是打印在控制台上是不行的, 这里就是相当于直接创建一个表，把刚才的结果复制进去，也就是create ... as ...就可以了
    
  
  ```mysql
  create table wc_result as select t.word, count(*) from ((select  explode(split(line, ' ')) as word from word_count) as t) group by t.word;
  ```
  
  

## 2, 重新做了一遍

* 01, 创建表格
  *这个表字段就是line，也就是一个字段存入一样数据。我们这里就一个字段，所以字段分隔符作用不大哈。行分隔符不需要指定，在linux上就是\n*
  
```mysql
create table word_count_new(line string) row format delimited fields terminated by ',';
```



* 02, 数据如下, 文件名是wordLines.txt
```mysql
ivanl001 is the king of world!
zhangbuer is awesome.
I like camera.
```



* 03, 加载数据到hive表
  
```mysql
load data local inpath '/root/ivanl001/test/wordLines.txt' into table word_count_new;
```



* 04, 首先是把行分隔成单词数组,如果有多行会有多个数组,然后把所有的数组炸裂成一个一个单词
  
```mysql
select explode(split(line, " ")) as word from word_count_new
```



* 05, 然后按照单词分组即可
  * 这里的意思大概就是把04中的那个结果做成一个视图t(可以理解成另外一张表)，从t中分组查找就好了。t中的字段就是一个word*

```mysql
select t.word,count(*) from ((select explode(split(line, " ")) as word from word_count_new) as t) group by t.word
```





