## 桥接模式

桥接模式是将抽象部分与它的实现部分分离，使它们都可以独立地变化。它是一种对象结构型模式，又称为柄体模式或接口模式。

![](https://gss1.bdstatic.com/-vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=63068dbf90529822113e3191b6a310ae/b2de9c82d158ccbf49a4044219d8bc3eb13541a9.jpg)

桥接模式的做法是把变化部分抽象出来，使变化部分与主类分离开来，从而将多个维度的变化彻底分离。最后，提供一个管理类来组合不同维度上的变化，通过这种组合来满足业务的需要。

假如我们有手机品牌Lenovo和iPhone，有两种软件，一个是通讯软件Chat，另一个是游戏软件Game. 那么我们如果要做软件的话肯定要做四个：用于Lenovo（Android系统）的Chat，用于iPhone的Chat，用于Lenovo的Game，用于iPhone的Game. 

按照上面的来分的话，应该有四种不同类型的软件。那么为了给出这四种不同类型软件的定义，我们可能要这样定义它的类：

    public interface PhoneInterface {}

    public class Lenovo implements PhoneInterface {}

    public class iPhone implements PhoneInterface {}

    public class LenovoChatSoft extends Lenovo {}

    public class iPhoneChatSoft extends iPhone {}

    public class LenovoGameSoft extends Lenovo {}

    public class iPhonegameSoft extends iPhone {}

当然，如果你要按照软件的类型来进行定义的话也是可以的。但是这种方式存在的问题是，如果加入了新的品牌（系统）和软件类型，整个类要爆炸了。这主要是因为每个实体包含了，品牌（系统）和软件类型两个维度。

那么我们可以使用下面的方式来定义各个实体类：

定义一个接口PhoneInterface来表示手机的类型，然后定义它的两个实现，Lenovo和iPhone。 它们内部各自维护一个接口，SoftInteface. SoftInterface有两个实现，ChatSoft和GameSoft。 即：

    public interface Soft {}

    public class GameSoft implements Soft {}

    public class ChatSoft implements Soft {}

    public interface PhoneInterface {
        void run();
    }

    public abstract class AbstractPhone {
        private Soft soft;

        public AbstractPhone(Soft soft) {
            this.soft = soft;
        }
        
        public abstract void run();
    }

    public class Lenovo extends AbstractPhone {
        
        public Lenovo(Soft soft) {
            super(soft);
        }

        public void run() {}
    }

    public class iPhone extends AbstractPhone {
        public iPhone(Soft soft) {
            super(soft);
        }

        public void run() {}
    }

也就是，我们将软件封装成一个接口，然后各种不同类型的软件只要实现该接口即可。对于品牌这个维度，我们定义了一个接口PhoneInterface，并添加了一个抽象的实现AbstractPhone，它内部维护一个Soft类型的实例。最后又两个具体的品牌类型实现Lenovo和iPhone。

或者参考这篇博文[http://blog.csdn.net/andy_gx/article/details/46885033](http://blog.csdn.net/andy_gx/article/details/46885033)

他举得例子也是将汽车品牌和导航仪进行组合，可以看出使用桥接模式大大减少了由于组合而带来的类爆炸。

### 桥接模式分析

理解桥接模式，重点需要理解如何将抽象化(Abstraction)与实现化(Implementation)脱耦，使得二者可以独立地变化。

1. 抽象化：抽象化就是忽略一些信息，把不同的实体当作同样的实体对待。在面向对象中，将对象的共同性质抽取出来形成类的过程即为抽象化的过程。
2. 实现化：针对抽象化给出的具体实现，就是实现化，抽象化与实现化是一对互逆的概念，实现化产生的对象比抽象化更具体，是对抽象化事物的进一步具体化的产物。
3. 脱耦：脱耦就是将抽象化和实现化之间的耦合解脱开，或者说是将它们之间的强关联改换成弱关联，将两个角色之间的继承关系改为关联关系。桥接模式中的所谓脱耦，就是指在一个软件系统的抽象化和实现化之间使用关联关系（组合或者聚合关系）而不是继承关系，从而使两者可以相对独立地变化，这就是桥接模式的用意。 

### 桥接模式优缺点

#### 1.桥接模式的优点
 
1. 桥接模式有时类似于多继承方案，但是多继承方案违背了类的单一职责原则（即一个类只有一个变化的原因），复用性比较差，而且多继承结构中类的个数非常庞大，桥接模式是比多继承方案更好的解决方法。 
2. 桥接模式提高了系统的可扩充性，在两个变化维度中任意扩展一个维度，都不需要修改原有系统。 
3. 实现细节对客户透明，可以对用户隐藏实现细节。 

#### 2.桥接模式的缺点

1. 桥接模式的引入会增加系统的理解与设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计与编程；
2. 桥接模式要求正确识别出系统中两个独立变化的维度，因此其使用范围具有一定的局限性。

### 桥接模式适用环境

在以下情况下可以使用桥接模式：
1. 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。
2. 抽象化角色和实现化角色可以以继承的方式独立扩展而互不影响，在程序运行时可以动态将一个抽象化子类的对象和一个实现化子类的对象进行组合，即系统需要对抽象化角色和实现化角色进行动态耦合。
3. 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。
4. 虽然在系统中使用继承是没有问题的，但是由于抽象化角色和具体化角色需要独立变化，设计要求需要独立管理这两者。
5. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。

