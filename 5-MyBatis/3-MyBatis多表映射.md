# MyBatis多表映射

## 一、多表映射概念

1. 多表查询结果映射思路：

    ResuultMap多层映射

2. 实体类设计方案

    多表关系：（双向查看）

    * 一对一：人和身份证号
    * 一对多：用户和用户订单
    * 多对多：老师和学生，部门和员工

    实体类设计关系（查询）：（单向查看）

    * 对一：订单对应用户式对一关系
      * 实体类设计：对一关系下，类只要包含单个对方对象类型属性即可
    * 例如：

        ```java
        public class Customer {

            private Integer customerId;

            private String customerName;

        }

        public class Order {

            private Integer orderId;

            private String orderName;

            private Integer customerId;

            //一个订单对应一个客户 对一
            //客户对象 装对应的客户信息
            private  Customer customer;
        }
        ```

    * 对多：用户对应的订单，老师对应的学生或者学生对应的讲师都是对多关系
      * 实体类设计：对多关系下，类只要包含单个对方类型集合属性即可
    * 例如：

        ```java
        public class Customer {

            private Integer customerId;

            private String customerName;

            //一个客户对应多个订单，体现对多关系
            private List<Order> orderList;

        }

        public class Order {

            private Integer orderId;

            private String orderName;

            private Integer customerId;

            private  Customer customer;
        }
        ```

    多表结果实体类设计小技巧：

    * 对一，属性中包含对方对象
    * 对多，属性中包含对方对象集合
    * 无论是多少张表联查，实体类设计都是两两考虑

3. 多表映射案例准备

    **数据库：**

    ```sql
    CREATE TABLE t_customer (
        customer_id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        customer_name CHAR(100)
    );

    CREATE TABLE t_order (
        order_id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        order_name CHAR(100),
        customer_id INT
    );

    INSERT INTO t_customer (customer_name) VALUES ('c01');

    INSERT INTO t_corder (order_name,customer_id) VALUES ('o1',1);
    INSERT INTO t_corder (order_name,customer_id) VALUES ('o2',1);
    INSERT INTO t_corder (order_name,customer_id) VALUES ('o3',1);
    ```

    实际开发过程中，一般不给数据库表设置外键约束,原因是避免调试不便。一般都是功能开发完成，再加外键约束检查是否有bug

    **实体类设计：**

    ```java
    @Data //lombook -> Getter|Setter
    public class Order {

        private Integer orderId;

        private String orderName;

        private Integer customerId;

        //一个订单对应一个客户 对一
        //客户对象 装对应的客户信息
        private  Customer customer;
    }

    @Data //lombook -> Getter|Setter
    public class Customer {

        private Integer customerId;

        private String customerName;

        //一个客户对应多个订单
        private List<Order> orderList;
    }
    ```

## 二、对一映射

1. 需求说明

    根据id查询订单，以及订单关联的用户的信息

2. OrderMapper接口

    ```java
    public interface OrderMapper {
        //根据id查询 订单信息 和 订单对应的客户
        Order queryOrderById(Integer id);
    }
    ```

3. OrderMapper.xml配置文件

    在“对一”关联关系中，使用association关键词接收对象数据，使用javaType指定对象实体类

    ```xml
        <!--自定义映射关系，定义嵌套对象的映射关系-->
    <resultMap id="orderMap" type="order">
        <!--第一层属性 order对象-->
        <!--order的主键选择id标签-->
        <id column="order_id" property="orderId"/>
        <result column="order_name" property="orderName"/>
        <result column="customer_id" property="customerId"/>
        <!--对象属性赋值 association
            property：对象属性名
            javaType：对象类型
        -->
        <association property="customer" javaType="customer">
            <id column="customer_id" property="customerId"/>
            <result column="customer_name" property="customerName"/>
        </association>
    </resultMap>
    
    <!--Order queryOrderById(Integer id);-->
    <select id="queryOrderById" resultMap="orderMap">
        <!--根据id查询 订单信息 和 订单对应的客户-->
        SELECT * FROM t_order tor JOIN t_customer tur
        ON tor.customer_id = tur.customer_id
        WHERE tor.order_id=#{id};
    </select>
    ```

4. junit测试

    ```java
    public class MyBatisTest {

        private SqlSession sqlSession;

        @BeforeEach
        public void init() throws IOException {
            sqlSession = new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis_config.xml"))
                    .openSession(true);
        }

        @AfterEach
        public void clean(){
            sqlSession.close();
        }

        @Test
        public void testToOne(){
            OrderMapper mapper = sqlSession.getMapper(OrderMapper.class);
            Order order = mapper.queryOrderById(1);
            System.out.println(order);
            System.out.println(order.getCustomer());
        }

    }
    ```

    测试结果：

    ![3-MyBatis多表映射0](image/3-MyBatis%E5%A4%9A%E8%A1%A8%E6%98%A0%E5%B0%84/1711256954563.png)  

## 三、对多映射

1. 需求说明

    查询客户和客户关联的订单信息

2. CustomerMapper接口

    ```java
    public interface CustomerMapper {
        //查询所有客户信息以及客户对应的订单信息
        List<Customer> queryList();
    }
    ```

3. CustomerMapper.xml文件

    在“对多”的关联关系中，使用collection属性接收集合数据，使用ofType指定集合中的泛型

    ```xml
        <resultMap id="customerMap" type="customer">
        <id column="customer_id" property="customerId"/>
        <result column="customer_name" property="customerName"/>
        <!--<collection 给对多的对象赋值-->
        <!--给集合属性赋值
            property 集合属性名
            ofType 集合的泛型类型
        -->
        <collection property="orderList" ofType="order">
            <id column="order_id" property="orderId"/>
            <result column="order_name" property="orderName"/>
            <result column="customer_id" property="customerId"/>
            <!--customer要不要赋值？不需要-->
        </collection>
    </resultMap>
    
    <select id="queryList" resultMap="customerMap">
        SELECT * FROM t_order tor JOIN t_customer tur
        ON tor.customer_id = tur.customer_id;
    </select>
    ```

4. junit测试

    ```java
    public class MyBatisTest {

        private SqlSession sqlSession;

        @BeforeEach
        public void init() throws IOException {
            sqlSession = new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis_config.xml"))
                    .openSession(true);
        }

        @AfterEach
        public void clean(){
            sqlSession.close();
        }

        @Test
        public void testToMany(){
            CustomerMapper mapper = sqlSession.getMapper(CustomerMapper.class);
            List<Customer> customers = mapper.queryList();
            System.out.println("customers = " + customers);

            for (Customer customer : customers) {
                List<Order> orderList = customer.getOrderList();
                System.out.println("orderList = " + orderList);
            }
        }
    }
    ```

    测试结果：

    ![3-MyBatis多表映射1](image/3-MyBatis%E5%A4%9A%E8%A1%A8%E6%98%A0%E5%B0%84/1711257581342.png)  

## 四、多表映射总结

### 4.1 多表映射优化

|settings属性|属性含义|可选值|默认值|
|-|-|-|-|
|autoMappingBehavior|指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。|NONE, PARTIAL, FULL|PARTIAL|

我们可以将autoMappingBehavior属性设置为FULL，进行多表resultMap映射的时候，可以省略符合列和属性命名映射规则（列名=属性名，或者开启驼峰映射也可以自定映射）的result标签

修改mybatis_config.xml：

```xml
    <settings>
        <!--resultMap标签 有没有嵌套都会自动帮我们映射result标签的属性和列-->
        <setting name="autoMappingBehavior" value="FULL"/>
    </settings>
```

修改Mapper.xml（以上例中CustomerMapper.xml为例）

```xml
    <!--resultMap中的多层result都可不做单独映射-->

    <resultMap id="customerMap" type="customer">
        <id column="customer_id" property="customerId"/>
        <!--<result column="customer_name" property="customerName"/>-->
        <!--<association 给对一的对象赋值-->
        <!--给集合属性赋值
            property 集合属性名
            ofType 集合的泛型类型
        -->
        <collection property="orderList" ofType="order">
            <id column="order_id" property="orderId"/>
            <!--<result column="order_name" property="orderName"/>
            <result column="customer_id" property="customerId"/>-->
            <!--customer要不要赋值？不需要-->
        </collection>
    </resultMap>
    
    <select id="queryList" resultMap="customerMap">
        SELECT * FROM t_order tor JOIN t_customer tur
        ON tor.customer_id = tur.customer_id;
    </select>
```

junit测试结果仍然与开启多层映射前相同：

![3-MyBatis多表映射2](image/3-MyBatis%E5%A4%9A%E8%A1%A8%E6%98%A0%E5%B0%84/1711258323745.png)  

### 4.2 多表映射总结

|关联关系|配置项关键词|所在文件和具体位置|
|-|-|-|
|对一|association标签/javaType属性/property属性|Mapper配置文件中的resultMap标签内|
|对多|collection标签/ofType属性/property属性|Mapper配置文件中的resultMap标签内|
