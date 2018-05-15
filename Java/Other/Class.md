# Class类

Class类包含了与类某个类有关的信息，Class也支持泛型。它们的效果是相同的，只是使用泛型具有编译器类型检查的效果，相对更加安全。

以下是使用Class类的一个测试例子。在这里我们可以通过反射来获取并修改private方法和private字段的accessible属性。当设置为true时，我们就可以对其进行调用或者修改。

此外，我们还可以进行自定义注解以及获取类、方法和字段的注解等信息。

### 示例1：获取Class对象的属性信息，修改private类型的字段，调用private的方法

    public static void main(String ...args) {
        Class<SubClass> subClass = SubClass.class;

        try {
            // 如果SubClass是private类型的，那么会抛出以下异常：
            // java.lang.IllegalAccessException: Class me.shouheng.rtti.RttiTest can not access a member of class
            // me.shouheng.rtti.RttiTest$SubClass with modifiers "private"
            SubClass sub = subClass.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

输出Class类的方法，这里输出的结果中包含了超类的方法：

        System.out.print("\nMethods:\n");
        System.out.println(Arrays.toString(subClass.getMethods()));

输出Class内部定义的方法，它只输出了在SubClass中新加入的方法的定义：

        System.out.print("\nDeclaredMethods:\n");
        System.out.println(Arrays.toString(subClass.getDeclaredMethods()));

        System.out.print("\nMethodInformation:\n");
        printMethodInfo(subClass);

输出Class的字段，输出的结果为空：

        System.out.print("\nFields:\n");
        System.out.println(Arrays.toString(subClass.getFields()));

输出Class中定义的字段：

        System.out.print("\nDeclaredFields:\n");
        System.out.println(Arrays.toString(subClass.getDeclaredFields()));

        System.out.print("\nFieldInformation:\n");
        printFieldInfo(subClass);

        System.out.print("\nClassInformation:\n");
        printClassInfo(subClass);
    }

    private static void printMethodInfo(Class<?> c) {
        SubClass sub = new SubClass();
        sub.setName("My Simple Name");
        // 打印方法信息
        Method[] methods = c.getDeclaredMethods();
        Method method = methods[0];
        System.out.println(method.getName());
        System.out.println(method.getReturnType());

输出方法的注解信息，对于能够输出的注解信息是有要求的：

        // 输出注解信息
        Annotation[] annotations = method.getDeclaredAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

可以在获取了方法的Method对象之后调用它的invoke方法进行触发：

        // 触发方法
        try {
            System.out.println(method.invoke(sub));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

修改方法的属性，这里我们是已经知道out方法是第二个方法的前提下进行的。还有直接使用Class对象的getMethod尝试使用方法名获取方法是不行的：

        try {
            // 设置out方法的accessible为true，这样可以从外部调用该方法
            method = methods[1];
            method.setAccessible(true);
            // 直接根据名称来获取out方法是不行的
    //      method = c.getMethod("out");
    //      method.setAccessible(true);
            method.invoke(sub);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    private static void printFieldInfo(Class<?> c) {
        SubClass sub = new SubClass();
        sub.setName("My Simple Name");
        Field[] fields = c.getDeclaredFields();
        Field field = fields[0];
        try {
            System.out.println(field.get(sub));
        } catch (IllegalAccessException e) {
            System.out.println(e);
        }
        Annotation[] annotations = field.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }
        System.out.println(field.isAccessible());
        System.out.println(field.getName());

我们可以通过修改一个字段的访问性来修改这个字段的属性：

        field.setAccessible(true);
        try {
            field.set(sub, "Shit");
            System.out.println(field.get(sub));
        } catch (IllegalAccessException e) {
            System.out.println(e);
        }
    }

    private static void printClassInfo(Class<?> c) {
        System.out.println(Arrays.toString(c.getAnnotations()));
        System.out.println(c.getCanonicalName());
        System.out.println(c.getClass());
        System.out.println(c.getGenericSuperclass());
        System.out.println(Arrays.toString(c.getGenericInterfaces()));
        System.out.println(Arrays.toString(c.getConstructors()));
        System.out.println(c.getPackage());
    }

    private static class SuperClass {}

    // private 类型的不行
    @Deprecated
    public static class SubClass extends SuperClass implements Interface {
        @me.shouheng.rtti.Field
        @NotNull
        private String name;
        private SuperClass sup;

        @Deprecated
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public SuperClass getSup() {
            return sup;
        }

        public void setSup(SuperClass sup) {
            this.sup = sup;
        }

        private void out() {
            System.out.println("private method in SubClass");
        }
    }

    private interface Interface {}

输出结果：

	Methods:
	[public java.lang.String me.shouheng.rtti.RttiTest$SubClass.getName(), public void me.shouheng.rtti.RttiTest$SubClass.setName(java.lang.String), public me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.getSup(), public void me.shouheng.rtti.RttiTest$SubClass.setSup(me.shouheng.rtti.RttiTest$SuperClass), public final void java.lang.Object.wait() throws java.lang.InterruptedException, public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]
	
	DeclaredMethods:
	[public java.lang.String me.shouheng.rtti.RttiTest$SubClass.getName(), private void me.shouheng.rtti.RttiTest$SubClass.out(), public void me.shouheng.rtti.RttiTest$SubClass.setName(java.lang.String), public me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.getSup(), public void me.shouheng.rtti.RttiTest$SubClass.setSup(me.shouheng.rtti.RttiTest$SuperClass)]
	
	MethodInformation:
	getName
	class java.lang.String
	@java.lang.Deprecated()
	My Simple Name
	private method in SubClass
	
	Fields:
	[]
	
	DeclaredFields:
	[private java.lang.String me.shouheng.rtti.RttiTest$SubClass.name, private me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.sup]
	
	FieldInformation:
	java.lang.IllegalAccessException: Class me.shouheng.rtti.RttiTest can not access a member of class me.shouheng.rtti.RttiTest$SubClass with modifiers "private"
	@me.shouheng.rtti.Field()
	false
	name
	Shit
	
	ClassInformation:
	[@java.lang.Deprecated()]
	me.shouheng.rtti.RttiTest.SubClass
	class java.lang.Class
	class me.shouheng.rtti.RttiTest$SuperClass
	[interface me.shouheng.rtti.RttiTest$Interface]
	[public me.shouheng.rtti.RttiTest$SubClass()]
	package me.shouheng.rtti

### 示例2：泛型参数的获取

下面是在实际的框架设计中会用到的一些方法，它尝试从类的泛型中获取泛型的名称：

    private static class Model {}

    private static class Product extends Model {}

    private static class Store<T extends Model> {

        public Store() {
            Class cls = this.getClass();
            ParameterizedType pt = (ParameterizedType) cls.getGenericSuperclass();
            Type[] args = pt.getActualTypeArguments();

            System.out.println(cls);
            System.out.println(pt);
            System.out.println(Arrays.toString(args));
            System.out.println(((Class) args[0]).getSimpleName());
        }
    }

    private static class ProductStore extends Store<Product> {}

    public static void main(String ...args) {
        ProductStore store = new ProductStore();
    }

输出结果：

	class me.shouheng.rtti.RttiTest$ProductStore
	me.shouheng.rtti.RttiTest.me.shouheng.rtti.RttiTest$Store<me.shouheng.rtti.RttiTest$Product>
	[class me.shouheng.rtti.RttiTest$Product]
	Product








