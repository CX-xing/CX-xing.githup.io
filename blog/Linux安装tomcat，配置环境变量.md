# Linux安装tomcat，配置环境变量

##### 检查linux是否安装tomcat

rpm -qa|grep tomcat

如果有通过

```
rpm -qa|grep tomcat
```

```
rpm -e rpm -qa|grep tomcat 
```

注意：一般tomcat安装都是通过压缩包的方式，所以这一步可以跳过

##### 下载Tomcat

```
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-9/v9.0.8/bin/apache-tomcat-9.0.8.tar.gz
```

##### 解压tomcat包

```
tar -zxvf apache-tomcat-9.0.8.tar.gz 
```

