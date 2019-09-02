```shell
# ---auto.offset.reset---
# earliest 
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费 
# latest 
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据 
# none 
topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
```



1, spark把offset默认存在内存



2,  app重启后，内存中offset数据就没了，那么这个时候的消费位置根据参数auto.offset.reset决定



3, 上面的方式明显不行，因为我们必须要保留offset数据

管理offset有两种方式：

3.1, 不修改代码的情况下，也就是服务意外down机，使用checkpoint机制，可以保证在服务down掉的情况下重新启动，可以正常消费

3.2, 如果需要修改代码，需要手动管理offset，可以把offset保存在zookeeper中或者mysql或者redis中去



