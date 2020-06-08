---
title: Spring Web MVC 错误处理
date: 2020-5-28 23:04:46
author: 东京易冷
categories: Spring
tags:
  - Spring
  - Spring Boot
  - Spring Web MVC
---

## Spring Web MVC 错误处理

#### 默认错误处理流程

Spring Boot 默认提供 `/error` 映射注册在 Servlet 容器中用来处理全部错误请求

Spring Boot 启动时注册了 `ErrorMvcAutoConfiguration` 自动配置类，用于注册错误处理相关的 Bean，其中包括：

- BasicErrorController ：提供了返回错误页面（优先根据错误码返回视图，没有则返回 error 视图）和错误数据（默认 JSON 格式）的 RequestMapping
- DefaultErrorAttributes ：用于解析错误请求的属性
- DefaultErrorViewResolver : 错误视图解析器
- 其他错误处理相关 bean

错误处理流程，以页面访问错误为例

1. 错误请求映射到 BasicErrorController#errorHtml
2. 通过 ErrorAttributes 解析请求得到错误属性，默认为 DefaultErrorAttributes
3. 通过 ErrorViewResolver 解析得到错误视图
4. 若错误视图不为空，则返回，否则返回默认视图 error

可以通过替换流程中用到的 Bean 修改默认错误处理

#### 根据错误码返回错误视图

在静态资源文件夹下添加 error 文件夹，并添加错误码命名的静态页面，可以作为该错误码的错误视图，以 static 文件夹为例

```
src/
+- main/
    +- java/
    | + <source code>
    +- resources/
        +- static/
            +- error/
            | +- 404.html
            | +- 5xx.html
            +- <other static assets>
```

以上文件结构表示，错误码为404时，返回404.html页面，错误码以5开头时，返回5xx.html页面

同时，错误页面还支持模板引擎，在模板引擎文件夹下添加 error 文件夹，并添加错误码命名的模板页面，作为该错误码的模板视图，以 `FreeMarker` 为例

```
src/
+- main/
    +- java/
    | + <source code>
    +- resources/
        +- templates/
            +- error/
            | +- 404.ftlh
            | +- 5xx.html
            +- <other templates>
```

#### 通过 ErrorController 处理错误请求

Spring Boot 默认注册了 `ErrorController` 或 `ErrorAttributes`，

希望完全替换默认行为，可以向 Spring 容器中注册 `ErrorController` 或 `ErrorAttributes` 类型的 Bean

通过 `ErrorController` 处理错误时，可以将 `BasicErrorController` 作为父类

#### 通过 @ControllerAdvice 处理错误请求

通过 `@ControllerAdvice` 可以细粒度的处理错误请求，具体到包和异常类型，用法如下

- 通过 `@ControllerAdvice` 注解指定当前类处理错误请求的包，默认当前类所在包。功能类似的注解有：`@RestControllerAdvice`
- 通过 `@ExceptionHandler` 注解指定当前方法处理错误请求的异常类型，默认全部异常类型

示例如下

```java
@ControllerAdvice(basePackageClasses = CustomController.class)
public class CustomControllerAdvice extends ResponseEntityExceptionHandler {

    @ResponseBody
    @ExceptionHandler(CustomException.class)
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new CustomErrorType(status.value(),ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer);
        request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }
}
```

#### 通过 ErrorViewResolver 解析错误视图

通过 `ErrorViewResolver` 可以自定义视图解析方式，用法如下

```java
@Component
public class CustomErrorViewResolver implements ErrorViewResolver {
    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        return new ModelAndView("viewName", model);
    }
}
```

#### 通过 ErrorPageRegistrar 注册错误视图

通过 `ErrorPageRegistrar` 注册错误页面，如下

```java
@Component
public class CustomErrorPageRegistrar implements ErrorPageRegistrar {
    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }
}
```

ErrorPageRegistrarBeanPostProcessor 会将所有的 ErrorPageRegistrar 处理，注册错误页面
