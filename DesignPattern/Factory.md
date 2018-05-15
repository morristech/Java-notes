## 工厂模式

工厂模式的类型还是比较多，相互之间有些细微的区别，不过都是用来创建新的对象了。所谓的工厂的含义就是用来生产对象的意思。使用它的好处是，我们可以将获取一个类的实例的操作封装到一个方法中。这样我们需要修改获取类的实例的逻辑的时候只要对该方法进行修改就可以了。

我们这里可以将其分成三种类型。

### 简单工厂模式

![](https://pic1.zhimg.com/09067f878916c0e4377bfadc82afc248_b.png)

以上就是简单工厂，它不属于23种里的一种，简单来理解就是创建一个工厂类用来根据条件返回不同类型的实体。即使这种方式也有静态工厂和实例工厂两种区分:

    publuc class MouseFactory {
        public Mouse createMouse(int i) {
            // 实际创建对象的逻辑
        }
    }

以及

    publuc class MouseFactory {  
        public static Mouse createMouse(int i) {
            // 实际创建对象的逻辑
        }
    }

后者被称为静态工厂，它的好处在于不需要创建对象就可以直接使用静态方法来获取一个类的实例。它的缺点是不能通过继承来改变创建方法的行为。

### 工厂模式

![](https://pic4.zhimg.com/69ab924585b751cb9e7bc7b7f9f2179b_b.png)

工厂模式在基类中定义创建类的实例的方法，然后要求子类在定义的时候实现这些方法。以上图为例，我们在`MouseFactory`类中定义了创建对象的方法`createMouse()`，然后它有两个子类`HpMouseFactory`和`DellMouseFactory`，子类中需要实现这两个方法。在上图中两个子类的工厂分别生产不同类型的产品`HpMouse`和`DellMouse`.

工厂方法将一个类的实例化延迟到了子类中，而且子类需要根据自己的工厂的性质来生产不同类型的“产品”。

### 抽象工厂

![](https://pic3.zhimg.com/ab2a90cfcc7a971b1e3127d1f531a486_b.png)

上面是抽象工厂的一个图示。这里的`PcFactory`就是一个抽象工厂，而`HpFactory`和`DellFactory`是其两个具体的实现。所谓抽象工厂不仅要实现某个类型的产品，它同时还要实现与之相连的其他产品。像在这里，一个“电脑生成工厂”不仅要实现键盘的生成，同时还要实现鼠标的生成。

### 抽像工厂和工厂方法的一些区别：

1. **抽象工厂通常通过工厂方法来实现具体的工厂**。在上面的案例中就是`PcFactory`需要子类`HpFactory`和`DellFactory`继承该类来实现两种电脑的生产工厂。


当需要创建产品的家族或想要生产相关产品的集合的时候，可以使用抽象工厂。

图片来自：[DesignPattern](https://link.zhihu.com/?target=http%3A//ichennan.com/2016/08/09/DesignPattern.html)



