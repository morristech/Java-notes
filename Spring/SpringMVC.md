## 2.Spring MVC

### 1.基本原理

#### 1.Spring MVC的处理请求的流程：

![](http://sishuok.com/forum/upload/2012/7/14/529024df9d2b0d1e62d8054a86d866c9__1.JPG)

#### 2.Spring Web MVC架构

![](http://sishuok.com/forum/upload/2012/7/14/57ea9e7edeebd5ee2ec0cf27313c5fb6__2.JPG)

### 2.Hello实例

在web.xml中加入以下内容来注册DispatcherServlet：

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.mvc</url-pattern>
    </servlet-mapping>

注意：

1. 这里`url-pattern`标签定义了url的匹配模式，即所有以`.mvc`结尾的请求会被MVC处理；

在springmvc-servlet中加入以下内容。

    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!-- HandlerAdapter -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!-- ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean name="/hello.mvc" class="my.shouheng.mvc.HelloController"/>

注意：

1. 这里的servlet配置文件的命名规则是，在web.xml中定义的servlet-name的后面加上`-servlet.xml`。如果采用其他的Servlet名称，也必须在后面加入`-servlet.xml`。
2. `InternalResourceViewResolver`定义了jsp文件的前缀和后缀路径，查找jsp文件的时候会到指定的前缀路径中查找指定后缀的文件；
3. 这里定义了bean，它的name就是url路径的匹配模式，也就是`hello.mvc`结尾的请求会被HelloController处理。
4. BeanNameUrlHandlerMapping的映射规则是，将Bean的name与url映射，比如这里的url为`/hello.mvc`，那么就必须有一个对应的name为`/hello.mvc`的Bean。
5. SimpleControllerHandlerAdapter表示所有实现了`Controller`接口的Bean可以作为Spring Web MVC中的处理器。如果需要其他类型的处理器可以通过实现HadlerAdapter来解决。
6. HandlerMapping是用来将url映射到指定的Handler的（策略模式），而HandlerAdapter是用来对指定的Handler进行适配，以支持多种Handler（适配器模式）。这样DispatcherServlet就可以将指定的url调用指定的Handler进行处理。

定义如下控制器：

	public class HelloController implements Controller {
	
	    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
	        ModelAndView mv = new ModelAndView();
	        //添加模型数据 可以是任意的POJO对象
	        mv.addObject("message", "Hello World!");
	        //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
	        mv.setViewName("hello");
	        return mv;
	    }
	}

注意：

1. 这里通过setViewName方法指定jsp的名称，这里会到/WEB-INF/jsp/中查找hello.jsp

**回顾一下以上程序的整个执行过程：**

![](http://sishuok.com/forum/upload/2012/7/14/8b42eeaa9b2423b154944c651ed23667__3.JPG)

首先用户请求会传递到DispatcherServlet（前段控制器），它接收所有以*.mvc结尾的请求。然后，它根据BeanNameUriHandlerMapping将指定的url，即/hello.mvc，映射到HelloWorldController。然后，DispatcherServlet再将HelloWorldController传递给SimpleControllerHandlerAdapter，它作为一个适配器，在这里真正调用HelloWorldController的方法执行业务逻辑。执行完毕业务逻辑之后，将结果ModelAndView传递到DispatcherServler。这时DispatcherServler再将ModelAndView传递给视图解析InternalResourceViewResolver，后者根据setViewName时赋值的名称，得到JstlView对象。然后，DispatcherServler对视图进行渲染，得到渲染结果之后返回给用户。

可见，其实这里的HandlerMapping和HandlerAdapter和ViewResolver是可以通过配置来指定不同的实现策略的。这里就像是模板、策略和适配器设计模式的组合，将一些可以由用户定制的处理逻辑交给我们来实现，而整个处理流程由SpringMVC进行控制。

### 3.使用注解定义SpringMVC

#### 1.示例

首先定义一个控制器，这里使用@RequestMapping注解指定它映射到的路径，

	@Controller
	public class HelloController {
	
	    @RequestMapping(value = "/hello")
	    public String sayHello() throws Exception {
	        ModelAndView mv = new ModelAndView();
	        //添加模型数据 可以是任意的POJO对象
	        mv.addObject("message", "Hello World!");
	        //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
	        mv.setViewName("hello");
	        return "hello";
	    }
    }

在springmvc-servlet.xml中做如下定义：

    <context:component-scan base-package="my.shouheng.*"/>
    <context:annotation-config/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

注意这里使用了默认的HandlerMapping和Adapter。这里的包名会有影响。

浏览器输入http://localhost:8080/palm/web/hello可以达到相同的效果。

这里的@RequstMapping也可以被用在类上面，还可以指定多个映射路径，比如@RequestMapping({"/", "/homepage"})

#### 2.接受请求的输入

1.通过路径参数接受输入：

    @RequestMapping(value = "/hello")
    public String sayHello(@RequestParam(value = "num", defaultValue = "10") int num) throws Exception {
        System.out.println(num);
        return "hello";
    }

这里通过@RequestParam指定参数的名称，可以使用defaultValue指定默认参数。

使用的方式是：http://localhost:8080/palm/web/hello?num=22

另一种方式是:

    @RequestMapping(value = "/hi/{num}")
    public String sayHi(@PathVariable("num") int num) throws Exception {
        System.out.println(num);
        return "hello";
    }

它的使用方式是：http://localhost:8080/palm/web/hi/22



### 4.过滤器

首先定义Filter：

	public class TheFilter implements Filter{
	
	    public void init(FilterConfig filterConfig) throws ServletException {
	        System.out.println("init, filterConfig" + filterConfig);
	    }
	
	    public void doFilter(ServletRequest servletRequest,
	                         ServletResponse servletResponse,
	                         FilterChain filterChain) throws IOException, ServletException {
	        System.out.println("doFilter, servletRequest: " + servletRequest
	                + "\nservletResponse: " + servletResponse
	                + "\nfilterChain: " + filterChain);
	        // 必须调用这个方法，否则请求就要在这里被拦截了
	        filterChain.doFilter(servletRequest, servletResponse);
	    }
	
	    public void destroy() {
	        System.out.println("destroy");
	    }
	}

注意，如果想要请求继续执行就必须调用filterChain.doFilter(servletRequest, servletResponse)，它让其他的过滤器对请求继续进行处理。如果没有这句话的话，该请求就会在这里被拦截掉。

然后在XML中配置过滤器：

    <filter>
        <filter-name>TheFilter</filter-name>
        <filter-class>my.shouheng.mvc.TheFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>TheFilter</filter-name>
        <url-pattern>/web/*</url-pattern>
    </filter-mapping>





