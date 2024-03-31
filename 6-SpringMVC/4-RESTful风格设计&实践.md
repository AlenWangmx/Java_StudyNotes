# RESTful风格设计&实践

## 一、RESTful风格概述

### 1.1 RESTful风格简介

### 1.2 RESTful风格特点

1. 每一个URI代表一种资源（URI是名词）
2. 客户端使用 GET、POST、PUT、DELETE 4个表示操作方式的动词对服务端进行操作；GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源；
3. 资源的表现形式是XML或者JSON；
4. 客户端与服务端之间的交互在请求之间是无状态的，从客户端到服务端的每个请求都必须包含理解请求所必须的信息

### 1.3 RESTful风格设计规范

## 二、RESTful风格实践

### 2.1 需求分析

* 数据结构：User{id 唯一标识，name用户名，age用户年龄}
* 功能分析
  * 用户数据分页展示功能（条件：page页数 默认为1，size 每页数量 ，默认为10）
  * 用户保存功能
  * 根据id查询用户详情功能
  * 根据id更新用户数据功能
  * 根据id删除用户数据功能
  * 多条件模糊查询用户功能（条件：keyword模糊关键字，page页数 默认为1，size 每页数量 ，默认为10）

### 2.2 RESTful风格接口设计

1. 接口设计

    ![4-RestFul风格设计&实践0](image/4-RestFul%E9%A3%8E%E6%A0%BC%E8%AE%BE%E8%AE%A1%26%E5%AE%9E%E8%B7%B5/1711612272989.png)  

2. 问题讨论

    ![4-RestFul风格设计&实践1](image/4-RestFul%E9%A3%8E%E6%A0%BC%E8%AE%BE%E8%AE%A1%26%E5%AE%9E%E8%B7%B5/1711612752787.png)

### 2.3 后台接口实现

1. 准备实体类

    ```java
    @Data
    public class User {

        private Integer id;

        private String name;

        private Integer age;
    }
    ```

2. 接口设计

     ```java
    @RestController
    @RequestMapping("user")
    public class UserController {

        @GetMapping()
        public List<User> page(@RequestParam(required = false,defaultValue = "1") int page,
                            @RequestParam(required = false,defaultValue = "10")int size){
            System.out.println("page = " + page + ", size = " + size);
            return null;
        }

        @PostMapping
        public User save(@RequestBody User user){
            return user;
        }

        @GetMapping("{id}")
        public User detail(@PathVariable Integer id){
            return null;
        }

        @PutMapping
        public User update(@RequestBody User user){
            return user;
        }

        @DeleteMapping("{id}")
        public User delete(@PathVariable Integer id){
            return null;
        }

        @GetMapping("search")
        public List<User> search(String keywork,
                                @RequestParam(required = false,defaultValue = "1") int page,
                                @RequestParam(required = false,defaultValue = "10") int size){
            return null;
        }
    }
    ```
