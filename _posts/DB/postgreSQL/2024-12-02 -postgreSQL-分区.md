---
layout: post
title:  PostgreSQL 表分区
date:   2024-12-01
categories: 数据库
tag: 数据库
author: YangLe
---



## PostgreSQL 分区

表分区就是根据分区策略，将数据分散到不同的子表中，并通过父表建立关联关系，从而实现数据物理上分区。

### 1 优点

1. 在某些情况下查询性能能够显著提升，特别是当那些访问压力大的行在一个分区或者少数几个分区时。划分可以取代索引的主导列、减小索引尺寸以及使索引中访问压力大的部分更有可能被放在内存中。 

2. 当查询或更新访问一个分区的大部分行时，可以通过该分区上的一个顺序扫描来取代分散到整个表上的索引和随机访问，可以改善性能 

3. 如果批量操作的需求是在分区设计时就规划好的，则批量加载和删除可以通过增加或者去除分区的方式来完成。执行alter table detach  partition或者使用drop  table 删除一个分区速度快于批量操作。也避免了delete带来的vacuum的开销 

4. 很少使用的数据可以被迁移到便宜且较慢的存储介质上

   

### 2 分区的两种形式

#### 2.1 表继承

表继承是10版本以前的主要使用方式。继承的方式需要使用约束来定义分区和规则，还需要使用触发器将数据路由到合适的分区，用户必须编写和维护这些代码。

表分区就是把逻辑上的一个大表分割成物理上的几块。当表的大小超过了数据库服务器物理内存大小时，建议使用分区表。在使用继承实现分区表时，一般会让父表为空，数据都存在子表中。

**实现步骤**

1. 创建父表，所有的分区都从父表继承。该表中没有数据，不要在其上定义任何检查约束，除非你希望约束所有的分区。同样，在其上定义索引或者唯一约束也没有意义。
2. 创建几个子表，每个都是从主表上继承的。通常这些表不会增加任何字段。我们将把子表称作分区，实际上它们就是普通的PostgreSQL表。
3. 给分区表增加约束，定义每个分区允许的健值。
4. 对于每个分区，在关键字字段上创建一个索引，也可创建其他你想创建的索引。严格来说，关键字字段索引并非必需的，但是在大多数情况下它是很有帮助的。如果你希望关键字值是唯一的，那么应该总是给每个分区创建一个唯一约束或者主键约束。
5. 定义一个规则或者触发器，把对主表的数据插入重定向到合适的分区表中。
6. 确保constraint_exclusion中的配置参数postgresql.conf是打开的。打开后，如果查询中WHERE子句的过滤条件与分区的约束条件匹配，那么该查询会智能地只查询此分区，而不会查询其他分区。

一个子表可以从多个父表继承，这种情况下它拥有所有父表字段的总和，并且子表中定义的字段也会加入其中。如果同一个字段名出现在多个父表中，或者同时出现在父表和子表的定义里，那么这些字段就会被融合，因此子表里只有一个这样的字段。想要融合，字段的数据类型必须相同，否则会报错。融合字段将会拥有其父字段的所有检查约束，并且如果某个父字段存在非空约束，那么融合后的字段也必须是非空的。

```sql
# 创建父表
CREATE TABLE vehiches (
    name text PRIMARY KEY,
    color text,
    weight float,
    area text,
    manufactureid int
);

# 创建子表
CREATE TABLE bikes(
    size float NOT NULL
) INHERITS(vehicles);

CREATE TABLE cars(
    displacement float NOT NULL
) INHERITS(vehicles);

CREATE TABLE trucks(
    load float NOT NULL
) INHERITS(vehicles);

# 只查询父表，不添加 only 会将父表和子表数据一起查出来
SELECT * FROM ONLY vehicles;
# 通过父表删除子表中数据
DELETE FROM vehicles WHERE name LIKE 'truck%';
# 通过父表更新子表记录
UPDATE vehicles SET color='BROWN' WHERE name='truck002';
# 仅更新父表记录
UPDATE ONLY vehicles SET color='BROWN' WHERE name='truck002';
```



#### 2.2 声明式分区

PostgreSQL提供一种方法将表划分成称为分区的片段，被划分的表称为分区表，这种分区方式由分区的方法以及要被用作分区键的列或者表达式列表组成。

当前支持的分区方法是范围（range）、列表（list）、哈希（hash）。

**Range 范围分区**

```sql
# 创建表
CREATE TABLE pkslow_person_r (
    age int not null,
    city varchar not null
) PARTITION BY RANGE (age);  # 指定以年龄作为分区字段 

# 创建分区表 ： 0-10岁、11-20岁、21-30岁、31岁以上 4个分区表
create table pkslow_person_r1 partition of pkslow_person_r for values from (MINVALUE) to (10);
create table pkslow_person_r2 partition of pkslow_person_r for values from (11) to (20);
create table pkslow_person_r3 partition of pkslow_person_r for values from (21) to (30);
create table pkslow_person_r4 partition of pkslow_person_r for values from (31) to (MAXVALUE);

# 插入数据 ：分区表对客户端是无感知的，可直接操作父表
insert into pkslow_person_r(age, city) VALUES (1, 'GZ');
insert into pkslow_person_r(age, city) VALUES (2, 'SZ');
insert into pkslow_person_r(age, city) VALUES (21, 'SZ');
insert into pkslow_person_r(age, city) VALUES (13, 'BJ');
insert into pkslow_person_r(age, city) VALUES (43, 'SH');
insert into pkslow_person_r(age, city) VALUES (28, 'HK');

# 查询所有数据
select * from pkslow_person_r;

# 查询特定分区的数据
select * from pkslow_person_r3;
```



**List** **列表分区**

```sql
# 创建分区的方式不同
CREATE TABLE pkslow_person_l1 PARTITION OF pkslow_person_l FOR VALUES IN ('GZ');
CREATE TABLE pkslow_person_l2 PARTITION OF pkslow_person_l FOR VALUES IN ('BJ');
CREATE TABLE pkslow_person_l3 PARTITION OF pkslow_person_l DEFAULT;
```



**Hash 哈希分区**

```sql
# 创建hash分区表
create table pkslow_person_h1 partition of pkslow_person_h for values with (modulus 4, remainder 0);
create table pkslow_person_h2 partition of pkslow_person_h for values with (modulus 4, remainder 1);
create table pkslow_person_h3 partition of pkslow_person_h for values with (modulus 4, remainder 2);
create table pkslow_person_h4 partition of pkslow_person_h for values with (modulus 4, remainder 3);
```

