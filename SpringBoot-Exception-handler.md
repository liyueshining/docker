# Spring Boot exception handler
springboot在又异常的情况下会默认重定向到/error, 如果需要获取到异常的详细信息的话就要做自定义处理。
需要使用RestControllerAdvice 和 ExceptionHandler注解
## Demo
```
@RestControllerAdvice
public class CustomControllerAdvice
{
    private static final Logger logger = LoggerFactory.getLogger(CustomControllerAdvice.class);

    @ExceptionHandler({ TypeMismatchException.class })
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResult requestTypeMismatch(TypeMismatchException ex) {
        return new ErrorResult(100, ex.getMessage());
    }

    @ExceptionHandler(HttpClientErrorException.class)
    public ResponseEntity<Object> clientException(Exception ex) {
        int status = ((HttpClientErrorException) ex).getRawStatusCode();
        ErrorResult result = new ErrorResult(status, ((HttpClientErrorException) ex).getStatusText());

        return new ResponseEntity(result, HttpStatus.valueOf(status));
    }

    @ExceptionHandler(HttpServerErrorException.class)
    public ResponseEntity<Object> serverException(Exception ex) {
        int status = ((HttpServerErrorException) ex).getRawStatusCode();
        ErrorResult result = new ErrorResult(status, ((HttpServerErrorException) ex).getStatusText());

        return new ResponseEntity(result, HttpStatus.valueOf(status));
    }

    //其他错误
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResult exception(Exception ex) {
        return new ErrorResult(500, ex.getMessage());
    }

}
```
在应用代码中有异常的话 可以通过以下方式抛出
```
@RestController
public class SampleController {
    private static final Logger logger = LoggerFactory.getLogger(SampleController.class);

    @RequestMapping(value = "/api/city", method = RequestMethod.GET)
    public String findAllCity() {
        return "Shanghai";
    }

    @RequestMapping(value = "/exception", method = RequestMethod.GET)
    public String exception() {
        throw new HttpClientErrorException(HttpStatus.FORBIDDEN, "wbfiwbebwb");
    }
}
```

上面的方式处理不了Filter中抛出的异常，需要在filter中自定义response的信息，如下：
```
@WebFilter(urlPatterns = "/*", filterName = "logRequestInfoFilter")
public class LogRequestInfoFilter implements Filter
{
    private static final Logger logger = LoggerFactory.getLogger(LogRequestInfoFilter.class);

    @Autowired
    Invoker invoker;

    @Value("${app.exception.level:app}")
    String filter;

    @Override
    public void doFilter(ServletRequest servletRequest,
            ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException
    {
        logger.info(invoker.getInvokerName());
        if (filter.equals(invoker.getInvokerName())) {

            ErrorResult result = new ErrorResult();
            result.setCode(HttpStatus.FORBIDDEN.value());
            result.setMessage("exception msg");

            HttpServletResponse response = ((HttpServletResponse)servletResponse);
            response.setStatus(HttpStatus.FORBIDDEN.value());
            response.setCharacterEncoding("UTF-8");
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().print(new JSONObject(result));

            return;
        }

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

## CORS
### 方式一，使用注解
```
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com")
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

### 方式二 使用配置
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```
