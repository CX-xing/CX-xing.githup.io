### LINUX常用命令总结

### iptables防火墙

```shell
# 查看防火墙状态
service iptables status  
# 停止防火墙
service iptables stop 
# 启动防火墙
service iptables start  
# 重启防火墙
service iptables restart  
# 永久关闭防火墙
chkconfig iptables off  
# 永久关闭后重启
chkconfig iptables on
```

### 开放端口检查

```shell
netstat -anlp | grep 3306
lsof -i:3306
telnet 192.168.25.133 3306 ##检查端口开放，防火墙未开启
```

### 查看系统状态

```shell
vmstat 

##查看系统资源使用
top

##参数详解
r：几个进程在占用cpu        　　 b：等待IO值
Swpd：多少交换内存              free：剩余内存（k）
Buff：数据缓冲区                cache：数据缓存区
Si：从内存进入内存交换区      　　so：从内存交换分区到内存
Bi：设备读入数据量           　　bo：设备写入数据量
Us：用户cpu使用率        　　　　id：cp空闲

# 查看系统相关信息
命令：uname -a

# 查看系统发行版本信息
命令：cat /etc/issue

# 查看所有服务　　
systemctl list-units --all --type=service

# 查看磁盘空间
df -h

#文件赋予权限
chmod 777 文件名
```

```shell
## githup初始化
echo "# mybatis-test" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/cx-xing/google-translater.git
git push -u origin master


## 查看远程目录
git remote -v
## 修改git地址远程仓库
git remote set-url origin https://github.com/cx-xing/google-translater.git
```

```shell
## 已安装的软件包
yum list installed
## 查找可以安装的软件包
yum list tomcat
## 卸载安装的软件包
yum remove tomcat
## 列出软件包的依赖
yum deplist tomcat
## info 显示软件包的描述信息和概要信息
yum info tomcat
## 升级所有软件包/升级指定软件包/检查更新软件包
yum update 
yum update tomcat
yum check-update
```

```shell
## 先检查安装的influxdb
rpm -qa|grep influx
## 打印结果
influxdb-1.3.3-1.x86_64 
## 卸载influxdb
rpm -e influxdb-1.3.3.x86_64
rm -rf /usr/local/redis/bin/redis*
```

