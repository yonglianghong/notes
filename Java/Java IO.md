##### 1. File

既表示一个文件，也表示一个目录

- 注意：

  - 既可以传入绝对路径（absolute），也可以传入相对路径（relative），相对路径`.`表示当前目录，`..`表示上级目录

- 主要变量与方法：

  ```java
  separator: / or \\ // 名称里面的分隔符
  pathSeparator : or ; // 多个路径之间的分隔符
  
  boolean createNewFile() // 创建新文件
  boolean mkdir() // 创建目录
  boolean mkdirs() // mkdir -p
  boolean delete() // 删除
  boolean exists() // 是否存在
  String getAbsolutePath() // 获取全路径
  String getName() // 获取名字
  boolean isDirectory() // 是否目录
  boolean isFile() // 是否文件
  String[] list() // 列出子file名字
  File[] listFiles() // 列出子file
  File[] listFiles(FileFilter/FilenameFilter filter) // 传入filter，策略模式
  boolean renameTo(File dest) // 更改名字
  long lastModified() // 修改时间
  long length() // 文件大小，字节数
  setExecutable(boolean executable) // 可执行
  setReadable(boolean readable) // 可读
  setWritable(boolean writable) // 可写
  ```

- 与`java.nio.file.Path`相似



##### 2. InputStream与OutputStream

InputStream与OutputStream是Java标准库提供的最基本的输入输出流。都是抽象类，面向字节，同步输入或输出。主要方法如下：

```java
// 读取输入流的下一个字节，返回字节的表示的int值(0~255)，如果读到末尾，返回-1
public abstract int read() throws IOException;

// 写入一个字节到输出流，虽然传入的是int参数，但只会写入一个字节，即只写入int最低8位表示字节的部分（相当于b & 0xff）
public abstract void write(int b) throws IOException;
```

InputStream：

- ByteArrayInputStream：字节数组输入流
- StingBufferInputStream：字符串输入流
- FileInputStream：文件输入流

OutputStream：

- ByteArrayOutputStream：字节数组输出流
- FileOutputStream：文件输出流



`InputStream`和`OutputStream`都是通过`close()`方法来关闭流，`OutputStream`还提供了一个`flush()`方法刷新缓存。

```java
public void readFile() throws IOException {
    InputStream input = null;
    try {
        input = new FileInputStream("src/readme.txt");
        int n;
        while ((n = input.read()) != -1) { // 利用while同时读取并判断
            System.out.println(n);
        }
    } finally {
        if (input != null) { input.close(); }
    }
}

public void writeFile() throws IOException {
    try (OutputStream output = new FileOutputStream("out/readme.txt")) {
        output.write("Hello".getBytes("UTF-8")); // Hello
    } // 编译器在此自动为我们写入finally并调用close()
}
```

`InputStream`提供了两个重载方法来支持读取多个字节：

- `int read(byte[] b)`：读取若干字节并填充到`byte[]`数组，返回读取的字节数
- `int read(byte[] b, int off, int len)`：指定`byte[]`数组的偏移量和最大填充数

先定义一个`byte[]`数组作为缓冲区，`read()`方法会尽可能多地读取字节到缓冲区， 但不会超过缓冲区的大小。`read()`方法的返回值不再是字节的`int`值，而是返回实际读取了多少个字节。如果返回`-1`，表示到结尾。

```java
public void readFile() throws IOException {
    try (InputStream input = new FileInputStream("src/readme.txt")) {
        // 定义1000个字节大小的缓冲区:
        byte[] buffer = new byte[1000];
        int n;
        while ((n = input.read(buffer)) != -1) { // 读取到缓冲区
            System.out.println("read " + n + " bytes.");
        }
    }
}
```



Filter模式（或者装饰器模式：Decorator）

- FilterInputStream
  - BufferedInputStream：缓冲
  - DataInputStream：从输入流读取基本类型数据
- FilterOutputStream
  - BufferedOutputStream：缓冲
  - DataOutputStream：向输出流写入基本类型数据



读取配置文件，在classpath中的资源文件，路径总是以`／`开头：

```java
try (InputStream input = getClass().getResourceAsStream("/default.properties")) {
    if (input != null) {
        // TODO:
    }
}
```

如果我们把默认的配置放到jar包中，再从外部文件系统读取一个可选的配置文件，就可以做到既有默认的配置文件，又可以让用户自己修改配置：

```java
Properties props = new Properties();
props.load(inputStreamFromClassPath("/default.properties"));
props.load(inputStreamFromFile("./conf.properties"));
```



##### 3. Reader和Writer

Reader：`Reader`是一个字符流，以`char`为单位读取

| InputStream                         | Reader                                |
| :---------------------------------- | :------------------------------------ |
| 字节流，以`byte`为单位              | 字符流，以`char`为单位                |
| 读取字节（-1，0~255）：`int read()` | 读取字符（-1，0~65535）：`int read()` |
| 读到字节数组：`int read(byte[] b)`  | 读到字符数组：`int read(char[] c)`    |



Writer：以`char`为单位输出

| OutputStream                           | Writer                                   |
| :------------------------------------- | :--------------------------------------- |
| 字节流，以`byte`为单位                 | 字符流，以`char`为单位                   |
| 写入字节（0~255）：`void write(int b)` | 写入字符（0~65535）：`void write(int c)` |
| 写入字节数组：`void write(byte[] b)`   | 写入字符数组：`void write(char[] c)`     |
| 无对应方法                             | 写入String：`void write(String s)`       |



Reader与Writer同样为抽象类，它们的实现类如下：

| Reader          | Writer          |
| --------------- | --------------- |
| FileReader      | FileWriter      |
| StringReader    | StringWriter    |
| CharArrayReader | CharArrayWriter |
| BufferedReader  | BufferedWriter  |



使用示例：

```java
public void readFile() throws IOException {
    try (Reader reader = new FileReader("src/readme.txt", StandardCharsets.UTF_8)) {
        char[] buffer = new char[1000]; // 字符缓冲
        int n;
        while ((n = reader.read(buffer)) != -1) {
            System.out.println("read " + n + " chars.");
        }
    }
}

public void readFile() throws IOException {
		try (Writer writer = new FileWriter("readme.txt", StandardCharsets.UTF_8)) {
    writer.write('H'); // 写入单个字符
    writer.write("Hello".toCharArray()); // 写入char[]
    writer.write("Hello"); // 写入String
		}
}

```



InputStream可以转换为Reader，OutputStream可以转换为Writer，利用`InputStreamReader`和`OutputStreamWriter`：

```java
try (Reader reader = new InputStreamReader(new FileInputStream("src/readme.txt"), "UTF-8")) {
    // TODO:
}

try (Writer writer = new OutputStreamWriter(new FileOutputStream("readme.txt"), "UTF-8")) {
    // TODO:
}
```



##### 4. RandomAccessFile

随机访问文件，独立于InputStream和OutputStream体系之外的IO类，同样是面向字节设计的。
构造方法`RandomAccessFile(File file ,  String mode)`支持指定mode，`r`表示只支持读，`rw`表示支持读和写。

主要方法：

```java
long getFilePointer( ) // Returns the current offset in this file. 返回当前偏移量
void seek(long pos ) // Sets the file-pointer offset. 设置读取或写入偏移量

int read() // returned as an integer in the range 0 to 255 (0x00-0x0ff)，返回0~255的int值
int read(byte[] b) // Reads up to b.length bytes of data from this file into an array of bytes. This method blocks until at least one byte of input is available. // 利用b作为缓冲

void write(int b) // Writes the specified byte to this file. 
void write(byte b[]) // Writes b.length bytes from the specified byte array to this file
```

用途：断点续传



##### 5. 标准IO

1. PrintStream

   PrintStream`是一种`FilterOutputStream`，它在`OutputStream`的接口上，额外提供了一些写入各种数据类型的方法：

   - 写入`int`：`print(int)`
   - 写入`boolean`：`print(boolean)`
   - 写入`String`：`print(String)`
   - 写入`Object`：`print(Object)`，实际上相当于`print(object.toString())`



2. System
   - System.in：即InputStream，`BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in))`
   - System.out：即PrintStream
   - System.err：即PrintStream



3. 标准IO重定向
   - System.setIn(InputStream)
   - System.setOut(PrintStream)
   - System.setErr(PrintStream)



##### 6. NIO

1. Channel（通道）：数据可以从Channel读到Buffer，也可以从Buffer写到Channel中
   - mFileChannel：从文件中读写数据
   - DatagramChannel：通过UDP读写网络中的数据
   - SocketChannel：通过TCP读写网络中的数据
   - ServerSocketChannel：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel



2. Buffer（缓冲）：

   - ByteBuffer
   - CharBuffer
   - DoubleBuffer
   - FloatBuffer
   - IntBuffer
   - LongBuffer
   - ShortBuffer

   旧I/O类库中有三个类被修改，可以直接产生FileChannel

   - FileInputChannel
   - FileOutputChannel
   - RandomAccessFile

   java.nio.channels.Channels类提供方法可以产生Reader和Writer



3. Selector（选择器）：

   Selector上可以注册多个channle，然后轮询上面的所有channel，看是否准备就绪（准备好读写），然后进行后续的IO操作。这样，一个单独的线程就可以管理多个channel，比如可以管理多个网络连接。这样就不用为每一个连接都创建一个线程，避免了多线程之间上下文切换导致的开销。



4. 字节顺序

   "big endian"（高位优先），将最重要的字节存放在地址最低的存储器单元

   "little endian"（低位优先），将最重要的字节存放在地址最高的存储器单元



##### 7. 内存映射与文件加锁

内存映射是利用操作系统底层的文件映射工具，假定整个文件都放在内存中，可以当作非常大的数组来访问。

```java
public class LargeMappedFiles {
    static int length = 0x8FFFFFF; // 128MB
    public static void main(String[] args) throws Exception {
        MappedByteBuffer out = 
                new RandomAccessFile("test.dat", "rw").getChannel()
                        .map(FileChannel.MapMode.READ_WRITE, 0, length); // 读写映射，只映射128MB的长度，可以只映射部分文件
        for(int i = 0; i < length; i++)
            out.put((byte)'x');
        System.out.println("Finished writing");
        for(int i = length/2; i < length/2 + 6; i++)
            System.out.println((char)out.get(i));
    }
}
```

文件加锁，允许同步访问某个文件。因为java的文件加锁直接映射到本地操作系统的加锁工具，因此竞争关系体现在不同虚拟机之间，或者操作系统的其他本地线程。

```java
public class FileLocking {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        FileLock fl = fos.getChannel().tryLock();
        if(fl != null){
            System.out.println("Locked File.");
            TimeUnit.MILLISECONDS.sleep(100);
            fl.release();
            System.out.println("Released Lock.");
        }
        fos.close();
    }
}
```



##### 8. 对象序列化

java序列化就是把java对象转换为字节序列的过程，反之亦然。transient变量不会被序列化，static变量是与类绑定，所以也不会被序列化。

serialVersionUID：Java序列化机制通过运行判断类的serialVersionUID来验证版本一致性。反序列化时，虚拟机会把传过来的字节流中serialVersionUID和本地相应类的serialVersionUID进行比较，如果相同就认为是一致的类，可以反序列化，否则抛出异常。如果没有显示指定uid，会自动生成一个。

Serializable接口：对象如果要序列化必须实现Serializable接口，如果该类中引用了别的实例变量，则也需要实现该接口。

序列化过程：虚拟机会首先试图调用待序列化对象里的writeObject和readObject方法，进行自定义的序列化和反序列化。如果在对象中没有定义，那么会默认调用的是`ObjectOutputStream.defaultWriteObject`和`ObjectInputStream.defaultReadObject`方法。

```java
public class SerializableObject implements Serializable{

    private static final long serialVersionUID = 1L;

    private String s0;
    private transient String s1; // 不会被序列化
    public static String s2 = "c";

    public SerializableObject(String s0, String s1) {
        this.s0 = s0;
        this.s1 = s1;
    }

    public String getS0() {
        return s0;
    }

    public String getS1(){
        return s1;
    }

    private void writeObject(ObjectOutputStream out) throws Exception {
        System.out.println("自定义序列化");
        out.defaultWriteObject();
        out.writeUTF(s1); // 手动增加
    }

    private void readObject(ObjectInputStream in) throws Exception {
        System.out.println("自定义反序列");
        in.defaultReadObject();
        s1 = in.readUTF();
    }

    public static void main(String[] args) throws Exception {
      
        File file = new File("src/main/resources/serialize.txt");
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        out.writeObject(new SerializableObject("a", "b"));
        out.close();

        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        SerializableObject so = (SerializableObject) in.readObject();
        System.out.println("s0 = " + so.getS0()); // a
        System.out.println("s1 = " + so.getS1()); // b
        in.close();
      
    }

}
```



##### 9. XML/JSON

```java
/**
<dependency>
    <groupId>org.jdom</groupId>
    <artifactId>jdom</artifactId>
    <version>1.1.3</version>
</dependency>
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <authorizes>
        <authorize>
            <ak>abc</ak>
            <sk>cba</sk>
        </authorize>
    </authorizes>
</config>
*/
import org.jdom.Document;
import org.jdom.Element;
import org.jdom.JDOMException;
import org.jdom.input.SAXBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;


public class XmlParser {

	private InputStream xmlstream = null;
	Element rootElement;
	public XmlParser(String xmlPah) {
		this.xmlstream = XmlParser.class.getClassLoader().getResourceAsStream(xmlPah);
        try {
            init();
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

	public void init() throws JDOMException, IOException {
        SAXBuilder jdomsaxBuilder = new SAXBuilder(false);
        Document doc = jdomsaxBuilder.build(xmlstream);
        rootElement = doc.getRootElement();
    }

    public Element getChild(String childName){
	    return rootElement.getChild(childName);
    }

    public List getChildren(){
	    return rootElement.getChildren();
    }


	public static void main(String[] args) throws JDOMException, IOException {
        Element element = (Element)new XmlParser("authorize-config.xml").getChild("authorizes").getChildren().get(0);
        System.out.println(element.getChildTextTrim("ak"));
	}
	
}
```

