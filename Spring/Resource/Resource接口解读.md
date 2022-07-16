## Resource接口解读

#### Resource介绍

Resource是spring的资源策略接口，Resource定义了一套资源框架。spring提供了更加丰富的资源的定义。对不同来源的资源文件都有相应的 Resource 实现∶文件（FileSystemResource）、Classpath 资源（ClassPathResource）、URL 资源（UrlResource）、InputStream资源（InputStreamResource）、Byte 数组（ByteArrayResource）等

> 在 Java 中，将不同来源的资源抽象成 URL，通过注册不同的 handler（URLStreamHandler）来处理不同来源的资源的读取逻辑，一般 handler 的类型使用不同前缀（协议，Protocol）来识别，如"fle∶"、"http∶"、"jar∶"等，然而 URL 没有默认定义相对 Classpath 或 ServletContext 等资源的 handler，虽然可以注册自己的 URLStreamHandler来解析特定的 URL前缀（协议），比如"classpath∶"，然而这需要了解 URL 的实现机制，而且URL也没有提供一些基本的方法，如检查当前资源是否存在、检查当前资源是否可读等方法。因而 Spring 对其内部使用到的资源实现了自己的抽象结构∶Resource 接口来封装底层资源。

#### Resource源码

Resource是spring的资源策略接口，Resource定义了一套资源框架。spring提供了更加丰富的资源的定义。实现了Resource接口的类拥有更加丰富的访问资源访问功能。

```java
public interface Resource extends InputStreamSource {
	// 判断资源是否存在
    boolean exists();
    
    default boolean isReadable() {
        return true;
    }
    //判断资源是否打开
    default boolean isOpen() {
        return false;
    }
    //判断资源是否是一个文件
    default boolean isFile() {
        return false;
    }
    //获取资源文件的URL
    URL getURL() throws IOException;
    //获取资源文件的URI
    URI getURI() throws IOException;
    
    File getFile() throws IOException;
    //默认实现，返回的是ReadableByteChannel，这个类属于Java的NIO中的管道。
    default ReadableByteChannel readableChannel() throws IOException {
        return Channels.newChannel(getInputStream());
    }
    long contentLength() throws IOException;
    long lastModified() throws IOException;
    //根据relativePath返回一个相对与该Resource的Resource
    Resource createRelative(String relativePath) throws IOException;
    @Nullable
    String getFilename();
    String getDescription();

}

```



#### Resource的实现类或继承接口

> WritableResource:可写资源接口，是Spring3.1版本新增的接口，有两个实现类FileSystemResource和PathResource，其中PathResource是Spring4.0提供的实现类。
>
> ByteArrayResource:二进制数组表示的资源，二进制数组资源可以在内存中通过程序构造。
>
> ClassPathResource:类路径下的资源，资源以相对路径的方式表示。
>
> FileSystemResource:文件系统资源，资源一文件系统的路径表示，如D:/aaa/vvv.java
>
> InputStreamResource：以输入流表示返回的资源。
>
> ServletContextResource:问访问Web容器上下文中的资源而设计的类，负责对于Web应用根目录的路径加载资源。它支持以流和URL的方式访问，在WAR解包的情况下，也可以通过File方式访问。该类还可以直接从JAR包中访问资源。
>
> UrlResource:URL封装了java.net.URL，它使用户能够访问任任何可以通过URL表示的资源，如文件系统的资源、HTTP资源、FTP资源等。
>
> PathResource:Spring4.0提供的读取资源文件的新类。Path封装了java.net.URL、java.nio.Path、文件系统资源，它使用户能够访问任何可以通过URL、Path、系统文件路径表示的资源，如文件系统的资源，HTTP资源、FTP资源等。



![Resource接口实现关系图](https://raw.githubusercontent.com/WenyaoL/blog-img/main/114.png)





#### FileSystemResource的使用例子

FileSystemResource是Resource其中的一个实现类，提供了文件系统资源访问的能力。下面有简单的使用例子。

```java
@SpringBootTest
public class ResourceTest {

    @Test
    public void testFileSystemResourc() throws IOException {
        String path = "F://test.txt";
        FileSystemResource fileSystemResource = new FileSystemResource(path);
        
        InputStream inputStream = fileSystemResource.getInputStream();
        //FileSystemResource实现了WritableResource接口
        OutputStream outputStream = fileSystemResource.getOutputStream();
        
        System.out.println(fileSystemResource.isFile());
        System.out.println("长度:"+fileSystemResource.contentLength());
        System.out.println("文件名"+fileSystemResource.getFilename());
    }
```

