# 异常处理

异常为我们提供了一种解决问题的机制，有了它我们就可以实现将业务逻辑和异常处理相分离，从而使我们的代码更加易于维护和理解。

## 1、Java异常的结构层次

![Java异常的结构层次](res/exceptions.png)

1. 所有的异常类是从java.lang.Exception类继承的子类。
2. Exception类是Throwable类的子类。除了Exception类外，Throwable还有一个子类Error，用来指示运行时环境发生的错误，我们最好不要在程序中定义Error的子类。
3. Java程序通常不捕获错误。错误一般发生在严重故障时，它们在Java程序处理的范畴之外。
4. 异常类有两个主要的子类：IOException类和RuntimeException类。
5. Java提供了三种可抛出结构：受检异常、运行时异常和错误。
6. 每个抛出的异常都要有文档

对于使用受检的异常还是未受检的异常的原则：**对于可恢复的情况，使用受检的异常；对于程序错误，使用运行时异常。**

运行时异常（可以参考上图）通常意味着程序不符合API规范，继续运行下去也无济于事。而受检异常则意味着，对出现了异常进行正确的处理之后，程序仍然能够继续执行。

## 2、异常处理程序的结构

    try {
        int k = 1/0;
    } catch (ArithmeticException e) {
        e.printStackTrace();
    } finally {
        System.out.println("finally");
    }

1. 异常处理系统会按照“就近”原则匹配异常，如果满足了匹配就不再继续向下匹配。因此，在catch语句中捕获异常的时候，要按照从子类到基类的顺序捕获（从具体错误到顶层错误）。
2. **异常链返回**：finally语句总是会执行，所以在一个方法中可以从多个点返回，却仍然能够保证清理工作的进行。

	    public static void main(String ...args) {
	        for (int i=1;i>=0;i--) {
	            System.out.println(f(i));;
	        }
	    }
	
	    private static int f(int i) {
	        try {
	            int k = 1 / i;
	            return 0;
	        } catch (ArithmeticException e) {
	            System.out.println("catch");
	            return 1;
	        } finally {
	            try {
	                Thread.sleep(1000);
	                System.out.println("finally");
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	    }

	以上程序的输出结果是：
	
		finally
		0
		catch
		finally
		1

	可见，finally总是会被调用，而且return语句要等待finally执行完毕才会调用。

## 3、异常信息丢失
	
如果在try语句块中已经出现了异常，而我们在finally语句块中进行异常处理的时候又抛出了异常，或者使用了return语句，那么这个时候我们的第一个抛出的异常会丢失。
丢失的后果就是当程序出现了错误的时候，我们没有办法对错误发生的位置进行定位。

## 4、异常使用指南

建议使用异常的情况：

1. 在恰当的级别处理问题（知道该如何处理的情况下捕获异常）；
2. 解决问题并且重新调用产生异常的方法；
3. 进行少许修补，然后绕过异常发生的地方继续执行；
4. 用别的数据进行计算，以代替方法预计会返回的值；
5. 把当前运行环境下能做的事情尽量做完，然后把相同的异常抛到高层；
6. 把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到高层；
7. 终止程序；
8. 进行简化；
9. 让类库和程序更安全。

注意：

1. 异常应该只用于异常的情况；它们永远不应该用于控制正常的控制流。
