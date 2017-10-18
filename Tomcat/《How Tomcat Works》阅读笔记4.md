## Session管理

##### 1.Session对象

Session的管理是通过Session管理器来完成的，管理器使用Manager接口表示，Session管理器需要且必须一个Context容器关联。Servlet通过HttpServletReqest接口的getSession()方法来获取一个Session对象。相关的逻辑是在HttpRequestBase中实现的：

    private HttpSession doGetSession(boolean create) {
        // ...
        Manager manager = null;
        if (context != null) manager = context.getManager();
        if (manager == null) return (null);

        if (requestedSessionId != null) {
            try {
                session = manager.findSession(requestedSessionId);
            } catch (IOException e) {
                session = null;
            }
            if ((session != null) && !session.isValid()) session = null;
            if (session != null) return (session.getSession());
        }

        if (!create) return (null);
        if ((context != null) && (response != null) &&context.getCookies() && response.getResponse().isCommitted()) {
            throw new IllegalStateException(sm.getString("httpRequestBase.createCommitted"));
        }

        session = manager.createSession();
        if (session != null) return (session.getSession());
        else return (null);
    }

所以，这里是从Context中获取Manager，然后使用Manager的方法来创建或者查找Session. 

默认，情况下Session会被存储在内存中，但是Session管理器也可以将Session进行持久化操作，存储到数据库或者文件中。

在Catalina中定义了Session接口，并提供了一个标准实现StandardSession，它同时实现了Session 和HttpSession，它还有一个外观类StandardSessionFacade. 

##### 2.Manager

Catalina中使用定义了Manager接口用于Session管理器。它内部的几个比较重要的方法有：

1. getContainer()/setContainder(): 将Manager和Context容器关联；
2. createSession(): 创建Session实例；
3. add()/remove(): 向Session池中添加或者移除Session实例；
4. getMaxInactiveInterval()/setMaxInactiveInterval()： 设置或者获取Session存活的时间长度；
5. load()/unload()：将Session对象持久化或者从持久化设备加载Session到内存中。 

Manager接口有多个实现。首先是ManagerBase，它内部实现了所需的大部分方法，并使用一个HashMap对Session进行缓存。ManagerBase还有两个子类分别是StandardManager和PersistentManagerBase。

1. StandardManager实现了Lifecycle接口，在strt()中使用load()加载Session，在stop()中使用unload()对Session进行持久化。此外，它还实现了Runnable，在调用了它的start()方法的时候会同时开启线程用于对失效的Session对象进行销毁，可以通过设置checkInterval来指定的检查的时间间隔。
2. PersistentManagerBase有两个子类，PersistentManager和DistributeManager. 

##### 3.程序的执行逻辑

本次程序相比于之前的程序改动的地方不多，

1. 首先，在Bootstrap中，在创建完了StandardContext的实例之后会创建一个StandardManager实例并将其赋值给Context. 
2. 然后是在基础阀SimpleWrapperValve中，调用阀所在的容器的父容器(即Context类型的容器)，并将其作为参数传入到HTTP请求中，因为我们在创建Session的时候会需要用它。
3. 接下来是在Servlet中，尝试调用HTTP请求的getSession()方法类获取Session，它又会调用上面的代码中的doGetSession()来尝试创建Session. 在上面的代码中也可以看出，它需要使用Context来获取Manager，然后使用该Manager，或者新建一个Session或者根据id去查找Session.(这部分代码在ManagerBase中实现) 
4. 另外，StandardManager还有stop()和start()方法，并在它们内部分别实现将Session持久化到文件中，以及从文件中读取持久化的Session的功能，依次来达到记住某个用户的目的。


