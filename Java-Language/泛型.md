## 泛型

1. 使用泛类型的代码意味着可以被很多不同类型的对象重用
2. 在之前定义ArrayList的时候是管理一个Object[]类型的数组，比如add方法以前是add(Object obj)，现在是add(E e)，那么以前的时候显然如果你定义一个String类型的ArrayList，传入File类型也是可以的，因为它也继承自Object，这显然就会出现错误。但是有了泛型之后，传入的只能是E类型的，不然会报错。
3. Java编译器最终将泛型版本编译为无泛型版本
4. Java泛型是使用类型擦除来实现的，它的好处只是提供了编译器类型检查机制。在泛型方法内部无法获取任何关于泛型参数类型的信息，泛型参数只是起到了占位的作用。如List<String>在运行时实际上是List类型，普通类型被擦除为Object。因为泛型是擦除的，所以泛型不能用于显式地引用运行时类型的操作中，例如：转型、instanceof操作或者new表达式。
5. 使用extends的意义就在于指定了擦除的上界。

### 1、泛型类

泛型类的声明与一般类的声明语法一致，但需要在声明的泛型类名称后使用<>指定一个或多个类型参数，如

    class MyClass <E>{} 

或 

    class MyClass <K,V>{}

### 2、泛型接口

泛型接口的声明与一般接口的声明语法一致，但需要在声明的泛型接口名称后使用<>指定一个或多个类型参数，如

    interface IMyMap<E> 

或

    intetface IMyMap<K,V> 

### 3、泛型方法

从下面的示例中，我们可以看出当定义了一个泛型方法的时候，可以提高代码的复用性——将两种不同的数据类型传入到方法中都能进行处理。如

    public void method(T e) {}

或

    public void method(List<?> list)

### 4、泛型参数的约束

形式：

    <T extends 基类或接口>

或

    <T extends 基类或接口1 & 基类或接口2>

前面的形式表示T需要是指定的基类或者接口的子类，后面的形式表示T需要是指定的接口或者基类1的子类并且是基类或者接口2的子类。

### 5、泛型的补偿

#### 5.1 创建实例

使用类型标签来获取指定类型的实例：

    try {
        Person person = Person.class.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }

但是使用上面的方式，要求指定的类型标签必须有默认的构造器。

#### 5.2 创建数组

可以使用`类型标签+Array.newInstance()`的方式实现：

    int[] arr = (int[]) Array.newInstance(int.class, 5);

下面的这种方式在运行时不会出错，但是在main方法中强制进行类型转换的时候会出错：

    private static  <T> T[] createArray(T t) {
        T[] arr = (T[]) new Object[5];
        arr[0] = t;
        return arr;
    }

    public static void main(String ...args) {
        Integer[] array = createArray(5); // ClassCastException
        System.out.println(Arrays.toString(array));
    }

这是因为它的运行时类型仍然是Object[]，写成下面的形式就不会错了：

    public static void main(String ...args) {
        Object[] array = createArray(5); // 不会出错
        System.out.println(Arrays.toString(array));
        Integer integer = (Integer) array[0]; // 也不会错
        System.out.println(integer);
    }

因为有查了擦除，数组的运行时类型只能是Object[].

#### 5.3 自限定的类型

    private static class SelfBounded<T extends SelfBounded<T>> {}

    private static class A extends SelfBounded<A> {}
    
    private static class B extends SelfBounded<A> {}

    private static class C extends SelfBounded {}

    // 错误!
    //    private static class D extends SelfBounded<C> {}

    // 错误!
    //    private static class E extends SelfBounded<B> {}

    // 错误!
    //    private static class F extends SelfBounded<D> {}

可以看出，当定义了`class M extends SelfBounded<N>`的时候，这里对N的要求是它的必须实现了`SelfBounded<N>`. 

#### 5.4 泛类与子类

虽然`Object obj = new Integer(123);`是可行的，但是`ArraylList<Object> ao = new ArrayList<Integer>();`是错误的，因为`Integer`是`Object`的派生类，但是`ArrayList<Integer>`不是`ArraylList<Object>`的派生类. 你可以将其理解成不能将鸡的容器赋值给鸭子的容器。

#### 5.5 通配符

根据上面的泛类与子类的关系，如果要实现一个函数，如

    void PrintArrayList(ArrayList<Object> c){
        for(Object obj:c){}
    }

那么`ArrayList<Integer>(10)`的实例是无法传入到该函数中的，因为`ArrayList<Integer>`和`ArrayList<Object>`是没有继承关系的。在这种情况下就可以使用通配符解决这个问题。我们可以定义上面的函数为如下形式，这样就可以将泛型传入了。

    PrintArrayList(ArrayList<?>c){
        for(Object obj:c){}
    }

当然，也可以指定通配符？的约束，即将其写成下面的形式

    <? extends 基类>
    <? super 派生类>

就是在运行时指定擦除的边界。

### 其他

下面一实例是Spring中获取Bean的一段代码：

    public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
        this.assertBeanFactoryActive();
        return this.getBeanFactory().getBean(name, requiredType);
    }