# SpringBoot Filter and Interceptor

## Filter and Interceptor Intruduction
Filter and Interceptor 都是aop的实现方式，两者的区别如下：  
1、Filter是依赖于Servlet容器，属于Servlet规范的一部分，而拦截器则是独立存在的，可以在任何情况下使用。  
2、Filter的执行由Servlet容器回调完成，而拦截器通常通过动态代理的方式来执行。  
3、Filter的生命周期由Servlet容器管理，而拦截器则可以通过IoC容器来管理。  

## Filter的两种实现方式

### 通过FilterRegistration的方式实现
#### Filter
```
public class OrderedFilter implements Filter{
    private static final Logger logger = LoggerFactory.getLogger(OrderedFilter.class);

    @Override
    public void doFilter(ServletRequest servletRequest,
            ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException{
        filterChain.doFilter(servletRequest, servletResponse);
        logger.info("Ordered Filter");
    }
}
```
#### Filter Registration
```
@Bean
public FilterRegistrationBean registFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new OrderedFilter());
    registration.addUrlPatterns("/*");
    registration.setName("OrderedFilter");
    registration.setOrder(1);
    return registration;
}
```
> 使用该方式时，一定要设置Order

### 通过注解（@WebFilter）的方式实现

#### 代码如下
```
@WebFilter(urlPatterns = "/*", filterName = "logRequestInfoFilter")
public class LogRequestInfoFilter implements Filter{
    private static final Logger logger = LoggerFactory.getLogger(LogRequestInfoFilter.class);

    @Autowired
    Invoker invoker;

    @Override
    public void doFilter(ServletRequest servletRequest,
            ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException{
        logger.info(invoker.getInvokerName());

        filterChain.doFilter(servletRequest, servletResponse);

        StringBuilder builder = new StringBuilder();
        builder.append(((HttpServletRequest) servletRequest).getMethod())
                .append(",")
                .append(((HttpServletRequest) servletRequest).getRequestURI())
                .append(",")
                .append(((HttpServletResponse) servletResponse).getStatus())
                .append(",")
                .append(servletRequest.getProtocol())
                .append(",")
                .append(servletRequest.getRemoteHost())
                .append(",")
                .append(((HttpServletRequest) servletRequest).getHeader("user-agent"));
        logger.info(builder.toString());
    }
}
```
从代码中可以看到 也可以通过依赖注入的方式注入依赖的bean。
@WebFilter可以设置url匹配模式，过滤器名称等。
> 需要注意的是@WebFilter是Servlet3.0的规范，并不是Springboot提供的。除了这个注解以外，还需在配置类中加另外一个注解：@ServletComponetScan，指定扫描的包,如：@ServletComponentScan(value = "com.moon.springbootpractice.filter")

### Filter的执行顺序
通过注解的方式实现的Filter并没有指定执行的顺序，但是却在通过注册的方式实现的Filter之前执行。@WebFilter这个注解并没有指定执行顺序的属性，其执行顺序依赖于Filter的名称，是根据Filter类名（注意不是配置的filter的名字）的字母顺序倒序排列，并且@WebFilter指定的过滤器优先级都高于FilterRegistrationBean配置的过滤器。
```
2019-04-28 15:15:07.569  INFO 22180 --- [nio-7008-exec-1] c.m.s.filter.LogRequestInfoFilter        : moon
2019-04-28 15:15:07.596  INFO 22180 --- [nio-7008-exec-1] c.m.s.filter.LogRequestInfoFilter        : GET,/practice/api/city,200,HTTP/1.1,0:0:0:0:0:0:0:1,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
2019-04-28 15:15:07.600  INFO 22180 --- [nio-7008-exec-1] c.m.s.filter.OrderedFilter               : Ordered Filter
```


## Interceptor
### 创建Interceptor
```
@Component
public class LogInterceptor implements HandlerInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(LogInterceptor.class);

    @Override
    public boolean preHandle(
            HttpServletRequest request, HttpServletResponse response,
            Object handler) {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest servletRequest,
            HttpServletResponse servletResponse, Object handler,
            @Nullable ModelAndView modelAndView) {

        StringBuilder builder = new StringBuilder();
        builder.append(servletRequest.getMethod())
                .append(",")
                .append(servletRequest.getRequestURI())
                .append(",")
                .append(servletResponse.getStatus())
                .append(",")
                .append(servletRequest.getProtocol())
                .append(",")
                .append(servletRequest.getRemoteHost())
                .append(",")
                .append(servletRequest.getHeader("user-agent"));
        logger.info(builder.toString());
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            @Nullable Exception ex) throws Exception {
    }
}
```
HandlerInterceptor这个接口包括三个方法，preHandle是请求执行前执行的，postHandler是请求结束执行的，但只有preHandle方法返回true的时候才会执行，afterCompletion是视图渲染完成后才执行，同样需要preHandle返回true，该方法通常用于清理资源等工作。除了实现该接口外，还需进行配置

```
@Configuration
public class InterceptorConfig implements WebMvcConfigurer
{
    @Autowired
    LogInterceptor logInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(logInterceptor).addPathPatterns("/**");
    }
}
```
执行结果如下：

```
2019-04-28 15:46:37.460  INFO 20388 --- [nio-7008-exec-1] c.m.s.filter.LogRequestInfoFilter        : moon
2019-04-28 15:46:37.494  INFO 20388 --- [nio-7008-exec-1] c.m.s.interceptor.LogInterceptor         : GET,/practice/api/city,200,HTTP/1.1,0:0:0:0:0:0:0:1,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
2019-04-28 15:46:37.494  INFO 20388 --- [nio-7008-exec-1] c.m.s.filter.LogRequestInfoFilter        : GET,/practice/api/city,200,HTTP/1.1,0:0:0:0:0:0:0:1,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
2019-04-28 15:46:37.497  INFO 20388 --- [nio-7008-exec-1] c.m.s.filter.OrderedFilter               : Ordered Filter
```
