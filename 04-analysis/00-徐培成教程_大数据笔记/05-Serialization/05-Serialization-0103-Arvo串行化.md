# 1，编译模式使用Arvo
* 01， 定义schema文件

```json
{
"namespace": "im.ivanl001.bigData.Avro", 
"type": "record",
"name": "emp",
"fields": [
  {"name": "age", "type": "int"}, 
  {"name": "address", "type": "string"},
  {"name":"name", "type": "string"},
  {"name":"id", "type": "int"},
  {"name":"salary", "type": "int"}
]}
```

* 02, 使用avro-tools-1.8.0.jar工具生成java文件到当前目录
  > java -jar avro-tools-1.8.0.jar compile schema emp.avsc .

* 03，把生成的文件拖入到工程中使用

* 04, 