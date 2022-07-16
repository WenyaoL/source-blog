## FileSystemResource解读

#### FileSystemResource介绍

spring的一个用于读取文件系统下的资源的一个实现类，使用起来非常简单。

#### FileSystemResource部分源码

获取文件的输入流

```java
	/**
	 * This implementation opens a NIO file stream for the underlying file.
	 * @see java.io.FileInputStream
	 */
	@Override
	public InputStream getInputStream() throws IOException {
		try {
			return Files.newInputStream(this.filePath);
		}
		catch (NoSuchFileException ex) {
			throw new FileNotFoundException(ex.getMessage());
		}
	}
```

判断资源是否可写(如果是目录那就肯定不可写啦)

```java
/**
	 * This implementation checks whether the underlying file is marked as writable
	 * (and corresponds to an actual file with content, not to a directory).
	 * @see java.io.File#canWrite()
	 * @see java.io.File#isDirectory()
	 */
	@Override
	public boolean isWritable() {
		return (this.file != null ? this.file.canWrite() && !this.file.isDirectory() :
				Files.isWritable(this.filePath) && !Files.isDirectory(this.filePath));
	}
```



#### FileSystemResource使用例子

读取一个文件，写（覆盖）些数据进去

```java
 @Test
    public void testFileSystemResourc() throws IOException {
        String path = "F://test.txt";
        FileSystemResource fileSystemResource = new FileSystemResource(path);

        System.out.println(fileSystemResource.isFile());
        System.out.println("长度:"+fileSystemResource.contentLength());
        System.out.println("文件名"+fileSystemResource.getFilename());

        //FileSystemResource实现了WritableResource接口
        OutputStream outputStream = fileSystemResource.getOutputStream();
        OutputStreamWriter outputStreamWriter = new OutputStreamWriter(outputStream);
        outputStreamWriter.write("123456789");
        outputStreamWriter.flush();
        outputStreamWriter.close();

    }
```

