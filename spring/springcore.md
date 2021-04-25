### springcore

* DI

* EVENTS

* RESOURCES

* AOP

  #### 重要组件

  * BeanFactory

  * ApplicationContext

    * 提供简单注入
    * 事件发布
    * 特定的上下文 例如：WebApplicationContext

  * 注入方式

    anno注入在xml注入之前执行

    xml中<context:annotation-config>隐式注册post-processor,只会查找同一个上下文中使用注解注入的bean

    ​		ConfigurationClassPostProcessor

    ​		AutowiredAnnotationBeanPostProcessor

    ​		CommonAnnotationBeanPostProcessor

    ​		PersistenceAnnotationBeanPostProcessor

    ​		EventListenerMethodProcessor

    * xml

      * 运行时可以动态修改bean

    * anno

      注解与BeanPostProcessor结合使用

      * @Require 强制注入bean,RequiredAnnotationBeanPostProcessor必须被注入用来支持@Require

      * @Autowired JSR330's @Inject可以代替@Autowired
        1. 通过构造函数进行注入，如果只有一个构造函数可以不需要该注解（4.3版本）
        2. 通过方法注入，支持多参数注入
        3. 通过字段注入，还可以结合构造函数注入方式一起使用
        4. 支持数组或者集合注入，可以通过bean implement org.springframework.core.Ordered或者@Order或者@Priority指定bean在集合中的顺序，@Priority无法作用于@Bean上
        5. 支持map注入，Map<String,Object> key为string类型，与bean名字类似
        6. 数组，集合，map必须至少有一个bean存在
        7. 通过构造函数注入时，如果有多个构造函数，可以设置required=fasle，作为候选的构造函数
        8. 通过注入Optional<Object> 或者@Nullable实现和@Require相同功能