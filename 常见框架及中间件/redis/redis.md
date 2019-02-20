下载安装包
wget http://download.redis.io/releases/redis-4.0.2.tar.gz
解压安装包并安装
tar xzf redis-4.0.2.tar.gz
cd redis-4.0.2
make install
Redis没有其他外部依赖，安装过程很简单。编译后在Redis源代码目录的src文件夹中可以找到若干个可执行程序，安装完后，在/usr/local/bin目录中可以找到刚刚安装的redis可执行文件。

启动和停止Redis
启动Redis
直接启动
直接运行redis-server即可启动Redis
[root@localhost bin]# redis-server

通过初始化脚本启动Redis
在Redis源代码目录的utils文件夹中有一个名为redis_init_script的初始化脚本文件。需要配置Redis的运行方式和持久化文件、日志文件的存储位置。步骤如下：
1、配置初始化脚本
首先将初始化脚本复制到/etc/init.d 目录中，文件名为 redis_端口号，其中端口号表示要让Redis监听的端口号，客户端通过该端口连接Redis。然后修改脚本第6行的REDISPORT变量的值为同样的端口号。
2、建立以下需要的文件夹。
目录名	Value
/etc/redis	存放Redis的配置文件
/var/redis/端口号	存放Redis的持久化文件
3、修改配置文件
首先将配置文件模板（redis-4.0.2/redis.conf）复制到/etc/redis 目录中，以端口号命名（如“6379.conf”），然后按照下表对其中的部分参数进行编辑。
参数	值	说明
daemonize	yes	使Redis以守护进程模式运行
pidfile	/var/run/redis_端口号.pid	设置Redis的PID文件位置
port	端口号	设置Redis监听的端口号
dir	/var/redis/端口号	设置持久化文件存放位置
现在也可以使用下面的命令来启动和关闭Redis了
/etc/init.d/redis_6379 start
/etc/init.d/redis_6379 stop

【重中之重】让Redis随系统自动启动，这还需要对Redis初始化脚本进行简单修改，执行命令：
vim /etc/init.d/redis_6379
在打开的redis初始化脚本文件头部第四行的位置，追加下面两句
# chkconfig: 2345 90 10 # description: Redis is a persistent key-value database
追加后效果如下：

上图红色框中就是追加的两行注释，添加完毕后进行保存，即可通过下面的命令将Redis加入系统启动项里了
//设置开机执行redis脚本
chkconfig redis_6379 on

通过上面的操作后，以后也可以直接用下面的命令对Redis进行启动和关闭了，如下
service redis_6379 start
service redis_6379 stop

经过上面的部署操作后，系统重启，Redis也会随着系统自动启动，并且上面的步骤里也配置了Redis持久化，下次启动系统或Redis时，有缓存数据不丢失的好处。
停止Redis
考虑到 Redis 有可能正在将内存中的数据同步到硬盘中，强行终止 Redis 进程可能会导致数据丢失。正确停止Redis的方式应该是向Redis发送SHUTDOWN命令，方法为：
redis-cli SHUTDOWN
当Redis收到SHUTDOWN命令后，会先断开所有客户端连接，然后根据配置执行持久化，最后完成退出。
Redis可以妥善处理 SIGTERM信号，所以使用 kill Redis 进程的 PID也可以正常结束Redis，效果与发送SHUTDOWN命令一样。

Redis命令
redis是一种高级的key:value存储系统，其中value支持五种数据类型：
1.字符串（strings）
2.字符串列表（lists）
3.字符串集合（sets）
4.有序字符串集合（sorted sets）
5.哈希（hashes）
而关于key，有几个点要提醒大家：
1.key不要太长，尽量不要超过1024字节，这不仅消耗内存，而且会降低查找的效率；
2.key也不要太短，太短的话，key的可读性会降低；
3.在一个项目中，key最好使用统一的命名模式，例如user:10000:passwd。
 
Strings
Redis存储结构是key:value，value是 strings数据类型。
命令：
语法：set key value
1） set name zhangsanfeng
a) 给Strings类型key是name 添加一个值。
2） get name
a) 获取key是name属性的值。
3） incr age
a) 给数字字符类型自动加1
b) 把数字字符类型自动转换成integer类型，然后执行再加上1
4） decr age
a) 给数字字符类型自动减1
b) 把数字字符类型自动转换成integer类型，然后执行减去1
5） incrby age 10
a) 给指定键值加速10
6） decrby age 10
a) 给指定键值减去10
Hash
Hash是集合类型，适合于用来存储对象。Java集合类型是用来存储对象。
存储数据结构分析：
第一种数据结构：
 
存取对象，使用一个key，使用一个key获取一个对象，必须使用反序列化。
缺点：
占用IO资源。
第二种数据结构：
 
缺点：
用户ID被多次使用，数据冗余。资源浪费。
第三种数据结构：
 
Redis存储结构：key是用户ID value:就是hash类型数据。
命令：
1） hset user username zhaowuji
a) 给user中Username属性设置一个值
2） hget user username
a) 获取User中Username属性的值
3） hdel user password …..
a) 删除User中属性
4） hsetnx user email 123@qq.com
a) 如果user中email属性值已经存在，不会覆盖
b) 如果不存在，设置值。
5） hmset user password 123 age 11
a) 同时设置多个值
6） Hmget user username age password
Lists
List集合数据结构：类似数组，数据是顺序存储。
List集合链表结构：通过指针从头指针查询到尾指针查找元素。
 
命令：
 
1） lpush mylist a b c d
 
a) 给list类型数据结构设置多个值
 
2） lrange mylist 0 -1
 
a) 获取mylist集合中所有值
 
b) 0：值链表开始位置
 
c) -1：链表的结束位置
 
3） lpop mylist
 
a) 出栈集合mylist：出栈链表头指针元素。
 
4） lrem mylist 3 a
 
a) 删除链表mylist中前3个等于a的值。
 
5） lset mylist 2 s
 
a) 给链表mylist集合中2角标位置设置一个值，覆盖原值。
 
6） linsert mylist after s b
 
a) 在集合链表mylsit中s元素后面插入一个b
 
Set
 
命令：
 
1） sadd myset a b c
 
a) 给set集合myset设置值：a b c
 
b)  Set集合元素值不允许重复
 
2） smembers myset
 
a) 获取集合myset中值
 
3） srem myset a b
 
a) 删除集合myset中元素
 
4） smove myset myset1 c
 
a) 把集合myset中的元素c移动到集合myset1中
 
 Sorted set
 
Set集合：有序集合。
 
 给set集合中每一元素都设置一个得分，根据得分排序。
 
 Set集合元素不允许重复，得分可以重复。
 
设置得分语法：ZADD key score member [score] [member]
 
命令：
 
1） zadd mysset 1 one 2 two 12 three 9 four 10 five
 
a) 给集合mysset集合添加5个元素，每一个元素都设置一个得分。
 
2） zcount mysset 1 10
 
a) 获取分数1到10的元素个数,默认是闭区间。
 
3） zcount mysset (1 10
 
a) 获取分数1到10的元素个数，左边是开区间（不包含1元素）
 
4） zcount mysset -inf +inf
 
a) 获取所有元素
 
b) –inf:最低值
 
c) +inf:最高值
 
5） zrange mysset 0 -1 withscores
 
a) 获取集合mysset中所有元素
 
b) 0:头部元素
 
c) -1表示尾部元素
 
d) Withscores：查询元素时候，把分数查询出来
 
6） zrangebyscore mysset 1 10 withscores limit 2 2
 
a) 根据分数大小来获取元素：
 
b) Limit分页获取值。
 
多数据库实例
 
Redis支持16个数据库实例，数据库实例角标从0—15，使用redis客户端登录redis服务器，默认登录0号数据库。
 
登录其他数据库实例：
 
登录语法：select 数据库实例ID（角标）
 
登录１号数据库：
 
127.0.0.1:6379> select 1
 
OK
 
127.0.0.1:6379[1]>
 
需求：把０号数据库数据移动１号数据库
 
命令：move user 1 //移动user到1号数据库
 
事务
 
事务命令：
 
开启事务：multi
 
提交事务：exec
 
回滚事务：discard
 
监听事务：watch（乐观锁）
 
用户A，用户B　同时读取商品数量itemNum=1，用户A，用户B都需要购买商品，商品数量需要减去1，使用乐观锁解决问题：
 
事务一致性
 
设计：商品数量添加，在添加过程中出一个错误，查看程序执行一致性。
 
127.0.0.1:6379> multi//开启事务
 
OK
 
127.0.0.1:6379> incr itemNum
 
QUEUED
 
127.0.0.1:6379> incr itemNum
 
QUEUED
 
127.0.0.1:6379> incrby itemNum 10
 
QUEUED
 
127.0.0.1:6379> lpop itemNum//使用操作list数据结构命令操作String类型，出现错误
 
QUEUED
 
127.0.0.1:6379> decr itemNum
 
QUEUED
 
127.0.0.1:6379> decrby itemNum 10
 
QUEUED
 
127.0.0.1:6379> exec//提交事务
 
1) (integer) 2
 
2) (integer) 3
 
3) (integer) 13
 
4) (error) WRONGTYPE Operation against a key holding the wrong kind of value
 
5) (integer) 12
 
6) (integer) 2
 
特点：redis事务不遵循ACID大一统理论，即使中间执行出现错误，后面不影响执行。
 
12.2 事务回滚
 
 
 
127.0.0.1:6379> mult//开启事务
 
OK
 
127.0.0.1:6379> incr itemNum
 
QUEUED
 
127.0.0.1:6379> incr itemNum
 
QUEUED
 
127.0.0.1:6379> discard//事务回滚
 
OK
 
127.0.0.1:6379> get itemNum
 
"2"
 
12.3 Watch
 
Watch监听事务：乐观锁
 
乐观锁：一旦发现被监听事务变量发生了变化，事务回滚。
 
127.0.0.1:6379> watch itemNum
 
OK
 
127.0.0.1:6379> multi
 
OK
 
127.0.0.1:6379> decr itemNum //当检测到itemNum发生变化时，此命令不执行。
 
QUEUED
 
127.0.0.1:6379> exec
 
(nil)
 
127.0.0.1:6379> get itemNum
 
"0"
 
 
 
 Redis持久化
 
Rdb（redis系统默认持久化策略）
 
Rdb持久化优点
 
1） 持久化文件将只包含一个文件
 
2） 对灾难恢复，主从复制 效率比较高。
 
3） 持久化工作：子进程
 
Rdb缺点
 
1） 数据安全性不是很好
 
Rdb数据持久化同步策略
 
 缺省情况下，Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率，在打开redis.conf文件之后，我们搜索save，可以看到下面的配置信息：
    save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。
    save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。
    save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。
 
 
 
 
 
Aof
 
Aof持久化优点
 
1） 根据redis aof同步策略，数据有更高安全性
 
Aof缺点
 
1） 性能比rdb低
工作原理
关于原理部分，我们主要来看RDB与AOF是如何完成持久化的，他们的过程是如何。
在介绍原理之前先说下Redis内部的定时任务机制，定时任务执行的频率可以在配置文件中通过 hz 10 来设置（这个配置表示1s内执行10次，也就是每100ms触发一次定时任务）。该值最大能够设置为：500，但是不建议超过：100，因为值越大说明执行频率越频繁越高，这会带来CPU的更多消耗，从而影响主进程读写性能。
定时任务使用的是Redis自己实现的 TimeEvent，它会定时去调用一些命令完成定时任务，这些任务可能会阻塞主进程导致Redis性能下降。因此我们在配置Redis时，一定要整体考虑一些会触发定时任务的配置，根据实际情况进行调整。
RDB的原理
在Redis中RDB持久化的触发分为两种：自己手动触发与Redis定时触发。
针对RDB方式的持久化，手动触发可以使用：
save：会阻塞当前Redis服务器，直到持久化完成，线上应该禁止使用。
bgsave：该触发方式会fork一个子进程，由子进程负责持久化过程，因此阻塞只会发生在fork子进程的时候。
而自动触发的场景主要是有以下几点：
根据我们的 save m n 配置规则自动触发；
从节点全量复制时，主节点发送rdb文件给从节点完成复制操作，主节点会触发 bgsave；
执行 debug reload 时；
执行 shutdown时，如果没有开启aof，也会触发。
由于 save 基本不会被使用到，我们重点看看 bgsave 这个命令是如何完成RDB的持久化的。

这里注意的是 fork 操作会阻塞，导致Redis读写性能下降。我们可以控制单个Redis实例的最大内存，来尽可能降低Redis在fork时的事件消耗。以及上面提到的自动触发的频率减少fork次数，或者使用手动触发，根据自己的机制来完成持久化。
AOF的原理
AOF的整个流程大体来看可以分为两步，一步是命令的实时写入（如果是 appendfsync everysec 配置，会有1s损耗），第二步是对aof文件的重写。
对于增量追加到文件这一步主要的流程是：命令写入=》追加到aof_buf =》同步到aof磁盘。那么这里为什么要先写入buf在同步到磁盘呢？如果实时写入磁盘会带来非常高的磁盘IO，影响整体性能。
aof重写是为了减少aof文件的大小，可以手动或者自动触发，关于自动触发的规则请看上面配置部分。fork的操作也是发生在重写这一步，也是这里会对主进程产生阻塞。
手动触发： bgrewriteaof，自动触发 就是根据配置规则来触发，当然自动触发的整体时间还跟Redis的定时任务频率有关系。
下面来看看重写的一个流程图：

对于上图有四个关键点补充一下：
1.在重写期间，由于主进程依然在响应命令，为了保证最终备份的完整性；因此它依然会写入旧的AOF file中，如果重写失败，能够保证数据不丢失。
2.为了把重写期间响应的写入信息也写入到新的文件中，因此也会为子进程保留一个buf，防止新写的file丢失数据。
3.重写是直接把当前内存的数据生成对应命令，并不需要读取老的AOF文件进行分析、命令合并。
4.AOF文件直接采用的文本协议，主要是兼容性好、追加方便、可读性高可认为修改修复。
不能是RDB还是AOF都是先写入一个临时文件，然后通过 rename 完成文件的替换工作。
从持久化中恢复数据
数据的备份、持久化做完了，我们如何从这些持久化文件中恢复数据呢？如果一台服务器上有既有RDB文件，又有AOF文件，该加载谁呢？
其实想要从这些文件中恢复数据，只需要重新启动Redis即可。我们还是通过图来了解这个流程：

启动时会先检查AOF文件是否存在，如果不存在就尝试加载RDB。那么为什么会优先加载AOF呢？因为AOF保存的数据更完整，通过上面的分析我们知道AOF基本上最多损失1s的数据。
性能与实践
通过上面的分析，我们都知道RDB的快照、AOF的重写都需要fork，这是一个重量级操作，会对Redis造成阻塞。因此为了不影响Redis主进程响应，我们需要尽可能降低阻塞。
1.降低fork的频率，比如可以手动来触发RDB生成快照、与AOF重写；
2.控制Redis最大使用内存，防止fork耗时过长；
3.使用更牛逼的硬件；
4.合理配置Linux的内存分配策略，避免因为物理内存不足导致fork失败。
在线上我们到底该怎么做？我提供一些自己的实践经验。
1.如果Redis中的数据并不是特别敏感或者可以通过其它方式重写生成数据，可以关闭持久化，如果丢失数据可以通过其它途径补回；
2.自己制定策略定期检查Redis的情况，然后可以手动触发备份、重写数据；
3.单机如果部署多个实例，要防止多个机器同时运行持久化、重写操作，防止出现内存、CPU、IO资源竞争，让持久化变为串行；
4.可以加入主从机器，利用一台从机器进行备份处理，其它机器正常响应客户端的命令；
5.RDB持久化与AOF持久化可以同时存在，配合使用。

