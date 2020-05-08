## Redis

- 启动服务器：
  - ./redis-server
  - 后台启动服务器：./redis-server &
  - 指定端口启动服务器：./redis-server --port [port] [&]
  - 按配置文件启动：./redis-server [path]
- 启动客户端：./redis-cli
  - 指定端口启动客户端： -p [port]
  - 指定ip启动客户端：-h [ip]
  - 输入密码启动：-a [password]
- 关闭客户端：quit
- 关闭客户端：
  - 在客户端中关闭服务：SHUTDOWN NOSAVE|SAVE
    - 不带参数默认SAVE
  - 在命令行中关闭服务：./redis-cli shutdown
  - 在命令行中关闭特定端口服务：./redis-cli -p [port] shutdown
- redis配置文件：redis.conf
  - 密码参数：requirepass [password]

注：通过ctrl+c终止服务redis时不会进行持久化存储的

注：redis所有操作都在客户端中进行

------

### redis命令

#### redis系统操作

- 关闭redis：shutdown nosave|save
  - 不带参数默认save
- 人为进行保存：save
- 查看系统信息：info



#### 数据库操作

- 选择数据库：select [dbid]
  - 数据库数量可在配置文件上修改，参数：databases [num]
  - 默认数据库为0
  - [dbid]的范围在0~databases-1
- 清除当前数据库所有数据：flushdb
- 清除所有数据库数据：flushall
- 查看当前数据库键值对数量：dbsize



#### 数据操作

- 获取所有key： keys *
- 获取特定key：keys [pattern]
- 随机获取key：randomkey
- 删除key：del [key] [key...]
- 判断key是否存在：exists [key] [key...]
- 重命名key：
  - rename [key] [newkey]
    - 存在覆盖效果，若newkey存在则将其覆盖
  - renamenx [key] [newkey]
    - newkey存在则不生效
- 返回key的剩余生存时间：ttl [key]
  - 返回-1即无过期时间
  - 返回-2即为过期或不存在
  - 单位为秒
- 设置key生存时间：expire [key] [seconds]
- 返回值的类型：type [key]

注：带m开头的指令带有批量效果具有原子性，带nx结尾的带有判断效果

##### string

- 设置key
  - set [key] [value]
    - 若key存在则覆盖value
  - 设置key同时获取旧的value：getset [key] [value]
  - 同时设置多个key：mset [key] [value] [key...] [value..]
  - 不覆盖式设置key：setnx [key] [value]
  - 不覆盖式批量设置key：msetnx [key] [value] [key...] [value...]
    - 具有事务的效果，其中一个失败该命令的效果全失效
- 设置key同时设置生存时间
  - setex [key] [seconds] [value]
  - psetex [key] [milliseconds] [value]
- 获取value：
  - get [key]
  - 范围获取key的value：getrange [key] [start] [end]
    - 取值范围[start,end]，从0开始
  - 同时获取多个key：mget [key] [key...]
- 获取字符串长度：strlen [key]

- Integer(String通用，但使用以下指令需要value为数值)：
  - key的value+1：
  - incr [key]
    - 若key不存在则创建key，默认+1即value=1
  - key的value-1：
    - decr [key]
      - 若key不存在则创建key，默认-1即value=-1
  - 设置key和value：
    - incrby [key] [increment]
      - 若key存在则在key的value值上**+**increment值
      - 若key不存在value=increment
    - decrby [key] [increment]
      - 若key存在则在key的value值上**-**increment值
      - 若key不存在value=-increment
- 拼接字符串或数字：append [key] [value]

##### hash

- 设置hash：
  - hset [hash] [key] [value]
    - [key] [value]为[hash]中的一个键值对
    - 若hash存在即在hash中添加[key] [value]
  - 批量设置hash的键值对：hmset [hash] [key] [value] [key...] [value...]
  - 带不可覆盖式：指令+nx
- 获取hash：
  - 获取value：hget [hash] [key]
  - 获取hash的所有key和value：
    - hgetall [hash]
      - 第一个值为key，第二个值为value，如此类推
  - 获取key的所有field：hkeys [hash]
  - 获取key的所有value：hvals [hash]
  - 批量获取key的value：hmget [hash] [key] [key...]
- 删除hash：
  - hdel [hash] [key] [value]
- 判断key中是否存在field：hexists [hash] [key]
- 获取key所有键值对数量：hlen [hash]

##### list

- 设置list：
  - lpush [list] [value] [value...]
    - 存储顺序为设置时的倒置
    - 若list存在即在list中push所有value值
- 获取list的长度：llen [list]
- 范围获取list的数据：lrange [list] [start] [end]
- 指定索引设置值：lset [list] [index] [value]
- 获取指定索引的值：lindex [list] [index]
- 移除第一个值：lpop [list]
  - 当list为空时删除list
- 移除最后一个值：rpop [list]
  - 当list为空时删除list

注：存储方式类似栈

##### set

- 设置set
  - sadd [set] [value] [value..]
    - 若set存在则在set添加value
- 获取set中值的个数：scard [set]
- 查看set所有值：smembers [set]
- 获取多个set的差(即找出不同)：sdiff [set] [set...]
  - 等于set1-(set1∩set2)
- 获取多个set的交集：sinter [set] [set...]
  - 等于set1∩set2
- 获取多个set的并集：sunion [set] [set...]
  - 等于set1∪set2
- 随机返回set中n个值：srandmember [set] [n]
- 判断值是否在set中：sismember [set] [value]
  - 返回1即存在，反之不存在
- 移除set中一个或多个值：srem [set] [value] [value...]
- 移除一个随机值并返回该值：spop [set]

注：set中不存在重复值且无序

##### sortedset

- 设置sortedset：
  - zadd [sortedset] [value] [score] [value...] [score...]
    - sortedset会根据score对value进行从小到大排序
- 获取sortedset中值的个数：zscard [set]
- 获取值的分数：zscore [sortedset] [value]
- 获取在分数区间的值的个数：zcount [sortedset] [start] [end]
  - [start,end]
- 返回sortedset的值的索引：zrank [sortedset] [value]
- 添加值的分数：zincrby [sortedset] [increment] [value]
  - 在原有分数上+increment，会改变排序
- 范围获取sortedset的值：zrange [sortedset] [start] [end] [withscores]
  - [start,end]，start和end均为下标
  - [withscores]为可选，添加即显示值的分数

注：sortedset时间复杂度为O(1)，且排序，值不可重复但分数可重复