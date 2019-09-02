> 方案：cAdvisor+InfluxDB+Grafana



* cAdvisor:Google开源的工具，用于监控Docker主机和容器系统资源，通过图形页面实时显示数据，但不存储;它通过 宿主机/proc、/sys、/var/lib/docker等目录下文件获取宿主机和容器运行信息。
* InfluxDB:是一个分布式的时间序列数据库，用来存储cAdvisor收集的系统资源数据。
* Grafana:可视化展示平台，可做仪表盘，并图表页面操作很方面，数据源支持zabbix、Graphite、InfluxDB、OpenTSDB、Elasticsearch等

它们之间关系:cAdvisor容器数据采集->InfluxDB容器数据存储->Grafana可视化展示