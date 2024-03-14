---
layout: post
title:  commons-io
date:   2024-03-13 00:00:00 +0800
categories: Apache Commons
tag: java类库
---







commons-io 是java文件IO 第一库是公认的，它的功能和代码质量都是极佳的，它好到没有人想到再写一个竞品与之竞争。

Apache Commons IO库主要提供以下几个方面的功能

1. **文件操作**：简化文件的读取和写入。
2. **流操作**：提供了更简单的方法来处理Java的输入输出流。
3. **文件监控**：能够监控文件系统的变化，比如文件的创建、修改和删除。

**为什么选择Apache Commons IO**

- **简化代码**：使用Apache Commons IO可以使得代码更加简洁，提高代码的可读性和可维护性。
- **功能强大**：提供了很多Java标准库中没有的便利功能。
- **社区支持**：作为Apache项目的一部分，它拥有强大的社区支持和持续的更新。



![图片]({{ '/styles/images/java/IO.png' | prepend: site.baseurl }})



## 常用类介绍

### 流相关

#### 1、关闭流

```java
InputStream inputStream = new FileInputStream("test.txt");
OutputStream outputStream = new FileOutputStream("test.txt");
// commons写法(可以传任意数量的流)
IOUtils.closeQuietly(inputStream, outputStream);
```

#### 2、读取流

```java
InputStream is = new FileInputStream("foo.txt");
// 输入流转换为byte数组
byte[] result2 = IOUtils.toByteArray(is);
// 输入流转换为字符串
String result2 = IOUtils.toString(is, "UTF-8");
// 将reader转换为字符串
String toString(Reader reader, String charset) throws IOException;
// 将url转换为字符串，也就是可以直接将网络上的内容下载为字符串
String toString(URL url, String charset) throws IOException;
```

#### 3、其他

```java
// 按照行读取结果
InputStream is = new FileInputStream("test.txt");
List<String> lines = IOUtils.readLines(is, "UTF-8");

// 将行集合写入输出流
OutputStream os = new FileOutputStream("newTest.txt");
IOUtils.writeLines(lines, System.lineSeparator(), os, "UTF-8");

// 拷贝输入流到输出流
InputStream inputStream = new FileInputStream("src.txt");
OutputStream outputStream = new FileOutputStream("dest.txt");
IOUtils.copy(inputStream, outputStream);
```



### 文件相关

文件相关主要有 FileUtils：文件工具类，FileNameUtils：文件名工具类，PathUtils：路径工具类

#### 1、文件读写

```java
File readFile = new File("test.txt");
// 读取文件
String str = FileUtils.readFileToString(readFile, "UTF-8");
// 读取文件为字节数组
byte[] bytes = FileUtils.readFileToByteArray(readFile);
// 按行读取文件
List<String> lines =  FileUtils.readLines(readFile, "UTF-8");

File writeFile = new File("out.txt");
// 将字符串写入文件
FileUtils.writeStringToFile(writeFile, "测试文本", "UTF-8");
// 将字节数组写入文件
FileUtils.writeByteArrayToFile(writeFile, bytes);
// 将字符串列表一行一行写入文件
FileUtils.writeLines(writeFile, lines, "UTF-8");
```

#### 2、移动和复制

```java
File srcFile = new File("src.txt");
File destFile = new File("dest.txt");
File srcDir = new File("/srcDir");
File destDir = new File("/destDir");
// 移动/拷贝文件
FileUtils.moveFile(srcFile, destFile);
FileUtils.copyFile(srcFile, destFile);
// 移动/拷贝文件到目录
FileUtils.moveFileToDirectory(srcFile, destDir, true);
FileUtils.copyFileToDirectory(srcFile, destDir);
// 移动/拷贝目录
FileUtils.moveDirectory(srcDir, destDir);
FileUtils.copyDirectory(srcDir, destDir);
// 拷贝网络资源到文件
FileUtils.copyURLToFile(new URL("http://xx"), destFile);
// 拷贝流到文件
FileUtils.copyInputStreamToFile(new FileInputStream("test.txt"), destFile);
// ... ...
```

#### 3、其他操作

```java
File file = new File("test.txt");
File dir = new File("/test");
// 删除文件
FileUtils.delete(file);
// 删除目录
FileUtils.deleteDirectory(dir);
// 文件大小，如果是目录则递归计算总大小
long s = FileUtils.sizeOf(file);
// 则递归计算目录总大小，参数不是目录会抛出异常
long sd = FileUtils.sizeOfDirectory(dir);
// 递归获取目录下的所有文件
Collection<File> files = FileUtils.listFiles(dir, null, true);
// 获取jvm中的io临时目录
FileUtils.getTempDirectory();
// ... ...
```

#### 4、文件名称相关

```java
// 获取名称，后缀等
String name = "/home/xxx/test.txt";
FilenameUtils.getName(name); // "test.txt"
FilenameUtils.getBaseName(name); // "test"
FilenameUtils.getExtension(name); // "txt"
FilenameUtils.getPath(name); // "/home/xxx/"

// 将相对路径转换为绝对路径
FilenameUtils.normalize("/foo/bar/.."); // "/foo"
```

#### 5、Path操作

```java
// path既可以表示目录也可以表示文件

// 获取当前路径
Path path = PathUtils.current();
// 删除path
PathUtils.delete(path);
// 路径或文件是否为空
PathUtils.isEmpty(path);
// 设置只读
PathUtils.setReadOnly(path, true);
// 复制
PathUtils.copyFileToDirectory(Paths.get("test.txt"), path);
PathUtils.copyDirectory(Paths.get("/srcPath"), Paths.get("/destPath"));
// 统计目录内文件数量
Counters.PathCounters counters = PathUtils.countDirectory(path);
counters.getByteCounter(); // 字节大小
counters.getDirectoryCounter(); // 目录个数
counters.getFileCounter(); // 文件个数
// ... ...
```

### 文件比较器

1. CompositeFileComparator：组合排序，将以下排序规则组合在一起
2. DefaultFileComparator：默认文件比较器，直接使用File的compare方法。（文件集合排序（ Collections.sort() ）时传此比较器和不传效果一样）
3. DirectoryFileComparator：目录排在文件之前
4. ExtensionFileComparator：扩展名比较器，按照文件的扩展名的ascii顺序排序，无扩展名的始终排在前面
5. LastModifiedFileComparator：按照文件的最后修改时间排序
6. NameFileComparator：按照文件名称排序
7. PathFileComparator：按照路径排序，父目录优先排在前面
8. SizeFileComparator：按照文件大小排序，小文件排在前面（目录会计算其总大小）

使用示例

```java
List<File> files = Arrays.asList(new File[]{
        new File("/foo/def"),
        new File("/foo/test.txt"),
        new File("/foo/abc"),
        new File("/foo/hh.txt")});
// 排序目录在前
Collections.sort(files, DirectoryFileComparator.DIRECTORY_COMPARATOR); // ["/foo/def", "/foo/abc", "/foo/test.txt", "/foo/hh.txt"]
// 排序目录在后
Collections.sort(files, DirectoryFileComparator.DIRECTORY_REVERSE); // ["/foo/test.txt", "/foo/hh.txt", "/foo/def", "/foo/abc"]
// 组合排序，首先按目录在前排序，其次再按照名称排序
Comparator dirAndNameComp = new CompositeFileComparator(
            DirectoryFileComparator.DIRECTORY_COMPARATOR,
            NameFileComparator.NAME_COMPARATOR);
Collections.sort(files, dirAndNameComp); // ["/foo/abc", "/foo/def", "/foo/hh.txt", "/foo/test.txt"]
```





### 文件监视器

Apache Commons IO提供了`FileMonitor`类，它可以帮助咱们轻松实现文件系统的监控。下面，通过一个示例来看看如何使用它。

```java
public class FileMonitoringExample {
    public static void main(String[] args) throws Exception {
        // 监控的文件夹路径
        String directoryPath = "监控的文件夹";

        // 创建一个文件观察器，用于监控指定的目录
        FileAlterationObserver observer = new FileAlterationObserver(
                directoryPath, FileFilterUtils.and(
                    FileFilterUtils.fileFileFilter(), // 只监控文件
                    FileFilterUtils.suffixFileFilter(".txt") // 只监控.txt文件
                )
        );

        // 创建一个监听器，用于响应文件变化事件
        observer.addListener(new FileAlterationListenerAdaptor() {
            @Override
            public void onFileCreate(File file) {
                System.out.println("文件被创建: " + file.getName());
            }

            @Override
            public void onFileDelete(File file) {
                System.out.println("文件被删除: " + file.getName());
            }
        });

        // 创建文件变化监控器，并添加观察器
        FileAlterationMonitor monitor = new FileAlterationMonitor(5000); // 检查间隔为5秒
        monitor.addObserver(observer);

        // 启动监控器
        monitor.start();
        System.out.println("文件监控启动，正在监控: " + directoryPath);
    }
}
```

在这个示例中，咱们首先创建了一个`FileAlterationObserver`来观察特定的文件夹。然后，添加了一个监听器`FileAlterationListenerAdaptor`，用来定义在文件创建或删除时的具体行为。最后，通过`FileAlterationMonitor`启动了监控过程。

#### **监控原理解析**

Apache Commons IO的文件监控机制基于轮询（Polling）原理。这意味着它会定期检查文件系统的状态，然后与上一次的状态进行对比，以确定是否有任何变化发生。虽然这种方法可能不如操作系统级别的文件事件通知那样实时，但它的优点是跨平台且实现简单。

通过上面的例子，咱们可以看到，Apache Commons IO提供的文件监控功能非常强大且易于使用。它让文件系统的监控变得简单，有助于咱们更好地管理文件相关的任务和事件。使用这个功能，可以在软件系统中实现自动化的文件处理逻辑，提高效率和可靠性。



### 文件过滤器

FileFilterUtils提供一些工厂方法用于生成各类文件过滤器，提供一些静态方法用于对指定的File集合进行过滤

```java
File dir = new File(PARENT_DIR);

// 匹配文件名
for (String file : dir.list(new NameFileFilter(acceptedNames, IOCase.INSENSITIVE))) {
	System.out.println("File found, named: " + file);
}

// 根据通配符匹配
dir.list(new WildcardFileFilter("*est*"));
// 匹配前缀
dir.list(new PrefixFileFilter("test"));
// 匹配后缀
dir.list(new SuffixFileFilter(".txt"));
// 逻辑或
dir.list(new OrFileFilter(new WildcardFileFilter("*est*"), new SuffixFileFilter(".txt")));
// 逻辑与
dir.list(new AndFileFilter(new WildcardFileFilter("*est*"), new NotFileFilter(new SuffixFileFilter(".txt"))));
```

基本功能过滤器

| 过滤器                 | 过滤内容     | 说明                                                 |
| ---------------------- | ------------ | ---------------------------------------------------- |
| FileFileFilter         | 基于类型过滤 | 仅接受文件                                           |
| DirectoryFilter        | 类型         | 仅接受目录                                           |
| PrefixFileFilter       | 名称         | 基于前缀（不带路径的文件名)                          |
| SuffixFileFilter       | 名称         | 基于后缀(不带路径的文件名)                           |
| NameFileFilter         | 名称         | 基于文件名称(不带路径的文件名)                       |
| WildcardFileFilter     | 名称         | 基于通配符(不带路径的文件名)                         |
| RegexFileFilter        | 名称         | 基于正则表达式                                       |
| AgeFileFilter          | 时间         | 基于最后修改时间                                     |
| MagicNumberFileFileter | 时间         | 基于Magic Number                                     |
| EmptyFileFilter        | 大小         | 基于文件或目录是否为空                               |
| SizeFileFilter         | 大小         | 基于文件尺寸                                         |
| HiddenFileFilter       | 隐藏属性     | 基于文件或目录是否隐藏                               |
| CanReadFileFilter      | 读写属性     | 基于是否可读                                         |
| CanWriteFileFilter     | 读写属性     | 基于是否可写入                                       |
| DelegateFileFilter     |              | 将普通的FileFilter和FilenameFilter包装成IOFileFilter |
| AndFileFilter          |              | 基于AND逻辑运算                                      |
| OrFileFilter           |              | 基于OR逻辑运算                                       |
| NotFileFilter          |              | 基于NOT逻辑运算                                      |
| TrueFileFilter         |              | 不进行过滤                                           |
| FalseFileFilter        |              | 过滤所有文件及目录                                   |





## 使用

pom依赖

```xml
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.8.0</version>
        </dependency>
```

JavaDoc：http://tool.oschina.net/apidocs/apidoc?api=commons-io



