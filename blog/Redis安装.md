# **redis安装**

##### **稳定版为4.0.11所以我们下载4.0.11**

```
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
```

##### **安装gcc**（安装可忽略）

```
yum -y install gcc //安装gcc
rpm -qa|grep gcc //验证gcc是否安装成功：
```

##### TCL 安装（安装可忽略）

```
##去下载tcl 然后安装
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
sudo tar xzvf tcl8.6.1-src.tar.gz -C /usr/local/
cd /usr/local/tcl8.6.1/unix/
sudo ./configure
sudo make
sudo make install


##或者直接用yum安装
yum -y install tcl


make test //测试是否成功
```



##### 下载源码，解压缩后编译源码redis开始正式安装

```
tar -xzf redis-4.0.11.tar.gz
cd /usr/local/redis-4.0.11
make //编译
make install PREFIX=/usr/local/redis-4.0.11 //安装(使用 PREFIX 指定安装目录)
```

######  **PREFIX=/usr/local/redis-4.0.11/ 代表初始化时把一些配置文件放倒/usr/local/redis-4.0.11/ 目录中，此时你会发现/usr/local/redis-4.0.11/ 目录下会有个bin，bin下面是一些启动文件**

##### **启动Redis 服务**（当然这样启动肯定是不行的，所以我们要修改下redis.conf的参数）

```
cp redis.conf /usr/local/redis-4.0.11/bin/
cd /usr/local/redis-4.0.11/
cd bin/
./redis-server redis.conf
```

##### **修改redis.conf的参数配置**

```
vi /etc/redis.conf 

daemonize yes #查找daemonize no改为 yes以守护进程方式运行 即以后台运行方式去启动
port 6379 #默认端口
bind 127.0.0.1 #默认绑定本机所有ip地址，为了安全，可以只监听内网ip
protected-mode no #在redis3.2之后，redis增加了protected-mode，在这个模式下，即使注释掉了bind 127.0.0.1，再访问redisd时候还是报错
requirepass 123456 #设置redis数据库连接密码


#修改dir ./为绝对路径, 默认的话redis-server启动时会在当前目录生成或读取dump.rdb 所以如果在根目录下执行redis-server /etc/redis.conf的话, 读取的是根目录下的dump.rdb,为了使redis-server可在任意目录下执行 所以此处将dir改为绝对路径 

dir /usr/local/redis-4.0.11/bin

#修改appendonly为yes 
#指定是否在每次更新操作后进行日志记录， Redis在默认情况下是异步的把数据写入磁盘， 
#如果不开启，可能会在断电时导致一段时间内的数据丢失。 因为 redis本身同步数据文件是按上面save条件来同步的， 
#所以有的数据会在一段时间内只存在于内存中。默认为no 

appendonly yes 

#redis 日志生成位置

logfile "/app/log/redis.log"
```

##### **将 Redis 添加到环境变量中：** 

```
 vi /etc/profile //在最后添加以下内容
```

```
## Redis env 
export PATH=$PATH:/usr/local/redis-4.0.11/bin 

source /etc/profile  //使配置生效
```

##### **现在就可以直接使用 redis-cli 等 redis 命令了:**

```
./redis-cli 
```

#### 使用redis启动脚本设置开机自启动

```
#!/bin/sh
#说明启动优先级
# chkconfig:   2345 90 10
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO
#端口
REDISPORT=6379
REDIS=redis
#服务启动脚本位置
EXEC=/usr/local/redis-4.0.11/bin/redis-server
#客户端连接脚本位置
CLIEXEC=/usr/local/redis-4.0.11/bin/redis-cli
#启动PID所在位置
PIDFILE=/var/run/redis_${REDISPORT}.pid
#启动配置文件所在位置
CONF="/usr/local/redis-4.0.11/bin/${REDIS}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

##### 将启动脚本复制到/etc/init.d目录下，本例将启动脚本命名为redisd（通常都以d结尾表示是后台自启动服务）

```
cp redis_init_script /etc/init.d/redisd-6379
```

##### 设置为开机自启动，直接配置开启自启动 chkconfig redisd-6379 on 

```
chkconfig redisd-6379 on
```

##### 后面就可以同过下面命令进行启动或者关闭redis了

```
#手动打开服务
service redisd-6379 start
#关闭服务
service redisd-6379 stop
```



```
JedisPoolConfig config = new JedisPoolConfig();

//连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true
config.setBlockWhenExhausted(true);

//设置的逐出策略类名, 默认DefaultEvictionPolicy(当连接超过最大空闲时间,或连接数超过最大空闲连接数)
config.setEvictionPolicyClassName("org.apache.commons.pool2.impl.DefaultEvictionPolicy");

//是否启用pool的jmx管理功能, 默认true
config.setJmxEnabled(true);

//MBean ObjectName = new ObjectName("org.apache.commons.pool2:type=GenericObjectPool,name=" + "pool" + i); 默认为"pool", JMX不熟,具体不知道是干啥的...默认就好.
config.setJmxNamePrefix("pool");

//是否启用后进先出, 默认true
config.setLifo(true);

//最大空闲连接数, 默认8个
config.setMaxIdle(8);

//最大连接数, 默认8个
config.setMaxTotal(8);

//获取连接时的最大等待毫秒数(如果设置为阻塞时BlockWhenExhausted),如果超时就抛异常, 小于零:阻塞不确定的时间,  默认-1
config.setMaxWaitMillis(-1);

//逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
config.setMinEvictableIdleTimeMillis(1800000);

//最小空闲连接数, 默认0
config.setMinIdle(0);

//每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
config.setNumTestsPerEvictionRun(3);

//对象空闲多久后逐出, 当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出,不再根据MinEvictableIdleTimeMillis判断  (默认逐出策略)   
config.setSoftMinEvictableIdleTimeMillis(1800000);

//在获取连接的时候检查有效性, 默认false
config.setTestOnBorrow(false);

//在空闲时检查有效性, 默认false
config.setTestWhileIdle(false);

//逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
config.setTimeBetweenEvictionRunsMillis(-1);

JedisPool pool = new JedisPool(config, "localhost");
```



```
#redis.conf
# Redis configuration file example.
# ./redis-server /path/to/redis.conf

################################## INCLUDES ###################################
#这在你有标准配置模板但是每个redis服务器又需要个性设置的时候很有用。
# include /path/to/local.conf
# include /path/to/other.conf

################################ GENERAL #####################################
#是否在后台执行，yes：后台运行；no：不是后台运行（老版本默认）
daemonize yes
#3.2里的参数，是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。要是开启了密码 和bind，可以开启。否 则最好关闭，设置为no。
protected-mode yes

#redis的进程文件
pidfile "/var/run/redis/redis_7021.pid"

#redis监听的端口号。
port 7021

#此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p。
tcp-backlog 511

#指定 redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求
bind 192.168.1.55

#配置unix socket来让redis支持监听本地连接。
unixsocket "/var/run/redis/redis_7021.sock"
#配置unix socket使用文件的权限
# unixsocketperm 700

# 此参数为设置客户端空闲超过timeout，服务端会断开连接，为0则服务端不会主动断开连接，不能小于0。
timeout 0

#tcp keepalive参数。如果设置不为0，就使用配置tcp的SO_KEEPALIVE值，使用keepalive有两个好处:检测挂掉的对端。降低中间设备出问题而导致网络看似连接却已经与对端端口的问题。在Linux内核中，设置了keepalive，redis会定时给对端发送ack。检测到对端关闭需要两倍的设置值。
tcp-keepalive 0

#指定了服务端日志的级别。级别包括：debug（很多信息，方便开发、测试），verbose（许多有用的信息，但是没有debug级别信息多），notice（适当的日志级别，适合生产环境），warn（只有非常重要的信息）
loglevel notice

#指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null。
logfile "/var/log/redis/redis_7021.log"

#是否打开记录syslog功能
# syslog-enabled no

#syslog的标识符。
# syslog-ident redis

#日志的来源、设备
# syslog-facility local0

#数据库的数量，默认使用的数据库是DB 0。可以通过”SELECT “命令选择一个db
databases 16

################################ SNAPSHOTTING ################################
# 快照配置
# 注释掉“save”这一行配置项就可以让保存数据库功能失效
# 设置sedis进行数据库镜像的频率。
# 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化）
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化）
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）
#save 900 1
#save 300 10
#save 60 10000

#当RDB持久化出现错误后，是否依然进行继续进行工作，yes：不能进行工作，no：可以继续进行工作，可以通过info中的rdb_last_bgsave_status了解RDB持久化是否有错误
stop-writes-on-bgsave-error yes

#使用压缩rdb文件，rdb文件压缩使用LZF压缩算法，yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间
rdbcompression yes

#是否校验rdb文件。从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置。
rdbchecksum no

#rdb文件的名称
dbfilename "dump.rdb"

#数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
dir "/var/lib/redis_7021"

################################# REPLICATION #################################
#复制选项，slave复制对应的master。
# slaveof <masterip> <masterport>

#如果master设置了requirepass，那么slave要连上master，需要有master的密码才行。masterauth就是用来配置master的密码，这样可以在连上master后进行认证。
masterauth "XYZ"

#当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：1) 如果slave-serve-stale-data设置为yes(默认设置)，从库会继续响应客户端的请求。2) 如果slave-serve-stale-data设置为no，除去INFO和SLAVOF命令之外的任何请求都会返回一个错误”SYNC with master in progress”。
slave-serve-stale-data yes

#作为从服务器，默认情况下是只读的（yes），可以修改成NO，用于写（不建议）。
slave-read-only yes

#是否使用socket方式复制数据。目前redis复制提供两种方式，disk和socket。如果新的slave连上来或者重连的slave无法部分同步，就会执行全量同步，master会生成rdb文件。有2种方式：disk方式是master创建一个新的进程把rdb文件保存到磁盘，再把磁盘上的rdb文件传递给slave。socket是master创建一个新的进程，直接把rdb文件以socket的方式发给slave。disk方式的时候，当一个rdb保存的过程中，多个slave都能共享这个rdb文件。socket的方式就的一个个slave顺序复制。在磁盘速度缓慢，网速快的情况下推荐用socket方式。
repl-diskless-sync no

#diskless复制的延迟时间，防止设置为0。一旦复制开始，节点不会再接收新slave的复制请求直到下一个rdb传输。所以最好等待一段时间，等更多的slave连上来。
repl-diskless-sync-delay 5

#slave根据指定的时间间隔向服务器发送ping请求。时间间隔可以通过 repl_ping_slave_period 来设置，默认10秒。
repl-ping-slave-period 5

#复制连接超时时间。master和slave都有超时时间的设置。master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。slave检测到上次和master交互的时间超过repl-timeout，则认为master离线。需要注意的是repl-timeout需要设置一个比repl-ping-slave-period更大的值，不然会经常检测到超时。
repl-timeout 60

#是否禁止复制tcp链接的tcp nodelay参数，可传递yes或者no。默认是no，即使用tcp nodelay。如果master设置了yes来禁止tcp nodelay设置，在把数据复制给slave的时候，会减少包的数量和更小的网络带宽。但是这也可能带来数据的延迟。默认我们推荐更小的延迟，但是在数据量传输很大的场景下，建议选择yes。
repl-disable-tcp-nodelay no

#复制缓冲区大小，这是一个环形复制缓冲区，用来保存最新复制的命令。这样在slave离线的时候，不需要完全复制master的数据，如果可以执行部分同步，只需要把缓冲区的部分数据复制给slave，就能恢复正常复制状态。缓冲区的大小越大，slave离线的时间可以更长，复制缓冲区只有在有slave连接的时候才分配内存。没有slave的一段时间，内存会被释放出来，默认1m。
repl-backlog-size 32mb

#master没有slave一段时间会释放复制缓冲区的内存，repl-backlog-ttl用来设置该时间长度。单位为秒。
repl-backlog-ttl 3600

#当master不可用，Sentinel会根据slave的优先级选举一个master。最低的优先级的slave，当选master。而配置成0，永远不会被选举。
slave-priority 100

#redis提供了可以让master停止写入的方式，如果配置了min-slaves-to-write，健康的slave的个数小于N，mater就禁止写入。master最少得有多少个健康的slave存活才能执行写命令。这个配置虽然不能保证N个slave都一定能接收到master的写操作，但是能避免没有足够健康的slave的时候，master不能写入来避免数据丢失。设置为0是关闭该功能。
# min-slaves-to-write 3

#延迟小于min-slaves-max-lag秒的slave才认为是健康的slave。
# min-slaves-max-lag 10

# 设置1或另一个设置为0禁用这个特性。
# Setting one or the other to 0 disables the feature.
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.

################################## SECURITY ###################################
#requirepass配置可以让用户使用AUTH命令来认证密码，才能使用其他命令。这让redis可以使用在不受信任的网络中。为了保持向后的兼容性，可以注释该命令，因为大部分用户也不需要认证。使用requirepass的时候需要注意，因为redis太快了，每秒可以认证15w次密码，简单的密码很容易被攻破，所以最好使用一个更复杂的密码。
requirepass "XYZ"

#把危险的命令给修改成其他名称。比如CONFIG命令可以重命名为一个很难被猜到的命令，这样用户不能使用，而内部工具还能接着使用。
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command SHUTDOWN REDIS_SHUTDOWN
rename-command FLUSHDB REDIS_FLUSHDB
rename-command FLUSHALL REDIS_FLUSHALL
rename-command KEYS REDIS_KEYS
#rename-command CONFIG REDIS_CONFIG
#rename-command SLAVEOF REDIS_SLAVEOF
#设置成一个空的值，可以禁止一个命令
# rename-command CONFIG ""
################################### LIMITS ####################################

# 设置能连上redis的最大客户端连接数量。默认是10000个客户端连接。由于redis不区分连接是客户端连接还是内部打开文件或者和slave连接等，所以maxclients最小建议设置到32。如果超过了maxclients，redis会给新的连接发送’max number of clients reached’，并关闭连接。
# maxclients 10000

#redis配置的最大内存容量。当内存满了，需要配合maxmemory-policy策略进行处理。注意slave的输出缓冲区是不计算在maxmemory内的。所以为了防止主机内存使用完，建议设置的maxmemory需要更小一些。
maxmemory 512mb

#内存容量超过maxmemory后的处理策略。
#volatile-lru：利用LRU算法移除设置过过期时间的key。
#volatile-random：随机移除设置过过期时间的key。
#volatile-ttl：移除即将过期的key，根据最近过期时间来删除（辅以TTL）
#allkeys-lru：利用LRU算法移除任何key。
#allkeys-random：随机移除任何key。
#noeviction：不移除任何key，只是返回一个写错误。
#上面的这些驱逐策略，如果redis没有合适的key驱逐，对于写命令，还是会返回错误。redis将不再接收写请求，只接收get请求。写命令包括：set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort。
maxmemory-policy allkeys-lru

#lru检测的样本数。使用lru或者ttl淘汰算法，从需要淘汰的列表中随机选择sample个key，选出闲置时间最长的key移除。
# maxmemory-samples 5

############################## APPEND ONLY MODE ###############################
#默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了。但是redis如果中途宕机，会导致可能有几分钟的数据丢失，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性。Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。
appendonly no

#aof文件名
appendfilename "appendonly.aof"

#aof持久化策略的配置
#no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
#always表示每次写入都执行fsync，以保证数据同步到磁盘。
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。
appendfsync everysec

# 在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no。如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，建议yes。Linux的默认fsync策略是30秒。可能丢失30秒数据。
no-appendfsync-on-rewrite yes

#aof自动重写配置。当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候Redis能够调用bgrewriteaof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。
auto-aof-rewrite-percentage 100
#设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写
auto-aof-rewrite-min-size 64mb

#aof文件可能在尾部是不完整的，当redis启动的时候，aof文件的数据被载入内存。重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项（redis宕机或者异常终止不会造成尾部不完整现象。）出现这种现象，可以选择让redis退出，或者导入尽可能多的数据。如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。
aof-load-truncated yes

################################ LUA SCRIPTING ###############################
# 如果达到最大时间限制（毫秒），redis会记个log，然后返回error。当一个脚本超过了最大时限。只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。要是已经调用了write，只能用第二个命令杀。
lua-time-limit 5000

################################ REDIS CLUSTER ###############################
#集群开关，默认是不开启集群模式。
#cluster-enabled yes

#集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件，请确保与实例运行的系统中配置文件名称不冲突
#cluster-config-file nodes-7021.conf

#节点互连超时的阀值。集群节点超时毫秒数
#cluster-node-timeout 30000

#在进行故障转移的时候，全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了，导致数据过于陈旧，这样的slave不应该被提升为master。该参数就是用来判断slave节点与master断线的时间是否过长。判断方法是：
#比较slave断开连接的时间和(node-timeout * slave-validity-factor) + repl-ping-slave-period
#如果节点超时时间为三十秒, 并且slave-validity-factor为10,假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移
#可能出现由于某主节点失联却没有从节点能顶上的情况，从而导致集群不能正常工作，在这种情况下，只有等到原来的主节点重新回归到集群，集群才恢复运作
#如果设置成０，则无论从节点与主节点失联多久，从节点都会尝试升级成主节
#cluster-slave-validity-factor 10

#master的slave数量大于该值，slave才能迁移到其他孤立master上，如这个参数若被设为2，那么只有当一个主节点拥有2 个可工作的从节点时，它的一个从节点会尝试迁移。
#主节点需要的最小从节点数，只有达到这个数，主节点失败时，它从节点才会进行迁移。
# cluster-migration-barrier 1

#默认情况下，集群全部的slot有节点分配，集群状态才为ok，才能提供服务。设置为no，可以在slot没有全部分配的时候提供服务。不建议打开该配置，这样会造成分区的时候，小分区的master一直在接受写请求，而造成很长时间数据不一致。
#在部分key所在的节点不可用时，如果此参数设置为”yes”(默认值), 则整个集群停止接受操作；如果此参数设置为”no”，则集群依然为可达节点上的key提供读操作
#cluster-require-full-coverage yes

################################## SLOW LOG ###################################
###slog log是用来记录redis运行中执行比较慢的命令耗时。当命令的执行超过了指定时间，就记录在slow log中，slog log保存在内存中，所以没有IO操作。
#执行时间比slowlog-log-slower-than大的请求记录到slowlog里面，单位是微秒，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。
slowlog-log-slower-than 10000

#慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。这个长度没有限制。只要有足够的内存就行。你可以通过 SLOWLOG RESET 来释放内存。
slowlog-max-len 128

################################ LATENCY MONITOR ##############################
#延迟监控功能是用来监控redis中执行比较缓慢的一些操作，用LATENCY打印redis实例在跑命令时的耗时图表。只记录大于等于下边设置的值的操作。0的话，就是关闭监视。默认延迟监控功能是关闭的，如果你需要打开，也可以通过CONFIG SET命令动态设置。
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################
#键空间通知使得客户端可以通过订阅频道或模式，来接收那些以某种方式改动了 Redis 数据集的事件。因为开启键空间通知功能需要消耗一些 CPU ，所以在默认配置下，该功能处于关闭状态。
#notify-keyspace-events 的参数可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知：
##K 键空间通知，所有通知以 __keyspace@__ 为前缀
##E 键事件通知，所有通知以 __keyevent@__ 为前缀
##g DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知
##$ 字符串命令的通知
##l 列表命令的通知
##s 集合命令的通知
##h 哈希命令的通知
##z 有序集合命令的通知
##x 过期事件：每当有过期键被删除时发送
##e 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送
##A 参数 g$lshzxe 的别名
#输入的参数中至少要有一个 K 或者 E，否则的话，不管其余的参数是什么，都不会有任何 通知被分发。详细使用可以参考http://redis.io/topics/notifications

notify-keyspace-events "e"

############################### ADVANCED CONFIG ###############################
#数据量小于等于hash-max-ziplist-entries的用ziplist，大于hash-max-ziplist-entries用hash
hash-max-ziplist-entries 512
#value大小小于等于hash-max-ziplist-value的用ziplist，大于hash-max-ziplist-value用hash。
hash-max-ziplist-value 64

#数据量小于等于list-max-ziplist-entries用ziplist，大于list-max-ziplist-entries用list。
list-max-ziplist-entries 512
#value大小小于等于list-max-ziplist-value的用ziplist，大于list-max-ziplist-value用list。
list-max-ziplist-value 64

#数据量小于等于set-max-intset-entries用iniset，大于set-max-intset-entries用set。
set-max-intset-entries 512

#数据量小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset。
zset-max-ziplist-entries 128
#value大小小于等于zset-max-ziplist-value用ziplist，大于zset-max-ziplist-value用zset。
zset-max-ziplist-value 64

#value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse），大于hll-sparse-max-bytes使用稠密的数据结构（dense）。一个比16000大的value是几乎没用的，建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。
hll-sparse-max-bytes 3000

#Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。当你的使用场景中，有非常严格的实时性需要，不能够接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存。
activerehashing yes

##对客户端输出缓冲进行限制可以强迫那些不从服务器读取数据的客户端断开连接，用来强制关闭传输缓慢的客户端。
#对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制，因为如果没有寻问，他们是不会接收数据的。
client-output-buffer-limit normal 0 0 0
#对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接。
client-output-buffer-limit slave 256mb 64mb 60
#对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接。
client-output-buffer-limit pubsub 32mb 8mb 60

#redis执行任务的频率为1s除以hz。
hz 10

#在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB执行一次fsync。这对于把文件写入磁盘是有帮助的，可以避免过大的延迟峰值。
aof-rewrite-incremental-fsync yes
```

##### redis性能查看与监控常用工具

```
redis-benchmark -h localhost -p 6379 -c 100 -n 100000 
```

100个并发连接，100000个请求，检测host为localhost 端口为6379的redis服务器性能

```
# Server
redis_version:2.6.9
redis_git_sha1:00000000
redis_git_dirty:0
redis_mode:standalone
os:Linux 3.4.9-gentoo x86_64
arch_bits:64
multiplexing_api:epoll            # redis的事件循环机制
gcc_version:4.6.3
process_id:18926
run_id:df8ad7574f3ee5136e8be94aaa6602a0079704cc    # 标识redis server的随机值
tcp_port:6379
uptime_in_seconds:120            # redis server启动的时间(单位s)
uptime_in_days:0                # redis server启动的时间(单位d)
lru_clock:321118                # Clock incrementing every minute, for LRU management TODO 不清楚是如何计算的

# Clients
connected_clients:3                # 连接的客户端数
client_longest_output_list:0    # 当前客户端连接的最大输出列表    TODO
client_biggest_input_buf:0        # 当前客户端连接的最大输入buffer TODO
blocked_clients:0                # 被阻塞的客户端数

# Memory
used_memory:573456                # 使用内存，单位B
used_memory_human:560.02K        # human read显示使用内存
used_memory_rss:1798144            # 系统给redis分配的内存（即常驻内存）
used_memory_peak:551744            # 内存使用的峰值大小
used_memory_peak_human:538.81K    # human read显示内存使用峰值
used_memory_lua:31744            # lua引擎使用的内存
mem_fragmentation_ratio:3.14    # used_memory_rss/used_memory比例，一般情况下，used_memory_rss略高于used_memory，当内存碎片较多时，则mem_fragmentation_ratio会较大，可以反映内存碎片是否很多
mem_allocator:jemalloc-3.3.1    # 内存分配器

# Persistence
##########################
# rdb和aof事redis的两种持久化机制
#
# rdb是通过配置文件设置save的时间的改动数量来操作
# 把上次改动后的数据达到设置的指标后保存到db
# 如果中间发生了crash，则数据会丢失
# 这种策略被叫做快照
#
# aof是持续的把写操作执行写入一个类似日志的文件
# 但是会影响应能
# 分为appendfsync always和appendfsync eversec
# 前者每次写操作都同步，数据安全性高，但是特别消耗性能
# 后者每秒同步一次，如果发生crash，则可能会丢失1s的数据
##########################
loading:0                        #
rdb_changes_since_last_save:0    # 自上次dump后rdb的改动
rdb_bgsave_in_progress:0        # 标识rdb save是否进行中
rdb_last_save_time:1366359865    # 上次save的时间戳
rdb_last_bgsave_status:ok        # 上次的save操作状态
rdb_last_bgsave_time_sec:-1        # 上次rdb save操作使用的时间(单位s)
rdb_current_bgsave_time_sec:-1    # 如果rdb save操作正在进行，则是所使用的时间
----------------------------
aof_enabled:0                    # 是否开启aof，默认没开启
aof_rewrite_in_progress:0        # 标识aof的rewrite操作是否在进行中
aof_rewrite_scheduled:0            # 标识是否将要在rdb save操作结束后执行
aof_last_rewrite_time_sec:-1    # 上次rewrite操作使用的时间(单位s)
aof_current_rewrite_time_sec:-1 # 如果rewrite操作正在进行，则记录所使用的时间
aof_last_bgrewrite_status:ok    # 上次rewrite操作的状态
-----------------------------
# 开启aof后增加的一些info信息
aof_current_size:0                # aof当前大小
aof_base_size:0                    # aof上次启动或rewrite的大小
aof_pending_rewrite:0            # 同上面的aof_rewrite_scheduled
aof_buffer_length:0                # aof buffer的大小
aof_rewrite_buffer_length:0        # aof rewrite buffer的大小
aof_pending_bio_fsync:0            # 后台IO队列中等待fsync任务的个数
aof_delayed_fsync:0                # 延迟的fsync计数器 TODO
-----------------------------

# Stats
total_connections_received:7    # 自启动起连接过的总数
total_commands_processed:7        # 自启动起运行命令的总数
instantaneous_ops_per_sec:0        # 每秒执行的命令个数
rejected_connections:0            # 因为最大客户端连接书限制，而导致被拒绝连接的个数
expired_keys:0                    # 自启动起过期的key的总数
evicted_keys:0                    # 因为内存大小限制，而被驱逐出去的键的个数
keyspace_hits:0                    # 在main dictionary(todo)中成功查到的key个数
keyspace_misses:0                # 同上，未查到的key的个数
pubsub_channels:0                # 发布/订阅频道数
pubsub_patterns:0                # 发布/订阅模式数
latest_fork_usec:0                # 上次的fork操作使用的时间（单位ms）
##########################
# pubsub是一种消息传送的方式，分为频道和模式两种
# 消息不支持持久化，消息方中断后再连接，前面的消息就会没了
# 频道是指通过SUBSCRIBE指定一个固定的频道来订阅
# 模式是指通过PSUBSCRIBE模式匹配来订阅相关的匹配给定模式的频道
##########################

# Replication
role:master                        # 角色
connected_slaves:1                # 连接的从库数
slave0:127.0.0.1,7777,online
-----------------------------
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0        # 标识主redis正在同步到从redis
slave_priority:100
slave_read_only:1
connected_slaves:0

# CPU
used_cpu_sys:0.00            # redis server的sys cpu使用率
used_cpu_user:0.12            # redis server的user cpu使用率
used_cpu_sys_children:0.00    # 后台进程的sys cpu使用率
used_cpu_user_children:0.00    # 后台进程的user cpu使用率

# Keyspace
db0:keys=2,expires=0
db1:keys=1,expires=0# Server
redis_version:5.0.4                               #redis版本
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:964fe9af98041665
redis_mode:standalone                             #运行模式，单机或集群
os:Linux 3.10.0-693.21.1.el7.x86_64 x86_64        #系统版本
arch_bits:64                                      #架构，32位或64位
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:4.8.5                                 #编译redis时所使用的gcc版本
process_id:2939                                   #redis服务器的进程id
run_id:11b5694d024d8c728c1448ec4163fb0c22b86375   #redis服务器的随机标识符(用于sentinel和集群)
tcp_port:6379                                     #redis服务监听端口
uptime_in_seconds:18316                           #redis服务启动时长，单位为秒
uptime_in_days:0                                  #redis服务启动时长，单位为天
hz:10                             #redis内部调度（进行关闭timeout的客户端，删除过期key等等）频率
configured_hz:10
lru_clock:4564768
executable:/usr/local/redis/redis-server          #执行文件位置
config_file:/usr/local/redis/./redis.conf         #配置文件位置

# Clients
connected_clients:2                               #已连接的客户端数(不包括通过slave连接的客户端)
client_recent_max_input_buffer:2                  
client_recent_max_output_buffer:0                 
blocked_clients:0                 #正在等待阻塞命令(BLPOP、BRPOP、BRPOPLPUSH)的客户端的数量

# Memory
used_memory:8985032                               #由redis分配器分配的内存总量，以字节为单位
used_memory_human:8.57M                           #易读方式   
used_memory_rss:15175680          #从操作系统的角度，返回redis已分配的内存总量(俗称常驻集大小)
used_memory_rss_human:14.47M        
used_memory_peak:14859000                         #redis的内存消耗峰值(以字节为单位) 
used_memory_peak_human:14.17M
used_memory_peak_perc:60.47%                      #峰值内存超出分配内存（used_memory）的百分比
used_memory_overhead:5407864      #服务器为管理其内部数据结构而分配的所有开销的字节总和
used_memory_startup:862032                        #Redis在启动时消耗的初始内存量（以字节为单位）
used_memory_dataset:3577168       
used_memory_dataset_perc:44.04%
allocator_allocated:8951208
allocator_active:15137792
allocator_resident:15137792
total_system_memory:512077824                     #系统内存总量
total_system_memory_human:488.36M                 
used_memory_lua:37888                             #Lua引擎使用的字节量
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0                                       #配置设置的最大可使用内存值
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.69        
allocator_frag_bytes:6186584
allocator_rss_ratio:1.00
allocator_rss_bytes:0
rss_overhead_ratio:1.00
rss_overhead_bytes:37888
mem_fragmentation_ratio:1.70      #used_memory_rss和used_memory之间的比率
mem_fragmentation_bytes:6224472   #used_memory_rss和used_memory之间的差值，单位字节
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:66616
mem_aof_buffer:0
mem_allocator:libc                                #内存分配器，在编译时选择
active_defrag_running:0                           #指示活动碎片整理是否处于活动状态的标志
lazyfree_pending_objects:0

# Persistence
loading:0                                         #服务器是否正在载入持久化rdb文件
rdb_changes_since_last_save:0                     #自上次rdb持久化以来发生改变的数值
rdb_bgsave_in_progress:0                          #服务器是否正在创建rdb文件
rdb_last_save_time:1564845192                     #最后一次成功rdb持久化的时间戳
rdb_last_bgsave_status:ok                         #最后一次rdb持久化是否成功
rdb_last_bgsave_time_sec:0                        #最后一次成功生成rdb文件耗时秒数
rdb_current_bgsave_time_sec:-1                    #当前bgsave已耗费的时间（如果有）
rdb_last_cow_size:438272                          #上次rbd保存操作期间写时复制分配的字节大小
aof_enabled:0                                     #aof功能是否开启
aof_rewrite_in_progress:0                         #标识aof的rewrite操作是否在进行中
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1                      #最后一次aof rewrite耗费的时长
aof_current_rewrite_time_sec:-1                   #当前rewrite已耗费的时间（如果有）
aof_last_bgrewrite_status:ok                      #最后一次bgrewrite是否成功
aof_last_write_status:ok                          #上次aof写入状态
aof_last_cow_size:0                      #上次AOF重写操作期间写时复制分配的大小（以字节为单位）

如果激活了AOF，则会添加以下附加字段：
aof_current_size:4201740                          #aof当前尺寸
aof_base_size:4201687                    #服务器启动时或者aof重写最近一次执行之后aof文件的大小
aof_pending_rewrite:0                    #是否有aof重写操作在等待rdb文件创建完毕之后执行?
aof_buffer_length:0                               #aof buffer的大小
aof_rewrite_buffer_length:0                       #aof rewrite buffer的大小
aof_pending_bio_fsync:0                           #后台I/O队列里面，等待执行的fsync调用数量
aof_delayed_fsync:0                               #被延迟的fsync调用数量

如果正在进行加载操作，则会添加以下附加字段：
loading_start_time                                #加载操作开始的基于纪元的时间戳
loading_total_bytes                               #文件总大小
loading_loaded_bytes                              #已加载的字节数
loading_loaded_perc                               #加载进度表示为百分比
loading_eta_seconds                               #ETA在几秒钟内完成负载


# Stats
total_connections_received:7212                   #服务接受的总连接数
total_commands_processed:2341631                  #服务器处理的总命令数
instantaneous_ops_per_sec:0                       #每秒处理的命令数
total_net_input_bytes:125344667                   #从网络读取的总字节数
total_net_output_bytes:1712517025                 #写入网络的总字节数
instantaneous_input_kbps:0.00                     #网络读取速率KB/sec
instantaneous_output_kbps:0.00                    #网络写入速率KB/sec 
rejected_connections:0                            #因达到最大连接数而拒绝的连接
sync_full:1                                       #给从节点完全同步的数量
sync_partial_ok:0                                 #接受的同步请求数量
sync_partial_err:0                                #拒绝的同步请求数量
expired_keys:0                                    #键到期的总数
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0                           #因达到maxmemory限制而被驱逐的键的数量
keyspace_hits:641037            #Number of successful lookup of keys in the main dictionary
keyspace_misses:9002            #Number of failed lookup of keys in the main dictionary
pubsub_channels:1
pubsub_patterns:0
latest_fork_usec:326                     #最新fork操作的持续时间（以微秒为单位）
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master                              #master or slave
connected_slaves:0                       #已建立连接的从节点数
master_replid:fe1bb6f5cfef91b36603d8e57081cc02890705c8    #复制ID
master_replid2:0000000000000000000000000000000000000000   #第二个复制ID，用于failover的PSYNC
master_repl_offset:86270959              #服务当前的复制偏移量
second_repl_offset:-1           #The offset up to which replication IDs are accepted
repl_backlog_active:0
repl_backlog_size:1048576                #复制积压缓冲区的总大小（B）
repl_backlog_first_byte_offset:85222384  #复制积压缓冲区的主偏移量
repl_backlog_histlen:1048576             #复制积压缓冲区中数据的大小

# CPU
used_cpu_sys:24.941282                    #Redis服务消耗的系统cpu
used_cpu_user:18.820039                   #Redis服务消耗的用户cpu
used_cpu_sys_children:0.050757            #后台进程占用的系统cpu
used_cpu_user_children:0.259980           #后台进程占用的用户cpu

# Cluster                                 #集群信息
cluster_enabled:0

# Keyspace                                #数据库的统计信息
db0:keys=85766,expires=0,avg_ttl=0
```

