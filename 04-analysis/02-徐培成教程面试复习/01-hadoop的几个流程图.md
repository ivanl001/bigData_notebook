hdfs数据写入过程：

- 01, 首先创建FileSystem.get的时候会根据配置文件的不同创建不同的文件系统，有可能是本地文件系统或者分布式文件系统
- 02, 然后开始创建输出流对象。
- 03, 之后写入write操作，在写入的时候判断根据数据量，如果大于512字节，那么，进行校验，形成一个4个字节的数据，和512字节的数据一起组成一个chunk，然后用来把所有的chunk放在一起，组成一个packet包，一个包中一共64k数据，包括516*126=65016，也就是126个chunk，然后还有33个字节的包头数据，最后组成一个包
- 04, 如果不大于512字节，直接放在缓存中，然后把后续的写入一起累加起来，最后还是不大于512字节，就等着close的时候直接一并把那些不足的给搞到一个packet中写出。
- 05, 在关闭的时候才会把从主线程中把包给放到序列中flush出去。
- 06, 第一个数据节点写完之后，后续会把第一个节点的数据写入到第二个节点，第二个节点写入到第三个节点，以此类推，直到写完所有副本。写完之后会进行消息通知，从队列中删除。如果出错，那么会重新把这个包放在队列最前，重新开始。
- 大致过程如上，有写地方描述是不对的，大体上是对的，具体参考源码

- `参考  97/756  [O'REILLY]Hadoop.The.Definitive.Guide.4th.Edition.2015.3`

  - ![IMAGE](assets/9A4CBAC625DF4291925A4FB4202C5349.jpg)

  - `参考  100/756  [O'REILLY]Hadoop.The.Definitive.Guide.4th.Edition.2015.3`
    - ![IMAGE](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/F2B1ACAD321BDEB648E1F47E4B1E4048.jpg)

- 作业提交过程流程图
  - ![IMAGE](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/8FC78F1B87E51C8D64291AA85ECB8426.jpg)

切片是针对mapper而言的， 有多少切片，就会有多少个mappe==》 默认情况下是这样，如果设置了切片最大大小，那就不一定了哈

- ![IMAGE](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/22E425DE62073EF1B26D1331CF02403A.jpg)





SecondaryNamenode的作用：帮助namenode合并编辑日志，减少Namenode的启动时间 #TODO

- ![IMAGE](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/7FDE9B6327EB17CAC473A03BD7F33820.jpg)
- 1. The secondary asks the primary to roll its in-progress edits file, so new edits go to a new file. The primary also updates the seen_txid file in all its storage directories.

- 2. The secondary retrieves the latest fsimage and edits files from the primary (using HTTP GET).
- 3. The secondary loads fsimage into memory, applies each transaction from edits, then creates a new merged fsimage file.
- 4. The secondary sends the new fsimage back to the primary (using HTTP PUT), and the primary saves it as a temporary .ckpt file.
- 5. The primary renames the temporary fsimage file to make it available.
- 滚动时间是一个小时，在hdfs-default.xml中有如下配置信息





