## 策略模式

**定义了算法簇，分别封装起来，让它们之间可以相互替换，此种模式让算法的变化独立于使用算法的客户。**

![](http://upload.ouliu.net/i/201710291354489czeh.png)

以上图为例，如果某部分逻辑比较容易变动，我们可以将其定义成一个接口(IA)。然后根据情景的需求，对该接口采取不同的实现策略(A1, A2)。在实际使用中，我们只定义该接口的实例(即使用IA定义a)，即所谓的“针对接口编程0”的策略，然后根据要求实例化一个需要的类型传入即可(将实例化的A1或者A2作为a的实现)。

### 示例

作为一个基本的例子，我们来看策咯模式在Android中的一个应用。我们知道在Android当中当向系统注册一个闹钟的时候根据API的不同会有多种不同的实现方式。那么，在这里我们可以将这些实现方式定义成一个策略接口，然后根据不同的API采取不同的实现策略，最后再根据API来获取对应的实现策略。

1.首先是一个策略接口的定义

    public interface AlarmSettingStrategy {
        void setRTCAlarm(Alarm alarm, PendingIntent sender);
    }

2.然后定义它的三个不同的实现。这里的实现是作为一个类的内部类来实现的，所以定义在了一个方法中：

    private final class IceCreamSetter implements AlarmSettingStrategy {
        @Override
        public void setRTCAlarm(Alarm alarm, PendingIntent sender) {
            alarmManager.set(AlarmManager.RTC_WAKEUP, alarm.getNextTime().getTimeInMillis(), sender);
        }
    }

    @TargetApi(Build.VERSION_CODES.KITKAT)
    private final class KitKatSetter implements AlarmSettingStrategy {
        @Override
        public void setRTCAlarm(Alarm alarm, PendingIntent sender) {
            alarmManager.setExact(AlarmManager.RTC_WAKEUP, alarm.getNextTime().getTimeInMillis(), sender);
        }
    }

    @TargetApi(23)
    private final class MarshmallowSetter implements AlarmSettingStrategy {
        @Override
        public void setRTCAlarm(Alarm alarm, PendingIntent sender) {
            alarmManager.setExact(AlarmManager.RTC_WAKEUP, alarm.getNextTime().getTimeInMillis(), sender);
        }
    }

3.在类AlarmsManager内部，我们来向系统中注册闹钟。那么，这里我们定义一个注册闹钟的策略

    private final AlarmSettingStrategy alarmSettingStrategy;

然后在代码中来根据API对其采取不同的实现策略。这里我们将获取实例的方法封装成一个工厂方法(也是在AlarmsManager内部定义)，并由其来返回指定API的实例。

    private AlarmSettingStrategy initSetStrategyForVersion() {
        if (PalmUtils.isMarshmallow()){
            return new MarshmallowSetter();
        } else if (PalmUtils.isKitKat()) {
            return new KitKatSetter();
        } else {
            return new IceCreamSetter();
        }
    }

### 总结

以上就是关于策略模式的一些基本的内容。还是比较容易理解的，即对某个逻辑将其定义成一个策略接口，然后根据要求对其采用不同的策略实现，并根据实际要求选择符合要求的策略实现即可。

它的好处是将某部分代码封装成独立的模块，那么如果我们要对这部分的逻辑进行修改，我们只需要选择一个符合要求的策略实现赋值给对应的策略实例即可。

