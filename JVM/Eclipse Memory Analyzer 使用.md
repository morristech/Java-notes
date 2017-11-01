# Eclipse Memory Analyzer 的使用

## 1、配置

### 1.1 内存资源分配

安装完成之后，为了更有效率的使用MAT，我们还需要做一些配置工作。因为通常而言，分析一个堆转储文件需要消耗很多的堆空间，为了保证分析的效率和性能，在有条件的情况下，我们会建议分配给 MAT 尽可能多的内存资源。你可以采用如下两种方式来分配内存更多的内存资源给 MAT：

1. 修改启动参数 MemoryAnalyzer.exe -vmargs -Xmx4g
2. 编辑文件 MemoryAnalyzer.ini，在里面添加类似信息 -vmargs – Xmx4g。

## 2、获得堆转储文件

通常来说，只要你设置了如下所示的 JVM 参数：

    -XX:+HeapDumpOnOutOfMemoryError

JVM 就会在发生内存泄露时抓拍下当时的内存状态，也就是我们想要的堆转储文件。

示例程序：

    static class OOMObject {}

    public static void main(String...args) {
        List<OOMObject> list = new ArrayList<OOMObject>();
        while (true) {
            list.add(new OOMObject());
        }
    }

VM启动参数是：

    -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError

当出现异常的时候会有如下输出：

	java.lang.OutOfMemoryError: Java heap space
	Dumping heap to java_pid5540.hprof ...
	Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
		at java.util.Arrays.copyOf(Arrays.java:3210)
		at java.util.Arrays.copyOf(Arrays.java:3181)
		at java.util.ArrayList.grow(ArrayList.java:261)
		at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
		at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
		at java.util.ArrayList.add(ArrayList.java:458)
		at oom.HeapOOM.main(HeapOOM.java:16)
	Heap dump file created [28363941 bytes in 0.243 secs]

这里`Heap dump file created`说明生成了堆转储文件，并被保存在项目的根目录下面。

## 3、生成分析报告

启动前面安装配置好的 Memory Analyzer tool , 然后选择菜单项 File- Open Heap Dump 来加载需要分析的堆转储文件。

### 3.1 生成分析报告

![](http://upload.ouliu.net/i/20171031145325awo0m.png)

如图所示，当我们将堆转存文件导入到了应用中之后，点击`Leak Suspects`可以生成一个分析文件，并作为zip格式被保存到项目的目录下面。它使用HTML来展示分析的信息，方便浏览，可以用来共享给其他同事。

也可以直接在工具栏上面选择`Leak Suspects`输出分析文件：

![](http://upload.ouliu.net/i/20171031150249juiou.png)

除了上面的`Leak Suspects`之外，还有选项`Top Components`也是用来生成分析报告的。所以，可以生成的分析报告有：

![](http://upload.ouliu.net/i/20171031150824nsw3y.png)

## 4.分析报告

### 4.1 步骤1 内存消耗的整体状况

![](http://upload.ouliu.net/i/201710311520448yl92.png)

该图显示的就是内存消耗的整体状况。在`Problem Suspect 1`中给出了关于该可疑对象的描述。

这个图是一个概要图，显示了一些统计信息，包括整个size大小，class数量，以及对象 的数量，同时还将生成一个大对象的top图，并线显示大对象占用内存的百分比。

### 4.2 步骤2 分析问题的所在

#### 4.2.1 查看可疑详情

在`Problem Suspect 1`中继续向下可以找到Details链接，点击该链接显示获取关于内存分析的详细报告：

![](http://upload.ouliu.net/i/20171031152750vs0pn.png)

从上面的图中可以看出，该详细描述中包含的内容比较多，这更方便我们分析问题。

#### 4.2.2 类内存占用柱状图

![](http://upload.ouliu.net/i/20171031153600s7g63.png)

如图，我们可以选择工具栏中的按钮，点击它之后，就可以为我们生成Histogram报告。该报告列出每個class产生了多少個实例，以及占有多大内存，所占百分比等信息。

只要有溢出，时间久了，溢出类的实例数量或者其占有的内存会越来越多，排名也就越来越前，通过多次对比不同时间点下的Histogram图对比就能很容易把溢出类找出来。

`Shallow Size`就是对象本身占用内存的大小，不包含对其他对象的引用，也就是对象头加成员变量（不是成员变量的值）的总和。

`Retained Size`是该对象自己的`Shallow Size`，加上从该对象能直接或间接访问到对象的`Shallow Size`之和。换句话说，`Retained Size`是该对象被GC之后所能回收到内存的总和。

#### 4.2.3 Dominator Tree（支配树）

![](http://upload.ouliu.net/i/20171031154049vyqhq.png)

如图是点击了工具栏上面的指定按钮之后就会显示一个支配树信息列表。支配树列出了每个对象(Object instance)与其引用关系的树状结构，还包含了占有多大内存，所占百分比。

一般发生内存泄漏的时候，都会直接锁定那个体量最大的对象。接下来一步步接近真相：展开最大对象的子树，试着找到retained size最接近最大对象的子节点（通常是一个数组或者集合）。

#### 4.2.4 定位溢出原因

通 过Histogram视图或者Dominator Tree视图，找到疑似溢出的对象或者类后，选择Path to GC Roots或者Merge Shortest Paths to GC Roots，这里有很多过滤选项，一般来讲可以选择exclude all plantom/weak/soft etc. references。这样就排除了虚引用、弱引用、以及软引用，剩下的就是强引用。从GC上说，除了强引用外，其他的引用在JVM需要的情况下是都可以 被GC掉的，如果一个对象始终无法被GC，就是因为强引用的存在，从而导致在GC的过程中一直得不到回收，因此就内存溢出了。

![](http://upload.ouliu.net/i/20171031154632vh5fx.png)

接下来就需要直接定位具体的代码，看看如何释放这些不该存在的对象，比如是否被cache住了，还是其他什么原因。

找到原因，清理干净后，再对照之前的操作，看看对象是否还再持续增长，如果不在，那就说明这个溢出点被成功的堵住了。

最后用jstat跟踪一段时间，看看Old和Perm区的内存是否最终稳定在一个范围内，如果长时间稳定在一个范围，那溢出的问题就解决了，如果还再继续增长，那继续用上述方法，看看是否存在其他代码的溢出点，继续找出，将其堵住。

 
