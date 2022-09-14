# Durid连接池配置

Spring Boot 2.0 以上默认使用 **Hikari** 数据源，Druid是阿里巴巴开源平台上一个数据库连接池实现，Druid可以很好的监控 DB 池连接，SQL 的执行情况，接口调用情况以及web应用。
Github地址：https://github.com/alibaba/momp/
整合过程:

> 导入起步依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.13</version>
</dependency>
```

> 编写配置文件进行配置(可将momp替换为dydb配置不同数据源)

```properties
spring.datasource.momp.url=
spring.datasource.momp.username=
spring.datasource.momp.password=
spring.datasource.momp.driverClassName=
#默认是Hikari连接池，切换为Druid
spring.datasource.momp.type= com.alibaba.druid.pool.DruidDataSource
# 初始化时建立物理连接的个数
spring.datasource.momp.initial-size=5
# 最大连接池数量
spring.datasource.momp.max-active=30
# 最小连接池数量
spring.datasource.momp.min-idle=5
# 获取连接时最大等待时间，单位毫秒
spring.datasource.momp.max-wait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.momp.time-between-eviction-runs-millis=60000
# 连接保持空闲而不被驱逐的最小时间
spring.datasource.momp.min-evictable-idle-time-millis=300000
# 用来检测连接是否有效的sql，要求是一个查询语句
spring.datasource.momp.validation-query=SELECT 1 FROM DUAL
# 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
spring.datasource.momp.test-while-idle=true
# 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
spring.datasource.momp.test-on-borrow=false
# 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
spri
```

> 读取配置文件

```java
@Bean(name = "mommpDataSource")
        @ConfigurationProperties(prefix = "spring.datasource.mommp")
        @Primary
        public DataSource dataSource() {
                return DruidDataSourceBuilder.create().build();
        }
```

- 读取的配置文件前缀是spring.datasource.mommp
- Primary标记为主数据源

- DataSourceBuilder需要druid依赖

```xml
<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.13</version>
</dependency>
```

> 配置Druid数据源监控

```java
/**
     	* 配置 Druid 监控界面
     	*/
   	 @Bean
   	 public ServletRegistrationBean statViewServlet(){
        	ServletRegistrationBean srb =
            new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
        	//设置控制台管理用户
        	srb.addInitParameter("loginUsername","admin");
        	srb.addInitParameter("loginPassword","admin");
        	//是否可以重置数据(页面上的重置按钮)
        	srb.addInitParameter("resetEnable","false");
        	return srb;
   	}
	配置一个web过滤器，排除一些静态资源
	@Bean
    	public FilterRegistrationBean statFilter(){
        	//创建过滤器
       	FilterRegistrationBean frb =
           	new FilterRegistrationBean(new WebStatFilter());
        	//设置过滤器过滤路径
        	frb.addUrlPatterns("/*");
        	//忽略过滤的形式
        	frb.addInitParameter("exclusions",
             "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        	return frb;
    	}

```

在地址栏中输入ip+端口/druid/login.html访问内置监控界面
==常见问题:==
如果filter类型一栏为空是无法监控到sql的，可能是配置文件中的filters没有配置成功。

