***influxDB 2.0 安装***

```bash
wget https://dl.influxdata.com/influxdb/releases/influxdb_2.0.0-beta.2_linux_amd64.tar.gz

tar xvzf path/to/influxdb_2.0.0-beta.2_linux_amd64.tar.gz

sudo cp influxdb_2.0.0-beta.8_linux_amd64/{influx,influxd} /usr/local/bin/

influxd
```

***influxDB 1.*  生产安装    yum**   

```bash
## CentOS用户可以使用yum程序包管理器安装InfluxDB的最新稳定版本：
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF

## 将存储库添加到yum配置后，通过运行以下命令安装并启动InfluxDB服务：
sudo yum install influxdb
## sudo systemctl start influxdb   启动/重启
service influxdb start  
service influxdb restart

## 使用以下-config 选项将过程指向正确的配置文件：
influxd -config /etc/influxdb/influxdb.conf

## 将环境变量设置为INFLUXDB_CONFIG_PATH配置文件的路径，然后开始该过程。
echo $INFLUXDB_CONFIG_PATH
/etc/influxdb/influxdb.conf
influxd
```

***influxDB 1.*  生产安装    rpm**   

```shell
## 下载
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.5.3.x86_64.rpm
## 解压
sudo yum localinstall influxdb-1.5.3.x86_64.rpm
## 启动
sudo systemctl start influxdb
// 启动成功的提示
Redirecting to /bin/systemctl start influxdb.service
## 关闭
sudo systemctl stop influxdb


## 先检查安装的influxdb
rpm -qa|grep influx
## 打印结果
influxdb-1.3.3-1.x86_64 
## 卸载influxdb
rpm -e influxdb-1.3.3.x86_64
## 打印结果
warning: /etc/influxdb/influxdb.conf saved as /etc/influxdb/influxdb.conf.rpmsave
Removed symlink /etc/systemd/system/multi-user.target.wants/influxdb.service.
Removed symlink /etc/systemd/system/influxd.service.
```

### docker 安装

```bash
## 下载：如果不指定版本号，会默认拉取最新的
docker pull influxdb:1.3.3
## 查看本地镜像,已经存在刚刚拉取的influxdb:1.3.3
docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
docker.io/influxdb   1.3.3               fa78fa0c3591        2 years ago         227 MB
##运行,把主机的8086端口映射到容器的8086端口，把主机的/root/metric映射到容器的/var/lib/influxdb目录，给服务起名为influxdb
docker run -p 8086:8086 -v /root/metric:/var/lib/influxdb -d --name influxdb fa78fa0c3591
```

### 用户管理

```shell
#显示用户
show users 
#创建用户
create user "username" with password 'password'
#创建管理员权限用户
create user "username" with password 'password' with all privileges
#删除用户
drop user "username"
```

### influxdb客户端

```shell
#####  influxDB名词
##  database：数据库；
##  measurement：数据库中的表；
##  points：表里面的一行数据。
#####  influxDB中独有的一些概念
##  Point由时间戳（time）、数据（field）和标签（tags）组成。
##  time：每条数据记录的时间，也是数据库自动生成的主索引；
##  fields：各种记录的值；
##  tags：各种有索引的属性。

## 启动
influx
## 退出
exit
## 查看转态
show status
## 查询数据库
show databases
## 创建一个数据库
create database "db_name"
## 删除数据库
drop database "db_name"
## 使用数据库
use db_name
## 查看该数据库下所有表
show measurements


## 创建表 注：直接在插入数据的时候指定表名,表自动创建，字段类型由传入的值决定
insert test,host=127.0.0.1,monitor_name=test count=1
curl -i -XPOST 'http://127.0.0.1:8086/write?db=metrics' --data-binary 'test,host=127.0.0.1,monitor_name=test count=1'

## 删除表
drop measurement measurement_name
## 查询表
select * from database limit 10
select * from test order by time desc

## 格式：数据库地址 + 端口 + query?db = 数据库名&q = 查询或删除或插入的SQL语句
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=metrics" --data-urlencode "q=select * from test order by time desc"

```

### 数据保存策略（Retention Policies）

```bash
## 查看当前数据库Retention Policies
show retention policies on "db_name"
## 创建新的Retention Policies
#  rp_name：策略名
#  db_name：具体的数据库名
#  3w：保存3周，3周之前的数据将被删除，influxdb具有各种事件参数，比如：h（小时），d（天），w（星期）
#  replication 1：副本个数，一般为1就可以了
#  default：设置为默认策略
create retention policy "rp_name" on "db_name" duration 3w replication 1 default

## 修改Retention Policies
alter retention policy "rp_name" on "db_name" duration 30d default

## 删除Retention Policies
drop retention policy "rp_name"
```

### 查询

```bash
## 查询当前数据库里的所有tag keys
show tag keys
## 查询当前数据库的所有field keys
show field keys
```

### influxDB 相关文件路径

```bash
## /usr/bin下
influxd    influxdb服务器
influx      influxdb命令行客户端
influx_inspect  查看工具
influx_stress  压力测试工具
influx_tsm  数据库转换工具（将数据库从b1或bz1格式转换为tsm1格式）

## /var/lib/influxdb/下
data  存放最终存储的数据，文件以.tsm结尾
meta  存放数据库元数据
wal  存放预写日志文件

## /var/log/influxdb下
influxd.log  日志文件

## /etc/influxdb下
influxdb.conf  配置文件

## /var/run/influxdb/
influxd.pid  PID文件
```

influxDB配置文件

```bash
全局配置
reporting-disabled = false  # 该选项用于上报influxdb的使用信息给InfluxData公司，默认值为false
bind-address = ":8088"  # 备份恢复时使用，默认值为8088

1、meta相关配置
[meta]
dir = "/var/lib/influxdb/meta"  # meta数据存放目录
retention-autocreate = true  # 用于控制默认存储策略，数据库创建时，会自动生成autogen的存储策略，默认值：true
logging-enabled = true  # 是否开启meta日志，默认值：true

2、data相关配置
[data]
dir = "/var/lib/influxdb/data"  # 最终数据（TSM文件）存储目录
wal-dir = "/var/lib/influxdb/wal"  # 预写日志存储目录
query-log-enabled = true  # 是否开启tsm引擎查询日志，默认值： true
cache-max-memory-size = 1048576000  # 用于限定shard最大值，大于该值时会拒绝写入，默认值：1000MB，单位：byte
cache-snapshot-memory-size = 26214400  # 用于设置快照大小，大于该值时数据会刷新到tsm文件，默认值：25MB，单位：byte
cache-snapshot-write-cold-duration = "10m"  # tsm引擎 snapshot写盘延迟，默认值：10Minute
compact-full-write-cold-duration = "4h"  # tsm文件在压缩前可以存储的最大时间，默认值：4Hour
max-series-per-database = 1000000  # 限制数据库的级数，该值为0时取消限制，默认值：1000000
max-values-per-tag = 100000  # 一个tag最大的value数，0取消限制，默认值：100000

3、coordinator查询管理的配置选项
[coordinator]
write-timeout = "10s"  # 写操作超时时间，默认值： 10s
max-concurrent-queries = 0  # 最大并发查询数，0无限制，默认值： 0
query-timeout = "0s  # 查询操作超时时间，0无限制，默认值：0s
log-queries-after = "0s"  # 慢查询超时时间，0无限制，默认值：0s
max-select-point = 0  # SELECT语句可以处理的最大点数（points），0无限制，默认值：0
max-select-series = 0  # SELECT语句可以处理的最大级数（series），0无限制，默认值：0
max-select-buckets = 0  # SELECT语句可以处理的最大"GROUP BY time()"的时间周期，0无限制，默认值：0

4、retention旧数据的保留策略
[retention]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "30m"  # 检查时间间隔，默认值 ："30m"

5、shard-precreation分区预创建
[shard-precreation]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "10m"  # 检查时间间隔，默认值 ："10m"
advance-period = "30m"  # 预创建分区的最大提前时间，默认值 ："30m"

6、monitor 控制InfluxDB自有的监控系统。 默认情况下，InfluxDB把这些数据写入_internal 数据库，如果这个库不存在则自动创建。 _internal 库默认的retention策略是7天，如果你想使用一个自己的retention策略，需要自己创建。
[monitor]
store-enabled = true  # 是否启用该模块，默认值 ：true
store-database = "_internal"  # 默认数据库："_internal"
store-interval = "10s  # 统计间隔，默认值："10s"

7、admin web管理页面
[admin]
enabled = true  # 是否启用该模块，默认值 ： false
bind-address = ":8083"  # 绑定地址，默认值 ：":8083"
https-enabled = false  # 是否开启https ，默认值 ：false
https-certificate = "/etc/ssl/influxdb.pem"  # https证书路径，默认值："/etc/ssl/influxdb.pem"

8、http API
[http]
enabled = true  # 是否启用该模块，默认值 ：true
bind-address = ":8086"  # 绑定地址，默认值：":8086"
auth-enabled = false  # 是否开启认证，默认值：false
realm = "InfluxDB"  # 配置JWT realm，默认值: "InfluxDB"
log-enabled = true  # 是否开启日志，默认值：true
write-tracing = false  # 是否开启写操作日志，如果置成true，每一次写操作都会打日志，默认值：false
pprof-enabled = true  # 是否开启pprof，默认值：true
https-enabled = false  # 是否开启https，默认值：false
https-certificate = "/etc/ssl/influxdb.pem"  # 设置https证书路径，默认值："/etc/ssl/influxdb.pem"
https-private-key = ""  # 设置https私钥，无默认值
shared-secret = ""  # 用于JWT签名的共享密钥，无默认值
max-row-limit = 0  # 配置查询返回最大行数，0无限制，默认值：0
max-connection-limit = 0  # 配置最大连接数，0无限制，默认值：0
unix-socket-enabled = false  # 是否使用unix-socket，默认值：false
bind-socket = "/var/run/influxdb.sock"  # unix-socket路径，默认值："/var/run/influxdb.sock"

9、subscriber 控制Kapacitor接受数据的配置
[subscriber]
enabled = true  # 是否启用该模块，默认值 ：true
http-timeout = "30s"  # http超时时间，默认值："30s"
insecure-skip-verify = false  # 是否允许不安全的证书
ca-certs = ""  # 设置CA证书
write-concurrency = 40  # 设置并发数目，默认值：40
write-buffer-size = 1000  # 设置buffer大小，默认值：1000

10、graphite 相关配置
[[graphite]]
enabled = false  # 是否启用该模块，默认值 ：false
database = "graphite"  # 数据库名称，默认值："graphite"
retention-policy = ""  # 存储策略，无默认值
bind-address = ":2003"  # 绑定地址，默认值：":2003"
protocol = "tcp"  # 协议，默认值："tcp"
consistency-level = "one"  # 一致性级别，默认值："one
batch-size = 5000  # 批量size，默认值：5000
batch-pending = 10  # 配置在内存中等待的batch数，默认值：10
batch-timeout = "1s"  # 超时时间，默认值："1s"
udp-read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0
separator = "."  # 多个measurement间的连接符，默认值： "."

11、collectd
[[collectd]]
enabled = false  # 是否启用该模块，默认值 ：false
bind-address = ":25826"  # 绑定地址，默认值： ":25826"
database = "collectd"  # 数据库名称，默认值："collectd"
retention-policy = ""  # 存储策略，无默认值
typesdb = "/usr/local/share/collectd"  # 路径，默认值："/usr/share/collectd/types.db"
auth-file = "/etc/collectd/auth_file"
batch-size = 5000
batch-pending = 10
batch-timeout = "10s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。默认值：0

12、opentsdb
[[opentsdb]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":4242"  # 绑定地址，默认值：":4242"
database = "opentsdb"  # 默认数据库："opentsdb"
retention-policy = ""  # 存储策略，无默认值
consistency-level = "one"  # 一致性级别，默认值："one"
tls-enabled = false  # 是否开启tls，默认值：false
certificate= "/etc/ssl/influxdb.pem"  # 证书路径，默认值："/etc/ssl/influxdb.pem"
log-point-errors = true  # 出错时是否记录日志，默认值：true
batch-size = 1000
batch-pending = 5
batch-timeout = "1s"

13、udp
[[udp]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":8089"  # 绑定地址，默认值：":8089"
database = "udp"  # 数据库名称，默认值："udp"
retention-policy = ""  # 存储策略，无默认值
batch-size = 5000
batch-pending = 10
batch-timeout = "1s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0　

14、continuous_queries
[continuous_queries]
enabled = true  # enabled 是否开启CQs，默认值：true
log-enabled = true  # 是否开启日志，默认值：true
run-interval = "1s"  # 时间间隔，默认值："1s"
```

