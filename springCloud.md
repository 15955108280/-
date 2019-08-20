# springCloud

### 1.微服务架构

##### 什么是微服务

**集群**：通过一组计算机软件和硬件连接起来高度紧密的写作完成计算机工作

- 高性能：拖过堕胎计算机完成同一共组，分摊压力，达到更高的效率
- 高可用：两机或多机工作内容，过程完全一致，可以相互顶替

**分布式**：一组计算机通过网络相互连接传递消息与通信并协调它们之间的行为而形成的系统。组件之间彼此进行交互以实现同一目标

- 低耦合：模块之间相互独立，便于扩展提高利用率
- 高吞吐：功能差分，分散到不同的模块进行

集群和分布式并不冲突

CAP：理论强一致性（consistency），极致可用性（availability），分区容错性（partition-tolerance）

##### 什么是springcloud

springcloud提供了分布式系统的一套快速结局方案

springcloud提供了分布式系统的一些常见模块的组件

##### spring cloud常用组件

服务治理：spring cloud Eureka

负载均衡： Spring Cloud Ribbon

熔断限流： Spring Cloud Hystrix

服务调用： Spring Cloud Feign

网关服务： Spring Cloud Zuul

配置中心： Spring Cloud Config

消息总线： Spring Cloud Bus

消息驱动： Spring Cloud Stream

服务追踪： Spring Cloud Sleuth

### 2.SpringCloud网关

##### 网关应用场景

同一外部入口，请求路由，认证授权，请求限流，请求日志和监控

##### Spring Cloud Zuul

- 服务路由

- 自定义过滤器Zuulfilter

  ​		filterType():过滤类型

  ​		filterOrder():过滤肾虚

  ​		shouldFilter():是否需要过滤

  ​		Run():过滤逻辑

1. 外部请求:arrow_right:zuul:arrow_right:(pre)选择路由:arrow_right:(routing)请求服务:arrow_right:(post)zuul

pre:字路由请求之前

routing：在请求路由之后

post：在请求路由到服务之后执行

error：早其他阶段发生错误的时候执行

###### 自定义zuul过滤器

~~~java
@EnableZuulProxy//启用zuul代理
@SpringBootApplication
public class SpringcloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudApplication.class, args);
    } 

}
~~~

~~~java
 @Override
    public Object run(){
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        String access_token = request.getParameter("access_token");
        if (Objects.equals(access_token,MuFilter.accessToken)){
            requestContext.setResponseStatusCode(HttpStatus.OK.value());
            requestContext.setResponseBody("yishouquan");
            requestContext.setSendZuulResponse(false);
        }else {
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
            requestContext.setResponseBody(HttpStatus.UNAUTHORIZED.getReasonPhrase());
            requestContext.setSendZuulResponse(false);
        }
        return null;
    }
~~~

### 3. Spring Cloud Eureka(服务注册与发现)

##### 3.1创建服务于注册中心

~~~java
@EnableEurekaServer//用来指定该项目为Eureka的服务注册中心
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

}
~~~

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在`application.properties`中问增加如下配置：

~~~java
server.port=1111

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
~~~

##### 创建服务提供方

~~~java
@RestController
public class ComputeController {

    private final Logger logger = Logger.getLogger(getClass());

    @Autowired
    private DiscoveryClient client;

    @RequestMapping(value = "/add" ,method = RequestMethod.GET)
    public Integer add(@RequestParam Integer a, @RequestParam Integer b) {
        ServiceInstance instance = client.getLocalServiceInstance();
        Integer r = a + b;
        logger.info("/add, host:" + instance.getHost() + ", service_id:" + instance.getServiceId() + ", result:" + r);
        return r;
    }

}
~~~

最后在主类中通过加上`@EnableDiscoveryClient`注解，该注解能激活Eureka中的`DiscoveryClient`实现，才能实现Controller中对服务信息的输出。

~~~java
@EnableDiscoveryClient
@SpringBootApplication
public class ComputeServiceApplication {

	public static void main(String[] args) {
		new SpringApplicationBuilder(ComputeServiceApplication.class).web(true).run(args);
	}

}
~~~

我们在完成了服务内容的实现之后，再继续对`application.properties`做一些配置工作，具体如下：

~~~~properties
spring.application.name=compute-service//指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务

server.port=2222

eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/ //属性对应服务中心配置的内容，指定服务注册中心

~~~~

### 4.Ribbon（负载均衡器）

Ribbon可以在通过客户端中配置的ribbonServerList服务端列表去轮询访问以达到均衡负载的作用

当Ribbon与Eureka联合使用时，ribbonServerList会被DiscoveryEnabledNIWSServerList重写，扩展成从Eureka注册中心中获取服务端列表。同时它也会用NIWSDiscoveryPing来取代IPing，它将职责委托给Eureka来确定服务端是否已经启动。

~~~java
@EnableDiscoveryClient
@SpringBootApplication
public class SpringcloudribbonApplication {
    @Bean
    @LoadBalanced//开启客户端负载均衡
    RestTemplate restTemplate(){
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudribbonApplication.class, args);
    }

}

~~~

在新的controller中访问客户端service方法

~~~java
@RestController
public class ConsumerController {

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }

}
~~~

#### 4.1 feign

~~~xml
 <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
 </dependency>
~~~

~~~java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients//注解开启Feign功能
public class FeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(FeignApplication.class, args);
	}

}
~~~

~~~java
@FeignClient("compute-service")//绑定接口对应的服务
public interface ComputeClient {

    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);

}
~~~

~~~java
@RestController
public class ConsumerController {

    @Autowired
    ComputeClient computeClient;//上方的接口

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add() {
        return computeClient.add(10, 20);
    }

}
~~~

### 5. Hystrix（断路器）

###### 5.1 ribbon中使用

在Spring Cloud中使用了Hystrix来实现断路器的功能。Hystrix是Netflix开源的微服务框架套件之一，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。

- pom引入依赖

~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
~~~

- 在eureka-ribbon的主类`RibbonApplication`中使用`@EnableCircuitBreaker`注解开启断路器功能：

~~~java
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker//开启断路器功能
public class RibbonApplication {

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}

}
~~~

- 改造原来的服务消费方式，新增`ComputeService`类，在使用ribbon消费服务的函数上增加`@HystrixCommand`注解来指定回调方法。

~~~~java
@Service
public class ComputeService {

    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "addServiceFallback")
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }

    public String addServiceFallback() {
        return "error";
    }

}
~~~~

- 提供rest接口的Controller改为调用ComputeService的addService

~~~java
@RestController
public class ConsumerController {

    @Autowired
    private ComputeService computeService;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return computeService.addService();
    }

}
~~~

###### 5.2 feign中使用

- @FeignClient注解中加入fallback=（新建的指定回调类）

~~~java
@FeignClient(value = "compute-service", fallback = ComputeClientHystrix.class)
public interface ComputeClient {

}
~~~

- 创建回调类`ComputeClientHystrix`，实现`@FeignClient`的接口，此时实现的方法就是对应`@FeignClient`接口中映射的fallback函数

~~~java
@Component
public class ComputeClientHystrix implements ComputeClient {

    @Override
    public Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b) {
        return -9999;
    }

}
~~~

### 6. config

- pom.xml中引入`spring-cloud-config-server`依赖，完整依赖配置如下

~~~xml
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
</dependency>
~~~

- 创建Spring Boot的程序主类，并添加`@EnableConfigServer`注解，开启Config Server

~~~java
@EnableConfigServer
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

}
~~~

- `application.properties`中配置服务信息以及git信息，例如

~~~properties
spring.application.name=config-server
server.port=7001

# git管理配置
#git仓库地址
spring.cloud.config.server.git.uri=http://git.oschina.net/didispace/SpringBoot-Learning/
#仓库路径下的相对搜索位置，可以配置多个
spring.cloud.config.server.git.searchPaths=Chapter9-1-4/config-repo

spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
~~~



