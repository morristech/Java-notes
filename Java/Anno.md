# 注解

## 1、基本

自定义注解，下面定义了一个名为UseCase的注解：

	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
	public @interface UseCase {
        public int id();
        public String description() default "default value";
	}

这是一个普通的注解的定义。从上面可以看出，在定义注解的时候，

1. 除了使用`@interface`声明，并且指定名称之外，我们还需要使用一些元注解。
2. 定义注解类似于接口中的方法的定义；
3. 可以通过default为指定的元素指定一个默认值，如果用户没有为其指定值，就使用默认值；
4. Java中提供了三种标准注解：`@Override`, `@Deprecated`, `@SuppressWarnnings`；
5. Java中提供了四种元注解：`@Target`, `@Retention`, `@Documented`, `@Inherited`.

## 2、元注解

### 1. @Target

1. 用来定义注解该应用于什么地方，是方法还是字段等等。
2. `@Target`注解的定义是：

        @Documented
        @Retention(RetentionPolicy.RUNTIME)
        @Target(ElementType.ANNOTATION_TYPE)
        public @interface Target {
            ElementType[] value();
        }

    可以看出这里内部使用的是一个ElementType类型的数组来保存定义的属性的。而在使用的时候它的表现形式是

        @Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})

3. `@Target`注解的数组的每个元素的类型是`ElementType`。它的取值和具体的含义如下：

|编号|取值|含义|
|:-:|:-:|:-:|
|1|TYPE|类、接口或者enum声明|
|2|FIELD|域声明，包括enum实例|
|3|METHOD|方法声明|
|4|PARAMETER|参数声明|
|5|CONSTRUCTOR|构造器声明|
|6|LOCAL_VARIABLE|局部变量声明|
|7|ANNOTATION_TYPE|注解声明|
|8|PACKAGE|包声明|
|9|TYPE_PARAMETER|类型参数声明|
|10|TYPE_USE|Use of a type|

### 2. @Retention

1. 用来定义该注解在哪一个级别可用，比如在源代码中、类文件中或者运行时等。
2. 它的定义是
	
        @Documented
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.ANNOTATION_TYPE)
		public @interface Retention {
		    RetentionPolicy value();
		}

3. 这里指定value元素的类型是`RetentionPolicy`，而它的取值是：

|编号|取值|含义|
|:-:|:-:|:-:|
|1|SOURCE|注解将被编译器丢弃|
|2|CLASS|注解在class文件中使用，但会被JVM丢弃|
|3|RUNTIME|VM将在运行期保留注解，故可以通过反射读取注解的信息|

### 3. @Documented

1. 将此注解包含在javadoc中

### 4. @Inherited

允许子类继承父类的注解

## 3、注解的使用

以下通过一个小程序来演示注解的基本使用方法：

这里我们为程序定义了两个注解：应用于字段的`@Column`注解和应用于方法`@Important`注解。

	@Target(value = {ElementType.FIELD})
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Column {
	    String name();
	}

	@Target(value = {ElementType.METHOD})
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Important { }

然后我们定义了一个Person类，并使用注解为其中的部分方法和字段添加注解：

    private static class Person {
	
        @Column(name = "age")
        private int age;

        @Column(name = "first_name")
        private String firstName;

        @Column(name = "last_name")
        private String lastName;

        private int temp;

        public int getAge() {
            return age;
        }

        @Important
        public void setAge(int age) {
            this.age = age;
        }

        public String getFirstName() {
            return firstName;
        }

        @Important
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }

        public String getLastName() {
            return lastName;
        }

        @Important
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
    }

然后，我们使用Person类来获取该类的字段和方法的信息，并输出具有注解的部分：

    public static void main(String ...args) {
        Class<?> c = Person.class;
        Method[] methods = c.getDeclaredMethods();
        for (Method method : methods) {
            if (method.getAnnotation(Important.class) != null) {
                System.out.println(method.getName());
            }
        }
        Field[] fields = c.getDeclaredFields();
        for (Field field : fields) {
            if (field.getAnnotation(Column.class) != null) {
                System.out.println(field.getName());
            }
        }
    }

输出结果：
	
	setAge
	setFirstName
	setLastName
	age
	firstName
	lastName

在上面的代码，我们通过为字段和方法添加自定义注解，然后再用反射判断该方法和字段是否用了自定义注解并获取注解的信息。这在很多时候非常有用，比如Spring等框架中就定义了大量的注解。