---
title: SpringBoot 缓存技术浅谈
catalog: true
date: 2019-01-22 16:28:19
subtitle:
header-img: sp.jpg
tags:
- 服务端开发
---
## 一、为什么要缓存
随着时间的积累，应用的使用用户不断增加，数据规模也越来越大，往往数据库查询操作会成为影响用户使用体验的瓶颈，数据库的访问越少越好，此时使用缓存往往是解决这一问题非常好的手段之一。

在开发中，如果相同的查询条件去频繁查询数据库, 会给数据库带来很大的压力。因此，我们需要对查询出来的数据进行缓存,这样客户端只需要从数据库查询一次数据，然后会放入缓存中，以后再次查询时可以从缓存中读取，这能大大减少访问时间。

本文来介绍 SpringBoot 来简单整合缓存，使用 SpringBoot + MyBatis + MySQL来进行数据库操作。

## 二、项目依赖与配置
**1、pom.xml 引入依赖**
为测试方便，除了 cache 依赖之外，还引入了 mysql、jdbc、mybatis 的依赖。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```
**2、application.properties配置文件**
项目连接数据库的基本配置。

```
spring.datasource.username=root
spring.datasource.password=12345678
spring.datasource.url=jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# mybatis使用驼峰命名法
mybatis.configuration.map-underscore-to-camel-case=true
```

## 三、数据库与实体类创建
**1、数据库创建**
简单起见，这里创建一个 `student` 表，里面只有 3 个字段：`student_id`，`student_name`，`student_age`。

![数据库字段](https://tva1.sinaimg.cn/large/006y8mN6gy1g8ggw0psblj30xc03s74z.jpg)

在表中加几条数据：

![数据](https://tva1.sinaimg.cn/large/006y8mN6gy1g8ggwc6rryj30do06sweb.jpg)

**2、对应的实体类**
```
import lombok.Data;

@Data
public class Student {

    private Integer studentId;
    
    private String studentName;
    
    private Integer studentAge;
}
```

## 三、Mapper层和Service层编写
**1、Mapper层对数据CRUD操作编写**
```
import org.apache.ibatis.annotations.*;

@Mapper
public interface StudentMapper {

    @Select("select * from student where student_id = #{id}")
    Student getStudentById(Integer id);

    @Delete("delete from student where student_id = #{id}")
    int deleteStudentById(Integer id);

    // 设置主键 student_id 自增
    @Options(useGeneratedKeys = true, keyProperty = "student_id")
    @Insert("insert into student(student_name) values(#{studentName})")
    int insertStudent(Student student);

    @Update("update student set student_name = #{studentName} where student_id = #{studentId}")
    int updateStudent(Student student);
}
```
**2、缓存注解**

所谓的缓存，就是将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，**不用调用方法**。

SpringBoot 的自动配置类 `CacheAutoConfiguration` 给容器中注册了一个 CacheManager：`ConcurrentMapCacheManager`，可以获取和创建 `ConcurrentMapCache` 类型的缓存组件；他的作用将数据保存在 `ConcurrentMap` 中；CacheManager 管理多个 Cache 组件的，对缓存的真正CRUD操作在Cache组件中。

|       注解  |   作用  |
| :-------- | :---------- |
| @Cacheable | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存  | 
| @CacheEvict | 清空缓存  | 
| @CachePut| 保证方法被调用，又希望结果被缓存。  | 
| @EnableCaching | 开启基于注解的缓存  | 

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class StudentServies {

    @Autowired
    private StudentMapper studentMapper;

    // 将方法的返回值缓存在value(stu)中，key默认为方法传入的参数；可类比数据库，在stu表中新增一条 key-返回值 记录
    // 当调用该方法时，会先去stu表中查找key，若有则不会进入方法；若没有，则新增一条 key-返回值 记录
    @Cacheable(value = "stu", key = "#id")
    public Student getStu(Integer id) {
        Student student = studentMapper.getStudentById(id);
        System.out.print("查询id = " + id + ": ");
        System.out.println(student);
        return student;
    }

    // 根据key更新stu中的缓存，若stu中没有这个key，则会在stu中新增缓存，无论在stu中是否有这个key，该方法都会执行
    @CachePut(value = "stu", key = "#student.studentId")
    public Student updateStu(Student student) {
        int s1 = studentMapper.updateStudent(student);
        System.out.println("更新");
        System.out.println(s1);
        return student;
    }
}
```
**注意：**
1. `@Cacheable` 和 `@CachePut` 标注的方法的返回值必须相同。
2. 不同 `value` 中的数据是不同的，即使 `key` 是相同的。
 
`@Cacheable`/`@CachePut`/`@CacheEvict` 主要的参数：
|       注解  |   作用  | 
| :-------- | :---------- | 
| value | 缓存的名称，在 Spring 配置文件中定义，必须指定至少一个  
| key | 缓存的 key，如果为空则按照方法的所有参数进行组合，如果指定则要按照 SpEL 表达式编写 | 
| condition| 缓存的条件，可以为空，使用 SpEL 编写，返回 true 或false，只有为 true 才进行缓存/清除缓存，在调用方法前后都能判断  | 
| allEntries | 是否清空所有缓存内容，缺省为 false，如果指定为 true，则方法调用后将立即清空所有缓存  | 
| beforeInvocation| 是否在方法执行前就清空，缺省为 false，如果指定为 true，则在方法还没有执行的时候就清空缓存， 缺省情况下，如果方法执行抛出异常，则不会清空缓存  | 
| unless | 用于否决缓存的，不像condition，该表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。条件为true不会缓存，fasle才缓存  | 

例如：
```
// 根据studentId来进行缓存，当 studentAge > 1 时才缓存，或者返回值的 personId < 0 时不缓存
@CachePut(value = "stu", key = "#student.studentId", condition = "#student.studentAge > 1", unless = "#result.personId < 0")
    public Student updateStu(Student student) {
        int s1 = studentMapper.updateStudent(student);
        System.out.println("更新");
        System.out.println(s1);
        return student;
    }
```