[toc]

## 1. 基础框架搭建

### 1.1 架构预览

我们要搭建的架构如下图所示：
![12f0f9dad1c2cb80014a812cd8691c57.png](微服务搭建.assets/Image [1]-1621167955951.png)
上面的架构中，主要包含以下节点：

* gateway:微服务网关，统一处理外部请求，是客户端和微服务之间的桥梁;
* auth:微服务认证服务，用于令牌(token)生成和校验；
* register:微服务注册中心，用于统一控制各个微服务的注册与发现；
* system:提供rbac服务;
* system-test:微服务调用测试;

### 1.2 Maven父模块搭建

首先我们使用IDEA创建一个名称为m-cloud的 Maven模块，该模块为整个工程的服务模块，用于聚合各个微服务子系统。在D盘根目录创建一个名称为cloud的文件夹，然后打开IDEA，点击Create New Project新建一个Maven项目，Project SDK选择JDK 1.8：
![bbf74821314c2f819c94bd3191856054.png](微服务搭建.assets/Image [2]-1621167958688.png)
![edfd89de66bc03e3b283ad9d99f04c0c.png](微服务搭建.assets/Image [3]-1621167960722.png)
为了加速maven的构建，我们设置idea的maven配置:
![1ee2afb4587bfdd276ef38c7243a8dac.png](微服务搭建.assets/Image [4]-1621167962916.png)
其中maven的配置文件settings.xml：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository/>
  <interactiveMode/>
  <usePluginRegistry/>
  <offline/>
  <pluginGroups/>
  <mirrors>
    <mirror>
     <id>aliyunmaven</id>
     <mirrorOf>central</mirrorOf>
     <name>阿里云公共仓库</name>
     <url>https://maven.aliyun.com/repository/central</url>
    </mirror>
    <mirror>
      <id>repo1</id>
      <mirrorOf>central</mirrorOf>
      <name>central repo</name>
      <url>http://repo1.maven.org/maven2/</url>
    </mirror>
    <mirror>
     <id>aliyunmaven</id>
     <mirrorOf>apache snapshots</mirrorOf>
     <name>阿里云阿帕奇仓库</name>
     <url>https://maven.aliyun.com/repository/apache-snapshots</url>
    </mirror>
  </mirrors>
  <proxies/>
  <activeProfiles/>
  <profiles>
    <profile>  
        <repositories>
           <repository>
                <id>aliyunmaven</id>
                <name>aliyunmaven</name>
                <url>https://maven.aliyun.com/repository/public</url>
                <layout>default</layout>
                <releases>
                        <enabled>true</enabled>
                </releases>
                <snapshots>
                        <enabled>true</enabled>
                </snapshots>
            </repository>
            <repository>
                <id>MavenCentral</id>
                <url>http://repo1.maven.org/maven2/</url>
            </repository>
            <repository>
                <id>aliyunmavenApache</id>
                <url>https://maven.aliyun.com/repository/apache-snapshots</url>
            </repository>
        </repositories>             
     </profile>
  </profiles>
</settings>

```

因为m-cloud模块是项目的父模块，仅用于聚合子模块，所以我们可以把src目录下的内容全部删了，保留pom.xml和m-cloud.iml，然后修改pom.xml，引入Spring Boot和Spring Cloud：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.work</groupId>
    <artifactId>m-cloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <description>微服务父项目</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

上面的pom配置中，我们指定了packaging为pom，表示这是一个纯聚合模块。

### 1.3 通用模块搭建

通用模块主要用于定义一些各个微服务通用的实体类，工具类或者第三方依赖等。点击File -> New -> Module...，新建一个Maven模块，Module SDK选择JDK 1.8：
![3605cc991f9b5963d0ff9c03a96278bb.png](微服务搭建.assets/Image [5]-1621167968077.png)
这个时候我们查看m-cloud的pom，会发现新增如下内容:

```xml
 <modules>
        <module>../m-common</module>
    </modules>
```

我们在m-common的pom里增加如下依赖:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>m-cloud</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../m-cloud/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>m-common</artifactId>
    <description>通用模块</description>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.51</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>
    </dependencies>

</project>
```

这样通用模块搭建完了，接下来我们搭建注册中心。

### 1.4 注册中心搭建

微服务注册中心的作用就是用于统一管理微服务实例，微服务间的调用只需要知道对方的服务名，而无需关注具体的IP和端口，便于微服务架构的拓展和维护。
在这一节中，我们先使用Eureka构建微服务注册中心（Eureka服务端），因为Eureka较为简单，无须启动第三方服务，只需要引入相关依赖即可。后续会介绍如何使用Nacos构建微服务注册中心。
在IDEA的菜单栏中点击File -> New -> Modules...，新建一个Spring Boot项目，模板选择Spring Initializr，Module SDK选择JDK 1.8：
![448b478f81b6d78ae5af09bc62705472.png](微服务搭建.assets/Image [8].png)
为了加快构建，脚手架地址可以填写：[https://start.aliyun.com/](https://start.aliyun.com/)
点击next,按如下填写:
![fa118fc5f0160f66a9e941f06b46e722.png](微服务搭建.assets/Image [9].png)
继续点击Next，在依赖列表中，搜索Eureka Server，然后添加进去：
![faff9d5a4c3fe1ceca634debb5d00aa7.png](微服务搭建.assets/Image [10].png)
点击Next:
![81846cecff7dbbbb6358c185650d74be.png](微服务搭建.assets/Image [11].png)
依然要注意路径。
![bba08e11a9774f8a46acc9b83575c8fa.png](微服务搭建.assets/Image [12].png)
红框内文件可以删除。
调整m-register的pom.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.work</groupId>
    <artifactId>m-register</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>m-register</name>
    <description>注册中心</description>

    <parent>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <artifactId>m-cloud</artifactId>
        <relativePath>../m-cloud/pom.xml</relativePath>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

接着修改m-cloud的pom，增加子模块:

```xml
 <modules>
        <module>../m-common</module>
        <module>../m-register</module>
    </modules>
```

打开m-register的入口类MRegisterApplication，在类上使用@EnableEurekaServer标注，用以开启Eureka服务端功能：

```java
@SpringBootApplication
@EnableEurekaServer
public class MRegisterApplication {

    public static void main(String[] args) {
        SpringApplication.run(MRegisterApplication.class, args);
    }

}
```

另外注意删除test下的内容。

然后开始编写项目配置文件，由于我个人比较习惯yml格式的配置，所以我将resources目录下的application.properties重命名为application.yml（后面章节搭建的boot项目，配置文件都采用yml格式，请知悉），在该配置文件中编写如下内容：

```yml
server:
  port: 8001
  servlet:
    context-path: /register

spring:
  application:
    name: m-register

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    instance-info-replication-interval-seconds: 30
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}${server.servlet.context-path}/eureka/
```

项目的端口号为8001（在上一节中我们已经约定过），context-path为/register，剩下的配置含义如下：

* spring.application.name，定义服务名称为m-register;

* eureka.instance.hostname，指定了Eureka服务端的地址，因为我们是在本地搭建的，所以填写为localhost即可；

* eureka.client.register-with-eureka，表示是否将服务注册到Eureka服务端，由于我们这里是单节点的Eureka服务端，所以这里指定false；

* eureka.client.fetch-registry，表示是否从Eureka服务端获取服务信息，因为这里是单节点的Eureka服务端，并不需要从别的Eureka服务端同步服务信息，所以这里设置为false；

* eureka.client.instance-info-replication-interval-seconds，微服务更新实例信息的变化到Eureka服务端的间隔时间，单位为秒，这里指定为30秒（这就是微服务启动后，要过一会才能注册到Eureka服务端的原因）。

* eureka.client.serviceUrl.defaultZone，指定Eureka服务端的地址，这里为当前项目地址，即 http://localhost:8001/register/eureka/


至此，一个简单的微服务注册中心搭建好了，我们运行入口类MRegisterApplication的main方法启动项目，启动后访问[http://localhost:8001/register/](http://localhost:8001/register/)，出现Eureka页面说明微服务注册中心搭建成功，目前还没有微服务实例注册进来，所以列表是空的。

目前Eureka服务端是“裸奔着”的，只要知道了Eureka服务端的地址后便可以将微服务注册进来，我们可以引入spring-cloud-starter-security来保护Eureka服务端。在m-register模块的pom文件中添加如下依赖：

```xml
<dependency> 
  <groupId>org.springframework.cloud</groupId>  
  <artifactId>spring-cloud-starter-security</artifactId> 
</dependency>
```

接下来我们在com.work.m.register.configure包下创建自己的资源保护配置类MRegisterWebSecurityConfigure：

```java
@EnableWebSecurity
public class MRegisterWebSecurityConfigure extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

该配置类用于开启Eureka服务端端点保护。在application.yml中配置访问Eureka服务的受保护资源所需的用户名和密码：

```yml
spring:
  application:
    name: m-register
  security:
    user:
      name: m
      password: 123456
```

由于现在Eureka服务端是受保护的，需要正确的用户名和密码才能访问，所以上面我们配置的eureka.client.serviceUrl.defaultZone路径也要配置上账号密码，否则将抛出com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server异常。
将eureka.client.serviceUrl.defaultZone的配置改为如下所示:

```yml
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    instance-info-replication-interval-seconds: 30
    serviceUrl:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:${server.port}${server.servlet.context-path}/eureka/
```

重启项目，再次访问 [http://localhost:8001/register/](http://localhost:8001/register/):
![ab181825b9391439d065bd7e75a3859b.png](微服务搭建.assets/Image [13].png)

这次需要登录才能访问，输入我们在配置文件中定义的用户名和密码（m，123456）后便可成功访问注册列表页面。到这里微服务注册中心m-register已经搭建完毕，下一节开始着手搭建微服务认证中心m-auth。

### 1.5 认证中心搭建

我们一般希望搭建的各个微服务系统是受保护的，只有通过合法的认证信息才能访问相关资源，所以在这一节中，我们将借助Spring Cloud OAuth和Spring Cloud Security搭建一个统一给微服务发放访问令牌的认证服务m-auth。
**OAuth2协议简介**
在微服务架构下，我们通常根据不同的业务来构建不同的微服务子系统，各个子系统对外提供相应的服务。客户端除了浏览器外，还可能是手机App，小程序等。在微服务架构出现之前，我们的系统一般为单体模式，客户端只是单一的浏览器，所以通常情况下都是通过Session进行客户端，服务端通信，而随着客户端种类越来越多，这种交互方式变得越来越困难，于是OAuth协议应运而生。
**密码模式简介**
在密码模式中，用户向客户端提供用户名和密码，客户端通过用户名和密码到认证服务器获取令牌。流程如下所示：
![863a09dfcb0af864d273fb446578b8f6.png](images\Image [14].png)

- Resource Owner，资源所有者，即当前正在使用系统的用户；
- Client，客户端，比如浏览器，App等；
- Authorization server，认证服务器，提供认证服务，并发放访问令牌。

如上图所示，密码模式包含了三个步骤：

1. 用户向客户端提供用户名和密码；
2. 客户端向认证服务器换取令牌；
3. 认证服务器发放令牌。
   其中第2步客户端发出的HTTP请求，包含以下参数：

- grant_type：授权类型，此处的值固定为password，必选项。
- username：用户名，必选项。
- password：密码，必选项。
- scope：权限范围，可选项。
  **搭建认证服务**

在了解了OAuth协议和密码模式后，我们开始搭建认证服务器。点击IDEA菜单栏 File -> New -> Module...，因为认证服务器也是一个Spring Boot应用，所以模板选择Spring Initializr，Module SDK选择JDK 1.8：
![fdfff33d7d106b631d2f225ee753c802.png](微服务搭建.assets/Image [15].png)
![6796ec7210d14f895eb94c06b486e36c.png](微服务搭建.assets/Image [16].png)
![7a616a7c2ddc41505d31a67f1b5a1499.png](微服务搭建.assets/Image [17].png)
依然要注意路径，是在根目录下。
参考m-register删除不必要的部分，修改pom:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>m-cloud</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../m-cloud/pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.work</groupId>
    <artifactId>m-auth</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>m-auth</name>
    <description>认证服务</description>

    <dependencies>
        <dependency>
            <groupId>com.work</groupId>
            <artifactId>m-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

并在m-cloud中引入子模块:

```xml
  <modules>
        <module>../m-common</module>
        <module>../m-register</module>
        <module>../m-auth</module>
    </modules>
```

因为后续我们需要将认证服务器生成的Token存储到Redis中，并且Redis依赖可能会被多个微服务使用到，于是我们在m-common模块中引入redis相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

此外，我们需要将m-auth服务注册到上一节搭建的m-register中，所以我们在febs-common模块里继续添加Eureka相关依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

在m-auth的入口类MAuthApplication上添加@EnableDiscoveryClient注解，用于开启服务注册与发现功能：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MAuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(MAuthApplication.class, args);
    }

}
```

接着编写配置文件application.yml，内容如下所示：

```yml
server:
  port: 8101

spring:
  application:
    name: m-auth

eureka:
  instance:
    lease-renewal-interval-in-seconds: 20
  client:
    register-with-eureka: true
    fetch-registry: true
    instance-info-replication-interval-seconds: 30
    registry-fetch-interval-seconds: 3
    serviceUrl:
      defaultZone: http://m:123456@localhost:8001/register/eureka/
```

上述配置指定了应用端口号为8101，服务名称为m-auth（别的微服务可以通过这个名称从注册中心获取m-auth提供的服务），剩下的为Eureka相关配置，含义如下：

- eureka.instance.lease-renewal-interval-in-seconds，向Eureka 服务端发送心跳的间隔时间，单位为秒，用于服务续约。这里配置为20秒，即每隔20秒向febs-register发送心跳，表明当前服务没有宕机；
- eureka.client.register-with-eureka，为true时表示将当前服务注册到Eureak服务端；
- eureka.client.fetch-registry，为true时表示从Eureka 服务端获取注册的服务信息；
- eureka.client.instance-info-replication-interval-seconds，新实例信息的变化到Eureka服务端的间隔时间，单位为秒；
- eureka.client.registry-fetch-interval-seconds，默认值为30秒，即每30秒去Eureka服务端上获取服务并缓存，这里指定为3秒的原因是方便开发时测试，实际可以指定为默认值即可；
- eureka.client.serviceUrl.defaultZone，指定Eureka服务端地址。
  和Eureka相关的编码结束后，我们开始编写和安全相关的配置类。

首先我们需要在包com.work.m.auth.configure定义一个WebSecurity类型的安全配置类MSecurityConfigure，代码如下所示：

```java
@EnableWebSecurity
public class MSecurityConfigure extends WebSecurityConfigurerAdapter {

    @Autowired
    private MUserDetailService userDetailService;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
                .antMatchers("/oauth/**")
                .and()
                .authorizeRequests()
                .antMatchers("/oauth/**").authenticated()
                .and()
                .csrf().disable();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailService).passwordEncoder(passwordEncoder());
    }
}
```

该类继承了WebSecurityConfigurerAdapter适配器，重写了几个方法，并且使用@EnableWebSecurity注解标注，开启了和Web相关的安全配置。上面代码中，我们首先注入了MUserDetailService，这个类下面会介绍到，这里先略过；然后我们定义了一个PasswordEncoder类型的Bean，该类是一个接口，定义了几个和密码加密校验相关的方法，这里我们使用的是Spring Security内部实现好的BCryptPasswordEncoder。BCryptPasswordEncoder的特点就是，对于一个相同的密码，每次加密出来的加密串都不同。

在MSecurityConfigure类中，我们还重写了WebSecurityConfigurerAdapter类的configure(HttpSecurity http)方法，其中requestMatchers().antMatchers("/oauth/**")的含义是：MSecurityConfigure安全配置类只对/oauth/开头的请求有效。最后我们重写了configure(AuthenticationManagerBuilder auth)方法，指定了userDetailsService和passwordEncoder。虽然我们现在正在搭建的是一个认证服务器，但是认证服务器本身也可以对外提供REST服务，比如通过Token获取当前登录用户信息，注销当前Token等，所以它也是一台资源服务器。于是我们需要定义一个资源服务器的配置类，在cc.mrbird.febs.auth.configure包下新建MResourceServerConfigure类：

```java
@Configuration
@EnableResourceServer
public class MResourceServerConfigure extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated();
    }
}
```

MResourceServerConfigure继承了ResourceServerConfigurerAdapter，并重写了configure(HttpSecurity http)方法，通过requestMatchers().antMatchers("/**")的配置表明该安全配置对所有请求都生效。类上的@EnableResourceServer用于开启资源服务器相关配置。相信到这里你肯定会有点困惑，貌似MSecurityConfigure和MResourceServerConfigure所做的工作是类似的，MSecurityConfigure对/oauth/开头的请求生效，而MResourceServerConfigure对所有请求都生效，那么当一个请求进来时，到底哪个安全配置先生效呢？其实并没有哪个配置先生效这么一说，当在Spring Security中定义了多个过滤器链的时候，根据其优先级，只有优先级较高的过滤器链会先进行匹配。那么MSecurityConfigure和MResourceServerConfigure的优先级是多少？首先我们查看MSecurityConfigure继承的类WebSecurityConfigurerAdapter的源码：

```java
@Order(100)
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {
   ......
}
```

可以看到类上使用了@Order(100)标注，说明其顺序是100。再来看看MResourceServerConfigure类上@EnableResourceServer注解源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ResourceServerConfiguration.class})
public @interface EnableResourceServer {
}
```

所以MResourceServerConfigure的顺序是3。在Spring中，数字越小，优先级越高，也就是说MResourceServerConfigure的优先级要高于MSecurityConfigure，这也就意味着所有请求都会被MResourceServerConfigure过滤器链处理，包括/oauth/开头的请求。这显然不是我们要的效果，我们原本是希望以/oauth/开头的请求由MSecurityConfigure过滤器链处理，剩下的其他请求由MResourceServerConfigure过滤器链处理。为了解决上面的问题，我们可以手动指定这两个类的优先级，让MSecurityConfigure的优先级高于MResourceServerConfigure。在MSecurityConfigure类上使用Order(2)注解标注即可。

说了这么多，总结下MSecurityConfigure和MResourceServerConfigure的区别吧：MSecurityConfigure用于处理/oauth开头的请求，Spring Cloud OAuth内部定义的获取令牌，刷新令牌的请求地址都是以/oauth/开头的，也就是说MSecurityConfigure用于处理和令牌相关的请求；MResourceServerConfigure用于处理非/oauth/开头的请求，其主要用于资源的保护，客户端只能通过OAuth2协议发放的令牌来从资源服务器中获取受保护的资源。
接着我们定义一个和认证服务器相关的安全配置类。在configure包下新建MAuthorizationServerConfigure，代码如下所示:

```java
@Configuration
@EnableAuthorizationServer
public class MAuthorizationServerConfigure extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;
    @Autowired
    private MUserDetailService userDetailService;
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("m")
                .secret(passwordEncoder.encode("123456"))
                .authorizedGrantTypes("password", "refresh_token")
                .scopes("all");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.tokenStore(tokenStore())
                .userDetailsService(userDetailService)
                .authenticationManager(authenticationManager)
                .tokenServices(defaultTokenServices());
    }

    @Bean
    public TokenStore tokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }

    @Primary
    @Bean
    public DefaultTokenServices defaultTokenServices() {
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setTokenStore(tokenStore());
        tokenServices.setSupportRefreshToken(true);
        tokenServices.setAccessTokenValiditySeconds(60 * 60 * 24);
        tokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);
        return tokenServices;
    }
}
```

MAuthorizationServerConfigure继承AuthorizationServerConfigurerAdapter适配器，使用@EnableAuthorizationServer注解标注，开启认证服务器相关配置。上面代码中，我们注入了在MSecurityConfigure配置类中注册的BeanAuthenticationManager和PasswordEncoder。此外，由于之前我们在m-common中引入了spring-boot-starter-data-redis依赖，而m-auth模块引用了m-common模块，所以在m-auth的上下文中已经装配好了Redis相关配置，如RedisConnectionFactory（自动装配特性）。在febs-auth中采用的是Redis默认配置，所以你会发现我们并没有在配置类application.yml中编写和Redis有关的配置，但是为了更为直观，建议还是在application.yml中添加如下配置：

```yml
spring:
  application:
    name: m-auth
  redis:
    database: 0
    host: dev.com
    port: 6379
    jedis:
      pool:
        min-idle: 8
        max-idle: 500
        max-active: 2000
        max-wait: 10000
    timeout: 5000
```

在MAuthorizationServerConfigure中，tokenStore使用的是RedisTokenStore，认证服务器生成的令牌将被存储到Redis中。defaultTokenServices指定了令牌的基本配置，比如令牌有效时间为60 * 60 * 24秒，刷新令牌有效时间为60 * 60 * 24 * 7秒，setSupportRefreshToken设置为true表示开启刷新令牌的支持。MAuthorizationServerConfigure配置类中重点需要介绍的是configure(ClientDetailsServiceConfigurer clients)方法。该方法主要配置了：客户端从认证服务器获取令牌的时候，必须使用client_id为m，client_secret为123456的标识来获取；该client_id支持password模式获取令牌，并且可以通过refresh_token来获取新的令牌；在获取client_id为febs的令牌的时候，scope只能指定为all，否则将获取失败；如果需要指定多个client，可以继续使用withClient配置。在定义好这三个配置类后，我们还需要定义一个用于校验用户名密码的类，也就是上面提到的MUserDetailService。在cc.mrbird.febs.auth路径下新增service包，然后在service包下新增MUserDetailService类，代码如下所示：

```java
@Service
public class MUserDetailService implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        MAuthUser user = new MAuthUser();
        user.setUsername(username);
        user.setPassword(this.passwordEncoder.encode("123456"));

        return new User(username, user.getPassword(), user.isEnabled(),
                user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                user.isAccountNonLocked(), AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));
    }
}
```

MUserDetailService实现了UserDetailsService接口的loadUserByUsername方法。loadUserByUsername方法返回一个UserDetails对象，该对象也是一个接口，包含一些用于描述用户信息的方法，源码如下：

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

这些方法的含义如下：

- getAuthorities获取用户包含的权限，返回权限集合，权限是一个继承了GrantedAuthority的对象；
- getPassword和getUsername用于获取密码和用户名；
- isAccountNonExpired方法返回boolean类型，用于判断账户是否未过期，未过期返回true反之返回false；
- isAccountNonLocked方法用于判断账户是否未锁定；
- isCredentialsNonExpired用于判断用户凭证是否没过期，即密码是否未过期；
- isEnabled方法用于判断用户是否可用。实际中我们可以自定义UserDetails接口的实现类，也可以直接使用Spring Security提供的UserDetails接口实现类org.springframework.security.core.userdetails.User。
  MUserDetailService中MAuthUser为我们自定义的用户实体类，代表我们从数据库中查询出来的用户。我们在m-common中定义该实体类，在m-cmmon模块下新增entity包，然后在entity包下新增MAuthUser：

```java
@Data
public class MAuthUser implements Serializable {
    private static final long serialVersionUID = -1748289340320186418L;

    private String username;

    private String password;

    private boolean accountNonExpired = true;

    private boolean accountNonLocked= true;

    private boolean credentialsNonExpired= true;

    private boolean enabled= true;
}
```


@Data注解为lombok注解，用于自动生成get，set方法。在FebsUserDetailService的loadUserByUsername方法中，我们模拟了一个用户，用户名为用户输入的用户名，密码为123456（后期再改造为从数据库中获取用户），然后返回org.springframework.security.core.userdetails.User。这里使用的是User类包含7个参数的构造器，其还包含一个三个参数的构造器User(String username, String password,Collection<? extends GrantedAuthority> authorities)，由于权限参数不能为空，所以这里先使用AuthorityUtils.commaSeparatedStringToAuthorityList方法模拟一个user:add权限。最后定义一个Controller，对外提供一些REST服务。在cc.mrbird.febs.auth路径下新增controller包，在controller包下新增SecurityController：

```java
@RestController
public class SecurityController {

    @Autowired
    private ConsumerTokenServices consumerTokenServices;

    @GetMapping("oauth/test")
    public String testOauth() {
        return "oauth";
    }

    @GetMapping("user")
    public Principal currentUser(Principal principal) {
        return principal;
    }

    @DeleteMapping("signout")
    public MResponse signout(HttpServletRequest request) throws MAuthException {
        String authorization = request.getHeader("Authorization");
        String token = StringUtils.replace(authorization, "bearer ", "");
        MResponse febsResponse = new MResponse();
        if (!consumerTokenServices.revokeToken(token)) {
            throw new MAuthException("退出登录失败");
        }
        return febsResponse.message("退出登录成功");
    }
}
```

其中currentUser用户获取当前登录用户，signout方法通过ConsumerTokenServices来注销当前Token。MResponse为系统的统一相应格式，我们在m-common模块中定义它，在M-common模块的com.work.common.entity路径下新增MResponse类：

```java
public class MResponse extends HashMap<String, Object> {

    private static final long serialVersionUID = -8713837118340960775L;

    public MResponse message(String message) {
        this.put("message", message);
        return this;
    }

    public MResponse data(Object data) {
        this.put("data", data);
        return this;
    }

    @Override
    public MResponse put(String key, Object value) {
        super.put(key, value);
        return this;
    }

    public String getMessage() {
        return String.valueOf(get("message"));
    }

    public Object getData() {
        return get("data");
    }
}
```

MAuthException为自定义异常，在m-common模块com.work.common.exception路径下新增exception包，然后在该包下新增MAuthException：

```java
public class MAuthException  extends Exception{

    private static final long serialVersionUID = -6916154462432027437L;

    public MAuthException(String message){
        super(message);
    }
}
```

**接口测试**

1. 获取token
   ![dbf996b63e9b5636e4d3e5547b66a1bc.png](微服务搭建.assets/Image [18].png)
2. 获取受保护资源
   ![502d8fb534b104ed055810c4a78b4459.png](微服务搭建.assets/Image [19].png)

### 1.7 网关服务搭建

在微服务的架构中，服务网关就是一个介于客户端与服务端之间的中间层。在这种情况下，客户端只需要跟服务网关交互，无需调用具体的微服务接口。这样的好处在于，客户端可以降低复杂性，无需关注具体是哪个微服务在提供服务。这一节我们将使用Spring Cloud Zuul搭建微服务网关m-gateway。

点击IDEA菜单栏 File -> New -> Module...，模板选择Spring Initializr，Module SDK选择JDK 1.8：

![image-20210508132316482](微服务搭建.assets/image-20210508132316482.png)

![image-20210508132358466](微服务搭建.assets/image-20210508132358466.png)

在依赖列表里添加zuul。

![image-20210508132444377](微服务搭建.assets/image-20210508132444377.png)

修改pom:

```yml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <artifactId>m-cloud</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../m-cloud/pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.work</groupId>
    <artifactId>m-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>m-gateway</name>
    <description>网关服务</description>


    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>com.work</groupId>
            <artifactId>m-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>



    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

在上述pom配置中，我们指定了父项目为m-cloud，并且引入了通用模块m-common。

修改m-cloud模块的pom，在modules标签里引入m-gateway：

```yml
 <modules>
        <module>../m-common</module>
        <module>../m-register</module>
        <module>../m-auth</module>
        <module>../m-gateway</module>
    </modules>
```

在m-gateway的入口类`MGatewayApplication`上添加`@EnableDiscoveryClient`注解，开启服务注册与发现，添加`@EnableZuulProxy`注解，开启Zuul服务网关功能。

接着编写配置文件application.yml，先添加如下内容：

```yml
server:
  port: 8301

spring:
  application:
    name: m-gateway

eureka:
  instance:
    lease-renewal-interval-in-seconds: 20
  client:
    register-with-eureka: true
    fetch-registry: true
    instance-info-replication-interval-seconds: 30
    registry-fetch-interval-seconds: 3
    serviceUrl:
      defaultZone: http://m:123456@localhost:8001/register/eureka/
```

应用端口号为8301，服务名称为m-gateway，剩下的Eureka配置在上一节微服务认证服务器搭建过程中已经进行了详细介绍，所以这里就不再赘述了。

接着在application.yml继续添加和Zuul有关的配置，内容如下所示：

```yml
zuul:
  routes:
    auth:
      path: /auth/**
      serviceId: m-auth
      sensitiveHeaders: "*"
  retryable: true
  ignored-services: "*"
  ribbon:
    eager-load:
      enabled: true

ribbon:
  ReadTimeout: 3000
```

以`/auth`开头的请求都会被转发到名称为m-auth的服务上，由于我们需要在请求头中携带令牌，所以`sensitiveHeaders`设置为`*`，表示不过滤请求头信息，即请求的请求头信息将原封不动的转发出去。此外，因为Zuul已经包含了ribbon和hystrix依赖，所以我们在使用Zuul的同时，可以添加ribbon和hystrix相关配置。

上述配置中剩下的内容含义如下：

- `zuul.retryable`，设置为true时，表示开启重试机制；
- `zuul.ignored-services`，Zuul配合Eureka后会有一套默认的配置规则，这里我们只想请求根据我们显示配置的路由规则走，所以设置为`*`，表示关闭所有默认路由配置规则；
- `zuul.ribbon.eager-load.enabled`，Zuul内部通过Ribbon按照一定的负载均衡算法来获取服务，Ribbon进行客户端负载均衡的Client并不是在服务启动的时候就初始化好的，而是在调用的时候才会去创建相应的Client，所以第一次调用的耗时不仅仅包含发送HTTP请求的时间，还包含了创建RibbonClient的时间，这样一来如果创建时间速度较慢，同时设置的超时时间又比较短的话，第一次请求很容易出现超时的情况。设置为true的时候表示开启Ribbon的饥饿加载模式，即在应用启动的时候就去获取相应的Client备用。
- `ribbon.ReadTimeout`，设置请求超时时间，单位为毫秒；

配置文件编写完毕后，我们需要编写一个web安全配置类。在m-gateway模块下的com.work.m.gateway.configure包下新增`MGatewaySecurityConfigure`配置类：

```java
@EnableWebSecurity
public class MGatewaySecurityConfigure extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
    }
}
```

因为m-gateway引入了m-common模块，m-common模块包含了Spring Cloud Security依赖，所以我们需要定义一个自己的WebSecurity配置类，来覆盖默认的。这里主要是关闭了csrf功能，否则会报csrf相关异常。

**接口测试**

1. 获取token

   ![image-20210508134938454](微服务搭建.assets/image-20210508134938454.png)

2.  获取保护资源

   ![image-20210508135139826](微服务搭建.assets/image-20210508135139826.png)

### 1.8 资源服务搭建

本节的目标是搭建两个微服务提供者（同时也是资源服务器）m-server-system和m-server-test，主要目的是为了演示认证服务器和资源服务器在分离的模式下，如何进行资源保护和资源获取。

#### 1.8.1 搭建m-server-system

因为存在多个微服务提供者，所以我们先新增一个m-server作为这些微服务的父项目，统一进行管理。

点击IDEA菜单栏 File -> New -> Module...新增一个Maven模块：

![image-20210508100409478](微服务搭建.assets/image-20210508100409478.png)

![image-20210508100440071](微服务搭建.assets/image-20210508100440071.png)

然后点击Finish完成模块创建。

因为m-server是一个纯聚合模块，所以可以把它src目录下的内容全部删了，至此，项目结构如下所示：

![image-20210508100549493](微服务搭建.assets/image-20210508100549493.png)

修改m-server的pom，引入m-common模块：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>m-cloud</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../m-cloud/pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.work</groupId>
    <artifactId>m-server</artifactId>
    <version>1.0-SNAPSHOT</version>
    <description>微服务</description>


    <dependencies>
        <dependency>
            <groupId>com.work</groupId>
            <artifactId>m-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

因为在创建的时候已经选择了m-cloud作为父模块，所以m-cloud模块的pom里已经引入了m-server作为子模块：

```xml
 <modules>
        <module>../m-common</module>
        <module>../m-register</module>
        <module>../m-auth</module>
        <module>../m-server</module>
    </modules>
```

创建好m-server模块后，我们开始创建m-server-system模块。点击IDEA菜单栏 File -> New -> Modules...，选择Spring Initialzr作为模板，Module SDK选择JDK 1.8：

![image-20210508112604788](微服务搭建.assets/image-20210508112604788.png)

![image-20210508112646376](微服务搭建.assets/image-20210508112646376.png)

修改m-server-system模块的pom，内容如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>m-server</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.work</groupId>
    <artifactId>m-server-system</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>m-server-system</name>
    <description>微服务提供者</description>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

在该pom中，指定了父模块为m-server，在m-server模块的pom的modules标签里也许引入m-server-system：

```xml
<modules>
        <module>m-server-system</module>
    </modules>
```

接下来开始编写和m-server-system模块相关的代码。

在m-server-system模块的入口类`MServerSystemApplication`上添加`@EnableDiscoveryClient`注解，开启服务注册与发现：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MServerSystemApplication {

    public static void main(String[] args) {
        SpringApplication.run(MServerSystemApplication.class, args);
    }

}
```

然后编写项目配置文件application.yml，内容如下所示：

```yml
server:
  port: 8201

spring:
  application:
    name: m-server-system

eureka:
  instance:
    lease-renewal-interval-in-seconds: 20
  client:
    register-with-eureka: true
    fetch-registry: true
    instance-info-replication-interval-seconds: 30
    serviceUrl:
      defaultZone: http://m:123456@localhost:8001/register/eureka/
```

应用端口号为8201，服务名称为m-server-system，剩下的配置为Eureka相关配置，前面章节都详细介绍过了，这里也不赘述了。接下来开始编写和安全有关的代码。

既然m-server-system是一个资源服务器，那么我们就必须创建一个资源服务器配置类。新建com.work.m.server.system.configure包，然后在该包下新增`MServerSystemResourceServerConfigure`配置类：

```java
@Configuration
@EnableResourceServer
public class MServerSystemResourceServerConfigure  extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated();
    }
}
```

上述配置表示所有访问m-server-system的请求都需要认证，只有通过认证服务器发放的令牌才能进行访问。

然后在m-server-system的入口类`MServerSystemApplication`上添加`@EnableGlobalMethodSecurity(prePostEnabled = true)`注解，表示开启Spring Cloud Security权限注解。

在配置文件application.yml添加oauth2相关配置：

```yml
security:
  oauth2:
    resource:
      id: ${spring.application.name}
      user-info-uri: http://localhost:8301/auth/user
```

`user-info-uri`路径为http://localhost:8301/auth/user，通过微服务网关，该请求将被转发到http://localhost:8101/user。该配置的作用主要有两个：

1. 到认证服务器里校验当前请求头中的令牌是否为合法的令牌；
2. 通过当前令牌获取对应的用户信息。

接着创建一个Controller，对外提供一些REST服务。在cc.mrbird.febs.server.system路径下新建controller包，然后在该包下新增`TestController`：

```java
@RestController
public class TestController {

    @GetMapping("info")
    public String test(){
        return "m-server-system";
    }

    @GetMapping("user")
    public Principal currentUser(Principal principal) {
        return principal;
    }
}
```

#### 1.8.2 搭建m-server-system-test

点击IDEA菜单栏 File -> New -> Module...，模板选择Spring Initializr，Module SDK选择JDK1.8：

![image-20210508130824195](微服务搭建.assets/image-20210508130824195-1621168062025.png)

![image-20210508130915278](微服务搭建.assets/image-20210508130915278-1621168063876.png)

项目建完以后，结构如之前所述，删除不必要的内容。

修改m-server-test模块的pom，内容如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>m-server</artifactId>
        <groupId>com.work</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.work</groupId>
    <artifactId>m-server-system-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>m-server-system</name>
    <description>微服务提供者测试</description>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

上述配置中，指定了父模块为m-server。

因为m-server-test模块的代码和m-server-system基本一致，所以下面开始贴代码，不做过多的说明。

在m-server-test模块的入口类`MServerTestApplication`上使用`@EnableDiscoveryClient`和`@EnableGlobalMethodSecurity(prePostEnabled = true)`注解标注。

编写配置文件application.yml，内容如下所示：

```yml
server:
  port: 8202

spring:
  application:
    name: m-server-test

eureka:
  instance:
    lease-renewal-interval-in-seconds: 20
  client:
    register-with-eureka: true
    fetch-registry: true
    instance-info-replication-interval-seconds: 30
    registry-fetch-interval-seconds: 3
    serviceUrl:
      defaultZone: http://m:123456@localhost:8001/register/eureka/
security:
  oauth2:
    resource:
      id: ${spring.application.name}
      user-info-uri: http://localhost:8301/auth/user
```

应用端口号为8202，服务名称为m-server-test。

接着在com.work.m.server.system.test.configure包下新建`MServerTestResourceServerConfigure`资源服务器配置类：

```java
@Configuration
@EnableResourceServer
public class MServerTestResourceServerConfigure extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated();
    }
}
```

在com.work.m.server.system.test.controller包下新建`TestController`对外提供一些REST服务：

```java
@RestController
public class TestController {

    @GetMapping("test1")
    @PreAuthorize("hasAnyAuthority('user:add')")
    public String test1(){
        return "拥有'user:add'权限";
    }

    @GetMapping("test2")
    @PreAuthorize("hasAnyAuthority('user:update')")
    public String test2(){
        return "拥有'user:update'权限";
    }

    @GetMapping("user")
    public Principal currentUser(Principal principal) {
        return principal;
    }
}
```

上面代码中，使用了权限注解保护方法，当用户已认证并且拥有`user:add`权限的时候，才能访问`test1`方法；当用户已认证并拥有`user:update`权限的时候才能访问`test2`方法。

## 2. 架构完善

### 2.1 参数配置化

在搭建m-auth的时候，我们在认证服务器配置类`MAuthorizationServerConfigure`里使用硬编码的形式配置了client_id，client_secret等信息。硬编码的形式不利于代码维护和升级，所以我们需要将它改造为可配置的方式。

在m-auth模块的com.work.m.auth.properties包下新建一个Client配置类`MClientsProperties`：

```java
@Data
public class MClientsProperties {
    private String client;
    private String secret;
    private String grantType = "password,authorization_code,refresh_token";
    private String scope = "all";
}
```

`client`对应client_id，`secret`对应client_secret，`grantType`对应当前令牌支持的认证类型，`scope`对应认证范围。`grantType`和`scope`包含默认值。

接着新建一个和Auth相关的配置类`MAuthProperties`：

```java
@Data
@SpringBootConfiguration
@PropertySource(value = {"classpath:m-auth.properties"})
@ConfigurationProperties(prefix = "m.auth")
public class MAuthProperties {

    private MClientsProperties[] clients = {};
    private int accessTokenValiditySeconds = 60 * 60 * 24;
    private int refreshTokenValiditySeconds = 60 * 60 * 24 * 7;
}
```

`clients`属性类型为上面定义的`FebsClientsProperties`，因为一个认证服务器可以根据多种Client来发放对应的令牌，所以这个属性使用的是数组形式；`accessTokenValiditySeconds`用于指定access_token的有效时间，默认值为`60 * 60 * 24`秒；`refreshTokenValiditySeconds`用于指定refresh_token的有效时间，默认值为`60 * 60 * 24 * 7`秒。

`@PropertySource(value = {"classpath:m-auth.properties"})`用于指定读取的配置文件路径；`@ConfigurationProperties(prefix = "m.auth")`指定了要读取的属性的统一前缀名称为`febs.auth`；`@SpringBootConfiguration`实质上为`@Component`的派生注解，用于将`MAuthProperties`纳入到IOC容器中。

自定义配置类还需引入`spring-boot-configuration-processor`依赖，因为这个依赖会在多个微服务子系统里使用到，所以将其添加到m-common的pom文件中：

```yml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

接下来在m-auth的resources路径下新建配置文件`m-auth.properties`：

```yml
m.auth.accessTokenValiditySeconds=86400
m.auth.refreshTokenValiditySeconds=604800

m.auth.clients[0].client=m
m.auth.clients[0].secret=123456
m.auth.clients[0].grantType=password,authorization_code,refresh_token
m.auth.clients[0].scope=all
```

去除`m.auth`前缀，剩下部分和`MAuthProperties`配置类属性名称对应上的话，就会被读取到`MAuthProperties`相应的属性中。数组形式的属性值使用`[]`加元素下标表示，具体可以参考properties文件的语法。

定义好`MAuthProperties`配置类后，我们就可以在认证服务器配置类`MAuthorizationServerConfigure`中注入使用了，改造`MAuthorizationServerConfigure`，代码如下所示：

```java
/**
 * oauth2 配置
 */
@Configuration
@EnableAuthorizationServer
public class MAuthorizationServerConfigure extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;
    @Autowired
    private MUserDetailService userDetailService;
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Autowired
    private MAuthProperties authProperties;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        MClientsProperties[] clientsArray = authProperties.getClients();
        InMemoryClientDetailsServiceBuilder builder = clients.inMemory();
        if (ArrayUtils.isNotEmpty(clientsArray)) {
            for (MClientsProperties client : clientsArray) {
                if (StringUtils.isBlank(client.getClient())) {
                    throw new Exception("client不能为空");
                }
                if (StringUtils.isBlank(client.getSecret())) {
                    throw new Exception("secret不能为空");
                }
                String[] grantTypes = StringUtils.splitByWholeSeparatorPreserveAllTokens(client.getGrantType(), ",");
                builder.withClient(client.getClient())
                        .secret(passwordEncoder.encode(client.getSecret()))
                        .authorizedGrantTypes(grantTypes)
                        .scopes(client.getScope());
            }
        }
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.tokenStore(tokenStore())
                .userDetailsService(userDetailService)
                .authenticationManager(authenticationManager)
                .tokenServices(defaultTokenServices());
    }

    @Bean
    public TokenStore tokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }

    @Primary
    @Bean
    public DefaultTokenServices defaultTokenServices() {
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setTokenStore(tokenStore());
        tokenServices.setSupportRefreshToken(true);
        tokenServices.setAccessTokenValiditySeconds(authProperties.getAccessTokenValiditySeconds());
        tokenServices.setRefreshTokenValiditySeconds(authProperties.getRefreshTokenValiditySeconds());
        return tokenServices;
    }
}
```

修改的地方主要有：

1. `configure(ClientDetailsServiceConfigurer clients)`方法由原先硬编码的形式改造成了从配置文件读取配置的形式，并且判断了client和secret不能为空；
2. `defaultTokenServices`方法指定有效时间也从原先硬编码的形式改造成了从配置文件读取配置的形式。

在后续的编码中，参数的配置方式都采取这种形式。

### 2.2 异常处理

目前系统的异常响应都是以Spring内部构建好的默认的格式返回，这一节中我们将自定义各种异常处理器，将默认的异常响应“翻译”为符合我们期望的格式响应。

默认情况下，当我们在获取令牌时输入错误的用户名或密码，系统返回如下格式响应:

```xml
{
    "error": "invalid_grant",
    "error_description": "Bad credentials"
}
```

当grant_type错误时，系统返回：

```xml
{
    "error": "unsupported_grant_type",
    "error_description": "Unsupported grant type: passwordd"
}
```

接下来我们定义一个异常翻译器，将这些认证类型异常翻译为友好的格式。在febs-auth模块cc.mrbird.febs.auth路径下新建translator包，然后在该包下新建`MWebResponseExceptionTranslator`：

```java
@Slf4j
@Component
public class MWebResponseExceptionTranslator implements WebResponseExceptionTranslator {

    @Override
    public ResponseEntity translate(Exception e) {
        ResponseEntity.BodyBuilder status = ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR);
        MResponse response = new MResponse();
        String message = "认证失败";
        log.error(message, e);
        if (e instanceof UnsupportedGrantTypeException) {
            message = "不支持该认证类型";
            return status.body(response.message(message));
        }
        if (e instanceof InvalidGrantException) {
            if (StringUtils.containsIgnoreCase(e.getMessage(), "Invalid refresh token")) {
                message = "refresh token无效";
                return status.body(response.message(message));
            }
            if (StringUtils.containsIgnoreCase(e.getMessage(), "locked")) {
                message = "用户已被锁定，请联系管理员";
                return status.body(response.message(message));
            }
            message = "用户名或密码错误";
            return status.body(response.message(message));
        }
        return status.body(response.message(message));
    }
}
```

`MWebResponseExceptionTranslator`实现了`WebResponseExceptionTranslator`接口，用于覆盖默认的认证异常响应。在`translate`方法中，我们通过`Exception`异常对象的类型和内容将异常归类，并且统一返回500HTTP状态码（`HttpStatus.INTERNAL_SERVER_ERROR`）。

此外，类上`@Component`注解用于将当前类注册为一个Bean，`@sl4j`注解为lombok类型注解，用于往当前类中注入`org.slf4j.Logger`日志打印对象。

要让这个异常翻译器生效，我们还需在认证服务器配置类`MAuthorizationServerConfigure`的`configure(AuthorizationServerEndpointsConfigurer endpoints)`方法里指定它：

```java
/**
 * oauth2 配置
 */
@Configuration
@EnableAuthorizationServer
public class MAuthorizationServerConfigure extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private MWebResponseExceptionTranslator exceptionTranslator;


    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.tokenStore(tokenStore())
                .userDetailsService(userDetailService)
                .authenticationManager(authenticationManager)
                .tokenServices(defaultTokenServices())
                .exceptionTranslator(exceptionTranslator);
    }

}
```

**处理资源服务器异常**

资源服务器异常主要有两种：令牌不正确返回401和用户无权限返回403。因为资源服务器有多个，所以相关的异常处理类可以定义在m-common通用模块里。

在m-common模块com.work.common.handler包下新建`MAuthExceptionEntryPoint`：

```java
public class MAuthExceptionEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        MResponse febsResponse = new MResponse();

        response.setContentType("application/json;charset=UTF-8");
        response.setStatus(401);
        response.getOutputStream().write(JSONObject.toJSONString(febsResponse.message("token无效")).getBytes());
    }
}
```

`MAuthExceptionEntryPoint`实现了`AuthenticationEntryPoint`接口的`commence`方法，在方法内自定义了响应的格式。

其中`application/json;charset=UTF-8`和HTTP状态码401，Spring都提供了相应的常量类，所以上面代码可以优化为：

```java
public class MAuthExceptionEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        MResponse febsResponse = new MResponse();

        response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.getOutputStream().write(JSONObject.toJSONString(febsResponse.message("token无效")).getBytes());
    }

}
```

此外，构造响应的这几行代码后续会经常使用到，所以我们可以将它抽取为一个工具方法。

在m-common模块的com.work.common.util包下新建`MUtil`工具类，并定义如下方法：

```java
@Slf4j
public class MUtil {

    /**
     * 设置响应
     *
     * @param response    HttpServletResponse
     * @param contentType content-type
     * @param status      http状态码
     * @param value       响应内容
     * @throws IOException IOException
     */
    public static void makeResponse(HttpServletResponse response, String contentType,
                                    int status, Object value) throws IOException {
        response.setContentType(contentType);
        response.setStatus(status);
        response.getOutputStream().write(JSONObject.toJSONString(value).getBytes());
    }
}
```

于是`MAuthExceptionEntryPoint`最终代码如下所示:

```java
public class MAuthExceptionEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        MResponse febsResponse = new MResponse();
        MUtil.makeResponse(
                response, MediaType.APPLICATION_JSON_UTF8_VALUE,
                HttpServletResponse.SC_UNAUTHORIZED, febsResponse.message("token无效")
        );
    }

}
```

接着在m-common的com.work.common.handler路径下新建`MAccessDeniedHandler`用于处理403类型异常：

```java
public class MAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
        MResponse febsResponse = new MResponse();
        MUtil.makeResponse(
                response, MediaType.APPLICATION_JSON_UTF8_VALUE,
                HttpServletResponse.SC_FORBIDDEN, febsResponse.message("没有权限访问该资源"));
    }
}
```

响应码为`HttpServletResponse.SC_FORBIDDEN`，即403。

因为m-common模块是一个普通的maven项目，并不是一个Spring Boot项目，所以即使在这两个类上使用`@Component`注解标注，它们也不能被成功注册到各个微服务子系统的Spring IOC容器中。我们可以使用`@Enable`模块驱动的方式来解决这个问题。

在m-common模块的com.work.common.configure包下新建`MAuthExceptionConfigure`配置类：

```java
public class MAuthExceptionConfigure {

    @Bean
    @ConditionalOnMissingBean(name = "accessDeniedHandler")
    public MAccessDeniedHandler accessDeniedHandler() {
        return new MAccessDeniedHandler();
    }

    @Bean
    @ConditionalOnMissingBean(name = "authenticationEntryPoint")
    public MAuthExceptionEntryPoint authenticationEntryPoint() {
        return new MAuthExceptionEntryPoint();
    }

}
```

在该配置类中，我们注册了`MAccessDeniedHandler`和`MAuthExceptionEntryPoint`。`@ConditionalOnMissingBean`注解的意思是，当IOC容器中没有指定名称或类型的Bean的时候，就注册它。以`@ConditionalOnMissingBean(name = "accessDeniedHandler")`为例，当微服务系统的Spring IOC容器中没有名称为`accessDeniedHandler`的Bean的时候，就将`MAccessDeniedHandler`注册为一个Bean。这样做的好处在于，子系统可以自定义自个儿的资源服务器异常处理器，覆盖我们在m-common通用模块里定义的。

接着定义一个注解来驱动该配置类。

在m-common模块的com.work.common.annotation包下新建`EnableMAuthExceptionHandler`注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MAuthExceptionConfigure.class)
public @interface EnableMAuthExceptionHandler {

}
```

在该注解上，我们使用`@Import`将`MAuthExceptionConfigure`配置类引入了进来。

最后演示下如何使用`@EnableMAuthExceptionHandler`注解。

因为m-auth，m-server-system，m-server-test都是资源服务器，所以它们三个都需要使用`@EnableMAuthExceptionHandler`注解实现资源服务器异常处理。三者的步骤都是一模一样的，所以这里以febs-auth模块为例，剩下的febs-server-system和febs-server-test照着配置即可。

在m-auth模块的入口类上使用`@EnableMAuthExceptionHandler`注解标注：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableMAuthExceptionHandler
public class MAuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(MAuthApplication.class, args);
    }

}
```

通过该注解，m-auth模块的IOC容器里就已经注册了`MAccessDeniedHandler`和`MAuthExceptionEntryPoint`。

然后在资源服务器配置类`MResourceServerConfigurer`里注入它们，并配置：

```java
@Configuration
@EnableResourceServer
public class MResourceServerConfigure extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .requestMatchers().antMatchers("/**")
                .and()
                .authorizeRequests()
                .antMatchers("/**").authenticated();
    }


    @Autowired
    private MAccessDeniedHandler accessDeniedHandler;
    @Autowired
    private MAuthExceptionEntryPoint exceptionEntryPoint;


    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.authenticationEntryPoint(exceptionEntryPoint)
                .accessDeniedHandler(accessDeniedHandler);
    }

}
```

**处理Zuul异常**

当Zuul转发请求超时时，系统返回如下响应：

```xml
{
    "timestamp": "2019-08-07T07:58:21.938+0000",
    "status": 504,
    "error": "Gateway Timeout",
    "message": "com.netflix.zuul.exception.ZuulException: Hystrix Readed time out"
}
```

当处理转发请求的微服务模块不可用时，系统返回：

```xml
{
    "timestamp": "2019-08-07T08:01:31.829+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "GENERAL"
}
```

自定义Zuul异常处理可以通过继承Zuul的`SendErrorFilter`过滤器来实现。

在m-gateway模块的com.work.m.gateway.filter路径下新建filter包，然后在该包下新建`MGatewayErrorFilter`过滤器：

```java
@Slf4j
@Component
public class MGatewayErrorFilter extends SendErrorFilter {

    @Override
    public Object run() {
        try {
            MResponse febsResponse = new MResponse();
            RequestContext ctx = RequestContext.getCurrentContext();
            String serviceId = (String) ctx.get(FilterConstants.SERVICE_ID_KEY);

            ExceptionHolder exception = findZuulException(ctx.getThrowable());
            String errorCause = exception.getErrorCause();
            Throwable throwable = exception.getThrowable();
            String message = throwable.getMessage();
            message = StringUtils.isBlank(message) ? errorCause : message;
            febsResponse = resolveExceptionMessage(message, serviceId, febsResponse);

            HttpServletResponse response = ctx.getResponse();
            MUtil.makeResponse(
                    response, MediaType.APPLICATION_JSON_UTF8_VALUE,
                    HttpServletResponse.SC_INTERNAL_SERVER_ERROR, febsResponse
            );
            log.error("Zull sendError：{}", febsResponse.getMessage());
        } catch (Exception ex) {
            log.error("Zuul sendError", ex);
            ReflectionUtils.rethrowRuntimeException(ex);
        }
        return null;
    }

    private MResponse resolveExceptionMessage(String message, String serviceId, MResponse febsResponse) {
        if (StringUtils.containsIgnoreCase(message, "time out")) {
            return febsResponse.message("请求" + serviceId + "服务超时");
        }
        if (StringUtils.containsIgnoreCase(message, "forwarding error")) {
            return febsResponse.message(serviceId + "服务不可用");
        }
        return febsResponse.message("Zuul请求" + serviceId + "服务异常");
    }
}
```

在该过滤器中，我们可以通过`RequestContext`获取到当前请求上下文，通过请求上下文可以获取到当前请求的服务名称`serviceId`和当前请求的异常对象`ExceptionHolder`等信息。通过异常对象我们可以继续获取到异常内容，根据不同的异常内容我们可以自定义想要的响应。

要让我们自定义的Zuul异常过滤器生效，还需要在m-gateway的配置文件中添加如下配置，让默认的异常过滤器失效：

```yaml
zuul:
  SendErrorFilter:
    error:
      disable: true
```

**全局异常处理器**

所谓的全局异常处理指的是全局处理Controller层抛出来的异常。因为全局异常处理器在各个微服务系统里都能用到，所以我们把它定义在m-common模块里。

在m-common模块的com.work.common.handler路径下新建`BaseExceptionHandler`：

```java
@Slf4j
public class BaseExceptionHandler {

    @ExceptionHandler(value = Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public MResponse handleException(Exception e) {
        log.error("系统内部异常，异常信息", e);
        return new MResponse().message("系统内部异常");
    }

    @ExceptionHandler(value = MAuthException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public MResponse handleFebsAuthException(MAuthException e) {
        log.error("系统错误", e);
        return new MResponse().message(e.getMessage());
    }

    @ExceptionHandler(value = AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public MResponse handleAccessDeniedException(){
        return new MResponse().message("没有权限访问该资源");
    }
}
```

然后以m-auth为例，在m-auth模块的com.work.m.auth.handler包下新建`GlobalExceptionHandler`类：

```java
@RestControllerAdvice
@Order(value = Ordered.HIGHEST_PRECEDENCE)
public class GlobalExceptionHandler extends BaseExceptionHandler {
}
```

对于通用的异常类型捕获可以在`BaseExceptionHandler`中定义，而当前微服务系统独有的异常类型捕获可以在`GlobalExceptionHandler`中定义。

m-server-system和m-server-test模块处理方式和m-auth一致，这里就不演示了。

### 2.3 Feign的使用

微服务之间服务的调用可以借助Spring Cloud Feign来完成，Spring Cloud Feign内部整合了Spring Cloud Ribbon和Spring Cloud Hystrix，所以它具有客户端负载均衡和服务容错的功能。Spring Cloud Feign本身使用也很简单，但是当我们需要用Feign访问受保护的服务的时候可能会遇到一些问题，这一节我们来逐步解决这些问题。

**Spring Cloud Feign微服务调用**

正如上面所说，微服务间的服务调用可以用Spring Cloud Feign来完成，比如m-server-test内部调用m-server-system提供的REST服务，下面演示如何操作。

因为Feign在多个微服务系统里都需要用到，所以我们在他们共有的通用模块m-common的pom里引入Feign依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

我们在m-server-system模块的`TestController`里新增一个方法，供m-server-test远程调用演示：

```java
@RestController
public class TestController {
	……
    @GetMapping("hello")
    public String hello(String name) {
        return "hello" + name;
    }
}
```

然后在m-server-test模块的入口类`MServerTestApplication`上使用`@EnableFeignClients`注解标注，开启Feign Client功能：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableFeignClients
public class MServerSystemTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(MServerSystemTestApplication.class, args);
    }

}
```

要在m-server-test模块里通过Feign调用m-server-system的`/hello`服务很简单，只需要定义一个Feign Client接口即可。在m-server-test模块的com.work.m.server.system.test.service包下新建`IHelloService`：

```java
@FeignClient(value = "m-server-system", contextId = "helloServiceClient", fallbackFactory = HelloServiceFallback.class)
public interface IHelloService {

    @GetMapping("hello")
    String hello(@RequestParam("name") String name);
}
```

`IHelloService`上通过`@FeignClient`注解标注表明这是一个Feign Client，该注解的属性含义如下：

1. `value`指定远程服务的名称，这个名称对应m-server-system模块配置文件application.yml里`spring.application.name`的配置，即注册到Eureka里的服务名称；
2. `contextId`指定这个Feign Client的别名，当我们定义了多个Feign Client并且`value`值相同（即调用同一个服务）的时候，需要手动通过`contextId`设置别名，否则程序将抛出异常；
3. `fallbackFactory`指定了回退方法，当我们调用远程服务出现异常时，就会调用这个回退方法。`fallback`也可以指定回退方法，但`fallbackFactory`指定的回退方法里可以通过`Throwable`对象打印出异常日志，方便分析问题。

我们可以定义一个微服务名称常量类，方便后续使用。在m-common模块下的com.work.common.entity目录下新建`MServerConstant`：

```java
public class MServerConstant {
    public static final String M_AUTH = "m-auth";
    public static final String M_CLOUD = "m-cloud";
    public static final String M_COMMON = "m-common";
    public static final String M_GATEWAY = "m-gateway";
    public static final String M_REGISTER = "m-register";
    public static final String M_SERVER = "m-server";
    public static final String M_SERVER_SYSTEM= "m-server-system";
    public static final String M_SERVER_TEST= "m-server-test";
}
```

然后上面的Feign Client可以改造为:

```java
@FeignClient(value = MServerConstant.M_SERVER_SYSTEM, contextId = "helloServiceClient", fallbackFactory = HelloServiceFallback.class)
public interface IHelloService {

    @GetMapping("hello")
    String hello(@RequestParam("name") String name);
}
```

接着定义回退方法`HelloServiceFallback`。在m-server-test模块的com.work.m.server.system.test.service.fallback包下新建`HelloServiceFallback`：

```java
@Slf4j
@Component
public class HelloServiceFallback implements FallbackFactory<IHelloService> {
    @Override
    public IHelloService create(Throwable throwable) {
        return new IHelloService() {
            @Override
            public String hello(String name) {
                log.error("调用m-server-system服务出错", throwable);
                return "调用出错";
            }
        };
    }
}
```

`HelloServiceFallback`实现`FallbackFactory`，泛型指定为上面定义的`IHelloService`。

要让Feign的回退生效，需要在配置文件application.yml里添加如下配置：

```yaml
feign:
  hystrix:
    enabled: true
```

因为Feign的回退功能是基于Hystrix实现的，所以需要开启它。

使用Feign Client很简单，只需要像调用本地Service那样即可。在m-server-test的`TestController`中使用上面定义的`IHelloService`Feign Client：

```java
@RestController
public class TestController {
    
    @Autowired
    private IHelloService helloService;
    
    @GetMapping("hello")
    public String hello(String name){
        return this.helloService.hello(name);
    }
    ......   
}
```

逐步启动m-register、m-auth、m-gateway、m-server-system和m-server-test模块，进行调用测试。

先获取令牌，然后发送 [localhost:8301/test/hello](localhost:8301/test/hello) GET请求，请求头上携带令牌：

![image-20210509223825704](微服务搭建.assets/image-20210509223825704.png)

查看具体错误:

![image-20210509223918816](微服务搭建.assets/image-20210509223918816.png)

因为Feign在调用远程服务的时候，并不会帮我们把原HTTP请求头部的内容也携带上，所以访问m-server-system的`/hello`服务的时候，请求头部没有访问令牌，于是抛出了401异常。

**改造Feign**

要解决上面的问题也很简单，只需要拦截Feign请求，手动往请求头上加入令牌即可。

在m-common模块的com.work.common.configure路径下新建`MOAuth2FeignConfigure`配置类，在该配置类里注册一个Feign请求拦截器：

```java
public class MOAuth2FeignConfigure {

    @Bean
    public RequestInterceptor oauth2FeignRequestInterceptor() {
        return requestTemplate -> {
            Object details = SecurityContextHolder.getContext().getAuthentication().getDetails();
            if (details instanceof OAuth2AuthenticationDetails) {
                String authorizationToken = ((OAuth2AuthenticationDetails) details).getTokenValue();
                requestTemplate.header(HttpHeaders.AUTHORIZATION, "Bearer " + authorizationToken);
            }
        };
    }
}
```

上面代码中，我们通过`SecurityContextHolder`从请求上下文中获取了`OAuth2AuthenticationDetails`类型对象，并通过该对象获取到了请求令牌，然后在请求模板对象`requestTemplate`的头部手动将令牌添加上去。

此外，因为上面我们在m-server-test的配置文件里添加了`feign.hystrix.enabled:true`开启了hystrix熔断机制，要让上面的请求拦截器能够顺利获取到令牌，还需要在m-server-test的配置文件里添加如下配置：

```yaml
feign:
  hystrix:
    enabled: true
hystrix:
  shareSecurityContext: true
```

`hystrix.shareSecurityContext=true`配置把SecurityContext对象从你当前主线程传输到Hystrix线程。

这段配置需要Hystrix依赖，我们在m-common的pom里添加该依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

接着定义一个`@Enable`类型注解，让该配置类`MOAuth2FeignConfigure`生效。在m-common模块的com.work.common.annotation路径下新建`@EnableMOauth2FeignClient`注解，在该注解里引入`MOAuth2FeignConfigure`配置类：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MOAuth2FeignConfigure.class)
public @interface EnableMOauth2FeignClient {

}
```

在m-server-test模块的入口类上使用`@EnableMOauth2FeignClient`注解标注，然后重启m-server-test模块。再次发送 [localhost:8301/test/hello](localhost:8301/test/hello) GET请求：

![image-20210509224707209](微服务搭建.assets/image-20210509224707209.png)

可以看到，服务调用成功，不再报401异常。



### 2.4 微服务保护

目前我们所有客户端请求都是通过微服务网关转发完成的，虽然我们约定如此，但是还是可以直接通过访问微服务地址的方式来获取服务，比如直接使用PostMan发送 [localhost:8202/hello](localhost:8202/hello) GET请求：

![image-20210516172201376](微服务搭建.assets/image-20210516172201376.png)

可以看到直接访问的方式也能成功调用服务。为了避免客户端请求绕过网关，直接调用微服务，我们可以做一些处理来解决这个问题。

**自定义Zuul过滤器**

解决这个问题的大致思路是：在网关转发请求前，请求头部加入网关信息，然后在处理请求的微服务模块里定义全局拦截器，校验请求头部的网关信息，这样就能避免客户端直接访问微服务了。

在自定义Zuul过滤器前，我们先来简单了解下Zuul的核心过滤器。Zuul中默认定义了4种不同生命周期的过滤器类型，如下图所示：

![image-20210516172405308](微服务搭建.assets/image-20210516172405308.png)

这4种过滤器处于不同的生命周期，所以其职责也各不相同：

- **PRE**：PRE过滤器用于将请求路径与配置的路由规则进行匹配，以找到需要转发的目标地址，并做一些前置加工，比如请求的校验等；
- **ROUTING**：ROUTING过滤器用于将外部请求转发到具体服务实例上去；
- **POST**：POST过滤器用于将微服务的响应信息返回到客户端，这个过程种可以对返回数据进行加工处理；
- **ERROR**：上述的过程发生异常后将调用ERROR过滤器。ERROR过滤器捕获到异常后需要将异常信息返回给客户端，所以最终还是会调用POST过滤器。

Spring Cloud Zuul为各个生命周期阶段实现了一批过滤器，如下所示：

![76.png](微服务搭建.assets/ezENes.png)

这些过滤器的优先级和作用如下表所示：

| 生命周期 | 优先级 | 过滤器                  | 描述                       |
| :------- | :----- | :---------------------- | :------------------------- |
| pre      | -3     | ServletDetectionFilter  | 标记处理Servlet的类型      |
| pre      | -2     | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
| pre      | -1     | FormBodyWrapperFilter   | 包装请求体                 |
| route    | 1      | DebugFilter             | 标记调试标志               |
| route    | 5      | PreDecorationFilter     | 处理请求上下文供后续使用   |
| route    | 10     | RibbonRoutingFilte      | serviceId请求转发          |
| route    | 10     | SimpleHostRoutingFilter | url请求转发                |
| route    | 50     | SendForwardFilter       | forward请求转发            |
| post     | 0      | SendErrorFilter         | 处理有错误的请求响应       |
| post     | 10     | SendResponseFilter      | 处理正常的请求响应         |

从上面的表格可以看到，`PreDecorationFilter`用于处理请求上下文，优先级为5，所以我们可以定义一个优先级在`PreDecorationFilter`之后的过滤器，这样便可以拿到请求上下文。

在m-gateway模块的com.work.m.gateway.filter路径下新建`MGatewayRequestFilter`：

```java
@Slf4j
@Component
public class MGatewayRequestFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 6;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        String serviceId = (String) ctx.get(FilterConstants.SERVICE_ID_KEY);
        HttpServletRequest request = ctx.getRequest();
        String host = request.getRemoteHost();
        String method = request.getMethod();
        String uri = request.getRequestURI();

        log.info("请求URI：{}，HTTP Method：{}，请求IP：{}，ServerId：{}", uri, method, host, serviceId);

        byte[] token = Base64Utils.encode(("m:zuul:123456").getBytes());
        ctx.addZuulRequestHeader("ZuulToken", new String(token));
        return null;
    }
}
```

自定义Zuul过滤器需要继承`ZuulFilter`，并实现它的四个抽象方法：

1. **filterType**，对应Zuul生命周期的四个阶段：pre、post、route和error，我们要在请求转发出去前添加请求头，所以这里指定为pre；
2. **filterOrder**，过滤器的优先级，数字越小，优先级越高。`PreDecorationFilter`过滤器的优先级为5，所以我们可以指定为6让我们的过滤器优先级比它低；
3. **shouldFilter**，方法返回boolean类型，true时表示是否执行该过滤器的run方法，false则表示不执行；
4. **run**，定义过滤器的主要逻辑。这里我们通过请求上下文`RequestContext`获取了转发的服务名称`serviceId`和请求对象`HttpServletRequest`，并打印请求日志。随后往请求上下文的头部添加了Key为`ZuulToken`，Value为`m:zuul:123456`的信息。这两个值可以抽取到常量类中。

在m-common的com.work.common.entity路径下新建`MConstant`：

```java
public class MConstant {
    /**
     * Zuul请求头TOKEN名称（不要有空格）
     */
    public static final String ZUUL_TOKEN_HEADER = "ZuulToken";
    /**
     * Zuul请求头TOKEN值
     */
    public static final String ZUUL_TOKEN_VALUE = "m:zuul:123456";
}
```

然后上面的过滤器可以改造为：

```java
@Slf4j
@Component
public class MGatewayRequestFilter extends ZuulFilter {

    ……

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        String serviceId = (String) ctx.get(FilterConstants.SERVICE_ID_KEY);
        HttpServletRequest request = ctx.getRequest();
        String host = request.getRemoteHost();
        String method = request.getMethod();
        String uri = request.getRequestURI();

        log.info("请求URI：{}，HTTP Method：{}，请求IP：{}，ServerId：{}", uri, method, host, serviceId);

        byte[] token = Base64Utils.encode(MConstant.ZUUL_TOKEN_VALUE.getBytes());
        ctx.addZuulRequestHeader(MConstant.ZUUL_TOKEN_HEADER, new String(token));
        return null;
    }
}
```

定义好Zuul过滤器后，我们需要在各个微服务里定义一个全局拦截器拦截请求，并校验Zuul Token。这个拦截器需要被众多微服务模块使用，所以把它定义在通用模块m-common里。

**校验Zuul Token**

在m-common模块的com.work.common.interceptor包下新建`MServerProtectInterceptor`拦截器：

```java
public class MServerProtectInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        // 从请求头中获取 Zuul Token
        String token = request.getHeader(MConstant.ZUUL_TOKEN_HEADER);
        String zuulToken = new String(Base64Utils.encode(MConstant.ZUUL_TOKEN_VALUE.getBytes()));
        // 校验 Zuul Token的正确性
        if (StringUtils.equals(zuulToken, token)) {
            return true;
        } else {
            MResponse febsResponse = new MResponse();
            response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            response.getWriter().write(JSONObject.toJSONString(febsResponse.message("请通过网关获取资源")));
            return false;
        }
    }
}
```

`MServerProtectInterceptor`实现了`HandlerInterceptor`的`preHandle`方法，该拦截器可以拦截所有Web请求。在`preHandle`方法中，我们通过`HttpServletRequest`获取请求头中的Zuul Token，并校验其正确性，当校验不通过的时候返回403错误。

要让该过滤器生效我们需要定义一个配置类来将它注册到Spring IOC容器中，在m-common模块的com.work.common.configure路径下新建`MServerProtectConfigure`：

```java
public class MServerProtectConfigure implements WebMvcConfigurer {

    @Bean
    public HandlerInterceptor mServerProtectInterceptor() {
        return new MServerProtectInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(mServerProtectInterceptor());
    }
}
```

我们在该配置类里注册了`MServerProtectInterceptor`，并且将它加入到了Spring的拦截器链中。

同样的，要让该配置类生效，我们可以定义一个`@Enable`注解来驱动它。在m-common模块的com.work.common.annotation路径下新建`EnableMServerProtect`：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MServerProtectConfigure.class)
public @interface EnableMServerProtect {

}
```

因为现在微服务需要校验Zuul Token，所以我们需要在上一节定义的Feign拦截器里也加入Zuul Token，否则Feign调用微服务会报403异常。改造m-common模块下的`MOAuth2FeignConfigure`类：

```java
@Bean
    public RequestInterceptor oauth2FeignRequestInterceptor() {
        return requestTemplate -> {

            // 添加 Zuul Token
            String zuulToken = new String(Base64Utils.encode(MConstant.ZUUL_TOKEN_VALUE.getBytes()));
            requestTemplate.header(MConstant.ZUUL_TOKEN_HEADER, zuulToken);

            Object details = SecurityContextHolder.getContext().getAuthentication().getDetails();
            if (details instanceof OAuth2AuthenticationDetails) {
                String authorizationToken = ((OAuth2AuthenticationDetails) details).getTokenValue();
                requestTemplate.header(HttpHeaders.AUTHORIZATION, "Bearer " + authorizationToken);
            }
        };
    }
```

在m-server-system，m-server-test和m-auth模块的入口类上使用`@EnableMServerProtect`注解标注。

**功能整合**

到目前为止，m-server-system和m-server-test模块的入口类上注解已经有点多了，比如m-server-test模块的入口类：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableMAuthExceptionHandler
@EnableMServerProtect
public class MAuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(MAuthApplication.class, args);
    }

}

```

其中`@EnableMServerProtect`、`@EnableMOauth2FeignClient`和`@EnableMAuthExceptionHandler`是我们自定义的，它们的作用如下：

1. `@EnableMServerProtect`，开启微服务防护，避免客户端绕过网关直接请求微服务；
2. `@EnableMOauth2FeignClient`，开启带令牌的Feign请求，避免微服务内部调用出现401异常；
3. `@EnableMAuthExceptionHandler`，认证类型异常翻译。

这三个功能都是微服务提供者必备的功能，所以我们可以定义一个注解将这三个功能整合在一起。

因为这三个注解都是通过`@Enable`类型注解来将配置类注册到IOC容器中，所以我们现在要做的就是将这三个配置类一次性都注册到IOC容器中。在Spring中，要将多个类进行注册，可以使用selector的方式。

在m-common模块的com.work.common.selector下新建selector包，然后在该包下创建`MCloudApplicationSelector`

```java
public class MCloudApplicationSelector implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{
                MAuthExceptionConfigure.class.getName(),
                MOAuth2FeignConfigure.class.getName(),
                MServerProtectConfigure.class.getName()
        };
    }
}
```

通过`selectImports`方法，我们一次性导入了`MAuthExceptionConfigure`、`MOAuth2FeignConfigure`和`MServerProtectConfigure`这三个配置类。

要让上面的代码生效，我们还需将`MCloudApplicationSelector`注册到IOC容器中。同之前的做法，我们可以通过注解的方式来实现。

在m-common模块的com.work.common.annotation路径下新建`MCloudApplicationSelector`注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MCloudApplicationSelector.class)
public @interface MCloudApplication {

}
```

现在m-server-test入口类上的三个注解就可以使用`@MCloudApplication`代替。

### 2.5 跨域处理

前后分离架构下肯定会遇到跨域的问题，因为我们的请求都是通过微服务网关来转发的，所以我们可以在网关处统一处理跨域。

在m-gateway模块下的com.work.m.gateway.configure模块下新建`MGateWayCorsConfigure`配置类:

```java
@Configuration
public class MGateWayCorsConfigure {

    @Bean
    public CorsFilter corsFilter() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.setAllowCredentials(true);
        corsConfiguration.addAllowedHeader(CorsConfiguration.ALL);
        corsConfiguration.addAllowedOrigin(CorsConfiguration.ALL);
        corsConfiguration.addAllowedMethod(CorsConfiguration.ALL);
        source.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(source);
    }
}
```

该配置类里注册了`CorsFilter`:

- `setAllowCredentials(true)`表示允许cookie跨域；
- `addAllowedHeader(CorsConfiguration.ALL)`表示请求头部允许携带任何内容；
- `addAllowedOrigin(CorsConfiguration.ALL)`表示允许任何来源；
- `addAllowedMethod(CorsConfiguration.ALL)`表示允许任何HTTP方法。

## 3. 完善登录流程

### 3.1表结构设计

在权限系统中，最核心的三张表为：用户表、角色表和菜单表（权限表），它们间的关系通常采用经典的RBAC（**R**ole-**B**ased **A**ccess **C**ontrol，基于角色的访问控制）模型。简单来说就是一个用户拥有若干角色，每一个角色拥有若干权限。这样就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般都是多对多的关系，如下图所示：

![81.png](微服务搭建.assets/mpOPpR-1621168186745.png)

比如获取用户名为MrBird的用户权限过程为：

1. 通过MrBird的user_id从t_user_role表获取对应的role_id；
2. 通过第1步获取的role_id从t_role_menu表获取对应的menu_id；
3. 通过第2步获取的menu_id从t_menu获取menu相关信息（t_menu表的permission为权限信息）。

下面开始设计这5张表（三张主表，两张关联表）。

在开始之前，我们需要安装MySQL（已经安装过的略过），这里采用5.7版本 https://dev.mysql.com/downloads/file/?id=487685，下载后直接安装即可。

安装好MySQL后，使用数据库连接工具创建一个名称为febs_cloud_base的数据库，个人使用的是Navicat，所以这里以Navicat为例。点击Navicat的连接按钮，类型选MySQL，连接刚刚安装好的MySQL数据库:

![82.png](微服务搭建.assets/mpxYMn-1621168233578.png)

连接成功后，在新建的连接上右键选择新建数据库：

![83.png](微服务搭建.assets/mpxOdf-1621168247073.png)

至此，数据库创建完毕，下面开始创建数据表。

**t_user表结构设计**

t_user表为用户表，用于存储用户信息。表结构如下所示:

| 字段名          | 类型     | 长度 | null | 主键 | 注释               |
| :-------------- | :------- | :--- | :--- | :--- | :----------------- |
| USER_ID         | bigint   | 20   | 否   | 是   | 用户ID             |
| USERNAME        | varchar  | 50   | 否   | 否   | 用户名             |
| PASSWORD        | varchar  | 128  | 否   | 否   | 密码               |
| DEPT_ID         | bigint   | 20   | 是   | 否   | 部门ID             |
| EMAIL           | varchar  | 128  | 是   | 否   | 邮箱               |
| MOBILE          | varchar  | 20   | 是   | 否   | 联系电话           |
| STATUS          | char     | 1    | 否   | 否   | 状态 0 锁定 1 有效 |
| CREATE_TIME     | datetime | 0    | 否   | 否   | 创建时间           |
| MODIFY_TIME     | datetime | 0    | 是   | 否   | 修改时间           |
| LAST_LOGIN_TIME | datetime | 0    | 是   | 否   | 最近访问时间       |
| SSEX            | char     | 1    | 否   | 否   | 性别 0男 1女 2保密 |
| AVATAR          | varchar  | 100  | 否   | 是   | 头像               |
| DESCRIPTION     | varchar  | 100  | 是   | 否   | 描述               |

其中，因为SEX为MySQL关键字，所以使用SSEX代替了。

对应的建表语句为：

```sql
CREATE TABLE `t_user`  (
  `USER_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `USERNAME` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户名',
  `PASSWORD` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '密码',
  `DEPT_ID` bigint(20) NULL DEFAULT NULL COMMENT '部门ID',
  `EMAIL` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '邮箱',
  `MOBILE` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '联系电话',
  `STATUS` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '状态 0锁定 1有效',
  `CREATE_TIME` datetime(0) NOT NULL COMMENT '创建时间',
  `MODIFY_TIME` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  `LAST_LOGIN_TIME` datetime(0) NULL DEFAULT NULL COMMENT '最近访问时间',
  `SSEX` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '性别 0男 1女 2保密',
  `AVATAR` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '头像',
  `DESCRIPTION` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`USER_ID`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户表' ROW_FORMAT = Dynamic;
```

往表里插入一条数据：

```sql
INSERT INTO `t_user` VALUES (1, 'MrBird', '$2a$10$gzhiUb1ldc1Rf3lka4k/WOoFKKGPepHSzJxzcPSN5/65SzkMdc.SK', 1, 'mrbird@qq.com', '17788888888', '1', '2019-06-14 20:39:22', '2019-07-19 10:18:36', '2019-08-02 15:57:00', '0', 'default.jpg', '我是帅比作者。');
```

密码为1234qwer经过`BCryptPasswordEncoder`加密后的值。

**t_role角色表**

t_role表为角色表，用于存储角色信息。表结构如下所示:

| 字段名      | 类型     | 长度 | null | 主键 | 注释     |
| :---------- | :------- | :--- | :--- | :--- | :------- |
| ROLE_ID     | bigint   | 20   | 否   | 是   | 角色ID   |
| ROLE_NAME   | varchar  | 10   | 否   | 否   | 角色名称 |
| REMARK      | varchar  | 100  | 是   | 否   | 角色描述 |
| CREATE_TIME | datetime | 0    | 否   | 否   | 创建时间 |
| MODIFY_TIME | datetime | 0    | 是   | 否   | 修改时间 |

对应的建表语句为：

```sql
CREATE TABLE `t_role`  (
  `ROLE_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `ROLE_NAME` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色名称',
  `REMARK` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '角色描述',
  `CREATE_TIME` datetime(0) NOT NULL COMMENT '创建时间',
  `MODIFY_TIME` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`ROLE_ID`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '角色表' ROW_FORMAT = Dynamic;
```

往表里插入一条数据：

```sql
INSERT INTO `t_role` VALUES (1, '管理员', '管理员', '2019-08-08 16:23:11', '2019-08-09 14:38:59');
```

**t_menu表结构设计**

t_menu为菜单表，用于存储菜单、按钮及其对应的权限标识，表结构如下：

| 字段名      | 类型     | 长度 | null | 主键 | 注释                  |
| :---------- | :------- | :--- | :--- | :--- | :-------------------- |
| MENU_ID     | bigint   | 20   | 否   | 是   | 菜单/按钮ID           |
| PARENT_ID   | bigint   | 20   | 否   | 否   | 上级菜单ID            |
| MENU_NAME   | varchar  | 50   | 否   | 否   | 菜单/按钮名称         |
| PATH        | varchar  | 255  | 否   | 否   | 对应路由path          |
| COMPONENT   | varchar  | 255  | 否   | 否   | 对应路由组件component |
| PERMS       | varchar  | 50   | 是   | 否   | 权限标识              |
| ICON        | varchar  | 50   | 是   | 否   | 图标                  |
| TYPE        | char     | 2    | 否   | 否   | 类型 0菜单 1按钮      |
| ORDER_NUM   | double   | 20   | 是   | 否   | 排序                  |
| CREATE_TIME | datetime | 0    | 否   | 否   | 创建时间              |
| MODIFY_TIME | datetime | 0    | 否   | 否   | 修改时间              |

对应的建表语句为：

```sql
CREATE TABLE `t_menu`  (
  `MENU_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '菜单/按钮ID',
  `PARENT_ID` bigint(20) NOT NULL COMMENT '上级菜单ID',
  `MENU_NAME` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '菜单/按钮名称',
  `PATH` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '对应路由path',
  `COMPONENT` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '对应路由组件component',
  `PERMS` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '权限标识',
  `ICON` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '图标',
  `TYPE` char(2) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '类型 0菜单 1按钮',
  `ORDER_NUM` double(20, 0) NULL DEFAULT NULL COMMENT '排序',
  `CREATE_TIME` datetime(0) NOT NULL COMMENT '创建时间',
  `MODIFY_TIME` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`MENU_ID`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '菜单表' ROW_FORMAT = Dynamic;
```

插入几条数据：

```sql
INSERT INTO `t_menu` VALUES (1, 0, '系统管理', '/system', 'Layout', NULL, 'el-icon-set-up', '0', 1, '2017-12-27 16:39:07', '2019-07-20 16:19:04');
INSERT INTO `t_menu` VALUES (2, 1, '用户管理', '/system/user', 'febs/system/user/Index', 'user:view', '', '0', 1, '2017-12-27 16:47:13', '2019-01-22 06:45:55');
INSERT INTO `t_menu` VALUES (3, 2, '新增用户', '', '', 'user:add', NULL, '1', NULL, '2017-12-27 17:02:58', NULL);
INSERT INTO `t_menu` VALUES (4, 2, '修改用户', '', '', 'user:update', NULL, '1', NULL, '2017-12-27 17:04:07', NULL);
INSERT INTO `t_menu` VALUES (5, 2, '删除用户', '', '', 'user:delete', NULL, '1', NULL, '2017-12-27 17:04:58', NULL);
```

**t_user_role表结构设计**

t_user_role表用于关联用户和角色表，表结构如下所示：

| 字段名  | 类型   | 长度 | null | 主键 | 注释   |
| :------ | :----- | :--- | :--- | :--- | :----- |
| USER_ID | bigint | 20   | 否   | 否   | 用户ID |
| ROLE_ID | bigint | 20   | 否   | 否   | 角色ID |

对应的建表语句为：

```sql
CREATE TABLE `t_user_role`  (
  `USER_ID` bigint(20) NOT NULL COMMENT '用户ID',
  `ROLE_ID` bigint(20) NOT NULL COMMENT '角色ID'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户角色关联表' ROW_FORMAT = Dynamic;
```

插入一条数据：

```sql
INSERT INTO `t_user_role` VALUES (1, 1);
```

通过这条数据，user_id为1的用户和role_id为1的角色进行了关联，即MrBird用户角色为管理员。

**t_role_menu表结构设计**

t_role_menu用于关联角色和菜单表，表结构如下所示：

| 字段名  | 类型   | 长度 | null | 主键 | 注释        |
| :------ | :----- | :--- | :--- | :--- | :---------- |
| ROLE_ID | bigint | 20   | 否   | 否   | 角色ID      |
| MENU_ID | bigint | 20   | 否   | 否   | 菜单/按钮ID |

对应的建表语句为：

```sql
CREATE TABLE `t_role_menu`  (
  `ROLE_ID` bigint(20) NOT NULL,
  `MENU_ID` bigint(20) NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '角色菜单关联表' ROW_FORMAT = Dynamic;
```

插入一些数据:

```sql
INSERT INTO `t_role_menu` VALUES (1, 1);
INSERT INTO `t_role_menu` VALUES (1, 2);
INSERT INTO `t_role_menu` VALUES (1, 3);
INSERT INTO `t_role_menu` VALUES (1, 4);
INSERT INTO `t_role_menu` VALUES (1, 5);
```

通过这些数据，管理员角色已经拥有系统管理、用户管理、新增用户、修改用户和删除用户权限。

**t_dept表结构设计**

在t_user表中，有个dept_id字段，表示用户对应的部门。t_dept表用于存储部门信息，它本质上和RBAC没啥关系，一般用于控制数据权限。t_dept表结构如下所示:

| 字段名      | 类型     | 长度 | null | 主键 | 注释       |
| :---------- | :------- | :--- | :--- | :--- | :--------- |
| DEPT_ID     | bigint   | 20   | 否   | 是   | 部门ID     |
| PARENT_ID   | bigint   | 20   | 否   | 否   | 上级部门ID |
| DEPT_NAME   | varchar  | 100  | 否   | 否   | 部门名称   |
| ORDER_NUM   | double   | 20   | 是   | 否   | 排序       |
| CREATE_TIME | datetime | 0    | 否   | 否   | 创建时间   |
| MODIFY_TIME | datetime | 0    | 是   | 否   | 修改时间   |

对应的建表语句为：

```sql
CREATE TABLE `t_dept`  (
  `DEPT_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '部门ID',
  `PARENT_ID` bigint(20) NOT NULL COMMENT '上级部门ID',
  `DEPT_NAME` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '部门名称',
  `ORDER_NUM` double(20, 0) NULL DEFAULT NULL COMMENT '排序',
  `CREATE_TIME` datetime(0) NULL DEFAULT NULL COMMENT '创建时间',
  `MODIFY_TIME` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`DEPT_ID`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '部门表' ROW_FORMAT = Dynamic;
```

插入一条部门数据：

```sql
INSERT INTO `t_dept` VALUES (1, 0, '开发部', 1, '2018-01-04 15:42:26', '2019-01-05 21:08:27');
```

到这里，我们的库表就准备完了。

## 4. 整合swagger

## 5. 整合第三方服务

## 6. 微服务部署