## 装时者模式

你可以将装饰者模式理解成**“添加一点佐料”**的意思，这里的“佐料”就是装饰的意思。“添加一点佐料”即在原始的对象的基础之上对其方法进行一些修改。它的UML建模如如下所示：

![](http://upload.ouliu.net/i/20171029144429rh51t.png)

它所要达到的装饰目的就是，对一个对象的方法进行修改，但是不会影响该方法的如何被调用。这就需要继承原来对象的类，并实现其中的方法，然后将装饰后的对象赋值给原来的对象。如：

    public abstract class Decorator extends BaseClass {
        // ... 如果你需要子类必须实现某些方法，就在这里将它们定义成抽象的
    }

上面的方法类是我们在原始对象和药定义的装饰器中间插入的一个类，理论上没有也可以，但是我们可以在这里添加一些所有的装时器都需要的逻，以此来对装饰器做一些约束。

    public class Decorator1 extends Decorator {

        private BaseClass baseClass;

        public Decorator1 (BaseClass baseClass) {
            this.baseClass = baseClass;
        }

        @Override
        public void method() {
            // 修改原始对象的逻辑
            // 如果需要使用原来的逻辑，就调用 baseClass.method();
        }

    }

以上就是我们实现的一个装饰器。注意这里我们将原始的对象传入，然后在我们的方法内部可以调用原始对象的方法进行逻辑操作。这里的Decorator1继承了Decorator，而Decorator继承了BaseClass。所以，在使用的时候，就可以像下面的样子进行使用：

    BaseClass baseClass = new BaseClass();
    baseClass = new Decorator1(baseClass);

从上面的例子大概可以理解所谓的“装饰”了，就是指对原来的baseClass的方法进行了修改，但是baseClass将如何被使用的逻辑不用变化。其实，这里虽然将其称为“装饰”，但是我们可以知道，baseClass的引用其实已经发生了变化。但是这种变化不影响它如何被使用，只是修改了内部的逻辑。即**对外不变，对内改变**。

典型的装饰器使用是我们熟知的**IO流**。它通过一层层包装来在原始的IO流基础之上增加逻辑以满足不同的使用场景的需求。
    















