* `参考  265/756  [O'REILLY]Hadoop.The.Definitive.Guide.4th.Edition.2015.3`
* 上面书中还有其他的输入格式



*多输入意思是比如你的文件有SequenceFile和txt文件，那么都需要相同的分析。那么就可以输入到一个Job中，然后根据不同的输入设定不同的mapper，reduce用相同的处理*


```java
//对不同的输入设定不同的mapper
MultipleInputs.addInputPath(wordcountJob, new Path("/Users/ivanl001/Desktop/bigData/input01/text01.txt"), TextInputFormat.class, IMWordCountMapper.class);
MultipleInputs.addInputPath(wordcountJob, new Path("/Users/ivanl001/Desktop/bigData/input01/seq01.seq"), SequenceFileInputFormat.class, IMWordCountMapperForSeq.class);
```