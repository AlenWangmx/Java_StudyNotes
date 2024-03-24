# MyBatis动态语句

## 一、动态语句需求和简介

经常遇到很多按照【多查询条件】进行查询的情况，比招聘网站的职位搜索等。其中经常出现【很多条件不取值】的情况，后台应该如何完成最终的sql语句？

![4-MyBatis动态语句0](image/4-MyBatis%E5%8A%A8%E6%80%81%E8%AF%AD%E5%8F%A5/1711268590732.png)  

【动态SQL】便是Mabatis高效解决这一问题的强大特征之一！

## 二、if和where标签*

## 三、set标签

## 四、trim标签（了解）

使用trim标签控制条件部分两端是否包含某些字符

* prefix属性：指定要动态添加的前缀
* suffix属性：指定要动态添加的后缀
* prefixOverrides属性：指定要动态去掉的前缀，使用“|”分隔有可能的多个值
* suffixOverrides属性：指定要动态去掉的后缀，使用“|”分隔有可能的多个值

## 五、choose/when/otherwise

## 六、foreach标签*

## 七、sql片段

抽取重复的SQL片段

```xml
<sql id="mySelectSql">
    select emp_id,emp_name,emp_salary,emp_genger from t_emp;
</sql>
```

引用已抽取到的SQL片段

```xml
<!--使用include标签引用声明的SQL片段-->
<include refid="mySelectSql"/>
```
