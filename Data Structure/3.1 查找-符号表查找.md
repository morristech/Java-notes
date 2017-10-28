# 符号表

**符号表**：有时也称为字典或索引，其最主要的目的是将一个键和一个值联系起来

## 1、顺序查询

### 1.1 代码

所谓的顺序查找就是将集合中的所有元素一个一个地遍历然后进行匹配查询。下面是一个基于链表实现的符号表，这里在获取某个键的值的时候要将链表全部遍历并进行匹配：

	public class LinkedList<Key extends Comparable<Key>, Value> {
	    private Node<Key, Value> first;
	
	    private static class Node<Key extends Comparable<Key>, Value> {
	        Key key;
	        Value value;
	        Node<Key, Value> next;
	
	        Node(Key key, Value value, Node<Key, Value> next) {
	            this.key = key;
	            this.value = value;
	            this.next = next;
	        }
	    }
	
	    public void add(Key key, Value value) {
	        if (key == null) {
	            throw new IllegalArgumentException("key cannot be null");
	        }
	        if (first == null) {
	            first = new Node<Key, Value>(key, value, null);
	        } else {
	            // 尝试获取指定键的结点
	            Node<Key, Value> nodeGet = getNode(key);
	            if (nodeGet != null) {
	                nodeGet.value = value;
	            } else {
	                // 将新加入的结点添加到链表的尾部
	                first = new Node<Key, Value>(key, value, first);
	            }
	        }
	    }
	
	    public Value get(Key key) {
	        Node<Key, Value> node = getNode(key);
	        if (node == null) {
	            return null;
	        } else {
	            return node.value;
	        }
	    }
	
	    private Node<Key, Value> getNode(Key key) {
	        Node<Key, Value> node = first;
	        while (node != null) {
	            if (node.key.equals(key)) {
	                return node;
	            }
	            node = node.next;
	        }
	        return null;
	    }
	}

### 1.2 性能

1. 含有N个键值对的基于无序链表的符号表中，未命中的查找和插入操作都需要N次比较。命中的查找在最坏的情况下需要N次查找。
2. 向一个空表中插入N个不同的键需要N<sup>2</sup>/2次比较。

## 2、二分查找

### 2.1 代码

下面是一个使用二分查找实现的符号表的例子，在这里我们使用了循环而不是递归的方式来进行二分查找：

	public class BinarySearchST<Key extends Comparable<Key>, Value> {
	    private Key[] keys;
	    private Value[] values;
	    private int size;
	
	    public BinarySearchST(int capacity) {
	        keys = (Key[]) new Comparable[capacity];
	        values = (Value[]) new Object[capacity];
	    }
	
	    public Value get(Key key) {
	        if (isEmpty()) {
	            return null;
	        }
	        int rank = rank(key);
	        if (rank < size && keys[rank].equals(key)) {
	            return values[rank];
	        }
	        return null;
	    }
	
	    public void put(Key key, Value value) {
	        if (key == null) {
	            throw new IllegalArgumentException("Key cannot be null.");
	        }
	        int rank = rank(key);
	        if (rank < size && keys[rank].equals(key)) {
	            values[rank] = value;
	        } else {
	            keys[size] = key;
	            values[size++] = value;
	        }
	    }
	
	    public int size() {
	        return size;
	    }
	    
	    public boolean isEmpty() {
	        return size == 0;
	    }
	
		// 使用循环的方式二分查找来获取指定的键对应的数组索引
	    private int rank(Key key) {
	        int lo = 0, hi = size - 1;
	        while (lo <= hi) {
	            int mid = lo + (hi - lo) / 2;
	            int cmp = keys[mid].compareTo(key);
	            if (cmp < 0) {
	                lo = mid + 1;
	            } else if (cmp > 0) {
	                hi = mid - 1;
	            } else {
	                return mid;
	            }
	        }
	        return lo;
	    }
	}

### 2.2 性能

1. 对N个键的有序数组进行二分查找最多需要lgN+1次比较（无论是否成功）；

## 3、线性索引查找

所谓线性索引就是将索引项集合组织成为线性结构，成为索引表。线性索引分成三种：稠密索引、分块索引和倒排索引。

### 3.1 稠密索引

稠密索引类似于你在一张纸条上记下各个东西的位置，然后按照纸条上面的位置到指定的位置去寻找物品。

### 3.2 分块索引

类似于数据库的表的分区，或者将各个地区分成不同的省份一样。即先将整体分成各个大的部分，然后在各个部分内部再使用其他索引方式来进行查找。

### 3.3 倒排索引

类似于搜索引擎，对每个词汇记录它出现的位置，然后再到指定的位置去寻找文章等等。




