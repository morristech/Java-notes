## 享元模式

**享元模式**（Flyweight Pattern）是一种软件设计模式。它使用共享物件，用来尽可能减少内存使用量以及分享资讯给尽可能多的相似物件；它适合用于只是因重复而导致使用无法令人接受的大量内存的大量物件。通常物件中的部分状态是可以分享。常见做法是把它们放在外部数据结构，当需要使用时再将它们传递给享元

![](http://upload.ouliu.net/i/20171030133841yv994.jpeg)

### 示例代码

抽象的享元

    public static abstract class Flyweight {
        public abstract void operation(int param);
    }

具体的享元

    public static class ConcreteFlyweight extends Flyweight {

        @Override
        public void operation(int param) {
            System.out.println("Concrete Flyweight ( " + hashCode() +" ) : " + param);
        }
    }

非用来共享的实体

    public static class UnsharedConcreteFlyweight extends Flyweight {

        @Override
        public void operation(int param) {
            System.out.println("Unshared Concrete Flyweight ( " + hashCode() +" ) : " + param);
        }
    }

享元的工厂，也就是将共享的实体放在一个哈希表中进行存储，然后每次从哈希表中尝试获取元素：

    public static class FlyweightFactory {
        private HashMap<String, Flyweight> flyweights = new HashMap<String, Flyweight>();

        public Flyweight getFlyWeight(String key) {
            Flyweight flyweight = flyweights.get(key);
            if (flyweight == null) {
                flyweight = new ConcreteFlyweight();
                flyweights.put(key, flyweight);
            }
            return flyweight;
        }
    }

测试代码

    public static void main(String...args) {
        final String key = "first flyweight";
        FlyweightFactory factory = new FlyweightFactory();
        Flyweight flyweight = factory.getFlyWeight(key);
        flyweight.operation(1);
        Flyweight flyweight1 = factory.getFlyWeight(key);
        flyweight1.operation(2);
    }

输出结果:

    Concrete Flyweight ( 685325104 ) : 1
    Concrete Flyweight ( 685325104 ) : 2

从上面看出，享元无非就是将指定类型的数据实体放进一个哈希表中缓存起来，以达到共享的目的。

### 更多

**1.应用实例**： 

1. JAVA 中的 String，如果有则返回，如果没有则创建一个字符串保存在字符串缓存池里面。 
2. 数据库的数据池。

**2.优点**：

1. 大大减少对象的创建，降低系统的内存，使效率提高。

**3.缺点**：

提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱。

**4.使用场景**： 

1. 系统有大量相似对象。 
2. 需要缓冲池的场景。

**5.注意事项**： 

1. 注意划分外部状态和内部状态，否则可能会引起线程安全问题。 
2. 这些类必须有一个工厂对象加以控制。
