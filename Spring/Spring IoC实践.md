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

## 二、基于XML配置方式组件管理

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

* 组件周期方法设置
  * 我们可以在组件类中定义方法，然后当IoC容器实例化和销毁组件对象的时候进行调用！这两个等待我们称为生命周期方法
  * 类似于Servlet的init/destory方法，我们可以在周期方法完成初始化和释放资源等工作

#### 2.4.1 周期方法声明

#### 2.4.2 周期方法配置

### 2.5 实验四：高级特性：FactoryBean特性和使用

### 2.6 实验六：基于XML方式整合三层架构组件

## 三、基于注解方式管理Bean

## 四、基于配置类方式管理Bean

## 五、三种配置方式总结

## 六、整合Spring5-Test5搭建测试环境
