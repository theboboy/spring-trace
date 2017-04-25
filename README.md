# spring-trace

## 关于 Trace

你可以轻松地跟踪方法调用来观察应用程序的状态。

**应用实例**

    [REQ] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, body={username:"hello"}
    |-->[Controller] HelloController.test()
    |   |-->[Service] HelloService.hello(holyeye)
    |   |   |-->[Repository] HelloRepository.helloQuery()
    |   |   |<--[Repository] HelloRepository.helloQuery() [size=2] 1ms.
    |   |<--[Service] HelloService.hello(holyeye) [<Member>] 1ms.
    |<--[Controller] HelloController.test() [hello holyeye] 1ms.
    [RES] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, status=200, time=3ms, ex=null
    
## 构建 TODO

## 测试 TODO

## 功能

**输出功能**

- 实时日志输出(`TRACE`)
- 统计日志输出
    - 慢逻辑日志(`SLOW_LOGIC`) ：输出特定的时间段内的日志
    - 应用程序异常日志(`APP_ERROR`) ：当异常时输出错误日志

**日志功能**

- 可以查看方法调用耗费的时间
    - 例) `hello.finds() took 2ms. [size=2]`

- 可以跟踪方法调用的参数和返回值
    - 返回值是 `null` 输出 `null` 例) `[null]`
    - 返回值是一个对象 输出该对象类型 例) `[<Member>]`
    - 返回值是一个集合 输出它的长度 例) `[size=10]`
         
- HTTP请求
    - 可以跟踪各种HTTP请求信息 特别是可以输出HTTP的body信息

## 注意事项

- 必须使用Spring框架
- 目前只支持注解配置
- 项目处于开发阶段 可能会有bug

## 如何使用

Spring框架 `@EnableTrace` 注解
```java
@EnableTrace(basePackages = "spring.trace.testweb")
```

如
```java
@Configuration
@EnableTrace(basePackages = "spring.trace.testweb")
public class TargetWebConfig extends WebMvcConfigurerAdapter {
}
```

Web应用程序要添加 `spring.trace.web.TraceLogFilter` 过滤器

```java
public class TestWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    //...
    @Override
    protected Filter[] getServletFilters() {
        return new Filter[]{new TraceLogFilter()};
    }
}
```

配置 `logback.xml`

```xml
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{MM-dd HH:mm:ss} [%thread] %.-1level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>

    <!-- 实时日志 -->
    <logger name="TRACE" level="trace" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <!-- 慢逻辑日志 -->
    <logger name="SLOW_LOGIC" level="info" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <!-- 应用程序异常日志 -->
    <logger name="APP_ERROR" level="info" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

</configuration>
```

-----------------------

> 注意：在生产环境输出太多的日志会带来性能问题 建议设置为仅输出应用程序异常日志和慢逻辑日志

-----------------------


## 运行效果

**实时日志**

    06-09 23:14:44 TRACE - [REQ] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, body=null
    06-09 23:14:44 TRACE - |-->[Controller] HelloController.test()
    06-09 23:14:44 TRACE - |   |-->[Service] HelloService.hello(holyeye)
    06-09 23:14:44 TRACE - |   |   |-->[Repository] HelloRepository.helloQuery()
    06-09 23:14:44 TRACE - |   |   |<--[Repository] HelloRepository.helloQuery() [void] 1ms.
    06-09 23:14:44 TRACE - |   |<--[Service] HelloService.hello(holyeye) [hello holyeye] 1ms.
    06-09 23:14:44 TRACE - |<--[Controller] HelloController.test() [hello holyeye] 1ms.
    06-09 23:14:44 TRACE - [RES] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, status=200, time=3ms, ex=null

**慢逻辑日志**

    06-09 23:14:44 [http-nio-8080-exec-6] E SLOW_LOGIC - TRACE LOG
    [REQ] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, body=null
    |-->[Controller] HelloController.test()
    |   |-->[Service] HelloService.hello(holyeye)
    |   |   |-->[Repository] HelloRepository.helloQuery()
    |   |   |<--[Repository] HelloRepository.helloQuery() [void] 1ms.
    |   |<--[Service] HelloService.hello(holyeye) [hello holyeye] 1ms.
    |<--[Controller] HelloController.test() [hello holyeye] 1ms.
    [RES] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/test, status=200, time=3ms, ex=null

**应用程序异常日志**

    06-09 23:28:28 [http-nio-8080-exec-9] E APP_ERROR - TRACE LOG
    [REQ] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/exception, body=null
    |-->[Controller] HelloController.exception()
    |   |-->[Service] HelloService.helloException()
    |   |<X-[Service] HelloService.helloException() Exception! java.lang.Exception: exception 1ms.
    |<X-[Controller] HelloController.exception() Exception! java.lang.Exception: exception 1ms.
    [RES] host=0:0:0:0:0:0:0:1, method=GET, url=http://localhost:8080/exception, status=200, time=6ms, ex=org.springframework.web.util.NestedServletException: Request processing failed; nested exception is java.lang.Exception: exception
    [EXCEPTION] Request processing failed; nested exception is java.lang.Exception: exception; trace=org.springframework.web.util.NestedServletException: Request processing failed; nested exception is java.lang.Exception: exception
    	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:973)
    	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:852)
    	at javax.servlet.http.HttpServlet.service(HttpServlet.java:621)
    	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:837)
    	at javax.servlet.http.HttpServlet.service(HttpServlet.java:728)
    	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:305)
    	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210)
    	at spring.trace.web.TraceLogFilter.doFilterInternal(TraceLogFilter.java:74)
    	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
    	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:243)
        ...

## 工作原理

使用 `ThreadLocal` 存储日志信息


## 其它

从 `org.springframework.aop.interceptor.CustomizableTraceInterceptor` 获取了很多的灵感来源 

## TODO

- 慢逻辑日志时间
- 要监听配置的操作（运行时处理）
- 支持@Async
- XML配置

## 许可

Apache License v2.0
