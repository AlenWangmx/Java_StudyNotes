# SpringMVC其他扩展

## 一、全局异常处理机制

### 1.1 异常处理的两种方式

对于异常处理，一般分为两种方式：

* 编程式异常处理：try-catch、try-catch-finally
* 声明式事务异常处理：@Throws 或 @ExceptionHandler

### 1.2 基于注解异常声明异常处理

1. 声明异常处理控制器类

    异常处理控制类，统一定义异常处理handler方法

    两个注解用法：

    * @ControllerAdvice：可以返回逻辑视图、转发和重定向
    * @RestControllerAdvice：@ControllerAdvice + @ResponseBody，直接返回json字符串

    ```java
    /**
    * 全局异常处理器:全局发生异常时，会走此类写的handler
    */

   //@ControllerAdvice   //可以返回逻辑视图 转发和重定向
   @RestControllerAdvice   //@ResponseBody直接返回json字符串
   public class GlobalExceptionHandler {

   }
    ```

1. 声明异常处理handler方法

    ```java
    /**
    * 全局异常处理器:全局发生异常时，会走此类写的handler
    */

   //@ControllerAdvice   //可以返回逻辑视图 转发和重定向
   @RestControllerAdvice   //@ResponseBody直接返回json字符串
   public class GlobalExceptionHandler {

       //发生异常 -> ControllerAdvice 注解的类型 -> @ExceptionHandler(指定的异常) -> handler

       /**
        *  指定的异常可以精准查找，或者查找父异常
        */

       //传入算数异常，调用此异常处理
       @ExceptionHandler(ArithmeticException.class)
       public Object ArithmeticExceptionHandler(ArithmeticException e){
           //自定义处理异常即可 handler
           String message = e.getMessage();
           System.out.println("message = " + message);

           return message;
       }

        //若无对应异常处理，调用此异常处理
       @ExceptionHandler(Exception.class)
       public Object ExceptionHandler(Exception e){
           //自定义处理异常即可 handler
           String message = e.getMessage();
           System.out.println("message = " + message);

           return message;
       }
   }
    ```

## 二、拦截器使用

### 2.1 拦截器概念

1. 拦截器和过滤器解决问题

   * 生活中：为了提高乘车效率，在乘客进入站台前统一检票
     * 乘客 ——> 检票口（拦截器）——> 候车站
   * 程序中：使用拦截器在请求到达具体handler方法前，统一执行检测
     * 用户登录了，进入用户界面
     * 没登陆，则被拦截回到登录界面

       ![5-SpringMVC其他扩展0](image/5-SpringMVC%E5%85%B6%E4%BB%96%E6%89%A9%E5%B1%95/1711616514015.png)  

2. 拦截器（SprringMVC机制）**VS** 过滤器（JavaWeb机制）

    * 相似点
      * 拦截：必须把请求拦住，才能执行后续操作
      * 过滤：拦截器或过滤器存在的意义就是对请求进行统一处理
      * 放行：对请求执行了必要的操作后，放请求通过，让请求访问其要求的资源
    * 不同点
      * 工作平台不同
        * 过滤器工作在Servlet容器中
        * 拦截器或工作在SpringMVC的基础上
      * 拦截的范围不同
        * 过滤器：能够拦截到的最大范围时整个Web应用
        * 拦截器：能够拦截到的最大范围时整个SpringMVC负责的请求
      * IOC容器支持
        * 过滤器：想得到IOC容器需要调用专门的工具方法，是间接的
        * 拦截器：本身就在IOC容器中，所以可以直接从IOC容器中装配组件，也就是可以直接得到IOC容器的支持
    * 选择：功能需要如果可以使用SpringMVC拦截器实现，就不使用过滤器

### 2.2 拦截器使用

1. 创建拦截器类

    创建拦截器类实现HandlerInterceptor接口，重写**preHandle、postHandle、afterCompletion**三个方法

    ```java
    public class MyInterceptor implements HandlerInterceptor {

        //执行handler之前！调用的拦截方法！
        //编码格式设置，登录保护，权限处理！！

        /**
         *  filter - doFilter
         * @param request 请求对象
         * @param response 响应对象
         * @param handler handler就是我们要调用的方法
         * @return true 放行 false 拦截
         * @throws Exception 抛出异常
         */
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            System.out.println("request = " + request + ", response = " + response + ", handler = " + handler);
            return true;//return true 放行 false 拦截
        }

        /**
         * 当handler方法执行完毕后触发的方法，handler报错则不执行
         * 此方法只有preHandler return true
         * 对结果处理，如敏感词检测等
         *
         * @param request 请求
         * @param response 响应
         * @param handler handler方法
         * @param modelAndView 返回的视图和共享域数据对象
         * @throws Exception 抛出异常
         */
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("MyInterceptor.postHandle");
        }

        /**
         * 渲染视图之后执行（最后），一定会执行
         * @param request 当前的HTTP请求
         * @param response 当前的HTTP响应
         * @param handler 异步开始的handler方法
         * @param ex handler 报错了 异常对象
         * @throws Exception
         */
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("MyInterceptor.afterCompletion");
        }
    }
    ```

2. 修改配置类添加拦截器

    ```java
    @EnableWebMvc
    @Configuration
    @ComponentScans({@ComponentScan("扫描包的路径"),
                    ...})
    public class MvcConfig implements WebMvcConfigurer {

        //@EnableWebMvc自动配置handlerMapping handlerAdapter json转换器

        //开启静态资源查找
        /*@Override
        public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
            configurer.enable();
        }*/

        //添加拦截器
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //配置方案1：拦截全部请求
            registry.addInterceptor(new MyInterceptor());
        }
    }
    ```

3. 拦截范围

    * 默认全部拦截

        ```java
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //配置方案1：拦截全部请求
            registry.addInterceptor(new MyInterceptor());
        }
        ```

    * 精准拦截

        ```java
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //配置方案2：指定地址拦截
            registry.addInterceptor(new MyInterceptor())
                    .addPathPatterns("/user/data");//拦截具体请求
            //或者
            // * （模糊拦截）** （任意多层字符串）
            registry.addInterceptor(new MyInterceptor())
                    .addPathPatterns("/user/**");//拦截user之内的所有请求
        }

    * 排除拦截

        ```java
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //配置方案3：排除拦截 排除的地址应该在拦截地址的内部！
            registry.addInterceptor(new MyInterceptor())
                    .addPathPatterns("/user/**").excludePathPatterns("/user/data1");
        }
        ```

4. 多个拦截器执行的顺序

    * preHandle()方法：SpringMVC会把所有拦截器收集到一起，然后按照配置【顺序】调用各个preHandle()方法
    * postHandle()方法：SpringMVC会把所有拦截器收集到一起，然后按照配置【相反的顺序】调用各个postHandle()方法
    * afterCompletion()方法：SpringMVC会把所有拦截器收集到一起，然后按照配置【相反的顺序】调用各个afterCompletion()方法

## 三、参数校验
