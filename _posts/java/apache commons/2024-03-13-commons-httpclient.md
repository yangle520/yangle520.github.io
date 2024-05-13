---
layout: post
title:  Apache Commons HttpClient
date:   2024-03-13 00:00:00 +0800
categories: Apache-Commons
tag: java类库
author: YangLe
---


**Apache HttpClient** 组件是为扩展而设计的，同时提供对基本HTTP协议的强大支持。

java.net包提供了通过HTTP访问资源的基本功能，但它并没有提供许多应用程序所需的全部灵活性或功能。HttpClient 组件通过提供一个高效、最新、功能丰富的包来填补这一空白，该包实现了最新HTTP标准的客户端。

**HttpClient** 作为 Java HTTP 客户端的工具类，API简单易懂，大大简化了Java HTTP 发送程序的复杂度，并且自动支持 Cookie 功能，GZip 解析，重定向支持，异步支持等等，用来代替 Java 自带的 API 是个不错的选择。

HttpClient 目前有三个大版本，他们是不兼容的，可以同时存在。HttpClient 3过去是 Commons 的一部分，所以一般来说看到 Apache HttpClient 3 的说法指的就是 Commons HttpClient，所属包 org.apache.commons.httpclient，maven 依赖如下

```xml
<!-- HttpClient 3 -->
<dependency>
    <groupId>commons-httpclient</groupId>
    <artifactId>commons-httpclient</artifactId>
    <version>3.1</version>
</dependency>
```

HttpClient 4 指的是 Apache HttpComponents 下的项目，所属包 org.apache.http，maven 依赖如下

```xml
<!-- HttpClient 4 -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>
```

HttpClient 5 指的是 Apache HttpComponents 下的最新项目，包结构是 org.apache.hc，依赖如下

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.1</version>
</dependency>
```

HttpClient 3 早已不在维护，推荐使用最新的HttpClient 5。HttpClient 5 支持（经典API）（异步API）（反应式API）。



### HttpClient 4

```java
// httpClient对象是线程安全的，可以单例使用，提升性能
CloseableHttpClient httpClient = HttpClientBuilder.create().build();
HttpGet httpGet = new HttpGet("http://test.com/?name=zs&age=11");
httpGet.setHeader("Content-Type", "application/json");
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectTimeout(2000) // 连接超时
        .setConnectionRequestTimeout(2000) // 请求超时
        .setSocketTimeout(2000) // 响应超时
        .build();
httpGet.setConfig(requestConfig);
try (CloseableHttpResponse res = httpClient.execute(httpGet)) {
    int code = res.getStatusLine().getStatusCode();
    if (code == HttpStatus.SC_OK) {
        HttpEntity entity = res.getEntity();
        println(EntityUtils.toString(entity, "UTF-8"));
    } else {
        System.err.println("请求失败，状态码：" + code);
    }
} catch (IOException e) {
    System.err.println("请求异常");
} finally {
    httpGet.reset();
    // httpGet.releaseConnection(); // 等价于reset()
}
```



### HttpClient 5

HttpClient 5 是目前最新版本。下面将着重介绍下。注意所属包是 org.apache.hc。HttpClient 5 除了包名和 HttpClient 4 有所区别，API的用法大体类似，只有小部分不一致

#### 1、POST请求

```java
// httpClient对象是线程安全的，可以单例使用，提升性能
CloseableHttpClient httpClient = HttpClients.createDefault();
HttpPost httpPost = new HttpPost("https://test.com/");
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectTimeout(Timeout.ofSeconds(2))
        .setConnectionRequestTimeout(Timeout.ofSeconds(2))
        .setResponseTimeout(Timeout.ofSeconds(2))
        .build();
httpPost.setConfig(requestConfig);
httpPost.setVersion(HttpVersion.HTTP_2); // 支持http2
httpPost.setHeader("Content-Type", "application/json");
// 支持多种entity参数，字节数组，流，文件等等
// 此处使用restful的"application/json"，所以传递json字符串
httpPost.setEntity(new StringEntity(JSON.toJSONString(paramsObj)));
try (CloseableHttpResponse res = httpClient.execute(httpPost)) {
    if (res.getCode() == HttpStatus.SC_OK) {
        HttpEntity entity = res.getEntity();
        println(EntityUtils.toString(entity));
    } else {
        System.err.println("请求失败，状态码：" + res.getCode());
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    // 释放连接资源
    httpPost.reset();
}
```

#### 2、重定向支持

有些服务器在可能会返回一个重定向的响应，状态码为302或301，如果是浏览器则会自动向重定向地址发起http请求，HttpClient 5 同样支持此功能，示例如下

```java
CloseableHttpClient httpClient = HttpClients.custom()
        .disableAutomaticRetries() //关闭自动重试
        .setRedirectStrategy(DefaultRedirectStrategy.INSTANCE)
        .build();
// ... ...
```

#### 3、异步支持

异步 API 的好处就不说了，HttpClient 5 异步 API 使用 NIO，可以使用少量的线程支持大量的并发，在并发量较大的情况下依然可以保持服务的稳定性和吞吐量。下面看一个简单的例子

```java
CloseableHttpAsyncClient httpClient = HttpAsyncClients.createDefault();
SimpleHttpRequest request = SimpleHttpRequest.create(Method.GET.name(), "https://www.baidu.com/");
httpClient.start();
httpClient.execute(request, new FutureCallback<SimpleHttpResponse>() {
    @Override
    public void completed(SimpleHttpResponse result) {
        // 响应成功
        println(result.getBodyText());
        IOUtils.closeQuietly(httpClient);
    }

    @Override
    public void failed(Exception ex) {
        // 响应出错
        ex.printStackTrace();
        IOUtils.closeQuietly(httpClient);
    }

    @Override
    public void cancelled() {
        // 响应取消
        println("cancelled");
        IOUtils.closeQuietly(httpClient);
    }
});
// ... ... 做其他业务处理
```