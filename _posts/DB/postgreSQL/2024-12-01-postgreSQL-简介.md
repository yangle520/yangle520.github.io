---
layout: post
title:  PostgreSQL 简介
date:   2024-12-01
categories: 数据库
tag: 数据库
author: YangLe
---



## PostgreSQL 简介



官网：https://www.postgresql.org

管理工具：https://www.pgadmin.org/download/

下载地址：https://www.enterprisedb.com/downloads/postgres-postgresql-downloads



### 1 优势

PostgreSQL 数据库是目前功能最强大的开源数据库，它是最接近工业标准 SQL92 的查询语言，至少实现了 SQL:2011 标准要求的179项主要功能中的160项(注：目前没有哪个数据库管理系统能完全实现 SQL:2011 标准中的所有主要功能)

- **稳定可靠**：PostgreSQL 是唯一能做到数据零丢失的开源数据库。
- **开源省钱**：PostgreSQL 数据库是开源的、免费的，而且使用的是类 BSD 协议，在使用和二次开发上基本没有限制
- **支持广泛**：PostgreSQL 数据库支持大量的主流开发语言，包括 C、C++、Perl、Python、Java、Tcl、PHP 等
- **社区活跃**：基本上每3个越推出一个补丁版本，这意味着已知的 bug 很快会被修复，有应用场景的需求也会及时得到响应



### 2 特征

- **多版本并发控制**：PostgreSQL使用多版本并发控制(MVCC、Multi version concurrency control)系统进行并发控制，该系统向每个用户提供了一个数据库的“快照”，用户在事物内的每个修改，对于其他的用户都不可见，直到该事务成功提交
- **数据类型**：包括文本、任意精度的数值数组、Json数据、枚举数据、XML数据等
- **NoSQL**：Json、JSONB、XML、Hstore 原生支持，甚至 NoSQL 数据库的外部数据包装器
- **数据仓库**：能平滑迁移至同属 PostgreSQL 生态的 GreenPlum、DeepGreen 等，使用 FDW（Foreign data warppers）进行ETL（Extract Transform Load）
- **函数**：通过函数可以在数据库服务器端执行指令程序
- **索引**：用户可以自定义索引方法，或使用内置的 B 树，哈希表 与 GiST索引（Generalized Search Tree）
- **触发器**：触发器是由 SQL 语句查询所触发的事件。
- **规则**：规则（RULE）允许一个查询能被重写，通常用来实现对视图（VIEW）的操作。
- **继承**：PostgreSQL 实现了表继承，一个表可以从0个或者多个其他表继承，而对一个表的查询则可以引用一个表的所有行或者该表的所有行加上它所有的后代表



### 3 数据库对象

三级存储结构：database -> schema -> table

每个PG服务包含多个独立的 database 。每个database 包含多个 schema，每个schema 包含多个表。



#### 3.1 Schema

PostgreSQL 模式Schema 可以看着是一个表的集合。一个Schema可以包含视图、索引、数据类型、函数和操作符等。

使用Schema的优势：

1. 允许多个用户使用一个数据库并且不会互相干扰
2. 数据库对象组织成逻辑组以便更容易管理
3. 第三方应用的对象可以放在独立的schema中，这样他们就不会与其他对象的名称发生冲突



#### 3.2 表

PG的表支持两种强大的功能

1. 继承：一张表可以有父表和子表
2. 创建一张表的同时，系统会自动为此表创建一种对应的自定义数据类型



### 4 命令

#### 4.1 常用命令

| 命令                                        | 说明                            |
| ------------------------------------------- | ------------------------------- |
| psql -h IP -p 端口 -U 用户名 -d 数据库名 -W | 链接指定服务器上的数据库        |
| \?                                          | 所有命令帮助                    |
| \l                                          | 列出所有数据库                  |
| \d                                          | 列出数据库中所有表              |
| \dt                                         | 列出数据库中所有表              |
| \d [table_name]                             | 显示指定表的结构                |
| \di                                         | 列出数据库中所有 index          |
| \dv                                         | 列出数据库中所有 view           |
| \h                                          | sql 命令帮助                    |
| \q                                          | 退出链接                        |
| \c                                          | 显示当前数据库名称和用户        |
| \c [database_name]                          | 切换到指定的数据库              |
| \conninfo                                   | 显示客户端连接信息              |
| \du                                         | 显示所有用户                    |
| \dn                                         | 显示数据库中所有 schema         |
| \encoding                                   | 显示字符集                      |
| select version();                           | 显示版本信息                    |
| \i testdb.sql                               | 执行 SQL 文件                   |
| \x                                          | 扩展展示结果信息                |
| \o /tmp/test.txt                            | 将下一条 SQL 执行结果导入文件中 |



#### 4.2 用户及授权

| 语句                                                         | 含义                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| **create** **user** 用户名 **password** '密码';              | 创建用户                             |
| **alter** **user** 用户名 **set** default_transaction_read_only = **on**; | 设置只读权限                         |
| **grant** **all** **on** **database** 数据库名 **to** 用户名; | 设置可操作的数据库                   |
| **grant** **select** **on** **all** **tables** **in** **schema** **public** **to** 用户名; | 授权                                 |
| **GRANT** **ALL** **ON** **TABLE** public.user **TO** mydata; | 授权 -全部                           |
| **GRANT** **SELECT**, **UPDATE**, **INSERT**, **DELETE** **ON** **TABLE** public.user **TO** mydata_dml; | 授权 - 读写                          |
| **GRANT** **SELECT** **ON** **TABLE** public.user **TO** mydata_qry; | 授权 - 只读                          |
| **revoke** **select** **on** **all** **tables** **in** **schema** **public** **from** 用户名; | 撤回在public模式下的权限             |
| **revoke** **select** **on** **all** **tables** **in** **schema** information_schema **from** 用户名; | 撤回在information_schema模式下的权限 |
| **revoke** **select** **on** **all** **tables** **in** **schema** pg_catalog **from** 用户名; | 撤回在pg_catalog模式下的权限         |
| **revoke** **all** **on** **database** 数据库名 **from** 用户名; | 撤回对数据库的操作权限               |
| **drop** **user** 用户名;                                    | 删除用户                             |



#### 4.3 Schema

如果不创建scheme，并且语句中不写scheme，则默认scheme使用内置的public。

| 语句                                                  | 含义                      |
| ----------------------------------------------------- | ------------------------- |
| **create** **schema** AUTHORIZATION **CURRENT_USER**; | 创建和当前用户同名 schema |
| **create** **schema** 模式名称;                       | 自定义创建模式（schema）  |
| **select** * **from** information_schema.schemata;    | 查看数据库下的所有 schema |



#### 4.4 数据库

| 语句                                                         | 含义                                          |
| ------------------------------------------------------------ | --------------------------------------------- |
| **select** datname **from** pg_database;                     | 查询所有数据库                                |
| **create** **database** 数据库名 owner 所属用户 **encoding** UTF8; | 创建数据库                                    |
| **drop** **database** 数据库名;                              | 删除数据库 <br />注意：删库前需要关闭所有会话 |
| **SELECT** pg_terminate_backend(pg_stat_activity.pid)<br/>**FROM** pg_stat_activity<br/>**WHERE** datname='mydb' **AND** pid<>pg_backend_pid(); | 关闭数据库所有会话                            |



#### 4.5 表

| 语句                                                         | 含义               |
| ------------------------------------------------------------ | ------------------ |
| **create** **table** "t_user" (<br/> "id" bigserial **not** **null**,<br/> "username" varchar (64) **not** **null**,<br/> "password" varchar (64) **not** **null**,<br/> "create_time" timestamp **not** **null** **default** **current_timestamp**,<br/> "update_time" timestamp **not** **null** **default** **current_timestamp**,<br/> **constraint** t_user_pk primary **key** (**id**)<br/>); | 创建表             |
| **select** table_name **from** information_schema.tables **where** table_schema = 'myuser'; | 查询schema中所有表 |
| **comment** **on** **column** "t_user"."id" **is** '主键';   | 添加注释           |
| **alter** **sequence** "t_user_ID_seq" restart **with** 1 **increment** **by** 1; | 创建自增序列       |
| **drop** **index** **if** **exists** "t_user_pkey";<br/>**alter** **table** "t_user" **add** **constraint** "t_user_pkey" primary **key** ("ID"); | 创建主键序列       |
| **drop** **table** **if** **exists** "t_template" **cascade**; | 删除表             |
| **drop** **index** **if** **exists** t_user_username;<br/>**create** **index** t_user_username **on** t_user (username); | 创建索引           |
| **drop** **index** **if** **exists** t_user_username;<br/>**create** **index** t_user_username **on** t_user (username); | 创建唯一索引       |



#### 4.6 查询

```sql
postgres=# \help SELECT
Command:     SELECT
Description: retrieve rows from a table or view
Syntax:
[ WITH [ RECURSIVE ] with_query [, ...] ]
SELECT [ ALL | DISTINCT [ ON ( expression [, ...] ) ] ]
    [ * | expression [ [ AS ] output_name ] [, ...] ]
    [ FROM from_item [, ...] ]
    [ WHERE condition ]
    [ GROUP BY grouping_element [, ...] ]
    [ HAVING condition [, ...] ]
    [ WINDOW window_name AS ( window_definition ) [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
    [ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start [ ROW | ROWS ] ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY ]
    [ FOR { UPDATE | NO KEY UPDATE | SHARE | KEY SHARE } [ OF table_name [, ...] ] [ NOWAIT | SKIP LOCKED ] [...] ]

#from_item 可以是以下选项之一
[ ONLY ] table_name [ * ] [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
```



### 5 扩展



| 扩展        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| PostGIS     | 提供地理空间数据类型与索引支持 （&pgPointCloud 点云，pgRouting 寻路） |
| TimescaleDB | 添加时间序列、持续聚合、分布式、列存储、自动压缩的能力       |
| PGVector    | 添加AI向量、嵌入数据类型支持，以及 ivfflat、hnsw 向量索引    |
| Citus       | 将经典的主从 PG 集群原地改造为水平分片的分布式数据库集群     |
| Hydra       | 添加列式存储与分析能力，提供比肩 ClickHouse 的强力分析能力   |
| ParadeDB    | 添加 ElasticSearch 水准的全文搜索能力与混合检索能力 （&zhparser 中文分词） |
| Apache AGE  | 图数据库扩展，为 PG 添加类 Neo4J 的 OpenCypher 查询支持      |
| PG GraphQL  | 添加原生内建的 GraphQL 查询语言支持                          |
| DuckDB FDW  | 通过PG 直接读写嵌入式分析数据库 DuckDB 文件(&DuckDB CLI 本体) |
| Supabase    | 基于 PG 的开源的 Firebase 替代，提供完整的应用开发存储解决方案 |
| FerretDB    | 基于 PG 的开源 MongoDB 替代，兼容 MongoDB API / 驱动协议     |
| PostgresML  | 使用 SQL 完成经典机器学习算法，调用、部署、训练 AI 模型      |





参考文章：https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MDQ4MTM5NQ==&action=getalbum&album_id=2977746367114936326#wechat_redirect