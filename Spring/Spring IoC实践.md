# Spring IoC实践

## 一、Spring IoC/DI实现步骤

![Spring IoC实践0](image/Spring%20IoC%E5%AE%9E%E8%B7%B5/1710139835226.png)  

### 1.1 编写元数据（配置）

* 配置元数据，即编写交给Spring容器管理组件的信息，配置方式有三种：XML、注解、java类
* 基于XML配置元数据的基本结构

    ```xml
    <bean id="" [1] class="..." [2]></bean>
    ```

* \<bean>标签==组件信息声明
* id属性是标识单个Bean定义的字符串
* class属性定义Bean的类型并使用完全限定的类名

### 1.2 实例化IoC容器

* 提供给ApplicationContext构造函数的位置路径是资源字符串地址，允许容器从各种外部资源（如本地文件系统、Java CLASSPATH等）加载配置元数据
* 我们应该选一个合适的容器实现类，进行IoC容器的实例化工作

    ```java
    //实例化IoC容器，读取外部配置文件，最终会在容器内进行IoC和DI动作
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml","daos.xml");
    ```

### 1.3 获取Bean（组件）

* ApplicationContext是一个高级工厂的接口，能够维护不同bean及其依赖项的注册表。通过使用方法getBean(String name, Class\<T> requiredType)，可以检索bean的实例
* 允许读取Bean定义并访问它们，如以下示例所示：

    ```java
    //创建IoC容器对象，指定外部配置文件，IoC也开始实例组件对象
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml","daos.xml");
    //获取IoC容器的组件对象
    PetStoreService service = coontext.getBean("petStore", PetStoreService.class)
    //使用组件对象
    List<String> userList = service.getUsernameList();
    ```

## 二、基于XML配置方式组件管理（**此方式基本已淘汰，了解原理即可**）

### 2.1 实验一：组件（Bean）信息声明配置（IoC）

* 目标：Spring IoC容器管理一个或多个bean。我们学习如何通过定义XML配置文件，声明组件类信息，交给Spring的IoC容器进行组件管理
* 思路

    ![Spring IoC实践1](image/Spring%20IoC%E5%AE%9E%E8%B7%B5/1710141936811.png)

1. 准备项目

   * 创建maven工程
   * 导入SpringIoC相关依赖

    ```xml
    <>
    ```

2. 待补充。。。

### 2.2 实验二：组件（Bean）依赖注入配置（DI）

* 目标：通过配置文件，实现IoC容器中Bean之间的引用（依赖注入、DI配置）
  * 主要涉及注入场景：基于构造函数的依赖注入和基于Setter的依赖注入
* 思路

    ![Spring IoC实践2](image/Spring%20IoC%E5%AE%9E%E8%B7%B5/1710311624699.png)  

#### 2.2.1 基于构造函数的依赖注入

* 介绍
  * 基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项
  * 下面示例演示一个只能通过构造函数注入进行依赖项注入的类
* 准备组件类

    ```java
    public class UserDao {
    }

    public class UserService {
        private UserDao userDao;

        public UserService(UserDao userDao) {
        this.userDao = userDao;
        }
    }
    ```

* 编写配置文件
* constructor-arg标签：可以引用构造参数ref引用其他bean标识

    ```xml
    <!--引用和被引用的组件 必须全部在ioc容器！不能分开-->

    <!--1.单个构造参数注入-->
    <!--步骤一：将他们都存放在ioc容器中-->
    <bean id="userDao" class="org.alan.ioc02.UserDao"/>
    <bean id="userService" class="org.alan.ioc02.UserService">
        <!--构造参数传值di的配置
            <constructor-arg 构造参数传值的di配置
                value = 直接属性值 String name = "" int age = "18"
                ref = 引用其它的bean的id值
                -->
        <constructor-arg ref="userDao"/>
    </bean>
    ```

#### 2.2.2 基于构造函数的依赖注入（多构造参数解析）

* 介绍
  * 基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项
  * 下面示例演示通过构造函数注入多个参数，参数包含其它bean和基本数据类型
* 准备组件类

    ```java
    public class UserDao {
    }

    public class UserService {
        private UserDao userDao;

        private int age;

        private String name;

        public UserService(UserDao userDao, int age, String name) {
            this.userDao = userDao;
            this.age = age;
            this.name = name;
        }
    }
    ```

* 编写配置文件

    ```xml
    <!--引用和被引用的组件 必须全部在ioc容器！不能分开-->

    <!--2.多个构造参数注入-->
    <bean id="userService1" class="org.alan.ioc02.UserService">
        <!--构造参数的【顺序】填写值-->
        <constructor-arg ref="userDao"/>
        <constructor-arg value="alan"/>
        <constructor-arg value="22"/>
    </bean>

    <bean id="userService2" class="org.alan.ioc02.UserService">
        <!--构造参数的名字name指定填写，【不用考虑顺序】-->
        <constructor-arg name="name" value="alan"/>
        <constructor-arg name="age" value="22"/>
        <constructor-arg name="userDao" ref="userDao"/>
    </bean>

    <bean id="userService3" class="org.alan.ioc02.UserService">
        <!--构造参数的下角标指定填写，不用考虑顺序 index=构造参数的下角标 从左到右 从0开始
        userDao=0 age=1 name=2-->
        <constructor-arg index="2" value="alan"/>
        <constructor-arg index="1" value="22"/>
        <constructor-arg index="0" ref="userDao"/>
    </bean>
    ```

#### 2.2.3 基于setter方法依赖注入

* 介绍
  * 开发中，除了构造函数注入（DI）更多的使用setter方法进行注入
  * 下面示例演示一个只能通过使用纯setter注入进行依赖项注入的类
* 准备组件类

    ```java
    public class MovieFinder {
    }

    public class SimpleMovieLister {
        private MovieFinder movieFinder;
        private String movieName;

        public void setMovieFinder(MovieFinder movieFinder) {
            this.movieFinder = movieFinder;
        }

        public void setMovieName(String movieName) {
            this.movieName = movieName;
        }
    }
    ```

* 编写配置文件
* property标签：可以给setter方法对应的属性赋值
* property标签：name属性代表set方法标识、ref代表引用bean的标识id、value属性代表基本属性值

    ```xml
    <!--3.触发setter方法进行注入-->
    <bean id="movieFinder" class="org.alan.ioc02.MovieFinder"/>
    <bean id="simpleMovieLister" class="org.alan.ioc02.SimpleMovieLister">
        <!--
        name = 属性名 setter方法的去掉set和首字母小写的值！调用set方法的名
            setMovieFinder -> movieFinder
            value | ref 二选一 value="直接属性值" ref="其它bean的值"
        -->
        <property name="movieName" value="功夫熊猫4"/>
        <property name="movieFinder" ref="movieFinder"/>
    </bean>
    ```

### 2.3 实验三：IoC容器创建和使用

* 介绍
  * 前两个实验只是讲解了如何在XML格式的配置文件编写IoC配置和DI配置
  * 想要配置文件中声明的组件类信息真正的进行实例化成Bean对象和形成Bean之间的引用关系，我们需要声明IoC容器对象，读取配置文件，实例化组件和关系维护的过程中都是在IoC容器中实现的

#### 2.3.1 容器实例化

```java
    //创建容器 选择合适的容器实现即可
    /**
     * 接口：BeanFactory
     *      ApplicationContext
     * 实现类：
     *      可以直接通过构造函数实例化
     *      ClassPathXmlApplicationContext      读取类路径下的xml配置方式
     *      FileSystemXmlApplicationContext     读取指定文件位置的xml配置方式
     *      AnnotationConfigApplicationContext  读取配置类方式的ioc容器
     *      WebApplicationContext               读取web项目专属的配置的ioc容器
     */

    //方式一：直接创建容器并且指定配置文件即可【推荐】
    //构造函数（String...配置文件）可以填写一个或则多个
    //ioc di
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring_03.xml","xx.xml");

    //方式二：先创建ioc容器对象，再指定配置文件，再刷新
    //源码的配置过程！ 创建容器[spring]和配置文件指定分开p[自己指定]
    ClassPathXmlApplicationContext applicationContext1 = new ClassPathXmlApplicationContext();
    applicationContext1.setConfigLocations("spring_03.xml","xx.xml");//在外部配置文件设置
    applicationContext1.refresh();//调用ioc和di配置，调用外部配置必须refresh
```

#### 2.3.2 Bean对象读取

```java
//1.创建ioc容器
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext();
    applicationContext.setConfigLocations("spring_03.xml");
    applicationContext.refresh();

    //2.读取ioc容器的组件
    //方案1：直接根据beanId获取即可 返回值类型是Object 需要强转换【不推荐】
    HappyComponent happyComponent = (HappyComponent) applicationContext.getBean("happyComponent");

    //方案二：根据beanId,同时指定bean的类型 Class
    HappyComponent happyComponent1 = applicationContext.getBean("happyComponent", HappyComponent.class);

    //方案三：直接根据类型获取
    //TODO：此方法存在一些问题：根据bean的类型获取，同一个类型，在ioc容器中只能有一个bean！！！
    //TODO：如果ioc容器中存在多个同类型的Bean，会出现：NoUniqueBeanDefinitionException
    //TODO：ioc的配置一定是实现类，但可以根据接口类型获取值！getBean(类型); instanceof ioc容器的类型 == true
    HappyComponent happyComponent2 = applicationContext.getBean(HappyConponent.class);
```

### 2.4 实验四：高级特性：组件（Bean）作用域和周期方法配置

#### 2.4.1 组件周期方法设置

* 我们可以在组件类中定义方法，然后当IoC容器实例化和销毁组件对象的时候进行调用！这两个等待我们称为生命周期方法
* 类似于Servlet的init/destory方法，我们可以在周期方法完成初始化和释放资源等工作

    ![Spring IoC实践3](image/Spring%20IoC%E5%AE%9E%E8%B7%B5/1710323562667.png)  

* 周期方法声明

    ```java
    public class JavaBean {
        /**
        * 必须是public 必须是void返回值 必须无参
        * 初始化方法 ——> 初始化业务逻辑即可
        */
        public  void init(){
            System.out.println("init...");
        }

        /**
        * 销毁方法
        */
        public void clear(){
            System.out.println("destory...");
        }
    }
    ```

* 周期方法配置

    ```xml
    <!--
    init-method = 初始化方法名
    destroy-method = 销毁方法名
    spring ioc容器就会在对应的时间节点回调对应的方法，我们可以在其中写对应的业务即可
    -->
    <bean id="javaBean" class="org.alan.ioc_04.JavaBean" init-method="init" destroy-method="clear"/>
    ```

#### 2.4.2 组件作用域配置

* Bean作用域配置
  * \<bean>标签声明Bean，只是将Bean的信息配置给SpeingIoC容器
  * 在IoC容器中，这些\<bean>标签对应的信息转成Spring内部BeanDefinition对象，BeanDefinition对象内，包含定义的信息（id，class，属性等）
  * 这意味着，BeanDefinition与“类”概念一样，SpringIoC容器可以根据BeanDifinition对象反射创建多个Bean对象实例
  * 具体创建多少个Bean的实例对象，由Bean的作用域Scope属性指定
* 作用域可选值

    |取值|含义|创建对象的时机|默认值|
    |-|-|-|-|
    |singleton|在IoC容器中，这个bean的对象始终为单实例|IoC容器初始化时|是|
    |prototype|这个bean在IoC容器中有多个实例|获取bean时|否|

* 如果是在WebApplicationContext环境下还会有另外两个作用域（但不常用）

    |取值|含义|创建对象的时机|默认值|
    |-|-|-|-|
    |request|请求范围内有效的实例|每次请求|否|
    |session|会话范围内有效的实例|每次会话|否|

* 作用域配置

    ```xml
    <!--声明一个组件信息，默认就是单例模式 一个bean -beanDefinition -组件对象
        prototype模式下getBean一次就会创建一个组件对象-->
    <bean id="javaBean2" class="org.alan.ioc_04.JavaBean2" scope="prototype"/>
    ```

### 2.5 实验四：高级特性：FactoryBean特性和使用

#### 2.5.1 FactoryBean简介

* FactoryBean接口是Spring IoC容器实例化逻辑的可插拔性点。
* 用于配置复杂的Bean对象，可以将创建过程存储在FactoryBean的getObject方法
* Factory Bean\<T>接口提供三种方法：
  * T getObject():
    * 返回此工厂创建的对象的实例。该返回值会被存储到IoC容器
  * boolean isSingleton()：
    * 如果此factoryBean返回单例，则返回true，否则返回false。此方法的默认实现返回true（注意，lombook插件的使用可能影响效果）
  * Class<?> getObjectType()：
    * 返回getObject()方法返回的对象类型，如果事先不知道类型，则返回null

    ![Spring IoC实践4](image/Spring%20IoC%E5%AE%9E%E8%B7%B5/1710330539738.png)

#### 2.5.2 FactoryBean使用场景

1. 代理类的创建
2. 第三方框架整合
3. 复杂对象实例化等

#### 2.5.3 FactoryBean应用

* 准备factoryBean实现类

    ```java
    public class JavaBean {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

    //创建工厂类实现FactoryBean<T>接口
    public class JavaBean_FactoryBean implements FactoryBean<JavaBean> {
        
        private  String value;

        @Override
        public JavaBean getObject() throws Exception {
            //使用自己的方式实例化
            JavaBean javaBean = new JavaBean();
            javaBean.setName(value);
            return javaBean;
        }

        @Override
        public Class<?> getObjectType() {
            return JavaBean.class;
        }

        public void setValue(String value) {
            this.value = value;
        }
    }
    ```

* 配置FactoryBean实现类

    ```xml
     <!--
        id ——> getObject方法返回的对象的标识
        Class ——> factoryBean标准化工厂类-->
    <bean id="javaBean" class="org.alan.ioc_05.JavaBean_FactoryBean">
        <!--此位置的属性：javaBean工厂类配置 而不是getObject方法-->
        <property name="value" value="alan"/>
    </bean>
    ```

* 测试类读取FactoryBean和FactoryBeanObject对象

    ```java
    /*
    读取使用FactoryBean工厂配置的组件对象
     */
    @Test
    public void test_05(){

        //1.创建ioc容器 就会进行组件对象的实例化 -> init
        ClassPathXmlApplicationContext applicationContext
                = new ClassPathXmlApplicationContext("spring_05.xml");

        //2.读取组件
        JavaBean javaBean = applicationContext.getBean("javaBean", JavaBean.class);
        System.out.println("javaBean = "+javaBean);

        //TODO：FactoryBean工厂也会加入到ioc容器 名字 &id
        Object bean = applicationContext.getBean("&javaBean");
        System.out.println("bean = "+bean);

        //3.正常结束ioc容器
        applicationContext.close();

    }
    ```

#### 2.5.4 FactoryBean和BeanFactory的区别

* **FactoryBean**是个Bean，对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似
* **BeanFactory**是个Factory，也就是IOC容器或对象工厂，它是Spring框架的基础，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。

### 2.6 实验六：基于XML方式整合三层架构组件

详情参考：<https://www.bilibili.com/video/BV1AP411s7D7> P27、28

## 三、基于注解方式管理Bean

### 实验一：Bean注解标记和扫描（IoC）

#### 3.1.1 注解理解

* 和XML配置文件一样，注解本身并不能执行，注解本身仅仅只是做一个标记，具体的功能是框架检测到注解标记的位置，然后针对这个位置按照注解标记的功能来执行具体的操作
* 本质上：所有一切的操作都是Java代码来完成的，xml和注解只是告诉框架中的Java代码如何执行

#### 3.1.2 扫描理解

Spring为了知道程序员在哪些地方标记了什么注解，就需要通过扫描的方式，来进行检测。然后根据注解进行后续操作

#### 3.1.3 准备Spring项目和组件

1. 准备项目pom.xml（必需依赖）

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.9.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    ```

2. 准备组件类

    ```java
    //Component组件
    public class CommonComponent {
    }

    //Dao组件
    public class XDao {
    }

    //Controller组件
    public class XController {
    }

    //Service组件
    public class XService {
    }
    ```

#### 3.1.4 组件添加标记注解

1. 组件标记注解和区别
    * spring提供了以下多个注解，可以直接标注在Java类上，将它们定义成Spring Bean

    |注解|说明|
    |-|-|
    |@Component|该注解用于描述Spring中的Bean，它是一个泛化的概念，仅仅表示容器中的一个组件（Bean），并且可以作用在应用的任何层次，例如Service层、Dao层等。使用时只需要将该注解标注在相应类上即可|
    |@Repositiry|该注解用于将数据访问层（Dao层）的类标识为Spring中的Bean，其功能与@Component相同|
    |@Service|该注解通常作用在业务层（Service），用于将业务层的类标识为Spring中的Bean，其功能与@Component相同|
    |@Controller|该注解通常作用在控制层（如SpringMVC的Controller），用于将控制层的类标识为Spring中的Bean，其功能与@Component相同|

   * @Controller、@Service、@Repositiry这三个注解只是在@Compoent注解的基础上起了三个新的名字，其在语法层面没有区别，区分不同的名字只是为了为了便于开发者分辨组件的作用
2. 使用注解标记

    ```java
    //Component组件
    @Component  //<Bean id="component">
    public class CommonComponent {
    }

    //Dao组件
    @Repository
    public class XDao {
    }

    //Controller组件
    @Controller
    public class XController {
    }

    //Service组件
    @Service
    public class XService {
    }
    ```

3. 配置文件确定扫描范围

    * 情况一：基本扫描配置

        ```xml
        <!--1. 普通配置包扫描
            base-package 指定ioc容器去哪里包下查找注解类 -> ioc容器
            一个包或者多个包 org.alan,org.alan.x.x
            指定包，相当于指定了子包内的所有类-->
        <context:component-scan base-package="org.alan"/>
        ```

    * 情况二：指定排除组件

        ```xml
        <!--2. 指定包，但是排除注解-->
        <context:component-scan base-package="org.alan">
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/><!--排除了Repository-->
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/><!--排除了Controller-->
        </context:component-scan>
        ```

    * 情况三：指定扫描组件

        ```xml
        <!--3. 指定包，指定包含注解-->
        <!--use-default-filters="false":指定包的所有注解不生效-->
        <context:component-scan base-package="org.alan" use-default-filters="false">
            <!--只扫描包下的注解-->
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
        </context:component-scan>
        ```

4. 组件BeanName问题

* 使用注解时，每个组件有一个唯一标识，默认情况下首字母小写的类名就是bean的id，例如：ASerivice的bean id为aService
* 第二种情况，在注解旁使用value属性指定（其中value关键字可省略）：

    ```java
    @Service(value="aaa")
    public class XService {
    }

    ```java
    @Service("aaa")//同上，效果相同
    public class XService {
    }
    ```

### 实验二：组件（Bean）作用域和周期方法注解

#### 3.2.1 组件周期方法配置

1. 周期方法概念
    * 我们可以在组件类中定义方法，然后当IoC容器实例化和销毁组件对象的时候进行调用，这两个方法我们称为生命周期方法
    * 类似于Servlet的init/destory方法，我们可以在周期方法完成初始化和释放资源等工作

2. 周期方法声明

    ```java
    @Component
    public class JavaBean {
        //周期方法命名随意，但必须是 public void 没有形参

        @PostConstruct //注解指定初始化方法
        public void init(){
            //初始化逻辑
        }

        @PreDestroy //注解指定销毁方法
        public void destroy(){
            //释放资源逻辑
        }
    }
    ```

#### 3.2.2 组件作用域配置

1. Bean作用域概念
    * \<bean>标签声明Bean，只是将Bean的信息配置给SpringIoC容器
    * 在IoC容器中，这些\<bean>标签对应的信息转成Spring内部BeanDefinition对象，Bean Definition对象内，包含定义的信息（id，class，属性等等）
    * 这意味着，BeanDefinition与“类”概念一样，SpringIoC容器可以根据BeanDifinition对象反射创建多个Bean对象实例
    * 具体创建多少个Bean的实例对象，由Bean的作用域Scope属性指定
2. 作用域可选值

    |取值|含义|创建对象的时机|默认值|
    |-|-|-|-|
    |singleton|在IoC容器中，这个bean的对象始终为单实例|IoC容器初始化时|是|
    |prototype|这个bean在IoC容器中有多个实例|获取bean时|否|

    如果是在WebApplicationContext环境下还会有另外两个作用域（但不常用）

    |取值|含义|创建对象的时机|默认值|
    |-|-|-|-|
    |request|请求范围内有效的实例|每次请求|否|
    |session|会话范围内有效的实例|每次会话|否|

3. 作用域配置

    ```java
    //@Scope(scopeName = ConfigurableBeanFactory.SCOPE_SINGLETON) //单实例
    @Scope(scopeName = ConfigurableBeanFactory.SCOPE_PROTOTYPE) //多实例
    @Component
    public class JavaBean {
        //周期方法命名随意 public void 没有形参
        @PostConstruct
        public void init(){}
        @PreDestroy
        public void destroy(){}
    }
    ```

### 实验三：Bean属性赋值：引用类型自动装配（DI）

### 实验四：Bean属性赋值：基本类型属性赋值（DI）

### 实验五：基于注解+XML方式整合三层架构组件

详情参考：<https://www.bilibili.com/video/BV1AP411s7D7> P33

## 四、基于配置类方式管理Bean

## 五、三种配置方式总结

## 六、整合Spring5-Test5搭建测试环境
