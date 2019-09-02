> 08, 协处理器处理被叫等

> 09, web可视化
 

## 08, 协处理器处理被叫等

*首先有两个地方需要协处理器处理：第一存储主叫的时候我们需要用协处理器相应的存储一条被叫的信息，这条信息的值是主叫的rowkey；第二，在读取的时候，需要找到主叫的rowkey后再次找到rowkey的值从来找到被叫信息所需要的字段值*

* 01，存储和读取都在如下代码中：
```java
package im.ivanl001.calllogs;

import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.coprocessor.BaseRegionObserver;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.regionserver.InternalScanner;
import org.apache.hadoop.hbase.regionserver.wal.WALEdit;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-17 15:11
 * #description :
 **/
public class IMCorprocessor extends BaseRegionObserver {

    private String tableNameStr = "ns3:calllogs";
    private String refKey = "refRowkey";

    //通过这个方法来在查询的时候获取被叫相关的信息
    @Override
    public boolean postScannerNext(ObserverContext<RegionCoprocessorEnvironment> e, InternalScanner s, List<Result> results, int limit, boolean hasMore) throws IOException {

        boolean scannerNext = super.postScannerNext(e, s, results, limit, hasMore);

        //新集合
        List<Result> newList = new ArrayList<Result>();

        //获得表名
        String tableName = e.getEnvironment().getRegion().getRegionInfo().getTable().getNameAsString();

        //判断表名是否是ns3:calllogs
        if (tableName.equals(tableNameStr)) {

            Table table = e.getEnvironment().getTable(TableName.valueOf(tableNameStr));

            for(Result result : results){

                //rowkey
                String rowkey = Bytes.toString(result.getRow());
                String flag = rowkey.split(",")[3] ;

                //主叫
                if(flag.equals("0")){
                    //如果是主叫，就把原先的结果返回回去即可
                    newList.add(result) ;
                }
                //被叫
                else{
                    //取出主叫号码
                    byte[] refrowkey = result.getValue("f2".getBytes(), refKey.getBytes()) ;
                    Get newGet = new Get(refrowkey);
                    newList.add(table.get(newGet));
                }
            }
            //注意：这里一定要清空
            results.clear();
            results.addAll(newList);
        }
        return scannerNext;
    }

    //通过这个方法来处理存入一条主叫信息的时候同时保存对应的被叫信息
    @Override
    public void postPut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability) throws IOException {

        //00, 调用父类方法
        super.postPut(e, put, edit, durability);

        //01，判断是否是我们需要处理的表
        TableName tableName = TableName.valueOf(tableNameStr);
        String tableNameStr = e.getEnvironment().getRegionInfo().getTable().getNameAsString();
        if (!tableNameStr.equals(tableName.getNameAsString())) {
            return;
        }

        //02, rowkey, 并放行被叫
        String rowkey = Bytes.toString(put.getRow()) ;
        //如果被叫就放行
        String[] arr = rowkey.split(",");
        if (arr[3].equals("1")) {
            return;
        }

        //03，处理hashcode,caller,time,flag,callee,duration
        String caller = arr[1] ;        //主叫
        String time = arr[2] ;      //通话时间
        String callee = arr[4] ;        //被叫
        String durationStr = arr[5] ;  //通话时长
        //这里传入两个时间，第一个时间是用来截取后计算区域值，第二个是放在rowkey中的
        String regionNo01 = IMRowKeyUtils.getHashcode(callee, time);

        //04，组建新的rowkey
        String rowKey01 = regionNo01 + "," + callee + "," + time + "," + "1" + "," + caller + "," + durationStr;

        //05，开始存数据
        Put negativePut = new Put(rowKey01.getBytes());
        negativePut.addColumn(Bytes.toBytes("f2"), Bytes.toBytes(refKey), Bytes.toBytes(rowkey));
        Table table = e.getEnvironment().getTable(tableName);
        table.put(negativePut);
    }
}
```

* 02, 上面需要用到的一个工具类如下：
```java
package im.ivanl001.calllogs;

import java.text.DecimalFormat;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-17 13:42
 * #description :
 **/
public class IMRowKeyUtils {

    private static DecimalFormat decimalFormat = new DecimalFormat("00");

    public static String setupRowkey(String calller, String startTimeStr, String theTimeStr) {
        return getHashcode(calller, startTimeStr) + "," + calller + "," + theTimeStr;
    }

    public static String getHashcode(String caller ,String callTime){
        int len = caller.length();
        //取出后四位电话号码
        String last4Code = caller.substring(len - 4);
        //取出时间单位,年份和月份.
        String mon = callTime.substring(0,6);
        //这里是通过亦或的方式，这里不是很明白这里的和之前的hashcode的区别，应该都可以,我们这里假设分有100个区
        int hashcode = (Integer.parseInt(mon) ^ Integer.parseInt(last4Code)) % 100 ;
        return decimalFormat.format(hashcode);
    }
}
```

* 03, 以上代码打成jar包发送到hbase的lib目录下，然后在hbase-site.xml中设置协处理器即可，这样就算是hbase shell中也能按照协处理器显示结果的哈

## 09, web可视化
*这里代码比较多，不放代码了，需要的话我后续传到github上*