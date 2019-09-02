#### 9, window

##### 9.1, window分类

###### 第一种分类：

* timewindow

* countwindow

###### 第二种分类：

* tumbling windows：滚动窗口（没有重叠）

* sliding windows：滑动窗口（有重叠）

* session windows: 会话窗口（类似web编程里面的session， 以不活动间隙作为分割）

  

![image-20190515171835301](assets/image-20190515171835301.png)



![image-20190515172057283](assets/image-20190515172057283.png)



![image-20190515172113784](assets/image-20190515172113784.png)

##### 9.2, window聚合分类

###### 增量聚合

![image-20190515172432319](assets/image-20190515172432319.png)

![image-20190515172458403](assets/image-20190515172458403.png)



###### 全量聚合

![image-20190515172746544](assets/image-20190515172746544.png)

![image-20190515173412726](assets/image-20190515173412726.png)



#### 10, time

##### 10.1, event time

* 事件产生事件，比如说日志产生时间

##### 10.2, ingest time

* 事件进入flink的时间

##### 10.3, processing time

* 事件被处理的时间

![image-20190515173808549](assets/image-20190515173808549.png)



#### 11, watermark

![image-20190515174457994](assets/image-20190515174457994.png)

![image-20190515174720193](assets/image-20190515174720193.png)

具体的使用方法：

![image-20190515180340601](assets/image-20190515180340601.png)

![image-20190515180227788](assets/image-20190515180227788.png)