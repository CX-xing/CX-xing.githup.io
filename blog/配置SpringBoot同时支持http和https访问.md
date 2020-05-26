# 配置SpringBoot同时支持http和https访问

##### 1 生成证书

如果配置了JAVA开发环境，可以使用keytool命令生成证书。我们打开控制台，输入：

```bash
keytool -genkey -alias jrunion -keyalg RSA -keysize 2048 -keystore D:\jrunion.keystore -validity 365
```

```bash
keytool -genkey -alias jrunion -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN" -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore jrunion.keystore -validity 365
```

命令含义如下：

- genkey 表示要创建一个新的密钥。

- alias 表示 keystore 的别名。

- keyalg 表示使用的加密算法是 RSA ，一种非对称加密算法。

- keysize 表示密钥的长度。

- keystore 表示生成的密钥存放位置。

- validity 表示密钥的有效时间，单位为天。

  执行过程

  ```csharp
  输入密钥库口令:123456
  再次输入新口令:123456
  您的名字与姓氏是什么?
    [Unknown]:  jrunion
  您的组织单位名称是什么?
    [Unknown]:  jrunion
  您的组织名称是什么?
    [Unknown]:  jrunion
  您所在的城市或区域名称是什么?
    [Unknown]:  beijing
  您所在的省/市/自治区名称是什么?
    [Unknown]:  beijingshi
  该单位的双字母国家/地区代码是什么?
    [Unknown]:  china
  CN=kaibowang, OU=yuxuelian, O=yuxuelian, L=chengdu, ST=chengdushi, C=china是否正确?
    [否]:  y
  
  输入 <tomcat> 的密钥口令
          (如果和密钥库口令相同, 按回车):
  再次输入新口令:
  
  Warning:
  JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore C:\Users\Administrator\.keystore -destkeystore C:\Users\Administrator\.keystore -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
  ```

命令执行完成后 ，我们在 D 盘目录下会看到一个名为 jrunion.keystor 的文件

##### 2 向已存在密钥库添加新密钥

```bash
keytool -genkeypair -alias jrunion_ip -keyalg RSA -validity 36500 -keystore D:\jrunion_ip.keystore -v
```

-alias bendiceshi_ip：新密钥的名字
-keystore D:\jrunion_ip.keystore：刚才生成的密钥库文件的位置
其余项同1

**3 查看密钥库中的项**

```bash
##添加-v会显示详细信息，这里为了篇幅考虑不加-v。
keytool -list -keystore D:\jrunion.keystore
keytool -list -keystore D:\jrunion.keystore -v
```

**4 导出证书**

```bash
keytool -exportcert -alias jrunion -file D:\jrunion.cer -storepass 123456 -keystore D:\jrunion.keystore -v
keytool -exportcert -alias jrunion_ip -file D:\jrunion_ip.cer -storepass 123456 -keystore D:\jrunion_ip.keystore -v
```

-alias bendiceshi：要导出的证书的名字，即刚才创建的密钥的名字，即keytool -list时显示的名字
-file ./bendiceshi.cer：要导出的证书的存储位置，这里我放在当前目录下
-keystore ./test.keystore：刚才创建的密钥库的位置

##### 5 安装证书

安装证书-下一步-将所有证书放入下列存储-浏览-受信任的根证书颁发机构-确定-

# springboot引入

接下来我们需要在项目中引入 https。

将上面生成的 jrunion.keystor 拷贝到 Spring Boot 项目的 resources 目录下。然后在 application.properties 中添加如下配置：

```
server.ssl.key-store=classpath:jrunion.keystor
server.ssl.key-alias=jrunion
server.ssl.key-store-password=123456
```

其中：

- key-store表示密钥文件名。
- key-alias表示密钥别名。
- key-store-password就是在cmd命令执行过程中输入的密码。

配置完成后，就可以启动 Spring Boot 项目了，此时如果我们直接使用 Http 协议来访问接口，就会看到如下错误：

```
Bad Request
This combination of host and port requires TLS.
```

改用 https 来访问 ，结果如下：需要浏览器点击高级继续访问（实际项目中只需要安装一个被浏览器认可的 https 证书即可）

##### 请求转发

考虑到 Spring Boot 不支持同时启动 HTTP 和 HTTPS ，为了解决这个问题，我们这里可以配置一个请求转发，当用户发起 HTTP 调用时，自动转发到 HTTPS 上。

具体配置如下：

```java
package com.jrunion.netscoutpoc.config;

import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TomcatConfig {

    //项目启动
    @Value("${server.port}")
    private int httpPort;
    //http访问
    @Value("${server.jrunion.http.port}")
    private int jrunionHttpPort;

    @Bean
    TomcatServletWebServerFactory tomcatServletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint constraint = new SecurityConstraint();
                constraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                constraint.addCollection(collection);
                context.addConstraint(constraint);
            }
        };
        factory.addAdditionalTomcatConnectors(createTomcatConnector());
        return factory;
    }

    private Connector createTomcatConnector() {
        Connector connector = new
                Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(jrunionHttpPort);
        connector.setSecure(false);
        connector.setRedirectPort(httpPort);
        return connector;
    }
}
```

在这里，我们配置了 Http 的请求端口为 8081，所有来自 8081 的请求，将被自动重定向到 8080 这个 https 的端口上。

如此之后，我们再去访问 http 请求，就会自动重定向到 https。



# tomcat引入

**copy密钥库文件**
将刚才生成的密钥库文件copy到你tomcat的conf目录，跟server.xml同级。**注意：这里一定要放在你实际运行的tomcat的conf目录下。**

修改tomcat配置文件

    <!--老版本tomcat的配置，在tomcat8下测试成功-->
    <Connector connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="443"/>
    <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" keystoreFile="/conf/jrunion.keystore"
               keystorePass="123456"/>
**注意：以上配置是老版本tomcat的配置，从tomcat8.5开始tomcat更改了配置形式，如上配置估计在tomcat10的时候会完全废弃，在tomcat9.0下测试成功的配置如下：**

```
<!--新版本tomcat的配置，在tomcat9下测试成功-->
 <Connector connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="443" />
     <Connector port="443" protocol="org.apache.coyote.http11.Http11Nio2Protocol"                        maxThreads="150" SSLEnabled="true" scheme="https" secure="true">
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="conf/jrunion.keystore"                                              certificateKeystorePassword="123456" />
        </SSLHostConfig>
     </Connector>

```

要修改的配置文件是tomcat的server.xml文件。仔细看上面的配置，第一个Connector是默认就有的，这里只是把8080端口改成了公认的80端口，把8443端口改成了公认的443端口。第二个Connector默认是注释掉的，搜索8443就能找到，直接把上述第二个Connector粘贴到server.xml中第一个Connector的下面，方便管理。keystoreFile=”/conf/jrunion.keystore”就是刚才copy的文件的位置，可以自己改到其他位置。keystorePass=”123456”就是刚才创建密钥库时使用的口令。