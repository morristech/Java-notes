
## 1. 依赖注入

### 1.基于XML的配置方式

下面是基于XML配置的一个xml文件：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd 
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

	    <bean id="component" class="org.restlet.ext.spring.SpringComponent">
            <!--要求component内部必须具有setDefaultTarget方法，不一定非得是名为defaultTarget的字段-->
	        <property name="defaultTarget" ref="restServer"/>
	    </bean>
	
	</beans>

说明：

1. 需要将`<bean>`标签定义在`<beans>`标签内部；
2. 使用基于XML注解的时候是不需要`<context:component-scan base-package="..."/>`标签的；
3. 如果想要为某个Bean中的字段注入值，可以在指定的`<bean>`标签内部使用`<property>`标签。需要注意的地方是，使用基于XML的注入的方式的时候，`<property>`标签中的**name是与指定字段的setter方法相关**的，而不是与字段名称相关的，并且该Bean内部**必须具有指定的Setter方法**。而当使用@Autowired的时候是不需要改Bean中的指定字段的Setter方法的，使用@Autowired的时候是根据Bean的类型来进行注入的（使用@Autowired的时候无法为非Bean字段注入）。
4. 其他值类型的注入方法：
		
		<property name="title" value="RoseIndia"/>
		<property name="authorsInfo"> 
		<map>
		<entry key="1" value="Deepak" />
		<entry key="2" value="Arun"/>
		<entry key="3" value="Vijay" />
		</map>
		</property>
 
    也是使用`<property>`标签，如果是基本数据类型，使用vlaue进行赋值；而`<map>`、`<set>`、`<list>`及`<array>`，则分别用来为Collection容器类型和数组类型赋值。

5. 除了使用Setter方法注入，还可以使用构造函数注入，要用到`<constructor-arg>`标签，规则和使用Setter方法注入相似；
6. 使用构造方法注入时，参数列表中的顺序是不重要的，这意味着参数列表(A a, B b)和(B b, A a)的调用方式是一样的，如果在一个类内部存在这样两个构造函数就会存在歧义。此时，可以在每个`<constructor-arg>`标签中加入一个index，用来指定指定参数在参数列表中的位置。像这样：

        <constructor-arg ref="A" index=0>
        <constructor-arg ref="B" index=1>

7. 使用构造函数注入的另一个问题是**循环依赖**，即A和B，A在构造方法中引用了B，B在构造方法中引用了A，那么A实例化的时候需要B，B实例化时需要A，就会抛出异常。解决方法是杜绝循环依赖，尽量使用Setter注入方式。

### 2.基于注解的配置方式

	@Service
	public class PalmSpringJaxObjectFactory implements ObjectFactory {
	
	    @Autowired
	    private PalmJaxBeanCollection palmJaxBeanCollection;
	
	    public <T> T getInstance(Class<T> aClass) throws InstantiateException {
	        return palmJaxBeanCollection.getBean(aClass);
	    }
	}

所需的xml配置文件

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	
	    <context:component-scan base-package="my.shouheng.palmcollege.facade.*"/>
	
	</beans>

说明：

1. 这里使用了`@Service`注解将`PalmSpringJaxObjectFactory`类声明为一个Bean；
2. 和`@Service`具有相同效果的还有`@Repository`和`@Controller`，他们继承自`@Component`，作用相似，只是针对不同的类的类型（当然也可能被用来根据注解识别类的类型）；
3. 基于注解的方式中需要指定要扫描的包，这通过使用`<context:component-scan beas-package="..."/>`标签来实现；
4. 上面也提到过注入值的时候，在基于注解的方式中注入可以使用@Autowired的，而且可以不为指定的字段定义Setter方法；
5. 对应与`<context:component-scan>`还有一个注解，名为`@ComponentScan`；
6. 使用@Component注解的时候，如果在其中指定了一个字符串，那么这个字符串是不会作为Bean的名称的，而且也不会将类的首字母变成小写来作为Bean的名称。

### 3.基于java的配置

以下是一个基于Java的配置方式：

	@Configuration
	public class Config {
	
	    @Bean
	    public BeanA beanA() {
	        BeanA beanA = new BeanA();
	        beanA.setB(beanB());
	        return beanA;
	    }
	
	    @Bean
	    public BeanB beanB() {
	        return new BeanB();
	    }
	}

1. 这里主要是调用指定的类的构造方法创建对象，并将值赋进去，看起来代码量比使用注解的方式要多；

### 3、其他

#### 1.Bean重写

当使用多个配置文件定义Bean的时候，如果不同的配置文件中存在相同的名称的Bean就会发生重写，规则是最后加载的Bean会覆盖先加载的Bean.

#### 2.depends-on特性

在`<bean>`标签内部或者使用@DependsOn注解，表示当前的Bean要在指定的Bean创建完毕之后才被创建。

#### 3.自动装配

可以在`<bean>`内部，通过`autowire`指定自动装配类型，可选的有byName和byType，分别表示根据名称和类型进行装配。也可以在@Bean注解中使用autowired指定装配类型。


自动装配只适用于Bean类型，值类型可以使用@Value注解来进行赋值。可以在@Value注解中使用占位符和SpEL表达式，并从中获取值。

#### 4.命名Bean

1. 可以在`<bean>`标签中使用id来为指定的Bean定义名称。
2. 可以通过在`<bean>`标签中使用name来为Bean指定名称，可以指定多个，中间用逗号、空格、分号分开、
3. 可以使用`<alias>`标签为已经有名字的Bean添加别名。
4. 可以在@Component注解的子注解中指定Bean的名称，如@Servce("asda")。
5. 可以在@Bean注解中使用name字段为指定Bean命名，可以在括号中指定多个名称。

#### 5.Bean实例化

1. 通过构造方法实例化。
2. 通过实例或者静态工厂方法实例化。
3. 可以通过实现FactoryBean<T>接口进行实例化。

#### 6.Bean作用域

通过在`<bean>`标签中使用scope指定或者使用@Scope注解：

1. singleton：单例的
2. prototype:每次创建一个实例，类似new操作
3. request
4. session
5. globalSession

#### 7.延迟初始化

1. 所谓延迟初始化就是在使用到指定Bean的时候才初始化，这是相对于在Spring容器启动阶段就初始化而言的。
2. 在`<beans>`标签中通过制定default-lazy-init="true"可以将内部全部bean延迟初始化；
3. 在`<bean>`标签中也可以指定某个Bean是否延迟初始化，在bean中定义的会覆盖在beans中定义的；
4. 可以使用@Lazy注解在代码中实现延迟初始化；
5. 延迟初始化的好处是加快了容器的启动时间，占用较少内存。缺点是若在元数据中存在Bean配置错误，在对方案测试之前无法被发现；
6. 如果一个依赖Bean被定义为预先初始化，那么延迟定义将没有作用。

#### 8.生命周期回调

1. 在`<bean>`标签中通过`init-method`和`destroy-method`指定某个函数为初始化方法和销毁方法。Bean创建完毕之后，init方法被调用；Bean生命周期结束之前会调用destroy方法。
2. 被指定的方法的方法名没有限制；
3. 可以使用@PostConstruct和@PreDestroy注解标记指定的方法来实现生命周期回调，要使用这种方式还要在XML中加入`<context:annnotation-config>`标签；
4. 实现生命周期的第三种选择是实现InitializingBean和DisposableBean接口。

#### 9.Bean定义配置文件

#### 10.环境

### 4、实例

#### 1.bean标签中的parent

    <bean id="securityContextInterceptor" abstract="true"
          class="com.oo.ejb3.interceptor.SecurityContextInterceptor">
        <property name="loginHistoryService" ref="loginHistoryService"/>
        <property name="sysAutoUpdateDAO" ref="sysAutoUpdateDAOImpl"/>
    </bean>

    <bean parent="securityContextInterceptor">
        <property name="businessInterface"
                  value="com.oo.ejb3.system.sessionbean.WSUserService"/>
        <property name="target" ref="userService"/>
    </bean>

有时候，我们会遇到像上面这种情况——在bean标签中使用了parent。这里和模板设计模式（或者策略）的理念相似，即使用parent指定的那个bean是一个模板的基类，在子类中我们会按照子类的要求为其中的字段赋不同的值，来得到一个新的Bean。

在被指定为parent的bean上面使用了abstract="true"，它指示该bean不被预先实例化。注意这并不要求该bean标签中的calss指定的类是抽象的。

一个子bean定义可以从父bean继承构造器参数值、属性值以及覆盖父bean的方法，并且可以有选择地增加新的值。如果指定了init-method，destroy-method和/或静态factory-method，它们就会覆盖父bean相应的设置。
