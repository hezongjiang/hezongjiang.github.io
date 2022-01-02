---
title: SpringBoot 之 JPA 详解
catalog: true
date: 2018-09-19 11:29:14
subtitle:
header-img: java.png
tags:
- 服务端开发
---

## 一、什么是JPA？什么是Mybatis？
ORM 框架的本质是简化编程中操作数据库的编码，发展到现在基本上就剩两家了，一个是宣称可以不用写一句 SQL 的 Hibernate，一个是可以灵活调试动态 SQL 的 Mybatis，两者各有特点，在企业级系统开发中可以根据需求灵活使用。

Hibernate 特点就是所有的 SQL 都用 Java 代码来生成，不用跳出程序去写（看）SQL，有着编程的完整性，发展到最顶端就是 Spring Data Jpa 这种模式了，基本上根据方法名就可以生成对应的 SQL 了。

>JPA(Java Persistence API) Java 持久层 API，是 JDK 5.0 注解或 XML 描述对象-关系表的映射关系，并将运行期内的实体对象持久化到数据库中。

![MySQL](https://upload-images.jianshu.io/upload_images/2708793-a83e1fa13ca3ca6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Mybatis 初期使用比较麻烦，需要各种配置文件、实体类、dao层映射关联、还有一大推其它配置。当然 Mybatis 也发现了这种弊端，初期开发了 Generator 可以根据表结果自动生产实体类、配置文件和 dao 层代码，可以减轻一部分开发量；后期也进行了大量的优化可以使用注解了，自动管理 dao 层和配置文件等，发展到最顶端就是今天的这种模式了 Mybatis-spring-boot-starter 就是 Springboot + Mybatis 可以完全注解不用配置文件，也可以简单配置轻松上手。

但是到底使用哪种呢？这里还是推荐是用 JPA。可能有人会说 JPA 只支持简单的 SQL 查询，遇到复杂的还是需要用 Mybatis。确实，这是一个理由，但是在实际开发中，JPA 已经能满足绝大部分需求了，除非项目特别复(qi)杂(pa)，不然还是推荐使用 JPA。

## 二、JPA 基本使用
1、MAVEN 配置
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
2、数据库配置
```
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=12345678
spring.datasource.url=jdbc:mysql://localhost:3306/myuser?characterEncoding=utf-8&useSSL=false
spring.jpa.show-sql=true
```

2、建立实体类声明
```
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;
import javax.persistence.*;

@Entity // 实体类注解
@Data // lombok 插件，自动给属性生成 getter 和 setter 方法
//@Table(name = "users") 如果实体类名与表面不相同用 @Table 注解
public class User {

    @Id // 主键注解
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 主键自动递增注解
    private Integer userId;

    private String userName;

    @JsonIgnore // 若一个属性，不需要返回给前端，用 @JsonIgnore 注解
    private Integer userGender;

    private Integer userAge;
 
    private String email;

    @Transient // 若一个属性，不是数据库中的字段，用 @Transient 注解
    private String userInfo;
}
```
3、建立 JpaRepository 接口
* JpaRepository 是一个口接口，没有提供任何方法，是一个标记接口
* 实现了 JpaRepository 接口就会被 Spring IOC 容器识别为 Repository Bean，会被纳入IOC容器中
```
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

// 新建一个接口，继承于 JpaRepository<实体类, 主键类型>
public interface UserRepository extends JpaRepository<User, Integer> {

    // 根据 user_name 查询，
    User findByUserName(String userName);

    // 根据性别，分页查询
    Page<User> findByUserGender(Integer gender, Pageable pageable);

    // 根据用户名和性别查找
    List<User> findByUserNameAndUserGender(String userName, Integer gender);

    // 查找年龄小于 age 的 User
    List<User> findByUserAgeLessThan(Integer age);

    // 查找年龄大于 age 的 User
    List<User> findByUserAgeGreaterThan(Integer age);

    // 查找年龄介于 [minAge，maxAge] 的 User
    List<User> findByUserAgeBetween(Integer minAge, Integer maxAge);

    // 查找名字包含 userName 的 User
    List<User> findByUserNameContaining(String userName);

    // 查找以 userNamePrefix 开头的 User
    List<User> findByUserNameStartingWith(String userNamePrefix);

    // 查找以 userNameSuffix 结尾的 User
    List<User> findByUserNameEndingWith(String userNameSuffix);
}
```
当然，还有一些最基本的 CRUD 就**不**用写了，因为 JpaRepository 中已经有了，直接调用就可以了。

至此，JPA 最基本的使用就完成了。

你可能会说，不对吧，接口没有实现啊。对！不需要实现，这样就已经可以用了。

……
……
……
……
……
……

那么 JPA 是通过什么规则来根据方法名生成 SQL 语句查询的呢？ 

其实JPA在这里遵循 Convention over configuration（约定大约配置）的原则，遵循 Spring 以及 JPQL 定义的方法命名。Spring提供了一套可以通过命名规则进行查询构建的机制。这套机制会把方法名首先过滤一些关键字，比如 find…By，read…By，query…By，count…By 和 get…By。

系统会根据关键字将命名解析成 2 个子语句，第一个 By 是区分这两个子语句的关键词。这个 By 之前的子语句是查询子语句（指明返回要查询的对象），后面的部分是条件子语句。如果直接就是 findBy… 返回的就是定义 Respository 时指定的领域对象集合，同时 JPQL 中也定义了丰富的关键字：and、or、Between 等等。

方便到不可相信，如果在 IntelliJ IDEA 还可以有提示，根本不需要记住命名规则，基本上 IDEA 给你的提示就足够了。

## 三、JPA 高级使用
除了上面的基本使用，在实际工作中，还是有很多无法满足我们的开发，这时就可以用 @Query 注解来写 SQL。

接上面的例子：
1、使用 SQL 查询
```
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
 
public interface UserRepositoiry extends JpaRepository<User, Integer> { 
 
	// 可以使用@Query注解在其value属性中写JPA语句灵活查询
	@Query("SELECT u FROM User u WHERE u.userId = (SELECT MAX(u2.id) FROM User u2)")
	Person getMaxUserId();
 
	// 在@Query注解中使用占位符
	@Query(value = "SELECT u FROM User u where u.userName = ?1 and u.email = ?2")
	List<Person> queryUserNameAndEmail(String name, String email);
 
	// 使用命名参数传递参数
	@Query(value = "SELECT u FROM User u where p.userName = :name")
	List<Person> queryUserName(@Param("name") String name);
 
	// SpringData可以在参数上添加%
	@Query("SELECT u FROM User p WHERE p.userName LIKE %?1%")
	List<Person> queryUserNameLike(String name);
 
	// SpringData可以在参数上添加%
	@Query("SELECT u FROM User u WHERE p.userName LIKE %:name%")
	List<Person> queryUserNameLike2(@Param("name")String name);

    @Query("SELECT u.username FROM User u WHERE u.username LIKE CONCAT('%',:username,'%')")
    List<String> findUsersWithPartOfName(@Param("username") String username);
 
	// 在@Query注解中添加nativeQuery=true属性可以使用原生的SQL查询
	@Query(value="SELECT count(1) FROM user", nativeQuery=true)
	long getTotalRow();
}
```

2、关联查询
从两张表中查找数据
```
public interface FloorContentRepos extends JpaRepository<FloorContent,String>{
    public Page<FloorContent> findByFloor_IdAndIsDeleteOrderByShowIndexAsc(String floorId,boolean b, Pageable pageable);
}
从例子中就可以看出JPA关联查询主要在“_”这个符号的使用，下面来给大家具体的介绍一下这个符号到底代表什么含义。
首先findBy是必须写的，表示使用JPA规则进行查询。
如果查询的是本张表中的内容，例如查询本张表中的name字段就可以这么写：findByName()。
如果查询的是楼层中的name字段就可以这么写：findByFloor_Name()。
如果是既要查询本张表中的name字段，也要查询楼层中的name字段，就可以这么写：findByFloor_NameAndName()。
从上面的案例就可以看出可以在findBy后面添加要关联的实体类，然后在实体类后面写上“_”，"_"符号后面是添加关联表的字段而不是本身表的字段，这点要记住。如何还想关联更多的表可以在后面添加：And+表名字+“_”+表中要查询的字段。或者只是想关联本身的查询字段可以在后面添加：And+查询的字段。
千万不要写错了，写错的话运行都运行不起来的。所以写的时候要多看看是否符合规则。
```