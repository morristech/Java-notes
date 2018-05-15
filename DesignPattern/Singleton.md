## 单例模式

确保一个类只有一个实例，并提供一个全局访问点。

在所有的设计模式中，单例模式是最简单也是最常用的一种设计模式，它只为一个实例提供一个全局对象，内次尝试去获取一个类的实例的时候，保证获取到的都是这一个对象。

这里我们给出它在Java当中的几种写法：

### 1.经典单例方法

    public class Singleton {
        private static Singleton singleton;

        private Singleton() {}

        public static Singleton getInstance() {
            if (singleton == null) {
                singleton = new Singleton();
            }
            return singleton;
        }
    } 

上面是第一种写法，当我们第一次调用getInstance()方法的时候，因为singleton是null的，所以就实例化一个。此后，每次再获取该实例的时候都不为null了，获取到的实例就是那个静态的singleton实例。

### 2.同步的单例方法

虽然上面的方法能达到获取单个实例的目的，但是在多线程中仍然是有问题的。如果一个线程正在尝试创建singleton的实例，而这时候刚好另一个线程进入了该方法。此时，因为第一个线程还没有将singleton创建完毕，singleton仍然是null，它也进入了该方法的内部。这时候执行到renturn的时候，可能返回两个实例。这在某些应用场景中可能会出现严重的错误，比如严格限制只能有一个类的实例的场景，或者当该类会占用很大的资源的时候。所以，为了解决这个问题，有必要对其在多线程环境中的使用进行一些优化。

下面是一种优化的策咯：

    public class Singleton {
        private static Singleton singleton;

        private Singleton() {}

        public sychronized static Singleton getInstance() {
            if (singleton == null) {
                singleton = new Singleton();
            }
            return singleton;
        }
    } 

这样我们在该方法上添加了`sychronized`关键字之后，就不会有两个方法同时进入`getInstance()`方法内部了。

虽然上述方法能够达到我们保证只存在一个实例的目的，但是这对性能造成了一定的影响，而且是没有必要的影响。这是因为，我们其实只要在每次尝试去创建类的实例上面加锁就可以了，没必要在该方法上面加锁。因为，每次尝试去获取类的实例的时候，不是一定要创建的。只有第一次进入之该方法时会创建，随后的访问都是简单的获取操作了。而我们在该方法上面加锁之后，每次获取操作也都要等待锁。这显然是没必要的，因此我们还需要继续对其进行优化。

### 3.使用立即初始化的方式

如果程序在创建和运行时负担不繁重，我们可以采用立即初始化的方式来创建单例。

    public class Singleton {
        private static Singleton singleton = new Singleton();

        private Singleton() {}

        public static Singleton getInstance() {
            return singleton;
        }
    } 

使用上面的做法，我们保证了JVM在每次加载该类的时候都会立即初始化唯一的实例，从而保证每次在获取的时候都只能获取到唯一的实例。虽然导致类装载的原因有很多种，在单例模式中大多数都是调用getInstance方法，但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance显然没有达到lazy loading的效果。

与上面的方式类似的还有使用静态代码块来获取类的实例的方式：

    public class Singleton {
        private static Singleton singleton;

        static { singleton = new Singleton(); } 

        private Singleton() {}

        public static Singleton getInstance() {
            return singleton;
        }
    } 

和上面在静态字段中直接初始化类的实例的方式差不都，只是将初始化的代码放进了一个静态代码块当中。这样实例的初始化时间仍然是在类第一次被加载的时候。

### 4.使用枚举

与上面使用静态立即初始化的方法类似，你也可以使用枚举的方式来获取一个类的实例。它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。下面是一个示例：

    public enum Singleton {
        SINGLETON;

        Singleton() {}
    } 

### 5.双重检查加锁的单例方法

利用双重检查加锁，首先检查是否实例已经创建了。如果实例尚未创建，才同步。这样只有在第一次创建的时候才会对多线程进行加锁限制：

    public class Singleton {
        private volatile static Singleton singleton;

        private Singleton() {}

        public static Singleton getInstance() {
            if (singleton == null) {
                sychronized(Singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }
    } 

这里volitile关键字的作用是，当多线程实例化singleton之后会立即被其他线程看到。

### 6.使用静态内部类
  
    public class Singleton {

        private static class SingletonHolder {
            private static Singleton INSTANCE = new Singleton(); 
        } 

        private Singleton() {}

        public static Singleton getInstance() {
            return SingletonHolder.INSTANCE;
        }
    } 

这里定义了一个静态内部类`SingletonHolder`，并在类中初始化一个静态`Singleton`字段`INSTANCE`。当调用了`Singleton.getInstance()`来获取`Singleton`实例的时候，会用`SingletonHolder.INSTANCE`返回实例。

这种方式同样能够满足多线程的需求，它同样能够保证在第一次获取的时候将实例初始化，此后获取的都是实例化的结果。与上面使用静态字段不同的是，它没有把创建实例的静态字段或者静态代码块放在`Singleton`中，而是放在了`SingletonHolder`类中。在使用静态字段或者代码块的时候，只要`Singleton`被加载，就会调用初始化`Singleton`的操作。但是，在这方式中只有当使用`SingletonHolder`类的时候才会执行初始化`Singleton`的操作。也就是只有调用`Singleton.getInstance()`方法的时候才会对`Singleton`执行初始化。

想象一下，如果实例化`Singleton`很消耗资源，我想让他延迟加载;另外一方面，我不希望在`Singleton`类加载时就实例化，因为我不能确保`Singleton`类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化`Singleton`显然是不合适的。这个时候，这种方式相比使用静态字段或者代码块的方式就显得很合理。

**如果将使用静态代码块和静态字段看作两个不同类型的单例方式，那么上面我们一共列出了7种不同的实现单例的方式。**

### 关于单例模式中的一些小概念

#### 1.饿汉式和懒汉式区别

1. 饿汉就是类一旦加载，就把单例初始化完成，保证getInstance的时候，单例是已经存在的了，
2. 而懒汉比较懒，只有当调用getInstance的时候，才回去初始化这个单例。

另外从以下两点再区分以下这两种方式：

##### 1.线程安全

1. 饿汉式天生就是线程安全的，可以直接用于多线程而不会出现问题，
2. 懒汉式本身是非线程安全的，为了实现线程安全有几种写法，需要增加锁或者使用静态内部类的方式等。

##### 2.资源加载和性能

1. 饿汉式在类创建的同时就实例化一个静态对象出来，不管之后会不会使用这个单例，都会占据一定的内存，但是相应的，在第一次调用时速度也会更快，因为其资源已经初始化完成，

2. 而懒汉式顾名思义，会延迟加载，在第一次使用该单例的时候才会实例化对象出来，第一次调用时要做初始化，如果要做的工作比较多，性能上会有些延迟，之后就和饿汉式一样了。

### 其他

1. 虽然，我们可以通过将一个类的构造器设置为private的，来避免外部直接调用构造器来初始化类的实例，但是用户可以通过AccessibleObject.setAccessible()方法，通过反射来调用私有构造器。我们可以让该类在创建第二个实例的时候抛出异常。

2. 单例的序列化问题：仅仅在类上声明implements Serializable是不够的，还需要声明所有实例域都是瞬时(transient)的，并提供一个readResolve方法。否则，每次反序列化一个实例时，都会创建一个新的实例。当然，枚举单例不会有这个问题。

	    public static class Singleton implements Serializable {
	        private static class SingletonHolder {
	            private static final Singleton INSTANCE = new Singleton();
	        }
	
	        private Singleton() {}
	
	        public static final Singleton getInstance() {
	            return SingletonHolder.INSTANCE;
	        }
	
	        private Object readResolve() {
	            return SingletonHolder.INSTANCE;
	        }
	    }

    原理是在ObjectInputStream内部获取实例之后，如果该实例实现了readResolve()方法，那么它就调用这个方法来获取。



