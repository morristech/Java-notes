# 散列表

使用散列查找算法分为两步：1).使用散列函数将被查找的键转化成数组的一个索引;2).处理碰撞冲突。处理碰撞冲突有两种方法：拉链法和线性探测法。

如果我们用一个数组来存储数据，这时候我们需要使用**散列函数**将要存储的键转换成数组的索引。因为转换之后的数组的索引仍然有可能与已有的索引发生冲突，所以就需要对散列的结果进行**碰撞冲突**。

针对不同的数据类型，我们有多种散列的方法，比如对于整数我们用**除留余数法**，即将大的整数除以数组的长度，这样将得到的余数作为数组的索引；对于浮点数，我们可以将键表示成二进制数然后使用除留余数法；对于字符串类型仍然使用**除留余数法**；对于一个对象类型，我们将其作为组合键处理，将各个字段组合起来计算得到一个索引。实际上在Java中，我们可以通过覆写**hashCode()**方法来计算一个对象的哈希值，覆写hashCode()在Java中的要求比较多，需要慎重处理。不过，Java本身对一些常用的类型覆写了hashCode()，比如String Integer Double等。

## 1、基于拉链法的散列表

所谓的拉链法就是将数组的每个元素定义成一个链表，对于产生碰撞冲突的键值对，我们都把它们塞到链表当中，这样数组的每个元素就像一条条的拉链，故称**拉链法**。

假设用于存储的数组是st[]，那么我们的每个st[i]就是一条链表，而链表st[i]的每个结点都应该存储一对键值对。我们之所以将这些键值对都放在链表st[i]中是因为它们的键散列之后的数组索引都是i，但是键是不同的。所以为了以后的查找，要在链表中将键和值都保存下来。

所以，从这里也可以看出，使用散列表能够提高查找效率的主要原因是，我们将每次的查找都放在一条链表中进行，不用在所有的数据中进行。因此，散列之后的键值对能否均匀地分配到数组的每个元素中是影响散列表的一个主要因素，这也是为什么我们需要**尽可能地将散列值均匀分布中的原因**。

下面是基于拉链法的散列表的基本操作：[SeparateChainingHashST.java](http://algs4.cs.princeton.edu/34hash/SeparateChainingHashST.java.html)

    public Value get(Key key) {
        if (key == null) throw new IllegalArgumentException("argument to get() is null");
        int i = hash(key);
        return st[i].get(key);
    } 

    public void put(Key key, Value val) {
        if (key == null) throw new IllegalArgumentException("first argument to put() is null");
        if (val == null) {
            delete(key);
            return;
        }

        // double table size if average length of list >= 10
        if (n >= 10*m) resize(2*m);

        int i = hash(key);
        if (!st[i].contains(key)) n++;
        st[i].put(key, val);
    } 

    public void delete(Key key) {
        if (key == null) throw new IllegalArgumentException("argument to delete() is null");

        int i = hash(key);
        if (st[i].contains(key)) n--;
        st[i].delete(key);

        // halve table size if average length of list <= 2
        if (m > INIT_CAPACITY && n <= 2*m) resize(m/2);
    } 

## 2、基于线性探测法的散列表

![](http://algs4.cs.princeton.edu/34hash/images/linear-probing.png)

上图可以很好地为我们解释线性探测法的基本原理。

**插入的基本思路**：在线性探测法中，我们每次先根据计算得到的哈希值试着往指定索引的数组中插入元素，如果数组的指定索引中已经存在一个元素（也就是发生了冲突），这时候我们沿着当前的索引往后继续寻找空的位置（即为null的数组元素），找到了为空的位置之后将该值放在该位置即可。

**查找的基本思路**：在线性探测法中查找指定键时，仍要先计算该键的哈希值以得到一个数组索引，然后从得到的索引开始，向后不断判断指定索引处的键是否与我们要查找的键相等，知道遇到了空的位置（即数组中元素为null的位置）。

**删除的基本思路**：从上面对查找的分析中可以看出，我们查找的时候是以得到null作为终点的，假如我们要删除的元素处在一个键簇的中间位置，这时我们单纯地将该值置为null，下次查找的时候后面的元素肯定都访问不到了（因为遇到null就终止了）。所以，在删除的时候我们不仅要将指定的元素置为null，还要将其后的所有元素重新添加到散列表中。

**键簇**：所谓键簇在基于线性探测法的散列表中指的就是前后都是null的一小段数据。我们查找的时候就是从键簇的某个位置开始直到键簇的结尾（遇到null）。因此，我们可以得知与基于拉链法的散列表不同，基于线性探测法的散列表是通过键要存储的元素分隔成一个个的键簇来提升查询的效率的（每次只在一段键簇中查询就好了）。所以，直观上来理解的话，键簇越短线性探测法的查找效率肯定越高了。但是想让键簇短就要牺牲存储空间，即将存储元素的数组的长度扩大。

下面是基于线性探测法的散列表的基本操作：[LinearProbingHashST.java](http://algs4.cs.princeton.edu/34hash/LinearProbingHashST.java.html)

    public void put(Key key, Value val) {
        if (key == null) throw new IllegalArgumentException("first argument to put() is null");

        if (val == null) {
            delete(key);
            return;
        }

        // double table size if 50% full
        if (n >= m/2) resize(2*m);

        int i;
        for (i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key)) {
                vals[i] = val;
                return;
            }
        }
        keys[i] = key;
        vals[i] = val;
        n++;
    }

    public Value get(Key key) {
        if (key == null) throw new IllegalArgumentException("argument to get() is null");
        for (int i = hash(key); keys[i] != null; i = (i + 1) % m)
            if (keys[i].equals(key))
                return vals[i];
        return null;
    }

    public void delete(Key key) {
        if (key == null) throw new IllegalArgumentException("argument to delete() is null");
        if (!contains(key)) return;

        // find position i of key
        int i = hash(key);
        while (!key.equals(keys[i])) {
            i = (i + 1) % m;
        }

        // delete key and associated value
        keys[i] = null;
        vals[i] = null;

        // rehash all keys in same cluster
        i = (i + 1) % m;
        while (keys[i] != null) {
            // delete keys[i] an vals[i] and reinsert
            Key   keyToRehash = keys[i];
            Value valToRehash = vals[i];
            keys[i] = null;
            vals[i] = null;
            n--;
            put(keyToRehash, valToRehash);
            i = (i + 1) % m;
        }

        n--;

        // halves size of array if it's 12.5% full or less
        if (n > 0 && n <= m/8) resize(m/2);
    }