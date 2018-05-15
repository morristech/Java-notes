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

------

## 1、Java 语言基础和JDK源码阅读

<h3 id="java">1. Java 语言基础</h3>

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

<h3 id="java_io">2. Java IO 模块</h3>

|编号|名称|
|:-:|:-:|
|1|[基本 IO](Java/IO/IO.md)|
|2|[NIO](Java/IO/NIO.md.md)|

<h3 id="java_con">3. Java 并发编程 模块</h3>

|编号|名称|
|:-:|:-:|
|1|[线程]()|
|2|[并发容器]()|
|3|[原子类]()|

<h3 id="java_collection">4. Java 集合框架 模块</h3>

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

1. [JVM概述](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/1.Java%E5%9F%BA%E6%9C%AC%E6%A6%82%E8%BF%B0.md)
2. [Java内存区域和内存溢出异常](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/2.Java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E5%92%8C%E5%86%85%E5%AD%98%E6%BA%A2%E5%87%BA%E5%BC%82%E5%B8%B8.md)
3. [垃圾收集器与内存分配策略](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/3.%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E4%B8%8E%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E7%AD%96%E7%95%A5.md)
4. [类文件结构](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/5.%E7%B1%BB%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84.md)
5. [虚拟机类加载机制](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/6.%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6.md)
6. [虚拟机字节码执行引擎](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/7.%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AD%97%E8%8A%82%E7%A0%81%E6%89%A7%E8%A1%8C%E5%BC%95%E6%93%8E.md)
7. [Java内存模型与线程](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/8.Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B.md)
8. [虚拟机性能监控与故障分析工具](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/4.%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%80%A7%E8%83%BD%E7%9B%91%E6%8E%A7%E4%B8%8E%E6%95%85%E9%9A%9C%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7.md)
9. [Eclipse Memory Analyzer 的使用](https://github.com/Shouheng88/Java-Programming/blob/master/JVM/Eclipse%20Memory%20Analyzer%20%E4%BD%BF%E7%94%A8.md)

<h2 id="database">6、数据库</h2>

|编号|名称|
|:-:|:-:|
|1|<a id="mysql" href="MySQL/MySQL.md">MySQL基础命令</a>|
|2|<a id="mysql_h" href="MySQL/Heigh_Performance_MySQL.md">高性能MySQL</a>|
|3|<a href="MySQL/Lock.md">乐观锁和悲观锁及其实现</a>|

## 7、Spring

1. [依赖注入](https://github.com/Shouheng88/Java-Programming/blob/master/Spring/Bean%E6%B3%A8%E5%85%A5.md)
2. [AOP](https://github.com/Shouheng88/Java-Programming/blob/master/Spring/AOP.md)
3. [Spring事务管理](https://github.com/Shouheng88/Java-Programming/blob/master/Spring/Spring%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86.md)
4. [Spring MVC](https://github.com/Shouheng88/Java-Programming/blob/master/Spring/Spring%20MVC.md)

## 8、MyBatis

1. [MyBatis映射](https://github.com/Shouheng88/Java-Programming/blob/master/MyBatis/MyBatis%E6%98%A0%E5%B0%84.md)

## 9、Tomcat

1. [《How Tomcat Works》阅读笔记1](https://github.com/Shouheng88/Java-Programming/blob/master/Tomcat/%E3%80%8AHow%20Tomcat%20Works%E3%80%8B%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B01.md)
2. [《How Tomcat Works》阅读笔记2](https://github.com/Shouheng88/Java-Programming/blob/master/Tomcat/%E3%80%8AHow%20Tomcat%20Works%E3%80%8B%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B02.md)
3. [《How Tomcat Works》阅读笔记3](https://github.com/Shouheng88/Java-Programming/blob/master/Tomcat/%E3%80%8AHow%20Tomcat%20Works%E3%80%8B%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B03.md)
4. [《How Tomcat Works》阅读笔记4](https://github.com/Shouheng88/Java-Programming/blob/master/Tomcat/%E3%80%8AHow%20Tomcat%20Works%E3%80%8B%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B04.md)
5. [《How Tomcat Works》阅读笔记5](https://github.com/Shouheng88/Java-Programming/blob/master/Tomcat/%E3%80%8AHow%20Tomcat%20Works%E3%80%8B%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B05.md)

## 10、高并发相关

1. [一致性哈希算法](https://github.com/Shouheng88/Java-Programming/blob/master/%E9%AB%98%E5%B9%B6%E5%8F%91%E7%9B%B8%E5%85%B3/%E4%B8%80%E8%87%B4%E6%80%A7%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95.md)

## 其他

1. [Linux命令](https://github.com/Shouheng88/Java-Programming/blob/master/linux%E5%91%BD%E4%BB%A4.md)
2. [正则表达式语法](https://github.com/Shouheng88/Java-Programming/blob/master/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F.md)
3. [进程间通信](https://github.com/Shouheng88/Java-Programming/blob/master/%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1.md)