## 适配器模式和外观模式

适配器模式将一个类的接口。转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1506002854655&di=236218cf0ec6dfca3c402d46885801b9&imgtype=0&src=http%3A%2F%2Fimages.cnitblog.com%2Fblog2015%2F674607%2F201504%2F101906587743735.png)

所谓适配器，你可以将其理解成将一种数据类型通过适配器让其适用于另一种类型。以上面的图为例。加入Client需要一个Targe类型（目标类型）的实体，但是我们只有一个Adaptee（被适配者）类型。这时候我们可以通过一个中间的适配器Adapter来完成。

它的操作一点也不神秘，仅仅是让Adapter实现Target，这样Adapter就可以被当成Target来使用了。而Adapter内部的逻辑则委托给Adaptee来实现。所以，我们需要为Adapter传入指定一个被适配者。

### 示例

我们可以考虑变压器的工作效果，假设高电压为`HightVoltage`、低电压为`LowVoletage`以及变压器为`Transformer`. 那么现在我们需要将高电压经过变压器转换成低电压来使用。那么我们可以定义

    public interface HightVoltage {
        int output();
    }

    public class Voltage220 implements HightVoltage {
        @Override
        public int output() { 
            return 220; 
        }
    }

    public interface LowVoltage {
        int output();
    }

    public class Transormer implements LowVoltage {
        private HightVoltage hightVoltage;

        public Transformer(HightVoltage hightVoltate) {
            this.hightVoltage = hightVoltage;
        }

        public int output() {
            hightVoltage.output() / 2;
        }
    }

    public static void main(String ...args) {
        HightVoltage voltage220 = new Voltage220();
        LowVoltage lowVoltage = new Transformer(voltage220);
    }

在上面的代码中我们需要实现的操作是将220V的电压转换成可用的110V的电压。这里我们使用了一个变压器将传入的电压做了除以2的处理。也许，你会觉得这更像是装饰器模式。因为它只是在高电压的基础上进行了一层封装来，修改了输出电压的方法。但是，这里和装饰器不同的一点是，最终我们适配器的结果赋值给了`LowVoltage`实例，而不是`HightVoltage`. 如果是装饰器的话，为了营造出我们只是为原来的对象“添加了一些佐料”的假象（实际上引用都已经变了），需要将装饰之后的结果赋值给原来的对象。这里并没有赋值给原来的对象，而是新的类型，所以这里是适配器而不是装饰器。这是装饰器和适配器直观上的区别。

## 外观模式

外观模式将一个或者数个类的复杂实现隐藏在背后，只暴露几个方法给客户端使用。通过使用外观类，可以将一个复杂的子系统变得更加容易。

![](http://images2015.cnblogs.com/blog/1016421/201609/1016421-20160910215031051-160598096.png)

以上是外观模式的一个类图。这里需要Facade类引用模块A、模块B和模块C，并三者组合起来实现的功能封装在Facade的一个方法中。

也就是将一系列操作封装在一个方法中供给外部调用，比如关闭电视机操作，它其实涉及的东西很多，但是对我们用户而言，只要按一下关机按钮就可以了。

### 另一个使用场景

以上是门面模式的一种使用情景。此外，它还被用来隐藏对外接口。比如，

    public interface IBase {
        void method01();
        void method02();
        void method03();
    }

    public class MyClass implements IBase {

        public MyClass() {}

        @Override
        public void method01() {}

        @Override
        public void method02() {}

        @Override
        public void method03() {}

        public void method04() {}

        public void method05() {}
    }

这里我们定义了一个基接口IBase，并且定义了一个它的实现MyClass，而且又在MyClass中添加了一些方法method04()、method05()，现在假设我们需要将MyClass给别人使用，但是我们又不希望将method04()、method05()方法暴露，而且也不希望用户在使用的时候将我们的给的实例强制转换成MyClass来使用。这时候我们可以使用门面模式来解决这个问题。我们只需要按照下面的方式定义一个门面，并将其作为参数提供给用户使用即可。

    public class MyClassFacade implements IBase {
        private MyClass myClass;

        public MyClassFacade(MyClass myClass) {
            this.myClass = myClass;
        }

        @Override
        public void method01() {
            myClass.method01();
        }

        @Override
        public void method02() {
            myClass.method02();
        }

        @Override
        public void method03() {
            myClass.method03();
        }
    }

### 外观模式的总结

为了使用外观模式，在设计的时候我们应该注意：

1. 在设计初期阶段，应该有意识的将不同的层进行分离，比如经典的三层框架；
2. 其次，在开发阶段，子系统往往因为不断重构而演变得越来越复杂，增加外观可以提供一个简单的接口。
