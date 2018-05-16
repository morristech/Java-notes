# Java 笔记

> Java 的学习笔记和整理的知识点，包含Java语言基础、Java服务端方向的框架、设计模式、计算机网络、算法、Java 虚拟机和数据库等多个方面的内容。想了解前端的内容参考[这里](https://github.com/Shouheng88/Front-end-notes)，Android相关的内容参考[这里](https://github.com/Shouheng88/Android-notes)。

## 目录结构

1. 语言基础和JDK源码阅读
    1. <a href="#java">Java 语言基础</a>
    2. <a href="#java_io">Java IO 模块</a>
    3. <a href="#java_con">Java 并发编程 模块</a>
    4. <a href="#java_collection">Java 集合框架 模块</a>
2. <a href="#design_pattern">设计模式</a>
3. <a href="#network">计算机网络</a>
4. <a href="#data_structure">数据结构</a>
5. <a href="#JVM">Java 虚拟机</a>
6. <a href="#database">数据库</a>
    1. <a href="#mysql">MySQL</a>
	2. <a href="#mysql_h">高性能 MySQL</a>
	3. <a href="#redis">Redis</a>
	4. <a href="#mongodb">Mongodb</a>
7. 服务端框架
    1. <a href="#spring">Spring</a>
	2. <a href="#spring_mvc">Spring MVC</a>
	3. <a href="#mybatis">Mybatis</a>
8. Java第三方库
    1. <a href="#guava">Guava</a>
	2. <a href="#joda_time">Joda-time</a>
	3. <a href="#log4j">Log4j</a>
9. <a href="#java_8">Java 8</a>
10. <a href="#container">容器</a>
    1. <a href="#tomcat">Tomcat</a>

------

## 1、Java 语言基础和JDK源码阅读

<h3 id="java">Java 语言基础</h3>

|编号|名称|
|:-:|:-:|
|1|[基本](Java/Base.md)|
|2|[运算符](Java/Operator.md)|
|3|[数据类型](Java/DataTypes.md)|
|4|[类 对象 接口](Java/CIO.md)|
|5|[数组](Java/Array.md)|
|6|[枚举](Java/Enum.md)|
|7|[异常处理](Java/Exceptions.md)|
|8|[注解](Java/Anno.md)|
|9|[范型](Java/Generics.md)|

其他

|编号|名称|
|:-:|:-:|
|1|[Object类及其方法](Java/Other/Object.md)|
|2|[Class类](Java/Other/Class.md)|
|3|[字符串相关问题总结](Java/Other/Strings.md)|

<h3 id="java_io">Java IO 模块</h3>

|编号|名称|
|:-:|:-:|
|1|[基本 IO](Java/IO/IO.md)|
|2|[NIO](Java/IO/NIO.md.md)|

<h3 id="java_con">Java 并发编程 模块</h3>

|编号|名称|
|:-:|:-:|
|1|[线程]()|
|2|[并发容器]()|
|3|[原子类]()|

<h3 id="java_collection">Java 集合框架 模块</h3>

|编号|名称|
|:-:|:-:|
|1|[容器框架]()|

<h2 id="design_pattern">2、设计模式</h2>

|编号|名称|
|:-:|:-:|
|1|[基本：UML 面向对象设计六大原则等](DesignPattern/Basic.md)|
|2|[策略模式](DesignPattern/Strategy.md)|
|3|[装饰者模式](DesignPattern/Decorator.md)|
|4|[代理模式](DesignPattern/Proxy.md)|
|5|[工厂模式 简单工厂 抽象工厂](DesignPattern/Factory.md)|
|6|[原型模式](DesignPattern/Prototype.md)|
|7|[模板模式](DesignPattern/Template.md)|
|8|[适配器模式 外观模式](DesignPattern/Adapter_and_facade.md)|
|9|[构建者模式](DesignPattern/Builder.md)|
|10|[备忘录模式](DesignPattern/Memento.md)|
|11|[迭代器模式 组合模式](DesignPattern/Iterator_and_combination.md)|
|12|[桥接模式](DesignPattern/Bridge.md)|
|13|[命令模式](DesignPattern/Command.md)|
|14|[责任链模式](DesignPattern/Responsibility.md)|
|15|[单例模式](DesignPattern/Singleton.md)|
|16|[观察者模式](DesignPattern/Watcher.md)|
|17|[状态模式](DesignPattern/State.md)|
|18|[中介者模式](DesignPattern/Mediator.md)|
|19|[享元模式](DesignPattern/Flyweight.md)|
|20|[访问者模式](DesignPattern/Visitor.md)|
|21|[解释器模式](DesignPattern/Interpreter.md)|

<h2 id="network">3、计算机网络</h2>

|编号|名称|
|:-:|:-:|
|1|[计算机网络概述](Network/Internet.md)|
|2|[应用层](Network/Application_layer.md)|
|3|[网络层](Network/Network_layer.md)|
|4|[运输层](Network/Transport_layer.md)|
|5|[链路层](Network/Link_layer.md)|

<h2 id="data_structure">4、数据结构</h2>

### 算法的度量和基本数据结构

|编号|名称|
|:-:|:-:|
|1|[算法的度量和基本数据结构](DataStructure/Basic.md)|

### 排序

|编号|名称|
|:-:|:-:|
|1|[排序](DataStructure/Sort.md)|

### 查找

|编号|名称|
|:-:|:-:|
|1|[符号表查找](DataStructure/Search_label.md)|
|2|[二叉查找树](DataStructure/Search_binary.md)|
|3|[平衡查找树](DataStructure/Search_avl.md)|
|4|[散列表](DataStructure/Search_hash.md)|
|5|[多路查找树](DataStructure/Search_MWays.md)|

### 图

|编号|名称|
|:-:|:-:|
|1|[无向图](DataStructure/Graph_undirected_graph.md)|
|2|[图的遍历](DataStructure/Graph_ergodic.md)|
|3|[有向图](DataStructure/Graph_directed_graph.md)|
|4|[最小生成树](DataStructure/Graph_mst.md)|
|5|[最短路径](DataStructure/Graph_shortest.md)|

<h2 id="JVM">5、JVM虚拟机</h2>

|编号|名称|
|:-:|:-:|
|1|[JVM概述](JVM/1.Java基本概述.md)|
|2|[Java内存区域和内存溢出异常](JVM/2.Java内存区域和内存溢出异常.md)|
|3|[垃圾收集器与内存分配策略](JVM/3.垃圾收集器与内存分配策略.md)|
|4|[类文件结构](JVM/5.类文件结构.md)|
|5|[虚拟机类加载机制](JVM/6.虚拟机类加载机制.md)|
|6|[虚拟机字节码执行引擎](JVM/7.虚拟机字节码执行引擎.md)|
|7|[Java内存模型与线程](JVM/8.Java内存模型与线程.md)|
|8|[虚拟机性能监控与故障分析工具](JVM/4.虚拟机性能监控与故障分析工具.md)|
|9|[Eclipse Memory Analyzer 的使用](JVM/EclipseMemoryAnalyzer使用.md)|

<h2 id="database">6、数据库</h2>

|编号|名称|
|:-:|:-:|
|1|<a id="mysql" href="MySQL/MySQL.md">MySQL基础命令</a>|
|2|<a id="mysql_h" href="MySQL/Heigh_Performance_MySQL.md">高性能MySQL</a>|
|3|<a href="MySQL/Lock.md">乐观锁和悲观锁及其实现</a>|

## 7、服务端框架

<h3 id="spring">Spring</h3>

1. [依赖注入](Spring/Bean注入.md)
2. [AOP](Spring/AOP.md)
3. [Spring事务管理](Spring/Spring事务管理.md)

<h3 id="spring_mvc">Spring MVC</h3>

[Spring MVC](Spring/SpringMVC.md)

<h3 id="mybatis">MyBatis</h3>

[MyBatis映射](MyBatis/MyBatis映射.md)

<h2 id="container">10、容器</h2>

<h3 id="tomcat">Tomcat</h3>

|编号|名称|
|:-:|:-:|
|1|[《How Tomcat Works》笔记1](Tomcat/HTW_note1.md)|
|2|[《How Tomcat Works》笔记2](Tomcat/HTW_note2.md)|
|3|[《How Tomcat Works》笔记3](Tomcat/HTW_note3.md)|
|4|[《How Tomcat Works》笔记4](Tomcat/HTW_note4.md)|
|5|[《How Tomcat Works》笔记5](Tomcat/HTW_note5.md)|

## 其他

1. [Linux命令](linux命令.md)
2. [正则表达式语法](正则表达式.md)
3. [进程间通信](进程间通信.md)
4. [一致性哈希算法](一致性哈希算法.md)