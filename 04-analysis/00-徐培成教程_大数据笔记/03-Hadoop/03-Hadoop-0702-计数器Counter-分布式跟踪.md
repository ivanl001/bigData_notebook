# 计数器：Counter



* `参考  275/756  [O'REILLY]Hadoop.The.Definitive.Guide.4th.Edition.2015.3`
* 通过上下文调用，获取Counter，然后可以输入相关的信息。如果在相同的节点上重复调用，个数就会加1，如下





* 这个主要用来debug的时候查看服务器和mapper以及reducer的节点信息等

```java
context.getCounter("Reducer", IMTrackerUtils.getInfo(this, "IMWordCountReducer")).increment(1);
```





* 上面使用的工具类IMTrackerUtils如下:

```java
package im.ivanl001.bigData.IMUtils;

import java.lang.management.ManagementFactory;
import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-24 17:54
 * #description : 调用的方法的追踪信息
 * : hostname + pid + tid + oid + methodName
 * 主机 + 进程 + 线程 + 对象 + 方法名
 *
 *  不知道为什么，这个方法慢的出奇，所以暂时不能用，先放在这里，之后有用到再研究
 *
 **/
public class IMTrackerUtils {

    public static String getInfo(Object o, String msg) {
        return getHostName() + " : " + getPid() + " : " + getTid() + " : " + getOid(o) + " : " + msg;
    }

    //得到主机名 ex：BooldeMacBook-Pro.local
    public static String getHostName(){
        try {
            return InetAddress.getLocalHost().getHostName();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        return null;
    }

    //得到进程号：ex：10497
    //注：ManagementFactory.getRuntimeMXBean().getName()可以直接得到进程号@主机号，ex：10497@BooldeMacBook-Pro.local：
    public static String getPid(){
        String info = ManagementFactory.getRuntimeMXBean().getName();
        return info.substring(0,info.indexOf("@"));
    }

    //得到线程
    public static String getTid(){
        String tid = Thread.currentThread().getName();
        return tid;
    }

    //得到对象
    public static String getOid(Object o){
        //String oid = o.toString();
        String sname = o.getClass().getSimpleName();
        return sname + "@" + o.hashCode();
    }

    //根据一个对象获取这个对象的全路径
    /*public static String getOid_full(Object o) {
        String oid = o.
    }*/

}

```