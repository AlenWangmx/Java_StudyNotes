# SpringMVC接收数据

## 一、访问路径设置

【**@RequestMapping**】注解的作用就是将请求的URL地址和吃力请求的方式（hanlder方法）关联起来，建立映射关系

SpringMVC接收到指定的请求，就会来找到在映射关系中对应的方法来处理这个请求

1. 精准路径匹配

    在@RequestMapping注解指定URL地址时，不使用任何通配符，按照请求地址进行精确匹配

    ```java
    @Controller
    @RequestMapping("user")
    public class UserController {

        /**
        * @WebServlet("必须使用/开头")
        * @RequestMapping 不需要/开头
        *         1.精准地址：【一个|多个】 user/login /user/login
        *         2.支持模糊地址 * 任意一层字符串 | ** 任意层任意字符串
        *             /user/* -> user/a user/aaaa 可以 /user/a/b 不行
        *             /user/** -. user/a user/a/a/a/a/a
        *         3.类上和方法上都添加@RequestMapping的区别
        *             类上提取通用的访问地址
        *             方法上时具体的handler地址
        *             访问：类地址+方法地址即可
        *         4.请求方式指定
        *             客户端 -> http (get|post|put|delete) -> ds -> handler
        *             默认情况：@RequestMapping("login") 主要地址正确，任何请求方式都可以访问
        *             指定请求方式：method = {RequestMethod.GET,RequestMethod.POST}
        *             不符合请求方式：会出现405异常
        *         5.注解进阶 只能使用在方法上
        *              get @GetMapping == @RequestMapping(xxx,method=GET);
        *              post @PostMapping == @RequestMapping(xxx,method=POST);
        *              put @PUTMapping == @RequestMapping(xxx,method=PUT);
        *              delete @DeleteMapping == @RequestMapping(xxx,method=DELETE)
        */



        @RequestMapping(value = "login",method = RequestMethod.POST)//作用注册地址 将handler注册到handlerMapping
        public String login(){
            return null;
        }

        @RequestMapping(value = "register",method = {RequestMethod.GET,RequestMethod.POST})
        public String register(){
            return null;
        }
    }

    ```

## 二、接收参数（重点）

### 2.1 param和json参数比较

在HTTP请求中，我们可以选择不同的参数模型，如param类型和json类型。下面对这两种参数类型进行区别和比较：

   1. 参数编码

        param类型的参数会被编码成ASCⅡ码。例如，假设 name=john doe，则会被编码成 name=john%20doe。而JSON类型的参数会被编码成UTF-8。

   2. 参数顺序

        param类型的参数没有顺序限制。但是，JSON类型的参数是有序的。JSON采用键值对的形式进行传递，其中键值对是有序排列的。

   3. 数据类型

        param类型的参数仅支持字符串类型、数值类型和布尔类型等简单类型。而JSON类型的参数则支持更复杂的类型数据，如数组、对象等。

   4. 嵌套性

        param类型的参数不支持嵌套。但是，JSON类型的参数支持嵌套，可以传递更为复杂的数据结构。

   5. 可读性

        param类型的参数格式比JSON类型的参数更加简单、易读。但是，JSON格式在传递嵌套数据结构时更加清晰易懂

总的来说，param类型的参数适用于单一的数据传递，而JSON类型的参数则适用于更复杂的数据结构传递。根据具体的业务需求，需要选择合适的参数类型。在实际开发中，常见的做法是：在GET请求中采用param类型的参数，而在POST请求中采用JSON类型的参数传递。

### 2.2 param参数接收

1. 直接接收

    **客户端请求**

    ![2-SpringMVC接收数据0](image/2-SpringMVC%E6%8E%A5%E6%94%B6%E6%95%B0%E6%8D%AE/1711349772503.png)  

    **handler接收参数**

    只要形参名和类型与传递参数相同，即可自动接收！

    ```java
    @Controller
    @RequestMapping("param")
    public class ParamController{

        @GetMapping(value="/value")
        @ResponseBody
        public String setupFrom(String name, int age){
            System.out.println("name =" + name, "age =" + age);
            return name+age;
        }
    }
    ```

2. **@RequestParam**注解

    可以使用@RequestParam注释将Servlet请求参数(即查询参数或表单数据)绑定到控制器中的方法参数

    **@RequestParam**使用场景：

    * 指定绑定的请求参数名
    * 要求请求参数必须传递
    * 为请求参数提供默认值

    基本用法：

    ```java
    public Object paramFrom(@RequestParam("name") String name,
                            @RequestParam("setAge") int age){
        System.out.println("name =" + name, "age =" + age);
        return name+age;
    }
    ```

### 2.3 路径 参数接收

路径传递参数是一种在URL路径中传递参数的方式。在RESTful的Web应用中，经常使用路径传递来表示资源的唯一标识符或更复杂的表示方式。而SpringMVC框架提供了@PathVariable注解来处理路径传递

* @PathVariable注解允许将URL中的占位符映射到控制器方法中的参数
* 例：将/user/{id}路径下的{id}映射到控制器方法的一个参数中，则可以使用@PathVariable注解来实现

示例：

```java
@Controller
@RequestMapping("path")
@ResponseBody
public class PathController {

    //  path/账号/密码

    //动态路径设计 {key} = *
    //{key} 在形参列表获取传入的参数
    //接收路径参数 String account,String password -> 接收param参数
    //必须使用@PathVariable,当路径名与传参名不一致时可给传参设置与路径名同名的value

    @RequestMapping("{account}/{password}")
    public String login(@PathVariable("account") String username,@PathVariable String password){
        System.out.println("username = " + username + ", password = " + password);
        return "username = " + username + ", password = " + password;
    }
}
```

### 2.4 json 参数接收

1. 前端发送JSON数据的示例：（使用postman测试）

    ```JSON
    {
        "name": "张三"
        "age": 18
        "gender": "男"
    }
    ```

2. **待补充。。。**
3. **待补充。。。**
4. **待补充。。。**
5. @EnableWebMvc注解说明

**待补充。。。**

## 三、接收Cookie数据

可以使用@**CookieValue**注释将HTTP Cookie的值绑定到控制器中的方法参数

考虑使用以下cookie的请求：

```java
@RequestMapping("data")
public String data(@CookieValue(value = "cookieName") String value){
    System.out.println("value = " + value);
    return value;
}
```

下面的示例演示如何获取cookie值：

```java
@GetMapping("save")
public String save(HttpServletResponse response){
    Cookie cookie = new Cookie("cookieName", "root");
    response.addCookie(cookie);
    return "ok";
}
```

## 四、接收请求头数据

**待补充**...

## 五、原生API对象操作

```java
public class ApiController {

    public void data(HttpServletResponse servletResponse,
                     HttpServletRequest request,
                     HttpSession session){
        //正常使用原生对象就可以
        //ServletContext [1.最大的配置文件 2.全局最大共享域 3.核心api getRealPath]
        //方案1：request获取 session获取
        ServletContext servletContext = request.getServletContext();
        ServletContext servletContext1 = session.getServletContext();
        //方案2：ServletContext 会自动装入到ioc容器！程序启动servletContext -> ioc容器
        //直接全局注入
    }
}
```

## 六、共享域对象操作

### 6.1 属性（共享）域作用回顾

了解参考：<https://www.bilibili.com/video/BV1AP411s7D7?p=112>

在JavaWeb中，共享域指的是在Selvlet中存储的数据，以便在同一Web应用程序的多个组件中进行共享和访问。常见的共享域有四种：ServletContext、HttpSession、HttpServletRequest、PageContext。

### 6.2 Request级别属性（共享）域

关键词：model、modelMap、modelAndView，基本不使用，了解即可

### 6.3 Session级别属性（共享）域

略

### 6.4 Application级别属性（共享）域

略

## 七、接收数据总结

![2-SpringMVC接收数据1](image/2-SpringMVC%E6%8E%A5%E6%94%B6%E6%95%B0%E6%8D%AE/1711440494931.png)  
