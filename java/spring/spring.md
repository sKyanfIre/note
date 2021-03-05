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

