## 第十章 安全性

Tomcat的安全校验机制并不复杂，它是在之前所了解的管道的基础之上，将校验的逻辑作为一个阀来在管道中执行的。Servlet支持通过web.xml配置访问权限。在深入了解Tomcat的安全机制之前有几个概念和相关的类、接口需要介绍一下。

##### 1、相关的类和接口

###### 1.领域Realm

它保存了所有有效用户的用户名和密码，并管理用户的用户名和密码等信息，它还有一些列的操作这些数据的方法，用来校验用户信息和权限等。存储用户信息的位置有多种选择，默认存储在tomcat-user.xml中。Realm的基本实现形式是RealmBase，它有JDBCRealm、JNDIRealm、MemoryRealm和UserDatabaseRealm等，分别对应不同的存储类型。Realm接口中有几个重要的方法：

1. poublic Principal authenticate(....): 它有四个重载的方法，用来根据存储的用户信息对传入的用户信息进行校验，当校验成功的时候会返回一个Principal对象；
2. haseRole()： 用于校验用户的角色信息；
3. setContainer()/getContainer(): 用于将Realm和Context相关联. 

领域通常与一个Context相关联，一个Context容器也只能有一个关联的Realm对象。

###### 2.主体对象Principal

主主体对象是Principal接口的实例，它的一个实现是GenericPrincipal，GenericPrincipal必须与一个Realm相关联。GenerialPrincipal有一个

1. Realm realm
2. String name
3. String password
4. List roles

四个重要的字段，而它的构造方法也是围绕着这四个字段进行的。因此，可以将其作为一个存储了用户信息的对象。

###### 3.登录配置LoginConfig类

LoginConfig是登录配置类，它封装了要使用的身份校验的方法，领域对象、登录页面的URI和错误页面的URI，它的一个构造方法是：

	/**
     * @param authMethod The authentication method
     * @param realmName The realm name
     * @param loginPage The login page URI
     * @param errorPage The error page URI
     */
    public LoginConfig(String authMethod, String realmName, String loginPage, String errorPage){ ... }

这里的authMethod对应不同的验证器。

###### 4.验证器Authenticator

验证器是Authenticator接口的实例，它起到标记作用，这样就可以使用instanceof检查某个组件是否为验证器。Authenticator的基本实现是AuthenticatorBase，它除了实现了Authenticator，还实现了ValveBase，这说明它是一个阀，而我们也确实将其作为一个阀来使用的。在Tomcat中有多种不同类型的验证器，分别用于不同的身份验证方式，比如基于表单的身份验证、SSL信息验证等等。

##### 2、程序的执行过程

###### 1.校验器的安装

上面也提到过我们进行权限的校验的时候是将其检验的逻辑作为一个阀来进行的，所以需要将阀放进管道中。在本程序中我们使用SimpleContextConfig监听事件的变化，并在开始的时候将阀装入到管道中：

	public class SimpleContextConfig implements LifecycleListener {
	
	  private Context context;

	  public void lifecycleEvent(LifecycleEvent event) {
	    if (Lifecycle.START_EVENT.equals(event.getType())) {
	      context = (Context) event.getLifecycle();
	      authenticatorConfig();
	      context.setConfigured(true);
	    }
	  }

这里监听事件并在开始START_EVENT的时候，使用authenticatorConfig()方法将身份校验的阀放进管道中。

	  private synchronized void authenticatorConfig() {
	    SecurityConstraint constraints[] = context.findConstraints();
	    if ((constraints == null) || (constraints.length == 0)) return;
	    LoginConfig loginConfig = context.getLoginConfig();
	    if (loginConfig == null) {
	      loginConfig = new LoginConfig("NONE", null, null, null);
	      context.setLoginConfig(loginConfig);
	    }
	
		// Context的容器是通过管道代理来实现的
	    Pipeline pipeline = ((StandardContext) context).getPipeline();
	    if (pipeline != null) {
	      Valve basic = pipeline.getBasic();
	      if ((basic != null) && (basic instanceof Authenticator))
	        return;
	      Valve valves[] = pipeline.getValves();
	      for (int i = 0; i < valves.length; i++) {
	        if (valves[i] instanceof Authenticator) return;
	      }
	    } else {
	      return;
	    }
	
上面的代码的意思是不要重复地添加安全校验的阀，因为一个Context只能有一个校验器。

	    if (context.getRealm() == null) return;
	
	    String authenticatorName = "org.apache.catalina.authenticator.BasicAuthenticator";
	    Valve authenticator = null;
	    try {
	      Class authenticatorClass = Class.forName(authenticatorName);
	      authenticator = (Valve) authenticatorClass.newInstance();
	      ((StandardContext) context).addValve(authenticator);
	    } catch (Throwable t) { }
	  }
	}
	
从上面的代码中可以看出，我们在决定是不是要将一个校验的阀装载到管道中是根据Context中的配置来决定的，即Context实例的那一系列的getter方法。最终，确实要进行校验的时候使用BasicAuthenticator的实例作为一个阀来进行身份校验。

###### 2.Bootstrep中的配置

上面提到过，在安装校验器的时候我们使用了Context实例的那一系列的getter方法，那么这些getter是在哪里进行配置的呢？Bootstrep. 下面是它的main方法中的一系列逻辑：

    Context context = new StandardContext();
    // ...context的一些基本的配置，路径，子容器，加载器，映射器等逻辑
	
	// 上面提到的生命事件监听器，在这里决定是否应该安装校验器
    LifecycleListener listener = new SimpleContextConfig();
    ((Lifecycle) context).addLifecycleListener(listener);

    // addPattern指明要遵循验证规则的URI的模式，addMethod指定哪种提交的方法要使用安全校验
	// 这里的意思是：使用GET提交的任意URI的方法都会进行安全校验
    SecurityCollection securityCollection = new SecurityCollection();
    securityCollection.addPattern("/");
    securityCollection.addMethod("GET");

	// 创建安全限制对象
    SecurityConstraint constraint = new SecurityConstraint();
    constraint.addCollection(securityCollection);
	// 指定了能够访问资源的用户的角色
    constraint.addAuthRole("manager");
    context.addConstraint(constraint);

	// 登录配置类
    LoginConfig loginConfig = new LoginConfig();
    loginConfig.setRealmName("Simple Realm");
    context.setLoginConfig(loginConfig);

    // 添加领域，使用了自定义的SimpleRealm
    Realm realm = new SimpleRealm();
    context.setRealm(realm);

####### 3.身份校验的逻辑

上面提到的内容都是用来对身份校验进行的配置，下面是如何执行身份校验。在上面提到过是将BasicAuthenticator作为一个阀来进行身份校验的，那么必定在它的invoke()方法中进行了身份校验的逻辑。实际也正是如此：在这里的身份校验中，Tomcat使用了模板设计模式，在AuthenticatorBase的invoke中制订了一些校验的模板（主要是对一些必需的信息进行校验），真正从Realm中进行校验的逻辑处于继承了AuthenticatorBase的BasicAuthenticator中。它通过调用Realm的authenticate方法进行身份校验：

    public boolean authenticate(HttpRequest request, HttpResponse response, LoginConfig config) throws IOException {

        Principal principal = ((HttpServletRequest) request.getRequest()).getUserPrincipal();
        if (principal != null) {
            return (true);
        }

        HttpServletRequest hreq = (HttpServletRequest) request.getRequest();
        HttpServletResponse hres = (HttpServletResponse) response.getResponse();
        String authorization = request.getAuthorization();
        String username = parseUsername(authorization);
        String password = parsePassword(authorization);
		// 在这里使用Realm进行身份校验，它通过调用Realm的authenticate()方法进行
        principal = context.getRealm().authenticate(username, password);
        if (principal != null) {
            register(request, response, principal, Constants.BASIC_METHOD, username, password);
            return (true);
        }

        String realmName = config.getRealmName();
        if (realmName == null)
            realmName = hreq.getServerName() + ":" + hreq.getServerPort();
        hres.setHeader("WWW-Authenticate", "Basic realm=\"" + realmName + "\"");
        hres.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
		
        return (false);
    }

所以，这里就将身份校验的逻辑和Realm关联了起来。

###### 4.关于Realm

在SimpleRealm中使用了不太优雅的硬编码的方式定义了一些用户信息，它们被存储在SimpleRealm的users字段中，后者是一个list容器。然后，authenticate方法的定义如下：

	  public Principal authenticate(String username, String credentials) {
	    if (username==null || credentials==null)
	      return null;
	    User user = getUser(username, credentials);
	    if (user==null)
	      return null;
	    return new GenericPrincipal(this, user.username, user.password, user.getRoles());
	  }

	  // 获取指定用户名和密码的用户信息，获取不到就意味着校验失败
	  private User getUser(String username, String password) {
	    Iterator iterator = users.iterator();
	    while (iterator.hasNext()) {
	      User user = (User) iterator.next();
	      if (user.username.equals(username) && user.password.equals(password))
	        return user;
	    }
	    return null;
	  }
	
      // 硬编码的方式创建用户的信息，数据被存放在users容器中
	  private void createUserDatabase() {
	    User user1 = new User("ken", "blackcomb");
	    user1.addRole("manager");
	    user1.addRole("programmer");
	    User user2 = new User("cindy", "bamboo");
	    user2.addRole("programmer");
	
	    users.add(user1);
	    users.add(user2);
	  }
	
	  // 用户对象
	  class User {
		public String username;
	    public ArrayList roles = new ArrayList();
	    public String password;

	    public User(String username, String password) {
	      this.username = username;
	      this.password = password;
	    }
		
		// ...
	  }

以上就是安全校验的主要逻辑。
