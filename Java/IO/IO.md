# Java IO

## 1、Java IO 体系

![Java IO 体系](/Java/res/io_frame.png)

### 1.1 概述

抽象类InputStream和OutputStream构成了输入/输出(IO)类层次的结构基础。
因为Java中使用的Unicode都是双字节的，所以抽象类Reader和Writer就是设计了的专门用于处理Unicode字符的单独的类层次结构。
即InputStream和OutputStream是基于单字节的，而Reader和Writer是基于双字节的。

当完成对流的读写，应该通过close关闭它，这将释放有限的操作系统资源。
关闭输出流还会冲刷用于该输出流的缓冲区：所有被临时置于缓冲区中，以便使用更大的包的形式传递的字符在关闭输出流时都将被送出。
如果不关闭文件，那么写出字节的最后一个包可能永远得不到传递。
此外，也可以使用flush()方法人为地冲刷输出。

直接对数据源进行操作的流叫做节点流，其他流可以在它们的基础之上进行一层包装来实现自己的逻辑。
那些不是直接对数据源进行操作的流就是非节点流。

### 1.2 装饰者模式

Java 的IO系统的设计使用了**装饰者**设计，注意这里的**FilterInputStream**，它的定义是：

    public class FilterInputStream extends InputStream {

        protected volatile InputStream in;

        protected FilterInputStream(InputStream in) {
            this.in = in;
        }

        public int read() throws IOException {
            return in.read();
        }

        // ...其他方法，它内部还定义了一些其他的方法，这里我们只给出一部分
    }

从这里也可以看出，它内部维护了一个InputStream对象，并在构造方法对其进行赋值，然后将具体的方法的实现都在内部交给它来执行。

#### 从装饰者模式理解IO体系

理解了这个设计原理，我们就会容易得理解上面这张图的框架的设计的层次。InputStream是一个抽象的基类，它的直接子类中除了上面提到的FilterInputStream，都对应于不同的输入类型。FilterInputStream是一个抽象的装饰器，它的子类分别代表着不同的功能。而FilterInputStream的作用则是进行装饰，意在对来自不同的输入类型的结果进行一层包装。

#### 使用装饰者模式的好处是

用户可以使用装饰器组合来实现流操作的复杂功能，某些流（节点流）能够从文件或其他位置获取字节，而其他流可以将字节组装成有用的数据类型，也就是责任是分开的。

## 2、字节流的读写

### 2.1 二进制文件的读写

FileInputStream和FileOutputStream提供了附着在磁盘文件的输入流和输出流，只要像其构造器传入文件名或者文件路径即可。

#### 二进制文件的读取 FileInputStream

    FileInputStream(...)    通过打开一个到实际文件的连接来创建一个 FileInputStream，

关于文件分隔符的说明：
反斜杠`\`是转义字符，因此在写文件路径的时候要使用`\\`或者`/`（实际的文件路径只有一个反斜杠）。但是，为了使程序便于移植最好使用常量字符串`java.io.File.separator`来获取文件分隔符。

#### 二进制文件的写入 FileOutputStream

其构造方法如下，

    FileOutputStream(File file) 					 
    FileOutputStream(File file, boolean append)  	

关于append：如果append为true，那么数据将被添加到文件尾，而具有相同名字的文件不会被删除；否则，该方法会删除所有具有相同名字的已有文件。 

#### 示例程序

    public static void main(String ...args) {
        File file = new File("." + File.separator + "io_test.txt");
        try {
            boolean succeed = file.exists() ? (file.delete() && file.createNewFile()) : file.createNewFile();
            // output
            OutputStream fos = new FileOutputStream(file);
            String str = "字节流的写入和读取的测试字符串";
            fos.write(str.getBytes());
            // input
            byte[] bytes = new byte[1024];
            InputStream fis = new FileInputStream(file);
            int ret = fis.read(bytes);
            System.out.println(new String(bytes));
            System.out.println(ret);
            fos.close();
            fis.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

输出结果：

	字节流的写入和读取的测试字符串
    45

### 2.2 基本数据类型的读写

#### 数据输入流 DataIntputStream

DataInputStream不是节点流，无法从磁盘中直接读取字节，它只有一个构造方法，需要传入节点流

    DataInputStream(InputStream in) 

该类具有一些列的读取数据类型的方法

    byte readByte()   
    char readChar()  
    // ...其他的数据类型的读取

#### 数据输出流  DataOutputStream类

DataOutputStream非节点流，需要通过其他节点创建，也只有一个构造方法

    DataOutputStream(OutputStream out) 

它也有一些列的写出数据类型的方法

    void writeByte(int v)      将一个 byte 值以 1-byte 值形式写出到基础输出流中。 
    void writeBytes(String s)  将字符串按字节顺序写出到基础输出流中。 
    void writeChar(int v)      将一个 char 值以 2-byte 值形式写入基础输出流中，

先写入高字节。 

    void writeChars(String s)  将字符串按字符顺序写入基础输出流。 
    void writeDouble(double v)
    // ...其他的数据类型写入

#### 示例程序

    public static void main(String ...args) {
        File file = new File("." + File.separator + "data_io.txt");
        Person person = new Person("Shhng", "Wng", 20, true, 10000);
        try {
            boolean succeed = file.exists() ? (file.delete() && file.createNewFile()) : file.createNewFile();
            OutputStream os = new FileOutputStream(file);
            DataOutputStream dos = new DataOutputStream(os);
            dos.writeUTF(person.firstName);
            dos.writeUTF(person.lastName);
            dos.writeInt(person.age);
            dos.writeBoolean(person.male);
            dos.writeDouble(person.wage);
            InputStream is = new FileInputStream(file);
            DataInputStream dis = new DataInputStream(is);
            Person ps = new Person(
                    dis.readUTF(),
                    dis.readUTF(),
                    dis.readInt(),
                    dis.readBoolean(),
                    dis.readDouble());
            System.out.println(ps);
            os.flush();
            os.close();
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static class Person {
        String firstName;
        String lastName;
        int age;
        boolean male;
        double wage;

        Person(String firstName, String lastName, int age, boolean male, double wage) {
            this.firstName = firstName;
            this.lastName = lastName;
            this.age = age;
            this.male = male;
            this.wage = wage;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "firstName='" + firstName + '\'' +
                    ", lastName='" + lastName + '\'' +
                    ", age=" + age +
                    ", male=" + male +
                    ", wage=" + wage +
                    '}';
        }
    }

输出结果：

    Person{firstName='Shhng', lastName='Wng', age=20, male=true, wage=10000.0}

说明：如果文件中不包含指定的数据类型的话，就会弹出异常. 输出的类型的顺序要和输入的顺序严格匹配，不然可能会有读写错误。

### 2.3 使用字节缓冲流提高读写效率

缓冲流不是节点流，需要为其指定一个节点流，它的作用是创建一个缓冲区，从而不必每次读入字符时都访问设备。一般将缓冲流放在节点流和另一种流之间，前者用于访问设备数据，后者提供获取各种数据类型的方法。

#### 字节缓冲输入流 BufferedInputStream

创建带缓冲区的流，带缓冲区的流在从流中读入字符使，不会每次都对设备访问，当缓冲区为空时，会向缓冲区读入一个新的的数据块。

    BufferedInputStream(InputStream in， int size) 

#### 字节缓冲输出流 BufferedOutputStream

创建一个带缓冲的流，带缓冲区的输出流在收集要写出的字符时，不会每次都对设备访问，当缓冲区填满或者流被冲刷时，数据被写出。

    BufferedOutputStream(OutputStream out, int size)

#### 示例程序

    public static void main(String ...args) {
        File fileToCopy = new File("." + File.separator + "who will succeed.mp4");
        File fileCopied = new File("." + File.separator + "who will succeed(copy).mp4");
        try {
            boolean succeed = fileCopied.exists() ? (fileCopied.delete() && fileCopied.createNewFile()) : fileCopied.createNewFile();
            if (!succeed) System.out.println("error to delete or create");
            copyWithBuffer(fileToCopy, fileCopied);
            boolean succeed2 = fileCopied.exists() ? (fileCopied.delete() && fileCopied.createNewFile()) : fileCopied.createNewFile();
            if (!succeed2) System.out.println("error to delete or create");
            copy(fileToCopy, fileCopied);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void copy(File fileToCopy, File fileCopied) throws IOException {
        long st = System.currentTimeMillis();
        InputStream is = new FileInputStream(fileToCopy);
        OutputStream os = new FileOutputStream(fileCopied);
        int b;
        while ((b = is.read()) != -1) {
            os.write(b);
        }
        os.flush();
        os.close();
        is.close();
        System.out.println(System.currentTimeMillis() - st);
    }

    private static void copyWithBuffer(File fileToCopy, File fileCopied) throws IOException {
        long st = System.currentTimeMillis();
        InputStream is = new FileInputStream(fileToCopy);
        OutputStream os = new FileOutputStream(fileCopied);
        BufferedInputStream bis = new BufferedInputStream(is, 15 * 1024);
        BufferedOutputStream bos = new BufferedOutputStream(os, 15 * 1024);
        int b;
        while ((b = bis.read()) != -1) {
            bos.write(b);
        }
        bos.flush();
        bos.close();
        bis.close();
        os.close();
        is.close();
        System.out.println(System.currentTimeMillis() - st);
    }

输出结果：

	564
    64606

## 3、字符流的读写

### ３.1 概述

以上代码的输出都是以字节输出的二进制，通过记事本打开的都是“乱码”. 所以，Java提供了Unicode字符流的写入和读取的类，用于对文本类型进行读写.

### 3.2 字符数据的读写

FileWriter和FileReader类使用Window默认字符编码GB 2112的字符输入输出. 若要指定文本编码方式可以使用InputStreamReader类和OutputStreamWriter类. 这两种流都不是节点流，需要向其中传入节点流创建对象，所以，其机制大致是先使用节点流从设备读取字符，然后使用指定字符集编码。注意这里传入的节点流是字节流，InputSteam和OutputSteam及其子类，而不是FileReader和FileWriter.

#### 字符输出流 InputStreamReader

显然该流也不是节点流，需要向其传入一个节点流，其构造方法

    InputStreamReader(InputStream in)                       使用默认字符集
    InputStreamReader(InputStream in, Charset cs) 
    InputStreamReader(InputStream in, String charsetName) 

该类有用于读取字符的方法: `read()`

#### 字符输出流 OutputStreamWriter

需要向其传入一个节点流，其构造方法

    OutputStreamWriter(OutputStream out)                    使用默认字符编码
    OutputStreamWriter(OutputStream out, Charset cs) 		
    OutputStreamWriter(OutputStream out, String charsetName) 

写入数据到字符输出流: `write(int c)`

### 3.3 使用字符缓冲提高读写速度

#### 字符缓冲输出流 BufferedWriter

使用了装饰器模式，用来对传入的Writer进行一层封装。其构造方法是：`BufferedWriter(Writer out)`

#### 字符缓冲输入流 BufferedReader

同理，其构造方法`BufferedReader(Reader in)`

### 3.2 文本文件的读写

这里的FileReader和FileWriter两个类分别继承了OutputStreamReader和OutputStreamWriter. 相对于它们的父类而言，除了构造方法，它们没有增加任何新的方法。

它跟父类的区别仅在于，它们在构造方法中需要传入文件相关的信息，然后直接使用这些信息创建了FileOutputStream和FileInputStream，并将创建的实例传递给父类的构造方法。

#### 文本文件的读取 FileReader

它继承了OutputStreamReader，其构造方法为：`FileReader(...)`. 

然后可以使用该类提供的`read()`方法进行单个字符的读取 	

#### 文本文件的写入 FileWriter

与FileReader类似，实际上内部使用了FileOutputStream来获取输出流的。

该类提供了写出字符的方法: `write(int c)`

#### 示例程序

    public static void main(String ...args) {
        File file = new File("." + File.separator + "data_io.txt");
        try {
            boolean succeed = file.exists() ? (file.delete() && file.createNewFile()) : file.createNewFile();
            StringBuilder sb = new StringBuilder();
            for (int i=0;i<100000;i++) {
                sb.append((char) i);
            }
            writeToFile(file, sb.toString());
            readFile(file);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void readFile(File file) throws IOException {
        Reader reader = new FileReader(file);
        BufferedReader br = new BufferedReader(reader, 10 * 1024);
        String s ;
        while ((s = br.readLine()) != null) {
            System.out.println(s);
        }
        reader.close();
        br.close();
    }

    private static void writeToFile(File file, String s) throws IOException {
        Writer writer = new FileWriter(file);
        BufferedWriter bw = new BufferedWriter(writer);
        bw.write(s);
        bw.flush();
        bw.close();
        writer.close();
    }

输出结果：

    惨不忍睹

### 3.5 文本的读写

#### 文本的写出

对文本输出，可以使用PrintWriter，该类具有以文本格式打印字符串和数字的方法，
该类的构造方法比较多，根据传入的文件方式，大致可以将其分成下面四种主要类型，其他方法只是在这四种方式的基础上，规定了编码方式和是否启用自动刷新

    PrintWriter(File file) 
    PrintWriter(OutputStream out) 
    PrintWriter(String fileName) 
    PrintWriter(Writer out) 

该类还有一系列的输出数据类型的方法，比如

    print(基本数据类型 var)
    println(基本数据类型 var)
    write(String s) 
    write(int c) 
    print(Object obj) 

可以用它来打印数字、字符、boolean值、字符串和对象。

#### 文本的读取

在Java SE5之前，处理文本输入的唯一方式是BufferReader类，它拥有readLine方法，可以读入一行文本。但是它没有任何用于读入数字的方法，而现在可以使用Scanner来读如文本输入。

#### 示例程序

	package ioprojects06;
	import ......
	public class Prog06 {
		private final static String seperator = java.io.File.separator;
		private final static String Parent = 
				"C:" + seperator + "Users" + seperator + "wangshouheng" 
	              + seperator + "Documents" + seperator + "JavaIO" + seperator;
		private final static String fileName = "文本输入输出.txt";
		public static void main(String[] args) throws IOException {
			testPrintWriter();
			testBufferReader();
			testScanner();
		}
		private static void testPrintWriter() throws FileNotFoundException{
			PrintWriter pWriter = new PrintWriter(Parent + fileName);
			pWriter.println("四大名著");
			pWriter.println(32.24);
			pWriter.println(true);
			pWriter.println(pWriter);
			pWriter.flush();  pWriter.close();
		}
		private static void testScanner() throws FileNotFoundException{
			Scanner scan = new Scanner(new File(Parent + fileName));
			System.out.print(scan.nextLine() + " ");
			System.out.print(scan.nextDouble() + " ");
			System.out.print(scan.nextBoolean() + " ");
			System.out.print(scan.next() + " ");
			scan.close();
		}
		private static void testBufferReader() throws IOException{
			BufferedReader bReader = new BufferedReader(new FileReader(Parent + fileName));
			String line;
			while((line = bReader.readLine())!=null)  System.out.print(line + " ");
			System.out.println();
		}
	}

输出结果：

    四大名著 32.24 true java.io.PrintWriter@499a12ee 
    四大名著 32.24 true java.io.PrintWriter@499a12ee 

## 4、随机文件的访问

### 4.1 概述

随机文件访问用于，比如MP3播放的快进功能，跳到指定位置进行播放. 

### 4.2 创建RandomAccessFile对象

其构造方法如下

    RandomAccessFile(File file, String mode) 
    RandomAccessFile(String name, String mode)

说明：关于mode的值与含义

    "r"           以只读方式打开。  
    "rw"          打开以便读取和写入。  
    "rws"         打开以便读取和写入，对于 "rw"，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备。  
    "rwd"         打开以便读取和写入，对于 "rw"，还要求对文件内容的每个更新都同步写入到底层存储设备。  

对于写出，该类提供了以下几个重要的方法：

    void write(int b)  
    void writeBoolean(boolean v)    除此之外还有其他的基本数据类型

对于读取，该类提供了以下几个重要的方法：

    int read() 
    boolean readBoolean()      还有其他基本数据类型的读取方法
    String readLine() 	

对于定位，该类提供了以下几个重要的方法：

    long getFilePointer()       返回此文件中的当前偏移量。 
    long length()               返回此文件的长度。 
    void seek(long pos)         设置到此文件开头测量到的文件指针偏移量，在该位置发生下一个读取或写入操作。 

#### 示例程序

	package ioprojects07;
	import ......
	public class Prog07 {
		private final static String seperator = java.io.File.separator;
		private final static String Parent = 
				"C:" + seperator + "Users" + seperator + "wangshouheng" 
	              + seperator + "Documents" + seperator + "JavaIO" + seperator;
		private final static String fileName = "随机文本.txt";
		public static void main(String[] args) throws IOException {
			outRandFile();
			inRandFile();
		}
		private static void outRandFile() throws IOException{
			RandomAccessFile randOut = new RandomAccessFile(Parent + fileName, "rw");
			for(int i=1;i<16;i++)  randOut.writeInt(i);
			randOut.close();
		}
		private static void inRandFile() throws IOException{
			RandomAccessFile randIn = new RandomAccessFile(Parent + fileName, "r");
			int RECORD_SIZE = 4; //每个记录的长度，字节
			long len = randIn.length(); //文件的总长度，字节
			int n = (int)(len / RECORD_SIZE); //记录的总数
			for(int i=n-1;i>=0;i--){ //从最后一个往前输出每个记录
				randIn.seek(RECORD_SIZE*i);  readData(randIn);
			}
	         randIn.close();
		}
		private static void readData(RandomAccessFile in) throws IOException{
			int d = in.readInt();  System.out.print(d + " ");
		}
	}

输出结果：

    15 14 13 12 11 10 9 8 7 6 5 4 3 2 1  

说明：可见上面的示例程序按照与写入相反的顺序读取了写入到文件当中的内容。主要是先获取文件的长度，然后根据每个记录的长度求得记录的总数，最后按照从后往前的顺序读取了每条记录。

## 5、读取ZIP文档

每个ZIP文档都有一个头，包含诸如文件名字和所使用的压缩方法等信息，在Java中，可以使用ZipInputStream和ZipOutputStream来读写ZIP文档。可能需要浏览单独的项，getNextEntry方法就可以返回一个描述项的ZipEntry类型对象。ZipInputStream的read方法在碰到当前项的结尾时返回-1（不是Zip文件结尾），然后你应该使用closeEntry来读入下一项。

典型的通读Zip文件的代码是

    ZipInputStream zin = new ZipInputStream(new FileInputStream(zipname));
    ZipEntry entry;
    while((entry=zin.getNextEntry())!=null){
        analyze entry
        read content of zin
        zin.closeEntry();
    }
    zin.close();

当然也可以使用其他的读入方式，比如

    Scanner in = new Scanner(zin);
    while(in.hasNextLine())  do something with in.nextLine

注意：读取单个Zip项之后不要关闭Zip输入流，否则将无法读取后续的项。
写出Zip文档时要用ZipOutputStream，对要放入到Zip文件的每一项，都应该创建一个ZipEntry对象，将文件名传递给ZipEntry构造器，它将设置诸如日期、解压方式等信息。然后，调用ZipOutputStream的putNextEntry方法写出新文件，并将文件发送到ZIP流中。框架是:

	FileOutputStream fout = new FileOutputStream(zipname);
	ZipOutputStream zout = new ZipOutputStream(fout);
	for all files{
	    ZipEntry ze = new ZipEntry(filename);
	    zout.putNextEntry(ze);
	    send data to zout
	    zout.closeEntry();
	}
	zout.close();

Jar是一种带有特殊项的ZIP文件，可以使用JarInputStream和JarOutputStream来写清单项。

#### 示例程序

	package ioprojects08;
	import ......
	public class Prog08 {
		private final static String seperator = java.io.File.separator;
		private final static String Parent = 
				"C:" + seperator + "Users" + seperator + "wangshouheng" 
				+ seperator + "Documents" + seperator + "JavaIO" + seperator;
		private final static String fileName = "压缩文件.zip";
		private final static Charset charset = Charset.forName("GBK"); //创建字符集
		public static void main(String[] args) throws IOException {
			writeZip();
			readZip();
		}
		private static void readZip() throws IOException{  //读取ZIP文档
			ZipInputStream zin = new ZipInputStream(new FileInputStream(Parent + fileName), charset); //ZIP读入流
			//ZipInputStream zin = new ZipInputStream(new FileInputStream(Parent + fileName));
			ZipEntry entry;
			while((entry = zin.getNextEntry()) != null){  //获取ZIP的项
				Scanner scan = new Scanner(zin);  //使用Scanner进行读取
				while(scan.hasNextLine())  System.out.print(scan.nextLine());  //输出读取的字符串
				System.out.println();
				zin.closeEntry();  //关闭项
			}
			zin.close();  //关闭流
		}
		private static void writeZip() throws IOException{  //写出ZIP文档
			ZipOutputStream zout = new ZipOutputStream(new FileOutputStream(Parent + fileName), charset);  //ZIP写出流
			String[] files = new String[]{
					"观棋柯烂，伐木丁丁，云边谷口徐行。"
					+ "卖薪沽酒，狂笑自陶情。苍径秋高，对月枕松根，一觉天明。"
					+ "认旧林，登崖过岭，持斧断枯藤。收来成一担，行歌市上，易米三升。"
					+ "更无些子争竞，时价平平。不会机谋巧算，没荣辱，恬淡延生。"
					+ "相逢处，非仙即道，静坐讲《黄庭》。",
					"Later, respectively, wander and suffer sorrow."};
			int i=1;
			for(String str : files){  //对每个字符串创建一个文件，名字是 文件1 ...
				ZipEntry ze = new ZipEntry("文件"+i+++".txt");
				zout.putNextEntry(ze);
				zout.write(str.getBytes());  //获取字符串的字节并输出
				//PrintWriter pWriter = new PrintWriter(zout);  
				//pWriter.write(str);
				zout.closeEntry();
			}
			zout.close();  //关闭流
		}
	}

输出结果：

    观棋柯烂，伐木丁丁，云边谷口徐行。卖薪沽酒，狂笑自陶情。苍径秋高，对月枕松根，一觉天明。认旧林，登崖过岭，持斧断枯藤。收来成一担，行歌市上，易米三升。更无些子争竞，时价平平。不会机谋巧算，没荣辱，恬淡延生。相逢处，非仙即道，静坐讲《黄庭》。
    Later, respectively, wander and suffer sorrow.

说明：如果在使用ZipInputStream和ZipOutputStream的时候不指定字符集，将出现以下错误

	Exception in thread "main" java.lang.IllegalArgumentException: MALFORMED
		at java.util.zip.ZipCoder.toString(ZipCoder.java:58)
		at java.util.zip.ZipInputStream.readLOC(ZipInputStream.java:297)
		at java.util.zip.ZipInputStream.getNextEntry(ZipInputStream.java:121)
		at ioprojects08.Prog08.readZIP(Prog08.java:28)
		at ioprojects08.Prog08.main(Prog08.java:21)

## 6、对象流域序列化

java支持对象序列化机制，它将任意对象写到流中，并在之后将其读回。为保存对象，需要先打开一个ObjectOutputStream对象。可以直接使用该对象的writeObject方法。为将数据读回，需要使用ObjectInputStream，它有一个readObject方法可以将数据读回。

但是要想在对象流中恢复和存储的类都应该实现Serializable接口，该接口中没有任何方法，实现了即可。

#### 程序示例

	package ioprojects09;
	import ......
	public class Prog09 {
		private final static String seperator = java.io.File.separator;
		private final static String Parent = 
				"C:" + seperator + "Users" + seperator + "wangshouheng" 
				+ seperator + "Documents" + seperator + "JavaIO" + seperator;
		private final static String fileName = "对象数据.dat";
		public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
			//写出对象
	         ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(Parent + fileName));
			Product cup = new Product("茶杯", 10, 20.0d, new Shape(Shape.ROUND));
			Product pen = new Product("笔", 15, 3.0d, new Shape(Shape.RECTANGLE));
			out.writeObject(cup);
			out.writeObject(pen);
	         //读取对象
			ObjectInputStream in = new ObjectInputStream(new FileInputStream(Parent + fileName));
			Product p1 = (Product) in.readObject();
			Product p2 = (Product) in.readObject();
	         //输出对象信息
			System.out.println("名字:" + p1.getName() + ";数量:" + p1.getNum() + ";价格:" + p1.getPrice() + ";形状=" + p1.getShape().getShape());
			System.out.println("名字:" + p2.getName() + ";数量:" + p2.getNum() + ";价格:" + p2.getPrice() + ";形状=" + p2.getShape().getShape());
		}
		//自定义类：产品
		private static class Product implements Serializable{
			private String name;
			private int num;
			private double price;
			private Shape shape;  //自定义类：产品形状
			public Product(String name, int num, double price, Shape shape){
				this.name = name;
				this.num = num;
				this.price = price;
				this.shape = shape;
			}
			public String getName(){  return name;  }
			public int getNum(){  return num;  }
			public double getPrice(){  return price;  }
			public Shape getShape(){  return shape;  }
		}
		//自定义类：形状
		private static class Shape implements Serializable{
			private String shape;
			public static final String ROUND = "ROUND";
			public static final String RECTANGLE = "RECTANGLE";
			public Shape(String shape){  this.shape = shape;  }
			public String getShape(){  return shape;  }
		}
	}

输出结果：

    名字:茶杯;数量:10;价格:20.0;形状=ROUND
    名字:笔;数量:15;价格:3.0;形状=RECTANGLE

说明：可以看出当被序列化写出的对象中包含引用类型时候无需对引用类型进行独立地调用writeObject，而且该引用类型会被自动地映射到指定的对象中去。但是被引用的类型必须也是实现了Serializable接口的。另外，当两个类共享一个引用类型的时候也无需对其独立地调用writeObject，它会被自动地映射。

## 7、磁盘、目录和文件的基本操作

### 7.1 Path类

Path是一个接口，表示的是目录名序列，也可以跟一个文件名。通常使用类Paths的静态方法来获取Path对象。这里简要地给出这两个类的一些比较重要的方法

Paths类

    static Path get(String first, String ..more) 	通过连接指定的字符串创建一个路径

Path类
   
    Path getParent();                   获取父路径
    Path getFileName();                 后去该路径最后的一个部件
    Path getRoot()                      获取根部件
    toFile()                            创建File对象

而File类中也有方法Path toPath()可以用来获取一个Path对象

#### 示例程序

	package ioprojects10;
	import ......
	public class Prog10 {
		public static void main(String[] args) {
			Path p = Paths.get("C:", "users", "wangshouheng" ,"Documents", "JavaIO", "对象数据.dat");
			System.out.println(p.getParent());
			System.out.println(p.getFileName());
			System.out.println(p.getRoot());
		}
	}

输出结果：

    C:\users\wangshouheng\Documents\JavaIO
    对象数据.dat
    C:\

### 7.2 Files类

#### 读写文件

该类提供了一些便捷的操作文件的方法。

1. 读取文件所有内容: `byte[] bytes = Files.readAllBytes(path) `然后可以将字节输入转换成字符串: `String content = new String(bytes, charSet)`
2. 将文件按行读入: `Files.readAllLines(path, charSet)`
3. 将字符串写到文件中: `Files.write(path, content.getBytes(charset))`
4. 向指定文件追加内容: `Files.write(path, content.getBytes(charset), StandardOpenOption.APPEND);` 
5. 直接打开输入流和输出流：`newInputStream(...)`和`newOutputStream()`

#### 复制、移动和删除文件

下面的这些方法适用于处理中等长度的文本文件，如果文件比较大，或者是二进制文件，尽量使用前面的那些流：

1. 将一个文件从一个位置移动到另一个位置，可以使用：`Files.copy(...)`
2. 移动文件（复制并删除原文件）：`Files.move(...)`
3. 如果目标路径存在该文件，那么复制或移动将会失败，若要覆盖已有的目标路径，可以使用REPLACE_EXISTING，如`Files.copy(...)`
4. 删除文件：`Files.delete(...)`, `Files.deleteIfExists(...) `

#### 创建文件和目录

1. 创建新目录

    1. `Files.createDirectory(...)`	   除了最后一个部件外，其他部分都必须是已存在的
    2. `Files.createDirectories(...)`  与上面的相比，它可以创建中间路径，没有上面的限制
    3. `Files.createFile(...)`	       创建空文件

2. 创建临时目录和文件的方法: `Files.createTempDirectory(...)`或`Files.createTempFile(...)`

#### 获取文件信息

1. 返回文件的字节数: `Files.size(...)`
2. 略，它还有很多静态的边界方法，简单地传入文件路径就可以获取到文件的信息

### 7.4 File类概述

#### File类构造方法

使用File的构造方法获取File实例，我们可以通过传入文件路径、URI和父文件并指定一个文件名的方式创建。（文件路径的分隔与系统有关，可以使用File.seprator和File.sepratorChar代替分隔符）

#### 文件和目录的基本操作

1. 构建表示文件路径的File对象:`new File(...)`
2. 判断目录/文件是否存在:`exists()`
3. 删除目录/文件:`delete()/deleteOnExit()`
4. 创建目录:`mkdir()/mkdirs()`
5. 创建新文件:`createNewFile()`
6. 重命名/移动目录/文件:`renameTo(File dist)`
7. 获取目录中的内容
	1. `listFiles(...)`:返回目录中的文件和目录的File对象数组
	2. `list(...)`:返回目录中文件和目录的字符串数组
8. 获取父目录
	1. `getParentFile()`:返回父目录的File对象
	2. `getPatent()`	
9. 创建临时文件:`createTempFile(...)`
10. 获取和设置目录/文件的属性

说明：

1. 在File.list方法中，有的需要我们传入一个FilenameFilter实例，这是一个接口。它使用了策略模式——我们只需要实现自己的接口方法，并在其中加入文件名称的匹配逻辑就可以对文件进行过滤。

## 8、Preferences

Preferences被用来存储一些小型的键值对信息，通常用里存储一些用户配置信息：

    public static void main(String...args) {
        Preferences prefer = Preferences.userNodeForPackage(PreferIOTest.class);
        // output
        prefer.put(Keys.USER_NAME.name(), "Wng");
        prefer.putLong(Keys.USER_PHONE.name(), 1582212198);
        prefer.putBoolean(Keys.IS_MALE.name(), true);
        prefer.putDouble(Keys.WAGE.name(), 45.5);
        // input
        System.out.println(prefer.getBoolean(Keys.IS_MALE.name(), true));
        System.out.println(prefer.getDouble(Keys.WAGE.name(), 0));
        System.out.println(prefer.getLong(Keys.USER_PHONE.name(), 0));
    }

    private enum Keys {
        USER_NAME,
        USER_PHONE,
        IS_MALE,
        WAGE;
    }

输出：

    true
    45.5
    1582212198
