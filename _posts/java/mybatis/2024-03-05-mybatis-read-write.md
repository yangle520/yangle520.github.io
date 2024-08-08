---
layout: post
title:  MyBatis 读写分离配置
date:   2024-01-15 00:00:00 +0800
categories: 框架
tag: MyBatis
author: YangLe
---

### 1、加入依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
```

### 2、yml配置多数据源

```yml
spring:
  datasource:
    dynamic:
      primary: master
      strict: false
      datasource: 
        master:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://ip:port/db_master
          username: ENC(kMN9s5ykTHkCd/MNtDFSpQ==)
          password: ENC(rQD6hzrt1fOC/F7EGP4BaOGzJ5FO6Scu)
        slave1:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://ip:port/db_slave
          username: ENC(kMN9s5ykTHkCd/MNtDFSpQ==)
          password: ENC(rQD6hzrt1fOC/F7EGP4BaOGzJ5FO6Scu)
```

### 3、自定义拦截器类，实现读写分离

```java
@Intercepts({
      @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
      @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
      @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
})
@Component
@Slf4j
public class DynamicInterceptor implements Interceptor {

   private static final String MASTER = "master";

   private static final String SLAVE = "slave1";

   @Override
   public Object intercept(Invocation invocation) throws Throwable {
      boolean synchronizationActive = TransactionSynchronizationManager.isSynchronizationActive();
      if (!synchronizationActive) {
         Object[] objects = invocation.getArgs();
         MappedStatement ms = (MappedStatement) objects[0];
         String currentDataSource = DynamicDataSourceContextHolder.peek();
         if (StrUtil.isEmpty(currentDataSource)) {
            if (ms.getSqlCommandType().equals(SqlCommandType.SELECT)) {
               DynamicDataSourceContextHolder.push(SLAVE);
            } else {
               DynamicDataSourceContextHolder.push(MASTER);
            }
            Object proceed = invocation.proceed();
            DynamicDataSourceContextHolder.clear();
            return proceed;
         }
      }

      Object proceed = invocation.proceed();
      DynamicDataSourceContextHolder.clear();
      return proceed;
   }

   @Override
   public Object plugin(Object target) {
      if (target instanceof Executor) {
         return Plugin.wrap(target, this);
      } else {
         return target;
      }
   }
}
```

### 4、注册拦截器类

```java
@Configuration
@MapperScan(basePackages = {"com.smart.mine.*.mapper"})
public class MybatisPlusConfig {
    @Bean
    public DynamicInterceptor dynamicInterceptor() {
        return new DynamicInterceptor();
    }
}
```

### 5、自定义使用

```java
@Operation(summary = "读主")
@GetMapping("/readMaster")
@DS("master") // 不加@DS注释，按DynamicInterceptor配置的库执行，配置后按指定的库执行
public Result<Dept> readMaster(Long id) {
   return Result.success(deptService.getById(id));
}
```

