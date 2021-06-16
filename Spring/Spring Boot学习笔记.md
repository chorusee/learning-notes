### Spring Boot学习笔记

#### 自动配置原理

主类上有一个`@SpringBootApplication`注解，这个注解上面有一个`@EnableAutoConfiguration`注解，这个注解中用`@Import`注解导入了`AutoConfigurationImportSelector`类，这个类实现了`ImportSelector`接口，`selectImports`方法从`spring.factories`文件中获取所有候选的需要自动配置的配置类名，返回一个String数组。具体是否需要自动配置，还要看配置类。

所有的自动配置类都在`org.spirng.boot.autoconfigure`包中。

如`Redis`的自动配置类上有一个注解：

```java
@ConditionalOnClass(RedisOperations.class)
```

即表明当导入了`spring-boot-starter-data-redis`之后，`RedisOperations`类存在，才会执行自动配置。

Spring Boot通过一系列的`@ConditionalXx`注解：

> **`@ConditionalOnBean`**：当容器里有指定Bean的条件下
> **`@ConditionalOnClass`**：当类路径下有指定的类的条件下
> `@ConditionalOnExpression`：基于`SpEL`表达式为true的时候作为判断条件才去实例化
> `@ConditionalOnJava`：基于JVM版本作为判断条件
> `@ConditionalOnJndi`：在JNDI存在的条件下查找指定的位置
> `@ConditionalOnMissingBean`：当容器里没有指定Bean的情况下
> `@ConditionalOnMissingClass`：当容器里没有指定类的情况下
> `@ConditionalOnWebApplication`：当前项目时Web项目的条件下
> `@ConditionalOnNotWebApplication`：当前项目不是Web项目的条件下
> **`@ConditionalOnProperty`**：指定的属性是否有指定的值
> `@ConditionalOnResource`：类路径是否有指定的值
> `@ConditionalOnOnSingleCandidate`：当指定Bean在容器中只有一个，或者有多个但是指定首选的Bean


来判断是否需要自动配置。


> `@Import`注解有几个用法：
>
> 1. 在配置类上加上`@Import({Test.class})`，处理配置类时会顺便把`Test`注册成Bean。
> 2. 在配置类上加上`@Import({AppConfigAux.class})`，其中`AppConfigAux`中有很多`@Bean`注解申明的Bean，Spring会把`ApplicationAux`类注册为Bean且将类内的Bean也注册。
> 3. 在配置类上加上`@Import({MyImportSelector.class})`，其中`MyImportSelector`类实现了`ImportSelector`接口，其中有一个`selectImports`方法。这个方法会返回需注册到Spring容器中的类的名字，Spring会将这些名字的类注册到容器里。

#### `SpringApplication.run()`方法详解

1. 调用静态的run方法之后做的第一件事是new一个`SpringApplication`实例，然后调用其run方法。

   在这个实例初始化的时候，它会提前做几件事：

   1）根据`classpath`下是否存在某些特征类判断是否应该创建一个Web应用使用的`ApplicationContext`还是普通的`ApplicationContext`；

   2）加载所有可用的`ApplicationContextInitializer`；

   3）加载所有可用的`ApplicationListener`；

   4）推断并设置main方法的定义类。

2. `SpringApplication`实例初始化完成后，就开始执行run方法的逻辑：

   1）创建并配置当前Spring Boot应用将要使用的`Environment`，包括配置要使用的`PropertySource`以及`Profile`；

   2）根据用户是否明确设置了`ApplicationContextClass`类型以及初始化阶段的推断结果，决定该为当前Spring Boot应用创建什么类型的`ApplicationContext`并创建完成。将之前准备好的Environment设置给创建好的`ApplicationContext`使用；

   3）调用所有可用的`ApplicationContextInitializer`的`initialize()`方法来对创建好的`ApplicationContext`进行进一步处理；

   4）将之前通过`@EnableAutoCOnfiguration`获取的所有配置类加载到已经准备完毕的`ApplicationContext`中；

   5）调用`ApplicationContext`的`refresh()`方法，`refresh()`方法是`ApplicationContext`的精髓，IOC、AOP都是在这个方法中完成。

   ![springboot启动步骤](C:\Workspace\笔记\Spring\res\springboot启动步骤.png)

   
### 配置文件属性注入

#### YAML

YAML支持三种数据：字符串、对象（Map）、数组

```yaml
# 字符串默认不用加上引号
name: Zhangsan
# 加上双引号不会转义特殊字符
name: "Zhangsan \n Lisi"  # 输出：Zhangsan 换行 Lisi
# 加上单引号会转义特殊字符
name: 'Zhangsan \n Lisi'  # 输出：Zhangsan \n Lisi

# 对象，或者键值对
friend:
  name: Zhangsan
  age: 18
# 行内写法
friend: {name: Zhangsan, age: 18}

# 数组
# 用 - 表示数组中的一个元素
pets:
  - cat
  - dig
  - pig
# 行内写法
pets: [cat, dog, pig]
```

#### 配置文件值注入

1. 通过定义一个配置类

   ```java
   @Data
   @ToString
   @ConfigurationProperties(prefix = "person")
   @Component
   public class Person {
       private String name;
       private Integer age;
   
       private Map<String, Object> map;
       private List<Object> list;
       private Dog dog;
   }
   ```

   ```yaml
   person:
     name: Zhangsan
     age: 18
   
     map:
       key1: value1
       key2: value2
   
     list:
       - elem1
       - elem2
   
     dog:
       name: Poppy
       age: 3
   ```

2. 直接通过`@Value`注解

   ```java
   @Data
   @ToString
   @Component
   public class Person2 {
       // @Value只能注入字符串
       @Value("${person.name}")
       private String name;
       @Value("${person.age}")
       private Integer age;
   }
   ```

通过配置类的方式，可以使用`@Validate`注解进行校验，而使用`@Value`不支持校验。

#### 配置文件加载位置

Spring Boot会扫描以下位置的`application.properties`或`application.yml`文件：

- `file:./config/`：jar包所在目录下的config目录
- `file:./`：jar包的同级目录
- `classpath:/config/`
- `classpath:/`

优先级由高到低，高优先级的配置会覆盖低优先级的配置。

Spring Boot也可以从以下位置加载配置，优先级从高到低：

+ 命令行参数

  如：`java -jar spring-boot-test-0.0.1-SNAPSHOT.jar --server.port=8000`

+ Java系统属性（`System.getProperties()`）

+ 操作系统环境变量

#### Spring Boot加载默认配置

每个自动配置类上都由一个注解`@EnableConfigurationProperties(XxProperties.class)`，如：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
    /* 省略 */
}
```

这个`XxProperties.class`类里面就是Spring帮我们写好的默认配置属性，如`Redis`的默认配置：

```java
@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {

	private int database = 0;

	private String url;

	private String host = "localhost";

	private String username;

	private String password;

	private int port = 6379;

	private boolean ssl;

	private Duration timeout;
    
    /* 省略 */
}
```

可以在我们的配置文件中通过`spring.redis.xx=yy`的方式覆盖默认属性。