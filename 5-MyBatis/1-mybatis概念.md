# MyBatis：提高持久层数据处理效率

## 一、MyBatis简介

### 1.1 简介

网站：<https://mybatis.net.cn/>

**什么是 MyBatis？**

* MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 1.2 持久层框架对比

* JDBC
  * SQL夹杂在Java代码中耦合度高，导致硬编码硬伤
  * 维护不易且实际开发需求中SQL有变化，频繁修改的情况多见
  * 代码冗长，开发效率低
* Hibernate和JPA
  * 操作简单，开发效率高
  * 程序中的长难复杂SQL，不容易做特殊优化
  * 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难
  * 反射操作太多，导致数据库性能下降
* MyBatis
  * 轻量级，性能出色
  * SQL和Java代码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
  * 开发效率稍逊于Hibernate，但是完全能够接受

开发效率：Hibernate > MyBatis > JDBC
运行效率：JDBC > MyBatis > Hibernate

### 1.3 快速入门

#### 1.3.1 准备数据模型

```sql
CREATE DATABASE mybatis_example;

USE mybatis_example;

CREATE TABLE t_emp(
  emp_id INT AUTO_INCREMENT,
  emp_name CHAR(100),
  emp_salary DOUBLE(10,5),
  PRIMARY KEY(emp_id)
);

INSERT INTO t_emp(emp_name,emp_salary) VALUES('tom',200.33);
INSERT INTO t_emp(emp_name,emp_salary) VALUES('jerry',666.66);
INSERT INTO t_emp(emp_name,emp_salary) VALUES('andy',777.77);
```

#### 1.3.2 项目搭建和准备

1. 项目搭建
2. 依赖导入

    ```xml
    <dependencies>
      <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis</artifactId>
          <version>3.5.11</version>
      </dependency>
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>8.0.33</version>
      </dependency>
      <dependency>
          <groupId>org.junit.jupiter</groupId>
          <artifactId>junit-jupiter-api</artifactId>
          <version>5.10.2</version>
          <scope>test</scope>
      </dependency>
    </dependencies>
    ```

3. 实体类准备

    ```java
    public class Employee {

        private Integer empId;

        private String empName;

        private Double empSalary;

        //getter setter toString...
    }
    ```

#### 1.3.3 准备Mapper接口和MapperXML文件

![1-mybatis概念0](image/1-mybatis%E6%A6%82%E5%BF%B5/1711098911120.png)  

1. 创建接口EmployeeMapper

    ```java
    public interface EmployeeMapper {

        Employee queryById(Integer id);

        int deleteById(Integer id);
    }
    ```

2. 定义mapper.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- namespace = mapper对应接口的全限定符-->
    <mapper namespace="org.alan.mapper.EmployeeMapper">
        <!--xml方式写sql语句，没有java代码-->
        <!--声明标签写sql语句 crud select insert update delete
            每个标签对应接口的一个方法！  方法的一个实现
            注意：以后的mapper接口不能重载！！！ 因为mapper.xml无法识别！（根据方法名识别）
            -->
        <select id="queryById" resultType="org.alan.pojo.Employee">
            <!--sql语句-->
            select emp_id empId,emp_name empName,emp_salary empSalary
            from t_emp where emp_id = #{id}
        </select>

        <delete id="deleteById">
            delete from t_emp where emp_id = #{id}
        </delete>
    </mapper>
    ```

#### 1.3.4 准备MyBatis配置文件

mybatis框架配置文件：数据库连接信息，性能配置，mapper.xml配置等

习惯上命名为mybatis_config.xml，这个文件名仅仅是建议，可自定义。后面整合Speing之后，这个配置文件可以省略。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/><!--数据库驱动路径-->
        <property name="url" value="${url}"/><!--数据库路径-->
        <property name="username" value="${username}"/><!--数据库用户名-->
        <property name="password" value="${password}"/><!--数据库用户密码-->
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/><!--mapper.xml路径-->
  </mappers>
</configuration>
```

1.3.5 运行测试

```java
@Test
public void test_01() throws IOException {

    //1.读取外部配置文件
    InputStream ips = Resources.getResourceAsStream("mybatis_config.xml");
    //2.创建sqlSessionFactory
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ips);
    //3.根据sqlSessionFactory创建sqlSession(每次业务创建一个，用完就释放)
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //4.获取接口的代理对象（代理技术） 调用代理对象的方法，就会查找mapper接口的方法
    EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
    Employee employee = mapper.queryById(1);
    System.out.println("employee = " + employee);

    //5.提交事务（非DQL）和释放资源
    sqlSession.commit();
    sqlSession.close();
}
```

* SqlSession：代表Java程序和数据库之间的会话。（HttpSession是Java程序和浏览器之间的对话）
