## 中介者模式

中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。

![](http://upload.ouliu.net/i/20171030095819gs87e.jpeg)

中介者模式主要用来解决对象与对象之间存在大量的关联关系时，导致系统非常复杂的问题，尤其是当多个类之间构成了网状的关系的时候。中介者模式通过将网状的结构，分离成为形状的结构来解决复杂的对象关联。具体而言是通过将对象 Colleague 之间的通信封装到一个类中单独处理来完成的。

中介者模式典型的使用场景有：

1. 中国加入 WTO 之前是各个国家相互贸易，结构复杂，现在是各个国家通过 WTO 来互相贸易。 
2. 机场调度系统。 
3. MVC 框架，其中C（控制器）就是 M（模型）和 V（视图）的中介者。

### 使用示例

下面的例子改编自《大话设计模式》：

    // 联合国
    public static abstract class UnitedNations {
        public abstract void declare(String msg, Country college);
    }

    // 抽象的国家
    public static abstract class Country {
        protected UnitedNations mediator;

        public Country(UnitedNations mediator) {
            this.mediator = mediator;
        }

        protected abstract void declare(String msg);
    }

    // 美国
    public static class USA extends Country {

        public USA(UnitedNations mediator) {
            super(mediator);
        }

        @Override
        protected void declare(String msg) {
            mediator.declare(msg, this);
        }

        public void getMsg(String msg) {
            System.out.println("USA Got:" + msg);
        }
    }

    // 伊拉克
    public static class Iraq extends Country {

        public Iraq(UnitedNations mediator) {
            super(mediator);
        }

        @Override
        protected void declare(String msg) {
            mediator.declare(msg, this);
        }

        public void getMsg(String msg) {
            System.out.println("Iraq Got:" + msg);
        }
    }

    // 安理会
    public static class UnitedNationsCouncil extends UnitedNations {
        private USA usa;
        private Iraq iraq;

        public void setUsa(USA usa) {
            this.usa = usa;
        }

        public void setIraq(Iraq iraq) {
            this.iraq = iraq;
        }

        @Override
        public void declare(String msg, Country college) {
            if (college == usa) {
                iraq.getMsg(msg);
            } else if (college == iraq) {
                usa.getMsg(msg);
            }
        }
    }

    public static void main(String...args) {
        UnitedNationsCouncil council = new UnitedNationsCouncil();
        USA usa = new USA(council);
        Iraq iraq = new Iraq(council);
        council.setIraq(iraq);
        council.setUsa(usa);
        usa.declare("放下武器，赶快投降");
        iraq.declare("我们绝不投降");
    }

输出结果：

    Iraq Got:放下武器，赶快投降
    USA Got:我们绝不投降

总结：

从上面的例子中我们可以看出，中介者在这里起到一个消息转发的作用：当USA发起消息的时候，USA通过内部维护的中介者mediator的declare()方法来将消息传递出去。同时在mediator内部也维护了USA和Iraq的两个实例。当某个实例调用declare()方法的时候，它在declare()方法内部对消息进行转发给指定的国家，即通过调用国家的getMsg()来传递消息。

### 优缺点

优点： 

1. 降低了类的复杂度，将一对多转化成了一对一。 
2. 各个类之间的解耦。 
3. 符合迪米特原则。

缺点：

1. 中介者会庞大，变得复杂难以维护。
2. 当系统中出现“多对多”的时候，不要急于使用中介者模式，而应首先考虑系统在设计上是不是合理、

### 使用场景

1. 系统中对象之间存在比较复杂的引用关系，导致它们之间的依赖关系结构混乱而且难以复用该对象。 
2. 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。

注意事项：不应当在职责混乱的时候使用。
