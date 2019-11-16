---
title: Spring Boot & Spring Boot Netflix   
date: 2019-11-10 10:54:06  
tags:  
- 微服务  
- Spring Cloud
- Spring Boot
- Spring Netflix 
---

### 概述  

### Demo
> 具体项目放在GitHub仓库：https://github.com/Tooi6/demo-spring-cloud

#### 统一依赖管理（demo-dependencies）
> 所有项目都会依赖这个项目，配置一些项目需要用到的插件、下载依赖时的第三方库  

#### demo-eureka（服务注册与发现）
>  Spring Cloud Netflix 的 Eureka，Eureka 是一个服务注册和发现模块  

```
<dependencies>
    <!-- Spring Boot Begin -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- Spring Boot End -->

    <!-- Spring Cloud Begin -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <!-- Spring Cloud End -->
</dependencies>

# application.yml 
spring:
  application:
    name: demo-tooi-eureka

server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 服务提供者（demo-service-admin）
> Eureka Client 注册到Eureka Service，提供服务  

```
<dependencies>
    <!-- Spring Boot Begin -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- Spring Boot End -->

    <!-- Spring Cloud Begin -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <!-- Spring Cloud End -->
</dependencies>

# application.yml
spring:
  application:
    name: demo-service-admin

server:
  port: 8763

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

#### 服务消费者Ribbon（demo-web-admin-ribbon）
> 在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于 http restful 的。Spring cloud 有两种服务调用方式，一种是 ribbon + restTemplate，另一种是 feign

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    </dependency>
</dependencies>

# 调用服务
@Service
public class AdminService {
    @Autowired
    RestTemplate restTemplate;

    public String sayHi(String message) {
        return restTemplate.getForObject("http://DEMO-SERVICE-ADMIN/hi?message=" + message, String.class);
    }
}
```

#### 服务消费者（Feign）  
> Feign 默认集成了 Ribbon，并和 Eureka 结合，默认实现了负载均衡的效果  

```
# pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

# 调用服务  
@FeignClient(value = "DEMO-SERVICE-ADMIN")
public interface AdminService {
    @GetMapping("hi")
    String sayHi(@RequestParam("message") String message);
}
```

#### 熔断器（Hystrix）
> 当对特定的服务的调用的不可用达到一个阀值（Hystrix 是 5 秒 20 次） 熔断器将会被打开。熔断器打开后，为了避免连锁故障，通过 **fallback** 方法可以直接返回一个固定值。  

- **Ribbon中使用**

```
# 添加依赖  
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

# 在 Application 中增加 @EnableHystrix 注解  

# 增加 fallbackMethod 熔断方法
@HystrixCommand(fallbackMethod = "hiError")
public String sayHi(String message) {
    return restTemplate.getForObject("http://DEMO-SERVICE-ADMIN/hi?message=" + message, String.class);
}

public String hiError(String message) {
    return String.format("message：%s , request error.",message);
}
```

- **Feign中使用**  

```
# 添加依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

# 配置 application.yml
feign:
  hystrix:
    enabled: true

# 在 service 中指定 fallback 类
@FeignClient(value = "DEMO-SERVICE-ADMIN",fallback = AdminServiceHystrix.class)
public interface AdminService {
    @GetMapping("hi")
    String sayHi(@RequestParam("message") String message);
}

# 创建并实现对应的Feign接口  
@Component
public class AdminServiceHystrix implements AdminService {
    @Override
    public String sayHi(String message) {
        return String.format("message：%s，request error",message);
    }
}
```

#### Hystrix 仪表盘 （Dashboard）
> 

- **Ribbon 和 Feign 改造方式相同** 

```
# 添加依赖  
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

# 在 Application 中添加 @EnableHystrixDashboard 注解

# 添加类  
@Configuration
public class HystrixDashboardConfiguration {
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```

#### 路由网关（demo-zuul）  
> Zuul 的主要功能是**路由转发和过滤器**。路由功能是微服务的一部分。Zuul 默认和 Ribbon 结合实现了**负载均衡**的功能


```
# pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>

# 在application 开启 @EnableZuulProxy 注解  

# application.yml  
spring:
  application:
    name: demo-zuul

server:
  port: 8769

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

zuul:
  routes:
    api-a:
      path: /api/a/**
      serviceId: demo-web-admin-ribbon
    api-b:
      path: /api/b/**
      serviceId: demo-web-admin-feign
```

- **配置网关路由失败时的回调**

```
# 添加类
@Component
public class WebAdminFeignFallbackProvider implements FallbackProvider {

    @Override
    public String getRoute() {
        // // ServiceId，如果需要所有调用都支持回退，则 return "*" 或 return null
        return "demo-web-admin-feign";
    }

    /**
     * 如果请求失败，则返回指定信息给调用者
     *
     * @param route
     * @param cause
     * @return
     */
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            /**
             * 网关向 api 服务请求失败了，但是消费者客户端向网关发起的请求是成功的，
             * 不应该把 api 的 404,500 等问题抛给客户端
             * 网关和 api 服务集群对于客户端来说是黑盒
             * @return
             * @throws IOException
             */
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.getReasonPhrase();
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                ObjectMapper objectMapper = new ObjectMapper();
                Map<String, Object> map = new HashMap<>();
                map.put("status", 200);
                map.put("message", "无法连接，请检查您的网络");
                return new ByteArrayInputStream(objectMapper.writeValueAsString(map).getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                // 和 getBody 中的内容编码一致
                headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
                return headers;
            }
        };
    }
}
```

- **使用路由网关的服务过滤功能**  

```
# 添加过滤器类  
public class Filter extends ZuulFilter{
    // 详细看源代码...
}
```

#### 分布式配置中心
> 在分布式系统中，由于服务数量巨多，为了方便服务配置**文件统一管理，实时更新**，所以需要分布式配置中心组件。  
在 Spring Cloud Config 组件中，分两个角色，一是 Config Server，二是 Config Client。 

> 新建git仓库：https://github.com/Tooi6/demo-remote-config.git


- **服务端（demo-config）**

```
# pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>

# application.yml
spring:
  application:
    name: demo-config
  cloud:
    config:
      label: master
      server:
        git:
          uri: https://github.com/Tooi6/demo-remote-config.git
          search-paths: respo
          username: youemail@emai.com
          password: youpassword
server:
  port: 8888
  
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
# 在 application 开启配置 @EnableConfigServer
```

- **客户端（改造Feign项目）**

```
# pom.xml 添加依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

# application.yml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      name: web-admin-feign
      label: master
      profile: dev
```

#### 服务链路追踪 （zipkin）
> ZipKin 是一个开放源代码的**分布式跟踪系统**，由 Twitter 公司开源，它致力于收集服务的定时数据，**以解决微服务架构中的延迟问题，包括数据的收集、存储、查找和展现。**

```
# pom.xml 添加依赖
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-server</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-ui</artifactId>
</dependency>

# 在 application 启动注解 @EnableZipkinServer

# application.yml
spring:
  application:
    name: demo-zipkin

server:
  port: 9411

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

management:
  metrics:
    web:
      server:
        auto-time-requests: false

# 给所有其他需要追踪的服务添加依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### Spring Boot Admin
> Spring Boot Admin是一个开源社区项目，用于**管理和监控**SpringBoot应用程序。 

- **Spring Boot Admin 服务端（demo-admin）**

```
# pom.xml
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>

# 在 application 类中添加 @EnableAdminServer 注解  

# application.yml
spring:
  application:
    name: hello-spring-cloud-admin
  zipkin:
    base-url: http://localhost:9411

server:
  port: 8084

management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: health,info

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

- **客户端（改造其他项目）** 

```
# pom.xml 添加依赖
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
</dependency>

# application.yml 添加配置  
spring:
  boot:
    admin:
      client:
        url: http://localhost:8084
```

