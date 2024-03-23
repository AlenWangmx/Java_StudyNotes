# MyBatis基本使用

## 一、向SQL语句传参

### 1.1 mabatis日志输出配置

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

* configuration（配置）
* properties（属性）
* settings（设置）
* typeAliases（类型别名）
* typeHandlers（类型处理器）
* objectFactory（对象工厂）
* plugins（插件）
* environments（环境配置）
  * environment（环境变量）
    * transactionManager（事务管理器）
    * dataSource（数据源）
* databaseIdProvider（数据库厂商标识）
* mappers（映射器）

我们可以在mybatis的配置文件使用settings标签设置，输出运行过程SQL日志！通过查看日志，我们可以判定#{}、${}的输出效果。

* settings设置项

    ![2-MyBatis基本使用0](image/2-MyBatis%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/1711110872996.png)  

* config.xml配置

    ```xml
    <settings>
        <!--开启了mybatis的日志输出，选择使用system进行控制台输出-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    ```

### 1.2 #{}形式

```xml
<!--
    #{key}：占位符 + 赋值 emp_id = ?  ?是赋值
    ${key}：字符串空间 " emp_id = " + id

    推荐使用#，可以防止【注入攻击】的问题
    局限性：?只能替代值的位置，不能替代容器名（标签，列名，sql关键字） emp_id = ? 不能写 ?=?
    
    sql：select * from 表 where 动态列名（从外部传入）${columnName} = 动态的值 #{columnValue}

    总结：传入动态值用 #{key}，传入动态的列名、容器名用${key}
-->
<select id="queryById" resultType="org.alan.pojo.Employee">
    select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_id = #{id}
</select>
```

### 1.3 ${}形式

```xml
<!--
    #{key}：占位符 + 赋值 emp_id = ?  ?是赋值
    ${key}：字符串空间 " emp_id = " + id

    推荐使用#，可以防止【注入攻击】的问题
    局限性：?只能替代值的位置，不能替代容器名（标签，列名，sql关键字） emp_id = ? 不能写 ?=?

    sql：select * from 表 where 动态列名（从外部传入）${columnName} = 动态的值 #{columnValue}

    总结：传入动态值用 #{key}，传入动态的列名、容器名用${key}
-->
<select id="queryById" resultType="org.alan.pojo.Employee">
    select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_id = #{id}
</select>
```

## 二、数据输入

### 2.1 MyBatis总体机制概括

![2-MyBatis基本使用1](image/2-MyBatis%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/1711112828809.png)  

### 2.2 概念说明

这里数据输入具体是指上层方法（例如Service方法）调用Mapper接口时，数据传入的形式
  
* 简单类型：只包含一个值的数据类型
  * 基本数据类型：int、byte、short、double、...
  * 基本数据类型的包装类型：Integer、Character、Double、...
  * 字符串类型：String
* 复杂类型
  * 实体类类型：Employee、Department、...
  * 集合类型：List、Set、Map、...
  * 数组类型：int[]、String[]、...
  * 复合类型：List\<Employee>、实体类中包含集合

### 2.3 单个简单类型参数

1. 接口中定义传入单个简单类型的方法

    ```java
        //根据id删除员工信息
        int deleteById(Integer id);

        //根据工资查询员工信息
        List<Employee> querySalary(Double salary);
    ```

2. 接口对应mapper.xml配置

    ```xml
    <!--
        场景一：传入的单个简单类型 key随便写 一般情况下推荐使用参数名
    -->
    <delete id="deleteById">
        delete from t_emp where emp_id = #{random}
    </delete>

    <select id="queryBySalary" resultType="org.alan.pojo.Employee">
        select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_salary = #{salary}
    </select>
    ```

### 2.4 实体类型参数

1. 接口中定义传入实体类型的方法

    ```java
    //插入员工数据【实体对象】
    //Employee为自定义的实体类
    int insertEmp(Employee employee);
    ```

2. 接口对应mapper.xml配置

    ```xml
     <!--
        场景二：传入的是实体对象 key该如何写?
            key = 对象的属性名（对应类中的成员属性），使用#{}传入
    -->
    <insert id="inertEmp">
        insert into t_emp (emp_name, emp_salary) values (#{empName},#{empSalary});
    </insert>
    ```

### 2.5 零散的简单类型数据

1. 接口中定义传入多个简单类型的方法

    ```java
    //根据员工姓名和工资查询员工信息
    List<Employee> queryByNameAndSalary(String name, Double salary);
    ```

2. 如何配置mapper.xml？

   * #{}中的key【不可以】随便写，【也不可以】按照方法的形参名写
   * 有两个方案：
     * 方案1（推荐）：由注解指定（@Param注解），在方法的参数列表中指定多个简单参数的key，key = @Param("value值")
     * 方案2：mybatis默认机制，形参列表从左到右的key依次为arg0、arg1、arg2...，或者param1、param2、...

3. 方案一(推荐)

   * 在方法的参数列表中使用@Param注解

       ```java
       //根据员工姓名和工资查询员工信息
       //使用@Param注解指定形参value值为a、b
       List<Employee> queryByNameAndSalary(@Param("a") String name, @Param("b") Double salary);
       ```

   * mapper.xml配置

       ```xml
       <!--配置中填写key为a、b-->
       <select id="queryByNameSalary" resultType="org.alan.pojo.Employee">
       select emp_id empId,emp_name empName,emp_salary empSalary
           from t_emp where emp_name = #{a} and emp_salary = #{b}
       </select>
       ```

4. 方案二

    ```xml
    <!--mybatis默认机制，形参列表从左到右的key依次为arg0、arg1、arg2...，或者param1、param2、...-->
    <select id="queryByNameSalary" resultType="org.alan.pojo.Employee">
       select emp_id empId,emp_name empName,emp_salary empSalary
           from t_emp where emp_name = #{arg0} and emp_salary = #{arg1}
    </select>
    <!--效果同上-->
    <select id="queryByNameSalary" resultType="org.alan.pojo.Employee">
       select emp_id empId,emp_name empName,emp_salary empSalary
           from t_emp where emp_name = #{param1} and emp_salary = #{param2}
    </select>
    ```

### 2.6 Map类型参数

1. 接口中定义传入多个简单类型的方法

    ```java
    //插入员工数据，传入的是一个map
    //定义map中的key(name=员工的名字，salary=员工的薪水)
    int insertEmpMap(Map data);
    ```

2. 接口对应mapper.xml配置

    ```xml
    <!--场景四：传入map如何指定key的值？
        key = map的key即可
    -->
    <insert id="inertEmpMap">
        insert into t_emp (emp_name, emp_salary) values (#{name},#{salary});
    </insert>
    ```

## 三、数据输出

### 3.1 输出概述

* 数据输出总体上有两种形式：
  * 增删改查操作返回的受影响行数：直接使用int或long类型接收即可
  * 查询操作的查询结果
* 我们需要做的是，指定查询的输出数据类型即可
* 并且在插入场景下，实现主键数据回显示

### 3.2 单个简单类型和定义别名

1. Mapper接口中的抽象方法

    ```java
    public interface EmployeeMapper {

        //如果是dml语句（插入 修改 删除）
        int deleteById(Integer id);

        //指定输出类型 查询语句
        //根据员工的id查询员工的姓名
        String queryNameById(Integer id);

        //根据员工的id查询员工的工资
        Double querySalaryById(Integer id);

        //根据id查询员工对象（Employee类）
        Employee queryById(Integer id); //此为【实体类型】，用作【定义别名】实例
    }
    ```

2. SQL语句

    ```xml
     <!--DML-->
    <delete id="deleteById">
        delete from t_emp where emp_id = #{id}
    </delete>

    <!--
        场景1：返回单个简单类型如何指定 resultType的写法返回值的数据类型
            resultType语法：
                1.类的全限定符
                2.别名简称
                    mybatis提供了72种默认的别名,这些都是常用的Java数据类型
                        基本数据类型 int double ... -> _int _double
                        包装数据类型 Integer Double ... -> int integer double
                        集合容器类型 Map List HashMap ... -> 全小写即可

                扩展：如果没有提供别名，需要自己定义或者写类的全限定符
                如何定义？
                    1.单独定义
                    <typeAliases>
                        <typeAlias alias="别名" type="类全限定符"/>
                    </typeAliases>

                    2.批量定义：将包下的所有类给予别名，首字母小写的类名别名就是别名
                        <typeAliases>
                            <package name="org.alan.pojo"/>
                        </typeAliases>
                    3.在【批量注解的同时】，在类上单独注解@Alias("别名")可给一个类单独起别名
    -->

    <select id="queryNameById" resultType="string">
        select emp_name from t_emp where emp_id = #{id}
    </select>

    <select id="querySalaryById" resultType="_double">
        select emp_salary from t_emp where emp_id = #{id}
    </select>

    
    <select id="queryById" resultType="employee"><!--resultType的值也可以填Employee类的全限定符-->
        select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_id = ${id}
    </select>
    ```

    Mybatis内部给常用的数据类型设定了很多别名。以int类型为例，可以写的名称有：int、integer、Integer、java.lang.Integer、Int、INT、INTEGER等等。具体参考：<https://mybatis.net.cn/configuration.html#typeAliases>

3. 自定义别名（\<typeAliases>标签）

* 如果没有提供别名，需要在mybatis_config.xml中使用\<typeAliases>标签自己定义（或者写类的全限定符）

    ```xml
    <!--单独定义-->
    <typeAliases>
        <typeAlias type="org.alan.pojo.Employee" alias="employee"/>
    </typeAliases>
    ```

    ```xml
    <!--批量定义：将包下的所有类给予别名，首字母小写的类名别名就是别名-->
    <typeAliases>
        <package name="org.alan.pojo"/>
    </typeAliases>
    ```

* Tip：在【批量注解的同时】，在类上单独注解@Alias("别名")可给一个类单独起别名（不常用）

### 3.3 返回实体类对象

1. 定义Mapper接口中的抽象方法

    ```java
    //根据id查询员工对象
    Employee queryById(Integer id); 
    ```

2. xml配置SQL语句

    ```xml
    <!--场景2：返回单个自定义实体类型
    resultType：返回值类型即可

    默认要求：
        查询时，返回单各实体类型，要求列名和类的属性名一致！
        这样才可以进行实体类的属性映射

    但是！可以进行设置支持java驼峰式名称自动映射
        emp_id -> empId == empId
    -->
    <select id="queryById" resultType="org.alan.pojo.Employee">
        select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_id = ${id}
    </select>
    <!--或者写别名（详情见3.2），效果同上-->
    <select id="queryById" resultType="employee">
        select emp_id empId,emp_name empName,emp_salary empSalary
        from t_emp where emp_id = ${id}
    </select>
    ```

3. 拓展：驼峰命名自动映射

    ![2-MyBatis基本使用2](image/2-MyBatis%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/1711126048501.png)  

    ```xml
    <settings>
        <!--在settings标签下开启驼峰命名自动映射 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```

    完成以上设置后，sql语句可简化为以下语句

    ```xml
        <select id="queryById" resultType="employee">
        select * from t_emp where emp_id = ${id}
    </select>
    ```

### 3.4 返回Map类型

适用于SQL查询返回的各个字段综合起来并不和任何一个现有的实体类对应，没法封装到实体类对象中。能够封装成实体类类型的，就不使用Map类型

1. Mapper接口的抽象方法

    ```java
    /**
     * 当输出的数据没有对应的类接收，使用Map接收
     */
    //查询部门工资最高的员工和他的工资
    Map<String,Object> selectEmpNameAndMaxSalary();
    ```

2. SQL语句

    ```xml
    <select id="selectEmpNameAndMaxSalary" resultType="map">
        SELECT emp_name 员工姓名,emp_salary 员工工资,(SELECT AVG(emp_salary) FROM t_emp) 部门平均工资
        FROM t_emp
        WHERE emp_salary=(SELECT MAX(emp_salary) FROM t_emp)
    </select>
    ```

3. junit测试

    ```java
    @Test
    public void test_01() throws IOException {

        //1.读取外部配置文件
        InputStream ips = Resources.getResourceAsStream("mybatis_config.xml");

        //2.创建sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ips);

        //3.获取SqlSession对象 【自动开启JDBC】
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //4.获取代理mapper对象
        EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
        Map<String, Object> map = mapper.selectEmpNameAndMaxSalary();
        System.out.println("map = " + map);

        //5.管理资源或者提交事务
        sqlSession.close();
    }
    ```

    测试结果：

    ![2-MyBatis基本使用0](image/2-MyBatis%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/1711173404941.png)  

### 3.5 返回List类型

1. Mapper接口的抽象方法

    ```java
    //6.查询工资高于传入值的员工姓名集合 >200
    List<String> queryNamesBySalary(Double salary);

    //7.查询全部员工信息
    List<Employee> queryAll();
    ```

2. SQL语句

    * 返回值是集合，resultType不需要指定集合类型，只需要指定<>内的泛型即可

    ```xml
    <!--场景4：返回集合类型该如何指定
        切记：返回值是集合，resultType不需要指定集合类型，只需要指定<>内的泛型即可
        为什么？
            mybatis -> ibatis -> selectOne 单个 | selectList 集合 -> selectOne调用了selectList
        -->
    <select id="queryNamesBySalary" resultType="string">
        select emp_name from t_emp where emp_salary > #{salary}
    </select>

    <select id="queryNamesBySalary" resultType="employee">
        select * from t_emp;
    </select>
    ```

### 3.6 返回主键值

#### 3.6.1 自增长类型主键

1. 接口中的抽象方法

    ```java
    //8.员工插入
    int insertEmp(Employee employee);
    ```

2. SQL语句

    ```xml
    <!--场景5：主键回显 获取插入的主键
        1.自增长型主键回显 mysql auto_increment
            //员工插入
            int insertEmp(Employee employee)

            useGeneratedKeys="true" 获取数据库自动增长的主键值
            keyColumn="emp_id" 主键列的值
            keyProperty="empId" 接收主键列值的属性
    -->
    <insert id="insertEmp" useGeneratedKeys="true" keyColumn="emp_id" keyProperty="empId">
        insert into t_emp (emp_name,emp_salary)
        value(#{empName},#{empSalary});
    </insert>
    ```

3. junit测试（拓展：自动提交事务）

    ```java
    @Test
    public void test_02() throws IOException {
        //1.读取外部配置文件
        InputStream ips = Resources.getResourceAsStream("mybatis_config.xml");
        //2.创建sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ips);
        //3.获取SqlSession对象 【自动开启JDBC】
        /*
        开启事务 openSession 自动开启事务，但不会自动提交事务 sqlSession.commit();
                openSession(true) 自动开启事务，也会自动提交事务 不需要sqlSession.commit();
        */
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取代理mapper对象
        EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);

        //创建员工对象，set需要插入的员工信息，EmpId为自增长不需要插入
        Employee employee = new Employee();
        employee.setEmpSalary(888.0);
        employee.setEmpName("WFJ");

        System.out.println(employee.getEmpId());//输出提交事务前的主键值
        System.out.println("-----------------------");
        int emp = mapper.insertEmp(employee);//openSession(true)自动提交事务

        System.out.println(employee.getEmpId());//提交事务后的主键值
        System.out.println("emp = " + emp);

        //5.管理资源或者提交事务
    //        sqlSession.commit();
        sqlSession.close();

    }
    ```

    实验结果：提交事务前的主键值为null，提交事务后的主键值为10

    ![2-MyBatis基本使用1](image/2-MyBatis%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/1711177762375.png)  

    注意：MyBatis是将自增主键的值设置到实体类对象中，而不是以Mapper接口方法返回值的形式返回

#### 3.6.2 非自增长类型主键

对于不支持自增型主键的数据库（如Oracle）或者字符串类型主键，则可以使用selectKey子元素：selectKey元素将会首先运行，id会被设置，然后插入语句会被调用

* 使用selectKey帮助插入UUID作为字符串类型主键示例：

    ```xml
    <insert id="insertUser" parameterType="User">
            <!--插入之前，先指定一段sql语句，生成一个主键值
                order="before|after" sql语句是在插入语句之前还是之后执行
                resultType = 返回值类型
                keyProperty = 查询结果给哪个属性赋值
            -->
            <selectKey keyProperty="id" resultType="string" order="BEFORE">
                SELECT UUID() as id
            </selectKey>
            INSERT INTO `user`(id, username, pwd) 
            VALUES (
                #{id}
                #{username}, 
                #{pwd}
            )
    </insert>
    ```

  * 在上例中，我们定义了一个insertUser的插入语句来将User对象插入到user表中。并使用selectKey来查询UUID并设置到id字段中
  * 通过keyProperty属性来指定查询到的UUID赋值给对象中的id属性，而resultType属性指定了UUID的类型为java.lang.String（mybatis别名string）。
  * 注意：上例中将selectKey放在了插入语句前，这是因为MySQL在insert语句中只支持一个select子句，而selectKey中查询UUID的语句就是一个select子句，因此我们需要将其放在前面。
  * 最后，将User对象插入到user表中时，我们直接使用对象中的id属性来插入主键值。

使用这种方式，我们可以方便地插入UUID作为字符串类型主键。当然，还有其他插入方法可以使用，如使用Java代码生成UUID并在类中显示设置值等。需要根据应用场景和需求选择合适的插入方式。

### 3.7 实体类属性和数据库字段对应关系

1. 别名对应

    将字段的别名设置成和实体类一致

    ```xml
    <select id="queryById" resultType="teacher">
        <!--根据MySQL语法给每个字段单独取与实体类属性名相同的别名-->
        select t_id tId,t_name tName from teacher where t_id = #{tId}
    </select>
    ```

2. 全局配置自动识别驼峰式命名规则

    在MyBatis全局配置文件加入如下配置

    ```xml
    <settings>
        <!--开启驼峰命名自动映射 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```

    SQL语句中可以不使用别名

    ```xml
    <select id="queryById" resultType="teacher">
        select * from teacher where t_id = #{tId}
    </select>
    ```

3. 使用resultMap

    ```xml
    <!--           
        方案3：resultMap 自定义映射，可深层次映射 （与resultType二选一）
                  resultType 按照规则自动映射 按照是否开启驼峰式映射，自己映射属性和别名，只能映射一层结构!
                  深层次的对象结构无法映射，多表查询的时候结果无法映射
                  Order             -               数据库
                    oderId            ->              order_id
                    orderName         ->              order_name
                    OrderItem orderItem
                                item_id  !->(无法映射)  item_id
    -->

    <!--
        声明resultMap标签，自己定义映射规则
            id标识 -> select resultMap="标识"
            type -> 具体的返回值类型 全限定符 | 集合只写泛型
                <id 主键映射
                <result 非主键映射
    -->
    <resultMap id="tMap" type="teacher">
        <id column="t_id" property="tId"/>
        <result column="t_name" property="tName"/>
    </resultMap>

    <select id="queryById" resultMap="tMap">
        select * from teacher where t_id = #{tId}
    </select>
    ```

## 四、CRUD强化练习

详情参考：<https://www.bilibili.com/video/BV1AP411s7D7?p=84>

## 五、mapperXML标签总结

MyBatis的真正强大的地方在于它的语句映射。依靠其强大的XML映射文件，编写SQL事务变得极其简单且清晰易懂。相比于直接编写JDBC代码，MyBatis节省了大量不必要的代码编写。MyBatis致力于减少使用成本，让用户能更专注于SQL代码。
