## 观察者模式

### 1.接口回调

如果你理解**接口回调**的一些原理的话，理解观察者模式还是比较容易的。所谓接口回调一般应用的场合是“你不知道这个方法什么时候回返回”。比如，一个方法中定义了一个局部变量i，随后开启了一个线程，并在线程内部修改了i，我们希望i的修改的结果最终能够被我们获取到。那么我们该怎么做？

使用return肯定不行，因为方法结束时，线程可能还没有结束，那么修改的内容我们可能获取不到。

    public int cal() {
        int i;
        new Thread(new Runnable() { 
            Thread.sleep(3000); // throw ... ignore it
            i++; 
        }).start();
        return i;
    }

解决的方法就是接口回调，我们可以在调用该方法的时候传入一个接口(callback)的实例。在线程中执行完所有的逻辑之后，我们使用该接口的方法（call方法）将i回调出来。

    public void cal(Callback callback) {
        int i;
        new Thread(new Runnable() { 
            Thread.sleep(3000); // throw ... ignore it
            i++; 
            if (callback != null) {
                callback.call(i);
            }
        }).start();
    }

其实这里也不算是将i回调出来，更准确地它更像是将我们实现的接口的代码（call方法）注入到了线程中调用接口的部分。

还有一些接口回调的典型场景，就是一些界面程序中的事件处理。它们同样也符合我们的“不知道在什么时候会被调用”的要求。

不过这里我们还是将这种机制称为**观察者模式**好了。所谓观察者模式就像在暗中观察一样，每次当**主题**发生变化的时候可以通知我们的**观察者**。

### 2.观察者模式

如果使用上面的那个线程的例子来做类比的话，i就是我们的**主题**，我们进行接口回调的类（将实现的Callback传入到cal方法时所处的类）叫做**观察者**。

![](https://github.com/Shouheng88/Blog-Articles/blob/master/resources/sdrs/observer.png)

以上是观察者模式的UML模型。这里的Subject就是主题的接口，Observer就是观察者的接口。

这里我们给出一份观察者模式的一般使用方法，它和接口回调的方式基本相同：

    public class ConcreteSubkect implements Subject {

        private List<Observer> observers;

        public void registerObserver(Observer o) {
            observers.add(o);
        }

        public void removeObserver(Observer o) {
            int i = observers.indexOf(o);
            if (i >= 0) {
                observers.remove(o);
            } 
        }

        public void notifyObservers() {
            for (Observer o : observers) {
                ((Observer) observers.get(i)).method();
            }
        }

        // ... 其他的方法，当主题变化的时候通知观察者

    }

从上面的代码中我们可以看出，在这里维护了不仅一个观察者，所以使用了列表进行维护。此外，需要将观察者注入到主题中，以动态监听主题的变化并通知观察者。

    public class ConcreteObserver implements Observer {
        
        private Subject subject;

        public ConcreteObserver(Subject subject) {
            this.subject = subject;
            subject.registerObserver(this);
        }
        
        public void method() {
            // ... 当主题发生变化的时候需要执行的逻辑
        }

    }

这里就是观察者的实现方法。在这里我们需要将主题作为构造方法的参数传入进来，并且还要将自身注册到该主题。然后，需要在method方法中加入当主题发生变化的时候要执行的逻辑。

### 3.主题信息获取

如果主题发生了变化，我么怎么将变化之后的信息传递给观察者呢？这要借助于method方法来实现，你可以先找到在观察者和主题中method被调用的位置。当主题发生变化的时候，我们可以在主题中调用传入的观察者的时候，顺便将主题更改后的信息，通过method的参数传入进来。在method方法内部直接使用参数进行处理即可。还有一种方式就是，在观察者内部维护一份主题的实例，当主题内容发生变化的时候，主题直接通知观察者，而不将变化后的信息传递进来。当收到了主题变化的通知之后，观察者通过主题直接获取主题上面的信息。

从传递变化的信息的角度来看，接口回调和观察者模式还是有很大不同的，接口回调直接将信息返回，监听者本身并不维护一份被监听对象的实例；而观察者要维护一份被观察对象的实例，并从上面获取信息。这意味着它们“监听”的对象不同，接口回调对象可能更多是没有办法实例化的，跟监听者属于同一层次的。观察者模式中的主题，应该和观察者属于两个不同的层次。主题可以被实例化，它也仅代表数据之类的合集。

观察者设计模式可以应用到MVC设计模式中，以及其他的需要监听的情形。