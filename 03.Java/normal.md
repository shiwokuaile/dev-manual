## SpringBoot 整合 Redis

**1、依赖**

```xml
<!-- redis 缓存操作 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- pool 对象池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

**2、配置**

```yaml
spring:
  redis:
    # 地址
    host: 192.168.241.161
    # 端口，默认为6379
    port: 6580
    # 数据库索引
    database: 0
    # 密码
    password: 123456
    # 连接超时时间
    timeout: 10s
    lettuce:
      pool:
        # 连接池中的最小空闲连接
        min-idle: 0
        # 连接池中的最大空闲连接
        max-idle: 8
        # 连接池的最大数据库连接数
        max-active: 8
        # #连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms
```

## SpringBoot 整合 MongoDB

**1、依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

**2、配置**

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:123456@192.168.241.161:27017/temp
```

## SpringBoot 整合 RestTemplate

**1、依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**2、配置**

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory clientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(3000);
        factory.setReadTimeout(5000);
        return factory;
    }
}
```

## @RestControllerAdvice

实现统一返回格式，全局异常处理

**1、实现**

```java
import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson2.JSON;
import com.xlh.common.constants.ResponseCodeEnum;
import com.xlh.common.entity.XlhException;
import com.xlh.common.entity.XlhResponseBody;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

@RestControllerAdvice
public class ResponseHandler  implements ResponseBodyAdvice<Object> {

    /**
     * 执行顺序
     * 异常处理 → supports() → beforeBodyWrite()
     * 为了避免对异常处理结果再处理，需要作判断
     * @param returnType
     * @param converterType
     * @return
     */
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return canWrite(returnType.getMethod().getName());
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        XlhResponseBody result = new XlhResponseBody(ResponseCodeEnum.SUCCESS, body);
        // 放行指定 url
        if (this.ignore(request)) {
            return body;
        } else {
            // 单独处理 String 类型（否则出现转换异常的错误）
            if (body instanceof String) {
                return JSON.toJSONString(result);
            }
            return result;
        }
    }

    /**
     * 处理自定义异常
     * 优先匹配具体，匹配不到向上匹配
     * @param e
     * @return
     */
    @ExceptionHandler(XlhException.class)
    public XlhResponseBody handlerBzException(XlhException e) {
        return new XlhResponseBody(ResponseCodeEnum.ERROR.getCode(), e.getMessage(), null);
    }

    @ExceptionHandler(Exception.class)
    public XlhResponseBody handlerException(Exception e) {
        return new XlhResponseBody(ResponseCodeEnum.ERROR, null);
    }

    private boolean canWrite(String name) {
        return !(StrUtil.equals(name, "handlerBzException") || StrUtil.equals(name, "handlerException"));
    }

    private boolean ignore(ServerHttpRequest request) {
        HttpServletRequest servletRequest = ((ServletServerHttpRequest) request).getServletRequest();
        String requestURI = servletRequest.getRequestURI();
        return requestURI.contains("swagger-resources") || requestURI.contains("/v3/api-docs");
    }
}
```
