# SpringMVC项目借助Springfox整合Swagger2

> SpringMVC版本：4.3.7 (实际测试的版本)
>
> Springfox官网：http://springfox.github.io/springfox/

#### 引入依赖(Gradle)

```groovy
// springfox-swagger2最小依赖
compile 'io.springfox:springfox-swagger2:2.9.2'
compile 'io.springfox:springfox-swagger-ui:2.9.2'
// jackson用于将springfox返回的文档对象转换成JSON字符串
compile 'com.fasterxml.jackson.core:jackson-annotations:2.9.9'
compile 'com.fasterxml.jackson.core:jackson-databind:2.9.9'
compile 'com.fasterxml.jackson.core:jackson-core:2.9.9'
// petStore是官方提供的一个代码参考, 可用于后期写文档时进行参考, 可不加
compile 'io.springfox:springfox-petstore:2.9.2'
```

#### 配置SpringMVC配置文件中配置

```xml
<!-- Swagger config配置类的路径，注意放在此处，否则生成不了api doc -->
<bean class="com.yinhai.config.swagger.SwaggerConfig"/>
<!--下面两个是静态资源访问配置(也可在SwaggerConfig中重写addResourceHandlers方法进行配置,此时SwaggerConfig需要继承WebMvcConfigurerAdapter)-->
<mvc:resources location="classpath:/META-INF/resources/" mapping="swagger-ui.html"/>
<mvc:resources location="classpath:/META-INF/resources/webjars/" mapping="/webjars/**"/>
```

#### 配置web.xml

- 正常配置

```xml
<servlet>
	<servlet-name>SpringMVC</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
		<param-name>contextConfigLocation</param-name> 
        <!-- 配置springmvc的配置文件加载位置 -->
		<param-value>classpath:spring/spring-mvc.xml</param-value>
	</init-param>
</servlet>
<servlet-mapping>
	<servlet-name>SpringMVC</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

访问地址为：http://localhost:8080/myNote/swagger-ui.html

- 结合自身项目实际情况，不能通过"/"进行配置，否则其他静态资源无法使用(可能是个例)

  > 公司项目已配置仅针对.do请求拦截*<url-pattern>/.do</url-pattern>*

  1. 第一种解决方案

     ```xml
     <!-- 已省略其他不修改的部分 -->
     <servlet-mapping>
     	<servlet-name>SpringMVC</servlet-name>
     	<url-pattern>/*.do</url-pattern>
         <!-- 新增如下规则 -->
         <url-pattern>/apis/*</url-pattern>
     </servlet-mapping>
     ```

     配置以上规则后，需要在SwaggerConfig中制定pathMapping为"/apis"

     ```java
     // 省略与上述描述相关的代码
     @Bean
     public Docket api() {
     	return new Docket(DocumentationType.SWAGGER_2)
     			.select()
     			.apis(RequestHandlerSelectors.any())
     			.paths(PathSelectors.any())
     			.build()
             	// 默认为"/"
     			.pathMapping("/apis");
     }
     ```

     访问地址：http://localhost:8080/myNote/apis/swagger-ui.html

     **注意：** *按该方案处理后，所有API的前缀均会增加apis，若项目已有历史接口存在，api文档接口地址与真实接口地址会有出入。*

  2. 第二种解决方案

     ```xml
     <!-- 已省略其他不修改的部分 -->
     <servlet-mapping>
     	<servlet-name>SpringMVC</servlet-name>
     	<url-pattern>/*.do</url-pattern>
         <!-- 为swagger配置额外的pattern-->
     	<url-pattern>/v2/api-docs</url-pattern>
     	<url-pattern>/v2/api-docs-ext</url-pattern>
     	<url-pattern>/swagger-resources</url-pattern>
     	<url-pattern>/swagger-resources/configuration/security</url-pattern>
     	<url-pattern>/swagger-resources/configuration/ui</url-pattern>
     </servlet-mapping>
     ```

     访问地址：http://localhost:8080/myNote/swagger-ui.html

#### 配置SwaggerConfig

```java
/**
 * @author kevinjoy89
 * @date 2019-07-19 17:34
 */
@Configuration
@Order(3)
@EnableWebMvc
@EnableSwagger2
public class SwaggerConfig {
	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
				.groupName("api") //  组名
				.tags(new Tag("PC","WEB相关接口"),new Tag("APP","APP相关接口"))
				.select()
            .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
				.paths(PathSelectors.any())
				.build()
				.apiInfo(apiInfo());
	}

	private ApiInfo apiInfo() {
		return new ApiInfoBuilder()
				.title("标准接口文档")
				.description("适用于: XXXX范围内系统使用")
				.version("1.0.0")
				.termsOfServiceUrl("no terms of service")
				.build();
	}
}

```

#### 配置接口类

```java
/**
 * 网格员注册处理类
 * @author kevinjoy89
 * @date 2018/11/07 14:43
 */
@RestController
@RequestMapping("/register/wgyAppRegisterAction")
@Api(tags = {"PC"}, value = "测试API")
public class WgyAppRegisterAction extends WebServiceAction {
    
    // 省略部分代码
    
    /**
     * 注册网格员
     * @return
     */
    @RequestMapping("registerWgyUser.do")
    @ApiOperation(value = "网格员注册", httpMethod = "POST", notes = "注册网格员账号")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "ssqx", value = "所属区县", required = true, paramType = "query",dataType = "String")
    })
    @ApiResponses({
            @ApiResponse(code = 200,message = "OK",response = boolean.class),
            @ApiResponse(code = 1, message = "所属区县不能为空"),
            @ApiResponse(code = 2, message = "测试代码")
    })
    public String registerWgyUser() {
        return null;
    }
    
    // 省略部分代码
    
}

```

配置完成后效果如下：

![1563615584954](C:\Users\Kevin\AppData\Roaming\Typora\typora-user-images\1563615584954.png)