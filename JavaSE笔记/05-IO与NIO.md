# I/O流

和**Stream流**都是**流**，它的**侧重点在于I/O(输入输出)，它侧重于数据的存储、传输及一些相关操作，它是技术层面的**
- **Stream流**的**侧重点在于计算，是为了从数据中提取新的信息或价值，侧重点是逻辑层面的**



## 文件的分类

- 从计算机最底层的视角看，**所有文件本质上都是二进制文件**，因为它们都由一串0和1的字节序列组成。

  然而，从我们**编程和使用的逻辑视角**看，我们必须根据这串字节的“**用途和编码方式**”将文件划分为两大类：**文本文件和二进制文件**

### 文本文件

#### 相关概念与知识

- 一种由**字符编码**构成的**二进制文件**，其**内容可以直接被人类阅读和理解**。

- **最终读者是『人类』。**

- 它**内部的字节序列**遵循一个**公开的、通用的“翻译协议”——字符编码** (如UTF-8, GBK)

  **任何具备文本处理能力的应用程序，如文本编辑器（Notepad++, VS Code）、集成开发环境（IDE, 如IntelliJ IDEA）、网页浏览器等，都内置了字符解码功能。它们能够遵循指定的字符编码规范（即“协议”），扮演“翻译官”的角色，将底层的字节序列正确地解码为人类可读的字符流。**

- **核心是信息的可读性与正确传达。** **如果翻译错了，就是传说中的“乱码”**。



#### 文本文件举例

- **源代码文件**

  - **`.java`**: Java语言的源代码文件。

  - **`.py`**: Python语言的源代码文件。

  - **`.c`, `.h`, `.cpp`**: C/C++语言的源代码和头文件。

  - **`.js`**: JavaScript脚本文件。

  - **`.go`**: Go语言的源代码文件。

- **网页与样式文件**

  - **`.html` / `.htm`**: 超文本标记语言文件，构成网页的骨架。

  - **`.css`**: 层叠样式表文件，定义网页的样式（颜色、字体、布局等）。

  - **`.svg`**: 可缩放矢量图形文件。虽然是图形，但其内容是用XML（一种文本标记语言）来描述图形的，所以它是一个文本文件。

- **数据交换与配置文件**

  - **`.json`**: 一种轻量级的数据交换格式，广泛用于Web API。

  - **`.xml`**: 可扩展标记语言，常用于配置文件和数据传输。

  - **`.yml` / `.yaml` **: 一种可读性极高的数据序列化格式，常用于配置文件。

  - **`.csv`**: 用逗号分隔值的表格数据文件，可以用Excel打开，但其本质是纯文本。

  - **`.ini`**: Windows早期常用的配置文件格式。

  - **`.properties`**: Java中常用的键值对配置文件。

- **通用文本文档与标记语言**

  - **`.txt`**: 最基本、最纯粹的文本文档。

  - **`.md` **: 一种轻量级标记语言，用于编写带有基本格式的文档。

  - **`.log`**: 程序运行时产生的日志文件，用于记录事件和错误信息。

  - **`.sql`**: 结构化查询语言脚本，用于操作数据库。





### 二进制文件

#### 相关概念与知识

- 是一个广义的概念，指**除文本文件**外，**所有遵循特定程序格式、内容不直接对应人类可读字符的文件**

- **最终读者是『特定程序』**

- 它**内部的字节序列**遵循一个**私有的、专有的“数据格式规范”** (如JPEG, MP4, PDF规范)

  只有**理解这个特定规范的程序（如图片查看器、视频播放器、PDF阅读器）**才能扮演**“解析器”**的角色，将字节流正确地“解析”成图像、声音或格式化文档。

- **核心是数据的结构完整与精确无误**。**如果结构错了，就是传说中的“数据损坏**”。



#### 二进制文件举例

- **图像文件 (Image Files)**

  - **`.jpg` / `.jpeg`**: 一种有损压缩的图片格式，常用于照片。

  - **`.png`**: 一种无损压缩的图片格式，支持透明背景。

  - **`.gif`**: 支持动画和简单透明的图片格式。

  - **`.bmp`**: Windows位图文件，通常未经压缩，文件较大。

  - **`.psd`**: Adobe Photoshop的工程文件，包含图层等复杂信息。

- **音频文件**

  - **`.mp3`**: 目前最流行的有损压缩音频格式。

  - **`.wav`**: 无损的音频格式，音质好但文件大。

  - **`.aac`**: 一种比MP3更先进的音频压缩格式。

  - **`.flac`**: 另一种流行的无损音频压缩格式。

- **视频文件**

  - **`.mp4`**: 目前最流行的视频容器格式，包含视频、音频和字幕等。

  - **`.avi`**: 早期Windows的视频格式。

  - **`.mkv` (Matroska Video)**: 一种开放标准的视频容器格式，功能强大。

  - **`.mov`**: 苹果QuickTime的视频格式。

- **文档与办公文件**

  - **`.pdf` (Portable Document Format)**: Adobe开发的便携式文档格式，能保持排版不变。

  - **`.doc` / `.docx`**: Microsoft Word的文档格式。

  - **`.xls` / `.xlsx`**: Microsoft Excel的电子表格格式。

  - **`.ppt` / `.pptx`**: Microsoft PowerPoint的演示文稿格式。

- **程序与库文件 (Programs & Libraries)**

  - **`.exe`**: Windows平台的可执行文件。

  - **`.dll` (Dynamic Link Library)**: Windows平台的动态链接库。

  - **`.class`**: Java源代码编译后生成的**字节码文件**，由JVM执行。

  - **`.jar`**: Java Archive，将多个.class文件和资源打包成的文件。

  - **`.so`**: Linux平台的共享库文件。

  - **`.a`**: 静态库文件。

- **压缩与归档文件 (Compressed & Archive Files)**

  - **`.zip`**: 一种非常流行的归档文件格式，支持多种压缩算法，被几乎所有操作系统原生支持。

  - **`.rar`**: 一种私有的归档文件格式，以其较高的压缩率而闻名，需要专门的软件（如WinRAR）来创建和完全解压。

  - **`.7z`**: 一种开放标准的归档文件格式，通常使用LZMA算法，以提供极高的压缩率而著称。

  - **`.tar` (Tape Archive)**: 这是一种**归档**格式，它只负责将多个文件和目录“打包”成一个文件，但**本身并不压缩**。

  - **`.gz` (GNU Zip)**: 这是一种**压缩**格式，通常用于压缩**单个文件**。在Linux/Unix世界中，非常常见地将两者结合使用，先用 tar 打包，再用 gz 压缩，形成 .tar.gz 或 .tgz 文件。

- **数据库文件 (Database Files)**

  - **`.db`**: 一个通用的数据库文件扩展名，常被多种数据库软件用作默认后缀。

  - **`.sqlite`**: SQLite数据库的文件格式。SQLite是一种轻量级的、嵌入式的、无服务器的数据库，非常适合移动应用和小型项目。

  - **`.mdb` / `.accdb`**: Microsoft Access数据库的文件格式。.mdb是早期版本（2003及以前）的格式，.accdb是较新版本（2007及以后）的格式。





## 编码和解码

### 字节流和字符流的诞生

​	在**Java**诞生之初，计算机世界的主流思想深受**C语言**的影响。在**C语言中**，**文件操作的基本单位就是字节**(**byte**)（**C**的基本类型没有**`byte`类型**，**C**中**大小为1字节**的基本类型是**`char`类型**，**Java**中**大小为1字节**的基本类型为**`byte`类型**，**Java**中的**`char`**是**2字节**）。**网络传输（如TCP/IP协议）**的单位也是**字节**。所以，Java的创始者们自然而然地设计了一套**基于字节的I/O模型**，**这套模型能够处理计算机世界里的一切数据**——从简单的文本到复杂的图像、声音。

​	所以，**字节流**在**Java的第一个正式版（JDK 1.0）**中就闪亮登场了。

​	很快，随着**Java**开始在**全球范围**内流行，一个**巨大的问题**浮现出来：**国际化**。早期的计算机系统大多基于**ASCII码**，用**一个字节（准确说是7位）就能表示所有英文字母、数字和符号**。用字节流处理起来毫无问题。 但**Java**的目标是 **"Write Once, Run Anywhere"（一次编写，到处运行）**，这个**“Anywhere”**也包括了**非英语国家**，**中文、日文、韩文、阿拉伯文等语言的字符数量远超ASCII的容量**，**必须使用多个字节来表示一个字符**（比如GBK, Big5, Shift_JIS, UTF-8等编码），程序员每次读写**文本**，都必须**手动编码/解码**，**又麻烦又容易出错**，**非常恶心**，只要**读和写用的编码不一致**，或者在**处理多字节字符时发生截断**，**乱码问题**就会像**幽灵**一样缠着开发者，并且IO代码里混杂了大量的**字节数组操作和编码转换逻辑**，**核心的业务逻辑被淹没**了。

​	开发者社区的呼声越来越高：**“我们有时候只是想简单地读写几行日志，或者解析一个配置文件，又不复杂，为什么操作起来这么痛苦？啊啊啊啊啊啊啊啊！！！”**

​	为了解决这个日益严重的国际化文本处理问题，Sun公司的工程师们在**JDK 1.1版本**中，进行了一次意义重大的**升级**，引入了全新的IO分支——**字符流**。这次更新**将复杂的、易错的“字节-字符”转换逻辑封装起来**，并**对程序员隐藏了这些底层细节**，让程序员可以**专注于“字符”和“字符串”**这个更高层次的逻辑概念，而不是底层的**“字节”**，并且提供了一些更**便利的API**，比如直接写入一个字符串(**`writer.write(String)`**)，或者直接读取一行 (**`reader.readLine()`**)等，**操作不再容易出错，变得非常方便和好用。**

​	**字符流诞生了**，**`Reader`** 和 **`Writer`** 作为**字符流的基类**诞生了。



### 字符集和字符编码

#### Unicode字符集(抽象集合) + UTF-8字符编码(规则)

##### 解析规则

- 可以把字节分成两类：**“领头字节 (Leading Byte)”** 和 **“后续字节 (Continuation Byte)”**

  - **对于单字节字符（比如英文字母、数字）**

    - **规则**：这个字节的**第一位**必须是 **0**。

    - **格式**：0xxxxxxx

      > 当计算机读到一个字节，发现它以 0 开头，就立刻知道：“这是一个单字节字符，它自己就代表一个完整的字符，读完它就结束了。” 这也完美兼容了老的 ASCII 编码。




  - **对于多字节字符（比如中文、日文、特殊符号）：**
    - 规则非常精妙，全看**领头字节**的开头有几个 1。
    - **领头字节**：
      - 如果一个字符占 **2个字节**，那么它的领头字节就以 **110** 开头。 (110xxxxx)
      - 如果一个字符占 **3个字节**（中文字符常见于此），领头字节就以 **1110** 开头。 (1110xxxx)
      - 如果一个字符占 **4个字节**，领头字节就以 **11110** 开头。 (11110xxx)
      - 规律：**领头字节里 1 的数量，代表了这个字符总共占多少字节。**

    - **后续字节**：
      - **规则**：所有不属于领头字节的“后续字节”，**都必须以 10 开头**。 (10xxxxxx)




### 乱码

- 乱码的本质就一句话：编码和解码时，使用了两套不同的字符编码方案
  - 比如一个文件，**用UTF-8保存，却用GBK打开**

示例代码

```java
public static void main(String[] args) throws UnsupportedEncodingException {
    String str = "Hello 世界";
    //1.编码
    byte[] bytes1 = str.getBytes(StandardCharsets.UTF_8);
    byte[] bytes2 = str.getBytes("GBK");
    System.out.println(Arrays.toString(bytes1));
    			//[72, 101, 108, 108, 111, 32, -28, -72, -106, -25, -107, -116]
    System.out.println(Arrays.toString(bytes2));
    			//[72, 101, 108, 108, 111, 32, -54, -64, -67, -25]

    //2.解码
    String str1 = new String(bytes1,StandardCharsets.UTF_8);
    String str2 = new String(bytes1,"GBK");
    System.out.println(str1);								//Hello 世界
    System.out.println(str2);								//Hello 涓栫晫

    String str3 = new String(bytes2,StandardCharsets.UTF_8);
    String str4 = new String(bytes2,"GBK");
    System.out.println(str3);								//Hello ����
    System.out.println(str4);								//Hello 世界
}
```








## I/O流体系图(部分)

![image-20250716211158143](./assets/image-20250716211158143.png)



## I/O流相关类

- 关流遵循”后开先关“原则
- 把包装流关了，里面的流也就关了

### 字节流

- 处理二进制文件

- 以字节（8位）为单位来处理数据
- **字符流可以指定字符编码**，或在构造方法中，或在成员方法中，但是**字节流不能指定，它也无需指定**

#### 字节输入流

##### `InputStream`类

- 抽象类



###### 构造方法

```java
public InputStream() 			// 唯一的构造函数，供子类调用，通常不直接使用。
```

###### 常用方法

```java
public abstract int read()					// 从输入流读取下一个数据字节。这是所有子类必须实现的核心抽象方法。
public int 			read(byte[] b) 			// 从输入流读取一些字节数，并将它们存储到缓冲区数组 b 中。
											//返回读入缓冲区的总字节数；如果因为已到达流末尾而不再有数据，则返回 -1。
public int 			read(byte[] b, int off, int len)	// 从输入流将最多 len 个数据字节读入字节数组。
public byte[] 		readAllBytes() 			// 从此输入流中读取所有剩余字节。(自JDK 9添加)
public byte[] 		readNBytes(int len) 	// 从输入流中读取所请求数量的字节到字节数组中。(自JDK 9添加)
public int 			readNBytes(byte[] b, int off, int len)// 从输入流将所请求数量的字节读入给定的字节数组(JDK 9添加)
    
public void 		close() 	// 关闭此输入流并释放与该流关联的所有系统资源(极其重要，推荐使用try-with-resources)
```

```java
public long 	skip(long n) 		// 跳过和丢弃此输入流中的 n 个数据字节。
public void 	skipNBytes(long n) throws IOException // 跳过并丢弃来自该输入流的 n 个字节的数据
public int 		available() 		// 返回可以不受阻塞地从此输入流读取（或跳过）的字节数的估计值。
public void 	mark(int readlimit) // 在此输入流中标记当前的位置。
public void 	reset() 			// 将此流重新定位到最后一次对此输入流调用 mark 方法时的位置。
public boolean 	markSupported() 	// 测试此输入流是否支持 mark 和 reset 方法。
public long 	transferTo(OutputStream out)
    								// 从此输入流读取所有字节，并按读取的顺序将字节写入给定的输出流。(自JDK 9添加)
    								// 这是实现文件复制等操作的极高效方法
```

```java
public static InputStream nullInputStream()
//返回一个不读取任何字节的InputStream(自JDK 11添加).在到达流末尾时，其read()方法始终返回-1
```



###### 一些方法的用法和细节

- 子类体系中相同方法返回值一样

- **`read()`方法**

  - 字节流中的这个方法，是在物理层面上一个字节一个字节的读取，返回值的范围是 **0 到 255**，代表读取到的那个字节的无符号值
  - 读了之后，下一次使用**`read()`方法**读的时候，会**到下一个字节**，读到流的末尾，会**返回 -1**，内部有指针的

- **`read(byte[] b)`方法**

  - 想要读取速度快，用**`read(byte[] b)`**而不是**`read()`**,**`read(byte[] b)`**的底层做了很大优化，是**一块一块读数据**

    - 传过去一个**“空蓝子”**，底层在读取的时候会尝试用传递过去的 **数组** 装数据，返回值是**这次往数组中装的数据个数(即读到的数据数)**，但是**如果没有读到数据，返回值是-1**，然后每调用一次方法，就把数据往这个数组中进行**尽可能多的装填**，**`byte`数组** 中的数据在每一次被装填的时候都是 **从数组头开始覆盖装填**，纯手写比缓冲流性能还高一点

      > 为啥没读到数据返回的是 -1 ，而不是返回 0？
      >
      > - 一个我认为的原因是为了一致性，即只要是**`read`方法**，读不到数据就返回-1
      >
      > - 还有一个原因是**0**这个返回值主要是为**非阻塞式IO（Non-Blocking IO）**准备的，尤其是在**网络编程**中：
      >   **想象一个网络服务器的场景：**
      >   服务器正在监听一个网络连接（Socket），
      >   它调用 **`socketInputStream.read(buffer)`**，想看看客户端有没有发数据过来。
      >   在这一瞬间，客户端恰好什么都没发。
      >   如果这时方法返回-1，服务器就会以为客户端断开了连接，从而关闭这个socket。这显然是错误的！
      >   所以，这时方法应该返回 **0**！这个0的含义是：“**我现在没收到数据，但这不代表连接断了，你过一会儿再来问问看。**”
      >   服务器看到**返回0**，就知道连接还是好的，于是它可以先去处理其他客户端的请求，稍后再回来检查这个连接。
      > - ......

    - 像缓冲流这种已经加快速度的流中，如果还使用这种方法来读取，会更快



##### `FileInputStream`文件输入流

###### 相关概念和知识

- 文件流

- 用于从文件系统的文件中获取输入字节。它通常用于读取诸如图像数据之类的原始字节流

- 如果文件**不存在**，它会立即抛出 **`FileNotFoundException` 异常**

- **文件指针**

  - 当一个 **`FileInputStream` 对象**被创建时，Java虚拟机会向操作系统请求**打开指定的文件**。操作系统响应此请求，并返回一个唯一的标识——在Unix/Linux中称为**文件描述符 (File Descriptor)**，在Windows中称为**文件句柄 (File Handle)**。这个句柄是程序与文件之间建立连接的凭证。

    这个由操作系统维护的文件句柄是“有状态”的，其核心是内置了一个**文件指针**。当文件被打开时，这个**指针自动被设置为 0**，即指向文件的起始字节。

    每当调用 read() 方法时，操作系统会从文件指针的当前位置开始读取数据。关键在于，**读取操作完成后，操作系统会自动将该指针向后移动相应的字节数**。例如，读取了8个字节，指针就自动前进8位。

    因此，**`FileInputStream`** 本身并不负责追踪或管理读取的位置。它更像一个忠实的“代理”，将Java层面的读取请求直接转发给操作系统，并利用操作系统提供的文件指针自动前进的能力，从而实现了天然的、顺序向前的读取流。

###### 构造方法

```java
public FileInputStream(String name) 
    				// 通过打开一个到实际文件的连接来创建一个 FileInputStream，该文件通过文件系统中的路径名 name 指定。
public FileInputStream(File file) 
    				//通过打开一个到实际文件的连接来创建一个 FileInputStream,该文件通过文件系统中的File对象file指定。
public FileInputStream(FileDescriptor fdObj) 		// 使用文件描述符 fdObj 创建一个 FileInputStream。
```

###### 常见方法

```java
public int 		read() 								// 从此输入流中读取一个数据字节。(调用本地native方法实现)
public int 		read(byte[] b) 						// 从此输入流中将 b.length 个字节的数据读入一个 byte 数组中。
public int 		read(byte[] b, int off, int len)	// 从此输入流中将 len 个字节的数据读入一个 byte 数组中。

public long 	skip(long n) 					// 从输入流中跳过并丢弃 n 个字节的数据。
public int 		available() 					// 返回下一次对此输入流调用方法可以不受阻塞地从此输入流读取的字节数。
public void 	close() 						// 关闭此文件输入流并释放与此流有关的所有系统资源。

public final FileDescriptor getFD() 			// 返回表示到此文件输入流所使用的实际文件连接的文件描述符。
public FileChannel 			getChannel() 		// 返回与此文件输入流有关的唯一 FileChannel 对象。(通往NIO的桥梁)
```



###### 一些方法的用法和细节

- **`read()`**

  - 示例代码

    ```java
    public static void main(String[] args) throws IOException {
        //创建对象
        FileInputStream fis = new FileInputStream("a.txt");
        //循环读取
        int b;
        while((b=fis.read())!=-1){
            System.out.println((char)b);
        }
        fis.close();
    }
    ```

- **`read(byte[] b)`**

  - 示例代码

    ```java
    byte[] buffer = new byte[1024 * 8]; // 8KB buffer
    int bytesRead;
    while ((bytesRead = fis.read(buffer)) != -1) {
        System.out.print(new String(buffer, 0, bytesRead));
    }
    ```

    


##### `ObjectInputStream`反序列化流

- **反序列化 (Deserialization)**：将文件或网络中的字节序列，重新恢复成内存中一模一样的Java对象
- 从底层的字节输入流中读取字节序列，将其反序列化，还原成Java对象
- 序列化的类必须实现**`java.io.Serializable` 接口**
- 序列化和反序列化对象的时候，建议用集合

###### 构造方法

```java
public 	  ObjectInputStream(InputStream in) 	// 创建从指定 InputStream 读取的 ObjectInputStream。
												// 会调用 readStreamHeader() 读取并验证流的头部信息。
protected ObjectInputStream() 	// 为完全重写反序列化的子类提供一种方法，使其不必为 ObjectInputStream 分配私有数据
```

###### 常见方法

```java
public final Object readObject() 
   					 //(最核心方法)从 ObjectInputStream 读取一个对象.对象的类、签名、及其非瞬态和非静态字段的值都会被恢复
    				//读不到会抛出异常
public Object readUnshared() 		// 从 ObjectInputStream 读取一个“非共享”对象。
   							   //此方法与readObject()类似,但它防止后续对同一对象的readObject()调用返回对同一对象的引用
public void defaultReadObject() 
    								// 从此流中读取当前类的非静态和非瞬态字段。只能在反序列化类的 readObject 方法中调用
public ObjectInputStream.GetField readFields() 
    									// 以名-值对的形式从流中读取持久字段。返回一个 GetField 对象，用于访问这些字段
```

```java
public int 		read() 										// 读取数据的字节。
public int 		read(byte[] buf, int off, int len)			// 读入一个字节数组。
public boolean 	readBoolean() 								// 读取一个 boolean 值。
public byte 	readByte() 									// 读取一个 8 位字节。
public int 		readUnsignedByte() 							// 读取一个无符号 8 位字节。
public char 	readChar() 									// 读取一个 16 位 char。
public short 	readShort() 								// 读取一个 16 位 short。
public int 		readUnsignedShort() 						// 读取一个无符号 16 位 short。
public int 		readInt() 									// 读取一个 32 位 int。
public long 	readLong() 									// 读取一个 64 位 long。
public float 	readFloat() 								// 读取一个 32 位 float。
public double 	readDouble() 								// 读取一个 64 位 double。
public String 	readLine() 									// (已弃用) 读取一行以换行符结尾的文本。不推荐使用。
public String 	readUTF() 									// 读取以 modified-UTF-8 格式编码的字符串。
public void 	readFully(byte[] buf) 						//读取字节，阻塞直到读取所有字节
public void 	readFully(byte[] buf, int off, int len) 	//读取字节，阻塞直到读取所有字节
```

```java
public int 		available() 					// 返回可以不受阻塞地读取的字节数。
public void 	close() 						// 关闭输入流。
public void 	registerValidation(ObjectInputValidation obj, int prio)
												// 注册一个在返回对象图前要验证的对象。这对于确保对象的内部一致性很有用
```

```java
protected Object   resolveObject(Object obj) 		//此方法允许 ObjectInputStream 的子类监视或替换从流中读取的对象
protected Class<?> resolveClass(ObjectStreamClass desc)		// 根据指定的类描述符加载本地类。
protected Class<?> resolveProxyClass(String[] interfaces)
   											// 返回一个与代理类描述符中指定的接口相对应的类，用于反序列化动态代理
protected void readStreamHeader()	 		// 读取并验证流魔法数和版本号。如果流无效,则抛出StreamCorruptedException
protected Object  readObjectOverride()
protected ObjectStreamClass readClassDescriptor()			// 从序列化流读取类描述符。
protected boolean enableResolveObject(boolean enable)
									// 允许此流的子类调用 resolveObject。必须传入 true 才能使 resolveObject 生效
```

```java
public final ObjectInputFilter  getObjectInputFilter()				//返回此流的反序列化过滤器
public final void setObjectInputFilter(ObjectInputFilter  filter)	//为流设置反序列化过滤器
public int skipBytes(int len)										//跳过字节
```



###### `Serializable`接口

- 这是一个标记型接口，用来标记一个类的对象是“可以被**序列化和反序列化**的”

- 关于**`serialVersionUID`**

  - **声明：`private static final long serialVersionUID`**

  - **作用：**版本控制、防止数据损坏和程序崩溃、将兼容性的决定权交给程序员等......****

  - 它的唯一目的，就是为这个类提供一个**版本控制的标识符**。

    - **序列化时**: 这个标识符会和对象的具体数据一起被写入字节流

    - **反序列化时**: **`ObjectInputStream`** 会先从流中读取这个 UID，然后与当前JVM中加载的对应类的 UID 进行比较。

      - **如果一致**：则认为版本兼容，继续进行反序列化。即使类增加或减少了字段，它也会尝试兼容处理（新增的字段为null，删除的字段被忽略）

      - **如果不一致**：则认为版本不兼容，立即抛出 **`InvalidClassException`**

        - 如果不想出现不一致的情况，就需要自己指定**`serialVersionUID`**的值,

          - 但是自己指定**必须严格按照** **`private static final long serialVersionUID`** **的格式来指定**,

          - 如果没有按照这个格式声明，那么声明的那个变量对于**Java序列化机制**来说，就**彻底失去了它的特殊意义**，它会将其**视作一个普普通通的成员变量**

- 关于**`transient`**

  - **`transient`** 是一个Java关键字，只能用来修饰类的**成员变量（实例变量）**.  它的唯一作用是：**向Java的默认序列化机制发出一个信号，告诉它在序列化这个对象时，请完全忽略被 transient 修饰的这个字段**

  - 序列化和反序列化时
    - **在序列化时** ：如果一个变量被**`transient`** 修饰，**`ObjectOutputStream`** 会**完全跳过**这个字段，不会为它写入任何数据到字节流中。
    - **在反序列化时**：当 **`ObjectInputStream`** 从字节流中读取数据来“复活”一个对象时，它当然找不到任何关于 **`transient`** 字段的数据。因此，**`ObjectInputStream`** 会将这个字段的值初始化为其**类型的默认值**。这是**极其重要**的一点



##### `ByteArrayInputStream`

- 让我们能像操作文件一样，方便地从内存中的一个字节数组里读取数据

###### 字段

```java
protected byte[] 	buf 		// 流所包含的字节数组缓冲区。
protected int 		pos 		// 要从缓冲区中读取的下一个字符的索引。
protected int 		mark 		// 流中当前标记的位置，默认为0。
protected int 		count 		// 缓冲区中最后一个有效字符的索引+1。其值应始终为非负数。
```



###### 构造方法

```java
public ByteArrayInputStream(byte[] buf) 			// 创建一个 ByteArrayInputStream，使用 buf 作为其缓冲区数组
public ByteArrayInputStream(byte[] buf, int offset, int length)
							//创建ByteArrayInputStream，使用 buf 作为其缓冲区数组，并从指定的偏移量开始，读取指定的长度
```

###### 常见方法

```java
public int 		read() 											// 从此输入流中读取下一个数据字节。
public int 		read(byte[] b, int off, int len)				// 将最多 len 个数据字节从此输入流读入 byte 数组。
public byte[] 	readAllBytes() 									// 从此输入流中读取所有剩余字节
public int 		readNBytes(byte[] b, int off, int len)			// 从输入流将所请求数量的字节读入给定的字节数组
```

```java
public long 	skip(long n) 									// 从此输入流中跳过 n 个输入字节。
public int 		available() 									// 返回可从此输入流读取的剩余字节数。
public boolean 	markSupported() 
    							//测试此InputStream是否支持mark/reset。ByteArrayInputStream支持此功能，返回true
public void 	mark(int readAheadLimit) 						// 在流中设置当前标记位置。
public void 	reset() 										// 将缓冲区的位置重置为标记位置。
public void 	close() 		  //关闭ByteArrayInputStream无效。此方法在关闭该流后仍可被调用，而不会生成IOException
```

```java
public long 	transferTo(OutputStream out)			//从此输入流读取所有字节，并按读取的顺序将字节写入给定的输出流
```



##### `BufferedInputStream`缓冲输入流

- 缓冲流
- **`FilterInputStream`**的子类
- 通过在内存中引入缓冲区（Buffer）来减少实际的 I/O 操作次数，从而提高数据传输的效率
- 默认缓冲区大小在大多数Java版本（包括JDK 8, 11, 17等）中是**8192**
- 关流的时候会把传入的流也关闭
- 底层加速的原理是它底层自动灵活的调用了底层的 **`read(byte[] b)`** 方法
  - 所以我记得在某些情况下，基础读取流会比缓冲流更快




###### 字段

```java
protected byte[] 	buf 		// 存储数据的内部缓冲区数组。
protected int 		count 		// 缓冲区中有效字节的数量。
protected int		pos 		// 缓冲区中下一个要读取的字节的索引。
protected int 		markpos 	// 最后一次调用 mark() 方法时的 pos 值。
protected int 		marklimit 	// 调用 mark() 后，在标记位置失效前允许读取的最大字节数
```

###### 构造方法

```java
public BufferedInputStream(InputStream in) // 创建一个 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用
public BufferedInputStream(InputStream in, int size) // 创建具有指定缓冲区大小的 BufferedInputStream
```

###### 常用方法

```java
public int 		read() 			// 从此输入流中读取下一个数据字节。(从内部缓冲区读取)
public int 		read(byte[] b, int off, int len)	// 将字节从此字节输入流读入指定的 byte 子数组中。

public long 	skip(long n) 	// 跳过和丢弃此输入流中的 n 个数据字节。
public int 		available() 	// 返回可以不受阻塞地从此输入流读取的字节数的估计值。
public void 	close() 		// 关闭此输入流并释放与该流关联的所有系统资源。
public void 	mark(int readlimit) 				// 在此输入流中标记当前的位置。
public void 	reset() 		// 将此流重新定位到最后一次对此输入流调用 mark 方法时的位置。
public boolean 	markSupported() // 测试此输入流是否支持 mark 和 reset 方法。缓冲流支持此功能。
```



##### `DataInputStream`数据输入流

- **它是`FilterInputStream`**的子类
- 允许应用程序以与机器无关的方式从底层输入流中读取基本的Java数据类型

###### 构造方法

```java
public DataInputStream(InputStream in) // 创建一个使用指定的基础 InputStream 的 DataInputStream
```

###### 常见方法

```java
public final boolean 	readBoolean() 		// 从此数据输入流中读取一个 boolean 值。
public final byte 		readByte() 			// 从此数据输入流中读取一个有符号的 8 位值。
public final int 		readUnsignedByte() 	// 从此数据输入流中读取一个无符号的 8 位值。
public final short 		readShort() 		// 从此数据输入流中读取一个有符号的 16 位值。
public final int 		readUnsignedShort() // 从此数据输入流中读取一个无符号的 16 位值。
public final char 		readChar() 			// 从此数据输入流中读取一个 char 值。
public final int 		readInt() 			// 从此数据输入流中读取一个有符号的 32 位整数。
public final long 		readLong() 			// 从此数据输入流中读取一个有符号的 64 位整数。
public final float 		readFloat() 		// 从此数据输入流中读取一个 32 位浮点数。
public final double 	readDouble() 		// 从此数据输入流中读取一个 64 位浮点数。
public final String 	readLine() 			// (已弃用) 读取此数据输入流中的下一行文本
public final String 	readUTF() 			// 读取一个以 modified-UTF-8 格式编码的字符串。
```

```java
public int 			read(byte[] b) 						 //从包含的输入流中读取一些字节数,将它们存储到缓冲区数组b中
public int 			read(byte[] b, int off, int len)	 //从包含的输入流中将 len 个字节读入一个 byte 数组中
public final void 	readFully(byte[] b) 				 //从包含的输入流中读取b.length个字节，并存储到缓冲区数组b中
public final void 	readFully(byte[] b, int off, int len)//从包含的输入流中准确地读取len个字节,并存储到缓冲区数组b中
```

```java
public static final String readUTF(DataInput in)
				// 从流 in 中读取用 modified-UTF-8 格式编码的 Unicode 字符串的表示形式；然后以 String 形式返回此字符串
public final int skipBytes(int n)
```



#### 字节输出流

##### `OutputStream`类

- 是抽象类



###### 构造方法

```java
public OutputStream() // 唯一的构造函数，供子类调用，通常不直接使用
```

###### 常见方法

```java
public abstract void 	write(int b) 			// 将指定的单个字节写入此输出流。这是所有子类必须实现的核心抽象方法。
public void 			write(byte[] b) 				// 将指定字节数组中的所有字节写入此输出流。
public void 			write(byte[] b, int off, int len)//将指定字节数组中从偏移量off开始的len个字节写入此输出流
public void 			flush() 						// 刷新此输出流并强制写出所有缓冲的输出字节
public void 			close() 	//关闭此输出流并释放与该流关联的所有系统资源(极其重要，推荐使用try-with-resources)
```

```java
public static OutputStream nullOutputStream() 	//返回一个新的OutputStream，它会丢弃所有被写入的数据(自JDK 11添加)
    											// 在测试或需要“空操作”输出流时非常有用
```



###### 一些方法的用法和细节

- 子类体系中相同方法返回值一样



##### `FileOutputStream`文件输出流

###### 相关概念和知识

- 用于将数据写入文件等。文件输出流用于写入诸如图像数据之类的原始字节的流

- **`FileOutputStream`** 的构造方法被设计为在默认情况(没有append指定)下**覆盖**写文件

  > 就是有个开关，开了后就能续写，不开就不能续写，默认是关的

  - **如果文件不存在**：它会尝试创建一个新的空文件，前提是父级路径要存在

  - **如果文件已存在**：它会打开该文件，并将其长度截断为0，以便从头开始写入新数据，这就是为什么在还没调用 **`write()`** 方法写入任何数据之前，文件内容就已经消失的原因

  - **如果父级路径不存在**：抛出异常

    ```java
    new FileOutputStream("d:/some/new/folder/data.txt")
        //如果 d:/some/new/folder 这个目录不存在，程序会直接抛出 FileNotFoundException 异常
    ```



###### 构造方法

```java
public FileOutputStream(String name) 
    					// 创建一个向具有指定名称的文件中写入数据的输出文件流。如果文件存在，则旧文件被覆盖。
public FileOutputStream(String name, boolean append)
                        // 创建一个向具有指定名称的文件中写入数据的输出文件流。如果 append 为 true，则将字节写入文件末尾。
public FileOutputStream(File file) 
    					// 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。如果文件存在，则旧文件被覆盖。
public FileOutputStream(File file, boolean append)
					// 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。如果 append 为 true，则将字节写入文件末尾
public FileOutputStream(FileDescriptor fdObj) 
    				// 创建一个向指定文件描述符处写入数据的输出文件流，该文件描述符表示一个到文件系统中某个实际文件的现有连接
```

###### 常见方法

```java
public void write(int b) 						// 将指定的字节写入此文件输出流。 (调用本地native方法实现)
public void write(byte[] b) 					// 将 b.length 个字节从指定 byte 数组写入此文件输出流中。
public void write(byte[] b, int off, int len)	// 将 len 个字节从指定 byte 数组的偏移量 off 处开始写入此文件输出流中
public void close() 							// 关闭此文件输出流并释放与此流有关的所有系统资源。

public final FileDescriptor getFD() // 返回与此流有关的文件描述符。
public FileChannel getChannel() // 返回与此文件输出流有关的唯一 FileChannel 对象。(通往NIO的桥梁)
```



**一些方法的用法和细节**

- **`write(int b)`** **方法会接收你传入的** **int** **值，但它会忽略掉高位的24个比特位，只将最低的8个比特位（也就是最后一个字节）写入到文件中**

- **`write(byte[] b)`**方法会把数组b中的数据**全部写入**，性能要比**`write(int b)`**高，纯手写比缓冲流性能还高一点



##### `ObjectOutputStream`序列化流

- **序列化 (Serialization)**：将内存中的Java对象，转换成一连串特定格式的**字节序列**，以便于存储或传输
- 将Java对象序列化，变成字节序列，并写入到底层的字节输出流
- 序列化的类必须实现**`java.io.Serializable` 接口**
- 序列化和反序列化对象的时候，建议用集合

###### 构造方法

```java
public ObjectOutputStream(OutputStream out) 
    							// 创建写入指定 OutputStream 的 ObjectOutputStream。会向流中写入序列化流头部信息
protected ObjectOutputStream() 	// 为完全重写序列化的子类提供一种方法，使其不必为 ObjectOutputStream 分配私有数据
```

###### 常见方法

```java
public final void 	writeObject(Object obj) 
    			  //(最核心方法)将指定的对象写入 ObjectOutputStream。对象的类、签名、及其非瞬态和非静态字段的值都会被写入
public void 		writeUnshared(Object obj) 	
 						// 将一个“非共享”对象写入 ObjectOutputStream。
    					//此方法与 writeObject() 类似，但它保证总是将一个新对象写入流中，而不是引用之前已写入的同一对象
public void 		defaultWriteObject() 
    					// 将当前类的非静态和非瞬态字段写入此流。此方法只能在序列化类的 writeObject 方法中调用
public ObjectOutputStream.PutField 	putFields() 
    					// 获取用于缓冲要写入流的持久字段的对象。
    					//此方法返回的 PutField 对象将用于在 writeFields 方法中设置字段值
public void 		writeFields() 				// 将已通过 putFields() 缓冲的持久字段写入流
```

```java
public void 		write(int val) 									// 写入一个字节。
public void 		write(byte[] buf) 								// 写入一个字节数组。
public void 		write(byte[] buf, int off, int len)				// 写入字节的子数组。
public void 		writeBoolean(boolean val) 						// 写入一个 boolean 值。
public void 		writeByte(int val) 								// 写入一个 8 位字节。
public void		 	writeShort(int val) 							// 写入一个 16 位 short。
public void 		writeChar(int val) 								// 写入一个 16 位 char。
public void 		writeInt(int val) 								// 写入一个 32 位 int。
public void 		writeLong(long val) 							// 写入一个 64 位 long。
public void 		writeFloat(float val) 							// 写入一个 32 位 float。
public void 		writeDouble(double val) 						// 写入一个 64 位 double。
public void 		writeBytes(String str) 						//以字节序列形式写入一个String(每个字符只写低8位)
public void 		writeChars(String str) 					  // 以字符序列形式写入一个 String (每个字符写2个字节)
public void 		writeUTF(String str)				      //以modified-UTF-8 格式的机器无关方式写入一个String
```

```java
public void 		flush() 	// 刷新流。此操作将所有缓冲的输出字节写入到底层流中。
public void 		reset() 	// 重置将丢弃已写入流的对象的状态。状态被重置为与新的 ObjectOutputStream 相同。
public void 		close() 	// 关闭流。此操作必须在刷新流之后执行。
```

```java
protected Object 	replaceObject(Object obj) 		// 此方法允许 ObjectOutputStream 的子类监视或替换要序列化的对象
protected void 		annotateClass(Class<?> cl) 		// 子类可实现此方法，以便将类数据存储在流中
protected void 		annotateProxyClass(Class<?> cl)	// 子类可实现此方法，以便将代理类数据存储在流中，用于序列化动态代理
protected void 		writeStreamHeader() 			// 写入流魔法数和版本号。如果子类需要追加自己的头部，可以重写此方法
protected void 		writeClassDescriptor(ObjectStreamClass desc)
													// 将指定的类描述符写入 ObjectOutputStream
protected boolean 	enableReplaceObject(boolean enable)
											//允许此流的子类调用replaceObject.必须传入true才能使replaceObject生效
protected void 		writeObjectOverride(Object obj)
											//子类用于重写默认writeObject行为的方法
```

```JAVA
protected void 		drain()								//清空 ObjectOutputStream 中的所有缓冲数据
public void 		useProtocolVersion(int version)		//指定写入流时要使用的流协议版本
```



###### `Serializable`接口

- 这是一个标记型接口，用来标记一个类的对象是“可以被**序列化和反序列化**的”

- 关于**`serialVersionUID`**

  - **声明：`private static final long serialVersionUID`**

  - **作用：**版本控制、防止数据损坏和程序崩溃、将兼容性的决定权交给程序员等......****

  - 它的唯一目的，就是为这个类提供一个**版本控制的标识符**。

    - **序列化时**: 这个标识符会和对象的具体数据一起被写入字节流

    - **反序列化时**: **`ObjectInputStream`** 会先从流中读取这个 UID，然后与当前JVM中加载的对应类的 UID 进行比较。

      - **如果一致**：则认为版本兼容，继续进行反序列化。即使类增加或减少了字段，它也会尝试兼容处理（新增的字段为null，删除的字段被忽略）

      - **如果不一致**：则认为版本不兼容，立即抛出 **`InvalidClassException`**

        - 如果不想出现不一致的情况，就需要自己指定**`serialVersionUID`**的值,

          - 但是自己指定**必须严格按照** **`private static final long serialVersionUID`** **的格式来指定**,

          - 如果没有按照这个格式声明，那么声明的那个变量对于**Java序列化机制**来说，就**彻底失去了它的特殊意义**，它会将其**视作一个普普通通的成员变量**

- 关于**`transient`**
  - **`transient`** 是一个Java关键字，只能用来修饰类的**成员变量（实例变量）**.  它的唯一作用是：**向Java的默认序列化机制发出一个信号，告诉它在序列化这个对象时，请完全忽略被 transient 修饰的这个字段**
  - 序列化和反序列化时
    - **在序列化时** ：如果一个变量被**`transient`** 修饰，**`ObjectOutputStream`** 会**完全跳过**这个字段，不会为它写入任何数据到字节流中。
    - **在反序列化时**：当 **`ObjectInputStream`** 从字节流中读取数据来“复活”一个对象时，它当然找不到任何关于 **`transient`** 字段的数据。因此，**`ObjectInputStream`** 会将这个字段的值初始化为其**类型的默认值**。这是**极其重要**的一点



##### `ByteArrayOutputStream`

- 许你将数据写入到内存中的一个字节数组里，而不是直接写入文件

###### 字段

```java
protected byte[] buf // 存储数据的缓冲区。
protected int count // 缓冲区中的有效字节数。
```



###### 构造方法

```java
public ByteArrayOutputStream() // 创建一个新的 ByteArrayOutputStream，初始容量为 32 字节。
public ByteArrayOutputStream(int size) // 创建一个新的 ByteArrayOutputStream，并指定初始缓冲区容量（以字节为单位）。
```

###### 常见方法

```java
public void 		write(int b) 					 // 将指定的字节写入此字节数组输出流。
public void 		write(byte[] b, int off, int len)//将指定字节数组中从偏移量off开始的len个字节写入此字节数组输出流
public void 		writeBytes(byte[] b) 			 //(自JDK 11添加)将指定字节数组的所有字节写入此输出流
```

```java
public byte[] 		toByteArray() 	//创建一个新分配的字节数组。其大小是此输出流的当前大小，内容是当前输出的有效副本
public String 		toString() 		//使用平台的默认字符集，通过解码字节将缓冲区内容转换为字符串
public String 		toString(String charsetName)//使用指定的字符集名称，通过解码字节将缓冲区内容转换为字符串
public String 		toString(Charset charset) 	//(自JDK 10添加)使用指定的字符集，通过解码字节将缓冲区内容转换为字符串
```

```java
public int 		size() 					// 返回缓冲区的当前大小。
public void 	reset()		 			// 将此字节数组输出流的 count 字段重置为零，从而丢弃输出流中目前已累积的所有输出
public void 	writeTo(OutputStream out) // 将此字节数组输出流的全部内容写入到指定的输出流参数中。
public void 	close() 	     //关闭ByteArrayOutputStream无效。此方法在关闭该流后仍可被调用，而不会生成IOException
```



##### `BufferedOutputStream`缓冲输出流

- **`FilterOutputStream`**的子类
- 通过在内存中引入缓冲区（Buffer）来减少实际的 I/O 操作次数，从而提高数据传输的效率
- 默认缓冲区大小在大多数Java版本（包括JDK 8, 11, 17等）中是**8192**
- 关流的时候会把传入的流也关闭



###### 字段

```java
protected byte[] 	buf 	// 存储数据的内部缓冲区数组。
protected int 		count 	// 缓冲区中有效字节的数量。
```

###### 构造方法

```java
public BufferedOutputStream(OutputStream out) // 创建一个新的缓冲输出流，以将数据写入指定的底层输出流。
public BufferedOutputStream(OutputStream out, int size)
    										// 创建一个新的缓冲输出流，以将具有指定缓冲区大小的数据写入指定的底层输出流。
```

###### 常见方法

```java
public void write(int b) 			// 将指定的字节写入此缓冲的输出流。(写入到内部缓冲区)
public void write(byte[] b, int off, int len)
    								// 将 len 个字节从指定的 byte 数组（偏移量为 off）写入此缓冲的输出流
public void flush() 				// 刷新此缓冲的输出流。这迫使所有缓冲的输出字节被写出到底层输出流中。
```



##### `PrintStream`字节打印流

###### 相关概念和知识

- **`FilterOutputStream`**的子类
- **`System.out`** 就是它的一个实例
- 设计初衷是为了**方便地打印各种数据值的文本表示形式**，它极大地简化了向控制台或文件输出信息的操作
- 它从不抛出 **`IOException`**；相反，异常情况会设置一个内部标志，可以通过 **`checkError()` 方法**测试

- **`System.out` (标准输出流)** **`System.err`** **(标准错误流)**都是这个流

###### 构造方法

```java
public PrintStream(OutputStream out) 					// 创建一个新的打印流。
public PrintStream(OutputStream out, boolean autoFlush) // 创建一个新的打印流，可选择是否开启自动刷新。
public PrintStream(OutputStream out, boolean autoFlush, String encoding) 
    													// 创建一个新的打印流，可指定自动刷新和字符编码。
public PrintStream(OutputStream out, boolean autoFlush, Charset charset) 
    													// (自JDK 10) 创建一个新的打印流，可指定自动刷新和字符集。

public PrintStream(String fileName) 					// 创建一个具有指定文件名的新的打印流，不自动刷新。
public PrintStream(String fileName, String csn)		    // 创建一个具有指定文件名和字符集名称的新的打印流。
public PrintStream(String fileName, Charset charset) 	// (自JDK 10) 创建一个具有指定文件名和字符集的新的打印流。

public PrintStream(File file) 							// 创建一个具有指定文件且不带自动行刷新的新打印流。
public PrintStream(File file, String csn) 				// 创建一个具有指定文件和字符集名称的新打印流。
public PrintStream(File file, Charset charset) 			// (自JDK 10) 创建一个具有指定文件和字符集的新打印流。
```

###### 常用方法

```java
public void 	flush() 				// 刷新该流。
public void 	close() 				// 关闭该流。
public boolean 	checkError() 			// 刷新流并检查其错误状态。如果已经发生错误，此方法将返回 true
protected void 	clearError()			// 清除此流的内部错误状态
```

```JAVA
public void 		write(int b) 						// 将指定的字节写入此流
public void 		write(byte[] buf, int off, int len)	// 将 len 字节从指定的 off 偏移量开始的字节数组写入此流
public void 		write(byte[] b) 					// (新增) 将 b.length 个字节写入此流
public void 		writeBytes(byte[] buf)				// 将指定字节数组中的所有字节写入此流
```

```JAVA
public PrintStream printf(String format, Object... args)	//使用指定格式字符串和参数，将一个格式化字符串写入此输出流
public PrintStream printf(Locale l, String format, Object... args)
    												// 使用指定区域、格式字符串和参数，将一个格式化字符串写入此输出流
public PrintStream format(String format, Object... args)			// 功能同 printf
public PrintStream format(Locale l, String format, Object... args)	// 功能同 printf
```

```JAVA
public void 	print(boolean b) 					// 打印 boolean 值。
public void 	print(char c) 						// 打印字符。
public void 	print(int i)					 	// 打印整数。
public void 	print(long l) 						// 打印 long 整数。
public void 	print(float f) 						// 打印浮点数。
public void 	print(double d) 					// 打印 double 精度浮点数。
public void 	print(char[] s) 					// 打印字符数组。
public void 	print(String s) 					// 打印字符串。
public void 	print(Object obj) 					// 打印对象（通过调用其 toString() 方法）。
    
public void 	println() 							// 通过写入行分隔符字符串终止当前行。
public void	 	println(boolean x) 					// 打印 boolean 值，然后终止行。
public void 	println(char x) 					// 打印字符，然后终止行。
public void 	println(int x) 						// 打印整数，然后终止行。
public void 	println(long x) 					// 打印 long 整数，然后终止行。
public void 	println(float x) 					// 打印浮点数，然后终止行。
public void 	println(double x) 					// 打印 double 精度浮点数，然后终止行。
public void 	println(char[] x) 					// 打印字符数组，然后终止行。
public void 	println(String x) 					// 打印 String，然后终止行。
public void 	println(Object x) 					// 打印 Object，然后终止行。
```

```JAVA
public PrintStream append(char c)				 				// 将指定字符附加到此输出流。
public PrintStream append(CharSequence csq) 					// 将指定字符序列附加到此输出流。
public PrintStream append(CharSequence csq, int start, int end) // 将指定字符序列的子序列附加到此输出流。
```

```JAVA
protected void setError()							//将流的错误状态设置为 true
```



##### `DataOutputStream`数据输出流

- **`FilterOutputStream`**的子类
- 数据输出流允许应用程序以适当的、可移植的方式将基本的Java数据类型写入输出流中

###### 字段

```java
protected int written // 到目前为止写入数据输出流的字节数。
```

###### 构造方法

```java
public DataOutputStream(OutputStream out) // 创建一个新的数据输出流，将数据写入指定的基础输出流。
```

###### 常见方法

```java
public final void 	writeBoolean(boolean v) // 将一个 boolean 值作为 1 字节值写入基础输出流。
public final void 	writeByte(int v) 		// 将一个 byte 值作为 1 字节值写入基础输出流。
public final void 	writeShort(int v) 		// 将一个 short 值作为 2 字节值写入基础输出流（高字节优先）。
public final void 	writeChar(int v) 		// 将一个 char 值作为 2 字节值写入基础输出流（高字节优先）。
public final void 	writeInt(int v) 		// 将一个 int 值作为 4 字节值写入基础输出流（高字节优先）。
public final void 	writeLong(long v) 		// 将一个 long 值作为 8 字节值写入基础输出流（高字节优先）。
public final void 	writeFloat(float v) 	
    						//使用Float.floatToIntBits将浮点参数转换为int值,然后将该int值作为4字节数量写入基础输出流
public final void 	writeDouble(double v) 
    				//使用Double.doubleToLongBits将double参数转换为long值,然后将该long值作为 8 字节数量写入基础输出流
public final void 	writeBytes(String s) //将字符串作为字节序列写入基础输出流。字符串中每个字符按顺序写出，丢弃其高八位。
public final void 	writeChars(String s) //将字符串作为字符序列写入基础输出流。每个字符都使用两个字节写入（高字节优先）
public final void 	writeUTF(String str) //以 modified-UTF-8 格式的机器无关方式将一个字符串写入基础输出流。
```

```java
public void 		write(int b) 			// 将指定字节（参数 b 的低 8 位）写入基础输出流。
public void 		write(byte[] b, int off, int len)
										// 将 len 个字节从指定的 byte 数组（从偏移量 off 处开始）写入基础输出流
public void 		flush() 			// 清空此数据输出流。这迫使所有缓冲的输出字节被写出到底层流中
public final int 	size() 				// 返回计数器 written 的当前值，即到目前为止写入此数据输出流的字节数
```



### 字符流

- 处理文本文件

- 当字符流在未明确指定编码格式的情况下处理数据时，它会使用平台的**默认字符集**。这个默认字符集由Java虚拟机（JVM）在启动时根据底层操作系统的区域设置来决定
- **字符流可以指定字符编码**，或在构造方法中，或在成员方法中，但是**字节流不能指定，它也无需指定**
- 读到不是字符的，会抛异常

#### 字符输入流

##### `Reader`类

- 抽象类



###### 字段

```java
protected Object lock // 用于同步此流上的操作的对象
```

###### 构造方法

```java
protected Reader() // 创建一个新的字符流 reader，其关键部分将同步 reader 自身。
protected Reader(Object lock) // 创建一个新的字符流 reader，其关键部分将同步给定的对象
```

###### 常用方法

```java
public abstract int  read(char[] cbuf, int off, int len)//将字符读入数组的一部分。这是子类必须实现的核心抽象方法之一
public abstract void close() 			// 关闭该流并释放与之关联的所有系统资源。这是子类必须实现的另一个核心抽象方法
```

```java
public int read() 							// 读取单个字符。如果已到达流的末尾，则返回 -1。
public int read(char[] cbuf) 				// 将字符读入数组。返回读入的字符数；如果已到达流末尾，则返回 -1。
public int read(java.nio.CharBuffer target)	// 尝试将字符读入指定的字符缓冲区
```

```java
public long 	skip(long n) 				// 跳过n个字符。
public boolean 	ready() 					// 判断此流是否已经准备好被读取。
public boolean 	markSupported() 			// 判断此流是否支持 mark() 操作。
public void	 	mark(int readAheadLimit) 	// 标记流中的当前位置。
public void 	reset() 					// 重置该流。
```

```java
public long transferTo(Writer out) // 从此 reader 读取所有字符，并按读取的顺序将字符写入给定的 writer(自JDK 10添加)
```

```java
public static Reader nullReader() // 返回一个新的 Reader，它不读取任何字符，并且在流的末尾。(自JDK 11添加)
```



###### 一些方法的用法和细节

- 子类体系中相同方法返回值一样

- **`read()`方法**

  - 和字节流的**`read()`方法**在物理层面上一个字节一个字节的读不同，这里的**`read()`**方法是是**逻辑层面的**。它会先读取一个或多个字节，然后根据指定的字符编码将这些字节**解码**成一个字符，最后才返回，返回值的范围是 **0 到 65535**，代表读取到的那个字符的Unicode码值。如果到达流的末尾，则返回 **-1**
  - Reader **不会只读一个字节就返回**。它会根据构造时指定的**字符编码**（如UTF-8）来分析读取到的字节序列

- **`read(char[] cbuf)`方法**

  - 想让读取速度更快，可以用这个方法

    - 传过去一个**“空蓝子”**，底层在读取的时候会尝试用传递过去的 **数组** 装数据，返回值是**这次往数组中装的数据个数(即读到的数据数)**，但是**如果没有读到数据，返回值是-1**，然后每调用一次方法，就把数据往这个数组中进行**尽可能多的装填**，**`char`数组** 中的数据在每一次被装填的时候都是 **从数组头开始覆盖装填**，纯手写比缓冲流性能还高一点

      > 为啥没读到数据返回的是 -1 ，而不是返回 0？
      >
      > - 一个我认为的原因是为了一致性，即只要是**`read`方法**，读不到数据就返回-1
      >
      > - 还有一个原因是**0**这个返回值主要是为**非阻塞式IO（Non-Blocking IO）**准备的，尤其是在**网络编程**中：
      >   **想象一个网络服务器的场景：**
      >   服务器正在监听一个网络连接（Socket），
      >   它调用 **`socketInputStream.read(buffer)`**，想看看客户端有没有发数据过来。
      >   在这一瞬间，客户端恰好什么都没发。
      >   如果这时方法返回-1，服务器就会以为客户端断开了连接，从而关闭这个socket。这显然是错误的！
      >   所以，这时方法应该返回 **0**！这个0的含义是：“**我现在没收到数据，但这不代表连接断了，你过一会儿再来问问看。**”
      >   服务器看到**返回0**，就知道连接还是好的，于是它可以先去处理其他客户端的请求，稍后再回来检查这个连接。
      > - ......

    - **字符流内部有精巧的机制（缓冲区和状态管理），确保永远不会返回一个“半个字符”。它要么成功解码出一个完整的字符，要么一个都不返回（并等待更多字节）**

    - 像缓冲流这种已经加快速度的流中，如果还使用这种方法来读取，会更快

  

##### `InputStreamReader`转换输入流

- **也是一个装饰器流，功能是“字节-->字符”的转换，也是字符流的基石**

- **文件、网络等数据源中存储的原始数据是字节（bytes），而不是字符（chars），想在字节和字符之间操作，必须通过转换流**

- **`InputStreamReader`**将传入的**字节转换为字符**

- **字符流们并不是是直接把字符写入文件，这是一个错误认知，实际上它们经过了封装和隐藏，内部执行了字节到字符的转换**
- 这个可以用搭配转换输出流将文件进行转码，其实字符流实现的方式很多



###### 构造方法

```java
public InputStreamReader(InputStream in)					 // 创建一个使用平台默认字符集的 InputStreamReader。
public InputStreamReader(InputStream in, String charsetName) // 创建使用指定字符集的InputStreamReader(最常用)
public InputStreamReader(InputStream in, Charset cs)		 //创建使用给定字符集的InputStreamReader(推荐,更健壮)
public InputStreamReader(InputStream in, CharsetDecoder dec) // 创建使用给定字符集解码器的 InputStreamReader。
```

###### 常见方法

```java
public String 		getEncoding() 							// 返回此流使用的字符编码的名称。
public int 			read() 									// 读取单个字符。
public int 			read(char[] cbuf, int off, int len)		// 将字符读入数组的一部分。(实现Reader的核心抽象方法)
public boolean 		ready() 								// 判断此流是否已准备好被读取。
public void 		close() 					// 关闭该流并释放与之关联的所有系统资源。(会关闭底层的InputStream)
```



##### `BufferedReader`缓冲输入流

- 缓冲流
- 通过在内存中引入缓冲区（Buffer）来减少实际的 I/O 操作次数，从而提高数据传输的效率
- 默认缓冲区大小在大多数Java版本（包括JDK 8, 11, 17等）中是**8192**
- 关流的时候会把传入的流也关闭
- 底层加速的原理是它底层自动灵活的调用了底层的**`read(char[] buffer)`** 方法
  - 所以我记得在某些情况下，基础读取流会比缓冲流更快




###### 构造方法

```java
public BufferedReader(Reader in) 			// 创建一个使用默认大小输入缓冲区的缓冲字符输入流。
public BufferedReader(Reader in, int sz) 	// 创建一个使用指定大小输入缓冲区的缓冲字符输入流。
```

###### 常见方法

```java
public int 				read() 								// 读取单个字符。
public int 				read(char[] cbuf, int off, int len) // 将字符读入数组的一部分。
public String 			readLine() 
//(最常用)读取一个文本行。通过下列字符之一即可认为某行已终止:换行('\n')、回车('\r')或回车后直接跟着换行。
public Stream<String> 	lines() // (自JDK 8添加) 返回一个元素为从此 BufferedReader 中读取的行的 Stream。
```

```java
public boolean 	ready() 					// 判断此流是否已经准备好被读取。
public boolean 	markSupported() 			// 判断此流是否支持 mark() 操作。
public void	 	mark(int readAheadLimit) 	// 标记流中的当前位置。
public void 	reset() 					// 重置该流。
public void 	close()						// 关闭流
```

###### 一些方法的用法和细节

- **`readLine()`**非常好用，也很常用



##### `StringReader` 

- 常方便的**内存流**，与 ByteArrayInputStream 对应，但它是**面向字符**的

###### 构造方法

```java
public StringReader(String s) // 创建一个新字符串 reader。其源是参数字符串 s。
```

###### 常见方法

```java
public int 		read() 										// 读取单个字符。
public int 		read(char[] cbuf, int off, int len)			// 将字符读入数组的一部分。
```

```java
public long 		skip(long ns) 				// 跳过输入流中的指定数量的字符。
public boolean 		ready() 					// 判断此流是否已准备好被读取。只要流未关闭，就返回 true。
public boolean 		markSupported() 			// 判断此流是否支持 mark() 操作。StringReader支持此功能，返回 true
public void 		mark(int readAheadLimit) 	// 标记流中的当前位置
public void 		reset() 			// 将该流重置到最新的标记，如果从未标记过，则重置到字符串的开头。
public void 		close() 			// 关闭StringReader无效。此方法在关闭该流后仍可被调用，而不会生成IOException
```



##### `FileReader`

- **`InputStreamReader`转换输入流**的子类

- 非常常见的**文件流**，设计初衷是为了让“读取文本文件”这个常用操作的代码更简洁

- **文件指针**

  - 当一个 **`FileInputStream` 对象**被创建时，Java虚拟机会向操作系统请求**打开指定的文件**。操作系统响应此请求，并返回一个唯一的标识——在Unix/Linux中称为**文件描述符 (File Descriptor)**，在Windows中称为**文件句柄 (File Handle)**。这个句柄是程序与文件之间建立连接的凭证。

    这个由操作系统维护的文件句柄是“有状态”的，其核心是内置了一个**文件指针**。当文件被打开时，这个**指针自动被设置为 0**，即指向文件的起始字节。

    每当调用 read() 方法时，操作系统会从文件指针的当前位置开始读取数据。关键在于，**读取操作完成后，操作系统会自动将该指针向后移动相应的字节数**。例如，读取了8个字节，指针就自动前进8位。

    因此，**`FileInputStream`** 本身并不负责追踪或管理读取的位置。它更像一个忠实的“代理”，将Java层面的读取请求直接转发给操作系统，并利用操作系统提供的文件指针自动前进的能力，从而实现了天然的、顺序向前的读取流。

###### 构造方法

```java
public 	FileReader(String fileName) 		// 在给定文件名的情况下创建一个新 FileReader。
public 	FileReader(File file) 				// 在给定 File 对象的情况下创建一个新 FileReader。
public 	FileReader(FileDescriptor fd) 		// 在给定 FileDescriptor 的情况下创建一个新 FileReader。
public 	FileReader(String fileName, Charset charset)
											// (自JDK 11添加) 在给定文件名和字符集的情况下创建一个新 FileReader
public 	FileReader(File file, Charset charset)
											// (自JDK 11添加) 在给定File对象和字符集的情况下创建一个新FileReader
```

###### 常见方法

- 没有定义任何属于自己的方法，**完全继承**了其父类 **`java.io.InputStreamReader`** 和祖父类 **`java.io.Reader`** 的所有公开方法



###### 一些方法和细节

- **`read()`**

  ```java
  int ch;
  while ((ch = f.read()) != -1) {
      System.out.println((char) ch);
  }
  f.close();
  ```

- **`read(char[] cbuf)`**

  ```java
  int len;
  char[] chars = new char[1024];
  while ((len = f.read(chars)) != -1) {
      System.out.println(new  String(chars,0,len));
  }
  f.close();
  ```

  



#### 字符输出流

##### `Writer`类

- 抽象类
- 没有直接写入字符的，形参为**`char c`**的方法，我感觉不好，服了



###### 字段

```java
protected Object lock // 用于同步此流上的操作的对象
```

###### 构造方法

```java
protected Writer() 				// 创建一个新的字符流 writer，其关键部分将同步 writer 自身。
protected Writer(Object lock) 	// 创建一个新的字符流 writer，其关键部分将同步给定的对象。
```

###### 常见方法

```java
public abstract void write(char[] cbuf, int off, int len)//写入字符数组的一部分。这是子类必须实现的核心抽象方法之一。
public abstract void flush() // 刷新该流的缓冲。这是子类必须实现的另一个核心抽象方法。
public abstract void close() // 关闭此流，在关闭前会先刷新它。这是子类必须实现的第三个核心抽象方法。
```

```java
public void write(int c)		// 写入单个字符。
public void write(char[] cbuf) 	// 写入字符数组。
public void write(String str) 	// 写入字符串。
public void write(String str, int off, int len)// 写入字符串的某一部分。
```

```java
public Writer append(char c) 			// 将指定字符附加到此 writer。
public Writer append(CharSequence csq) 	// 将指定的字符序列附加到此 writer。
public Writer append(CharSequence csq, int start, int end)// 将指定字符序列的子序列附加到此 writer。
```

```java
public static Writer nullWriter() // 返回一个新的 Writer，它会丢弃所有写入的字符数据。(自JDK 11添加)
```



###### 一些方法的用法和细节

- 子类体系中相同方法返回值一样



##### `OutputStreamWriter`转换输出流

- **转换流也是一个装饰器类，功能是”字符-->字节”的转换，也是字符流的基石**

- **文件、网络等数据源中存储的原始数据是字节（bytes），而不是字符（chars），想在字节和字符之间操作，必须通过转换流**

- **`OutputStreamWriter`**将**字节** 转换成 **字符**

- **字符流们并不是是直接把字符写入文件，这是一个错误认知，实际上它们经过了封装和隐藏，内部执行了字节到字符的转换**

- 这个可以用搭配转换输入流将文件进行转码，其实字符流实现的方式很多



###### 构造方法

```java
public OutputStreamWriter(OutputStream out) // 创建一个使用平台默认字符集的 OutputStreamWriter。
public OutputStreamWriter(OutputStream out, String charsetName)
    										// 创建使用指定字符集的 OutputStreamWriter。(最常用)
public OutputStreamWriter(OutputStream out, Charset cs)
											// 创建使用给定字符集的 OutputStreamWriter。(推荐，更健drawn)
public OutputStreamWriter(OutputStream out, CharsetEncoder enc)
											// 创建使用给定字符集编码器的 OutputStreamWriter。
```

###### 常见方法

```java
public String 			getEncoding() 							// 返回此流使用的字符编码的名称。
public void 			write(int c) 							// 写入单个字符。
public void 			write(char[] cbuf, int off, int len)	//写入字符数组的一部分(实现Writer的核心抽象方法)
public void 			write(String str, int off, int len)		// 写入字符串的某一部分。
public void 			flush() 								// 刷新该流的缓冲(会刷新底层的OutputStream)
public void 			close() 						// 关闭此流，在关闭前会先刷新它(会关闭底层的OutputStream)
```



##### `BufferedWriter`缓冲输出流

- 缓冲流
- 通过在内存中引入缓冲区（Buffer）来减少实际的 I/O 操作次数，从而提高数据传输的效率
- 默认缓冲区大小在大多数Java版本（包括JDK 8, 11, 17等）中是**8192**

- 关流的时候会把传入的流也关闭



###### 构造方法

```java
public BufferedWriter(Writer out) 			// 创建一个使用默认大小输出缓冲区的缓冲字符输出流。
public BufferedWriter(Writer out, int sz) 	// 创建一个使用指定大小输出缓冲区的缓冲字符输出流。
```

###### 常见方法

```java
public void newLine() 
    			//(最常用)写入一个行分隔符。该行分隔符字符串由系统属性 line.separator 定义，不一定是一个换行符 ('\n')。
public void write(int c) 							// 写入单个字符。
public void write(char[] cbuf, int off, int len) 	// 写入字符数组的一部分。
public void write(String s, int off, int len) 		// 写入字符串的某一部分。
public void flush() 								// 刷新该流的缓冲。
public void close()									// 关闭流
```



###### 一些方法的用法和细节

- **`write(String s)`和 `newLine()`的搭配**非常好用，也很常用



##### `PrintWriter`字符打印流

- 这是 **`PrintStream`** 的**字符流版本**，也是Java中处理**文本输出**的**首选工具**
- 与 **`PrintStream`** 相比，**`PrintWriter`** 更加规范，因为它严格地操作字符，并且在通过 **`OutputStream`** 构造时，会利用 **`OutputStreamWriter`** 进行正确的字符到字节的转换
- 是处理文本输出的**首选和推荐**方式
- 个人更喜欢，也更想让这个操作控制台，但是没办法哈哈哈

###### 字段

```java
protected Writer out // 此 PrintWriter 所基于的字符输出流。
```

###### 构造方法

```java
public PrintWriter(Writer out) 						// 创建新 PrintWriter，不自动刷新
public PrintWriter(Writer out, boolean autoFlush) 	// 创建新 PrintWriter，可选择是否开启自动行刷新

public PrintWriter(OutputStream out) 				// 根据现有的 OutputStream 创建不带自动行刷新的新 PrintWriter
public PrintWriter(OutputStream out, boolean autoFlush)//根据现有的OutputStream创建新 PrintWriter，可指定自动刷新
public PrintWriter(OutputStream out, boolean autoFlush, Charset charset)
    											// (自JDK 10) 根据现有 OutputStream 和字符集创建新 PrintWriter

public PrintWriter(String fileName) 				// 创建具有指定文件名的、不带自动行刷新的新 PrintWriter
public PrintWriter(String fileName, String csn) 	//创建具有指定文件名和字符集名称的、不带自动行刷新的新PrintWriter
public PrintWriter(String fileName, Charset charset)
    										//(自JDK 10)创建具有指定文件名和字符集的、不带自动行刷新的新PrintWriter

public PrintWriter(File file) 					// 创建具有指定文件的、不带自动行刷新的新 PrintWriter
public PrintWriter(File file, String csn) 		// 创建具有指定文件和字符集名称的、不带自动行刷新的新 PrintWriter
public PrintWriter(File file, Charset charset) 	// (自JDK 10) 创建具有指定文件和字符集的新 PrintWriter
```

###### 常见方法

```java
public void 		flush() 		// 刷新该流。
public void 		close() 		// 关闭该流。
public boolean 		checkError() 	// 如果流未关闭，则刷新流并检查其错误状态。
protected void 		clearError()	//清除此流的错误状态
protected void 		setError() 		// 指示已发生错误。此方法供子类使用。
```

```java
public void 			write(int c) 						// 写入单个字符。
public void 			write(char[] buf, int off, int len)	// 写入字符数组的某一部分。
public void 			write(char[] buf) 					// 写入一个字符数组。
public void 			write(String s, int off, int len)	// 写入字符串的某一部分。
public void 			write(String s) 					// 写入字符串。
```

```java
public PrintWriter 	printf(String format, Object... args)
    												// 使用指定格式字符串和参数，将一个格式化的字符串写入此writer
public PrintWriter 	printf(Locale l, String format, Object... args)
    												//使用指定区域,格式字符串和参数，将一个格式化的字符串写入此writer
public PrintWriter 	format(String format, Object... args)			// 功能同 printf。
public PrintWriter 	format(Locale l, String format, Object... args)	// 功能同 printf。
```

```java
public void 		print(boolean b) 		// 打印 boolean 值。
public void 		print(char c) 			// 打印字符。
public void			print(int i) 			// 打印整数。
public void 		print(long l) 			// 打印 long 整数。
public void 		print(float f) 			// 打印浮点数。
public void 		print(double d) 		// 打印 double 精度浮点数。
public void 		print(char[] s) 		// 打印字符数组。
public void 		print(String s) 		// 打印字符串。
public void 		print(Object obj) 		// 打印对象（通过调用其 toString() 方法）

public void 		println() 				// 通过写入行分隔符字符串终止当前行。
public void 		println(boolean x) 		// 打印 boolean 值，然后终止行。
public void			println(char x) 		// 打印字符，然后终止行。
public void 		println(int x) 			// 打印整数，然后终止行。
public void 		println(long x) 		// 打印 long 整数，然后终止行。
public void 		println(float x) 		// 打印浮点数，然后终止行。
public void 		println(double x) 		// 打印 double 精度浮点数，然后终止行。
public void 		println(char[] x) 		// 打印字符数组，然后终止行。
public void 		println(String x) 		// 打印 String，然后终止行。
public void 		println(Object x) 		// 打印 Object，然后终止行。
```

```java
public PrintWriter 	append(char c) 								// 将指定字符附加到此 writer
public PrintWriter 	append(CharSequence csq) 					// 将指定字符序列附加到此 writer
public PrintWriter 	append(CharSequence csq, int start, int end)// 将指定字符序列的子序列附加到此 writer
```



##### `StringWriter`

- **`OutputStreamWriter`转换输出流**的子类
- 许你将字符数据写入到内存中的一个字符串缓冲区里，而不是直接写入文件

###### 构造方法

```java
public StringWriter() 					// 使用默认初始字符串缓冲区大小创建一个新的字符串 writer。
public StringWriter(int initialSize) 	// 使用指定初始字符串缓冲区大小创建一个新的字符串 writer。
```

###### 常见方法

```java
public void write(int c) 								// 写入单个字符到字符串缓冲区。
public void write(char[] cbuf, int off, int len)		// 写入字符数组的一部分到字符串缓冲区。
public void write(String str) 							// 写入一个字符串到字符串缓冲区。
public void write(String str, int off, int len)			// 写入字符串的某一部分到字符串缓冲区。
```

```java
public String 			toString() 		// 以字符串形式返回该缓冲区的内容。
public StringBuffer 	getBuffer() 	// 返回该 writer 使用的字符串缓冲区本身。
```

```java
public StringWriter 	append(char c) 									// 将指定字符附加到此 writer
public StringWriter 	append(CharSequence csq) 						// 将指定的字符序列附加到此 writer
public StringWriter 	append(CharSequence csq, int start, int end)	// 将指定字符序列的子序列附加到此 writer
```

```java
public void 			flush() 	// 刷新流。刷新此流无效，因为所有内容都直接写入内存中的 StringBuffer。
public void 			close() 	// 关闭 StringWriter 无效。此方法在关闭该流后仍可被调用，而不会生成 IOException。
```



##### `FileWriter` 

- **`OutputStreamWriter`的子类**

- 用于写入字符文件的便捷类

- **`FileWriter`** 的构造方法被设计为在默认情况(没有append指定)下**覆盖**写文件

  > 就是有个开关，开了后就能续写，不开就不能续写，默认是关的

  - - **如果文件不存在**：它会尝试创建一个新的空文件，**前提是父级路径要存在**

    - **如果文件已存在**：它会打开该文件，并将其长度截断为0，以便从头开始写入新数据，这就是为什么在还没调用 **`write()`** 方法写入任何数据之前，文件内容就已经消失的原因

    - **如果父级路径不存在**：抛出异常




###### 构造方法

```java
public FileWriter(String fileName) // 在给定文件名的情况下构造一个 FileWriter 对象。如果文件存在，则覆盖。
public FileWriter(String fileName, boolean append)
						// 在给出文件名以及一个指示是否附加写入数据的 boolean 值的情况下构造一个 FileWriter 对象。
    
public FileWriter(File file) // 在给定 File 对象的情况下构造一个 FileWriter 对象。如果文件存在，则覆盖。
public FileWriter(File file, boolean append)
							// 在给定File对象以及一个指示是否附加写入数据的boolean值的情况下构造一个FileWriter对象
public FileWriter(FileDescriptor fd) // 在给定 FileDescriptor 的情况下构造一个 FileWriter 对象。
    
public FileWriter(String fileName, Charset charset)		
    								// (自JDK 11添加) 在给定文件名和字符集的情况下构造一个新 FileWriter。
public FileWriter(String fileName, Charset charset, boolean append)
									// (自JDK 11添加) 指示是否附加写入数据。
    
public FileWriter(File file, Charset charset)
									// (自JDK 11添加) 在给定 File 对象和字符集的情况下构造一个新 FileWriter。
public FileWriter(File file, Charset charset, boolean append)
									// (自JDK 11添加) 指示是否附加写入数据。
```

###### 常见方法

- 没有定义任何属于自己的方法，**完全继承**了其父类 **`java.io.OutputStreamWriter`** 和祖父类 **`java.io.Writer`** 的所有公开方法



###### 一些方法和细节



### 一些别的流

#### `RandomAccessFile`随机访问流

- **`public class RandomAccessFile extends Object implements DataOutput , DataInput , Closeable`**

- **`RandomAccessFile`** **既不是标准的字节流，也不是字符流，但它的操作核心是基于字节的。**
- 它**既能读又能写**，拥有一个“文件指针”，可以**自由地在文件的任何位置移动**并进行读写操作
- 它实现了 **`DataInput`** 和 **`DataOutput`** 接口，所以它拥有所有 **`DataInputStream`** 和 **`DataOutputStream`** 的功能，可以直接读写Java基本数据类型

##### 构造方法

```java
public RandomAccessFile(String name, String mode) //创建从中读取和向其中写入（可选）的随机访问文件流，该文件具有指定名称
public RandomAccessFile(File file, String mode) //创建从中读取和向其中写入（可选）的随机访问文件流,该文件由File参数指定
```

> **关于 mode 参数的说明:**
>
> - **"`r`"** - 只读。如果文件不存在，则会抛出 FileNotFoundException。
> - **"`rw`"** - 读写。如果文件不存在，则会尝试创建该文件。
> - **"`rws`"** - 读写。要求对文件的内容和元数据（如文件长度、最后修改时间）的每个更新都同步写入到底层存储设备。
> - **"`rwd`"** - 读写。要求对文件内容的每个更新都同步写入到底层存储设备。

##### 常见方法

```java
public void 	seek(long pos) 	// (最核心方法) 设置到此文件开头测量到的文件指针偏移量，在该位置发生下一个读取或写入操作
public long 	getFilePointer() 			// 返回此文件中文件指针的当前偏移量
public long 	length() 		 			// 返回此文件的长度
public void 	setLength(long newLength) 	// 设置此文件的长度。如果新长度小于当前长度，则文件将被截断
```

```JAVA
public int 		  read() 							//从此文件中读取一个数据字节。
public int 		  read(byte[] b, int off, int len)	// 将 len 个数据字节从此文件读入 byte 数组。
public int 		  read(byte[] b) 					// 将 b.length 个数据字节从此文件读入 byte 数组。
public final void readFully(byte[] b) 				// 从此文件将 b.length 个字节读入 byte 数组，从当前文件指针开始
public final void readFully(byte[] b, int off, int len)// 从此文件将 len 个字节读入 byte 数组，从当前文件指针开始
public int 		  skipBytes(int n) 					// 尝试跳过输入的 n 个字节以丢弃跳过的字节。
public boolean 	  readBoolean() 					// 从此文件读取一个 boolean。
public byte 	  readByte() 						// 从此文件读取一个有符号的 8 位值。
public int 		  readUnsignedByte() 				// 从此文件读取一个无符号的 8 位数。
public short 	  readShort() 						// 从此文件读取一个有符号的 16 位数。
public int 		  readUnsignedShort() 				// 从此文件读取一个无符号的 16 位数。
public char	 	  readChar() 						// 从此文件读取一个字符。
public int 		  readInt() 						// 从此文件读取一个有符号的 32 位整数。
public long 	  readLong() 						// 从此文件读取一个有符号的 64 位整数。
public float 	  readFloat() 						// 从此文件读取一个 float。
public double 	  readDouble() 					  	// 从此文件读取一个 double。
public String 	  readLine() 						// 从此文件读取文本的下一行。
public String 	  readUTF() 						// 从此文件读取一个采用 modified-UTF-8 格式的字符串。
```

```JAVA
public void 	write(int b) 					// 向此文件写入指定的字节。
public void 	write(byte[] b) 			// 将 b.length 个字节从指定 byte 数组写入到此文件，并从当前文件指针开始。
public void 	write(byte[] b, int off, int len)// 将 len 个字节从指定 byte 数组的偏移量 off 处开始写入此文件。
public void 	writeBoolean(boolean v) 		// 将 boolean 写出到此文件，用 1 个字节值表示。
public void 	writeByte(int v) 				// 将 byte 写出到此文件，用 1 个字节值表示。
public void 	writeShort(int v) 				// 将 short 写出到此文件，用 2 个字节表示。
public void 	writeChar(int v) 				// 将 char 写出到此文件，用 2 个字节表示。
public void 	writeInt(int v) 				// 将 int 写出到此文件，用 4 个字节表示。
public void 	writeLong(long v) 				// 将 long 写出到此文件，用 8 个字节表示。
public void	 	writeFloat(float v) 			// 将 float 写出到此文件，用 4 个字节表示。
public void 	writeDouble(double v) 			// 将 double 写出到此文件，用 8 个字节表示。
public void 	writeBytes(String s) 			// 将字符串按字节序列写出到此文件。
public void 	writeChars(String s) 			// 将字符串按字符序列写出到此文件。
public void 	writeUTF(String str) 			// 以 modified-UTF-8 格式将一个字符串写入此文件。
```

```JAVA
public void	 				close() 			// 关闭此随机访问文件流并释放与该流相关的所有系统资源。
public final FileDescriptor getFD() 			// 返回与此流有关的不透明文件描述符。
public FileChannel 			getChannel()	 	 	// 返回与此文件关联的唯一 FileChannel 对象。(通往NIO的桥梁)
```



#### `Properties`类

- 是**`Map`**集合体系中**`Hashtable`类**的子类
- 但是 这个类的 **核心是文件的输入/输出**，而不是作为一个普通的内存 **`Map`** 来用
- **`Properties`** 类提供了两个独一无二、至关重要的方法：**`load()`** 和 **`store()`**。这两个方法**都必须依赖 I/O 流**。如果没有了 I/O，**`Properties`** 就退化成了一个功能受限的 **`Hashtable`**，其存在的意义就大打折扣了

##### 字段

```java
protected Properties defaults // 一个属性列表，作为此属性列表的默认值。
```

##### 构造方法

```java
public Properties() 					// 创建一个没有默认值的空属性列表。
public Properties(Properties defaults) // 创建一个带有指定默认值的空属性列表。
```

##### 常见方法

```java
public Object 		setProperty(String key, String value)
							// 调用 Hashtable 的 put 方法。是存储属性的首选方法，因为它强制键和值为字符串。
public String 		getProperty(String key) 
    						// 用指定的键在此属性列表中搜索属性。如果在此属性列表中未找到该键，则接着递归检查默认属性列表。
public String 		getProperty(String key, String defaultValue)
							// 用指定的键在属性列表中搜索属性。如果找不到，则返回提供的默认值。
```

```java
//load是输入操作，把文件（输入流）中的所有数据“倒入”到当前调用load方法的对象的Properties双列集合中
public void load(Reader reader) 				// 按简单的面向行的格式，从输入字符流中读取属性列表（键和元素对）
public void load(InputStream inStream) 			// 从输入字节流中读取属性列表。注意：假定流使用 ISO 8859-1 字符编码
public void loadFromXML(InputStream in) 			// 将 XML 文档所表示的所有属性加载到此属性表中。

//store是输出操作，把当前调用load方法的对象的Properties双列集合中的所有数据“倒出”到文件（输出流）中
//String comments:用来给生成的 .properties 文件在文件顶部添加一段描述性的注释的
public void store(Writer writer, String comments)	// 将此 Properties 表中的属性列表（键和元素对）写入输出字符流
public void store(OutputStream out, String comments)// 将此 Properties 表中的属性列表（键和元素对）写入输出字节流
public void storeToXML(OutputStream os, String comment) 	// 发出一个表示此表中包含的所有属性的XML文档
public void storeToXML(OutputStream os, String comment, String encoding)
														//使用指定的编码，发出一份表示此表中包含的所有属性的XML文档
public void storeToXML(OutputStream os, String comment, Charset charset)
											// (自JDK 10) 使用指定的字符集，发出一份表示此表中包含的所有属性的XML文档
```

```java
public Set<String> 		stringPropertyNames() //返回此属性列表中的键集,其中键及其对应的值是字符串.返回的Set是类型安全的
public void 			list(PrintStream out) // 将属性列表打印到指定的输出流。此方法对调试很有用。
public void 			list(PrintWriter out) // 将属性列表打印到指定的输出流。此方法对调试很有用。
public Enumeration<?> 	propertyNames() 
    						//返回此属性列表中的所有键的枚举，如果在默认属性列表中未找到同名的键，则包括默认属性列表中的键
```



## I/O工具库

### Commons io

- Commons IO 是 Apache 软件基金会 Apache Commons 项目中的一个组件，它是一个专注于简化 Java I/O（输入/输出）操作的工具库
- Java 原生的 java.io 包功能很强大，但对于一些日常、重复性的任务，代码会显得很冗长和繁琐（例如，完整读取一个文件、递归删除目录等）。Commons IO 的目标就是提供一组简单、健壮、高效的工具类，用更少的代码来完成这些常见任务。



# NIO