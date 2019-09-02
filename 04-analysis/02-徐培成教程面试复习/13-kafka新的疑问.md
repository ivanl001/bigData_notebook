如下两种是不同的模式还是新老API的不同？

```java
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records) {
             buffer.add(record);
         }
         if (buffer.size() >= minBatchSize) {
             insertIntoDb(buffer);    //消除处理，存到db
             consumer.commitSync();   //同步发送ack
             buffer.clear();
         }
     }
```



```java
Properties props = new Properties();
        props.put("zookeeper.connect", "slave02:2181");
        props.put("group.id", "testgroup01");
        props.put("zookeeper.session.timeout.ms", "50000");//这个时间设置500好像不行，太短了？？？
        props.put("zookeeper.sync.time.ms", "250");
        props.put("auto.commit.interval.ms", "1000");

        ConsumerConfig consumerConfig = new ConsumerConfig(props);
        ConsumerConnector consumer = Consumer.createJavaConsumerConnector(consumerConfig);

        Map<String, Integer> topicCount = new HashMap<String, Integer>();
        topicCount.put("test", new Integer(1));

        Map<String, List<KafkaStream<byte[], byte[]>>> consumerStreams = consumer.createMessageStreams(topicCount);

        List<KafkaStream<byte[], byte[]>> streams = consumerStreams.get("test");

        for (final KafkaStream stream : streams) {
            ConsumerIterator<byte[], byte[]> consumerIte = stream.iterator();
            while (consumerIte.hasNext())
                System.out.println("Message from Single Topic :: " + new String(consumerIte.next().message()));
        }
        if (consumer != null){
            consumer.shutdown();
        }
```

