## 命令模式

命令模式将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也可支持撤销的操作。

以下是一个典型的命令模式的类图：

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1505999073874&di=b328b1840450ddd6de84bff6773be897&imgtype=0&src=http%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F784420%2F201511%2F784420-20151125011635827-1718294345.png)

### 1.命令模式的组成

命令模式由以下角色组成：

1. **命令角色**(Command)：为所有的 命令声明了一个接口，调用命令对象的execute()方法，就可以让接收者进行相关的动作。这个接口也可以定义一个undo()方法，用于撤销命令。

2. **具体命令角色**(Concrete Command)：定义了动作和接收者之间的绑定关系。调用者只要调用execute()就可以发出请求。然后Concrete Command调用接收者的一个或多个动作。

3. **接收者角色**(Receiver)：负责具体实施和执行一个请求。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能。

4. **请求者(调用者)角色**(Invoker)：持有命令对象，并在某个时间点调用命令对象的execute()方法来讲请求付诸实践。

5. **客户角色**(Client)：创建一个具体命令对象并设定该命令对象的接收者。

如果以一个烤肉店为例的话，那么这里的命令角色相当于顾客的订单，接收者角色相当于服务于，请求者相当于厨师。服务员把命令传达给厨师，然后厨师来执行命令。

### 2.命令模式的示例

1.命令角色

    public interface Command {
        public void execute();
    }

2.具体命令角色

    public class ConcreteCommand implements Command {  
        private Receiver receiver;  
  
        public ConcreteCommand(Receiver receiver) {  
            this.receiver = receiver;  
        }  
  
        @Override  
        public void execute() {  
            receiver.doAction();  
        }  
    }  
 
3.接收者角色

    public class Receiver {  
        public void doAction() {  
            // 
        }  
    }

4.请求者（执行者）角色

    public class Invoker {   
        private Command command;  
  
        public Invoker(Command command) {  
            this.command = command;  
        }  
  
        public void doInvokerAction() {  
            command.execute();  
        }  
    }

5.客户角色

    public class Client {  
        public static void main(String[] args) {  
            Receiver receiver = new Receiver();  
            Command command = new ConcreteCommand(receiver);  
            Invoker invoker = new Invoker(command);  
            invoker.doInvokerAction();  
        }  
    }

从上面的执行过程可以看出，Command要维护一个Receiver，而Invoker维护一个Command，这是命令模式的三个主要部分之间的关系。

复合命令

    public class MacroCommand implements Command {  
        Command[] commands; 

        public MacroCommand(Command[] commands) {  
            this.commands=commands;  
        }  

        @Override  
        public void execute() {  
            for(Command command : this.commands)  
               command.execute();  
        }  
  
        @Override  
        public void undo() {  
            for(Command command : this.commands)  
                command.undo();  
        }
    }

### 3.命令模式作用

1. 它能较容易地设计一个命令队列；
2. 在需要的情况下，可以较容易地将命令记入日志；
3. 允许接收请求的一方决定是否要否决请求；
4. 容易的实现对请求的撤销和重做；
5. 由于加进新的具体命令类不影响其他的类，因此增加新的具体命令类很容易；
6. 命令模式把请求一个操作的对象与知道怎么执行一个操作的对象分割开。