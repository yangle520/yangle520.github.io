---
layout: post
title:  Apache Commons Pool
date:   2024-03-13 00:00:00 +0800
categories: Apache-Commons
tag: java类库
author: YangLe
---



**Apache Commons Pool** 开源软件库提供了一个对象池 API 和许多对象池实现。

对象池是一种享元模式的实现，常用于各种连接池的实现。 比如我们常见的数据库连接池 **DBCP**，Redis 客户端 **Jedis** 等都依赖 Commons-Pool。

Commons-Pool 作为对象池工具包，支持对象的管理、跟踪和监控，并且支持自定义池化对象来扩展对象管理的行为，如果有相关需求可以使用



### 简介

**Commons-Pool 主要有三个角色**：

**PooledObject**：**池化对象**，用于包装实际的对象，提供一些附件的功能。如 Commons-Pool 自带的 DefaultPooledObject 会记录对象的创建时间，借用时间，归还时间，对象状态等，PooledSoftReference 使用 Java 的软引用来持有对象，便于 JVM 内存不够时回收对象。当然我们也可以实现 PooledObject 接口来定义我们自己的对象包装器。

**PooledObjectFactory**：**对象工厂**，ObjectPool 对于每个对象的核心操作会代理给 PooledObjectFactory。需要一个新实例时，就调用 makeObject 方法。

需要借用对象时会调用 activateObject 方法激活对象，并且根据配置情况决定是否验证对象有效性，通过 validateObject 方法验证。

归还对象时会调用 passivateObject 方法钝化对象。

需要销毁对象时候调用 destroyObject 方法。

PooledObjectFactory 必须是线程安全的。

**ObjectPool** ：**对象池接口**，用于管理池中的所有对象，对于每个对象的操作会代理给 ObjectFactory。ObjectPool 有多个实现，GenericObjectPool 提供了多种配置选项，包括限制空闲或活动实例的数量、在实例处于池中空闲时将其逐出等。从版本 2 开始，GenericObjectPool 还提供了废弃实例跟踪和删除功能。SoftReferenceObjectPool 可以根据需要增长，但允许垃圾收集器根据需要从池中逐出空闲实例。



### 使用方式

首先我们创建一个对象用于测试，对象构造函数使用随机延迟模拟创建的复杂

```java
/**
 * 复杂的对象，创建出来比较耗时间
 */
public class ComplexObject {

    private String name;

    public ComplexObject(String name) {
        try {
            long t1 = System.currentTimeMillis();
            // 模拟创建耗时操作
            ThreadLocalRandom tlr = ThreadLocalRandom.current();
            Thread.sleep(4000 + tlr.nextInt(2000));
            long t2 = System.currentTimeMillis();
            System.out.println(name + " 创建耗时: " + (t2-t1) + "ms");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

其次创建一个 ComplexPooledObjectFactory 实现。当我们有复杂的操作，比如激活对象，钝化对象，销毁对象（需要释放资源）等就需要实现 PooledObjectFactory 来定制，如果没有这些操作选择继承 BasePooledObjectFactory 抽象类更方便。下面分别给出两种创建的代码示例

1. 继承 BasePooledObjectFactory

```java
public class SimplePooledObjectFactory extends BasePooledObjectFactory<ComplexObject> {
    @Override
    public ComplexObject create() {
        // 随机指定一个名称，用于区分ComplexObject
        String name = "test" + ThreadLocalRandom.current().nextInt(100);
        return new ComplexObject(name);
    }
    @Override
    public PooledObject<ComplexObject> wrap(ComplexObject obj) {
        // 使用默认池化对象包装ComplexObject
        return new DefaultPooledObject(obj);
    }
}
```

2. 实现 PooledObjectFactory

```java
public class ComplexPooledObjectFactory implements PooledObjectFactory<ComplexObject> {

    @Override
    public PooledObject<ComplexObject> makeObject() {
        // 随机指定一个名称，用于区分ComplexObject并使用默认池化对象包装ComplexObject
        String name = "test" + ThreadLocalRandom.current().nextInt(100);
        return new DefaultPooledObject<>(new ComplexObject(name));
    }

    @Override
    public void destroyObject(PooledObject<ComplexObject> p) {
        // 销毁对象，当清空，空闲对象大于配置值等会销毁多余对象
        // 此处应释放掉对象占用的资源，如关闭连接，关闭IO等
    }

    @Override
    public boolean validateObject(PooledObject<ComplexObject> p) {
        // 验证对象状态是否正常，是否可用
        return true;
    }

    @Override
    public void activateObject(PooledObject<ComplexObject> p) {
        // 激活对象，使其可用
    }

    @Override
    public void passivateObject(PooledObject<ComplexObject> p) {
        // 钝化对象，使其不可用
    }
}
```

测试代码

```java
public static void main(String[] args) throws Exception {
    // 创建配置对象
    GenericObjectPoolConfig<ComplexObject> poolConfig = new GenericObjectPoolConfig<>();
    // 最大空闲实例数，空闲超过此值将会被销毁淘汰
    poolConfig.setMaxIdle(5);
    // 最大对象数量，包含借出去的和空闲的
    poolConfig.setMaxTotal(20);
    // 最小空闲实例数，对象池将至少保留2个空闲对象
    poolConfig.setMinIdle(2);
    // 对象池满了，是否阻塞获取（false则借不到直接抛异常）
    poolConfig.setBlockWhenExhausted(true);
    // BlockWhenExhausted为true时生效，对象池满了阻塞获取超时，不设置则阻塞获取不超时，也可在borrowObject方法传递第二个参数指定本次的超时时间
    poolConfig.setMaxWaitMillis(3000);
    // 创建对象后是否验证对象，调用objectFactory#validateObject
    poolConfig.setTestOnCreate(false);
    // 借用对象后是否验证对象 validateObject
    poolConfig.setTestOnBorrow(true);
    // 归还对象后是否验证对象 validateObject
    poolConfig.setTestOnReturn(true);
    // 每30秒定时检查淘汰多余的对象, 启用单独的线程处理
    poolConfig.setTimeBetweenEvictionRunsMillis(1000 * 60 * 30);
    // 每30秒定时检查期间是否验证对象 validateObject
    poolConfig.setTestWhileIdle(false);
    // jmx监控，和springboot自带的jmx冲突，可以选择关闭此配置或关闭springboot的jmx配置
    poolConfig.setJmxEnabled(false);

    ComplexPooledObjectFactory objectFactory = new ComplexPooledObjectFactory();
    GenericObjectPool<ComplexObject> objectPool = new GenericObjectPool<>(objectFactory, poolConfig);
    // 申请对象
    ComplexObject obj1 = objectPool.borrowObject();
    println("第一次申请对象：" + obj1.getName());
    // returnObject应该放在finally中 避免业务异常没有归还对象，demo仅做示例
    objectPool.returnObject(obj1);
    // 申请对象， 由于之前归还了，借用的还是之前的对象
    ComplexObject obj2 = objectPool.borrowObject();
    println("第二次申请对象：" + obj2.getName());
    // 再次申请对象，由于之前没有归还，借用的是新创建的
    ComplexObject obj3 = objectPool.borrowObject();
    println("第三次申请对象：" + obj3.getName());

    // returnObject应该放在finally中 避免业务异常没有归还对象，demo仅做示例
    objectPool.returnObject(obj2);
    objectPool.returnObject(obj3);
}
```



### KeyedObjectPool

Commons-Pool 还有 KeyedPooledObjectFactory，KeyedObjectPool 接口，它支持 Key Value 形式。

用法和第二节的类似，区别是对象借用和归还操作需要额外传递自定义的 Key 参数

```java
public interface KeyedPooledObjectFactory<K, V> {
    // 通过参数创建对象
    PooledObject<V> makeObject(K key);
    // 通过参数激活对象，使其可用
    void activateObject(K key, PooledObject<V> obj);
    // 通过参数钝化对象，使其不可用
    void passivateObject(K key, PooledObject<V> obj);
    // 通过参数验证对象状态是否正常，是否可用
    boolean validateObject(K key, PooledObject<V> obj);
    // 通过参数销毁对象，当清空，空闲对象大于配置值等会销毁多余对象
    // 此处应释放掉对象占用的资源，如关闭连接，关闭IO等
    void destroyObject(K key, PooledObject<V> obj);
}
```