####  spring cloud

* 服务注册与发现

* 配置中心

  ​	1.配置git保存配置

  ​	2.配置client可以通过`spring.application.name` 和 `spring.profiles.active`获取特定的配置文件

  ​	3.git仓库添加的配置加密

  * 1.对称加密/非对称加密，需要配置加密key，手动加密git保存的配置
  * 2.HashiCorp Vault

  ​    4.刷新机制

  *  client调用刷新接口,通过spring actuator

  * git commit hook通过spring bus 通知config server和config client 

    ![image-20210303151859075](https://gitee.com/zzz_123456/picgo/main/20210303151911.png)

    1.github/gitlab配置webhook 接口为config server的/monitor

    2.config server 添加monitor依赖后能处理/monitor

    3.添加 spring cloud stream kafka/rabbit，config server将接收的webhook，发送到消息borker

    4.config client进行消费，config client 添加依赖spring-cloud-starter-bus-amqp/spring-cloud-starter-bus-kafka

* 服务发现负载均衡

* 熔断器/断路器Hystrix

  处理请求失败和超时，避免让失败在分布式调用堆栈中层叠

  请求失败，调用fallback 和 自我修正

  打开的断路器偶尔会进入 half-open状态，再次调用方法，失败进入打开状态，成功进入关闭状态

  追踪方法失败比例，超过threshold时，所有调用该方法的请求全部调用Fallback

