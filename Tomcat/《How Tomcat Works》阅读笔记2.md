## 第三章 连接器

连接器和容器是Catalina的两个重要的模块，连接器连接了请求、响应和Servlet等，因此它是很重要的组成部分。

###### 1.StringManager
在Tomcat中用于记录错误消息的类是StringManager，它根据传入的包名将错误信息存储在指定的路径下面。错误信息的文件名后缀是.properties，它通过在文件名的后面增加_ja等来支持多语言。它使用的单例的工厂方法，并在其内部使用报名作为键对指定包的StringManager进行缓存。

###### 2.连接器的类

在第三章中作者介绍了简单的连接器实现，并在第四章中介绍了Tomcat的默认的连接器实现。这里的类的类型和第四章中的类型大致相同。它可以大致分成下面的5种类型：

1. 连接器及其支持类(HttpConnector和HttpProcessor);
2. 表示HTTP请求的类(HttpRequest)及其支持类;
3. 表示HTTP响应的类(HttpResponse)及其支持类;
4. 外观类，对请求和响应的类进行一层封装;
5. 常量类，用于存储一些配置信息. 

###### 3.程序的执行过程

1. 首先，启动类是Bootstrp类，它具有一个main方法，在其内部实例化一个HttpConnector类，并调用后者的start()方法来启动连接器。HttpConnector实现了Runnable接口并将自身作为参数传入到线程Thread中，然后在start()方法中启动线程。所以，start()方法被调用之后就立即执行了HttpConnector的run()方法。
2. 在HttpConnector中实例化一个ServerSocket，并在循环中对请求进行监听。获取到请求的Socket之后会实例化一个HttpProcessor，并将获取到的Socket作为参数传入它的process()方法。所以，这里其实对每个请求都创建了一个HttpProcessor实例。
3. 在HttpProcessor的process()方法中先从传入的Socket上面获取输入流和输出流。这里将输入流作为参数传入给了一个实例化的SocketInputStream对象，并将得到的实例化SocketInputStream和输出流分别传给实例化的HttpRequest和HttpResponse对象。这里的SocketInputStream继承了InputStream，它直接对输入流进行读取并提供了解析请求的方法。
4. 依然在HttpProcessor的process()方法中,在获取了输入和输出流之后，仍然在该类中，使用parseRequest()和parseHeader()对请求的参数进行解析并将获取到的结果传递给HttpRequest。HttpRequest还提供了一系列的getter()方法用来获取这些解析的结果。
5. 接下来的逻辑就和之前差不多了——实例化一个ServletProcessor并将上面得到的请求和响应对象作为参数传入。在这里我们调用了一个新的MordenServlet，它从传入的请求上获取请求的参数并组装成HTML格式传送给客户端。

###### 另外，

1. 这里的大部分代码都与解析请求的逻辑相关，在本章中包含了解析请求行、请求头、Cookie和参数等的方法，在第四章又包含了一些针对Http1.1的请求的解析方法，这些方法大都依赖于该协议的格式。请求行是指在传入的uri之后的可选参数，当用户使用POST提交请求的时候，请求体可能会有参数。
2. 这里使用了HttpRequestLine来封装请求行，使用HttpHeader来封装请求头。
3. 与第二章不同，这里在输出的时候获取的用于输入的输出流的方式是

		 public PrintWriter getWriter() throws IOException {
		    ResponseStream newStream = new ResponseStream(this);
		    newStream.setCommit(false);
		    OutputStreamWriter osr = 
				new OutputStreamWriter(newStream, getCharacterEncoding());
		    writer = new ResponseWriter(osr);
		    return writer;
		  }

	它成功地解决了在第二章中出现的两句话不同都输出的问题。

## 第四章 Tomcat的默认连接器

Tomcat中使用的默认连接器需要满足的要求有：

1. 实现了Connector接口；
2. 负责创建实现了Request接口的request对象；
3. 负责创建实现了Response接口的response对象；

这里的连接器的工作与第三章类似：等待HTTP请求，创建request和response对象，然后调用Container接口的invoke()方法，将request和response对象传给servler容器。Container接口的invoke()方法的逻辑和之前的ServletProcessor的逻辑类似——使用类加载器根据请求的路径到指定的路径中获取Class实例，创建它的实例并将request和response对象传入它的serice()方法。

###### 1.Connector接口

Tomcat的连接器必须实现Connector接口，该接口中声明的众多方法中比较重要的几个有：1).getContainer()、2).setContainer()、3).createRequest()、4).createResponse(). 前面两个方法与Servler容器相关，后面的两个方法分别用于创建HTTP的请求对象和响应对象。

###### 2.HttpConnector类

无疑的，Tomcat的默认连接器的实现比第三章的简单的连接器要复杂的多，它在很多地方对性能进行了优化，尤其是对线程的使用上，因此这里也可以作为学习线程的使用的一个不错的例子。（毕竟，相比于线程教学中的一些示例程序，这里的现实意义更多一些。）

1. 这里的HttpConnector类除了实现了Runnable接口和Connector接口，还实现了Lifecycle接口，这是一个与生命周期相关的接口，会在随后对其进行介绍。现在只要知道在实例化一个HttpConnector之后就调用initialize()和start()方法即可；
2. 在initialize()中使用了一个单例的DefaultServerSocketFactory实例来获取ServerSocket对象。其实，在DefaultServerSocketFactory内部并没有太多复杂的逻辑，仅仅是直接根据情况使用new关键字创建一个ServerSocket对象而已，那么将其作为一个工厂有什么好处呢？这里我觉得将其作为工厂的好处是，以后如果想要修改获取实例的逻辑只要对这个工厂进行修改即可，而其他部分代码是不用动的，所以这就是“封装”的好处吧。
3. 在start()方法中，与在第三章的逻辑相似，先实例化一个Thread并将当前对象作为参数传入并调用线程的start()方法，只是这里将该线程设置成了守护线程（有什么好处呢？）。在启动了线程之后，根据要求的HttpProcessor的最小数量，创建了指定数量的HttpProcessor，并它们压入栈processors中，接下来就开始执行run()方法中的逻辑。
4. 在run()方法中仍然使用一个循环来监听来自客户端的请求。在获取到了请求之后会尝试从processors栈中获取一个HttpProcessor用来处理请求。如果栈中没有可用的HttpProcessor就会去创建新的HttpProcessor。但是如果能够创建的HttpProcessor已经达到了最大数量就不会再进行创建，而该HTTP请求也会被忽略。如果想要创建不限数量的HttpProcessor，可以将maxProcessors置为负数。新创建的HttpProcessor在创建完毕之后会立即调用它的start()方法（HttpProcessor也实现了Lifecycle方法）并将其放进created中。created是一个Vector类型的容器，用来记录和管理所有创建的HttpProcessor。
5. 对一个HTTP请求，如果获取到了HttpProcessor，那么就将Socket赋值给它的assign()方法。之前说在创建了新的HttpProcessor之后会立即调用它的start()方法，本质上HttpProcessor也实现了Runnable和Lifecycle，所以在start()方法中的操作其实就是开启了一个“处理器线程”。
6. 开启了线程之后该“处理器线程”就会在循环中调用await()来视图获取一个Socket。在await()方法中使用一个循环，并用!available变量作为循环的条件，在该循环内部调用wait()让线程处于等待的状态。在调用了HttpProcessor的assign()方法之后会将socket赋值给HttpProcessor中的socket，将available置为true，然后调用notifyAll()，并唤醒在await()上面的等待线程。此时，available为true，如果此时assign()方法又被调用，那么会在assign()陷入等状态待。在await()方法中，会将socket返回，将avaliable置为false，并调用notifyAll()唤醒线程。这样做调用await()的时候又会被阻塞，而调用assign()的地方又可以进行正常操作。总之，await()和assign()就是在交替运行。
7. 在HttpProcessor的run()方法中获取到了Socket之后会使用process()方法对其近似处理，处理完毕之后就会调用HttpConnector的recycle()方法将该HttpProcessor重新压入栈中，这样该HttpProcessor就又可以被再次使用了。

###### 另外，

1. HttpProcessor的await()和assign()方法的主要部分

	    synchronized void assign(Socket socket) {
	        while (available) {
	            try {
	                wait();
	            } catch (InterruptedException e) {}
	        }
	        this.socket = socket;
	        available = true;
	        notifyAll();
	    }

	    private synchronized Socket await() {
	        while (!available) {
	            try {
	                wait();
	            } catch (InterruptedException e) {}
	        }
	        Socket socket = this.socket;
	        available = false;
	        notifyAll();
	        return (socket);
	    }

	HttpProcerssor是在HttpConnector的栈中进行管理的。弹出的、还没有被再次压入栈中的HttpProcessor在process()完毕之前不会将HttpProcessor重新压入栈中，所以不会被连续调用两次assign()方法。那么这里的assign()和await()之间的状态就简单多了。这里值得学习的地方是进行阻塞和唤醒的方式，先使用一个循环将调用进行阻塞，然后在其他事件处理完毕之后唤醒阻塞线程继续执行其他逻辑，从而达到两个方法相互交替的执行的目的。

2. 关于创建HttpProcessor的逻辑

	    private HttpProcessor createProcessor() {
	        synchronized (processors) {
	            if (processors.size() > 0) {
	                return ((HttpProcessor) processors.pop());
	            }
	            if ((maxProcessors > 0) && (curProcessors < maxProcessors)) {
	                return (newProcessor());
	            } else {
	                if (maxProcessors < 0) {
	                    return (newProcessor());
	                } else {
	                    return (null);
	                }
	            }
	        }
	    }

	这里的逻辑是：先试着从栈中弹出一个HttpProcessor，如果没可用的(栈为空)，就根据可创建的
	HttpProcessor的最大数量来决定是否创建新的HttpProcessor。如果最大数量为负数就可以创建无限数量的HttpProcessor，如果不为负数就创建指定最大数量的HttpProcessor，达到了最大数量之后就不再继续创建，而是返回null，此时达到的HTTP请求也会被忽略。

3. 关于工厂方法和将执行某种逻辑的方法封装的好处

	在我个人写代码的时候只会将那种明确的会被多个地方调用的逻辑独立成一个方法，在Tomcat的代码中有很多地方将不会被多次调用的逻辑也封装成了一个方法。此外，还将创建HttpProcessor的逻辑封装成了一个工厂方法。笔者认为这样的好处在于：将执行某种逻辑的代码独立出来，当需要修改某个逻辑的时候只要修改这个方法即可，而不用到代码中去到处寻找要修改的地方。

4. 经过修改之后的连接器变成了多线程的

	即可以创建指定数量的HttpProcessor,每个HttpProcessor拥有一个独立的线程，可以达到复用和多线程执行的目的。






