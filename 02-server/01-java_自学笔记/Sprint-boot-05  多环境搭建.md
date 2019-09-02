非常easy

* 01, 生成相应的文件,格式application-{$profile}.properties
  > application-test.properties
  > application-dev.properties
  > application-prod.properties

* 02, 需要使用哪个环境，在application.properties配置如下属性,
  *指定当前环境,spring.profiles.active={$properties}*

  > spring.profiles.active=test

