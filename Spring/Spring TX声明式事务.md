# Spring TX声明式事务

## 一、声明式事务概念

### 1.1 编程式事务

编程式事务是指手动编写程序来管理事务，即通过编写代码的方式直接控制事务的提交和回滚。在Java中，通常使用事务管理器（如Spring中的PlatformTransactionManager）来实现编程式事务

编程式事务的主要优点有灵活性高，可以按照自己的需求来控制事务的粒度、模式等等。但是，编写大量的事务控制代码容易出现问题，对代码的可读性和可维护性有一定影响

```java
Connection conn = ...;

tyr{
    //开启事务
    conn.setAutoCommit(false);
    //核心业务
    //业务代码
    //提交事务
    conn.commit();

}catch(Exception e){

    //回滚事务
    conn.rollback();

}finally{

    //释放数据库连接
    conn.close();

}
```

编程式的实现方式存在的缺陷：

* 细节没有被屏蔽：具体操作过程中，所有细节都需要程序员自己完成，比较繁琐
* 代码复用性不高：如果没有有效抽取出来，每次实现功能都需要自己编写代码，代码就没有得到复用

### 1.2 声明式事务

* 声明式事务是指使用注解或XML配置的方式来控制事务的提交和回滚
* 开发者只需要添加配置即可，具体事务的实现由第三方框架实现，避免我们直接进行事务操作
* 使用声明式事务可以将事务的控制和业务逻辑分离开来，提高代码的可读性和可维护性
* 区别
  * 编程式事务需要手动编写代码来管理事务
  * 而声明式事务可以通过配置文件或注解来控制事务

### 1.3 Spring事务管理器

1. Spring声明式事务对应依赖
   * spring-tx：包含声明式事务实现的基本规范（事务管理器规范接口和事务增强等等）
   * spring-jdbc：包含DataSource方式事务管理器实现类DataSourceTransactionManager
   * spring-orm：包含其它持久层框架的事务管理器实现类，例如：Hibernate/Jpa等

2. Spring声明式事务对应事务管理器接口

    ![Spring TX声明式事务0](image/Spring%20TX%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1/1710921026395.png)  

    我们现在要使用的事务管理器是org.springframework.jdbc.datasource.DataSourceTransactionManager，整合JDBC方式、JdbcTemplate方式、Mybatis方式的事务实现

    DataSourceTransactionManager类中的主要方法：
      * doBegin()：开启事务
      * doSuspend()：挂起事务
      * doResume()：恢复挂起的事务
      * doCommit()：提交事务
      * doRollback()：回滚事务

## 二、基于注解的声明式事务

### 2.1 准备工作

1. 准备项目依赖

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
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>6.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>6.1.4</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>6.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>6.1.4</version>
        </dependency>
    ```

2. 外部配置文件

    ```properties
    alan.url=jdbc:mysql://localhost:3306/studb
    alan.drive=com.mysql.cj.jdbc.Driver
    alan.username=root
    alan.password=202077
    ```

3. config配置类

    ```java
    @Configuration
    @ComponentScan("org.alantx")
    @PropertySource("classpath:jdbc.properties")
    public class javaConfig {

        @Bean
        public DataSource dataSource(@Value("${alan.drive}") String driver,
                                    @Value("${alan.url}") String url,
                                    @Value("${alan.username}") String username,
                                    @Value("${alan.password}") String pwd){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setDriverClassName(driver);
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(pwd);
            return dataSource;
        }

        @Bean
        public JdbcTemplate jdbcTemplate(DataSource dataSource){
            JdbcTemplate jdbcTemplate = new JdbcTemplate();
            jdbcTemplate.setDataSource(dataSource);
            return jdbcTemplate;
        }
    }
    ```

4. 准备dao/service层

    ```java
    //Dao层
    @Repository
    public class StudentDao {

        @Autowired
        private JdbcTemplate jdbcTemplate;

        public void updateNameById(String name,Integer id){
            String sql = "update students set name = ? where id = ?;";
            int rows = jdbcTemplate.update(sql, name, id);
        }

        public void updateAgeById(Integer age,Integer id){
            String sql = "update students set age = ? where id = ?;";
            int rows = jdbcTemplate.update(sql,age,id);
        }
    }

    //Service层
    @Service
    public class StudentService {

        @Autowired
        private StudentDao studentDao;

        public void changeInfo(){
            studentDao.updateAgeById(100,1);
            System.out.println("-----------------");
            studentDao.updateNameById("test1",1);
        }
    }
    ```

5. 测试环境搭建

    ```java
    @SpringJUnitConfig(javaConfig.class)
    public class SpringTest {

        @Autowired
        private StudentService studentService;

        @Test
        public void test(){
            studentService.changeInfo();
        }
    }
    ```

6. 测试结果：id为1的条目的name，age中的数据更改为test1，100

    ![Spring TX声明式事务1](image/Spring%20TX%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1/1710997565426.png)  

### 2.2 基本事务控制

选择对应事务管理器加入到ioc容器，Spring声明式事务给我们提供了对各管理器实现，需要哪种加入到ioc容器即可（mybatis jdbc Template -> DataSourceTM，hibernate -> HibernateTM，...）

1. 配置事务管理器
   * 数据库相关配置(Config)

    ```java
    @Bean
    public TransactionManager transactionManager(DataSource dataSource){
        //内部进行事务操作
        DataSourceTransactionManager dataSourceTransactionManager
                = new DataSourceTransactionManager();
        //需要链接池对象
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }
    ```

2. 使用注解指定方法添加到事务

    ```java
        /**
     * 添加事务
     *  @Transactional
     *  位置：方法|类上
     *  方法：当前方法有事务
     *  类上：类下的所有方法都有事务
     */

    @Transactional
    public void changeInfo(){
        studentDao.updateAgeById(88,1);
        System.out.println("-----------------");
        studentDao.updateNameById("test2",1);
    }
    ```

### 2.3 事务属性：只读

1. 只读介绍
   * 对一个【查询操作】来说，如果我们把它设置成只读，就能够明确告诉数据库，这个操作不涉及写操作。这样数据库就能针对查询操作来进行优化
2. 设置方式

    ```java
    //readOnly = true 把当前事务设置为只读，默认是false
    @Transactional(readOnly = true)
    ```

3. 针对DML动作（插入、更新、删除）设置只读模式
    * 会抛出以下异常：java.sql.SQLException

4. @transactional注解放在类上

    ```java
    @Transactional//类中所有方法都添加事务
    @Service
    public class StudentService {

        @Autowired
        private StudentDao studentDao;

        /**
        * 添加事务
        *  @Transactional
        *  位置：方法|类上
        *  方法：当前方法有事务
        *  类上：类下的所有方法都有事务
        *
        *  1. 只读模式
        *      只读模式可以提查询事务的效率！ 推荐事务中只有查询代码，使用只读模式
        *      默认：
        *      解释：一般情况下，都是通过类添加注解添加事务
        *      类下所有方法都有事务
        *      查询方法可以通过再次添加注解，设置为只读模式，提高效率
        */
        public void changeInfo(){
            studentDao.updateAgeById(88,1);
            System.out.println("-----------------");
            studentDao.updateNameById("test2",1);
        }

        @Transactional(readOnly = true)//单独设置只读覆盖类设置
        public void getStudentInfo(){
            //查询 没有添加事务
            //获取学生信息 查询数据库 不修改
        }
    }
    ```

### 2.4 事务属性：超时时间

1. 需求
   * 事务在执行过程中，有可能因为遇到某些问题，导致程序卡住，从而长时间占用数据库资源。而长时间占用资源，大概率是因为运行出现了问题（可能是Java程序或MySQL数据库或网络连接等等）
   * 此时这个很可能出问题的程序应该被【回滚】，撤销它已做的操作，事务结束，把资源让出来，让其它正常程序可以执行
   * 概括来说就是一句话：超时回滚，释放资源
2. 设置超时时间

    ```java
    /**
     *      2.超时时间
     *          默认：永远不超时 -1
     *          设置timeout = 时间 单位秒 超时时间，就会回滚事务和释放异常
     */
    @Transactional(readOnly = false,timeout = 3)
    public void changeInfo(){
        studentDao.updateAgeById(88,1);
        System.out.println("-----------------");
        studentDao.updateNameById("test2",1);
    }
    ```

3. 若事务超时
   * 抛出TransactionTimedOutException异常

### 2.5 事务属性：事务异常

1. 事务在遇到Exception异常中，默认只有遇到RuntimeException异常才会回滚，而IOException异常不会回滚

2. 指定异常回滚（rollbackFor），指定异常不回滚（noRollbackFor，使用场景较少）
   * 当属性rollbackFor = Exception.class时，所有Exception异常都回滚

    ```java
    /**
     *   3.指定异常回滚和指定异常不回滚
     *      默认情况下，指定发生运行时异常事务才会回滚
     *      我们可以指定Exception异常来控制所有异常都回滚
     *      rollbackFor = Exception.class
     *      noRollbackFor = 回滚异常范围内，控制某个异常不回滚（使用场景较少）
     */
    @Transactional(readOnly = false,timeout = 3,rollbackFor = Exception.class)
    public void changeInfo() throws FileNotFoundException {
        studentDao.updateAgeById(88,1);
        new FileInputStream("xxxx");
        System.out.println("-----------------");
        studentDao.updateNameById("test2",1);
    }
    ```

### 2.6 事务属性：事务隔离级别

1. 事务隔离级别
   * 数据库事务的隔离级别是指在多个事务并发执行时，数据库系统为了保证数据一致性所遵循的规定。常见的隔离级别包括：
     * 读未提交（Read Uncommitted）：事务可以读取未被提交的数据，容易产生脏读、不可重复读和幻读等问题。实现简单但不太安全，一般不用。
     * 读已提交（Read Committed）：事务只能读取已经提交的数据，可以避免脏读问题，但可能引发不可重复读和幻读。
     * 可重复读（Reportable Read）：在一个事务中，相同的查询将返回相同的结果，不管其它事务对数据做了什么修改。可以避免脏读和不可重复读，但仍有可能出现幻读的问题
     * 串行化（Serializable）：最高的隔离级别，完全禁止了开发，只允许一个事务执行完毕之后才能执行另一个事务。可以避免以上所有问题，但效率较低，不适用于高并发场景。

        ![Spring TX声明式事务2](image/Spring%20TX%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1/1711009688437.png)  

    不同的隔离级别适用于不同的场景，需要根据实际业务需要进行选择和调整

2. 事务隔离级别设置

    ```java
    /**
     *   4.隔离级别设置
     *      推荐设置第二个隔离级别
     *      isolation = Isolation.READ_COMMITTED
     */
    @Transactional(readOnly = false,timeout = 3,rollbackFor = Exception.class
    ,isolation = Isolation.READ_COMMITTED)
    public void changeInfo() throws FileNotFoundException {
        studentDao.updateAgeById(88,1);
        System.out.println("-----------------");
        studentDao.updateNameById("test2",1);
    }
    ```

### 2.7 事务属性：事务传播级别

1. 事务传播行为要研究的问题

    ![Spring TX声明式事务3](image/Spring%20TX%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1/1711011038475.png)  

    * 代码举例

        ```java
        @Transactional
        public void MethodA(){
            // ...
            Method();
            //...
        }

        //在被调用的子方法中设置传播行为，代表如何处理调用的事务！是加入还是成为新事务
        @Transactional(propagation = Propagation.REQUIRES_NEW)
        public void MethodB(){

        }
        ```

2. propagation属性

    @Transactional注解通过propagation属性设置事务的传播行为。它的默认值为：

    ```java
    Propogation propagation() default Propagation.REQUIRED;
    ```

    propagation属性的可选值由org.springframework.transaction.annotation.Propagation枚举类提供：

    |名称|含义|
    |-|-|
    |REQUIRED（默认值）|当前方法必须工作在事务中；如果当前线程上有已经开启的事务可用，那么就在这个事务中运行；如果当前线程上没有已经开启的事务，那么就自己开启新事物，在新事务中运行；所以当前方法有可能和其他方法共用事务；在共用事务的情况下，当前方法会因为其他方法回滚而受连累|
    |REQUIRED_NEW|当前方法必须工作在事务中；不管当前线程上是否已经有已经开启的事务，都要开启新事务，在新事务中运行，不会和其它方法共用事务，避免被其他方法连累|

3. 测试

    * 声明两个独立的修改事务

        ```java
        /*
        声明两个独立修改数据库的事务业务方法
        */
        @Transactional(propagation = Propagation.REQUIRED)
        public void changeAge(){
            studentDao.updateAgeById(99,1);
        }

        @Transactional(propagation = Propagation.REQUIRED)
        public void changeName(){
            studentDao.updateNameById("test2",1);
            int i = 1/0;
        }
        ```

    * 声明一个整合业务的方法

        ```java
        @Service
        public class TopService {

            @Autowired
            private StudentService studentService;

            @Transactional
            public void topService(){
                studentService.changeAge();
                studentService.changeName();
            }
        }
        ```

    * 添加传播行为测试

        ```java
        @SpringJUnitConfig(javaConfig.class)
        public class SpringTxTest {
            
            @Autowired
            private StudentService studentService;
            
            @Autowired
            private TopService topService;
            
            @Test
            public void test(){
                topService.topService();
            }
        }
        ```

    * 测试结果：子事务changeName()异常，子事务changeAge()未异常，但依然与异常的事务一同回滚，最终未修改数据库。若子事务gropagation属性设置为REQUIRED_NEW，则未出现异常的子事务不与出现异常的子事务一同回滚，未出现异常的子事务正常完成修改数据库操作。

4. 其他传播行为值（了解即可）

    ![Spring TX声明式事务4](image/Spring%20TX%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1/1711013393552.png)  
