
## 第五章 Servlet容器

在Tomcat中有4种不同类型的容器，分别对应4种不同的概念层次:

1. Engine： 表示整个Catalina Servlet引擎；
2. Host: 表示包含一个或多个Context容器的虚拟主机；
3. Context: 表示一个Web应用程序，一个Context可以有多个Wrapper；
4. Wrapper: 表示一个独立的servlet. 

它们分别对应4个接口，分别是Engine、Host、Context和Wrapper，它们都继承自Container接口。4个标准实现类分别是StandardEngine、StandardHost、StandardContext和StandardWrapper. 一个容器可以包含一个或多个子容器，这使用addChild()、removeChild()等方法来完成。但是，Wrapper是最底层的容器，它不能再继续添加子容器。容器还可以添加一些支持组件，比如载入器、记录器和管理器等。

###### 1.Wrapper容器

管道包含Servlet要调用的任务，用阀来表示具体要执行的任务，可以动态增删阀，阀的执行顺序是依次执行的，机制类似Servlet中的过滤器。

Tomcat在实现对阀的遍历的时候没采用硬编码的数组遍历的方式，它采用了另一种方式，本质上也是对数组的遍历：

首先，阀是实现了Valve接口的对象，管道是实现了Pipeline的对象，在管道内维护了一个Value类型的数组，用来记录所有的阀。管道Pipeline中定义了一个与Container中的invoke()方法签名相同的方法。当我们调用了某个容器的invoke()方法的时候，实际上被代理到管道中并使用invoke()方法来执行。

在对阀进行遍历的时候，Tomcat使用了一个ValueContext来完成，它包含了一个invokeNext()方法。下面是一个管道的遍历的逻辑实现：

	  public void invoke(Request request, Response response)
	    throws IOException, ServletException {
	    (new SimplePipelineValveContext()).invokeNext(request, response);
	  }

即在调用管道的invoke()方法之后会实例化一个SimplePipelineValveContext对象，并调用它的invokeNext()方法来继续执行。下面是该方法的实现：

	public void invokeNext(Request request, Response response)
      throws IOException, ServletException {
      int subscript = stage;
      stage = stage + 1;
      if (subscript < valves.length) {
        valves[subscript].invoke(request, response, this);
      } else if ((subscript == valves.length) && (basic != null)) {
        basic.invoke(request, response, this);
      } else {
        throw new ServletException("No valve");
      }
    }

注意这里的``valves[subscript].invoke(request, response, this)``一行，它将SimplePipelineValveContext对象作为参数传入到Value的invoke()方法中，那么在Value的invoke()方法中又做了什么呢：

	  public void invoke(Request request, Response response, ValveContext valveContext) throws IOException, ServletException {
		// ...
	    valveContext.invokeNext(request, response);
	  }

也就是这里又继续调用了SimplePipelineValveContext的invokeNext()方法。所以，又回到了invokeNext()方法来继续进行数组的遍历。当Value数组遍历完毕之后就执行基础阀，即上面的代码中的basic. 在ex05的中基础阀是SimpleWrapperValve(在SimpleWrapper的构造方法中传入)，它一样实现了Value接口，下面是它的invoke方法（主要代码）：

	  public void invoke(Request request, Response response, ValveContext valveContext) 
		throws IOException, ServletException {
	    SimpleWrapper wrapper = (SimpleWrapper) getContainer();
	    Servlet servlet = wrapper.allocate();
		servlet.service(hreq, hres);	   	
	  }

也就是说，我们将之前的加载Servlet的并调用servlet的service()的方法都放进了Value中进行实现。也就是说，在这里我们已经将Servlet的概念抽象成了管道中的一个阀，这样我们就可以对一个请求调用多个阀了。

还有一个问题，在上面的代码中，获取Servlet的时候，并不是像我们之前的那样使用类加载器直接加载而是使用了SimpleWrapper的allocate()方法。其实，在allocate()的内部也是使用了类加载器从文件系统中加载类的，只是在本代码中，又将类加载器从代码中抽取出来作为一个独立的类进行封装，这样就可以按照需求改变类加载器的方式。

###### 2.Context容器

上面的容器虽然实现了调用多个阀，但是没有办法根据请求的路径调用不同的Servlet，使用Context容器可以满足我们的这个要求，简单地说，Context实现该功能也并没有使用什么很高深的办法，仅仅是根据预定义的url模式和指定的Servlet映射关系，当获取到HTTP请求的时候根据获取的URL和映射关系来选择要调用的Servlet即可。

与上面不同的是，在这我们不将执行单一功能的Servlet作为一个基础阀，而是先根据url来尝试获取指定的Servlet，然后再用获取到的Servlet执行逻辑，以达到用不同的url匹配不同的Servlet的目的。

使用Context接口的一些方法可以很容易地实现我们上述要求的功能：

之前也提到过Wrapper是最底层的容器，其他容器可以增加一至多个容器，所以这里我们就使用它的这个特性来完成上述目的。

在代码ex05中，我们需要在启动的时候指定url匹配的模式:

	Wrapper wrapper1 = new SimpleWrapper();
    wrapper1.setName("Primitive");
    wrapper1.setServletClass("PrimitiveServlet");
	
	Wrapper wrapper2 = new SimpleWrapper();
    wrapper2.setName("Modern");
    wrapper2.setServletClass("ModernServlet");

    Context context = new SimpleContext();
    context.addChild(wrapper1);
    context.addChild(wrapper2);

在这里我们将定义的两个Wrapper传入SimpleContext中，在SimpleContext中使用一个名为children的HaspMap对它们进行缓存，键是name，值是servlet的类名。

然后，创建用于指定的协议的Mapper映射器：

    Mapper mapper = new SimpleContextMapper();
    mapper.setProtocol("http");
    context.addMapper(mapper);

在SimpleContext中使用了名为mappers的HashMap对映射器进行缓存，键是协议，值是映射器。

最后，是定义Url的匹配模式，
	
    context.addServletMapping("/Primitive", "Primitive");
    context.addServletMapping("/Modern", "Modern");

同样，在SimpleContext中使用了名为servletMappings的HashMap进行缓存，键是Url模式，值是映射器的名字。

所以，根据上面的映射关系，实现根据请求调用不同的映射器的逻辑就是：先根据请求获取到指定协议的映射器，然后根据url的模式得到Wrapper的名称，再使用指定的映射器来查找该Wrapper容器，最后调用得到的容器的invoke()方法即可。

大致的思路大致是这个样子，本质上很多的查找的操作都是调用了Context里面的方法来完成的，因为毕竟是它内部维护了上述映射关系。所以，在读代码的时候你就可以看到Mapper和Context来回调用，好在逻辑并不算复杂。
