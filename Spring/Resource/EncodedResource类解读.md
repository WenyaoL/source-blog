## EncodedResource类解读

#### EncodedResource介绍

EncodedResource是spring中Resource编码相关的封装类，EncodedResource里面封装了一个Resource成员属性，其实主要功能就是通过指定的编码获取资源的输入流。



#### EncodedResource部分源码

EncodedResource主要看getReader()方法，这个方法通过指定的编码方式返回InputStreamReader。

```java
public class EncodedResource implements InputStreamSource {
	
	private final Resource resource;

	@Nullable
	private final String encoding;

	@Nullable
	private final Charset charset;
	
	//.......
	
    //获取InputStreamReader
	public Reader getReader() throws IOException {
		if (this.charset != null) {
			return new InputStreamReader(this.resource.getInputStream(), this.charset);
		}
		else if (this.encoding != null) {
			return new InputStreamReader(this.resource.getInputStream(), this.encoding);
		}
		else {
			return new InputStreamReader(this.resource.getInputStream());
		}
	}
}
```

