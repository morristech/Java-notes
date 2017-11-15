# 平衡查找树

二叉树虽然能解决查找的问题，但是在某些情况下，比如当任意相邻的两个元素之间是按照升序或者降序插入的话，那么得到的树将是一个链表的形状。当然这只是一种极端的情况，只是用来说明，树的形状对二叉查找的效率有很大的影响。

这里的平衡二叉树能够让树保持完美的平衡，不会出现极端的成为链表等，而且我们也会试图去保持树的完美平衡。

它实现这个平衡目的的方式是：对于3-结点，如果再向其插入数据，就会使其变成4-结点。4-结点会被分解成一棵子树，并将子树的根节点“进位”父结点。所以，如果进位并导致了根结点变成了4-结点，那么根结点就会被分解成一棵子树。也就是说，实际上平衡查找树的根结点是会发生变化的，而且这种变化的趋势总是倾向于，将树的左子树中的高度与右子树的高度相差。

平衡查找树的定义：平衡查找树是一种二叉排序树，其中每一个节点的左子树和右子树的高度相差至多等于1.

我们将树上结点的左子树深度减去右子树深度的值称为平衡因子，那么平衡二叉树上所有结点的平衡因子只可能是-1, 0, 1. 

## 1、2-3查找树

![http://algs4.cs.princeton.edu/33balanced/images/23tree-anatomy.png](http://algs4.cs.princeton.edu/33balanced/images/23tree-anatomy.png)

1. 将一棵标准二叉查找树的结点称为2-结点（含有一个键和两条链接），3-结点是指含有两个键和三条链接的结点；
2. 一棵2-3查找树或为一棵空树，由2-结点和3-结点构成，并且左链接执行的键都小于该结点，右链接指向的键都大于该结点

如图所示是一课2-3查找树，在它的内部插入新的结点的时候比平衡查找树略复杂。这里面包含一种类似“进位”的方式，即如果在某个结点插入了新的结点之后它变成了4-结点（3个键4个链接），那么需要将其分解并将分解后的中间结点传递到父结点中。如果父结点成了4-结点，那么就继续讲父结点分解，并向上传递，一直打到最顶层的根结点为止。

![](http://upload.ouliu.net/i/20171027161229ghawb.png)

上图就是关于2-3树插入结点的时候结点的分裂和融合的示意图，实际上我们可以借助上面的思考方式来考虑2-3树的一些操作的执行过程。即如果是以上面的4-结点为例的话，从左到右四个链接分别代表的含义是：小于A的结点构成的树，A和C之间的结点构成的树，C和E之间的结点构成的树以及大于E的结点构成的树。

## 2、红黑二叉查找树

### 2.1 定义

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-encoding.png)

如图所示，红黑二叉查找树只是将2-3查找树中的3-结点转表示成由一条**左斜的红色链接**相连的两个2-结点。这种表示方法的优点是我们可以无需修改就使用二叉查找树的get()方法。

有些图在表示红黑树的时候，没有使用红连接，而是使用红色的结点，比如这里的a就应该被表示成一个红色的结点。虽然表达方式不一样，但是实际意义是一样的——a和b构成了2-3树中的3结点。

红黑二叉查找树的**等价定义**:

1. 红链接均为左链接；
2. 树的根为黑色，叶子节点（即所谓的空节点）为黑色；（根必须为黑色的，但可以是3-结点）
2. 没有任何一个结点同时和两个红链接相连；
3. 该树是完美黑色平衡，即任意空链接到根结点的路径上的黑链接数量相同。

实际上红黑二叉查找树将红链接画平时，一棵红黑树就是一棵2-3树。我们在二叉树的基础之上增加了一个红黑的条件，这使得红黑树的红链接既能够表示2-3树中的3-结点的情形，又能具有二叉树的只具有两个分支的性能。即它既满足了我们对效率的要求，又能够比较方便地实现。

注意下面的关于红黑二叉树的操作中，除了完成基本的功能之外，是**如何维护树的平衡的**。

### 2.2 红黑树的表示

与二叉树类似，只是这里需要在二叉树的基础之上增加一个字段color用于表示红黑二叉查找树的**结点**的颜色：

	public class RedBlackBST<Key extends Comparable<Key>, Value> {
	    private final static boolean RED = true;
	    private final static boolean BLACK = false;
	    private Node<Key, Value> root;
	
	    private static class Node<Key extends Comparable<Key>, Value> {
	        Key key;
	        Value value;
	        Node leftChild, rightChild;
	        int N;
	        boolean color; // 增加了一个颜色字段
	
	        public Node(Node leftChild, Key key, Value value, int n, boolean color, Node rightChild) {
	            this.key = key;
	            this.value = value;
	            this.leftChild = leftChild;
	            this.rightChild = rightChild;
	            N = n;
	            this.color = color;
	        }
	    }
	}

### 2.3 旋转

#### 2.3.1 原理

在实现某些操作的时候（比如插入、删除等），会出现红链接在右侧或者两条连续的红链接的状况，在继续操作之前需要对这些情况进行修复，这就需要对其进行**旋转**。将右侧的红链接转换为左侧的红链接的过程叫做**左旋**，相反的操作叫做**右旋**。

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-left-rotate.png)

图 左旋的示意图

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-right-rotate.png)

图 右旋的示意图

其实所谓的左旋和右旋是比较容易理解的，你可以将其想象成旋转就是将E和S中的那个“低”一些的“拎”起来的过程。比如左旋时，就像将S“拎”起来，拎起来之后，E和S之间的那个结点，自然地从S滑落到了E上面。

旋转的时候有几个地方需要注意：旋转之前和之后根结点的颜色的变化，两个结点的颜色变化，两个结点的子结点的变化。

#### 2.3.2 代码实现

下面是左旋和右旋的代码实现，以及容易出现错误的一些地方：

    private Node<Key, Value> rotateLeft(Node<Key, Value> node) {
        Node<Key, Value> right = node.rightChild;
        // node.color = BLACK; // 错误：结点的颜色应该是链接到结点的链接的颜色，所以旋转之后node的结点的颜色变成红色才对
        node.color = RED;
        // right.color = RED; // 错误：旋转之之后right结点的颜色是node的颜色，本质上旋转只是这棵子树内部的变化，对外不变
        right.color = node.color;
        right.leftChild = node; // right的左结点变成了node
        node.rightChild = right.leftChild; // 当前节点的右结点变成了right的左结点
        right.N = node.N; // 因为right成了这棵子树的根，子树的结点总数不变，而结点总数就是node的N的值
        node.N = 1 + size(node.leftChild) + size(node.rightChild); // 需要重新统计一下
        return right; // 这里的意思是将旋转之后的这棵子树的根结点返回
    }
    
    private Node<Key, Value> rotateRight(Node<Key, Value> node) {
        Node<Key, Value> left = node.leftChild;
        left.color = node.color;
        node.color = RED;
        node.leftChild = left.rightChild;
        root.rightChild = node;
        left.N = node.N;
        node.N = 1 + size(node.leftChild) + size(node.rightChild);
        return left;
    }

### 2.4 颜色转换

当存在两个连续的红链接的时候，我们需要将红链接的颜色转换成黑色，并将中间结点的根结点的颜色转换成红色。在使用红黑树的时候应该将红黑树和我们之前讨论的2-3树联系起来。因为我们知道一个红链接相连的两个结点就相当于2-3树中的3结点。所以如果两个红链接连续的话，就相当于一个4-结点。这时候我们需要将这个4-结点转换成一个树，并且需要将中间结点“进位”到父结点中。在红黑树中表示的进位话就是通过一个红链接联系在一起了。

![](http://algs4.cs.princeton.edu/33balanced/images/color-flip.png)

### 2.5 红黑树的插入

使用插入方式与二叉树类似，首先也要先不断地在树中查找待插入结点的位置，区别在于插入了新的结点之后原来的链接就可能会由黑链接转换成红链接，当红链接在左侧和右侧的时候又要根据情况对红链接进行旋转处理。不过不论怎么处理也逃不出左旋、右旋和颜色转换三种操作。所以，理解起来并不复杂。

    public void put(Key key, Value value) {
        root = put(root, key, value);
        root.color = BLACK;
    }

    private Node<Key, Value> put(Node<Key, Value> node, Key key, Value value) {
        if (node == null) {
            // 新插入的结点总是红结点
            return new Node<Key, Value>(null, key, value, 1, RED, null);
        }
        int cmp = key.compareTo(node.key);
        if (cmp < 0) {
            node.leftChild = put(node.leftChild, key, value);
        } else if (cmp > 0) {
            node.rightChild = put(node.rightChild, key, value);
        } else {
            node.value = value;
        }

        // 注意下面的三个语句的先后顺序
        // 最后需要对左旋和右旋之后可能出现的两个连续红链接进行处理

        // 存在一个右红链接，需要左旋转
        if (isRed(node.rightChild) && !isRed(node.leftChild)) node = rotateLeft(node);
        // 左侧存在两个连续的红链接的情况
        if (isRed(node.leftChild) && isRed(node.leftChild.leftChild)) node = rotateRight(node);
        // 结点的左右存在两个红链接的情况，需要变换颜色
        if (isRed(node.rightChild) && isRed(node.leftChild)) flipColors(node);
        
        return node;
    }

    private boolean isRed(Node x) {
        if (x == null) return false;
        return x.color == RED;
    }

红黑树等同于2-3树，因此在插入的时候我们将新插入的结点用红链接和父结点相连，在2-3树中的意义就是将新插入的结点与原来的结点融合组成一个3-结点或者4-结点，然后再将该结点进行分解。在红黑树中所谓“分解”则是通过旋转和颜色转换来实现的。这使得红黑树能够像2-3树一样进行插入操作，这也是它的效率比平衡二叉树效率高的原因。

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-construction.png)

参考上面的红黑树的插入过程，我们可以知道：

1. 如果插入到一个结点的左子树，可能会出现连续两个红连接，这就要rotateRight()；rotateRight()之后可能会出现左右一个结点的左右两个链接为红色的情形，就需要flipColor()。
2. 如果插入到一个结点的右子树，那么会出现一个右的红连接，因此需要rotateLeft()；rotateLeft()之后可能出现连续两个左红链接，因此要rotateRight()；rotateRight()之后可能会出现左右一个结点的左右两个链接为红色的情形，就需要flipColor()。
3. 上面所述的意思是在插入方法中的三个判断语句的顺序是不可颠倒的。如果插入到左子树，可能要经历后面两个if语句；如果插入到右子树，可能要经历三个if语句。

### 2.6 红黑树的删除

红黑树的删除和二叉查找树不一样，区别在于我们删除的结点的同时需要维护树的平衡性。我们在讨论红黑树的删除的时候，可以**将红黑树看作2-3-4树**，即为了便于处理，我们现在允许4-结点的存在。还有一个需要注意的地方是，树是平衡的，所以要借助在2-3-4树中讨论的话，**一个结点的两个子结点应该是同时存在或者同时不存在的（根结点除外）**。

如果一个结点是3-结点，那么我们可以直接将其删除，因为3结点删除一个结点之后成了2-结点，但仍然平衡。所以，问题在于2-结点的删除，**既要删除结点，又要维护树的平衡**。

先考虑删除最小键的情形，我们要删除一个最小键，就应该不断在左子树中查找。同时，为了达到最终被删除的结点不是一个2-结点的目的，我们从根结点向下遍历的时候，就开始不断对树进行调整。

如果是插入结点并且一个结点达到了最大容量，就要不断向上级“进位”，这里为了使被删除的结点不是2-结点就要不断**“借位”**。

    // 假设h是红结点，h的左子结点和h的左子结点的子结点都是黑色的，
    // 让h的左子结点或者它的子结点之一成为红结点
    private Node moveRedLeft(Node h) {
        flipColors(h);
        if (isRed(h.right.left)) { 
            h.right = rotateRight(h.right);
            h = rotateLeft(h);
            flipColors(h);
        }
        return h;
    }

    // 假设h是红结点，h的右子结点和h的右子结点的子结点都是黑色的，
    // 让h的右子结点或者它的子结点之一成为红结点
    private Node moveRedRight(Node h) {
        flipColors(h);
        if (isRed(h.left.left)) { 
            h = rotateRight(h);
            flipColors(h);
        }
        return h;
    }

    private Node balance(Node h) {
        if (isRed(h.right))                      h = rotateLeft(h);
        if (isRed(h.left) && isRed(h.left.left)) h = rotateRight(h);
        if (isRed(h.left) && isRed(h.right))     flipColors(h);

        h.size = size(h.left) + size(h.right) + 1;
        return h;
    }

#### 2.6.1 删除最小键

    public void deleteMin() {
        // 如果根结点的两个子结点都是黑色的，就将根结点置为红色的
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMin(root);
        if (!isEmpty()) root.color = BLACK;
    }

    private Node deleteMin(Node h) { 
        if (h.left == null) return null;

        // 结点h的不是3-结点并且h的左子结点不是3-结点
        // 所以，我们要保证当前结点是3-结点或者当前结点的子结点是3-结点
        // 因为如果当前结点是3-结点，子结点可以向其“借位”
        // 当前结点的左子结点是3-结点，就可以直接删除了
        // 所以以上两种情况不需要处理，直接进入下次递归即可
        if (!isRed(h.left) && !isRed(h.left.left))
            h = moveRedLeft(h); // 效果是把当前结点变成3-结点

        h.left = deleteMin(h.left);
        return balance(h);
    }

#### 2.6.2 删除最大的键

删除最大的键的时候需要注意，因为红连接都是左连接，所以和删除最小键的时候有所不同。

    public void deleteMax() {
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMax(root);
        if (!isEmpty()) root.color = BLACK;
    }

    private Node deleteMax(Node h) { 
        // 当前结点是红结点
        if (isRed(h.left))
            h = rotateRight(h);

        if (h.right == null)
            return null;

        // 当前结点不是3-结点(因为旋转过了)，并且它的子结点也不是3-结点
        if (!isRed(h.right) && !isRed(h.right.left))
            h = moveRedRight(h);

        h.right = deleteMax(h.right);

        return balance(h);
    }

#### 2.6.3 删除指定的键

    public void delete(Key key) { 
        if (!contains(key)) return;

        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = delete(root, key);
        if (!isEmpty()) root.color = BLACK;
    }

    // 删除已h为根的树中的指定键的结点
    // 这里的删除的操作融合了上面的两种删除方式
    private Node delete(Node h, Key key) { 
        if (key.compareTo(h.key) < 0)  {
            if (!isRed(h.left) && !isRed(h.left.left))
                h = moveRedLeft(h);
            h.left = delete(h.left, key);
        } else {
            if (isRed(h.left))
                h = rotateRight(h);
            if (key.compareTo(h.key) == 0 && (h.right == null))
                return null;
            if (!isRed(h.right) && !isRed(h.right.left))
                h = moveRedRight(h);
            if (key.compareTo(h.key) == 0) {
                Node x = min(h.right);
                h.key = x.key;
                h.val = x.val;
                h.right = deleteMin(h.right);
            }
            else h.right = delete(h.right, key);
        }
        return balance(h);
    }

### 2.7 红黑树的查找

复用二叉查找树的查找方法。
