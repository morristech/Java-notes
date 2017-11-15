# 算法的度量和基本数据结构

## 1、数据结构

数据结构是相互之间存在一种或多种关系的数据的集合。

### 1.1 三要素

1. 数据结构三要素是：1.数据的逻辑结构；2.数据的物理结构；3.数据的运算。
2. 数据结构是相互之间存在一种或多种特定关系的数据元素的集合。

#### 1.1.1 逻辑结构 

分为线性结构和非线性结构，

1. 线性结构：线性表、栈、队列
2. 非线性结构：树、图、集合

#### 1.1.2 存储结构 

即物理结构，分四种（存储数据时不只要存储元素的值，还要存储数据元素间的关系）

1. 顺序存储：存储位置在物理上连续
2. 链接存储：链表形式（不同结点存储空间不一定连续，但是结点内存储单元地址必须连续）
3. 索引存储：使用1个数组记录索引
4. 散列存储：根据元素关键字计算该元素存储地址，如Hash存储

#### 1.1.3 逻辑结构

1. 集合结构：数据元素除了同属于一个集合之外，没有其他关系；
2. 线性结构：数据元素之间是一对一的关系；
3. 树形结构：数据元素之间存在一种多对多的层次关系；
4. 图形结构：数据元素之间是多对多的关系；

### 1.2 数据类型

1. 原子类型：不可再分的数据类型
2. 结构类型：其值可以再分成若干成分的数据类型
3. 抽象数据类型(ADT)：一个数学模型以及定义在该模型之上的一组操作，通常用数据对象、数据关系、基本运算操作集这样三元组表示

## 2、算法的度量

### 2.1 时间复杂度

时间复杂度就是程序逻辑执行的次数。通常在求解时间复杂度的时候，会对其进行简化。下面是推导大O阶的方法：

1. 用常数1取代运行时间中的所有加法常数
2. 在修改后的运行次数函数中，只保留最高阶项；
3. 如果最高阶项存在且不是1，则去除与这个项相乘的常数。

常见的时间复杂度：

#### 2.2.1 常数阶

    int num = 0, n = 100;
    num = (1 + n) * n / 2;
    printf(num);

主义上面的时间复杂度是常数阶O(1)，而不是O(3).

#### 2.2.2 线性阶

    for(int i=0;i<n;i++) {
        // 执行时间复杂度为O(1)的操作
    }

上面的时间复杂度是O(n).

#### 2.2.3 对数阶

    int cnt = 1;
    while (cnt < n) {
        cnt *= 2;
        // 执行时间复杂度为O(1)的操作
    }

上面的时间复杂度是O(logn).

#### 2.2.4 平方阶

    int i, j;
    for (int i=0;i<n;i++) }
        for (int j=0;j<n;j++) {
           // 执行时间复杂度为O(1)的操作
        }
    }

上面的计算的时间复杂度是O(n<sup>2</sup>)

    int i, j;
    for (int i=0;i<n;i++) }
        for (int j=0;j<m;j++) {
           // 执行时间复杂度为O(1)的操作 
        }
    }

上面的计算的时间复杂度是O(nm)

下面的程序的时间复杂度也是O(n<sup>2</sup>):

    int i, j;
    for (int i=0;i<n;i++) }
        for (int j=i;j<n;j++) {
           // 执行时间复杂度为O(1)的操作
        }
    }

执行的次数，O(f(n))，其中f(n)是执行的次数，表示执行时间与f(n)成正比
时间复杂度的大小关系：

	O(1)<O(log2n)<O(n)<O(nLogn)<O(n^2)<O(n^3)<O(2^n)<O(n!)<O(nn)

### 2.2 空间复杂度

所需的物理存储空间

    题目：算法
    void fun(int n){
        int i=1;
        while(i<=n) {
            i*=2;
        }
    }
    的时间复杂度是？
    思路：函数中运算次数最多的是i*=2; 这一行，那么假设它执行了t次，t次时i=2^t。
    因此，有:2^t<=n，于是得t<=log2n ,于是可得O(log2n)

## 3、线性表

线性表是具有相同数据类型的n个数据元素的有限序列。

### 3.1 顺序表

线性表的顺序存储，即存储在一组连续的存储单元里，如数组。顺序表可以是静态分配的，也可以是动态分配的。动态分配的，比如用指针指示动态数组，静态分配的是创建数组的时候就指定数组的大小。

注意，在线性表当中插入或者删除数据是要移动其他元素的，而访问的是否直接使用索引访问即可。所以，对于线性表访问第i个位置的元素的时间复杂度为O(1)，在第i个位置插入元素的时间复杂度为O(n). 

### 3.2 单链表

#### 3.2.1 定义

普通的链表，定义的形式是

	public class LinkedList<E> {
		transient int size = 0;
        transient Node<E> first;

        private static class Node<E> {
	        E item;
	        Node<E> next;
	
	        Node(E element, Node<E> next) {
	            this.item = element;
	            this.next = next;
	        }
        }
	}

这里使用泛型的来代表链表的每个节点中存储的数据，使用内部类Node类定义链表的一个节点。

#### 3.2.2 单链表的操作

1. 每次建立链表时将结点插入到头部：建立一个长度为n的链表的时间复杂度是O(n)
2. 每次建立链表时将结点插入到尾部：建立一个长度为n的链表的时间复杂度是O(n)
3. 插值节点：从头结点出发，顺着next指针往下找，时间复杂度为O(n)
4. 按值查找表结点：时间复杂度为O(n)
5. 插入和删除结点操作：时间复杂度为O(1)
6. 求结点长度：时间复杂度为O(n)，遍历，每次增加1

### 3.3 双向链表

#### 3.3.1 定义

下面的是一份基于Java的双向链表的实现：

	public class LinkedList<E> {
    	transient int size = 0;
    	transient Node<E> first;
    	transient Node<E> last;
	
	 	private static class Node<E> {
	        E item;
	        Node<E> next;
	        Node<E> prev;
	
	        Node(Node<E> prev, E element, Node<E> next) {
	            this.item = element;
	            this.next = next;
	            this.prev = prev;
	        }
    	}
	}

### 3.4 静态链表

借助数组类描述链表，结点有数据域data和指针域next，描述（这种设计的好处是适用于不支持指针的计算机语言）

### 3.5 栈

#### 3.5.1 定义

栈是一种**后进先出**的数据结构，下面是一种使用单向链表实现的栈：

	public class Stack<E>{
	    private int size;
	    private Node<E> first;
	
	    private static class Node<E> {
	        E element;
	        Node<E> next;
	
	        Node(E element, Node<E> next) {
	            this.element = element;
	            this.next = next;
	        }
	    }
	
	    public void push(E element) {
	        first = new Node<E>(element, first);
	        size++;
	    }
	
	    public E pop() {
	        if (size == 0) {
	            throw new UnsupportedOperationException("The stack is empty.");
	        }
	        size --;
	        Node<E> oldFirst = first;
	        first = oldFirst.next;
	        return oldFirst.element;
	    }
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	}

#### 3.5.2 题目：

	题1：3个不同的元素依次进栈，能得到几种不同的出栈序列？
	5种，abc acb bac bca cba，最后一种如果以c开头，那么ab必然已经存入了栈中，取出的顺序只能是ba，即以c开头的只有cba. 
	
	题2：a b c d e f 以所给的顺序依次进栈，若在操作时允许出栈，这得不到的序列为？
	A fedbca	B bcafed	C dcefba	D cabdef
	这种题应该从每个选项的第一个字母入手，以C为例，如果d处在第一个，那么说明前面的a b c肯定已经存在栈中，那么它们必然按照c b a的顺序出来；
	如果题目中的c b a的出现次序不对，那么就得不到。然后再使用相同的思路判断第二个字符的情况。答案D

### 3.6 队列

#### 3.6.1 普通队列

队列也是一种线性表，特性是**先入先出**，队列和栈的主要区别是插入、删除操作的限定不一样。下面是基于Java的一种使用链表来实现的队列：

	public class Queue<E> {
	    private Node<E> first, last;
	    private int size;
	
	    private static class Node<E> {
	        E element;
	        Node<E> next;
	
	        Node(E element, Node<E> next) {
	            this.element = element;
	            this.next = next;
	        }
	    }
	
	    public void enqueue(E element) {
	        size++;
	        Node<E> node = new Node<E>(element, null);
	        if (last == null) {
	            first = node;
	            last = node;
	            return;
	        }
	        last.next = node;
	        last = node;
	    }
	
	    public E dequeue() {
	        if (size == 0) {
	            throw new UnsupportedOperationException("The queue is empty.");
	        }
	        size--;
	        E element = first.element;
	        first = first.next;
	        return element;
	    }
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	}


#### 3.6.2 双端队列

队首和队尾都允许入队和出队的队列。

#### 3.6.3 应用: 

1. 栈在括号匹配中的应用：与``[([][])]``类似的括号匹配的问题，遇到一个左括号，判断是否合法，若合法就先存在栈中，等待右括号出现，看是否匹配；
2. 栈在表达式求值中的应用：中缀：``A+B*(C-D)-E/F``，后缀：``ABCD-*+EF/-``，然后按照计算的规则，依次进栈、出栈即可求得表达式的结果；
3. 栈在递归中的应用：递归函数在求解的时候，要不断返回结果、传入参数等，因此效率不高，可以借助栈将递归问题转换为非递归问题；
4. 队列在层次遍历中的应用:比如遍历二叉树等；
5. 队列在计算机系统中的应用：用于任务分配，当任务无法立刻全部完成的时候，对无法立刻完成的任务，先将其存入到队列中，等待其他任务完成之后再去执行这些任务。

### 3.7 背包

**背包**是一种不支持从中删除元素的集合数据类型，它的目的就是**帮助用例收集并迭代遍历所有收集到的元素**。使用背包的很多场景可能使用栈或者队列也能实现，但是使用背包可以说明元素存储的顺序不重要。下面的是一份基于Java的背包的实现，在这里我们只是在之前栈的代码的基础之上做了一些修改，并让其实现Iterable接口以在foreach循环中使用背包遍历元素：

	public class Bag<E> implements Iterable<E>{
	    private int size;
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
	        size++;
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
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	}

## 4、非线性表

### 4.1 树

#### 4.1.1 概念

1. 根：树的最上层的顶点；
2. 父结点：某个节点上面的节点；
3. 祖先结点：父节点的父节点等；
4. **结点的度**：树中结点的**子结点个数**；
5. **树的度**：树中**结点的最大度数**；
6. 分支结点：度大于0的结点；
7. **叶子结点**：度等于0的结点；
8. 树的高度：树中结点的最大层数
9. 路径：树中两个结点之间所经过的结点序列
10. 路径长度：路径上经过的边的个数
11. 树的路径长度：从树根到每一结点的路径长度之和
12. 森林：多个树就组成了森林，只要把树个根结点删去就成了森林，只要给n课树加上结点，就成了树。

#### 4.1.2 题目

	题：一棵有n个结点的树，所有结点的度数之和为_______.
	问题转换成：一棵有3个结点的树，所有结点的度数之和为____. 因为题目是选择，所以应该尽量简化题目。答案n-1

### 4.2 二叉树

1. **二叉树**：每个结点的度不大于2的树；
2. **满二叉树**：除了叶子结点，其他结点的度都为2，也就是不存在只有1个结点的分支；
3. **完全二叉树**：相对于满二叉树，它有的结点度不为2，但是存在的分支的编号与满二叉树相同

![](http://upload.ouliu.net/i/20171027085057wfvvk.png)

在上图中左侧的是完全二叉树，右侧的是满二叉树。

说明：所谓的编号就是指每层从左到右，按照满二叉树的形式编号，上面的满二叉树每个结点的值就是它们的编号。这个编号也是我们在使用顺序存储的时候对应数组的下标。

#### 4.2.1 二叉树的存储结构

1.**顺序存储** 

这种存储方式，可以理解为使用数组存储，**注意开始存储的下标是1**，这是为了与二叉树的性质对应，另外有时候我们也可以将数组的第一个元素作为哨兵。顺序存储的基本思想是，按照满二叉树的编号顺序，如果指定编号（其实就是数组的下标）处有结点的话，数组指定位置的值即为结点的值，否则为0（表示空结点）。当然，也可以在各个元素中存放一些具有具体含义的值。

![](http://upload.ouliu.net/i/20171027085341nf1jm.png)

如图所示的树在数组中的实际存储为：- 1 2 3 0 4 0 5 0 0 6 0（第0位不使用）。

2.**链式存储**

下面是使用Java代码实现的一个二叉树，这里每个结点要包含左右两个子结点以及相应的数据实体。显然，

	public class Tree<E> {
	    private Node<E> root;
	
	    private static class Node<E> {
	        E element;
	        Node<E> leftChild;
	        Node<E> rightChild;
	
	        Node(Node<E> leftChild, E element, Node<E> rightChild) {
	            this.leftChild = leftChild;
	            this.element = element;
	            this.rightChild = rightChild;
	        }
	    }
	}

#### 4.2.2 遍历

1. 先序遍历：a.访问根结点  b.按照先序遍历左子树  c.按照先序遍历右子树
2. 中序遍历：a.按照中序遍历左子树  b.访问根结点  c.按照中序遍历右子树
3. 后序遍历：a.按照后序遍历右子树  b.按照后序遍历左子树  c.访问根结点
4. 层次遍历：借助队列，先将根节点入队，然后出队，访问该结点，将其子结点入队，访问其根结点...直到队列为空。

![](http://upload.ouliu.net/i/201710270854515irpd.png)

说明：上述三幅图从左到右的遍历方式依次是先序遍历、中序遍历和后序遍历。

作为练习，这里给出遍历二叉树的方法，可以观察一下二叉树的实现，以及中序遍历的时候输出的二叉树的值，其中outNode()方法用来输出结点的值：

    /* 先序遍历 */
    public void former() {
        former(root);
    }

    private void former(Node<Key, Value> node) {
        if (node == null) return;
        outNode(node); // 先序
        former(node.leftChild);
        former(node.rightChild);
    }

    /* 中序遍历，注意中序输出的结果和排序的结果的关系 */
    public void center() {
        center(root);
    }

    private void center(Node<Key, Value> node) {
        if (node == null) return;
        center(node.leftChild);
        outNode(node); // 中序
        center(node.rightChild);
    }

    /* 后序遍历 */
    public void latter() {
        latter(root);
    }

    private void latter(Node<Key, Value> node) {
        if (node == null) return;
        latter(node.leftChild);
        latter(node.rightChild);
        outNode(node); // 后序
    }

结论：

1. 可以看出所谓的中序、先序和后序的主要区别就在于输出遍历结果时候的位置；
2. 中序遍历的时候输出结果就是二叉堆的元素的从小到大的排序结果。
3. 那么从上面的操作中可以看出，所谓的中序就是将输出操作夹在了两个递归之间。因此，如果指定的树不是二叉树，而是多叉树，也就是我们使用一个列表来保存各个子结点，那么是没有中序输出的。

#### 4.2.3 构造二叉树

注意如果只知道二叉树的先序序列和后序序列是无法唯一地确定一棵二叉树的。

![](http://upload.ouliu.net/i/201710270855546qo0q.png)

如图所示的二叉树，它的三种遍历方式：

1. 先序遍历结果是：-  +  a  *  b  -  c  d  /  e  f
2. 中序遍历结果是：a  +  b  *  c  -  d  -  e  /  f
3. 后序遍历结果是：a  b  c  d  -  *  +  e  f  /  -

说明：先序遍历的时候，根结点处在第1的位置；中序遍历的时候根节点前面是左子树，后面是右子树；后序遍历的时候，根结点处在最后的位置。可以根据上面的思路，依次进行判断，从而可以写出完整的树。

#### 4.2.4 二叉树的性质

1. 二叉树的第i层上至多有2<sup>i-1</sup>个结点(i>=1)
2. 深度为k的二叉树至多有2<sup>k-1</sup>个结点(k>=1)
3. 对任何一棵二叉树T，如果其终端结点数为n<sub>0</sub>，度为2的结点树为n<sub>2</sub>，则n<sub>0</sub>=n<sub>2</sub>+1.
4. 具有n个结点的完全二叉树的深度为(log<sub>2</sub>n + 1)向下取整的结果
5. 对于一颗有N个结点的完全二叉树的结点按层序编号，对任一结点i，有
	1. 其父结点是i/2向下取整的结果
	2. 左孩子是2i，右孩子是2i+1

## 5、图

### 5.1 术语

1. 如果两个顶点通过一条边相连，我们就称这两个顶点是**相邻**的，并称这条边**依附于**这两个顶点。某个顶点的**度数**，即为依附于它的边的总数；
2. 如果从任意一个顶点都存在一条路径达到另一个顶点，就成这幅图是**连通图**；
3. 树是一幅无环图，互不相连的树组成的集合称为森林。
4. **有向图**：边是带方向的，假如边是从A到B的，那么从A到B可以，从B到A就不行，直观来讲就是图的边带箭头；
5. **无向图**：边不带方向，如果能从A到B就能从B到A，直观来讲就是图的边不带箭头；
6. 完全图：无向图中任意两个顶点之间都存在边的为无向完全图；有向图中，任意两个顶点之间都存在方向相反的弧的是有向完全图。
7. 顶点的**度**、**入度**和**出度**：顶点的度是依附于该点的边的数目，对于有向图又有入度和出度之分，顶点的度等于入度+出度。
8. 一条有向边的第一个顶点称为它的**头**，第二个顶点称为它的**尾**。
9. 在一幅有向图中，**两个顶点之间的关系有四种**：不相连；存在从v到w的边；存在从w到v的边；同时存在从v到w的边和从w到v的边。
10. **有向环**是一条至少含有一条边且起点和终点相同的有向路径。
11. **简单有向环**是一条除了起点和终点必须相同之外，不含重复的顶点和边的环。
12. **有向五环图(DAG)**是一幅不含有向环的有向图。
13. **强连通**：如果两个顶点v和w是相互可达的，则称它们是强连通的，如果一幅图任意两个顶点都是强连通的，则称这幅图是强连通的。
14. **强连通分量**：一幅图中的强连通的顶点构成的最大子集，一幅图可以有多个强连通分量组成。

### 5.2 图的存储

#### 5.2.1 邻接矩阵

使用一个V*V矩阵来表示，其中V是顶点的个数。矩阵的某个元素中存储的可以是布尔变量，表示该边是否存在，在实际应用中也可以将其设置为某个边的权重。使用这种方式的缺点是当数据量比较大的时候会占用很大的存储空间。

#### 5.2.2 邻接表数组

![](http://upload.ouliu.net/i/20171027081751s4epj.png)

即使用一个以顶点为数组的索引的列表数组来表示图。以下是图的一个基于Java的实现：

	public class Graph {
	    private Bag<Integer>[] adj;
	    
	    public Graph(int V) {
	        adj = new Bag[V];
	        for (int i=0;i<V;i++) {
	            adj[i] = new Bag<Integer>();
	        }
	    }
	}

可以看出实际在这里我们使用了背包来存储图的数据。adj的具体含义是：比如，如果顶点0对应的数组元素是adj[0]，这里的adj[0]是一个背包，它里面存储了与顶点0相连的其他顶点。如果是无向图的话，只要在背包中存储所有相关的顶点就可以了。如果是有向图的话，需要存储该点指向的顶点的值。

#### 5.2.3 十字链表

![](http://chuantu.biz/t6/114/1509063561x1942832654.png)

这里将图的顶点和图的边用两种不同类型的数据结构表示。它结合了邻接矩阵和邻接表的特点，如果能够理解邻接表的方式的话，十字链表方式也不难理解——它相当于在邻接表存储方式上增加了一个额外的链。在邻接表中，我们将从某个顶点指出的边作为一个链表。而在十字链表中，我们不仅保存了从某个顶点指出的链，还要保存指向某个顶点的边构造的链。

如上图中，V1->(1,0)->(1,2)是从v1指出的两个边形成的一条链，而V1->(2,1)则保存了指向顶点1的链。

#### 5.2.4 邻接多重表

![](http://upload.ouliu.net/i/20171027082840648v3.png)

它跟邻接表的区别在于，在邻接表中，同一条边由用两个顶点表示，而在邻接多重表中只用一个顶点表示。

#### 5.2.5 边集数组

![](http://upload.ouliu.net/i/2017102708362071w60.png)

上图就是边集数组的表示方法，可以看出它使用一个数组来存储各个边。




