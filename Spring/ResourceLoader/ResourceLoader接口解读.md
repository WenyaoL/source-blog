## ResourceLoader接口解读

### ResourceLoader源码

ResourceLoader接口比较简单，就是要求实现该接口的类能获取Resource。

spring提供了一个默认的ResourceLoader实现类，叫做：DefaultResourceLoader

```csharp
/**
 * 加载资源的策略接口 (如类路径和文件系统资源)
 */
public interface ResourceLoader {

    // 用于从类路径加载的伪URL前缀：“classpath：”
    String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

    // 返回指定资源位置的Resource对象
    Resource getResource(String location);

    //返回此ResourceLoader使用的ClassLoader。
    @Nullable
    ClassLoader getClassLoader();
}
```