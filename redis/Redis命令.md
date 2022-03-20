## Redis命令

### key管理

**keys ***

返回所有key，可以模糊匹配，比如keys abc* 表示返回abc开通的key

**exists <key>**

判断是否存在指定的key，存在返回1，不存在返回0

**expire <key> <seconds>**

设置key的过期时间

**pexpire key milliseconds**

设置key的过期时间，单位为毫秒

**ttl <key>**

查看某个key剩余时间，当key不存在时，返回-2; 存在但是没有设置expire，返回-1; 否则返回剩余秒数。

**pttl <key>**

以毫秒为单位返回剩余时间

**PERSIST key**

取消超时时间

**DEL key [key ...]**

删除一个或多个key

**RENAME key newkey**

修改key的名称

**MOVE key db**

将当前数据库的key移动到指定的数据库中

**TYPE key**

获取key所存储值的类型

### Redis数据类型

### String类型

#### String命令

##### 赋值语法

多次设置，值会覆盖，且无视类型

```
SET key value [expiration EX seconds|PX milliseconds] [NX|XX]
```

如果key不存在，设置值，返回1。如果key存在，则不设置并返回0。解决分布式锁的方案之一

```
SETNX key value
```

设置一个key的值和过期时间

```
SETEX key seconds value
```

覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。

```
SETRANGE key offset value
```

```
127.0.0.1:6379> get name
"xitestu" 
127.0.0.1:6379> setrange name 2 nyul 
(integer) 7
127.0.0.1:6379> get name
"xinyulu"			
```

一次性写入多个值

```
MSET key value [key value ...]
```

##### 取值语法

获取指定key的值，如果key不存在，返回nil，如果key存储的不是字符串类型，返回一个错误
```
GET key
```

获取存储在指定key中字符串的子字符串。字符串的截取范围由start和end两个偏移量决定
```
GETRANGE key start end					
```

```
127.0.0.1:6379> get name
"xinyulu"
127.0.0.1:6379> getrange name 2 4
"nyu"					
```

对 key 所储存的字符串值，获取指定偏移量上的位(bit)，参考文档: https://blog.csdn.net/sinat_38740436/article/details/88599751

```
GETBIT key offset
```

设置指定key的值，并且返回key的旧值。当前key不存在时，设置指定的值，并返回nil

```
GETSET key value
```

返回key所存储字符串长度

```
STRLEN key
```

批量读

```
MGET key [key ...]			
```

##### 删除语法

删除指定key

```
DEL key [key ...]		
```

##### 自增自减

将key中存储的数字值增1。如果key不存在，那么key的值会先被初始化为0，再执行incr

```
INCR key					
```

将key中存储的数字加上指定的增量值

```
INCRBY key increment
```

DECRBY key decrement

```
DECR key
```

##### 字符串拼接

为指定的key追加至末尾，如果不存在，为其赋值

```
APPEND key value
```

#### 应用场景

* String通常用于保存单个字符串或json字符串数据
* 因为String是二进制安全的，所以完全可以把一个图片文件的内容作为字符串来存储
* 计数器(常规key-value缓存应用。微博数，粉丝数

### Hash类型

#### Hash命令

##### 赋值语法


为指定的KEY，设定field/value

```
HSET key field value
```


同时将多个field-value对设置到哈希表key中

```
HMSET key field value [field value ...]
```


只有在指定的field不存在时，才设置该field的值

```
HSETNX key field value				
```

##### 取值语法


获取key中，指定field的值

```
HGET key field
```


获取key中，指定的多个field的值

```
HMGET key field [field ...]
```


获取key中所有field的值

```
HGETALL key
```


获取key中所有的field

```
HKEYS key
```


获取key中field的数量

```
HLEN key	
```

##### 删除语法

删除一个或多个key中的field

```
HDEL key field [field ...]				
```

##### 其它语法

为哈希表key中指定的field的整数value增加increment

```
HINCRBY key field increment
```


为哈希表key中指定的field的浮点value增加increment

```
HINCRBYFLOAT key field increment
```


查看key中，指定的field是否存在

```
HEXISTS key field			
```

#### 应用场景
常用于存储一个对象

### List类型

#### List命令

##### 赋值语法


将一个或多个值插入到列表头部(从左侧添加)

```
LPUSH key value [value ...]
```


将一个或多个值插入到列表尾部(从右侧添加)

```
RPUSH key value [value ...]
```


将一个值插入到已存在列表的头部。如果列表不存在，操作无效

```
LPUSHX key value
```


将一个值插入到已存在列表的尾部，如果列表不存在，操作无效

```
RPUSHX key value	
```

##### 取值语法

获取列表长度

```
LLEN key
```


通过索引获取列表中的元素

```
LINDEX key index
```


获取列表指定范围内的元素，0表示第一个元素，-1表示最后一个元素

```
LRANGE key start stop
```

##### 删除语法

移除并获取列表的第一个元素

```
LPOP key
```


移除并获取列表最后一个元素

```
RPOP key
```


移除并获取列表第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

```
BLPOP key [key ...] timeout
```


移除并获取列表最后一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

```
BRPOP key [key ...] timeout
```


对一个列表进行修剪，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除

```
LTRIM key start stop	
```

##### 修改语法

通过索引设置列表元素的值

```
LSET key index value
```


在列表的元素pivot前或者后插入元素

```
LINSERT key BEFORE|AFTER pivot value
```

##### 高级命令


将source的最后边元素弹出，插入到destination的最左边

```
RPOPLPUSH source destination
```


将source最右边的元素弹出，插入到destination的最左边，并返回这个元素。如果source没有元素可弹，将阻塞列表直到超时或有元素为止

```
BRPOPLPUSH source destination timeout	
```

### Set类型

SADD key member.…：向set中添加一个或多个元素

SREMkey member …∶移除set中的指定元素

SCARD key：返回set中元素的个数

SMEMBER key member：判断一个元素是否存在于set中

SMEMBERS：获取set中的所有元素

SINTER key1 key2.…：求key1与key2的交集

SDIFFkey1 key2.…：求key1与key2的差集

SUNION key1 key2…：求key1和key2的并集

### SortedSet

ZADD key score member：添加一个或多个元素到sorted set，如果已经存在则更新其score值

ZREM key member∶删除sorted set中的一个指定元素

ZSCORE key member：获取sorted set中的指定元素的score值

ZRANK key member∶获取sorted set 中的指定元素的排名

ZCARD key∶获取sorted set中的元素个数

ZCOUNT key min max：统计score值在给定范围内的所有元素的个数

ZINCRBY keyincrement member;让sorted set中的指定元素自增，步长为指定的increment值

ZRANGE key min max∶按照score排序后，获取指定排名范围内的元素

ZRANGEBYSCORE key min max：按照SCore排序后，获取指定score范围内的元素

ZDIFF、ZINTER、ZUNION：求差集、交集、并集

### key的命名建议

* key不要太长，尽量不要超过1024字节，这不仅消耗内存，而且会降低查找的效率，key也不要太短，太短的话，key的可读性会降低
* 在一个项目中，key最好使用统一的命名模式，例如user:123:password
* key名称区分大小写