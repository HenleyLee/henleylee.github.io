---
title: Java IO 知识整理
categories: Java
tags:
  - Java
  - IO
abbrlink: 5feeeebf
date: 2018-11-24 10:30:28
---

Java IO 是一套 Java 用来读写数据（输入和输出）的 API。大部分程序都要处理一些输入，并由输入产生一些输出。Java 为此提供了 `java.io` 包。`java.io` 包几乎包含了所有操作输入、输出需要的类。所有这些流类代表了输入源和输出目标。`java.io` 包中的流支持很多种格式，比如：基本类型、对象、文件、网络数据流、内存缓冲等等。

一个流可以理解为一个数据的序列。输入流表示从一个源读取数据，输出流表示向一个目标写数据。Java 为 I/O 提供了强大的而灵活的支持，使其更广泛地应用到文件传输和网络编程中。

## Java IO 的用途 ##
Java IO 中包含了许多 `InputStream`、`OutputStream`、`Reader`、`Writer`的子类。这样设计的原因是让每一个类都负责不同的功能。这也就是为什么 IO 包中有这么多不同的类的缘故。各类用途汇总如下：
 - 文件访问
 - 网络访问
 - 内存缓存访问
 - 线程内部通信(管道)
 - 缓冲
 - 过滤
 - 解析
 - 读写文本(Readers / Writers)
 - 读写基本类型数据(long, int etc.)
 - 读写对象

> 当通读过 Java IO 类的源代码之后，我们很容易就能了解这些用途。这些用途或多或少让我们更加容易地理解，不同的类用于针对不同业务场景。

## Java IO 关系图 ##
Java InputStream UML 类关系图如下：
![Java InputStream UML](https://henleylee.github.io/medias/java/io_inputstream_uml.png)

Java OutputStream UML 类关系图如下：
![Java OutputStream UML](https://henleylee.github.io/medias/java/io_outputstream_uml.png)

Java Reader UML 类关系图如下：
![Java Reader UML](https://henleylee.github.io/medias/java/io_reader_uml.png)

Java Writer UML 类关系图如下：
![Java Writer UML](https://henleylee.github.io/medias/java/io_writer_uml.png)

## Java IO 类的关系 ##
### InputStream ###
InputStream：Java IO 中的顶级的字节输入流的抽象类，定义了最基础的输入、读取的相关方法。实现了 `Closeable` 接口。
 - FileInputStream：继承自 `InputStream` 的文件输入流类，用于从本地文件中读取字节数据。
 - ByteArrayInputStream：继承自 `InputStream` 的字节数组输入流类，它包含一个内部缓冲区，该缓冲区包含从流中读取的字节；通俗点说，它的内部缓冲区就是一个字节数组，而 `ByteArrayInputStream` 本质就是通过字节数组来实现的。`InputStream` 通过 `read()` 向外提供接口，供它们来读取字节数据；而 `ByteArrayInputStream` 的内部额外的定义了一个计数器，它被用来跟踪 `read()` 方法要读取的下一个字节。
 - StringBufferInputStream：继承自 `InputStream` 的字节输入流类，其中读取的字节由字符串的内容提供的输入流。该类已过时，不推荐使用，由 `StringReader` 代替。
 - PipedInputStream：继承自 `InputStream` 的管道输入流类，在使用管道通信时，必须与 `PipedOutputStream` 配合使用。让多线程可以通过管道进行线程间的通讯。
 - ObjectInputStream：继承自 `InputStream` 的对象输入流类，实现了 `ObjectInput` 和 `ObjectStreamConstants` 接口。作用是从输入流中读取 Java 对象和基本数据。只有支持 `Serializable`  或 `Externalizable` 接口的对象才能被 `ObjectInputStream/ObjectOutputStream` 所操作！
 - SequenceInputStream：继承自 `InputStream` 的输入合并流类。`SequenceInputStream` 会将与之相连接的流集组合成一个输入流并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末 尾为止。合并流的作用是将多个源合并合一个源。
 - AudioInputStream：继承自 `InputStream` 的音频输入流类。音频输入流是具有指定音频格式和长度的输入流。长度用示例帧表示，不用字节表示。提供几种方法，用于从流读取一定数量的字节，或未指定数量的字节。音频输入流跟踪所读取的最后一个字节。可以跳过任意数量的字节以到达稍后的读取位置。音频输入流可支持标记。设置标记时，会记住当前位置，以便可以稍后返回到该位置。
 - FilterInputStream：继承自 `InputStream` 的过滤输入流类（装饰器超类），是用来“封装其它的输入流，并为它们提供额外的功能”。
   - BufferedInputStream：继承自 `FilterInputStream` 的带缓冲区功能的输入流类（装饰器子类），默认缓冲区大小是8K，能够减少访问磁盘的次数，提高文件读取性能。
   - DataInputStream：继承自 `FilterInputStream` 的数据输入流类，实现了 `DataInput` 接口。它允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型。
   - PushbackInputStream：继承自 `FilterInputStream` 的回退输入流类。允许试探性的读取数据流，如果不是我们想要的则返还回去。
   - LineNumberInputStream：继承自 `FilterInputStream` 的行号输入流类。可以获取当前的行号或设置当前行号，已过时，已经被 `LineNumberReader` 替代。
   - CheckedInputStream：继承自 `FilterInputStream` 的校验输入流类。
   - CipherInputStream：继承自 `FilterInputStream` 的密钥输入流类。
   - DigestInputStream：继承自 `FilterInputStream` 的摘要处理输入流类。
   - InflaterInputStream：继承自 `FilterInputStream` 的解压缩处理输入流类。
     - GZIPInputStream：继承自 `InflaterInputStream` 的 gzip 文件处理输入流类。
     - ZipInputStream：继承自 `InflaterInputStream` 的解压缩处理输入流类。
       - JarInputStream：继承自`ZipInputStream` 的解压缩处理输入流类。
   - DeflaterInputStream：继承自 `FilterInputStream` 的压缩数据输入流类。
   - ProgressMonitorInputStream：继承自 `FilterInputStream` 的进度监控输入流类。

### OutputStream ###
OutputStream：Java IO 中的顶级的字节输出流的抽象类，定义了最基础的输出、写入的相关方法。实现了 `Closeable` 和 `Flushable` 接口。
 - FileOutputStream：继承自 `OutputStream` 的文件输出流类，用于向本地文件中写入字节数据。
 - ByteArrayOutputStream：继承自 `OutputStream` 的字节数组输出流类，`ByteArrayOutputStream` 中的数据会被写入一个 `byte` 数组。缓冲区会随着数据的不断写入而自动增长。可使用 `toByteArray()` 和 `toString()` 获取数据。
 - PipedOutputStream：继承自 `OutputStream` 的管道输出流类，在使用管道通信时，必须与 `PipedInputStream` 配合使用。让多线程可以通过管道进行线程间的通讯。
 - ObjectOutputStream：继承自 `OutputStream` 的对象输出流类，实现了 `ObjectOutput` 和 `ObjectStreamConstants` 接口。作用是把 Java 对象和基本数据写入到对象输出流中。只有支持 `Serializable` 或 `Externalizable` 接口的对象才能被 `ObjectInputStream/ObjectOutputStream` 所操作！
 - FilterOutputStream：继承自 `OutputStream` 的过滤输出流类，是用来“封装其它的输出流，并为它们提供额外的功能”。
   - BufferedOutputStream：继承自 `FilterOutputStream` 的带缓冲区功能的输出流类，默认缓冲区大小是8K，能够提高文件的写入效率。
   - DataOutputStream：继承自 `FilterOutputStream` 的数据输出流类，实现了 `DataOutput` 接口。它允许应用程序以与机器无关方式向底层输入流中写入基本 Java 数据类型。
   - PrintStream：继承自 `FilterOutputStream` 的打印输出流类，实现了 `Appendable` 和 `Closeable` 接口。使它们能够方便地打印各种数据值表示形式。`PrintStream` 永远不会抛出 IOException。`PrintStream` 提供了自动flush 和 字符集设置功能。所谓自动 `flush`，就是往 `PrintStream` 写入的数据会立刻调用 `flush()` 函数。
   - CheckedOutputStream：继承自 `FilterOutputStream` 的校验输出流类。
   - CipherOutputStream：继承自 `FilterOutputStream` 的密钥输出流类。
   - DigestOutputStream：继承自 `FilterOutputStream` 的摘要处理输出流类。
   - InflaterOutputStream：继承自 `FilterOutputStream` 的解压缩处理输出流类。
   - DeflaterOutputStream：继承自 `FilterOutputStream` 的解压缩数据输出流类。
     - GZIPOutputStream：继承自 `DeflaterOutputStream` 的 gzip 文件解压缩输出流类。
     - ZipOutputStream：继承自 `DeflaterOutputStream` 的 zip 文件解压缩输出流类。
       - JarOutputStream：继承自 `ZipOutputStream` 的 zip 文件解压缩输出流类。

### Reader ###
- Reader：Java IO 中的顶级的字符读取的抽象类，定义了最基础的读取方法。实现了 `Readable` 和 `Closeable` 接口。
 - BufferedReader：继承自 `Reader` 的带缓冲功能的字符流类，默认缓冲区大小是8K，从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。创建 `BufferReader` 时，我们会通过它的构造函数指定某个 `Reader` 为参数。`BufferReader` 会将该 `Reader` 中的数据分批读取，每次读取一部分到缓冲中；操作完缓冲中的这部分数据之后，再从 `Reader` 中读取下一部分的数据。
   - LineNumberReader：继承自 `BufferedReader`，可以获取当前的行号或设置当前行号。
 - InputStreamReader：继承自 `Reader`，用于将从字节流转换成字符流的 `Reader`。是字节流通向字符流的桥梁。如果不指定字符集编码，该解码过程将使用平台默认的字符编码。
   - FileReader：继承自 `InputStreamReader`，用来读取字符文件的便捷类。
 - StringReader：继承自 `Reader`，用于字符串读取的字符流。
 - CharArrayReader：继承自 `Reader` 的字符数组输入流类。
 - FilterReader：继承自 `Reader` 的字符过滤输入流抽象类。
   - PushbackReader：继承自 `FilterReader` 的字符回退输入流类。
 - PipedReader：继承自 `Reader` 的字符管道输入流类。作用是可以通过管道进行线程间的通讯。必须和 `PipedWriter` 配合使用。

### Writer ###
- Writer：Java IO 中的顶级的字符写入的抽象类，定义了最基础的写入方法。实现了 `Appendable`、`Closeable` 和 `Flushable` 接口。
 - BufferedWriter：继承自`Writer`的带缓冲功能的字符流类，默认缓冲区大小是8K，从字符输出流中写入字符到文本中，缓冲各个字符，从而实现字符、数组和行的高效写入。
 - OutputStreamWriter：继承自 `Reader`，用于将从字节流转换成字符流的 `Writer`。是字节流通向字符流的桥梁。如果不指定字符集编码，该解码过程将使用平台默认的字符编码。
   - FileWriter：继承自 `OutputStreamWriter`，用来向文件中写入字符的便捷类。
 - FilterWriter：继承自 `Writer` 的字符过滤输出流抽象类。与 `FilterOutputStream` 功能一样、只是简单重写了父类的方法、目的是为所有装饰类提供标准和基本的方法、要求子类必须实现核心方法、和拥有自己的特色。这里 `FilterWriter` 没有子类、可能其意义只是提供一个接口、留着以后的扩展，本身是一个抽象类。
 - StringWriter：继承自 `Writer`，用于字符串写入的字符流。
 - PipedWriter：继承自 `Writer` 的字符管道输出流类。作用是可以通过管道进行线程间的通讯。必须和 `PipedReader` 配合使用。
 - CharArrayWriter：继承自 `Writer` 的字符数组输出流类。
 - PrintWriter：继承自 `Writer` 的打印写入类，提供了 `PrintStream` 的所有打印方法，其方法也从不抛出 IOException。与 `PrintStream` 的区别：作为处理流使用时，`PrintStream` 只能封装 `OutputStream` 类型的字节流，而 `PrintWriter` 既可以封装 `OutputStream` 类型的字节流，还能够封装 `Writer` 类型的字符输出流并增强其功能。

![Java IO Mind](https://henleylee.github.io/medias/java/io_mind_all.png)

## Java IO 的分类 ##
 - **`字节流和字符流`**：
   - `字节流：`以字节为单位，每次次读入或读出是8位数据。可以读任何类型数据。
   - `字符流：`以字符为单位，每次次读入或读出是16位数据。其只能读取字符类型数据。
 - **`输出流和输入流`**：
   - `输入流：`从文件读入到内存。只能进行读操作。
   - `输出流：`从内存读出到文件。只能进行写操作。
 - **`节点流和处理流`**：
   - `节点流：`直接与数据源相连，读入或读出。
   - `处理流：`与节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流。处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

> **注**：输入输出流中的入和出，都是相对于系统内存而言的。为什么要有处理流？直接使用节点流，读写不方便，为了更快的读写文件，才有了处理流。

流按类型分类的结构图如下：
![流按类型分类](https://henleylee.github.io/medias/java/io_mind_optype.png)

流按用途分类的结构图如下：
![流按用途分类](https://henleylee.github.io/medias/java/io_mind_object.png)

## Java IO 的主要方法 ##
### InputStream 中的主要方法 ###

| 方法                                    | 描述                                                                                                           |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `abstract int read()`                   | 从流中读取数据，读取一个字节，返回值为所读得字节                                                               |
| `int read(byte b[])`                    | 从流中读取数据，读取多个字节，放置到字节数组 b 中，通常读取的字节数量为 b 的长度，返回值为实际独取的字节的数量 |
| `int read(byte b[], int off, int len)`  | 从流中读取数据，读取 len 个字节，放置到以下标 off 开始字节数组 b 中，返回值为实际读取的字节的数量              |
| `long skip(long n)`                     | 读指针跳过 n 个字节不读，返回值为实际跳过的字节数量                                                            |
| `int available()`                       | 返回值为流中尚未读取的字节的数量                                                                               |
| `void close()`                          | 关闭输入流                                                                                                     |
| `synchronized void mark(int readlimit)` | 记录当前指针的所在位置，`readlimit` 表示读指针读出的 `readlimit` 个字节后，所标记的指针位置才实效              |
| `synchronized void reset()`             | 把读指针重新指向用 `mark` 方法所记录的位置                                                                     |
| `boolean markSupported()`               | 当前的流是否支持读指针的记录功能                                                                               |

> **注**：其中`read()`返回的是读入的一个字节所对应的 int 值(0-255),而`read(byte[] b)`和`read(byte[] b, int off, int len)`返回的是读入的字节数。

### OutputStream 中的主要方法 ###

| 方法                                     | 描述                                                                                               |
|------------------------------------------|----------------------------------------------------------------------------------------------------|
| `abstract void write(int b)`             | 输出数据，往流中写一个字节 b                                                                       |
| `void write(byte b[])`                   | 输出数据，往流中写一个字节数组 b                                                                   |
| `void write(byte b[], int off, int len)` | 输出数据，把字节数组 b 中从下标 off 开始，长度为 len 的字节写入到流中                              |
| `void flush()`                           | 刷空输出流，并输出所有被缓存的字节。由于某些流支持缓存功能，该方法将把缓存中所有内容强制输出到流中 |
| `void close()`                           | 关闭输出流                                                                                         |

### Reader 中的主要方法 ###

| 方法                                      | 描述                                 |
|-------------------------------------------|--------------------------------------|
| `int read(java.nio.CharBuffer target)`    | 试图读取字符入指定的字符缓冲区       |
| `int read()`                              | 读取单个字符                         |
| `int read(char cbuf[])`                   | 读取字符到一个数组中                 |
| `int read(char cbuf[], int off, int len)` | 读取字符到一个数组中的一部分         |
| `long skip(long n)`                       | 跳过 n 个字符                        |
| `boolean ready()`                         | 通知此流是否已准备好被读取           |
| `boolean markSupported()`                 | 告诉此流是否支持 `mark()` 操作       |
| `void mark(int readAheadLimit)`           | 标记流中的当前位置                   |
| `void reset()`                            | 重置流                               |
| `void close()`                            | 关闭该流并释放与之关联的所有系统资源 |

### Writer 中的主要方法 ###

| 方法                                                  | 描述                                      |
|-------------------------------------------------------|-------------------------------------------|
| `void write(int c)`                                   | 写入单个字符                              |
| `void write(char cbuf[])`                             | 写入字符数组                              |
| `abstract void write(char cbuf[], int off, int len)`  | 写入字符数组的一部分                      |
| `void write(String str)`                              | 写入一个字符串                            |
| `void write(String str, int off, int len)`            | 写入一个字符串的一部分                    |
| `Writer append(CharSequence csq)`                     | 将指定的字符序列追加写到 writer 中        |
| `Writer append(CharSequence csq, int start, int end)` | 将指定的字符序列的子序列追加写入此 writer |
| `Writer append(char c)`                               | 将指定字符追加到这个 writer               |
| `abstract void flush()`                               | 刷新流                                    |
| `abstract void close()`                               | 关闭流，但要先刷新它                      |

## Java IO 的使用示例 ##
### FileInputStream 和 FileOutputStream ###
```java
/**
 * 测试复制文件 a.txt 中的内容到 b.txt 文件中
 */
private static void testCopyByFileStream() {
    try (
            InputStream in = new FileInputStream("D:/test/a.txt");
            OutputStream out = new FileOutputStream("D:/test/b.txt", true)
    ) {
        int len;
        byte[] bytes = new byte[1024];
        while ((len = in.read(bytes)) != -1) {
            out.write(bytes, 0, len);
        }
    } catch (IOException e) {
        log.error("文件读取写入失败!", e);
    }
}
```

### BufferedInputStream 和 BufferedOutputStream ###
```java
/**
 * 测试通过 BufferedInputStream 和 BufferedOutputStream 来复制文件 a.txt 中的内容到 b.txt 文件中
 */
private static void testCopyByBufferedStream() {
    try (
            InputStream in = new BufferedInputStream(new FileInputStream("D:/test/a.txt"));
            OutputStream out = new BufferedOutputStream(new FileOutputStream("D:/test/b.txt"))
    ) {
        int len;
        byte[] bytes = new byte[1024];
        while ((len = in.read(bytes)) != -1) {
            out.write(bytes, 0, len);
        }
    } catch (IOException e) {
        log.error("通过缓冲区的方式来做文件读取写入失败!", e);
    }
}
```

### ByteArrayOutputStream ###
```java
/**
 * 测试将内容写入到 ByteArrayOutputStream 中并打印出来，不需要关闭流
 */
private static void testByByteArrayStream() {
    String text = "Hello World!";
    ByteArrayOutputStream byteOut = new ByteArrayOutputStream(8);
    try {
        byteOut.write(text.getBytes());
    } catch (IOException e) {
        log.error("写入字节数据出错!", e);
    }

    byte[] buffer = byteOut.toByteArray();
    for (byte b : buffer) {
        log.info("{}", (char) b);
    }
}
```

### PipedInputStream 和 PipedOutputStream ###
```java
/**
 * 发送消息的线程
 */
public class Sender extends Thread {

    private static final Logger log = LoggerFactory.getLogger(Sender.class);

    /** 管道输出流对象，它和管道输入流(PipedInputStream)对象绑定，从而可以将数据发送给“管道输入流” */
    private PipedOutputStream pipedOut;

    public Sender(PipedOutputStream pipedOut) {
        this.pipedOut = pipedOut;
    }

    public PipedOutputStream getPipedOut() {
        return pipedOut;
    }

    @Override
    public void run() {
        String strInfo = "Hello World!";
        try {
            pipedOut.write(strInfo.getBytes());
            pipedOut.close();
        } catch (IOException e) {
            log.error("向管道中写入数据出错!", e);
        }
    }

}
```

```java
/**
 * 接收消息的线程
 */
public class Receiver extends Thread {

    private static final Logger log = LoggerFactory.getLogger(Test.Receiver.class);

    /** 管道输入流对象，它和管道输出流(PipedOutputStream)对象绑定，从而可以接收“管道输出流”的数据 */
    private PipedInputStream pipedIn;

    public Receiver(PipedInputStream pipedIn) {
        this.pipedIn = pipedIn;
    }

    public PipedInputStream getPipedIn() {
        return pipedIn;
    }

    @Override
    public void run() {
        byte[] buf = new byte[2048];
        try {
            int len = pipedIn.read(buf);
            log.info("{}", new String(buf, 0, len));
            pipedIn.close();
        } catch (IOException e) {
            log.error("从管道中读取数据出错!", e);
        }
    }

}
```

```java
/**
 * PipedInputStream 和 PipedOutputStream 的测试类
 */
public class PipedStreamTest {

    private static final Logger log = LoggerFactory.getLogger(PipedStreamTest.class);

    public static void main(String[] args) {
        Sender sender = new Sender(new PipedOutputStream());
        Receiver receiver = new Receiver(new PipedInputStream());

        try {
            // 将管道输入流和管道的输出流进行连接.
            receiver.getPipedIn().connect(sender.getPipedOut());
            // 启动线程
            sender.start();
            receiver.start();
        } catch (IOException e) {
            log.info("发送接收消息出错!", e);
        }
    }

}
```

### SequenceInputStream ###
```java
/**
 * 测试使用 testBySequenceStream 合并输入流来统一读取写入
 */
private static void testBySequenceStream() {
    try (
            InputStream in1 = new FileInputStream("D:/test/a.txt");
            InputStream in2 = new FileInputStream("D:/test/b.txt");
            OutputStream out = new FileOutputStream("D:/test/c.txt");
            SequenceInputStream seqIn = new SequenceInputStream(in1, in2)
    ) {
        int len;
        while ((len = seqIn.read()) != -1) {
            out.write(len);
        }
    } catch (IOException e) {
        log.error("合并输入流写入失败!", e);
    }
}
```

### PrintStream ###
```java
/**
 * 测试使用 PrintStream 将数据输出到指定的文件
 */
private static void testOutputByPrintStream() {
    System.out.println("Hello World!");
    File file = new File("D:/test/d.txt");
    try {
        System.setOut(new PrintStream(new FileOutputStream(file)));
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    System.out.println("这些内容在文件中才能看到哦！");
}
```

## Java IO 具有的对称性 ##
Java I/O库具有两个对称性，它们分别是：
 - **`输入-输出对称性：`** `InputStream` 和 `OutputStream` 各自占据 `Byte` 流的输入与输出的两个平行的等级结构的根部。而 `Reader` 和 `Writer` 各自占据 `Char` 流的输入与输出的两个平行的等级结构的根部。
 - **`byte-char 对称：`** `InputStream` 和 `Reader` 的子类分别负责 `Byte` 和 `Char` 流的输入；`OutputStream` 和 `Writer` 的子类分别负责 `Byte` 和 `Char` 流的输出，它们分别形成平行的等级结构。


## Java IO 遵循的设计原则 ##
- **`单一职责原则：`**每一个 IO 类的都具有单独的功能和职责，需要对某种类型的流做修改时，只需要修改具体对应的 IO 流类即可。
- **`里氏替换原则：`**子类完全实现了父类的方法，且有自己的个性，子类覆盖或实现父类的方法时输入参数和返回结果的范围没有变化。
- **`接口隔离原则：`**输入输出流所实现的接口有所不同，而且接口比较细化，接口中的方法少。
- **`依赖倒置原则：`**Java IO 中的各个具体的功能类，都继承或聚合了 `InputStream/OutputStream` 的抽象类，都共同依赖了抽象，而不是依赖了某个具体的功能的IO类。
- **`开闭原则：`**当用户需要新增一种新特征、新方式的IO流时，可以直接新增一种类的流，让其继承或聚合 `InputStream/OutputStream/FilterInputStream/FilterOutputStream` 等类，从而来书写其特有的功能，减少。
- **`聚合/组合复用原则：`**就比如 `FileInputStream` 中对 `FileDescriptor` 和 `FileChannel` 的使用，都是采用了聚合的方式，而非继承的方式，从而使类之间的耦合度降低，提高了灵活性。

## Java IO 使用到的设计模式 ##
Java 的 I/O 库总体设计是符合装饰者模式(Decorator)跟适配器模式(Adapter)的。这个库中处理流的类叫做流类，比如 FileInputStream、FileOutputStream、DataInputStream 及 DataOutputStream 都是流处理器的例子。

### 装饰者模式(Decorator) ###
由于 Java I/O 库需要很多性能的各种组合，如果这些性能都是用继承来实现，那么每一种组合都需要一个类，这样就会造成大量行重复的类出现。如果采用装饰模式，那么类的数目就会大大减少，性能的重复也可以减至最少。因此装饰模式是 Java I/O 库基本模式。装饰模式的引进，造成灵活性和复杂性的提高。因此在使用 Java I/O 库时，必须理解 Java I/O 库是由一些基本的原始流处理器和围绕它们的装饰流处理器所组成的。

在由 `InputStream`、`OutputStream`、`Reader` 和 `Writer` 代表的等级结构内部，有一些流处理器可以对另一些流处理器起到装饰作用，形成新的，具有改善了的功能的流处理器。装饰者模式是 Java I/O 库的整体设计模式。以字节输入流为例，`InputStream` 是需要被装饰的抽象构件，而其下的 `FileInputStream`、`ByteArrayInputStream` 等是被装饰的具体构件；`FilterInputStream` 是装饰器的父类，`BufferedInputStream`、`DataInputStream` 则是具体的装饰器类。

### 适配器模式(Adapter) ###
适配器模式是 Java I/O 库中第二个重要的设计模式。适配器模式应用到了原始流处理器的设计上面，构成了 I/O 库所有流处理器的起点。

在由 `InputStream`、`OutputStream`、`Reader` 和 `Writer` 代表的等级结构内部，有一些流处理器是对其它类型的流源的适配。如：`StringBufferInputStream` 就是一个适配器类。它继承了 `InputStream` 类型，同时持有一个对 `String` 类型的引用。这是将 `String` 对象适配成 `InputStream` 类型的对象形式的适配器模式。`PipedOutputStream` 也是一个适配器类。`PipedOutputStream` 总是和 `PipedInputStream` 一起使用，它接收一个类型为 `PipedInputStream` 的输入类型，并将之转换成 `OutputStream` 类型的输出流，这是一个对象形式的适配器模式应用。

## Java IO 中的流处理器 ##
根据输入流的源的类型，可以将这些流类分成两种，即原始流类(Original Stream)和链接流处理器(Wrapper Stream)。

### 原始流处理器 ###
原始流处理器接收一个 Byte 数组对象、String 对象、FileDiscriptor 对象或者不同类型的流源对象。原始流处理器包括：
 - ByteArrayInputStream：为多线程的通信提供缓冲区操作功能，接收一个 Byte 数组作为流的源。
 - FileInputStream：建立一个与文件有关的输入流。接收一个 File 对象作为流的源。
 - PipedInputStream：可以与 PipedOutputStream 配合使用，用于读入一个数据管道的数据，接收一个 PipedOutputStream 作为源。
 - StringBufferInputStream：将一个字符串缓冲区转换为一个输入流。接收一个 String 对象作为流的源(JDK 帮助文档上说明：已过时。此类未能正确地将字符转换为字节。从 JDK 1.1 开始，从字符串创建流的首选方法是通过 StringReader 类进行创建。只有字符串中每个字符的低八位可以由此类使用)。


### 链接流处理器 ###
链接流处理器就是可以接收另一个流对象作为源，并对之进行功能扩展的类。InputStream类型的链接处理器包括以下几种，它们都接收另一个 InputStream 对象作为流源：
 - FilterInputStream：称为过滤输入流，它将另一个输入流作为流源。这个类的子类包括以下几种：
   - BufferedInputStream：用来从硬盘将数据读入到一个内存缓冲区中，并从缓冲区提供数据。
   - DataInputStream：提供基于多字节的读取方法，可以读取原始类型的数据。
   - LineNumberInputStream：提供带有行计数功能的过滤输入流。
   - PushbackInputStream：提供特殊的功能，可以将已经读取的字节“推回”到输入流中。
 - ObjectInputStream：可以将使用 ObjectInputStream 串行化的原始数据类型和对象重新并行化。
 - SeqcueneInputStream：可以将两个已有的输入流连接起来，形成一个输入流，从而将多个输入流排列构成一个输入流序列。

## Java IO 设计的优缺点 ##
### 优点 ###
 - 功能全面、强大和灵活，同时还防止了过多的类膨胀。
 - 符合开闭原则，开发可以通过各种装配能实现各种功能。

### 缺点 ###
 - 复杂难记，API 使用繁琐。如：`JDK 1.6`中，从文本文件中读取数据转换为字符串，就得至少10行代码。
 - IO 流中的绝大多数方法都有 `IOException` 的检查异常，导致开发人员不得不在各个使用到的地方捕捉或再抛出异常，代码繁琐，不符合当前的主流设计思路。

