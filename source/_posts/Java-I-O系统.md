---
title: Java I/O系统
date: 2021-01-14 16:51:43
tags:
---

对于程序语言的设计者而言，创建一个好的输入输出`I/O`系统是一项艰难的任务。

## File类

它既能代表一个特定文件的名称，又能够代表一个目录下的一组文件的名称。

`File`类和`FilenameFilter`接口

```
public class TextFile {

    public static void main(String[] args) {
        File path = new File(".");
        String[] list;
        if(args.length == 0) {
            list = path.list();
        } else {
            list = path.list(new DirFilter(args[0]));
        }
        Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
        for (int i = 0; i < list.length; i++) {
            System.out.println(list[i]);
        }
    }

}

class DirFilter implements FilenameFilter{

    private Pattern pattern;

    public DirFilter(String regex) {
        this.pattern = Pattern.compile(regex);
    }

    @Override
    public boolean accept(File dir, String name) {
        return pattern.matcher(name).matches();
    }
}
```

Directory用于获取regex匹配的文件名称

```
public class Directory {

    public static File[] local(File dir, final String regex) {
        return dir.listFiles(new FilenameFilter() {
            private Pattern pattern = Pattern.compile(regex);

            @Override
            public boolean accept(File dir, String name) {
                return pattern.matcher(new File(name).getName()).matches();
            }
        });
    }

    public static File[] local(String path, final String regex) {
        return local(new File(path), regex);
    }

    public static TreeInfo walk(String start, String regex) {
        return recurseDirs(new File(start), regex);
    }

    public static TreeInfo walk(File start, String regex) {
        return recurseDirs(start, regex);
    }

    public static TreeInfo walk(File start) {
        return recurseDirs(start, ".*");
    }

    public static TreeInfo walk(String start) {
        return recurseDirs(new File(start), ".*");
    }

    static TreeInfo recurseDirs(File startDir, String regex) {
        TreeInfo result = new TreeInfo();
        for (File item : Objects.requireNonNull(startDir.listFiles())) {
            if (item.isDirectory()) {
                result.dirs.add(item);
                result.addAll(recurseDirs(item, regex));
            } else {
                if (item.getName().matches(regex)) {
                    result.files.add(item);
                }
            }
        }
        return result;
    }

    public static class TreeInfo implements Iterable<File> {

        public List<File> files = new ArrayList<>();
        public List<File> dirs = new ArrayList<>();

        @Override
        public Iterator<File> iterator() {
            return files.iterator();
        }

        void addAll(TreeInfo other) {
            files.addAll(other.files);
            dirs.addAll(other.dirs);
        }

        @Override
        public void forEach(Consumer<? super File> action) {

        }

        @Override
        public Spliterator<File> spliterator() {
            return null;
        }

        @Override
        public String toString() {
            return "TreeInfo{" +
                    "files=" + files +
                    ", dirs=" + dirs +
                    '}';
        }
    }

    /**
     * test
     */
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println(walk("."));
        } else {
            for (String arg : args) {
                System.out.println(walk(arg));
            }
        }
    }

}
```

根据策略处理regex匹配到的文件

```
public class ProcessFiles {
    /**
     * 策略接口
     */
    public interface Strategy {
        void process(File file);
    }

    private Strategy strategy;
    private String ext;

    public ProcessFiles(Strategy strategy, String ext) {
        this.strategy = strategy;
        this.ext = ext;
    }

    public void start(String[] args) {
        try {
            if (args.length == 0) {
                processDirectoryTree(new File("."));
            } else {
                for (String arg : args) {
                    File fileArg = new File(arg);
                    if (fileArg.isDirectory()) {
                        processDirectoryTree(fileArg);
                    } else {
                        if (!arg.endsWith("." + ext)) {
                            arg += "." + ext;
                            strategy.process(new File(arg).getCanonicalFile());
                        }
                    }

                }
            }

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public void processDirectoryTree(File root) {
        for (File file : Directory.walk(root.getAbsolutePath(), ".*\\." + ext)) {
            strategy.process(file);
        }
    }

    /**
     * test
     */
    public static void main(String[] args) {
        new ProcessFiles(System.out::println, "java").start(args);
    }

}
```

目录检查及创建

```
public class MakeDirectories {

    public static void main(String[] args) {
        File file = new File("src/io/MakeDirectories.java");
        System.out.println(file.getAbsolutePath());
        System.out.println(file.getName());
        System.out.println(file.canWrite());
        System.out.println(file.canRead());
        System.out.println(file.length());
        System.out.println(file.getParent());
        System.out.println(file.isFile());
        System.out.println(file.isDirectory());
        System.out.println(file.lastModified());
    }

}
```

## 输入和输出

编程语言的I/O类库中常使用**流**这个概念，它代表任何有能力产出数据的数据源对象或者是有能力接收数据的接收端对象。**流**屏蔽了实际的I/O设备中处理数据的细节。

### InputStream类型

Inputstream的作用是用来表示那些从不同数据源产生输入的类。

1. 字节数组
2. String
3. 文件
4. **管道**，工作方式与实际的管到类似，即，从一端输入，从另一端输出。
5. 一个由其他种类的流组成的序列，以便我们可以将它们收集合并到一个流内。
6. 其他数据源，如Internet连接等。

| 类                      | 功能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ByteArrayInputStream    | 允许将内存的缓冲区当做InputStream使用                        |
| StringBufferInputStream | 将String转换成InputStream                                    |
| FileInputStream         | 用于从文件中读取信息                                         |
| PipedInputStream        | 产生用于写入相关PipedOutputStream的数据。实现管道化概念      |
| SequenceInputStream     | 将两个或多个InputStream对象转换成单一InputStream             |
| FilterInputStream       | 抽象类，作为***装饰器***的接口。其中***装饰器***为其他InputStream的类提供有用功能 |

### OutputStream类型

OutputStream决定了输出索要去往的目标：字节数组（但不是String，不过可以自己用字节数组创建String）、文件或管道。

| 类                    | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| ByteArrayOutputStream | 在内存中创建缓冲区。所有送往***流***的数据都要放置在此缓冲区 |
| FileOutputStream      | 用于将信息写入至文件                                         |
| PipedOutputStream     | 任何写入其中的信息都会自动为相关PipedInputStream的输入。实现管道化概念 |
| FilterOutputStream    | 抽象类，作为***装饰器***的接口。其中，***装饰器***为其他OutputStream提供有用的功能 |

## 添加属性和有用的接口

FilterInputStream和FilterOutputStream是用来提供装饰器接口以控制特定输入流和输出流的两个类，他们的名字并不是很直观。

### 通过FilterInputStream从InputStream读取数据

FilterInputStream类型

| 类                    | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| DataInputStream       | 与DataOutputStream搭配使用，因此我们可以按照可移植方式从流读取基本数据类型**int**、**char**、**long**等 |
| BufferedInputStream   | 使用它可以防止每次读取时都进行实际写操作，代表**使用缓冲区** |
| LineNumberInputStream | 跟踪输入流中的行号；可调用getLineNumber()和setLineNumber(int) |
| PushbackInputStream   | 具有**能弹出一个字节的缓冲区**。因此可以将读取到的最后一个字符回退 |

### 通过FilterOutputStream向OutputStream写入

FilterOutputStream类型

| 类                   | 功能                                                         |
| -------------------- | ------------------------------------------------------------ |
| DataOutputStream     | 与DataInputStream搭配使用，因此可以按照可移植方式向流中写入基本类型数据**int** **char** **long**等 |
| PrintStream          | 用于产出格式化输出。其中DataOutputStream处理数据的存储，PrintStream处理显示 |
| BufferedOutputStream | 使用它以避免每次发送数据时都要要进行实际的操作。代表**使用缓冲区**。可以调用flush()清空缓冲区 |

## Reader和Writer

Reader和Writer提供兼容Unicode与面向***字符***的I/O功能。

InputStream和OutputStream提供面向***字节***的功能。

## 自我独立的类：RandomAccessFile

RandomAccessFile适用于由大小已经知道的记录组成的文件，所以我们可以使用seek()将记录从一处转移到另一处，然后读取或修改几率。文件中记录的大小不一定都相同，只要我们能够确定那些记录有多大以及他们在文件中的位置即可。

## I/O流的经典使用方式

尽管可以通过不同的方式组合I/O流类，但是我们可能也就只用到其中的几种组合。

### 缓冲输入文件

```java
package io;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class BufferedInputFile {

    public static String read(String filename) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader(filename));
        String s;
        StringBuilder sb = new StringBuilder();
        while((s = in.readLine()) != null) {
            sb.append(s).append("\n");
        }
        in.close();
        return sb.toString();
    }

    public static void main(String[] args) throws IOException {
        System.out.println(read("src/io/BufferedInputFile.java"));
    }

}
```

### 从内存中输入

```
package io;

import java.io.IOException;
import java.io.StringReader;

public class MemoryInput {

    public static void main(String[] args) throws IOException {
        StringReader in = new StringReader(BufferedInputFile.read("src/io/MemoryInput.java"));
        int c;
        while ((c = in.read()) != -1) {
            System.out.println((char) c);
        }
    }

}
```

### 格式化的内存输入

```
package io;

import java.io.ByteArrayInputStream;
import java.io.DataInputStream;
import java.io.IOException;

public class FormattedMemoryInput {

    public static void main(String[] args) {
        try {
            DataInputStream in = new DataInputStream(new ByteArrayInputStream(BufferedInputFile.read("src/io/FormattedMemoryInput.java").getBytes()));
            // 检查剩余字符
            while (in.available() != 0) {
                System.out.println((char) in.readByte());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

### 基本的文件输出

```
package io;

import java.io.*;

public class BasicFileOutput {

    static String file = "src/io/BasicFileOutput";

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(BufferedInputFile.read("src/io/BasicFileOutput.java")));
        PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(file)));
        int lineCount = 1;
        String s;
        while((s = in.readLine()) != null) {
            out.println(lineCount++ + ": " + s);
        }
        out.close();
        System.out.println(BufferedInputFile.read("src/io/BasicFileOutput"));
    }

}
```

### 文本文件输出的快捷方式

```
package io;

import java.io.*;

public class FileOutputShortcut {

    static String file = "src/io/FileOutputShortcut";

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(BufferedInputFile.read("src/io/FileOutputShortcut.java")));
        PrintWriter out = new PrintWriter(file);
        int lineCount = 1;
        String s;
        while ((s = in.readLine()) != null) {
            out.println(lineCount++ + "\t :" + s);
        }
        out.close();
        System.out.println(BufferedInputFile.read(file));
    }
    
}
```

### 存储和恢复数据

PrintWriter可以对数据进行格式化，以方便人们的阅读。但是为了输出可供另一个**流**恢复的数据，我们需要使用DataOutputStream写入数据，并用DataInputStream恢复数据。

 ```
package io;

import java.io.*;

public class StoringAndRecoveringData {

    public static void main(String[] args) throws IOException {
        String filePath = "src/io/Data";
        DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream(filePath)));
        out.writeDouble(3.1415926);
        out.writeUTF("That was pi");
        out.writeDouble(1.41413);
        out.writeUTF("Square root of 2");
        out.close();
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream(filePath)));
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
    }

}
 ```

保证数据一致性：

1. 为文件中的数据采用固定的格式
2. 将额外的信息保存在文件中，以便能够对其进行解析以确定数据的存放位置

### 读写随机访问文件

使用RandomAccessFile类

### 管道流

PipedInputStream、PipedOutputStream、PipedReader和PipedWriter管道流用于任务之间的通信，在并发中使用。

## 文件读写的实用工具

```
package io;

import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.TreeSet;

/**
 * @author: luu
 * @date: 2021-01-15 11:49
 **/
public class MyTextFile extends ArrayList<String> {

    public static String read(String fileName) {
        StringBuilder sb = new StringBuilder();
        try {
            try (BufferedReader in = new BufferedReader(new FileReader(new File(fileName).getAbsoluteFile()))) {
                String s;
                while ((s = in.readLine()) != null) {
                    sb.append(s).append("\n");
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return sb.toString();
    }

    public static void write(String fileName, String text) {
        try {
            try (PrintWriter out = new PrintWriter(new File(fileName).getAbsoluteFile())) {
                out.print(text);
            }
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public MyTextFile(String fileName, String splitter) {
        super(Arrays.asList(read(fileName).split(splitter)));
        if ("".equals(get(0))) {
            remove(0);
        }
    }

    public MyTextFile(String fileName) {
        this(fileName, "\n");
    }

    public void write(String fileName) {
        try {
            try (PrintWriter out = new PrintWriter(new File(fileName).getAbsoluteFile())) {
                for (String item : this) {
                    out.println(item);
                }
            }
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        String file = read("src/io/MyTextFile.java");
        write("src/io/MyTextFile", file);
        MyTextFile myTextFile = new MyTextFile("src/io/MyTextFile");
        myTextFile.write("src/io/MyTextFile2");
        TreeSet<String> words = new TreeSet<>(new MyTextFile("src/io/MyTextFile.java", "\\W+"));
        System.out.println(words.headSet("a"));
    }

}
```

### 读取二进制文件

```
package io;

import java.io.*;

public class BinaryFile {

    public static byte[] read(File bFile) throws IOException {
        try (BufferedInputStream bf = new BufferedInputStream(new FileInputStream(bFile))) {
            byte[] data = new byte[bf.available()];
            bf.read(data);
            return data;
        }
    }

    public static byte[] read(String bFile) throws IOException {
        return read(new File(bFile).getAbsoluteFile());
    }

    public static void main(String[] args) throws IOException {
        byte[] read = read("src/io/BinaryFile.java");
        for (byte b : read) {
            System.out.println(b);
        }
    }

}
```

## 标准I/O

标准I/O这个属于参考的是Unix中**程序所使用的单一信息流**的概念。

标准I/O的意义在于，我们可以很容易的把程序串联起来，一个程序的标准输出可以成为另一个程序的标准输入。这是一个强大的工具。

### 从标准输入中读取

按照标准I/O模型，Java提供了`System.in` `System.out` `System.err`。

### 将System.out转换成PrintWriter

```
package io;

import java.io.PrintWriter;

public class ChangeSystemOut {

    public static void main(String[] args) {
        PrintWriter out = new PrintWriter(System.out, true);
        out.println("Hello, world");
    }

}
```

 ### 标准I/O重定向

Java的`System`类提供了简单的静态方法调用，以允许我们对标准输入、输出和错误I/O流进行重定向：

+ setIn(InputStream)
+ setOut(PrintStream)
+ setErr(PrintStream)

如果在显示器上创建大量输出，并且滚动太快以至于无法阅读，此时重定向就显得极为有用。

## 进程控制

经常会需要在Java内部执行其他操作系统程序，并且要控制这些程序的输入和输出。Java类库提供了执行这些操作的类`Process`类。

```
package io;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class OSExecute {

    public static void command(String command) {
        boolean err = false;
        try {
            Process process = new ProcessBuilder(command.split(" ")).start();
            BufferedReader result = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String s;
            while((s = result.readLine()) != null) {
                System.out.println(s);
            }
            BufferedReader errors = new BufferedReader(new InputStreamReader(process.getErrorStream()));
            while((s = errors.readLine()) != null) {
                System.out.println(s);
                err = true;
            }
        } catch (Exception e) {
            if(!command.startsWith("CMD /C")){
                command("CMD /C" + command);
            } else {
                throw new RuntimeException(e);
            }
        }
        if (err) {
            throw new RuntimeException("Errors executing " + command);
        }
    }

    public static void main(String[] args) {
        command("git status");
    }

}
```

## 新I/O

JDK1.4中`java.nio.*`包中引入了新的JavaI/O类库，其目的在于提高速度。

速度的提高来自于所使用的的结构更接近于操作系统执行I/O的方式：**通道**和**缓冲器**。

唯一与**通道**打交道的是**缓冲器**``ByteBuffer`，使用时我们只需与**缓冲器**打交道。

```
package io;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * @author: luu
 * @date: 2021-01-15 13:11
 **/
public class GetChannel {

    private static final int BAIZE = 1024;
    private static final String FILE_PATH = "src/io/GetChannel";

    public static void main(String[] args) throws IOException {
        FileChannel fc = new FileOutputStream(FILE_PATH).getChannel();
        fc.write(ByteBuffer.wrap("Some text ".getBytes()));
        fc.close();
        fc = new RandomAccessFile(FILE_PATH, "rw").getChannel();
        fc.position(fc.size());
        fc.write(ByteBuffer.wrap("Some more".getBytes()));
        fc.close();
        fc = new FileInputStream(new File(FILE_PATH)).getChannel();
        ByteBuffer buff = ByteBuffer.allocate(BAIZE);
        fc.read(buff);
        buff.flip();
        while (buff.hasRemaining()) {
            System.out.print((char) buff.get());
        }
    }

}
```

`ByteBuffer`使用`allocate()`方法来分配ByteBuffer。

`ByteBuffer`需要调用`flip()`方法让它做好被别人读取的准备。

`ByteBuffer`需要调用`clear()`方法清空**缓冲器**，让它为下一个read做好准备。

`FileChannel.read()`返回`-1`时，表示已经到达了输入的末尾。

可以使用`transferTo()` `transferFrom()`方法将一个通道和另一个通道连接。

### 转换数据

使用`java.nio.charset.Charset`类，该类提供了把数据编码成多种不同类型的字符集工具。

### 获取基本类型

向`ByteBuffer`插入基本类型数据的最简单方法是利用`asCharBuffer()` `asShortBuffer()`等获取该缓冲器上的视图，然后使用视图上的put方法。

### 视图缓冲器

视图缓冲器`view buffer`可以让我们通过某个特定的基本数据类型的视窗查看其地城的`ByteBuffer`。

### 字节存放顺序

不同的机器可能会使用不同的字节排序方法来存储数据。`big endian`高位优先将最重要的字节存放在地址最低位的存储单元，`little endian`低位优先则是将最重要的字节放在地址最高的存储器单元。

`ByteBuffer`默认是以高位优先的相识存储数据的。

我们可以使用`ByteOrder.BIG_ENDIAN`或`ByteOrder.LITTLE_ENDIAN`的`order()`方法改变其字节排列顺序。

### 用缓冲器操纵数据

### 缓冲器的细节

### 内存映射文件

内存映射文件允许我们创建和修改哪些因为太大而不能存入内存的文件。

### 文件加锁

JDK1.4引入了文件加锁机制，运维我们同步访问某个作为共享资源的文件。

文件锁对于其他的操作系统进程是可见的，因为Java的文件加锁直接映射到了本地操作系统的加锁工具。

### 对映射文件的部分加锁

文件映射通常应用于极大的文件。我们可能需要对这种巨大的文件进行部分加锁，以便其他的进程可以修改文件中未被加锁的部分。例如数据库就是这样，因此过个用户可以同时访问他。

## 压缩

Java I/O类库中的类支持读写压缩格式的数据流。

| 压缩类               | 功能                                                   |
| -------------------- | ------------------------------------------------------ |
| CheckedInputStream   | GetCheckSum()为任何InputStream产生校验和               |
| CheckedOutputStream  | GetCheckSum()为任何OutputStream产生校验和              |
| DeflaterOutputStream | 压缩类的基类                                           |
| ZipOutputStream      | 一个DeflaterOutputStream，用于将数据压缩成Zip文件格式  |
| GZIPOutputStream     | 一个DeflaterOutputStream，用于将数据压缩成GZIP文件格式 |
| InflaterInputStream  | 解压类的基类                                           |
| ZipInputStream       | 一个InflaterInputStream，用于解压Zip文件格式的数据     |
| GZIPInputStream      | 一个InflaterInputStream，用于解压GZIP文件格式的数据    |

### 用GZIP进行简单的压缩

压缩类的使用非常直观，直接将输出流封装成GZIPOutputStream或ZipOutputStream，并将输入流封装成GZIPInputStream或ZipInputStream即可。

### 用Zip进行多文件保存

### Java档案文件

Zip格式也被应用JAR`Java Archive`Java档案文件格式中。

## 对象序列化

Java的***对象序列化***将那些实现了**Serializable**接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。

序列化机制能够自动弥补不同操作系统之间的差异。

对象的序列化的概念加入到语言中是为了支持两种主要特性。一是Java的远程方法调用，二是Java Bean来说，对象的序列化是必须的。

## XML

对象序列化的一个重要限制是它只是Java的解决方案，只有Java程序才能反序列化这种对象。

一种更具通用性的解决方案是将数据转换为XML格式，这可以使其被各种各样的平台和语言使用。

## Preferences

