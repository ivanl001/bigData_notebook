> 参考连接：
>
> datax github官方地址：https://github.com/alibaba/DataX
>
> 

#### 1, 安装使用

##### 1.1, 下载地址

> http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz



##### 1.2, 使用方式(DataX工具包,非源码编译方式)

* 下载后解压至本地某个目录，进入bin目录，即可运行同步作业：

  ```shell
  $ cd  {YOUR_DATAX_HOME}/bin
  $ python datax.py {YOUR_JOB.json}
  ```

  

* 自检脚本：    

  ```shell
  $ python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json
  ```

  

##### 1.3, 配置示例：从stream读取数据并打印到控制台

###### 1.3.1, 第一步, 查找配置文件模板（json格式）

> 可以通过命令查看配置模板： python datax.py -r {YOUR_READER} -w {YOUR_WRITER}
>
> 如：python datax.py -r streamreader -w streamwriter， 会打印如下信息：
>
> 当然也可以通过官网的github地址下载配置的模板， 里面还有具体的字段的详细解释

```shell
DataX (DATAX-OPENSOURCE-3.0), From Alibaba !
Copyright (C) 2010-2017, Alibaba Group. All Rights Reserved.


Please refer to the streamreader document:
     https://github.com/alibaba/DataX/blob/master/streamreader/doc/streamreader.md 

Please refer to the streamwriter document:
     https://github.com/alibaba/DataX/blob/master/streamwriter/doc/streamwriter.md 
 
Please save the following configuration as a json file and  use
     python {DATAX_HOME}/bin/datax.py {JSON_FILE_NAME}.json 
to run the job.

{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "streamreader", 
                    "parameter": {
                        "column": [], 
                        "sliceRecordCount": ""
                    }
                }, 
                "writer": {
                    "name": "streamwriter", 
                    "parameter": {
                        "encoding": "", 
                        "print": true
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": ""
            }
        }
    }
}
```



###### 1.3.2, 第二步, 根据模板自定义配置文件

```json
#stream2stream.json
{
  "job": {
    "content": [
      {
        "reader": {
          "name": "streamreader",
          "parameter": {
            "sliceRecordCount": 10,
            "column": [
              {
                "type": "long",
                "value": "10"
              },
              {
                "type": "string",
                "value": "hello，你好，世界-DataX"
              }
            ]
          }
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "encoding": "UTF-8",
            "print": true
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 5
       }
    }
  }
}
```

###### 1.3.3, 第三步, 启动datax，根据配置json文件执行即可

* 如下执行后即可通过streamwriter把streamreader从内存中读取的内存打印在控制台上

```shell
$ python ../bin/datax.py stream2stream.json
```



