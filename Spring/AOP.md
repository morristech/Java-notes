## AOP

### 使用XML

除了Spring包之外还要加入

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.2</version>
        </dependency>

切面类，实现了MethodBeforeAdvice和AfterReturningAdvice两个接口：

	public class LogAOP implements MethodBeforeAdvice, AfterReturningAdvice, ThrowsAdvice{
	
	    public void before(Method method, Object[] objects, Object o) throws Throwable {
	        System.out.println("before");
	    }
	
	    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
	        System.out.println("afterReturning");
	    }
	
        public void afterThrowing(Object[] args, ServletException e, Method method, Object target){
            System.out.println("afterThrowing" );
        }
	
	    public void after(JoinPoint joinPoint) {
	        System.out.println("after");
	    }
	
	    public void around(ProceedingJoinPoint pjp) throws Throwable{
	        long current = System.currentTimeMillis();
	        pjp.proceed();
	        System.out.println("around cost:" + (System.currentTimeMillis() - current));
	    }
	}

定义的类:

	public class MyBean {
	
	    public void say() {
	        System.out.println("say");
	    }
	
	    public void sayHello() {
	        System.out.println("Hllo");
	    }
	}

XML配置文件：

    <bean name="myBean" class="my.shouheng.aop.MyBean"/>

    <bean id="log" class="my.shouheng.aop.LogAOP"/>

    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(public * say*(..))"/>
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:aspect ref="log">
            <aop:after method="after" pointcut-ref="pointcut"/>
            <aop:around method="around" pointcut-ref="pointcut"/>
            <!--<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut"/>-->
        </aop:aspect>
    </aop:config>

1. 这里pointcut标签中定义了切点的id和规则expression，即public的任何返回类型的以say开头的任何输入参数的方法会被当作切点；
2. 这里的advisor标签指定了切点要执行的逻辑，这里是LogAOP类，以及针对的切点。
3. 所以，简单来理解就是“pointcut定义了插入的规则，advice定义了切入处要执行的逻辑，advisor将pointcut和advice联系起来”

输出结果：

	before method:say
	say
	afterReturning method:say
	afterReturning cost:73
	before method:sayHello
	Hllo
	afterReturning method:sayHello
	afterReturning cost:0

### 切入点指示符

#### 1.类签名表达式`Whithin(<type name>)`

1. within(my.palm..*)：匹配my.palm及其子包中所有类的所有方法
2. within(my.palm.ClassName):匹配my.palm.ClassName类中所有方法
3. within(MyInterface+)：匹配MyInterface接口所有实现类的所有方法
4. within(my.palm.BaseClass)：匹配my.palm.BaseClass及其子类的所有方法

#### 2.方法签名`execution(<scope> <return type> <full-qulified-class-name>.<method>(params))`

1. execution(* my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有方法
2. execution(public * my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有公共方法
3. execution(public * my.shouhengn.Palm.*(long,..)):my.shouhengn.Palm中所有第一个参数为long型的公共方法
4. execution(public String my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有返回类型为String的公共方法

#### 3.其他

1. bean(*Service):所有后缀名为Service的Bean
2. @anotation(my.shouheng.Annotation):所有使用Annotaion注解的方法会被调用
3. @within(my.shouheng.Annotation):所有使用Annotaion注解的类的方法会被调用
4. this(my.shouheng.MyInterface):所有实现了my.shouheng.MyInterface接口的代理对象的方法会被调用

#### 4.通配符

1. `..`:任意参数列表，任何数量的包
2. `+`:给定类的任何子类
3. `*`:任何数量的字符

### 使用注解的方式

声明切面：

	@Aspect
	public class LogAOP {
	
	    @Before(value = "execution(public * say*(..))")
	    public void before() throws Throwable {
	        System.out.println("before");
	    }
	
	    @AfterReturning(value = "execution(public * say*(..))")
	    public void afterReturning() throws Throwable {
	        System.out.println("afterReturning");
	    }
	
	    @AfterThrowing("execution(public * say*(..))")
	    public void afterThrowing(){
	        System.out.println("afterThrowing" );
	    }
	
	    @After("execution(public * say*(..))")
	    public void after(JoinPoint joinPoint) {
	        System.out.println("after");
	    }
	
	    @Around("execution(public * say*(..))")
	    public void around(ProceedingJoinPoint pjp) throws Throwable{
	        long current = System.currentTimeMillis();
	        pjp.proceed();
	        System.out.println("around cost:" + (System.currentTimeMillis() - current));
	    }
	}

除了在每个方法上面定义切面的切入点规则，也可以显得定义一个方法在上面使用@PointCur注解指定切入规则。然后，在各个注解中调用该切入点即可。

使用切面的类：

	public class MyBean {
	
	    public void say() {
	        System.out.println("say");
	    }
	
	    public void sayHello() {
	        System.out.println("Hllo");
	    }
	
	    public void sayWithException() {
	        System.out.println("sayWithException");
	        throw new IllegalArgumentException("Exception");
	    }
	}

配置文件：

    <bean name="myBean" class="my.shouheng.aop.MyBean"/>
    <bean name="log" class="my.shouheng.aop.LogAOP"/>
    <aop:aspectj-autoproxy/>

测试类：

    public static void main(String ...args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("/XmlBeanContext.xml");
        MyBean myBean = ctx.getBean("myBean", MyBean.class);
        myBean.say();
        myBean.sayWithException();
    }

输出结果：

	before
	Exception in thread "main" java.lang.IllegalArgumentException: Exception
		......
	say
	around cost:32
	after
	afterReturning
	before
	sayWithException
	after
	afterThrowing

#### 当指定的方法中具有参数的时候

比如如下方法，这里具有一个int类型的参数count，那么我们怎么在切面的方法中获取到从count方法中传入的参数呢？

    public void count(int count) {  }
	
	@Aspect
	public class LogAOP {
	
	    @Before(value = "execution(public * count(int)) && args(count)")
	    public void before(int count) throws Throwable {
	        System.out.println("before " + count);
	    }
   }

这里是在切面中获取切点的一个示例——我们需要在切点的定义中指定args参数，然后在其中指定参数的名称。这里，当我们调用count方法的时候，就会调用方法之前先调用这里的方法输出count的值。




