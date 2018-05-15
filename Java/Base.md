# 基本

> 注：下面的文中所有的规范来自阿里的Java开发规范。

## 1、术语

|编号|结论|
|:-:|:-:|
|1|**JDK**: 编写Java程序的程序员使用的软件|
|2|**JRE**: 运行Java程序的用户使用的软件|
|3|Java的执行模式是**编译和解释型**，Java程序首先由编译器转换为标准字节代码，然后由JVM来解释执行，**JVM**将字节代码程序同操作系统和硬件分开，使Java程序独立于平台运行|
|4|**垃圾回收机制**是一个系统线程，对内存使用进行跟踪|
|5|Class类对象由Java编译器自动生成，隐藏在.class文件中|
|6|JVM运行Java代码的步骤: **装载-链接-初始化**（运行时不需要加载代码）|
|7|**动态绑定**: 在运行时能够自动地选择调用哪个方法的现象|
|8|**Jar**是一种压缩的文件，使用了ZIP格式，其中包含一个清单文件MANIFEST.MF位于META-INF目录|
|9|**JavaScript**是网页中使用的脚本语言，语法类似于Java，除此之外无任何联系|
|10|Java大小写敏感|

## 2、Java程序的基本结构

	package me.shouheng;
	
	import java.util.ArrayList;
	import java.util.List;
	
	public class Test {
	    public static void main(String ...args) {
	        System.out.println("Hello world!");
	        List<String> s = new ArrayList<String>();
	    }
	}

简单说明：

|编号|说明|
|:-:|:-:|
|1|main方法用于控制程序的开始和结束，main方法包含以下要素：public、static、void和String[] args|
|2|可以使用new关键字创建类的实例对象，可以直接通过类名调用类所提供的静态成员|

### 包

|编号|说明|
|:-:|:-:|
|1|包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。【规范】|
|2|package语句必须是源程序文件中第一个非注释、非空白语句|
|3|包的成员可以包含子包、类、接口。如果源代码中没有指定包，则使用默认包|
|4|一个包（子包）的成员不能重名，即使不同的类型|

包的访问方式：

1. 完全限定名称方式访问：`java.net.InetAddress hostAdd`
2. 导入包成员：`import java.net.InetAddress`
3. 导入整个包：`import java.net.*`
4. 导入**类**的静态成员：`import static java.lang.Math.*`
5. 访问包成员名称冲突：需要使用完全限定名称方式

## 3、标识符

由**字符、数字、下划线、美元符号($)组成，且开头不能为数字，分大小写，没有长度限制，不能与关键字相同**. 约定如下

|编号|元素|约定|
|:-:|:-:|:-:|
|1|包|常用名词，全部小写|
|2|类、接口|常用名词，每个单词首字母大写；（PascalCase规则）|
|3|方法|常用动词，首字母小写；（camelCase规则）|
|4|常量|常用名词，全部大写，单词之间用下划线(_)分隔|
|5|变量|成员名称，首字母小写. （camelCase规则）|

规范：

|编号|规范|
|:-:|:-:|
|1|【强制】代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束|
|2|【强制】代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式|
|3|【强制】类名使用 UpperCamelCase 风格，必须遵从驼峰形式，但以下情形例外：DO/BO/DTO/VO/AO|
|4|【强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长|
|5|【强制】抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类命名以它要测试的类的名称开始，以Test结尾|
|6|【强制】POJO 类中布尔类型的变量，都不要加is ，否则部分框架解析会引起序列化错误|
|7|【推荐】如果使用到了设计模式，建议在类名中体现出具体模式|
|8|【推荐】 如果是形容能力的接口名称，取对应的形容词做接口名（ 通常是–able的形式）|
|9|【强制】对于 Service 和 DAO 类，基于 SOA 的理念，暴露出来的服务一定是接口，内部的实现类用 Impl 的后缀与接口区别|

此外，Service / DAO 层方法命名规约

1. 获取单个对象的方法用get做前缀。
2. 获取多个对象的方法用 list 做前缀。
3. 获取统计值的方法用 count 做前缀。
4. 插入的方法用 save（ 推荐 ） 或 insert 做前缀。
5. 删除的方法用 remove（ 推荐 ） 或 delete 做前缀。
6. 修改的方法用 update 做前缀。

## 2、程序流程

### 2.1 基本程序结构

1. 顺序结构
2. 选择结构：if语句和switch语句
3. 循环结构：for循环、while循环、do...while循环、for each循环
4. 跳转语句：break语句、continue语句、return语句

#### 两个需要注意的地方

1. 在Java中for each循环体语句序列中，数组或集合的元素是只读的，其值不能改变，若要改变可用常规for循环.
2. 应该强制在使用for循环的时候加上`{}`。下面的程序示例：

假如有一段程序：

    for (int i=0;i<10;i++) int k = 5;
    for (int i=0;i<10;i++) {int k = 5;}

上面的两种写法：第一种是错误的，for循环可以不加{}，但是那仅限于执行语句，不能包含变量声明语句。第一段代码定义了k，它的作用域是包含for循环的整个方法，因此会造成重复定义的编译错误。

#### foreach

foreach循环使用了迭代器设计模式，它的基本使用方法：如果想要自己的数据集合能够使用foreach的方式遍历集合的元素，那么我们需要自己的容器实现Iterable接口。下面是在为背包变成可迭代的列子：

	public class Bag<E> implements Iterable<E>{
	    private Node<E> first;
	
	    private static class Node<E> {
	        E element;
	        Node<E> next;

	        Node(E element, Node<E> next) {
	            this.element = element;
	            this.next = next;
	        }
	    }
	
	    public void add(E element) {
	        first = new Node<E>(element, first);
	    }
	
	    public Iterator<E> iterator() {
	        return new ListIterator();
	    }
	
	    private class ListIterator implements Iterator<E> {
	        private Node<E> current = first;
	
	        @Override
	        public boolean hasNext() {
	            return current != null;
	        }
	
	        @Override
	        public E next() {
	            E element = current.element;
	            current = current.next;
	            return element;
	        }
	
	        @Override
	        public void remove() {}
	    }
	}
