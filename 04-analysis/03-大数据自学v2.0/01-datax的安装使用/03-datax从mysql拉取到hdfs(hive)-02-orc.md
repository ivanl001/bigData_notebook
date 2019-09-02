> HdfsWriter提供向HDFS文件系统指定路径中写入TEXTFile文件和ORCFile文件,文件内容可与hive中表关联。



#### 1, 创建对应的hive表

- 如果不需要hive显示，请忽略第一步，第二步中的hdfs地址可以填写自己想存储到hdfs上的地址
- 我们这里按照orc格式创建，这里内部表

```sql
CREATE TABLE `aries.srv_ptn_analysis_order_charge`(
	  `rowkey` string, 
	  `platform` int, 
	  `app_market` int, 
	  `app_market_name` string, 
	  `pay_type` string, 
	  `charge_type` int, 
	  `amount` int, 
	  `order_id` string, 
	  `order_status` int, 
	  `create_time` string, 
	  `user_id` string, 
	  `client_ip` string, 
	  `client_version` string, 
	  `activity_id` string, 
	  `free_days` int, 
	  `free_start_date` string, 
	  `expire_date` string, 
	  `expire_day` string, 
	  `accounting_year` string, 
	  `accounting_month` string, 
	  `accounting_hour` string, 
	  `purchase_start_date` string, 
	  `purchase_end_date` string, 
	  `promoter_id` string, 
	  `promoter_channel_name` string, 
	  `total_amount` int)
	PARTITIONED BY ( 
	  `product_line` int, 
	  `accounting_day` string)
	ROW FORMAT SERDE 
	  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
	WITH SERDEPROPERTIES ( 
	  'serialization.null.format'='0') 
	STORED AS INPUTFORMAT 
	  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
	OUTPUTFORMAT 
	  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
```

- 找到该表的存储位置：/user/hive/warehouse/aries.db/srv_ptn_analysis_order_charge ，接着做第二步

#### 2, 添加分区

> alter table aries.srv_ptn_analysis_order_charge add partition (product_line = 6, accounting_day = '20190428')

#### 3, 编写配置文件

- sql2hdfs_order_charge.json
- 我们表比较简单，没有分区，如果有分区，一定要先添加分区，然后再拉取数据哈

```json
{
	"setting": {},
	"job": {
		"setting": {
			"speed": {
				"channel": 1
			}
		},
		"content": [{
			"reader": {
				"name": "mysqlreader",
				"parameter": {

					"connection": [{
						"querySql": ["select   CONCAT(FLOOR(RAND()*10000),a.userId)  as rowkey,platform,-1 as app_market,appMarketName,payType,chargeType,amount,orderId,orderStatus,createTime,userId,-1 as client_ip,-1 as client_version,-1 as activity_id,-1 as free_days,-1 as free_start_date,expireDate,date_format(expireDate,'%Y%m%d') as expire_day,YEAR(CreateTime) as accounting_year,MONTH(CreateTime) as accounting_month,HOUR(CreateTime) AS accounting_hour,purchaseStartDate,purchaseEndDate,-1 as promoter_id,-1 as promoter_channel_name,totalAmount from  etl_chargeorders a where  a.CreateTime >= '2019-04-28 00:00:00' AND a.CreateTime <= '2019-04-28 23:59:59' AND a.ProductLine=6"],
						"jdbcUrl": ["jdbc:mysql://10.10.146.253:3306/bi_data_etl"]
					}],
					"password": "edFbbcF3Zdpvipfo",
					"username": "etl_read"
				}
			},
			"writer": {
				"name": "hdfswriter",
				"parameter": {
					"defaultFS": "hdfs://devtest.node1.com:8020",
					"fileType": "orc",
					"path": "/user/hive/warehouse/aries.db/srv_ptn_analysis_order_charge/product_line=6/accounting_day=20190428",
					"fileName": "datax_etl_chargeorders_$yesterday",
					"column": [{
						"name": "rowkey",
						"type": "string"

					}, {
						"name": "platform",
						"type": "int"

					}, {
						"name": "app_market",
						"type": "int"
					}, {
						"name": "app_market_name",
						"type": "string"
					}, {
						"name": "pay_type",
						"type": "string"
					}, {
						"name": "charge_type",
						"type": "int"
					}, {
						"name": "amount",
						"type": "int"
					}, {
						"name": "order_id",
						"type": "string"
					}, {
						"name": "order_status",
						"type": "int"
					}, {
						"name": "create_time",
						"type": "string"
					}, {
						"name": "user_id",
						"type": "string"
					}, {
						"name": "client_ip",
						"type": "string"
					}, {
						"name": "client_version",
						"type": "string"
					}, {
						"name": "activity_id",
						"type": "string"
					}, {
						"name": "free_days",
						"type": "int"
					}, {
						"name": "free_start_date",
						"type": "string"
					}, {
						"name": "expire_date",
						"type": "string"
					}, {
						"name": "expire_day",
						"type": "string"
					}, {
						"name": "accounting_year",
						"type": "string"
					}, {
						"name": "accounting_month",
						"type": "string"
					}, {
						"name": "accounting_hour",
						"type": "string"
					}, {
						"name": "purchase_start_date",
						"type": "string"
					}, {
						"name": "purchase_end_date",
						"type": "string"
					}, {
						"name": "promoter_id",
						"type": "string"
					}, {
						"name": "promoter_channel_name",
						"type": "string"
					}, {
						"name": "total_amount",
						"type": "int"
					}],
					"writeMode": "append",
					"fieldDelimiter": "\u0001"
				}
			}
		}]
	}
}
```



#### 3, 执行拉取操作

> python ../bin/datax.py sql2hdfs_order_charge.json

- 如果需要把datax的日志数据， 可以如下：

> python ../bin/datax.py sql2hdfs_order_charge.json >> ivanl001.log &



