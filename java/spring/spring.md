### Spring

spring application context

DI

@Configuration @Bean

autowiring

component scanning

autoconfiguration



spring initializer

spring tool suite



@SpringBootApplication

@RunWith

@WebMvcTest



interface  

* WebMvcConfigurer





#### 配置

* 元数据

  resource/META-INF/additional-spring-configuration-metadata.json

  ```json
  {
    "properties": [
      {
        "name": "secure.white.url",
        "type": "java.lang.String",
        "description": "Description for secure.white.url."
    }
  ] }
  ```

  

* spring.profiles.active

* @ConfigurationProperites

* @Profile

* ${}引用其他配置



#### Spring Security

oauth服务 

1.登录接口 
根据不同granttype校验登录信息
	例如校验账号密码
	生成jwt令牌，令牌通过公钥签名，返回jwt
2.刷新接口
	jwt令牌解析后，里面有自定义信息和默认的信息如jwt有效期和有效

####  事务

>  特性:

* 原子性 Atomic
* 一致性 Consistent
* 隔离性 Isolated
* 持久性 Durable 

> 事务属性



![image-20210310205341011](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210310205341011.png)

`TransactionDefinition`中常量

* 传播行为 

  PROPAGATION_MANDATORY 强制执行事务，无事务则抛异常

  PROPAGATION_REQUIRED 无事务则创建新事务

  PROPAGATION_REQUIRES_NEW 创建新事务

  PROPAGATION_NESTED 事务存在则创建嵌套事务

  PROPAGATION_SUPPORTS 支持事务

  PROPAGATION_NOT_SUPPORTED 不支持事务

  PROPAGATION_NEVER 事务存在则抛异常

  

  

* 隔离级别

* 回滚规则

* 事务超时

* 是否只读

> 核心组件

*  事务管理器

  事务管理器将事务功能委托给特定的平台

  * jdbc.datasource.DataSourceTransactionManager
  * jms.connection.JmsTransactionManager
  * orm.jpa.JpaTransactionManager