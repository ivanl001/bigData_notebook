## 1, oozie简单认识



### 1.1, workflow.xml

* 流程控制标签start to等
* 动作标签action等

### 1.2, job.properties

* namenode=hdfs://nameservice1:8020
* jobTracker=nameservice1:8032
* queuename=default
* examplesRoot=examples
* 下面这个是workflow.xml的路径
* oozie.wf.application.path=${namenode}/user/...



