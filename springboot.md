#                            springboot

### spring boot配置

@SpringBootApplication包括

1. @Configuration：配置类上标注这个注解
2. @EnableAutoConfiguration：开启自动给配置功能
3. @ComponentScan：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
~~~



#### 注入值

~~~java
@value//可以将值单个注入到相应的属性上

@ConfigurationPropertires(profix="xxxx")// 可以将properties的值撇两注入
@PropertiesSources//确定引入properties文件的路径
~~~

|                | @value | @ConfigurationPropertires |
| -------------- | ------ | ------------------------- |
| 松散语法       | 支持   | 不支持                    |
| spel           | 不支持 | 支持                      |
| jsr303数据校验 | 支持   | 不支持                    |
| 复杂类型的封装 | 支持   | 不支持                    |

~~~java
@Email//表示注入的值必须是邮箱
//@Value("${person.last‐name}")
private String lastName;
~~~

@propertiesSources：加载指定的配置未见

@ImportSources:导入spring配置文件让配置文件中的属性生效

#### progfile

多个配置文件可以用profile配置  application-xx.properties或者application-xx.yml

###### 激活配置文件

- 命令行：java -jar spring-boot-02-conﬁg-0.0.1-SNAPSHOT.jar --spring.proﬁles.active=dev；可以在测试的时候直接指定那个配置文件生效
- 虚拟机：-Dspring.proﬁles.active=dev
- 配置文件：在配置文件中指定 spring.proﬁles.active=dev

###### 配置文件加载顺序

~~~~Java、
–ﬁle:./conﬁg/

–ﬁle:./

–classpath:/conﬁg/

–classpath:/
//优先级从高到底
~~~~

springboot会从四个位置加载文件，互补配置，

我们还可以通过外部命令的方式改变默认配置文件的位置

java -jar spring-boot-02-conﬁg-02-0.0.1-SNAPSHOT.jar --spring.conﬁg.location=G:/application.properties

###### 外部配置文件加载顺序

- 命令行参数
  所有的配置都可以在命令行上进行指定
  java -jar spring-boot-02-conﬁg-02-0.0.1-SNAPSHOT.jar --server.port=8087 --server.context-path=/abc
  多个配置用空格分开； --配置项=值
- 来 自 java:comp/env 的 JNDI 属 性
- Java系统属性（System.getProperties()）
- 操作系统环境变量
- RandomValuePropertySource配置的random.*属性值

~~~java
spring.profiles.active=dev//确定那个配置文件生效（直接卸载配置文件中）
~~~

~~~yaml
server:
   port: 8080
---
server:
   port:8090
spring:
   profiles: active//确定哪个文件激活
---
~~~

​	其中---表示一块区域，相当于一个文件

yaml（yml）：k:(空格)v：表示一对键值对（空格必须有）；

​		以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

~~~yaml
person:
	name: yxp //:后面需要有一个空格
~~~

“”：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思name: “zhangsan \n lisi”：输出；zhangsan 换行 lisi

‘’：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据name: ‘zhangsan \n lisi’：输出；zhangsan \n lisi

#### 自动配置原理

spring boot启动的时候加载主配置类，开启了自动配置功能@EnableConfiguration	

@EnableConfiguration的作用

- 利用EnableAutoConﬁgurationImportSelector给容器中导入一些组件？
- 可以查看selectImports()方法的内容
- 获取候选的配置
- SpringFactoriesLoader.loadFactoryNames()
- 扫描所有jar包类路径下 META‐INF/spring.factories
  把扫描到的这些文件的内容包装成properties对象
  从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中

每一个自动配置类进行自动配置

以HttpEncodingAutoConfiguration为例解释自动配置类的原理

~~~~java
@Configuration //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class) //启动指定类的
ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把
HttpEncodingProperties加入到ioc容器中
@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果
满足指定的条件，整个配置类里面的配置就会生效； 判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有这个类
CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing =
true) //判断配置文件中是否存在某个配置 spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
//他已经和SpringBoot的配置文件映射了
private final HttpEncodingProperties properties;
    //只有一个有参构造器的情况下，参数的值就会从容器中拿
public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
this.properties = properties;
}
@Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取
@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
public CharacterEncodingFilter characterEncodingFilter() {
CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
filter.setEncoding(this.properties.getCharset().name());
filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
return filter;
}
~~~~

根据当前不同的条件判断，决定这个配置类是否生效？

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取 的，这些类里面的每一个属性又是和配置文件绑定的；

所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘；配置文件能配置什么就可以参照某个功 能对应的这个属性类

~~~java
@ConfigurationProperties(prefix = "spring.http.encoding") //从配置文件中获取指定的值和bean的属
性进行绑定
public class HttpEncodingProperties {
public static final Charset DEFAULT_CHARSET = Charset.forName("UTF‐8");
~~~

