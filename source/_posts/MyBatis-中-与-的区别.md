---
title: 'MyBatis 中 # 与 $ 的区别'
catalog: true
date: 2019-10-30 20:00:35
subtitle:
header-img: timg.jpg
tags: 服务端开发
---
刚开始用 MyBatis 时，就发现有的地方用 `#`，有的地方用 `$`，但一直都没在意，直到程序出 Bug 了，才去了解。在项目中有个点击排序的功能调试了许久，终寻因，总结之。 

需求是这样的，页面有个 table，有一列的上下箭头可点击并排序。对于这种需求，我的 mybatis.xml 的 SQL 配置写成了如下：
```
<if test="map.nameSort!=null and map.nameSort!=''"> 
　　ORDER BY columnName #{map.nameSort} 
</if>
```
`nameSort` 即前端传的排序方式，`asc` 或 `desc`。

预期的输出应该是这样的：`ORDER BY columnName desc`。

但是，真正跑起来时，排序的效果一直没出现，经常一番查找，发现是 MyBatis 的 `#{}` 传值的问题，它将 SQL 语句编译成了如下：

`ORDER BY columnName 'desc'` 或者 `ORDER BY columnName 'asc'`

这样，`desc` 或 `asc` 就成了字符串，而不是关键字，SQL 语句的意思就全变了。

排序没效果的问题找到原因了，解决之，MyBatis 提供了另一种绑定参数的方式 `${param}`，将 SQL 配置改为：`ORDER BY columnName ${map.nameSort}`

这样一来，MyBatis 会直接将 `nameSort` 的值加入 SQL 中，不会转义。正确结果：`ORDER BY columnName desc`

最后，对于 MyBatis 中 `#` 和 `$` 绑定参数的区别做个总结：

1. `#{}` 将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。如：`order by #{id}`，如果传入的值是 `111`，那么解析成 SQL 时的值为 `order by "111"`  ，如果传入的值是 `abc`，则解析成的 SQL 为 `order by "abc"`。

2. `${}` 将传入的数据直接显示生成在 SQL 中。如：`order by ${id}`，如果传入的值是 `111`，那么解析成 SQL 时的值为 `order by 111`，如果传入的值是 `abc`，则解析的 SQL 为 `order by abc`。

3. `#{}` 方式能够很大程度防止 SQL 注入。

4. `${}` 方式无法防止 SQL 注入。

5. `${}` 方式一般用于传入数据库对象，例如传入表名，需要注意的是如果传入表名，一定要用 `${}`，而不能用 `#{}`，因为 `#{}` 会给表名加上引号，如 `"tableName"`，SQL 中表名不能加引号（可以加反引号）。

ps：在使用 MyBatis 中还遇到 `<![CDATA[]]>` 的用法，在该符号内的语句，将不会被当成字符串来处理，而是直接当成sql语句，比如要执行一个存储过程。