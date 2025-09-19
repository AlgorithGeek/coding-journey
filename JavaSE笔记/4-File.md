# File

## 文件的几种路径

### 绝对路径

- **绝对路径**是一个**文件的完整路径**，它**从文件系统的根目录开始，逐级向下，直到文件名本身**
  - **根目录可以被理解为计算机文件系统的**最顶层目录**，是所有其他文件和目录的**起点**或**祖先**
    - **Windows** 采用的是**多根目录**的结构，**每个磁盘分区（盘符）都有自己的根目录**
      - **表示符号**：**盘符 + 冒号 + 反斜杠**，例如 **`C:\`**、**`D:\`** 等
    - **Linux、macOS 以及其他所有类Unix系统**采用的是**单根目录**结构。**整个文件系统只有一个唯一的根目录**。
      - **表示符号**：一个 **单独的正斜杠`/`**

### 相对路径

- **相对路径不是从根目录开始的**，而是从一个**当前工作目录**开始
  - **“当前工作目录”**通常是你**启动Java程序（运行JVM）时所在的目录**
- **表示方法**
  - **`. `**(一个点)：代表**当前目录**。
  - **`.. `**(两个点)：代表**上一级目录**（父目录）。

### ClassPath类路径







## File类

- **`java.io.File`** **对象代表的不是文件或目录本身，而是文件系统中的一个“路径名”的抽象表示**
  - 它的主要作用是**定位**文件系统中的一个东西，并提供**操作**这个东西（创建、删除、重命名等）的方法
  - 它本身**不包含**文件等的**内容数据**（比如文本、图片数据）

### 获取对象的方式

**构造方法**

```java
public File(String  pathname)				//通过将给定的路径名字符串转换为抽象路径名来创建一个新的 File 实例
public File(String  parent, String  child)	//从父路径名字符串和子路径名字符串创建一个新的 File 实例
public File(File  parent, String  child)	//从父抽象路径名和子路径名字符串创建一个新的 File 实例
public File(URI  uri)						//通过将给定的 file: URI 转换为抽象路径名来创建一个新的 File 实例
```



#### 一些字段

```java
public static final String separator  
    							//与系统有关的默认名称分隔符,为了方便,它被表示为一个字符串.(Windows是\,Linux/macOS是/)
public static final char   separatorChar		// 与系统有关的默认名称分隔符。
public static final String pathSeparator		// 路径列表中的路径分隔符。(Windows是 ;，Linux/macOS是 :)
public static final char   pathSeparatorChar	// 与系统有关的路径分隔符。
```



### 常用方法

```java
public String getName()				// 返回由此抽象路径名表示的文件或目录的名称。
public String getParent()			// 返回此抽象路径名的父目录的路径名字符串，如果此路径名没有指定父目录，则返回 null
public File   getParentFile()		// 返回此抽象路径名的父目录的File对象，如果此路径名没有指定父目录，则返回 null
public String getPath()				// 将此File对象转换为路径名字符串，这个路径就是构造时传入的路径。
public String getAbsolutePath() 	// 返回此File对象的绝对路径名字符串
public File   getAbsoluteFile()		// 返回此File对象的绝对路径形式
public String getCanonicalPath()	// 返回此File对象的规范路径名字符串 (会解析 ".." 和 ".")
public File   getCanonicalFile()	// 返回此File对象的规范形式
public long   length()				// 返回由此抽象路径名表示的文件的长度（以字节为单位）。
           					// 注意：如果此路径名表示目录，则返回值未指定。
public long   lastModified()		// 返回此抽象路径名表示的文件最后一次被修改的时间（毫秒时间戳）
public long   getTotalSpace()		// 返回此抽象路径名所在分区的大小（以字节为单位）
public long   getFreeSpace()		// 返回此抽象路径名所在分区的未分配字节数
public long   getUsableSpace()		// 返回此抽象路径名所在分区上可用于此虚拟机的字节数
```

```java
public boolean exists()			// 测试此抽象路径名表示的文件或目录是否存在。（最常用）
public boolean isDirectory()	// 测试此抽象路径名表示的文件是否是一个目录。
public boolean isFile()			// 测试此抽象路径名表示的文件是否是一个标准文件。
public boolean isHidden()		// 测试此抽象路径名命名的文件或文件夹是否是隐藏的。
public boolean isAbsolute()		// 测试此抽象路径名是否为绝对路径。
public boolean canRead()		// 测试应用程序是否可以读取此抽象路径名表示的文件。
public boolean canWrite()		// 测试应用程序是否可以修改此抽象路径名表示的文件。
public boolean canExecute()		// 测试应用程序是否可以执行此抽象路径名表示的文件。
```

```java
public boolean createNewFile()		// 当且仅当具有该名称的文件尚不存在时，原子地创建一个由此抽象路径名命名的新的空文件
public boolean mkdir()				// 创建由此抽象路径名命名的目录。(如果父目录不存在则会失败)
public boolean mkdirs()				// 创建由此抽象路径名命名的目录，包括任何必需但不存在的父目录。（更常用）
public boolean delete()				// 删除由此抽象路径名表示的文件或目录。注意：如果路径是目录，则该目录必须为空才能删除
public void    deleteOnExit()		// 请求在虚拟机终止时，删除此抽象路径名表示的文件或目录。
public boolean renameTo(File dest)	// 重新命名此抽象路径名表示的文件
   								//注意：许多情况下此操作不可靠，推荐使用 Files.move()
public boolean setLastModified(long time)		// 设置由此抽象路径名表示的文件或目录的最后修改时间
public boolean setReadOnly()					// 标记该抽象路径名指定的文件或目录，以便只允许读取操作
public boolean setWritable(boolean writable)	// 设置此抽象路径名的所有者或每个人的写权限。
public boolean setWritable(boolean writable, boolean ownerOnly)//设置所有者或每个人对此抽象路径名的写权限
public boolean setReadable(boolean readable)	// 设置此抽象路径名的所有者或每个人的读权限。
public boolean setReadable(boolean readable, boolean ownerOnly)//设置所有者或所有人对此抽象路径名的读取权限
public boolean setExecutable(boolean executable)// 设置此抽象路径名的所有者或每个人的执行权限
public boolean setExecutable(boolean executable, boolean ownerOnly)//设置所有者或每个人对此抽象路径名的执行权限
```

```java
public String[] list()						// 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录
public String[] list(FilenameFilter filter)
    								//返回一个字符串数组,这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录
public File[] listFiles()			//返回一个File对象数组,这些对象表示此抽象路径名表示的目录中的文件和目录(最常用)
public File[] listFiles(FilenameFilter filter)//返回File对象数组,表示此抽象路径名表示的目录中满足指定过滤器的文件和目录
public File[] listFiles(FileFilter filter)	  //返回File对象数组,表示此抽象路径名表示的目录中满足指定过滤器的文件和目录
```

```java
public static File[] listRoots()		// 列出可用的文件系统根。(Windows返回 C:\, D:\ 等，Linux/macOS返回 /)
public static File createTempFile(String prefix,String suffix)
            					//在默认的临时文件目录中创建一个空文件,使用给定的前缀和后缀生成其名称
public static File createTempFile(String prefix,String suffix,File directory)
            					//在指定的目录中创建一个新的空文件,使用给定的前缀和后缀字符串生成其名称
```

```java
public Path     toPath()				// 返回从此抽象路径构造的java.nio.file.Path对象(非常重要，通往现代API的桥梁)
public URI 		toURI()					//构造一个表示此抽象路径名的 file: URI
    
public int      compareTo(File pathname)// 按字典顺序比较两个抽象路径名。
public boolean  equals(Object obj)		// 测试此抽象路径名与给定对象是否相等。
public int 		hashCode()				// 计算此抽象路径名的哈希码。
public String 	toString()				// 返回此抽象路径名的路径名字符串。其实就是 getPath() 的返回值。
```
