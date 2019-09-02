02，使用mvn命令，下载工件的所有依赖软件包，方式如下:
*在放有pom.xml的文件夹下建立一个lib文件夹，然后在pom.xml文件位置执行如下命令，会自动把所有依赖jar包下载到刚才的lib文件夹中去，注意修改其中的groupid什么的跟pom.xml中的一致哈*

mvn -DoutputDirectory=./lib -DgroupId=im.ivanl001 -DartifactId=14_CallLogKafkaConsumer -Dversion=1.0-SNAPSHOT dependency:copy-dependencies

