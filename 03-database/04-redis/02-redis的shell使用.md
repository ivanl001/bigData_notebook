http://redisdoc.com/



## key相关

```shell
/usr/local/redis-5.0.5/src/redis-cli

shutdown

# 获取数据库，一共有16个数据库，分别是0-15
config get databases

# 使用3号库
select 3

# 查询所有key
keys *

# 设置获取
set foo bar
get foo

# 某个key是否存在，1代表true，0代表false
exists foo

# key的数据类型
type foo

# 删除某个key
del foo

# 查看数据库数据数量
dbsize

# 设置过期时间
expire foo 10

# 查看还有多少秒过期，-1代表永不过期，-2代表已经过期
set foo bar
ttl foo

# 清空数据库
flushdb

# 通杀全部库
flushall
```



## string

```shell
# 设置
set foo bar
# 重复设置会直接覆盖
set foo ivanl00
# 获取
get foo
# 追加
append foo ivanl001
# value长度
strlen foo
# key不存在才会设置
setnx foo ivanl002

# 获取value的部分内容，substring
getrange foo 0 1
# 设置value的部分内容
setrange foo 10 'ivanl001 is the king of world'
# 设置valu同时设置过期时间
setex foo10 10 bar10
# 获取值之后，把新的值赋值给key
getset foo10 foo10000000


# 同时设置多个
mset foo bar foo01 bar01
# 同时获取多个
mget foo foo01
# 同时不存在设置多个
# 注意：需要所有的key都不存在才会设置
msetnx foo bar foo01 bar01 foo02 bar02


set sum 0
incr sum
decr sum
# 按照步长增加减少
incrby sum 2
decrby sum 2
```



## List

```shell
# 设置元素，增加元素
# 增加在左边
lpush ivanl ivanl001 ivanl002
# 在右边增加
rpush ivanl ivanl010

# 推出元素，删除元素
# 从左边删除
lpop ivanl
# 从右边删除
rpop ivanl

# 从右边推出一个元素，补充到左边
rpoplpush ivanl ivanl


# 按照索引下标范围获得元素
lrange ivanl 0 1
# 按照指定下标获得元素
lindex ivanl 0
# 获取列表长度
llen ivanl
# 在指定value前插入, before或者after
linsert ivanl before ivanl02 ivanl011

```



## Set

* 可以自动排重
* Set就是value为null的hash表
* ZSet 和Set的区别是ZSet会排序

```shell
# 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
sadd <key>  <value1>  <value2> .....   
sadd sfoo foo01 foo02 foo01

#  取出该集合的所有值。
smembers <key>
smembers sfoo

#  判断集合<key>是否为含有该<value>值，有返回1，没有返回0
sismember <key>  <value>
 
# 返回该集合的元素个数。
scard   <key>

# 删除集合中的某个元素。
srem <key> <value1> <value2> ....

# 随机从该集合中吐出一个值。
spop <key>  

# 随机从该集合中取出n个值，不会从集合中删除
srandmember <key> <n>

# 返回两个集合的交集元素。
sinter <key1> <key2>  

# 返回两个集合的并集元素。
sunion <key1> <key2>  

# 返回两个集合的差集元素。
sdiff <key1> <key2>  
```



## Hash

* Redis hash 是一个键值对集合

* 类似Java里面的Map<String,Object>



```shell
# 给<key>集合中的  <field>键赋值<value>
hset <key>  <field>  <value>
hset 10120945 name ivanl001

# 从<key1>集合<field> 取出 value 
hget <key1>  <field>  
hget 10120945 name

# 批量设置hash的值
hmset <key1>  <field1> <value1> <field2> <value2>...  
hmset 10120945 age 10 hobby tenisball

# 查看哈希表 key 中，给定域 field 是否存在。 
hexists key  <field>

# 列出该hash集合的所有field
hkeys <key>   
hkeys 10120945

# 列出该hash集合的所有value
hvals <key>   
hvals 10120945

# 为哈希表 key 中的域 field 的值加上增量 increment 
hincrby <key> <field>  <increment> 

# 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
hsetnx <key>  <field> <value>

```



## zset

