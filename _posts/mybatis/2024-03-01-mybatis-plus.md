---
layout: post
title:  MyBatis-Plus介绍
date:   2024-03-01 00:00:00 +0800
categories: 框架
tag: MyBatis 
---



* content
{:toc}




MyBatis的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

仓库：https://github.com/baomidou/mybatis-plus

文档：https://baomidou.com/



### 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

### 支持数据库

MySQL，Oracle，DB2，H2，HSQL，SQLite，PostgreSQL，SQLServer，Phoenix，Gauss ，ClickHouse，Sybase，OceanBase，Firebird，Cubrid，Goldilocks，csiidb，informix，TDengine，redshift

### 框架结构

![图片]({{ '/styles/images/mybatis/mybatis-plus-framework.jpg' | prepend: site.baseurl }})



## 注解

### @TableName

- 描述：表名注解，标识实体类对应的表
- 使用位置：实体类

```java
@TableName("sys_user")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

属性：

| 属性             | 默认值 | 描述                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| value            | ""     | 表名                                                         |
| schema           | ""     | schema                                                       |
| keepGlobalPrefix | false  | 是否保持使用全局的 tablePrefix 的值（当全局 tablePrefix 生效时） |
| resultMap        |        | xml 中 resultMap 的 id（用于满足特定类型的实体类对象绑定     |
| autoResultMap    | false  | 是否自动构建 resultMap 并使用（如果设置 resultMap 则不会进行 resultMap 的自动构建与注入），只适合个别字段 设置了 typeHandler 或 jdbcType 的情况 |
| excludeProperty  | {}     | 需要排除的属性名                                             |

说明：

因为 MP 底层是 MyBatis，所以 MP 只是帮您注入了常用 CRUD 到 MyBatis 里，注入之前是动态的（根据您的 Entity 字段以及注解变化而变化），但是注入之后是静态的（等于 XML 配置中的内容）



###  @TableId

- 描述：主键注解
- 使用位置：实体类主键字段

```java
@TableName("sys_user")
public class User {
    @TableId
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

属性：

| 属性  | 默认值      | 描述                                                         |
| ----- | ----------- | ------------------------------------------------------------ |
| value | ""          | 主键字段名                                                   |
| type  | IdType.NONE | 指定主键类型<br />AUTO：数据库ID自增<br />NONE：无状态，未设置，跟随全局<br />INPUT：insert 前自行 set 主键值<br />ASSIGN_ID：为Null时，自动填充，雪花算法生成<br /><br />ASSIGN_UUID：为Null时，自动填充，UUID.replace("-","") |

### @TableField

- 描述：字段注解（非主键）

```java
@TableName("sys_user")
public class User {
    @TableId
    private Long id;
    @TableField("nickname")
    private String name;
    private Integer age;
    private String email;
}
```

属性：

| 属性             | 默认值                   | 描述                                                         |
| ---------------- | ------------------------ | ------------------------------------------------------------ |
| value            | ""                       | 数据库字段名                                                 |
| exist            | true                     | 是否为数据库表字段                                           |
| condition        | ""                       | 字段 where 实体查询比较条件                                  |
| update           | ""                       | 字段 update set 部分注入, 该注解优于 el 注解使用<br/>例1：@TableField(.. , update="%s+1") 其中 %s 会填充为字段 输出 SQL 为：update 表 set 字段=字段+1 where ...<br/>例2：@TableField(.. , update="now()") 使用数据库时间 输出 SQL 为：update 表 set 字段=now() where ... |
| insertStrategy   | FieldStrategy.DEFAULT    | 当insert操作时，该字段拼接insert语句时的策略                 |
| updateStrategy   | FieldStrategy.DEFAULT    | 当更新操作时，该字段拼接set语句时的策略                      |
| whereStrategy    | FieldStrategy.DEFAULT    | 字段验证策略之 where: 表示该字段在拼接where条件时的策略      |
| fill             | FieldFill.DEFAULT        | 字段自动填充策略                                             |
| select           | true                     | 是否进行 select 查询                                         |
| keepGlobalFormat | false                    | 是否保持使用全局的 format 进行处理                           |
| jdbcType         | JdbcType.UNDEFINED       | JDBC 类型 (该默认值不代表会按照该值生效)                     |
| typeHandler      | UnknownTypeHandler.class | 类型处理器 (该默认值不代表会按照该值生效)                    |
| numericScale     | ""                       | 指定小数点后保留的位数                                       |

FieldStrategy枚举：

- IGNORED：忽略判断，直接拼接到SQL里，等于 ALWAYS
- ALWAYS：总是加入SQL，无论字段值是否为NULL
- NOT_NULL：非NULL判断
- NOT_EMPTY：非空判断(只对字符串类型字段)
- DEFAULT：1. 在全局里代表 NOT_NULL。2. 在注解里代表 跟随全局
- NEVER：不加入 SQL

FieldFill枚举：

- DEFAULT：默认不处理
- INSERT：插入时填充字段
- UPDATE：更新时填充字段
- INSERT_UPDATE：插入和更新时填充字段

### @Version

- 描述：乐观锁注解、标记 `@Version` 在字段上

### @EnumValue

- 描述：普通枚举类注解(注解在枚举字段上)

```java
@Getter
@AllArgsConstructor
public enum SourceEnum implements BaseEnum<String> {
    ZY("zy", "智运"),
    DZ("dz", "大宗");
    
    @EnumValue
    private final String code;
    @JsonValue
    private final String name;
}

@Data
@EqualsAndHashCode(callSuper = true)
public class SyncData extends BaseEntity {

	@TableField(value = "source")
	private SourceEnum source;

}
```





### @TableLogic

- 描述：表字段逻辑处理注解（逻辑删除）

属性：

| 属性   | 默认值 | 描述             |
| ------ | ------ | ---------------- |
| value  | ""     | 默认逻辑未删除值 |
| delval | ""     | 逻辑删除值       |

### @KeySequence

- 描述：序列主键策略 `oracle`

属性：

| 属性   | 默认值       | 描述                                                         |
| ------ | ------------ | ------------------------------------------------------------ |
| value  | ""           | 序列名                                                       |
| dbType | DbType.OTHER | 数据库类型，未配置默认使用注入，IKeyGenerator 实现，多个实现必须指定 |

### @InterceptorIgnore

- `value` 值为 `1` | `yes` | `on` 视为忽略，例如 `@InterceptorIgnore(tenantLine = "1")`
- `value` 值为 `0` | `false` | `off` | `空值不变` 视为正常执行。

### @OrderBy

- 描述：内置 SQL 默认指定排序，优先级低于 wrapper 条件查询

| 属性 | 默认值          | 描述                     |
| ---- | --------------- | ------------------------ |
| asc  | **false**       | 默认倒序，设置 true 顺序 |
| sort | Short.MAX_VALUE | 数字越小越靠前           |

