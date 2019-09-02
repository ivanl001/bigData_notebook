* stage是并行的task的集合, 在一个stage中的task都有相同的shuffle依赖、计算相同的分区函数
* stage有两种：ShuffleMapStage和ResultStage
* 每次shuffle一次就会重新形成一个新的stage

* Job是stage的集合，一个作业包括一个或者多个stage

