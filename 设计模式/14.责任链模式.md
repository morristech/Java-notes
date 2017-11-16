## 责任链模式

责任链模式是一种设计模式。在责任链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。

![](http://upload.ouliu.net/i/20171030081024jx1wv.png)

责任链模式涉及到的角色如下所示：

1. **抽象处理者(Handler)角色**：定义出一个处理请求的接口。如果需要，接口可以定义 出一个方法以设定和返回对下家的引用。这个角色通常由一个Java抽象类或者Java接口实现。上图中Handler类的聚合关系给出了具体子类对下家的引用，抽象方法handleRequest()规范了子类处理请求的操作。
2. **具体处理者(ConcreteHandler)角色**：具体处理者接到请求后，可以选择将请求处理掉，或者将请求传给下家。由于具体处理者持有对下家的引用，因此，如果需要，具体处理者可以访问下家。

比较常见的例子是，我们发起工作流的时候，需要我们的各级主管来进行审批。类似的活动，即某个任务或者请求需要一系列执行者来处理的情形。

    public abstract class Handler {

        // 持有后继的责任对象
        protected Handler successor;

        /**
        * 示意处理请求的方法，虽然这个示意方法是没有传入参数的
        * 但实际是可以传入参数的，根据具体需要来选择是否传递参数 */
        public abstract void handleRequest();

    　　 /**
        * 取值方法 */
        public Handler getSuccessor() {
        　　return successor;
        }

        /**
        * 赋值方法，设置后继的责任对象 */
        public void setSuccessor(Handler successor) {
            this.successor = successor;
        }
    }

具体处理者角色

    public class ConcreteHandler extends Handler {

        /**
        * 处理方法，调用此方法处理请求 */
        @Override
        public void handleRequest() {

           /**
            * 判断是否有后继的责任对象
            * 如果有，就转发请求给后继的责任对象
            * 如果没有，则处理请求 */
            if(getSuccessor() != null) {
              　　System.out.println("放过请求");
              　　getSuccessor().handleRequest();
      　　    } else {
                System.out.println("处理请求");
            }
        }
    }

客户端类

    public class Client {
        public static void main(String[] args) {
          　　// 组装责任链
    　      　Handler handler1 = new ConcreteHandler();
          　　Handler handler2 = new ConcreteHandler();
          　　handler1.setSuccessor(handler2);
          　　// 提交请求
      　    　handler1.handleRequest();
        }
    }

### 应用实例：
 
1. 红楼梦中的"击鼓传花";
2. JS 中的事件冒泡。 
3. JAVA WEB 中 Apache Tomcat 对 Encoding 的处理，Struts2 的拦截器，jsp servlet 的 Filter。

### 优缺点

优点： 

1. 降低耦合度。它将请求的发送者和接收者解耦。 
2. 简化了对象。使得对象不需要知道链的结构。 
3. 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。 
4. 增加新的请求处理类很方便。

缺点： 

1. 不能保证请求一定被接收。 
2. 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。 
3. 可能不容易观察运行时的特征，有碍于除错。


### 使用场景

1. 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。 
2. 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。 
3. 可动态指定一组对象处理请求。



