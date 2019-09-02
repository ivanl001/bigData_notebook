

## 1, hadoop输入格式

| -                                        | -                                        |
| ---------------------------------------- | ---------------------------------------- |
| FileInputFormat                          |                                          |
| -----------------TextInputFormat         | hadoop默认，位移做key, 一行内容做value   |
| -----------------KeyValueTextInputFormat | 会根据空格把一行切开, 作为key, value输入 |
| -----------------SequenceFileInputFormat | 序列文件的输入格式                       |
|                                          |                                          |
| DBInputFormat                            | 数据库读如格式                           |
|                                          |                                          |



2, hadoop输出格式

| -                                         | -                                                    |
| ----------------------------------------- | ---------------------------------------------------- |
| FileOutputFormat                          | hadoop默认应该，写出文件，应该可以指定压缩格式什么的 |
| -----------------TextOutputFormat         | 继承FileOutputFormat，直接写文本文件                 |
| -----------------SequenceFileOutputFormat | 输出序列文件，和序列输入文件匹配                     |
| -----------------MapFileOutputFormat      |                                                      |
| -----------------ParquetOutputFormat      |                                                      |
|                                           |                                                      |
| DBOutputFormat                            | 数据库写出格式                                       |
|                                           |                                                      |
|                                           |                                                      |





```java
//5.1, 文件格式输出，默认是这种
//wordcountJob.setOutputFormatClass(FileOutputFormat.class);
//FileOutputFormat.setOutputPath(wordcountJob, new Path(outputFolderStr));

//5.2, 文本格式输出
wordcountJob.setOutputFormatClass(TextOutputFormat.class);
TextOutputFormat.setOutputPath(wordcountJob, new Path(outputFolderStr));
```

