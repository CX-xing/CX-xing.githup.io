# RabbitMQ安装

##### 安装依赖文件：

```shell
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget
```

##### 安装erlang 语言环境：

```
wget http://www.erlang.org/download/otp_src_18.3.tar.gz  //下载erlang包
tar -xzvf otp_src_18.3.tar.gz  //解压
cd otp_src_18.3/ //切换到安装路径
./configure --prefix=/usr/local/erlang  //生产安装配置
make && make install  //编译安装
```

##### 配置erlang环境变量：

```
vi /etc/profile  //在底部添加以下内容
	#set erlang environment
	ERL_HOME=/usr/local/erlang
	PATH=$ERL_HOME/bin:$PATH
	export ERL_HOME PATH
	
source /etc/profile  //生效
```

##### 测试一下是否安装成功,在控制台输入命令erl

```shell
erl  //如果进入erlang的shell则证明安装成功，退出即可。
```

##### 下载安装RabbitMQ：

```
cd /usr/local  //切换到计划安装RabbitMQ的目录，我这里放在/usr/local
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-generic-unix-3.6.1.tar.xz  //下载RabbitMQ安装包
```

##### 对于下载xz包进行解压，首先先下载xz压缩工具：

```
yum install xz
```

##### 对rabbitmq包进行解压：

```
xz -d rabbitmq-server-generic-unix-3.6.1.tar.xz
tar -xvf rabbitmq-server-generic-unix-3.6.1.tar
```

##### 解压后多了个文件夹rabbitmq-server-3.6.1 ，重命名为rabbitmq以便记忆。

```
mv rabbitmq_server-3.6.1/ rabbitmq
```

##### 配置rabbitmq环境变量：

```
vi /etc/profile 
#set rabbitmq environment 
export PATH=$PATH:/usr/local/rabbitmq/sbin 

source /etc/profile //生效
```

##### 启动服务：

```
rabbitmq-server -detached //启动rabbitmq，-detached代表后台守护进程方式启动。
```

##### 查看状态，如果显示如下截图说明安装成功：

```
rabbitmqctl status
```

```
启动服务：rabbitmq-server -detached【 /usr/local/rabbitmq/sbin/rabbitmq-server  -detached 】
查看状态：rabbitmqctl status【 /usr/local/rabbitmq/sbin/rabbitmqctl status  】
关闭服务：rabbitmqctl stop【 /usr/local/rabbitmq/sbin/rabbitmqctl stop  】
列出角色：rabbitmqctl list_users
```

##### 配置网页插件：

##### 首先创建目录，否则可能报错：

```
mkdir /etc/rabbitmq
```

##### 然后启用插件：

```
rabbitmq-plugins enable rabbitmq_management
```

##### 配置防火墙：

##### 配置linux 端口 15672 网页管理 5672 AMQP端口：

```
firewall-cmd --permanent --add-port=15672/tcp
firewall-cmd --permanent --add-port=5672/tcp
systemctl restart firewalld.service
```

##### 配置访问账号密码和权限：

##### 默认网页是不允许访问的，需要增加一个用户修改一下权限，代码如下：

```
rabbitmqctl add_user cxing cxing  //添加用户，后面两个参数分别是用户名和密码
rabbitmqctl set_permissions -p / cxing ".*" ".*" ".*"  //添加权限
rabbitmqctl set_user_tags cxing administrator  //修改用户角色
```

##### 然后就可以远程访问了，然后可直接配置用户权限等信息。

##### 登录：http://ip:15672 登录之后在admin里面把guest删除。







```
关于RabbitMQ的一些基本操作
$ sudo chkconfig rabbitmq-server on  # 添加开机启动RabbitMQ服务
$ sudo  rabbitmq-server start # 启动服务
$ sudo  rabbitmq-server status  # 查看服务状态
$ sudo  rabbitmq-server stop   # 停止服务

# 查看当前所有用户
$ sudo rabbitmqctl list_users

# 查看默认guest用户的权限
$ sudo rabbitmqctl list_user_permissions guest

# 由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户
$ sudo rabbitmqctl delete_user guest

# 添加新用户
$ sudo rabbitmqctl add_user username password
rabbitmqctl add_user cxing cxing

# 设置用户tag
$ sudo rabbitmqctl set_user_tags username administrator

# 赋予用户默认vhost的全部操作权限
$ sudo rabbitmqctl set_permissions -p / username ".*" ".*" ".*"

# 查看用户的权限
$ sudo rabbitmqctl list_user_permissions username

# 查看队列
$ sudo rabbitmqctl list_queues
或
$ sudo rabbitmqctl list_queues -p ${vhostpath}

# 清除队列命令
sudo rabbitmqctl purge_queue ${queue_name}
或
rabbitmqctl purge_queue ${queue_name}
或
rabbitmqctl -p ${vhostpath} purge_queue ${queue_name}

sudo rabbitmqctl stop_app;
sudo rabbitmqctl reset;
sudo rabbitmqctl start_app;
```

