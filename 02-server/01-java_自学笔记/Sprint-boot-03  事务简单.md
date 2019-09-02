https://www.cnblogs.com/kangoroo/p/8192503.html

## 3、Service层

* 在设计service层的时候，应该合理的抽象出方法包含的内容。

* 然后将方法用@Trasactional注解注释，默认的话在抛出Exception.class异常的时候，就会触发方法中所有数据库操作回滚，当然这指的是增、删、改。

* 当然，@Transational方法是可以带参数的，具体的参数解释如下


```java
@Service
public class GeoFenceService {

    @Autowired
    private MoonlightMapper moonlightMapper;

    @Transactional
    public int addGeoFence(GeoFence geoFence) {
        String formatTime = TimeFunction.transTimeToFormatPerfect(System.currentTimeMillis());
        geoFence.setCreateTime(formatTime);
        geoFence.setUpdateTime(formatTime);
        return moonlightMapper.insertOne(geoFence);
    }

    @Transactional
    public int batchGeoFence(List<GeoFence> geoFenceList) {
        String formatTime = TimeFunction.transTimeToFormatPerfect(System.currentTimeMillis());
        for (GeoFence geoFence : geoFenceList) {
            geoFence.setCreateTime(formatTime);
            geoFence.setUpdateTime(formatTime);
        }
        return moonlightMapper.insertBatch(geoFenceList);
    }
}
```

## 4、开启事务

* 最后你要在Application类中开启事务管理，开启事务管理很简单，只需要@EnableTransactionManagement注解就行

```java
@EnableTransactionManagement
@SpringBootApplication
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```

### 5,测试一下

* 可以做一个简单的测试，主动抛出异常，测试一下是否真的能保证事务性。

* 在执行完插入之后，手动抛出一个空指针异常，可以发现数据真的回滚了。

```java
@Service
public class GeoFenceService {

    @Autowired
    private MoonlightMapper moonlightMapper;

    @Transactional
    public int addGeoFence(GeoFence geoFence) {
        String formatTime = TimeFunction.transTimeToFormatPerfect(System.currentTimeMillis());
        geoFence.setCreateTime(formatTime);
        geoFence.setUpdateTime(formatTime);
        int count = moonlightMapper.insertOne(geoFence);
        String a = null;
        a.indexOf('c');
        return count;
    }
}
```