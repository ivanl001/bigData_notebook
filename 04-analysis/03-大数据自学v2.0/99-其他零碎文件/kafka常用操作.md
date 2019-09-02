```shell
# kafka创建主题
kafka-topics.sh --create --zookeeper centos01:2181 --replication-factor 3 --partitions 1 --topic kafka2storm

# kafka生产
kafka-console-producer.sh --broker-list centos01:9092 --topic kafka2storm

# kafka消费
kafka-console-consumer.sh --bootstrap-server centos01:9092 --topic kafka2storm --from-beginning
```

