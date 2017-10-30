## 状态模式

状态模式允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。

![](http://my.csdn.net/uploads/201205/11/1336719144_5496.jpg)

以上是状态模式的类图。它包含了的类主要可以分成三种类型：

1. 上下文（Context）: 定义了一个客户接口，就是整个状态模式所维护的一个对象。
2. 抽象状态类（State）: 定义了所有状态的顶层的类，它内部定义的方法是实际操作中需要用到的方法。
3. 具体状态类（ConcreteState）: 具体的状态实现，它实现了State接口。

如果，我们以一个闹钟的状态转换为例。那么上下文Context就是一个闹钟的对象，它不仅包含了闹钟需要的字段，同时还要维护闹钟所有的状态并在各个状态之间进行切换。而抽象状态中需要定义的就是闹钟对外的一些方法，比如开启、关闭、触发、睡眠等。而具体的状态类就是抽象状态的实现，此外它通常还要维护一个上下文对象，我们通过操作上下文对象来实现每个状态的逻辑。

### 示例

我们这里使用一个闹钟的状态转换的例子，闹钟总共包含禁用(DisabledState)、睡眠(SnoozedState)、触发(FiredState)、等待(WaitingState)状态，它们之间的转换关系是。闹钟禁用之后其他的状态都无法使用，闹钟启用之后就进入等待状态（等待响铃时间到来），当达到了闹钟的响铃时间的时候闹钟进入触发状态，触发状态之后可以选择让其睡眠5分钟或者暂停响铃。选择了睡眠之后进入睡眠状态，选择了暂停响铃之后就进入了等待状态。

#### 1.抽象的状态

首先给出的是抽象的状态

	public interface State {
	
	    /**
	     * 重新设置闹钟的时间
	     *
	     * @param hour 小时
	     * @param minute 分钟 */
	    void reset(int hour, int minute);
	
	    /**
	     * 让闹钟睡眠指定的时间段*/
	    void snooze();
	
	    /**
	     * 当闹钟已经响铃的时候撤销响铃 */
	    void cancel();
	
	    /**
	     * 使能闹钟 */
	    void enable();
	
	    /**
	     * 禁用闹钟 */
	    void disable();
	}

该类是抽象状态的定义类，可以考虑一下这里定义的方法的性质——它们都是用来提供给用户的操作闹钟的方法。并且当闹钟处于不同的状态的时候，各个方法的逻辑可能有所不同，其中也会涉及一些逻辑的转换。

#### 2.具体的状态实现

这里我们只给出闹钟处于触发状态时候闹钟的逻辑实现：

	public class FiredState implements State{
	    private AlarmMachine alarmMachine;
	
	    public FiredState(AlarmMachine alarmMachine) {
	        this.alarmMachine = alarmMachine;
	    }
	
	    @Override
	    public void reset(int hour, int minute) {
	        alarmMachine.setTime(hour, minute);
	        alarmMachine.setState(alarmMachine.getWaitingState());
	    }
	
	    @Override
	    public void snooze() {
	        int minute = alarmMachine.getSnoozeMinute();
	        if (alarmMachine.getMinute() + minute > 60) {
	            if (alarmMachine.getHour() + 1 > 23) {
	                alarmMachine.setNextTime(0,
	                        alarmMachine.getMinute() + minute - 60);
	            } else {
	                alarmMachine.setNextTime(alarmMachine.getHour() + 1,
	                        alarmMachine.getMinute() + minute - 60);
	            }
	        } else {
	            alarmMachine.setNextTime(alarmMachine.getHour(),
	                    alarmMachine.getMinute() + minute);
	        }
	        alarmMachine.setState(alarmMachine.getSnoozedState());
	    }
	
	    @Override
	    public void cancel() {
	        alarmMachine.setState(alarmMachine.getWaitingState());
	    }
	
	    @Override
	    public void enable() {
	        System.out.println("闹钟已经处于开启状态！");
	    }
	
	    @Override
	    public void disable() {
	        alarmMachine.setState(alarmMachine.getDisabledState());
	    }
	}

这里的AlarmMachine是闹钟的对象，我们使用它来修改闹钟的信息以及对闹钟的状态进行转换。

这里reset方法是重新设置闹钟的时间的意思，从这里我们可以看出当对闹钟重新设置了时间之后，首先，将设置的时间信息更新到闹钟实体中。然后，将闹钟的状态从当前的触发状态转换到等待状态。

这里的snooze方法是指当闹钟触发之后我们选择睡眠的操作，这时候我们会在原来的时间的基础之上增加睡眠的时间，并将增加之后的结果赋值到原来的闹钟的字段上面。当设置完了闹钟的时间之后，我们还要将闹钟的状态从当前的触发状态转换到睡眠状态。

接下来是cancel()方法，当闹钟处于触发状态的时候，如果我们选择了该方法，那么我们会直接更改当前闹钟的状态，使其进入等待响铃的状态，而无需对闹钟的时间进行更改。

对于enable方法，由于我们已经处于了使能状态，所以不需要为该方法添加逻辑。

最后是disable方法，当我们选择了该方法之后会将闹钟的状态设置成关闭状态。

#### 3.闹钟上下文

以下就是我们的闹钟的上下文：

	public class AlarmMachine {
	    private State snoozedState;
	    private State firedState;
	    private State waitingState;
	    private State disabledState;
	
	    /**
	     * 闹钟初始的状态是禁用的*/
	    private State state;
	
	    /**
	     * 闹钟初始的时间是0:00 */
	    private int hour = 0, minute = 0;
	
	    /**
	     * 下个响铃的时间 */
	    private int nextHour = 0, nextMinute = 0;
	
	    private int snoozeMinute = 5;
	
	    public AlarmMachine() {
	        snoozedState = new SnoozedState(this);
	        firedState = new FiredState(this);
	        waitingState = new WaitingState(this);
	        disabledState = new DisabledState(this);
	        state = disabledState;
	    }
	
	    public void fire() {
	        if (state == disabledState) {
	            System.out.println("闹钟没启用！");
	        } else {
	            state = firedState;
	        }
	    }
	
	    public void reset(int hour, int minute) {
	        state.reset(hour, minute);
	    }
	
	    public void snooze() {
	        state.snooze();
	    }
	
	    public void cancel() {
	        state.cancel();
	    }
	
	    public void enable() {
	        state.enable();
	    }
	
	    public void disable() {
	        state.disable();
	    }
	
	    public void setState(State state) {
	        this.state = state;
	    }
	
	    public void setTime(int hour, int minute) {
	        this.hour = hour;
	        this.minute = minute;
	    }
	
	    public void setNextTime(int hour, int minute) {
	        this.nextHour = hour;
	        this.nextMinute = minute;
	    }
	
	    // getter....
	}

这里我们定义了闹钟的所有的状态，而且还需要指定它的初始状态和初始时间。还要注意到这里我们定义了许多与抽象状态中定义相同的方法，而且在其中直接调用了当前状态的对应的方法来实现。至于其他的setter和getter方法没有太难理解的内容。

从这里我们也可以看出，实际上我们是在这里定义了闹钟的全部状态的实例，闹钟的状态之间的切换是在具体的状态之中完成的。这样我们只需要在上下文中定义全部状态并指定当前状态，然后可以在各个状态内部实现对应于各个状态的逻辑。

从直观上理解，它的好处是，我们只需要在各个状态类中实现，当处于这个状态中时各个方法该怎么做就可以了，逻辑的意义更加清晰。

### 总结

1. 我们直接将各个方法封装在Context中，用户不会与各个状态类交互，他们只需要与Context进行交互；
2. 与程序状态机(OSM)不同，状态模式使用类来表示各个状态；
3. Context会将行为委托给当前的状态；
4. 通过将每个状态封装进一个类，我们把以后需要做的任务给局部化了；
5. 状态模式和策略模式有相同的类图，但是它们的意图不同；
6. 使用状态模式通常会导致设计中类的数目大量增加。