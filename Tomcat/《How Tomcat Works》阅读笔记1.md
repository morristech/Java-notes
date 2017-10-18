编译当前目下位于/src/ex01/pyrmont中的java文件的cmd指令是：

    javac -d . -classpath ./lib/commons-beanutils.jar;./lib/commons-collections.jar;./lib/commons-daemon.jar;./lib/commons-digester.jar;./lib/commons-logging.jar;./lib/commons-modeler.jar;./lib/jakarta-regexp-1.2.jar;./lib/mx4j.jar;./lib/naming-common.jar;./lib/naming-factory.jar;./lib/naming-resources.jar;./lib/servlet.jar;./lib/tomcat-util.jar; ./src/com/brainysoftware/pyrmont/util/*.java ./src/ex01/pyrmont/*.java

#### TODO 
1. 学习一下在编译的时候IDEA到底都为我们做了什么，以及一些常见的java操作的指令


## 第一章 一个简单的web服务器

#### 代码ex01的执行过程：
	
1. 首先，通过上面的编译的指令将.java文件编译成.class文件，在**HttpServer**中包含了一个mian方法，在它的内部对使用HttpServer创建服务端，并指定地址是127.0.0.1端口是8080，然后使用一个无限循环自动监听，直到获取了/SHUTDOWN指令才停止监听循环并推出；
2. 在循环监听中使用了两个类Request和Resonpe，前者用于解析获取到的请求，后者用于对请求进行响应；
3. **Requset**对从Socket中建立连接并获取到的输入流中读入的信息进行解析，也就是说我们发送的Http请求是从Socker中读取到的，然后将其作为字符串进行处理，在Request中处理Http请求的方式是使用index获取空格，这实际上是依赖于Http协议中的一些格式，总之就是简单的字符串处理；
4. 同样，**Response**是对从Socket中建立连接并获取到的输出流进行输出，也就是向客户端返回执行结果。这里具体的方式是根据Requset解析出url到指定的目录去寻找对应名称的文件，并将其作为字符流输出。

#### 另外，

1.在服务器端获取文件的时候使用了

	public static final String WEB_ROOT =
    System.getProperty("user.dir") + File.separator  + "webroot";

这里的**System.getProperty("user.dir")**是获取当前项目的根目录的意思。就是你在cmd中执行java指令时的工作路径。

2.上面的代码在Chrome中执行的时候是没有在浏览器中显示出预期的结果的，但是在Explore中达到了预期的效果。在Chrome里的请求报错 "CAUTION: Provisional headers are shown"，我们可以使用**chrome://net-internals**来查找被屏蔽的请求以及可能的原因。

3.执行程序的指令是
	
	java ex01.pyrmont.HttpServer

其中HttpServer是编译之后的HttpServer.class文件，并且包含了一个main方法。

4.程序编译的时候输出错误信息提示（这是因为代码的注释中加了中文导致的）：

	错误:编码gbk的不可映射字符

解决的方法是：1).在编译的时候指定字符编码集，如

	javac -encoding UTF-8HelloWorlewww.Java

2).记事本打开java源文件，另存为选择ANSI编码。

## 第二章 一个简单的servlet容器

要自定义一个servlet容器需要实现javax.servlet.Servlet接口，它包含了5个方法，其中最重要的三个与生命周期有关的方法是：init(), service(), destroy(). 其中，init()方法在实例化Servlet之后被调用；service()在客户端的请求达到时调用，它接收javax.servlet.ServletRequest和javax.servlet.Servlet.ServletResponse两个类型的参数，前者封装了HTTP的请求信息，后者封装了HTTP的响应信息；destroy()方法将在servlet实例被从服务中移除时调用。

#### 代码ex02的执行过程：
1. ex02的代码月ex01基本相同，这是在监听的循环中使用Uri的字符串进行判断，如果URI中包含了"servlet"字符串，那么我们就使用Servlet进行处理，否则仍然使用之前的逻辑；
2. 在这里仅对ex01中的Request和Response进行了封装，让它们分别实现了ServletRequest和ServletResponse接口，并在获取到的Servlet中将它们作为参数传入；
3. 因为这里的Servlet比较简单，在service()方法内我们并没有看到PrimitiveServlet究竟是如何使用的Request和Response；
4. 决定要使用哪个Servlet的逻辑大致是：首先从URI中解析出要调用的Servlet的类名，然后使用类加载器从文件系统中获取该类Class类型，然后使用它来获取该类的实例，最后将Request和Response作为参数传入，并调用service()方法即可。

#### 另外，

1.因为在Request和Response中除了包含了ServletRequest和ServletResponse接口中的方法，还包含了返回静态资源的方法。我们不希望这些方法被调用，但是又不能将其这些方法作为私有的。因此，这里又使用了外观类，主要是对这两个类进行了一层封装，也就是只将希望暴露给Servlet的方法暴露了出来。

2.不知道是不是我的书的问题，路径中的PrimitiveServlet的s应该是大写的……

3.这里执行代码的指令是

	java -classpath ./lib/servlet.jar;./ ex02.pyrmont.HttpServer1	

加入``-classpath ./lib/servlet.jar;./``的作用是：因为在本代码的执行时需要依赖一些其他的类，注意这里的``·/``是和``./lib/servlet.jar``一起的，并用``;``分隔开，用来指定多个路径，而不是``./ex02.pyrmont.HttpServer1``。

4.UML建模图还是蛮重要的。
