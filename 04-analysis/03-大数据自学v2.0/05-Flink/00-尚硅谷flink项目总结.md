# 1, 热门商品统计

01, 首先根据商品点击信息把字符串map成模型方便后续处理

02, assignAscendingTimestamps设置事件时间戳

03, filter过滤掉不需要的数据

04, keyBy操作，根据商品id进行分组

05, timeWindow, 然后分完组之后，进行window聚合操作，统计过去一小时商品的点击数量

- 也就是说根据商品id进行聚合

06, .aggregate( new IMCountAggator01(), new IMWindowFunction01() )，真正的聚合处，第一个函数表示预聚合函数，第二个表示预聚合后窗口的输出函数，注意：输出的时候，需要把窗口的endtime加到输出对象中去，方便后续取出来进行定时器的设置

07, .keyBy("windowEnd")，因为上一个窗口输出的时候带有windendtime，所以根据windowendTime进行分组

08, 分组后，进行每个数据的处理，.process(new TopNHotItems01(5))， 内部通过状态进行存储商品点击信息。然后设置定时器，定时器触发的时候，从状态中取出所有的商品点击信息，然后写出到指定的地方即可



```scala
val stream11 =  value.map(line => {
  	val lineSplited = line.split(",")
  	UserBehavior01(lineSplited(0).toLong, lineSplited(1).toLong, lineSplited(2).toInt, 				lineSplited(3), lineSplited(4).toLong)
	})
.assignAscendingTimestamps(_.timestamp * 1000)
.filter(_.behavior == "pv")
//这两个居然上面这个不对，打开就爆红
//.keyBy(_.itemId)
//如果这里是"itemId"，后续的key是Tuple，如果是_.itemId，后续返回的是String，也就是对象
.keyBy("itemId")
.timeWindow(Time.hours(1), Time.minutes(5))
.aggregate( new IMCountAggator01(), new IMWindowFunction01() )     //这个是有预聚合的，而不是等一个窗口所有的数据完整了才做处理
.keyBy("windowEnd")
.process(new TopNHotItems01(5))
.print()


//上面代码中：
//自定义聚合函数
class IMCountAggator01 extends AggregateFunction[UserBehavior01, Long, Long]{
  override def add(value: UserBehavior01, accumulator: Long): Long = {accumulator+1}
  override def createAccumulator(): Long = 0L
  override def getResult(accumulator: Long): Long = accumulator
  override def merge(a: Long, b: Long): Long = {a+b}
}

//自定义实现window函数
class IMWindowFunction01 extends WindowFunction[Long, ItemViewCount01, Tuple, TimeWindow]{
  override def apply(key: Tuple, window: TimeWindow, input: Iterable[Long], out: Collector[ItemViewCount01]): Unit = {
    val itemId = key.asInstanceOf[Tuple1[Long]].f0
    val count = input.iterator.next()
    out.collect(ItemViewCount01(itemId, window.getEnd, count))
  }
}

class TopNHotItems01(topSize: Int) extends KeyedProcessFunction[Tuple, ItemViewCount01, String]{
  //1, 首先要创建状态
  private var itemState : ListState[ItemViewCount01] = _
  
  override def open(parameters: Configuration): Unit = {
    super.open(parameters)

    //2, 开始的时候先注册描述器，方便后续进行查找
    val itemStateDesc = new ListStateDescriptor[ItemViewCount01]("itemState", classOf[ItemViewCount01])

    //3, 获取状态
    itemState = getRuntimeContext.getListState(itemStateDesc)
  }

  override def processElement(value: ItemViewCount01, ctx: KeyedProcessFunction[Tuple, ItemViewCount01, String]#Context, out: Collector[String]): Unit = {
    //4, 加入状态
    itemState.add(value)

    //5, 注册定时器，方便后续进行处理
    ctx.timerService().registerEventTimeTimer(value.windowEnd + 1)
  }

  override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Tuple, ItemViewCount01, String]#OnTimerContext, out: Collector[String]): Unit = {
    super.onTimer(timestamp, ctx, out)
    val allItems : ListBuffer[ItemViewCount01] = ListBuffer()

    //下面需要隐式类型转换
    import scala.collection.JavaConversions._
    for (item <- itemState.get()){
      allItems += item
    }

    val sortedItems = allItems.sortBy(_.count)(Ordering.Long.reverse).take(topSize)

    // 将排名数据格式化，便于打印输出
    val result: StringBuilder = new StringBuilder
    result.append("====================================\n")
    result.append("时间：").append(new Timestamp(timestamp - 1)).append("\n")

    for( i <- sortedItems.indices ){
      val currentItem: ItemViewCount01 = sortedItems(i)
      // 输出打印的格式 e.g.  No1：  商品ID=12224  浏览量=2413
      result.append("No").append(i+1).append(":")
      .append("  商品ID=").append(currentItem.itemId)
      .append("  浏览量=").append(currentItem.count).append("\n")
    }
    result.append("====================================\n\n")

    //状态已经拿到，清理空间
    itemState.clear()
    Thread.sleep(1000)
    out.collect(result.toString())
  }
```



## 2, 实时流量统计

01, 根据apach日志信息把字符串map成模型方便后续处理

02, 因为是访问记录，有可能会网络延迟导致发送时间慢于实际时间，所以事件事件不一定是有顺序的，所以需要设置水位线，来减少误差，设置水位线`assignTimestampsAndWatermarks`

03, `.filter(_.method == "GET")`过滤出需要的内容

04, .keyBy(_.url), 根据页面进行分组

05, 创建窗口`.timeWindow(Time.minutes(1), Time.seconds(5))`

06, 窗口聚合， `.aggregate(new IMPreAggregator(), new IMWindowFunction03())`

07, .keyBy(_.windowEnd)然后根据窗口的结束时间进行分组

08,  `.process(new IMTopNProcessFunction03(3))`, 通过process函数处理每条信息，函数内部有定时器和出发函数。先通过ListState保存状态，然后到达窗口结束时间的时候从状态中选出需要信息，并清空状态。需要的信息保存到存储到地方即可



```scala
//5, 读取文件
senv.readTextFile("/Users/ivanl001/Documents/00-zhangbuer/learning_code/04-analysis/01-BigData_v2/0103-Flink-z-project/010301-project-sgg-project01/src/main/resources/apachetest.log")
.map(line => {
  val lineArray = line.split(" ")
  val simpleDateFormat = new SimpleDateFormat("dd/MM/yyyy:HH:mm:ss")
  val timestamp = simpleDateFormat.parse(lineArray(3)).getTime

  ApacheLogEvent(lineArray(0), lineArray(1), timestamp, lineArray(5), lineArray(6))
})
.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[ApacheLogEvent](Time.seconds(10)) {
  override def extractTimestamp(element: ApacheLogEvent): Long = {
    return element.eventTime
  }
})
.filter(_.method == "GET")
.keyBy(_.url)
.timeWindow(Time.minutes(1), Time.seconds(5))
.aggregate(new IMPreAggregator(), new IMWindowFunction03())
.keyBy(_.windowEnd)
.process(new IMTopNProcessFunction03(3))
.print()

senv.execute("dddd")
```

