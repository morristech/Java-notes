## MVC

MVC不是一种设计模式，它使用设计模式的组合来实现，MVC即“模型-视图-控制器”。有了前面的设计模式的基础，要理解MVC设计模式并不困难。

下面是一个用于演示的MVC设计模式：

这里定义了一些用于MVC的接口：
	
	public interface IModel {
	    int getValue();
	    void setValue(int value);
	    void registerObserver(IObserver observer);
	    void removeObserver(IObserver observer);
	    void notifyObservers();
	}

以上是用于模型的接口，它包含了对应指定模型的一些特定方法setValue()和getValue()。

	public interface IController {
	    void start();
	    void stop();
	    void increase();
	    void decrease();
	}

以上是用于控制器的一些接口，它的方法是向用户提供的接口方法。

	public interface IObserver {
	    void modelChanged();
	}

以上是用于观察者的接口，这里只是定义了一个用于通知视图的方法。

下面是MVC模式的具体的实现

	public class ConcreteModel implements IModel {
	    private List<IObserver> observers;
	    private int value;
	
	    public ConcreteModel() {
	        this.observers = new ArrayList<IObserver>();
	    }
	
	    public int getValue() {
	        return value;
	    }
	
	    public void setValue(int value) {
	        this.value = value;
	        notifyObservers();
	    }
	
	    public void registerObserver(IObserver observer) {
	        observers.add(observer);
	    }
	
	    public void removeObserver(IObserver observer) {
	        int i = observers.indexOf(observer);
	        if (i >= 0) {
	            observers.remove(observer);
	        }
	    }
	
	    public void notifyObservers() {
	        for (IObserver o : observers) {
	            o.modelChanged();
	        }
	    }
	}

以上是Model的定义。它内部定义了一个观察者的集合，还有一个字段。当字段改变的时候，我们通知各个观察者。

	public class ConcreteView implements IObserver {
	    private IModel model;
	    private IController controller;
	
	    public ConcreteView(IModel model, IController controller) {
	        this.model = model;
	        this.controller = controller;
	        model.registerObserver(this);
	    }
	
	    public void start() {
	        System.out.println("View Start.");
	    }
	
	    public void stop() {
	        System.out.println("View Stop.");
	    }
	
	    public void show() {
	        System.out.println(model.getValue());
	    }
	
	    public void modelChanged() {
	        show();
	    }
	}

以上是View的定义，它需要传入一个Controller和Model。因为，我们需要从Model上面获取模型的数据信息，并用于展示。同时，他实现了IObserver接口，并在构造方法中奖当前的view注册到model中，以便可以接收到数据变化的通知。

	public class ConcreteController implements IController{
	    private IModel model;
	    private ConcreteView view;
	
	    public ConcreteController(IModel model) {
	        this.model = model;
	        this.view = new ConcreteView(model, this);
	    }
	
	    public void start() {
	        view.start();
	    }
	
	    public void stop() {
	        view.stop();
	    }
	
	    public void increase() {
	        model.setValue(model.getValue() + 1);
	    }
	
	    public void decrease() {
	        model.setValue(model.getValue() - 1);
	    }
	}

以上是控制器的定义。我们需要向其传入Model，用于修改模型的信息。而修改之后的又会调用视图层显示出来。所以，

测试代码：

    public static void main(String ...args) {
        IModel model = new ConcreteModel();
        IController controller = new ConcreteController(model);
        controller.start();
        controller.stop();
        controller.increase();
        controller.decrease();
    }

输出结果：

	View Start.
	View Stop.
	1
	0
