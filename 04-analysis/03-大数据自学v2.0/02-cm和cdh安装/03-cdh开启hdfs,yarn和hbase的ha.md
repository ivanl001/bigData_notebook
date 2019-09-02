参考文章：https://cloud.tencent.com/developer/article/1078380



如果要开启yarn的ha，参考如下文章：我这里不再赘述

https://blog.csdn.net/lucklilili/article/details/89669309



hbase高可用这里可参考：

https://blog.csdn.net/silentwolfyh/article/details/86658133

### hdfs的高可用

#### 1, 进入hdfs的页面

![image-20190511181423717](assets/image-20190511181423717.png)

#### 2, 点击右侧操作，下拉框中选中"启动High Availability"

![image-20190511181454392](assets/image-20190511181454392.png)



#### 3, 在新的页面选择JN和新的nn节点

![image-20190511181940545](assets/image-20190511181940545.png)

![image-20190511182038160](assets/image-20190511182038160.png)

#### 4, 设置一下jn的目录

![image-20190511182217334](assets/image-20190511182217334.png)



#### 5, 继续后就开始启用

![image-20190511182256127](assets/image-20190511182256127.png)



#### 6, 等待片刻，就ok了

* 注意：格式化namenode会报红，不过也没问题

  ![image-20190511182701340](assets/image-20190511182701340.png)



#### 7, 成功

![image-20190511182756055](assets/image-20190511182756055.png)



成功页面验证：

[http://centos01:50070](http://centos01:50070/):进入这个会显示active节点

[http://centos02:50070](http://centos03:50070/):进入这个会显示standby节点



#### 8, 按提示更改hue配置

##### 8.1, 先按照提示中Documntation的连接找到对应的hue内容

![image-20190511183739787](assets/image-20190511183739787.png)



##### 8.2,  具体如下

###### 1, [Add the HttpFS](https://www.cloudera.com/documentation/enterprise/latest/topics/admin_httpfs.html#xd_583c10bfdbd326ba-590cb1d1-149e9ca9886--7968__section_dmb_3s1_bn) role.

* 1.1, Go to the HDFS service.

* 1.2, Click the **Instances** tab.

* 1.3, Click **Add Role Instances**.

* 1.4, Click the text box below the **HttpFS** field. The Select Hosts dialog box displays.

* 1.5, Select the host on which to run the role and click **OK**.

* 1.6, Click **Continue**.

* 1.7, Check the checkbox next to the **HttpFS** role and select **Actions for Selected** > **Start**.

* ![image-20190511185532833](assets/image-20190511185532833.png)

* 上面就是说要给hdfs的namenode两个节点分别添加实例HttpFS，我nn分别是centos01，centos02，所以如下：（下面关于负载均衡的我没看太懂，先放着，应该不影响的）

  ![image-20190511184534166](assets/image-20190511184534166.png)

###### 2, After the command has completed, go to the **Hue** service.

###### 3, Click the **Configuration** tab.

###### 4, Locate the **HDFS Web Interface Role** property or search for it by typing its name in the Search box.

###### 5, Select the **HttpFS** role you just created instead of the NameNode role, and save your changes.

###### 6, Restart the Hue service.

###### 2,3,4,5,6说的就是要给hue更改配置项HDFS Web Interface Role为：HttpFS。如果后续ha切换，我感觉这里应该也需要重新更改哈

* ![image-20190511185432091](assets/image-20190511185432091.png)



#### 9, 按照提示再更新hive相关

![image-20190511185723595](assets/image-20190511185723595.png)

##### 9.1, 如果hive启动，则先关闭

![image-20190511190105825](assets/image-20190511190105825.png)

##### 9.2, 更新Hive Metastore NameNode

![image-20190511190156217](assets/image-20190511190156217.png)

##### 9.3, 启动hive服务

![image-20190511190356222](assets/image-20190511190356222.png)



#### 10, 查看hdfs实例

![image-20190511182959930](assets/image-20190511182959930.png)

#### 11, 手动进行故障迁移

##### 11.1, 进入hdfs

![image-20190511183249264](assets/image-20190511183249264.png)



##### 11.2,点击"手动故障迁移"即可

![image-20190511183321210](assets/image-20190511183321210.png)



### yarn的高可用

#### 1, 进入yarn页面，点击操作的“启动High Availability”

* 本版本是5.12.1, 对于5.7.6的版本位置可能稍微有所不同

![image-20190801174451681](assets/image-20190801174451681.png)



#### 2, 配置备用的Resource Manager

* 备用活跃的两台机器的性能最好大致一样，不能差别太大

![image-20190801174653565](assets/image-20190801174653565.png)



#### 3,  等待启动

![image-20190801174807785](assets/image-20190801174807785.png)

4, 完成！！！



### hbase的高可用

#### hbase的比较简单，直接添加角色实例，添加一个master即可

![image-20190801175420273](assets/image-20190801175420273.png)