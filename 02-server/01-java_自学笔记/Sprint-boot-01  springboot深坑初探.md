

> 后来发现这种问题的出现是在mac的idea 2017.3，window上好像没有这个问题，mac的其他版本不清楚，注意一下即可



* 目前是2018年12月15日，在 https://start.spring.io/ 的网站上现在主要有如下几个非snapshot版本：
2.1.1 
2.0.7 
1.5.18

* 刚开始随意选择了一个最新的版本2.1.1， 结果下载好了导入到idea中，正常下载maven依赖后，@RequestMapping始终没法resolve，不论怎么更换maven依赖，添加其他架包或者重新添加架包，或者重新下载都无济于事，搞得我怀疑人生

* 后来搞了半下午也没搞定，后来说重新下载再试一遍的时候无意或者有意的下载了2.0.7，结果@RequestMapping能用了！

* 再然后写好请求启动测试的时候，报错说什么类似如下的东西，往上说要添加什么Hibernate依赖啥的，但是我根本没用Hibernate，明显感觉这个问题也很奇葩，因为我就写了一个RequestMapping的请求，其他啥也没做啊：
  ```
  ***************************
  APPLICATION FAILED TO START
  ***************************
  
  Description:
  
  The Bean Validation API is on the classpath but no implementation could be found
  
  Action:
  
  Add an implementation, such as Hibernate Validator, to the classpath
  ```

* 最后，抱着侥幸的心理， 我又重新来了一遍，下载了最老的版本1.5.18，然后代码和依赖都和之前的是一摸一样的，结果你猜怎么着

* 结果就好了！！！

* 真是都是泪！！！！！！！！