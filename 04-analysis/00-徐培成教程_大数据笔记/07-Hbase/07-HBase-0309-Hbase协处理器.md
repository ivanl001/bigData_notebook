*以下步骤貌似在hbase 2.1.0中好像没有成功，换成低版本的1.2的是可以的，2.1版本报错缺少包，不知道是不是需要另外引入*

> 更多协处理器的内容也是参考书籍《[中文]HBase权威指南.pdf》这本书

* 01， 首先代码实现
```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.hbase.CoprocessorEnvironment;
import org.apache.hadoop.hbase.client.Durability;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.coprocessor.BaseRegionObserver;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.regionserver.RegionScanner;
import org.apache.hadoop.hbase.regionserver.wal.WALEdit;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-13 14:52
 * #description : 协处理器
 **/
public class A07_Coprocessor extends BaseRegionObserver {


    //在下面进行每个生命过程的触发，大多数时间都可以重新触发过程的哈
    @Override
    public void start(CoprocessorEnvironment e) throws IOException {
        super.start(e);
        FileWriter fw = new FileWriter("/data/hbase/userLogs/coprocessor_test.txt",true);

        fw.write("ivanl001 is the king of world!----A07_Coprocessor----start");
        fw.flush();
        fw.close();
        System.out.println("kkkkkk");
    }

    @Override
    public void stop(CoprocessorEnvironment e) throws IOException {
        super.stop(e);
    }

    @Override
    public void preOpen(ObserverContext<RegionCoprocessorEnvironment> e) throws IOException {
        super.preOpen(e);
    }

    @Override
    public void postOpen(ObserverContext<RegionCoprocessorEnvironment> e) {
        super.postOpen(e);
    }

    @Override
    public void prePut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability) throws IOException {
        super.prePut(e, put, edit, durability);
    }

    @Override
    public void postPut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability) throws IOException {
        super.postPut(e, put, edit, durability);
    }

    @Override
    public RegionScanner preScannerOpen(ObserverContext<RegionCoprocessorEnvironment> e, Scan scan, RegionScanner s) throws IOException {
        return super.preScannerOpen(e, scan, s);
    }
}
```

* 02, 打包发送到服务器上，注意分发

* 03, 配置hbase-site.xml，添加如下的属性 
```xml
<property>
  <name>hbase.coprocessor.region.classes</name>
  <value>im.ivanl001.bigData.Hbase.A07_Coprocessor</value>
</property>
```

* 04，重启hbase应该就可以执行协处理器中的功能了，我们这里是打印到文件中，可以在几台区域服务器上看到这些文件了
  > stop-hbase.sh

  > start-hbase.sh
  