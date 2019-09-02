### KafkaConsumer

注意哈：下面是0.10.0版本的API，这个是新的ConsumerAPI，是可以提交offset

如果使用的是0.0.8或之前的kafka的话，使用老的consumerAPI，里面是没有提交offset的选择的，一般不推荐使用老版本kafka了

```
//注意：下面这些都是新的consumerAPI，如果是0.0.8之前的版本，是不会让提交offset的，因为老版本kafka直接把offset提交给了zookeeper了
```

参考：http://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html

```java
package im.ivanl001.streaming.a00_flink_source;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;

import java.util.*;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-23 20:34
 * #description :
 **/
public class KafkaConsumer_01 {


    public static void main(String[] args) {

        //---------------------------------------手动提交偏移值-手动的处理完每个分区的数据才会提交--------------------------------------------
        //In the example below we commit offset after we finish handling the messages in each partition.
        Properties props = new Properties();
        props.put("bootstrap.servers", "centos01:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "false");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("kafka_flink"));
        try {
            while(true) {
                ConsumerRecords<String, String> records = consumer.poll(Long.MAX_VALUE);
                for (TopicPartition partition : records.partitions()) {
                    List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
                    for (ConsumerRecord<String, String> record : partitionRecords) {
                        System.out.println(record.offset() + ": " + record.value());
                    }
                    long lastOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
                    consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
                }
            }
        } finally {
            consumer.close();
        }
        //---------------------------------------手动提交偏移值-手动的处理完每个分区的数据才会提交--------------------------------------------



        //---------------------------------------手动提交偏移值-使用commitSync--------------------------------------------
        /*Properties props = new Properties();
        props.put("bootstrap.servers", "centos01:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "false");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("kafka_flink"));
        final int minBatchSize = 6;
        List<ConsumerRecord<String, String>> buffer = new ArrayList<>();
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                buffer.add(record);
            }
            if (buffer.size() >= minBatchSize) {
//                insertIntoDb(buffer);
                System.out.println(buffer);
                consumer.commitSync();
                buffer.clear();
            }
        }*/
        //---------------------------------------手动提交偏移值-使用commitSync--------------------------------------------




        //---------------------------------------自动提交偏移值--------------------------------------------
        /*Properties props = new Properties();
        props.put("bootstrap.servers", "centos01:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        consumer.subscribe(Arrays.asList("kafka_flink"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records)
                System.out.printf("offset = %d, key = %s, value = %s \n", record.offset(), record.key(), record.value());
        }*/
        //---------------------------------------自动提交偏移值--------------------------------------------
    }
}
```



### KafkaProducer

```java
package im.ivanl001.streaming.a99_flink_sink;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-24 09:29
 * #description :
 **/
public class KafkaProducer_01 {

    public static void main(String[] args) throws InterruptedException {

        Properties props = new Properties();
        props.put("bootstrap.servers", "centos01:9092");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);
        for(int i = 0; i < 100; i++){
            producer.send(new ProducerRecord<String, String>("kafka_flink", null, Integer.toString(i)));
            Thread.sleep(1000);
        }

        producer.close();
    }
}
```