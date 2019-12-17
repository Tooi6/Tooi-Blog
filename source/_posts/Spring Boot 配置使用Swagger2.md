---
title: Spring Boot 配置使用Swagger2    
date: 2019-12-1 20:55:55  
tags:  
- Swagger2  
---

### 概述  
> swagger2 是一个规范和完整的框架，用于生成、描述、调用和可视化Restful风格的web服务,缺点是代码植入性较强。  

### Spring Boot 配置Swagger2  

- **添加依赖**  

```
<!-- Swagger2 Begin -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
<!-- Swagger2 End -->
```

- **配置Swagger2**  
> 新建 Swagger2Config 配置类
```
@Configuration
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.tooi.icoin.service.posts.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("iCoin 文章服务文档")
                .description("icoin-service-posts")
                .termsOfServiceUrl("https://tooi6.github.io/")
                .version("1.0.0")
                .build();
    }
}
```

- **启用 Swagger2**  
> Spring boot启动类添加 **@EnableSwagger2** 注解


- **在Controller中使用Swagger注解**  
> 在Controller方法上添加注解，用于说明接口  

```
@ApiOperation(value = "文章分页查询")
@ApiImplicitParams({
        @ApiImplicitParam(name = "pageNum", value = "当前页码", required = true, dataType = "int", paramType = "path"),
        @ApiImplicitParam(name = "pageSize", value = "每页条数", required = true, dataType = "int", paramType = "path"),
        @ApiImplicitParam(name = "tbPostsPostJson", value = "查询条件，文章对象json字符串", required = false, dataType = "int", paramType = "json")
})
```

- **访问接口文档**  
> http://ip:port/swagger-ui.html

- **注解说明**

```
@Api：修饰整个类，描述 Controller 的作用
@ApiOperation：描述一个类的一个方法，或者说一个接口
@ApiParam：单个参数描述
@ApiModel：用对象来接收参数
@ApiProperty：用对象接收参数时，描述对象的一个字段
@ApiResponse：HTTP 响应其中 1 个描述
@ApiResponses：HTTP 响应整体描述
@ApiIgnore：使用该注解忽略这个API
@ApiError：发生错误返回的信息
@ApiImplicitParam：一个请求参数
@ApiImplicitParams：多个请求参数
```

