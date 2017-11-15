## 容器

数组的效率比容器高，但是我们应该“优先选择容器而不是数组”，只有在已证明性能成为问题时，才应该将程序重构为使用数组。

## 1 容器的框架

![](http://images2015.cnblogs.com/blog/1010726/201706/1010726-20170621004756882-1379253225.gif)

以上是容器的框架的结构，它展示了整个集合框架中的所有可用的API之间的关系。

我们可以将Java容器类划分成两种不同的概念：

1. Collection： 是一组独立元素的列表，它有三个子类List, Set和Queue. 它们之间的区别是:List必须按照插入的顺序保存元素；Set不能有重复元素；Queue是按照队列的顺序
2. Map：表示键值对类型的对象

此外，还有两个工具类可以使用：Arrays和Collections.

## 2 Collection

它不仅名称叫做Collection（集合），很多方法的定义也和"集合"的概念是相似的。

这里给出它的定义，因为它作为框架中顶层的基接口，规定了其内部的各个方法该实现什么样的功能。我们先给出它的定义，然后再看在几个重要的子类中是如何实现它们的：

	public interface Collection<E> extends Iterable<E> {
	    int size();
	    boolean isEmpty();
	    boolean contains(Object o);
	    Iterator<E> iterator();
	
	    // 返回列表元素组成的数组
	    Object[] toArray();
	
	    // 返回列表元素组成的数组，不同的是它可以根据传入的泛型数组，返回指定类型的数组
	    <T> T[] toArray(T[] a);
	
	    boolean add(E e);
	    boolean remove(Object o);
	    boolean containsAll(Collection<?> c);
	    boolean addAll(Collection<? extends E> c);
	    boolean removeAll(Collection<?> c);
	
	    // 只保留该集合中处于c中的元素，即只保留当前集合与c的交集元素
	    boolean retainAll(Collection<?> c);
	
	    void clear();
	}

## 3 List                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        

在Collection的基础之上，List又增加了几个方法。从下面的方法的定义中，我们可以看出这些方法是针对List列表类型的容器定义的方法：

    boolean addAll(int index, Collection<? extends E> c);
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);
    int indexOf(Object o);
    int lastIndexOf(Object o);
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);
    List<E> subList(int fromIndex, int toIndex);

这里的前面的几个方法从字面意思中就可以看出大致的作用。后面的`listIterator()`方法的作用是返回一个迭代器，这个迭代器不仅能向后遍历，还能向前遍历；而`subList`的作用是返回容器中元素的一个视图（如果在视图中添加元素，添加的元素也会反应到原来的容器上）。

subList视图实现插入元素的具体方式是（以ArrayList为例）：在创建SubList（通常继承自AbstractList，作为一个List实现）的时候，将外部类作为参数传入到Sublist的构造方法中（作为this传入），然后当用户调用视图的添加元素的方法的时候，它调用外部类ArrayList的add方法，同时将元素添加到外部类容器中。(装饰器模式)

List的一个顶层实现是AbstractList，它是一个抽象的类，为我们实现了List中定义的一些方法。而我们熟知的ArrayList和LinkedList是在它的基础之上完善的。

### 3.1 ArrayList

它内部是基于**动态数组**来实现的，所以它具有数组的一些特性。比如，访问固定位置的某一元素效率比较高，而对容器中的元素查找效率比较低（需要遍历数组进行查找）。

它**默认的初始容量是10**，每次加入一个新的元素的时候，会将加入元素之前的容量与新加入的元素数量的和与加入元素之前的容量的3/2进行对比，选择较大的一个作为将要拓充到的容量。具体的调整数组大小的操作是由**System.arrayCopy**来实现的。

至于在ArrayList中的删除操作，也是先将元素置为null，之后使用动态System.arrayCopy来调整数组的大小。

在Arrays中还有一个ArrayList的实现**Arrays.ArrayList**，它跟这里的ArrayList是不同的。我们通过Arrays.asList可以获得该ArrayList. 虽然也是继承了AbstractList，但是它在实现的时候没有实现add相关的方法，所以是不能向其中添加新元素的。

### 3.2 LinkedList

它内部是使用双向链表来实现的，所以它的一些操作和双向链表逻辑相同。比如查找某个元素的时候需要对链表进行遍历，而插入元素的时候效率比较高。

我们可以很容易地将它改造成队列、栈或双端队列。

#### 3.3 Stack

队列数据结构，我们可以使用LinkedList来实现一个队列，但是在Java提供的API中，队列是继承自Vector的，而Vector内部使用数组来实现。这就导致了Stack在性能上，相比于使用链表的方式还有值得提升的地方。

### 4 Set

它并没有在Collection的基础之上再增加新的方法。它的一个抽象基类的实现是`AbstractSet`，不过在AbstractSet中并没有实现太多的方法。

它不允许插入两个重复的元素。它有几个重要的实现：TreeSet将元素存储在红黑树数据结构中，而HashSet使用的是散列函数；LinkedHashList使用了散列，但是看起来它使用了链表来维护元素的插入顺序。

### 4.1 HashSet

它内部是借助于HashMap来实现的：当向HashSet中插入一个元素的时候，我们将插入的元素作为HashMap的键，一个默认的Object作为值插入到HashMap中。因为HashMap中不会有两个相同的键，所以在HashSet中就不会有两个相同的值。

HashSet没有提供普通的get方法，我们只能对其进行遍历。它在遍历的时候当然返回的是HashMap的键的迭代器。

### 4.2 TreeSet

与HashSet类似，它内部使用了TreeMap来实现自己的功能。

## 5 Map

Map与Collection一样，Map是所有的基于键值对的容器的顶层接口。下面是它内部定义的一些方法：

	public interface Map<K,V> {
	    int size();
	    boolean isEmpty();
	    boolean containsKey(Object key);
	    boolean containsValue(Object value);
	    V get(Object key);
	    V put(K key, V value);
	    V remove(Object key);
	    void putAll(Map<? extends K, ? extends V> m);
	    void clear();
	    Set<K> keySet();
	    Collection<V> values();
	    Set<Map.Entry<K, V>> entrySet();
	    interface Entry<K,V> {
	        K getKey();
	        V getValue();
	        V setValue(V value);
	        boolean equals(Object o);
	        int hashCode();
	        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
	        }
	        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
	        }
	        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
	            Objects.requireNonNull(cmp);
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
	        }
	        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
	            Objects.requireNonNull(cmp);
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
	        }
	    }
	    boolean equals(Object o);
	    int hashCode();
	}

从上面我们可以看出，在Map接口内部，除了定义了一些常用的操作方法之外，还定义了Map中键值对的接口Entry.

### 5.1 HashMap

要理解HashMap的实现原理需要数据结构方面的知识，这里给出之前整理的一些数据结构和Hash码方法覆写相关的内容：[哈希表](http://s)及[覆写equals和hashCode的方法](https://github.com/Shouheng88/Java-Programming/blob/master/Java%20Language/Object%E7%B1%BB.md).

Java集合API中的HashMap是使用**拉链法**来解决碰撞冲突的。所谓拉链法就是使用一个保存了各个链表的头结点的数组存储各个链表。每次对指定的键值对，我们计算键的哈希码，并对得到的哈希码取余操作，从而得到该键值对所在的链表，然后将其插入到指定的链表中。不过，为了提升性能在HashMap的实现中对许多点进行了优化：动态调整数组的大小；当一个链表存储的元素比较多的时候，采用红黑树来提升查找的效率等。

下面我们看在HashMap中一些实现的逻辑：

1.首先是每个结点的定义

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }

从这里可以看出该结点实现了Map中定义的Entry，这个结点是适用于链表的。还有一个结点是适用于红黑树的：

    private static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }

该树节点继承了LinkedHashMap.Entry，而LinkedHashMap.Entry实际上继承了HashMap.Entry. 这样，我们就可以将TreeNode赋值给Node了。

在向哈希表中插入一个元素的时候，在插入期间它会对该结点所在的链表的大小进行判断。当该链表的长度大于等于TREEIFY_THRESHOLD-1(即为7)的时候，就调整该链表为树结构。

2.计算哈希码

    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

这个方法其实并不复杂，我们可以参考一下[知乎](https://zhihu.com/question/20733617/answer/111577937 )上面的回答。说白了，这里就是为了提高散列码的随机性。因为插入的元素分布越均匀，我们在进行查找的时候效率就越高（链表比较短）。而至于为什么取后16位进行异或操作呢？因为我们将链表的头结点保存在数组中，所以需要将这里的哈希码映射到指定的数组索引。我们这里使用的是除留余数法，就是将哈希码除以某个数之后的余数作为数组的索引。而在HashMap中实际是这样实现除法操作的：

    table[e.hash & (newCap - 1)]
    // 比如当newCap为16的时候，减1得15，即后四位全1，再与hash值取并
    // 那么就得到了hash值在数组中的索引，实际上是为了降低除法的低效率

也就是将指定的哈希码与哈希表容量减一的并作为数组的索引。我们没有使用取余的操作，因为除法和取余是现代操作系统最慢的操作。我们在计算哈希表的容量的时候故意使用2的整数次幂（减1之后就是低位全1的二进制），在计算数组时将其减1再和哈希码取并，这样就避免了使用除法。这也是为什么哈希表的容量要是2的整数次幂的原因。所以，也就是因为我们计算索引时只用后面的几位数这个原因，才将h移位之后异或的。

保证每次取得的哈希表的容量是2的整数次幂的相关代码：

    private static int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

该方法保证了每次计算返回的结果是不小于cap的最小的2的整数次幂。

HashMap在性能方面进行了许多优化，使用了哈希表和红黑树两种数据结构，因此也是面试的热点。这里是一份简单的源码分析：[HashMap的源码分析](https://github.com/Shouheng88/Java-Programming/blob/master/Java%20Language/HashMap.java)。

3.关于HashMap中的几个字段

1. loadFactor：负载因子，默认为0.75，值等于尺寸除以容量。负载因子取值0.75是空间和效率的一个折衷。毕竟但当负载因子小的时候，空间消耗会比较大；而当负载因子比较大的时候，虽然空间消耗小了，但是查询效率低了（因为要等到插入1000个元素才调整大小，这时候碰撞的几率大了，链表的长度可能会增加，降低了查找的效率）。
2. threshold：下次扩容的临界值，size>=threshold就会扩容
3. size：已存元素的个数
4. DEFAULT_INITIAL_CAPACITY：默认的哈希表的容量，值为16.

 当HashMap存放的元素越来越多，到达临界值(阀值)threshold时，就要对Entry数组扩容，HashMap在扩容时，新数组的容量将是原来的2倍，由于容量发生变化，原有的每个元素需要重新计算索引，再存放到新数组中去，也就是所谓的rehash。HashMap默认初始容量16，加载因子0.75，也就是说最多能放16*0.75=12个元素，当put第13个时，HashMap将发生rehash，rehash的一系列处理比较影响性能，所以当我们需要向HashMap存放较多元素时，最好指定合适的初始容量和加载因子，否则HashMap默认只能存12个元素，将会发生多次rehash操作。

### 5.2 HashTable

相比于HashMap，HashTable的实现就显得简单得多。它内部同样自己定义了一个结点：

    private static class Entry<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Entry<K,V> next;
    }

而且也是采用了基于拉链阀的碰撞处理机制。它定义了一个

    private transient Entry<?,?>[] table;

下面是它的根据哈希码计算数组的索引的方法：

    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;

虽然，当插入的元素的数量超过指定的阈值的时候，它也会重新调整数组的大小。但是它的插入操作中是不存在将链表改变成红黑树的优化的。并且这里直接使用了取余的方式获取索引，不存在扰动，从而也不会强制要求容量必须为2的整数次幂。

因为它的散列的均匀性没有HashMap调节得那么好，所以它的性能可能会比HashMap要差一些。而且，它的每个方法上面都加了sychronized关键字进行修饰，这样虽然保证了在多线程环境中的数据一致性，但是，在非多线程环境中，无疑是一种不必要的开销。

## 6、线程安全的容器

### 6.1 同步集合Collections.synchronizedXXX方法

该方法用来对指定的集合进行装饰，它使用了装饰者模式，我们看下面的一个例子

    Map m = Collections.synchronizedMap(new HashMap(...));

这里使用了一个匿名的HashMap，因为HashMap不是线程安全的，但是这封装了之后返回的集合就是线程安全的了。下面我们看该API究竟做了什么操作：

它返回了SynchronizedMap：

    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
        return new SynchronizedMap<>(m);
    }

该Map的定义是

    private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
        private final Map<K,V> m;     // Backing Map
        final Object      mutex;        // Object on which to synchronize

        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }

        public V get(Object key) {
            synchronized (mutex) {return m.get(key);}
        }

        public V put(K key, V value) {
            synchronized (mutex) {return m.put(key, value);}
        }
    }

在上面的代码中，我们只给出了它的一部分内容的定义，从上面我们就很容易地看出来。它实际上就是对我们定义的Map进行了一层封装，所以我们称其为装饰者模式。它的锁不是加在每个方法上面的，而是使用了粒度更小的方式——对每个操作，对当前的对象进行加锁。这样能够保证每个put和get方法本身是原子的和线程安全的，而实际的值的获取和插入，还是对原始的容器进行操作的。

