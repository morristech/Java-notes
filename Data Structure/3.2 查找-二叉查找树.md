# 二叉查找树

## 1、概念

**二叉查找树(BST)**是一种二叉树，它的每个结点都只有左右两个链接，分别指向自己的左子结点和右子结点，并且每个结点都大于其左子树中的任意结点的键，并且小于任意右子树的任意结点的键。(注意和堆的区别)

## 2、代码实现

### 2.1 使用循环的方式来进行查找和插入

下面的代码中使用了循环的方式来将指定的键值对插入到表中。**缺点**：相比于使用递归的方式这里的代码显得比较冗长，而且在循环之中没办法统计和更新加入新的分支之后它的各个父结点的分支统计N的数量信息：

	public class BST<Key extends Comparable<Key>, Value> {
	    private Node<Key, Value> root;
	    private int size;
	
        // 二叉查找树的结点的定义
	    private static class Node<Key extends Comparable<Key>, Value> {
	        Node<Key, Value> leftChild;
	        Key key;
	        Value value;
	        Node<Key, Value> rightChild;
	        int N; // 每个结点的子结点的总数
	
	        Node(Node<Key, Value> leftChild, Key key, Value value, int N, Node<Key, Value> rightChild) {
	            this.leftChild = leftChild;
	            this.key = key;
	            this.value = value;
	            this.rightChild = rightChild;
	            this.N = N;
	        }
	    }
	
	    public int size() {
	        return size(root);
	    }
	
	    private int size(Node<Key, Value> node) {
	        if (node == null) return 0;
	        else return node.N;
	    }
	
	    public void put(Key key, Value value) {
	        if (key == null) throw new IllegalArgumentException("Key cannot be null.");
	        Node<Key, Value> newNode = new Node<Key, Value>(null, key, value, 1, null);
	        if (root == null) {
	            root = newNode;
	        } else {
	            Node<Key, Value> node = root;
	            while (true) {
	                int cmp = key.compareTo(node.key);
	                if (cmp > 0) {
	                    if (node.rightChild == null) {
	                        node.rightChild = newNode;
	                        size++;
	                        break;
	                    } else {
	                        node = node.rightChild;
	                    }
	                } else if (cmp < 0) {
	                    if (node.leftChild == null) {
	                        node.leftChild = newNode;
	                        size++;
	                        break;
	                    } else {
	                        node = node.leftChild;
	                    }
	                } else {
	                    node.value = value;
	                    break;
	                }
	            }
	        }
	    }
	
	    public Value get(Key key) {
	        if (isEmpty() || key == null) {
	            return null;
	        }
	        Node<Key, Value> node = root;
	        while (true) {
	            int cmp = key.compareTo(node.key);
	            if (cmp > 0) {
	                if (node.rightChild == null) {
	                    return null;
	                } else {
	                    node = node.rightChild;
	                }
	            } else if (cmp < 0) {
	                if (node.leftChild == null) {
	                    return null;
	                } else {
	                    node = node.leftChild;
	                }
	            } else {
	                return node.value;
	            }
	        }
	    }
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	}

### 2.2 使用递归的方式进行查找和插入

如上所述，上面的代码使用了循环的方式来实现二叉树，下面使用递归的方式来实现二叉树的查找和插入：

    public void put(Key key, Value value) {
        root = put(root, key, value);
    }

    // 下面这段递归的代码并不难理解：每次根据比较的结果，
    // 如果比当前结点大，就到右子树中继续递归查找
    // 如果比当前结点小，就到左子树中继续递归查找
    private Node<Key, Value> put(Node<Key, Value> node, Key key, Value value) {
        if (node == null) { // 递归终止的条件，注意返回的对象的意义
            return new Node<Key, Value>(null, key, value, 1, null);
        }
        int cmp = key.compareTo(node.key);
        if (cmp < 0) {
            node.leftChild = put(node.leftChild, key, value);
        } else if (cmp > 0) {
            node.rightChild = put(node.rightChild, key, value);
        } else {
            node.value = value;
        }
        node.N = size(node.leftChild) + size(node.rightChild) + 1;
        return node;
    }

    public Value get(Key key) {
        return get(root, key);
    }

    // 也是使用递归的方式来进行查找操作
    private Value get(Node<Key, Value> node, Key key) {
        if (node == null) {
            return null;
        }
        int cmp = key.compareTo(node.key);
        if (cmp < 0) {
            return get(node.leftChild, key);
        } else if (cmp > 0) {
            return get(node.rightChild, key);
        } else {
            return node.value;
        }
    }

可以看出，递归可以在添加完各个结点之后返回的时候将结点的统计信息加入到N字段上。这个在上面的基于循环的方式实现的二叉树中是没有实现的。

### 2.3 性能

1. **二叉树的性能与二叉树的形状有一定的关系**，考虑一种极端的情况，如果每次插入的数据相互之间是一种递增的关系，那么得到的二叉树就应该是一个链表的形状。这种情形下的二叉树的查找和插入的效率是最低的，而当二叉查找树是完全平衡的时候，查找的效率是最高的。
2. 由N个随机键构成的二叉查找树中，查找命中平均需要的比较次数为**2lnN(约1.39lgN)**. 
3. 由N个随机键构成的二叉查找树中，插入操作和查找未命中平均需要的比较次数为**2lnN(约1.39lgN)**. 

### 2.4 二叉树的其他方法实现

虽然，上面我们已经实现了二叉树的基本的插入和查找的操作，但是在实际的二叉树应用中，我们还需要提供一些其他的方法来方便使用。

#### 2.4.1 最大键和最小键

根据二叉树的性质，我们可以知道，不断地对二叉树的左子结点进行查找，即可得到键最小的结点。同理，不断对二叉树的有子结点进行查找，即可得到最大的结点。下面是使用循环来实现的获取最小和最大结点的值的方法：

    public Value getMax() {
        Node<Key, Value> node = root;
        while (node != null) {
            if (node.rightChild == null) {
                return node.value;
            }
            node = node.rightChild;
        }
        return null;
    }

    public Value getMin() {
        Node<Key, Value> node = root;
        while (node != null) {
            if (node.leftChild == null) {
                return node.value;
            }
            node = node.leftChild;
        }
        return null;
    }

#### 2.4.2 排名

所谓的排名就是返回二叉树的某个结点在所有结点中的编号，这里我们使用rank(Key)方法来获取某个键对应的排名，使用select(int)来获取某个排名对应的键。

如果是根结点，那么我们直接返回它的左子树的结点的数量即可；如果要查找的键在左子树中，我们返回该键在左子树中的排名；如果要查找的键在右子树中，我们要返回根节点加上它在右子树中的排名。

    public int rank(Key key) {
        return rank(root, key);
    }

    private int rank(Node<Key, Value> node, Key key) {
        if (node == null) {
            return 0;
        }
        int cmp = key.compareTo(node.key);
        if (cmp > 0) { // 使用递归的方式，不断加上各根结点的左子树+根的数量，加上它自己在左子树中的排名
            return 1 + size(node.leftChild) + rank(node.rightChild, key);
        } else if (cmp < 0) {
            return rank(node.leftChild, key);
        } else {
            return size(node.leftChild);
        }
    }

    public Key select(int index) {
        Node<Key, Value> node = select(root, index);
        if (node == null) {
            return null;
        }
        return node.key;
    }

    private Node select(Node<Key, Value> node, int index) {
        if (node == null) {
            return null;
        }
        // 根据当前结点的左右两个子树的结点的大小与索引的比较的结果
        // 来决定要向左子树继续查找，还是向右子树继续查找
        int size = size(node.leftChild);
        if (size > index) {
            return select(node.leftChild, index);
        } else if (size < index) {
            return select(node.rightChild, index - size - 1);
        } else {
            return node;
        }
    }

#### 3.4.3 删除最大键和最小键

对于删除最小键，不断深入到左子树中，直到遇到一个左子树为空的结点，将其删除即可；对于删除最大键，不断深入右子树，直到遇到一个右子树为空的结点，将其删除即可。

下面是使用Java实现的删除最大键和最小键的实现，这里除了删除最大和最小的键之外，还加入了统计结点数量的相关的逻辑。

    public void delMax() {
        root = delMax(root);
    }

    private Node<Key, Value> delMax(Node<Key, Value> node) {
        if (node.rightChild == null) {
            return node.leftChild;
        }
        node.rightChild = delMax(node.rightChild); // 当当前结点的右侧结点为空，就将当前结点的左子结点作为右结点
        node.N = size(node.leftChild) + size(node.rightChild) + 1;
        return node;
    }

	public void delMin() {
        root = delMin(root);
    }

    private Node<Key, Value> delMin(Node<Key, Value> node) {
        if (node.leftChild == null) {
            return node.rightChild;
        }
        node.leftChild = delMin(node.leftChild); // 左结点的左子节点为空时返回左结点的右结点作为自己的左结点
        node.N = size(node.leftChild) + size(node.rightChild) + 1;
        return node;
    }

#### 3.3.4 删除操作

删除某个节点的时候需要分成几种情况来处理：

1. 当某个**被删的结点两个子结点只存在一个**的时候，直接将被删结点的子树连接连接到被删结点处即可；
2. 当某个**被删的结点两个子结点都不存在**的时候，直接将其删除即可；
3. 当某个**被删的结点两个子结点都存在**的时候，我们可以将被删结点的右子树称为T。如下图所示，我们需要找到T中最小的结点，即图中的绿色的结点。因为它最小，所以它有个比较好的特性——左子树为空。就像下图，我们只要将该绿色的结点从T中移动到被删的位置。如果该绿色结点有右子树，我们可以将子树连接到N原来的位置。

![](http://upload.ouliu.net/i/20171027112448lxjmu.png)

下面是使用Java实现的一份从树中删除某个键的结点的代码：

    public void delete(Key key) {
        root = delete(root, key);
    }

    private Node<Key, Value> delete(Node<Key, Value> node, Key key) {
        if (node == null) {
            return null;
        }
        int cmp = key.compareTo(node.key);
        if (cmp > 0) { // 比当前的结点大，到右子树中继续查找并删除
            node.rightChild = delete(node.rightChild, key);
        } else if (cmp < 0) { // 比当前结点小，到左子树中继续查找并删除
            node.leftChild = delete(node.leftChild, key);
        } else { // 与当前结点相等
            // 如果左子结点或者右子结点为空的话，直接将另一个子树连接到被删除的位置即可
            if (node.leftChild == null) {
                return node.rightChild;
            }
            if (node.rightChild == null) {
                return node.leftChild;
            }
            // 当被删的结点左右子结点都存在时的情况
            Node<Key, Value> temp = node;
            node = min(temp.rightChild);
            node.rightChild = delMin(temp.rightChild);
            node.leftChild = temp.leftChild;
        }
        node.N = size(node.leftChild) + size(node.rightChild) + 1;
        return node;
    }

如果删除的结点中有子树的时候这里的处理方式是，先获取树中最小的结点N，然后删除T中最小的结点并将根据结点赋值为N的右结点，删除的过程中实际上已经将N原来的右子树的结点放在了N原来的位置，然后将被删结点的左子树赋值给N即可。

#### 3.4.5 范围查找

获取二叉树的所有子结点的键的时候，我们可以使用背包来实现：

    public Iterable<Key> keys(Key lo, Key hi) {
        Bag<Key> keys = new Bag<Key>();
        addKey(root, keys, lo, hi);
        return keys;
    }

    private void addKey(Node<Key, Value> node, Bag<Key> bag, Key lo, Key hi) {
        if (node == null) {
            return;
        }
        int cmplo = lo.compareTo(node.key); // <=0
        int cmphi = hi.compareTo(node.key); // >=0
        if (cmphi >= 0 && cmplo <= 0) {
            bag.add(node.key);
        }
        addKey(node.leftChild, bag, lo, hi);
        addKey(node.rightChild, bag, lo, hi);
    }