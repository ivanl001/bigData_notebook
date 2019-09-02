1, 首先创表

```shell
create 'ns1:counters','f1'
```



2, 通过计数器改变某个值，比如点击

```shell
# 对该表的第一行的f1的click字段增加1
incr 'ns1:counters', 'r1', 'f1:click'

# 对该表的第一行的f1的click字段增加20
incr 'ns1:counters', 'r1', 'f1:click', 20
```





```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Increment;
import org.apache.hadoop.hbase.client.Table;
import org.junit.Test;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-13 14:25
 * #description : Hbase的计数器
 **/
public class A06_Counter {



    @Test
    public void hbase_counter_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns3:counters");
        Table table = connection.getTable(tableName);

        Increment 
          = new Increment("20181113".getBytes());
        //这里可以同时添加很多行
        increment.addColumn("daily".getBytes(), "hits".getBytes(), 10);

        table.increment(increment);
    }
}

```