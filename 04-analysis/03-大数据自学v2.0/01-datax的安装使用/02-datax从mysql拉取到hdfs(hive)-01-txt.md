> HdfsWriter提供向HDFS文件系统指定路径中写入TEXTFile文件和ORCFile文件,文件内容可与hive中表关联。
>



#### 1, 创建对应的hive表

* 如果不需要hive显示，请忽略第一步，第二步中的hdfs地址可以填写自己想存储到hdfs上的地址

* 如果是orc格式，那么创建的时候一定要指定好格式，因为默认是txt格式，如果不指定格式，但是文件存储的是orc的话， 显示肯定是乱码。

* 我们这里按照txt格式创建，这里内部表

```sql
CREATE TABLE `aries.test01`(
	  `device_imei` string, 
	  `platform` string, 
	  `app_market` string, 
	  `client_version` string, 
	  `user_id` string, 
	  `product_line` string, 
	  `accounting_day` string)
```

* 找到该表的存储位置：/user/hive/warehouse/aries.db/test01 ，接着做第二步

#### 2, 编写配置文件

* sql2hdfs.json
* 我们表比较简单，没有分区，如果有分区，一定要先添加分区，然后再拉取数据哈

```json
{
	"job": {
		"setting": {
			"speed": {
				"channel": 3
			}
		},
		"content": [{
			"reader": {
				"name": "mysqlreader",
				"parameter": {
					"username": "root",
					"password": "youzi",
					"column": [
						"device_imei",
						"platform",
						"app_market",
						"client_version",
						"user_id",
						"product_line",
						"accounting_day"
					],
					"connection": [{
						"table": [
							"ods_device_new"
						],
						"jdbcUrl": [
							"jdbc:mysql://10.19.128.178:3306/workflow"
						]
					}]
				}
			},
			"writer": {
				"name": "hdfswriter",
				"parameter": {
					"defaultFS": "hdfs://devtest.node1.com:8020",
					"fileType": "text",
					"path": "/user/hive/warehouse/aries.db/test01",
					"fileName": "ivanl001",
					"column": [{
							"name": "device_imei",
							"type": "string"
						},
						{
							"name": "platform",
							"type": "string"
						},
						{
							"name": "app_market",
							"type": "string"
						},
						{
							"name": "client_version",
							"type": "string"
						},
						{
							"name": "user_id",
							"type": "string"
						},
						{
							"name": "product_line",
							"type": "string"
						},
						{
							"name": "accounting_day",
							"type": "string"
						}
					],
					"writeMode": "append",
					"fieldDelimiter": "\t"
				}
			}
		}]
	}
}
```



#### 3, 执行拉取操作

> python ../bin/datax.py sql2hdfs.json

* 如果需要把datax的日志数据， 可以如下：

> python ../bin/datax.py sql2hdfs.json >> ivanl001.log &



