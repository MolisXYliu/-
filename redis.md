# Redis
## Redis版本
1.	3.2.5 for Linux 
2.	不用考虑在Windows环境下对Redis的支持
Redis官方没有提供对Windows环境的支持，是微软的开源小组开发了对Redis对Windows的支持.

## 安装步骤
1.	下载获得redis-3.2.5.tar.gz后将它放入我们的Linux目录/opt
2.	解压命令:tar -zxvf redis-3.2.5.tar.gz
3.	解压完成后进入目录:cd redis-3.2.5
4.	在redis-3.2.5目录下执行make命令
    - 运行Make命令时出现错误,提示 gcc：命令未找到 ,原因是因为当前Linux环境中并没有安装gcc 与 g++ 的环境
5.	安装gcc与g++
    - 能上网的情况:
      yum install gcc
      yum install gcc-c++
    - 不能上网[建议]
参考Linux课程中<<03_在VM上安装CentOS7>>中的第40步骤
6.	重新进入到Redis的目录中执行 make distclean后再执行make 命令.
7.	执行完make后，可跳过Redis test步骤，直接执行 make install

##  Redis的启动
1.	默认前台方式启动 
    - 直接执行redis-server 即可.启动后不能操作当前命令窗口
2.	推荐后台方式启动
    - 拷贝一份redis.conf配置文件到其他目录，例如根目录下的myredis目录  /myredis
    - 修改redis.conf文件中的一项配置 daemonize 将no 改为yes，代表后台启动
    - 执行配置文件进行启动 执行 redis-server /myredis/redis.conf

## 客户端访问
1.	使用redis-cli 命令访问启动好的Redis
    - 如果有多个Redis同时启动，则需指定端口号访问  redis-cli -p 端口号
2. 测试验证,通过 ping 命令 查看是否 返回 PONG

## 关闭Redis服务
1.	单实例关闭 
    - 如果还未通过客户端访问，可直接 redis-cli shutdown 
    - 如果已经进入客户端,直接 shutdown即可.
2.	多实例关闭
    - 指定端口关闭 redis-cli -p 端口号 shutdown

##  Redis的五大数据类型
### key
| 名称 | 效果 |
| :---: | :---: |
| keys  *  | 查看当前库的所有键|
| exists \<key> | 判断某个键是否存在 |
| type \<key> | 查看键的类型 |
| del \<key> | 删除某个键 |
| expire \<key> \<seconds> | 为键值设置过期时间，单位秒 |
| ttl \<key> | 查看还有多久过期,-1表示永不过期,-2表示已过期 |
| dbsize | 查看当前数据库中key的数量 |
| flushdb | 清空当前库 |
| Flushall | 通杀全部库 |

### String 
1.	String是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value
2.	String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
3.	String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M
4.	常用操作
get \<key>	查询对应键值  

set \<key> \<value>	添加键值对  

append \<key> \<value>	将给定的   \<value>追加到原值的末尾  

strlen \<key>	获取值的长度  

senx \<key> \<value>	只有在key 不存在时设置key的值  

incr \<key>	将key中存储的数字值增1
只能对数字值操作，如果为空，新增值为1  

decr \<key>	将key中存储的数字值减1
只能对数字之操作，如果为空,新增值为-1  

incrby /decrby \<key> 步长	将key中存储的数字值增减，自定义步长  

mset \<key1> \<value1> \<key2> \<value2>	同时设置一个或多个key-value对mget \<key1> \<key2>  \<key3>	同时获取一个或多个value
msetnx \<key1> \<value1> \<key2> \<value2>	同时设置一个或多个key-value对，当且仅当所有给定的key都不存在  

getrange \<key> <起始位置> <结束位置>	获得值的范围,类似java中的substring
setrange \<key> <起始位置> \<value>	用\<value>覆盖\<key>所存储的字符串值，从<起始位置>开始  

setex \<key> <过期时间> \<value>	设置键值的同时，设置过去时间，单位秒
getset \<key> \<value>	以新换旧,设置了新值的同时获取旧值

5.	详说 incr key 操作的原子性
    - 所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。
    - 在单线程中， 能够在单条指令中完成的操作都可以认为是" 原子操作"，因为中断只能发生于指令之间。
    - 在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。
    - Redis单命令的原子性主要得益于Redis的单线程
    - 思考: java中i++是否是原子操作?

## List
1.	单键多值
2.	Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。
3.	它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差
4. 常用操作
lpush/rpush  \<key>  \<value1>  \<value2>  	从左边/右边插入一个或多个值。  
lpop/rpop  \<key>	从左边/右边吐出一个值。值在键在，值光键亡。  
rpoplpush  \<key1>  \<key2>  	从\<key1>列表右边吐出一个值，插到\<key2>列表左边  
lrange \<key> \<start> \<stop>	按照索引下标获得元素(从左到右)  
lindex \<key> \<index>	按照索引下标获得元素(从左到右)  
llen \<key>	获得列表长度  
linsert \<key>  before \<value>  \<newvalue>	在\<value>的后面插入\<newvalue> 插入值  
lrem \<key> \<n>  \<value>	从左边删除n个value(从左到右)

## Set
1.	Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的
2.	Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表,所以添加，删除，查找的复杂度都是O(1)。
3.	常用操作
sadd \<key>  \<value1>  \<value2> ....	将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
smembers \<key>	取出该集合的所有值。
sismember \<key>  \<value>	判断集合\<key>是否为含有该\<value>值，有返回1，没有返回0
scard   \<key>	返回该集合的元素个数。  
srem \<key> \<value1> \<value2> ....	删除集合中的某个元素。  
spop \<key>  	随机从该集合中吐出一个值。  
srandmember \<key> \<n>	随机从该集合中取出n个值。  
不会从集合中删除
sinter \<key1> \<key2>  	返回两个集合的交集元素。  
sunion \<key1> \<key2>  	返回两个集合的并集元素。  
sdiff \<key1> \<key2>  	返回两个集合的差集元素。

## Hash
1.	Redis  hash 是一个键值对集合
2.	Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
3.	类似Java里面的Map<String,Object>
4.	分析一个问题: 现有一个JavaBean对象，在Redis中如何存?
5.	常用操作
hset \<key>  \<field>  \<value>	给<key>集合中的  <field>键赋值\<value>  
hget \<key1>  \<field>   	从\<key1>集合\<field> 取出 value  
hmset \<key1>  \<field1> \<value1> \<field2> \<value2>...   	批量设置hash的值  
hexists key  \<field>	查看哈希表 key 中，给定域 field 是否存在。  
hkeys \<key>   	列出该hash集合的所有field  
hvals \<key>    	列出该hash集合的所有value  
hincrby \<key> \<field>  \<increment>	为哈希表 key 中的域 field 的值加上增量 increment  
hsetnx \<key>  \<field> \<value>	将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在

##  zset (sorted set)
1.	Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个评分（score） ，这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。
2.	因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。
3.	常用操作
zadd  \<key> \<score1> \<value1>  \<score2> \<value2>...	将一个或多个 member 元素及其 score 值加入到有序集 key 当中  
zrange \<key>  \<start> \<stop>  [WITHSCORES]   	返回有序集 key 中，下标在\<start> \<stop>之间的元素
带WITHSCORES，可以让分数一起和值返回到结果集。  
zrangebyscore key min max [withscores] [limit offset count]	返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。  
zrevrangebyscore key max min [withscores] [limit offset count]	同上，改为从大到小排列。  
zincrby \<key> \<increment> \<value>	为元素的score加上增量
zrem  \<key>  \<value>  	删除该集合下，指定值的元素  
zcount \<key>  \<min>  \<max>	统计该集合，分数区间内的元素个数
zrank \<key>  \<value>	返回该值在集合中的排名，从0开始。


## Redis的相关配置
1.	计量单位说明,大小写不敏感

2.	include
类似jsp中的include，多实例的情况可以把公用的配置文件提取出来
3.	ip地址的绑定 bind
    - 默认情况bind=127.0.0.1只能接受本机的访问请求
    - 不写的情况下，无限制接受任何ip地址的访问
    - 生产环境肯定要写你应用服务器的地址
    - 如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的相应
4.	tcp-backlog
    - 可以理解是一个请求到达后至到接受进程处理前的队列.
    - backlog队列总和=未完成三次握手队列 +  已经完成三次握手队列
    - 高并发环境tcp-backlog 设置值跟超时时限内的Redis吞吐量决定
5.	timeout
一个空闲的客户端维持多少秒会关闭，0为永不关闭。
6.	tcp keepalive
对访问客户端的一种心跳检测，每个n秒检测一次，官方推荐设置为60秒
7.	daemonize
是否为后台进程
8.	pidfile
存放pid文件的位置，每个实例会产生一个不同的pid文件
9.	log level
四个级别根据使用阶段来选择，生产环境选择notice 或者warning
10.	log level
日志文件名称
11.	syslog
是否将Redis日志输送到linux系统日志服务中
12.	syslog-ident
日志的标志
13.	syslog-facility
输出日志的设备
14.	database
设定库的数量 默认16
15.	security
在命令行中设置密码

16.	maxclient
最大客户端连接数
17.	maxmemory
设置Redis可以使用的内存量。一旦到达内存使用上限，Redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。如果Redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，
那么Redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。
18.	Maxmemory-policy
    - volatile-lru：使用LRU算法移除key，只对设置了过期时间的键
    - allkeys-lru：使用LRU算法移除key
    - volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
    - allkeys-random：移除随机的key
    - volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
    - noeviction：不进行移除。针对写操作，只是返回错误信息
19.	Maxmemory-samples
设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小。
一般设置3到7的数字，数值越小样本越不准确，但是性能消耗也越小。

## Redis的Java客户端 Jedis
1.	Jedis所需要的jar包 ,可通过Maven的依赖引入
Commons-pool-1.6.jar
Jedis-2.1.0.jar
2.	使用Windows环境下Eclipse连接虚拟机中的Redis注意事项
    - 禁用Linux的防火墙：Linux(CentOS7)里执行命令 ： systemctl stop firewalld.service   
    - redis.conf中注释掉bind 127.0.0.1 ,然后 protect-mode no。
3. Jedis测试连通性
````java
  public static void main(String[] args) {
        Jedis jedis=new Jedis("192.168.119.132",6379);
        String ping = jedis.ping();
        System.out.println(ping);
        jedis.set("a","茉莉");
        String a = jedis.get("a");
        System.out.println(a);
        jedis.close();
    }
````
输出
````java
"C:\Program Files\Java\jdk-11.0.1\bin\java.exe" "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2020.1\lib\idea_rt.jar=49643:C:\Program Files\JetBrains\IntelliJ IDEA 2020.1\bin" -Dfile.encoding=UTF-8 -classpath C:\Users\wwwli\IdeaProjects\testRedis\out\production\testRedis1;C:\Users\wwwli\IdeaProjects\testRedis\testRedis1\lib\commons-pool2-2.4.2.jar;C:\Users\wwwli\IdeaProjects\testRedis\testRedis1\lib\jedis-2.8.1.jar com.lxy.test.test
PONG
茉莉
````

## Redis 事务 
### multi 、exec、discard
1.	从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，至到输入Exec后，Redis会将之前的命令队列中的命令依次执行。
2.	组队的过程中可以通过discard来放弃组队。

### 事务中的错误处理
1.	组队中某个命令出现了报告错误，执行时整个的所有队列会都会被取消。
2.	如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。
---
# 锁
悲观锁(Pessimistic Lock), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁

乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

---

### Redis事务的使用
1.	WATCH key[key….]
在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
2.	unwatch
取消 WATCH 命令对所有 key 的监视。
如果在执行 WATCH 命令之后， EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。
3.	三特性
    - 单独的隔离操作 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
    - 没有隔离级别的概念 队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题 
    - 不保证原子性 Redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 

---
## ab工具(模拟高并发)
1.	CentOS6 默认安装 ,CentOS7需要手动安装
2.	联网: yum install httpd-tools
无网络: 进入cd  /run/media/root/CentOS 7 x86_64/Packages
	    顺序安装
			 apr-1.4.8-3.el7.x86_64.rpm
     		 apr-util-1.5.2-6.el7.x86_64.rpm
     		 httpd-tools-2.4.6-67.el7.centos.x86_64.rpm   
3.	ab –n 请求数  -c 并发数  -p  指定请求数据文件
     -T  “application/x-www-form-urlencoded”   测试的请求

## LUA脚本
1. Lua 是一个小巧的脚本语言，Lua脚本可以很容易的被C/C++ 代码调用，也可以反过来调用C/C++的函数，Lua并没有提供强大的库，一个完整的Lua解释器不过200k，所以Lua不适合作为开发独立应用程序的语言，而是作为嵌入式脚本语言。
很多应用程序、游戏使用LUA作为自己的嵌入式脚本语言，以此来实现可配置性、可扩展性。这其中包括魔兽争霸地图、魔兽世界、博德之门、愤怒的小鸟等众多游戏插件或外挂
2. LUA脚本在Redis中的优势将复杂的或者多步的redis操作，写为一个脚本，一次提交给redis执行，减少反复连接redis的次数。提升性能。LUA脚本是类似redis事务，有一定的原子性，不会被其他命令插队，可以完成一些redis事务性的操作但是注意redis的lua脚本功能，只有在2.6以上的版本才可以使用。
3. 利用lua脚本淘汰用户，解决超卖问题。

## Redis 持久化
### RDB
1.	在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。
2.	备份是如何执行的
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。
3. 关于fork 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“写时复制技术”，一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。
4.	RDB保存的文件
在redis.conf中配置文件名称，默认为dump.rdb
5. RDB文件的保存路径
默认为Redis启动时命令行所在的目录下,也可以修改
6. 手动保存快照
    - save: 只管保存，其它不管，全部阻塞
    - bgsave:按照保存策略自动保存
7.	RDB的相关配置
    - stop-writes-on-bgsave-error yes
当Redis无法写入磁盘的话，直接关掉Redis的写操作
    - rdbcompression yes
进行rdb保存时，将文件压缩
    - rdbchecksum yes
在存储快照后，还可以让Redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
8. RDB的备份 与恢复
    - 备份:先通过config get dir  查询rdb文件的目录 , 将*.rdb的文件拷贝到别的地方
    - 恢复: 关闭Redis，把备份的文件拷贝到工作目录下,启动redis,备份数据会直接加载。
9.	RDB的优缺点
    - 优点: 节省磁盘空间,恢复速度快.
    - 缺点: 虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。	  在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就
	  会丢失最后一次快照后的所有修改

### AOF
1.	以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
2.	AOF默认不开启，需要手动在配置文件中配置
3.	可以在redis.conf中配置文件名称，默认为 appendonly.aof AOF文件的保存路径，同RDB的路径一致
4.	AOF和RDB同时开启，redis听谁的？
5.	AOF文件故障备份
AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载
6.	AOF文件故障恢复
如遇到AOF文件损坏，可通过
redis-check-aof  --fix  appendonly.aof   进行恢复
7.	AOF同步频率设置
    - 始终同步，每次Redis的写入都会立刻记入日志
    - 每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。
    - 把不主动进行同步，把同步时机交给操作系统。
8.	Rewrite
    - AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof。
    - Redis如何实现重写AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。
    - 何时重写 重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。
    - 系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。
9.	AOF的优缺点
    - 优点:
        - 备份机制更稳健，丢失数据概率更低。
        - 可读的日志文本，通过操作AOF稳健，可以处理误操作。
    - 缺点:
        - 比起RDB占用更多的磁盘空间
        - 恢复备份速度要慢
        - 每次读写都同步的话，有一定的性能压力。
---
###  RDB和AOF 用哪个好
-	官方推荐两个都启用。
-	如果对数据不敏感，可以选单独用RDB
-	不建议单独用 AOF，因为可能会出现Bug。
-	如果只是做纯内存缓存，可以都不用
---

## Redis主从复制
### 主从复制的目的
1.	读写分离，性能扩展
2.	容灾快速恢复

### 主从配置
1.	原则: 配从不配主
2.	步骤: 准备三个Redis实例，一主两从
     - 拷贝多个redis.conf文件include
	 - 开启daemonize yes
	 - Pid文件名字pidfile
	 - 指定端口port
	 - Log文件名字
	 - Dump.rdb名字dbfilename
	 - Appendonly 关掉或者换名字 
3.	info replication  打印主从复制的相关信息
4.	slaveof  \<ip>  \<port>   成为某个实例的从服务器

### 相关问题:
-	切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的123是否也可以复制 (是)
-	从机是否可以写？set可否？ (否)
-	主机shutdown后情况如何？从机是上位还是原地待命 (待命)
-	主机又回来了后，主机新增记录，从机还能否顺利复制 (可以)
-	其中一台从机down后情况如何？依照原有它能跟上大部队吗？ (能)

##  薪火相传模式演示
1.	上一个slave可以是下一个slave的Master，slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险.
	中途变更转向:会清除之前的数据，重新建立拷贝最新的
	风险是一旦某个slave宕机，后面的slave都没法备份

2.	反客为主(小弟上位)
当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。
用 slaveof  no one  将从机变为主机。

3.	哨兵模式 sentinel (推举大哥)
反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库.
4. 配置
    - 调整为一主二从模式
    - 自定义的/myredis目录下新建sentinel.conf文件
    - 在配置文件中填写内容
sentinel  monitor  mymaster  127.0.0.1  6379  1
		其中mymaster为监控对象起的服务器名称， 1 为 至少有多少个哨兵同意迁移的
		数量。
    - 启动哨兵
执行redis-sentinel  /myredis/sentinel.conf

##  Redis集群(需要先安装ruby环境 注意版本)
### 什么是集群
1.	Redis 集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。
2.	Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求
### 集群操作
1.	以集群的方式进入客户端
redis-cli  -c  -p  端口号
2.	通过cluster nodes 命令查看集群信息

3.	redis cluster 如何分配这六个节点
一个集群至少要有三个主节点。
选项 --replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。
分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。
4.	什么是slots 
    - 一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。
    - 集群中的每个节点负责处理一部分插槽。 举个例子， 如果一个集群可以有主节点， 其中：   
    - 
        节点 A 负责处理 0 号至 5500 号插槽。  

        节点 B 负责处理 5501 号至 11000 号插槽。  

        节点 C 负责处理 11001 号至 16383 号插槽
5.	在集群中录入值
    - 在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口.
    - redis-cli客户端提供了 –c 参数实现自动重定向。如 redis-cli  -c –p 6379 登入后，再录入、查询键值对可以自动重定向。
    - 不在一个slot下的键值，是不能使用mget,mset等多键操作。
    - 可以通过{}来定义组的概念，从而使key中{}内相同内容的键值对放到一个slot中去
6. 查询集群中的值
    - CLUSTER KEYSLOT \<key> 计算键 key 应该被放置在哪个槽上。
    - CLUSTER COUNTKEYSINSLOT \<slot> 返回槽 slot 目前包含的键值对数量
    - CLUSTER GETKEYSINSLOT \<slot> \<count> 返回 count 个 slot 槽中的键






