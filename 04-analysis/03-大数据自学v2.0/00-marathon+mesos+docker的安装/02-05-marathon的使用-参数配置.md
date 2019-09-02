* 看文档吧，marathon的使用，主要是配置文件

https://blog.csdn.net/weixin_33875564/article/details/87213235

* 官网文档

https://mesosphere.github.io/marathon/docs/deployments.html





- 保证每台机器上只运行一个
- UNIQUE Operator: 它告诉marathon，强制这个app的所有tasks的属性唯一. 例如下面的constraints保证了每个节点上只运行app的一个task：

```json
"constraints": [["hostname", "UNIQUE"]]
```





* 在指定机器上运行
* This example uses the specified `hostname`, which means an app must run on a specific node.

```shell
$ curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{
    "id": "sleep-cluster",
    "cmd": "sleep 60",
    "instances": 3,
    "constraints": [["hostname", "CLUSTER", "a.specific.node.com"]]
  }'
```

