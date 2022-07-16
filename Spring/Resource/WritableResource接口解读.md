## WritableResource接口解读

#### 源码

WritableResource继承于Resource，提供了资源的写能力。

```java
public interface WritableResource extends Resource {

	//是否可写
	default boolean isWritable() {
		return true;
	}

	//返回OutputStream
	OutputStream getOutputStream() throws IOException;
	
    //返回WritableByteChannel(Java nio中的Channel)
	default WritableByteChannel writableChannel() throws IOException {
		return Channels.newChannel(getOutputStream());
	}

}
```

#### 例子

FileSystemResource实现了WritableResource所有具备了写的能力，可以尝试一下使用该接口获取输出流。

```java
@Test
    public void testFileSystemResourc2() throws IOException {
        String path = "F://test.txt";
        WritableResource fileSystemResource = new FileSystemResource(path);

        System.out.println("长度:"+fileSystemResource.contentLength());


        //FileSystemResource实现了WritableResource接口
        OutputStream outputStream = fileSystemResource.getOutputStream();
        OutputStreamWriter outputStreamWriter = new OutputStreamWriter(outputStream);
        outputStreamWriter.write("123456789");
        outputStreamWriter.flush();
        outputStreamWriter.close();

        System.out.println("长度:"+fileSystemResource.contentLength());

        WritableByteChannel writableByteChannel = fileSystemResource.writableChannel();
        ByteBuffer allocate = ByteBuffer.allocate(1024 * 10);
        allocate.put("abcdefghijk".getBytes());
        allocate.flip();
        writableByteChannel.write(allocate);
        writableByteChannel.close();

        System.out.println("长度:"+fileSystemResource.contentLength());
    }
```

