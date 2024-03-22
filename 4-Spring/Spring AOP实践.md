# Spring AOP实践

## 一、场景设定和问题复现

## 二、解决技术：代理模式

## 三、面对切面编程思维（AOP）

详细参考【SpringFramework概念】【四】

## 四、Spring AOP框架介绍和关系梳理

1. AOP是一种区别于OOP的编程思维，用来完善和解决OOP的【非核心代码冗余】和【不方便统一维护】问题
2. 代理技术（动态代理|静态代理）是实现AOP思维编程的具体技术，但是自己使用动态代码代理实现代码比较繁琐
3. Spring AOP框架，基于AOP编程思维，封装动态代理技术，简化动态代理技术实现的框架；SpringAOP内部帮助我们实现动态代理，我们只需要写少量的配置，指定生效范围即可完成面向切面思维编程的实现

## 五、Spring AOP基于注解方式实现和细节

### 5.1 Spring AOP底层技术组成

![Spring AOP实践0](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710842646318.png)  

* 动态代理：JDK原生方式，需要被代理的目标类必须实现接口。因为这个技术要求代理对象和目标对象实现同样的接口
* CGlib：通过继承被代理的目标类实现代理，所以不需要目标类实现接口
* AspectJ：早期的AOP实现框架，SpringAOP借用了AspectJ中的AOP注解

### 5.2 初步实现

1. 加入依赖

    * ioc/di -> spring-context
    * aop -> spring-aop/spring-aspects

    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>6.1.4</version>
    </dependency><!--依赖于spring-context，可不导入-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>6.1.4</version>
    </dependency>
    ```

2. 准备接口

    ```java
    public interface Calculator {

    int add(int i,int j);

    int sub(int i,int j);

    int mul(int i,int j);

    int div(int i, int j);

    }
    ```

3. 纯净实现类，加入IoC容器

    ```java
    /**
    * aop - 只针对ioc容器的对象 - 创建代理对象 -> 代理对象存储到ioc容器
    */
    @Component
    public class CalculatorPureImpl implements Calculator {
        @Override
        public int add(int i, int j) {
            int result= i+j;
            return result;
        }

        @Override
        public int sub(int i, int j) {
            int result= i-j;
            return result;
        }

        @Override
        public int mul(int i, int j) {
            int result= i*j;
            return result;
        }

        @Override
        public int div(int i, int j) {
            int result= i/j; 
            return result;
        }
    }
    ```

4. 编写ioc的配置类

    ```java
    @Configuration
    @ComponentScan("org.aoptest")
    public class JavaConfig {
    }
    ```

5. 增强类，定义三个增强方法

    ```java
    /**
    * 1、强类的内部要存储增强代码
    *  具体定义几个方法，根据插入的位置决定
    * 2、定义代码存储增强方法
    *  前置 @Before
    *  后置 @AfterReturning
    *  异常 @AfterThrowing
    *  最后 @After
    *  环绕 @Around
    *
    *  如下结构：
    *  try{
    *      前置
    *      目标方法执行
    *      后置
    *  }catch(){
    *      异常
    *  }finally{
    *      最后
    *  }
    *
    * 3.配置切点表达式 [选中插入方法 ——> 切点]
    * 4.补全注解
    *  加入ioc容器 @Component
    *  配置切面 @Aspect = 切点 + 增强
    *  5.开启aspect注解的支持
    */

    @Component
    @Aspect
    public class LogAdvice {

        @Before("execution(* org.aoptest.service..impl.*.*(..))")
        public void start(){
            System.out.println("方法开始了");
        }

        @After("execution(* org.aoptest.service..impl.*.*(..))")
        public void end(){
            System.out.println("方法结束了");
        }

        @AfterThrowing("execution(* org.aoptest.service..impl.*.*(..))")
        public void error(){
            System.out.println("方法报错了");
        }
    }
    ```

6. 增强类的配置

    ```xml
    <!--方法1（不推荐）：xml配置开启aspectj的注解配置支持-->
    <aop:aspectj-autoproxy/>
    ```

    ```java
    @Configuration
    @ComponentScan("org.aoptest")
    //方法二：注解开启aspectj的注解支持
    @EnableAspectJAutoProxy 
    public class JavaConfig {
    }
    ```

7. 测试环境

    ```java
    @SpringJUnitConfig(value = JavaConfig.class)
    public class SpringTest {
        //aop - 代理 - jdk - 接口 - 代理类 - 代理对象和目标对象是兄弟关系 要用接口接值
        //aop - ioc - 代理对象
    //    private CalculatorPureImpl calculator;
        @Autowired
        private Calculator calculator;

        @Test
        public void test(){
            int add = calculator.div(1, 1);
            System.out.println("add = " + add);
        }
    }
    ```

    输出结果：

    ![Spring AOP实践1](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710849268311.png)  

### 5.3 获取通知细节信息

### 5.4 切点表达式语法【重要】

1. 切点表达式作用

    AOP切点表达式（Pointcut Expression）是一种用于指定切点的语言，它可以通过定义匹配规则，来选择需要被切入的目标对象

    ![Spring AOP实践2](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710851295149.png)  

2. 切点表达式语法*
    * 表达式总结
  
        ![Spring AOP实践3](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710851460598.png)  
    * 语法细节

      ```java
      /*
        TODO：切点表达式
            固定语法 execution(1 2 3.4.5(6))
            1.访问修饰符
                public / private
            2.方法的返回参数类型
                String int void
            如果不考虑访问修饰符和返回值！这两位整合成一起写 *
            如果不考虑，必须两个都不考虑！不能出现例如 * String
            3.包的位置
                具体包：org.alan.service.impl
                单层模糊：org.alan.service.* *单层模糊
                多层模糊：org.impl ..任意层的模糊
                细节：..不能开头
                找所有impl包：   org.impl    不能写..impl   写成：*..impl
            4.类的名称
                具体：CalculatorPureImpl
                模糊：*Impl
            5.方法名：语法和类名一致
            6.(6)形参列表
                没有参数()
                有具体参数(String) (String,int)
                模糊参数：(..) 有没有参数都可以，有多个也可以
                部分模糊：(String) String后面没有无所谓
                        (..int) 最后一个参数是int
                        (String..int)
        */
      ```

### 5.5 重用（提取）切点表达式

1. 当前类中提取（不推荐）
    * 定义一个方法，注解 @Pointcut()
  
        ```java
        @Pointcut("execution(* org.aoptest.service..impl.*.*(..))")
        public void area(){}

        //增强方法中引用切点表达式的方法即可
        @Before("area()")
        public void start(){
            System.out.println("方法开始了");
        }
        ```

2. 创建一个存储切点的类

    * 单独的类维护切点表达式
  
        ```java
        //创建单独的类MyPointCut.Class
        @Component
        public class MyPointCut {

            @Pointcut("execution(* org.aoptest.service..impl.*.*(..))")
            public void area(){}

            @Pointcut("execution(* org..impl.*.*(..))")
            public void area2(){} 
        }

        //其他类的切点方法 类的全限定符.方法名()
        @Before("org.aoptest.MyPointCut.area()")
        public void start(){
            System.out.println("方法开始了");
        }

        @AfterReturning(value = "org.aoptest.MyPointCut.area2()",returning = "result")//“value =”可省略
        public void afterReturning(JoinPoint joinPoint,Object result){}
        ```

### 5.6 环绕通知（@Around）

1. 环绕通知（@Around）需要在通知中定义方法的执行
    * joinPoint 目标方法（获取目标方法信息，多了一个执行方法）
    * return 目标方法的返回值
    * 一个@Around便可以完成@Before、@AfterReturning、@AfterThrowing等增强方法，两种方式不分好坏，主要看开发者习惯哪种方式

    ```java
    @Component
    @Aspect
    public class TxAroundAdvice {

        /**
        * 环绕通知需要在通知中定义目标方法的执行
        * @param joinPoint 目标方法（获取目标方法信息，多了一个执行方法）
        * @return 目标方法的返回值
        */
        @Around("org.aoptest.MyPointCut.area()")
        public Object transaction(ProceedingJoinPoint joinPoint){

            //保证目标方法的执行
            Object[] args = joinPoint.getArgs();
            Object result = null;

            try {
                //增强代码 -> before
                System.out.println("开启事务");
                result = joinPoint.proceed(args);
                System.out.println("结束事务");

            } catch (Throwable e) {
                System.out.println("事务回滚");
                throw new RuntimeException(e);

            }finally {

            }
            return result;
        }
    }
    ```

### 5.7 切面优先级设置

1. 相同目标方法上同时存在多个切面时，切面的优先级控制切面的内外嵌套顺序
   * 优先级高的切面：外面
   * 优先级低的切面：里面
2. 使用@Order注解可以控制切面的优先级
   * @Order(小数)：优先级高
   * @Order(大数)：优先级低

    ![Spring AOP实践4](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710913965315.png)  

### 5.8 CGlib动态代理生效

参考【七】中的 7.1场景-【场景五】

### 5.9 注解实现小结

![Spring AOP实践5](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710914995398.png)  

## 六、Spring AOP基于XML方式实现（了解即可）准备工作

参考：<https://www.bilibili.com/video/BV1AP411s7D7> P55（非常用方式，了解即可）

## 七、Spring AOP对获取Bean的影响

### 7.1场景

1. 场景一
   * bean对应的类没有实现任何接口
   * 根据bean本身的类型获取bean
     * 测试：IOC容器中同类型的bean只有一个
       * 正常获取到IOC容器中的那个bean对象
     * 测试：IOC容器中同步类型的bean对象有多个
       * 会抛出NoUniqueBeanDefinitionException异常，表示IOC容器中这个类型的bean有多个
2. 场景二
   * bean对应的类实现了接口，这个接口也只有这一个实现类
      * 测试：根据接口类型获取bean
      * 测试：根据类型获取bean
      * 结论：上面两种情况其实都能够正常获取到bean，而且是同一个对象
3. 场景三
   * 声明一个接口
   * 接口有多个实现类
   * 接口所有实现类都放入IoC容器
      * 测试：根据接口类型获取bean
        * 会抛出NoUniqueBeanDefinitionException异常，表示IOC容器中这个类型的bean有多个
      * 测试：根据类获取bean
        * 正常获取到IOC容器中的bean对象
4. 场景四
    * 声明一个接口
    * 接口有一个实现类
    * 创建一个切面类，对上面接口的实现类应用通知
      * 测试：根据接口类型获取bean
        * 正常获取
      * 测试：根据类获取bean
        * 无法获取
  
    原因分析
    * 应用了切面后，真正放在IoC容器中的是代理类的对象
    * 目标类并没有被放到IoC容器中，所以根据目标类的类型从IoC容器中是找不到的

        ![Spring AOP实践6](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710916586092.png)  

5. 场景五
    * 声明一个类
    * 创建一个切面类，对上面的类应用通知
      * 测试：根据类获取bean，可以获取

        ![Spring AOP实践7](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710916745763.png)  

    * debug查看真实类型，使用了cglib动态代理，而非jdk代理

        ![Spring AOP实践8](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710916838284.png)  

### 总结

* 对实现了接口的类应用切面
  
    ![Spring AOP实践9](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710916912993.png)  

* 对没有实现接口的类应用切面

    ![Spring AOP实践10](image/Spring%20AOP%E5%AE%9E%E8%B7%B5/1710916980054.png)  

如果使用AOP技术，目标类有接口，必须使用接口类型接收IoC容器中的代理组件
