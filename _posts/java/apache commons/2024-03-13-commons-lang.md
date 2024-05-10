---
layout: post
title:  Apache Commons Lang
date:   2024-03-13 00:00:00 +0800
categories: Apache Commons
tag: java类库
author: YangLe
---


**Apache Commons Lang**是对java.lang的扩展，基本上是commons中最常用的工具包

lang最新版本是2.6，最低要求Java1.2以上，目前官方已不在维护。lang3目前最新版本是3.12.0，最低要求Java8以上。相对于lang来说完全支持Java8的特性，废除了一些旧的API。该版本无法兼容旧有版本，于是为了避免冲突改名为lang3



![图片]({{ '/images/java/commons_lang.png' | prepend: site.baseurl }})



### 日期相关

1、字符串转日期

```java
final String strDate = "2021-07-04 11:11:11";
final String pattern = "yyyy-MM-dd HH:mm:ss";
// 原生写法
SimpleDateFormat sdf = new SimpleDateFormat(pattern);
Date date1 = sdf.parse(strDate);
// commons写法
Date date2 = DateUtils.parseDate(strDate, pattern);
```

2、日期转字符串

```java
final Date date = new Date();
final String pattern = "yyyy年MM月dd日";
// 原生写法
SimpleDateFormat sdf = new SimpleDateFormat(pattern);
String strDate = sdf.format(date);
// 使用commons写法
String strDate = DateFormatUtils.format(date, pattern);
```

3、日期计算

```java
final Date date = new Date();
// 原生写法
Calendar cal = Calendar.getInstance();
cal.setTime(date);
cal.add(Calendar.DATE, 5); // 加5天
cal.add(Calendar.HOUR_OF_DAY, -5); // 减5小时
// 使用commons写法
Date newDate1 = DateUtils.addDays(date, 5); // 加5天
Date newDate2 = DateUtils.addHours(date, -5); // 减5小时
Date newDate3 = DateUtils.truncate(date, Calendar.DATE); // 过滤时分秒
boolean isSameDay = DateUtils.isSameDay(newDate1, newDate2); // 判断是否是同一天
```



### 字符串相关

1、字符串判空

```java
String str = "";
// 原生写法
if (str == null || str.length() == 0) {
    // Do something
}
// commons写法
if (StringUtils.isEmpty(str)) {
    // Do something
}  
/* StringUtils.isEmpty(null)      = true
 * StringUtils.isEmpty("")        = true
 * StringUtils.isEmpty(" ")       = false
 * StringUtils.isEmpty("bob")     = false
 * StringUtils.isEmpty("  bob  ") = false
 */
StringUtils.isNotEmpty(str);
/* 
 * null, 空串，空格为true
 * StringUtils.isBlank(null)      = true
 * StringUtils.isBlank("")        = true
 * StringUtils.isBlank(" ")       = true
 * StringUtils.isBlank("bob")     = false
 * StringUtils.isBlank("  bob  ") = false
 */
StringUtils.isBlank(str);
// isBlank取反
StringUtils.isNotBlank(str);
// 任意一个参数为空则结果为true
StringUtils.isAnyEmpty(str1, str2, str3);
// 所有参数为空则结果为true
StringUtils.isAllEmpty(str1, str2, str3);
```

2、字符串去空格

```java
// 去除两端空格，不需要判断null
String newStr = StringUtils.trim(str);
/*
 * 去除两端空格，如果是null则转换为空字符串
 * StringUtils.trimToEmpty(null)          = ""
 * StringUtils.trimToEmpty("")            = ""
 * StringUtils.trimToEmpty("     ")       = ""
 * StringUtils.trimToEmpty("abc")         = "abc"
 * StringUtils.trimToEmpty("    abc    ") = "abc"
 */
newStr = StringUtils.trimToEmpty(str);
/*
 * 去除两端空格，如果结果是空串则转换为null
 * StringUtils.trimToNull(null)          = null
 * StringUtils.trimToNull("")            = null
 * StringUtils.trimToNull("     ")       = null
 * StringUtils.trimToNull("abc")         = "abc"
 * StringUtils.trimToNull("    abc    ") = "abc"
 */
newStr = StringUtils.trimToNull(str);
/*
 * 去两端 给定字符串中任意字符
 * StringUtils.strip(null, *)          = null
 * StringUtils.strip("", *)            = ""
 * StringUtils.strip("abc", null)      = "abc"
 * StringUtils.strip("  abc", null)    = "abc"
 * StringUtils.strip("abc  ", null)    = "abc"
 * StringUtils.strip(" abc ", null)    = "abc"
 * StringUtils.strip("  abcyx", "xyz") = "  abc"
 */
newStr = StringUtils.strip(str, "stripChars");
// 去左端 给定字符串中任意字符
newStr = StringUtils.stripStart(str, "stripChars");
// 去右端 给定字符串中任意字符
newStr = StringUtils.stripEnd(str, "stripChars");
```

3、字符串分隔

```java
/*
 * 按照空格分割字符串 结果为数组
 * StringUtils.split(null)       = null
 * StringUtils.split("")         = []
 * StringUtils.split("abc def")  = ["abc", "def"]
 * StringUtils.split("abc  def") = ["abc", "def"]
 * tringUtils.split(" abc ")    = ["abc"]
 */
 StringUtils.split(str);
 // 按照某些字符分割 结果为数组，自动去除了截取后的空字符串
 StringUtils.split(str, ",");
```

4、取子字符串

```java
// 获得"ab.cc.txt"中最后一个.之前的字符串
StringUtils.substringBeforeLast("ab.cc.txt", "."); // ab.cc
// 获得"ab.cc.txt"中最后一个.之后的字符串（常用于获取文件后缀名）
StringUtils.substringAfterLast("ab.cc.txt", "."); // txt
// 获得"ab.cc.txt"中第一个.之前的字符串
StringUtils.substringBefore("ab.cc.txt", "."); // ab
// 获得"ab.cc.txt"中第一个.之后的字符串
StringUtils.substringAfter("ab.cc.txt", "."); // cc.txt
// 获取"ab.cc.txt"中.之间的字符串
StringUtils.substringBetween("ab.cc.txt", "."); // cc
// 看名字和参数应该就知道干什么的了
StringUtils.substringBetween("a(bb)c", "(", ")"); // bb
```

5、其他

```java
// 首字母大写
StringUtils.capitalize("test"); // Test
// 字符串合并
StringUtils.join(new int[]{1,2,3}, ",");// 1,2,3
// 缩写
StringUtils.abbreviate("abcdefg", 6);// "abc..."
// 判断字符串是否是数字
StringUtils.isNumeric("abc123");// false
// 删除指定字符
StringUtils.remove("abbc", "b"); // ac
// ... ... 
```

6、随机字符串

```java
// 随机生成长度为5的字符串
RandomStringUtils.random(5);
// 随机生成长度为5的"只含大小写字母"字符串
RandomStringUtils.randomAlphabetic(5);
// 随机生成长度为5的"只含大小写字母和数字"字符串
RandomStringUtils.randomAlphanumeric(5);
// 随机生成长度为5的"只含数字"字符串
RandomStringUtils.randomNumeric(5);
```





### 反射相关

1、属性操作

```java
public class ReflectDemo {
    private static String sAbc = "111";
    private String abc = "123";
    public void fieldDemo() throws Exception {
        ReflectDemo reflectDemo = new ReflectDemo(); 
        
        // 原生写法 - 反射获取对象实例属性的值
        Field abcField = reflectDemo.getClass().getDeclaredField("abc");
        abcField.setAccessible(true);// 设置访问级别，如果private属性不设置则访问会报错
        String value = (String) abcField.get(reflectDemo);// 123
        
        // commons写法 - 反射获取对象属性的值  方法名如果不含Declared会向父类上一直查找
        String value2 = (String) FieldUtils.readDeclaredField(reflectDemo, "abc", true);//123
        // 反射获取类静态属性的值
        String value3 = (String) FieldUtils.readStaticField(ReflectDemo.class, "sAbc", true);//111
        // 反射设置对象属性值
        FieldUtils.writeField(reflectDemo, "abc", "newValue", true);
        // 反射设置类静态属性的值
        FieldUtils.writeStaticField(ReflectDemo.class, "sAbc", "newStaticValue", true);        
    }
}
```

2、获取注解方法

```java
// 原生写法
List<Method> annotatedMethods = new ArrayList<Method>();
for (Method method : ReflectDemo.class.getMethods()) {
    if (method.getAnnotation(Test.class) != null) {
        annotatedMethods.add(method);
    }
}
// commons写法
Method[] methods = MethodUtils.getMethodsWithAnnotation(ReflectDemo.class, Test.class);
```

3、方法调用

```java
private static void testStaticMethod(String param1) {}
private void testMethod(String param1) {}
  
public void invokeDemo() throws Exception {
    // 调用函数"testMethod"
    ReflectDemo reflectDemo = new ReflectDemo();
    // 原生写法
    Method testMethod = reflectDemo.getClass().getDeclaredMethod("testMethod");
    testMethod.setAccessible(true); // 设置访问级别，如果private函数不设置则调用会报错
    testMethod.invoke(reflectDemo, "testParam");
    
    // commons写法
    MethodUtils.invokeExactMethod(reflectDemo, "testMethod", "testParam");
    // 调用static方法
    MethodUtils.invokeExactStaticMethod(ReflectDemo.class, "testStaticMethod", "testParam");
    // 调用方法(含继承过来的方法)
    MethodUtils.invokeMethod(reflectDemo, "testMethod", "testParam");
    // 调用static方法(当前不存在则向父类寻找匹配的静态方法)
    MethodUtils.invokeStaticMethod(ReflectDemo.class, "testStaticMethod", "testParam");
}
```



### 系统相关

主要是获取操作系统和JVM一些信息

```java
// 判断操作系统类型
boolean isWin = SystemUtils.IS_OS_WINDOWS;
boolean isWin10 = SystemUtils.IS_OS_WINDOWS_10;
boolean isWin2012 = SystemUtils.IS_OS_WINDOWS_2012;
boolean isMac = SystemUtils.IS_OS_MAC;
boolean isLinux = SystemUtils.IS_OS_LINUX;
boolean isUnix = SystemUtils.IS_OS_UNIX;
boolean isSolaris = SystemUtils.IS_OS_SOLARIS;
// ... ...

// 判断java版本
boolean isJava6 = SystemUtils.IS_JAVA_1_6;
boolean isJava8 = SystemUtils.IS_JAVA_1_8;
boolean isJava11 = SystemUtils.IS_JAVA_11;
boolean isJava14 = SystemUtils.IS_JAVA_14;
// ... ...

// 获取java相关目录
File javaHome = SystemUtils.getJavaHome();
File userHome = SystemUtils.getUserHome();// 操作系统用户目录
File userDir = SystemUtils.getUserDir();// 项目所在路径
File tmpDir = SystemUtils.getJavaIoTmpDir();
```