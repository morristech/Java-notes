# 排序

在下面的排序算法中会使用到的方法：

    protected static void exchange(Comparable[] arr, int fromPos, int toPos) {
        Comparable temp = arr[fromPos];
        arr[fromPos] = arr[toPos];
        arr[toPos] = temp;
    }

    protected static boolean less(Comparable cmp1, Comparable cmp2) {
        return cmp1.compareTo(cmp2) < 0;
    }

exchange()用于在将数组中的两个元素的位置交换；less()用于比较两个元素的大小，如果cmp1小于cmp2就返回true. 这里我们将上述两个方法放在一个顶层的名为Sort的类中。

**排序的稳定性**：两个数a和b，它们的大小相等。如果在排序之前，a排在b前面，排序之后，a仍然排在b前面，那么我们称这种排序算法是稳定的。

## 冒泡排序
 
    public static void sort(Comparable[] arr) { // 增序
        for (int i=0;i<arr.length-1;i++) {
            for (int j=i+1;j<arr.length;j++) {
                if (less(arr[j], arr[i])) {
                    exchange(arr, j, i);
                }
            }
        }
    }

所谓的冒泡该怎么理解呢？其实就是每次在数组中i后面的所有的元素中找到一个比arr[i]小的元素和arr[i]交换。如果我们将数组从0到尾竖着放置的话，那么我们就可以看到，数组中比较小的元素会一个个地被移动到数组的顶端上来，这操作看上去就像气泡向上浮动的过程，因此可以叫做冒泡排序。

不过，本质上来说，这个算法的复杂度和选择排序没有什么不同，不过就是每次从指定的元素后面的元素中选择一个最小的元素放在现在的位置。它的效率甚至比不上选择排序——毕竟交换的次数更多一些。

也难怪上面的排序算法效率如此低下，毕竟不是什么正宗的冒泡排序算法。下面的是正宗的冒泡排序算法：

    public static void sort(Comparable[] arr) { // 增序
        for (int i=0;i<arr.length-1;i++) {
            for (int j=arr.length-1;j>i;j--) {
                if (less(arr[j], arr[j-1])) {
                    exchange(arr, j, j-1);
                }
            }
        }
    }

这个算法和上面相比，看上去改动不是很大。主要是将内部的循环从j自增，改成了j自减的形式，而且比较和交换不再是针对i和j处的两个元素，而是j和j-1. 可以这样理解：我们外层的循环从0开始至到倒数第二个元素，然后另一个循环从底部向上直到i处。每次比较的时候是j和j的前一个元素进行比较，交换的也是它们。

![](http://upload.ouliu.net/i/20171028195003rvx8v.png)

以上就是冒泡排序算法的图示，也就是在第二层循环中，我们从尾部开始找到一个元素，如果它比上面的元素小，我们就将其与上面的元素交换。这么做的效果是，我们在一次循环结束之后将最小的元素带到了最顶端。比如上面在将2带到1下面的时候，发现1比2小，此时就丢掉2，继续带着1向上走。在将1移动到最顶端的同时，我们将2移动到了第3个位置。这也是正宗冒泡排序算法比普通冒泡排序算法高效的原因。

### 冒泡排序算法的优化

假如要排序的序列是{2,1,3,4,5,6,7,8,9}，也就是除了2和1都是有序的。如果按照上面的算法，我们在将2和1交换之后还要继续对循环进行遍历，而实际上这个时候整个序列已经是有序的了。因此，我们可以添加一个布尔类型的变量记录是否进行了交换，如果没有进行交换的话，也就意味着整个数组全部都有序了，因此也就没有必要再进行循环了。

    public static void sort(Comparable[] arr) { // 增序
        boolean exchanged = true; // 默认值为true
        for (int i=0;i<arr.length-1 && exchanged;i++) { // 将布尔值加入到for条件中
            exchanged = false; // 开始循环的时候默认是false
            for (int j=arr.length-1;j>i;j--) {
                if (less(arr[j], arr[j-1])) {
                    exchange(arr, j, j-1);
                    exchanged = true; // 当进行了交换的时候才设置为true
                }
            }
        }
    }

冒泡排序算法的时间复杂度是O(n<sup>2</sup>).

## 1、选择排序

### 1.1 过程

首先找到数组中的第一个元素交换，然后从剩下的元素中选择最小的和数组的第二个元素交换，如此往复，直到将整个数组排序。

### 1.2 代码

	public class Selection extends Sort{

	    public static void sort(Comparable[] arr) { // 增序
	        int length = arr.length;
	        for (int i=0;i<length-1;i++) {
	            int min = i;
	            for (int j=i+1;j<length;j++) {
	                if (less(arr[j], arr[min])) {
	                    min = j;
	                }
	            }
	            if (min != i) {
	                exchange(arr, min, i);
	            }
	        }
	    }
	}

在这份代码中应该注意的地方是循环的边界。只要将这里的less()方法换成bigger()就可以实现按照降序的方式进行排序。

### 1.3 性能

1. 运行时间和输入无关，即使数组有序也会一遍遍执行选择排序过程；
2. 数据移动最少，因为每次排序的时候只交换两个元素的位置；
3. 选择排序需要N<sup>2</sup>/2次比较和N次交换。
4. 时间复杂度是O(n<sup>2</sup>).

## 2、插入排序

### 1.1 过程

插入排序遍历的开始位置是下标为1的元素，它从下标1开始向0依次比较相邻的两个元素，如果第j-1个元素大于第j个元素就交换两者的位置，然后再将位置移动到2、3……按照这样的方式依次进行比较。它能保证每轮比较完毕之后第i个元素之前的元素都是按照顺序（这里是增序）的方式排列的。从而保证当比较完毕最后的元素的时候整个数组都是按照增序的方式排列。

下面是插入算法的实现方式：

![](http://upload.ouliu.net/i/201710282114261xy5q.jpeg)

结合下面的代码来看，这里所谓的“插入”就是靠内部的循环来实现的。它的效果就像是我们将指定位置i的元素插入到了指定的位置，而实际上“插入”是通过对比和移动来实现的。

如果我们对一副乱掉的扑克牌进行排序，那我们可能使用的就是插入算法——每次从后面挨个找出比较小的元素插入到前面适合的位置中去。

### 1.2 代码

	public class Insert extends Sort{
	
	    public static void sort(Comparable[] arr) {
	        int length = arr.length;
	        for (int i=1;i<length;i++) {
	            for (int j=i;j>0 && less(arr[j], arr[j-1]);j--) {
	                exchange(arr, j, j-1);
	            }
	        }
	    }
	}

注意循环的边界，以及第二个循环的条件。

### 1.3 性能

1. 当倒置（也就是与我们期望顺序相反）很少的时候，插入排序的顺序可以很快，因为它依赖于输入数组中的元素的顺序；
2. 最坏情况下需要N<sup>2</sup>/2次比较和N<sup>2</sup>/2次交换。
3. 时间复杂度是O(n<sup>2</sup>).

## 3、希尔排序

### 3.1 过程

仔细去看希尔排序的时候它的逻辑和插入排序相似，区别在于它不只对相邻的两个元素进行排序，而是先对整个数组的局部进行调整，方式是将指定间隔的元素按照顺序进行排列，然后不断缩小间隔，直到1，最终使用插入排序的方式完成最终的排序。

以下是希尔排序的一个实现过程，这里使用每次取一半的方式来缩小间隔。而在代码中，我们使用每次取3分之一的方式来缩小间隔，但是基本的算法思路是一致的：

![](http://upload.ouliu.net/i/20171028205943xjsdy.jpeg)

### 3.2 代码

	public class Shell extends Sort{
	
	    public static void sort(Comparable[] arr) {
	        int length = arr.length;
	        int h = 1;
	        while (h < length / 3) {
	            h = 3 * h + 1; // h的值可以是1,4,13,40……
	        }
	        while (h >= 1) {
	            for (int i=h;i<length;i++) {
	                for (int j=i;j>=h && less(arr[j], arr[j-h]); j-=h) {
	                    exchange(arr, j, j-h);
	                }
	            }
	            h /= 3;
	        }
	    }
	}

如上面的代码，在循环中我们每次执行``h/=3``，这样的效果是先对数组的局部进行排序。比如，如果数组有16个元素的话，那么开始循环时h=13，在循环中达到的效果是将间隔为13的元素分别作为一组进行排序；然后，将间隔为4的元素分别作为一组元素进行排序；最后将间隔为1的元素作为1组进行排序，也就达到了插入排序的效果。但是，它能够提升性能的地方就在于，经过之前的局部排序处理之后，整个数组不再是没有顺序的了，因此也就可以提升插入排序的效率。

### 3.3 性能

1. 希尔排序可以应用于大型数组，并且数组越大优势越明显（相比于插入排序）
2. 最坏时间复杂度依然为O(n2)，一些经过优化的增量序列如Hibbard经过复杂证明可使得最坏时间复杂度为O(n3/2)。

## 4、归并排序

### 4.1 过程

要将一个数组排序，可以先（递归地）将它分成两半分别排序，然后将结果归并起来。它是基于分治的思想：

![](http://upload.ouliu.net/i/20171028210617cun0q.jpeg)

从上图中可以看出，它先将一个数组分成两份，然后再将子数组分成两份……最后只需要对两个元素进行排序，并将排序的结果合并不断地起来。最终，它可以得到原数组的排序结果。

### 4.2 代码

下面是基于Java的实现的一份归并排序的算法，这里使用了一个辅助数组aux:

	public class Merge extends Sort{
	
	    private static Comparable[] aux;
	
	    public static void sort(Comparable[] arr) {
	        aux = new Comparable[arr.length];
	        sort(arr, 0, arr.length - 1);
	    }
	
	    private static void sort(Comparable[] arr, int lo, int hi) {
	        if (hi <= lo) {
	            return;
	        }
	        int mid = lo + (hi - lo) / 2;
	        sort(arr, lo, mid);
	        sort(arr, mid + 1, hi);
	        merge(arr, lo, mid, hi);
	    }
	
	    private static void merge(Comparable[] arr, int lo, int mid, int hi) {
	        int i = lo, j = mid + 1;
	        System.arraycopy(arr, lo, aux, lo, hi + 1 - lo);
	        for (int k=lo;k<=hi;k++) {
	            if (i>mid) {
	                arr[k] = aux[j++]; // 如果游标i超过了mid就将游标j的剩余元素赋值给arr即可
	            } else if (j > hi) { 
	                arr[k] = aux[i++]; // 如果游标j超过了hi就将游标i的剩余元素赋值给rr即可
	            } else if (less(aux[j], aux[i])) {
	                arr[k] = aux[j++]; // 将游标i和游标j中较小的赋值给arr
	            } else {
	                arr[k] = aux[i++];
	            }
	        }
	    }
	}

对于merge()方法，我们使用如下的一个数组来进行分析：

	[i][ ][ ][ ][ ][ ][ ][ ][mid][j][ ][ ][ ][ ][ ][ ][ ]

在每次调用merge()方法的时候会先将传入数组arr拷贝到辅助数组aux，这里使用了System.arraycopy()方法实现。然后，我们可以将i和j看成是两个游标，它们在数组中从左向右移动，上面merge()方法的功能是：先将下标i和下标j中较小的元素赋值给arr，赋值之后游标向右移动。如果两个游标中的哪个超出了范围的上限，那么将另一个游标剩下的元素全部赋值给arr即可。所以，这段代码的功能就是将两个数组的所有元素按照从小到大的顺序合并到一个数组中。

在sort()方法中，我们使用了递归的方式每次都将数组分成两半进行排序，然后将排序的结果合并成一个数组。因此，上面排序的最终效果是：以长度为16的数组a为例，最终效果是：

1. 将a[0]和a[1]排序，a[2]和a[3]排序，并将结果合并成a[0..3]；
2. 将a[4]和a[5]排序，a[6]和a[7]排序，并将结果合并成a[4..7];
3. 将a[0..3]和a[4..7]的结果合并成a[0..7]；
4. 同理，将a[8..15]进行排序；
5. 将a[0..7]和a[8..15]的排序结果合并成a[0..15]，即完成了数组的全部排序。

最终的排序都变成了相邻的两个元素之间的排序再合并。

两外，自底向上的排序方法：

    public static void sort(Comparable[] arr) {
        aux = new Comparable[arr.length];
        int length = arr.length;
        for (int sz=1;sz<length;sz*=2) { // sz:子数组的大小
            for (int lo=0;lo<length-sz;lo+=sz*2) { // lo:子数组的索引
                merge(arr, lo, lo+sz-1, Math.min(lo+2*sz-1, length-1));
            }
        }
    }

下面是自底向上排序算法的执行过程：

![图 自底向上排序算法的执行过程](http://algs4.cs.princeton.edu/22mergesort/images/mergesortBU.png)

所谓的自底向上的意思是先将数组梅两两之间进行比较，然后将比较的结果合并，四四之间合并，八八合并，最终得到整体的排序结果。

### 4.3 性能

1. 对于长度为N的任意数组，自顶向下的归并排序需要(NlgN)/2至NlgN次比较；
2. 归并排序的主要缺点是辅助数组的额外空间和N的大小成正比；
3. 可以考虑对如上的排序算法进行优化。
4. 从上文可看出，每次合并操作的平均时间复杂度为O(n)，而完全二叉树的深度为|log2n|。总的平均时间复杂度为O(nlogn)。而且，归并排序的最好，最坏，平均时间复杂度均为O(nlogn)。

## 5、快速排序

### 5.1 过程

快速排序是一种分治排序算法，它将一个数组分成两个子数组，将两部分分别进行排序。它和归并排序的区别是：归并排序需要将两个数组归并以合成一个完整的数组，而快速排序只要保证两个子数组都有序，整个数组也就有序了。

### 5.2 代码

下面是一份使用Java实现的快速排序的算法：

	public class Quick extends Sort{
	
	    public static void sort(Comparable[] arr) {
	        sort(arr, 0, arr.length - 1);
	    }
	
	    private static void sort(Comparable[] arr, int lo, int hi) {
	        if (hi <= lo) {
	            return;
	        }
	        int j = partition(arr, lo, hi);
	        sort(arr, lo, j - 1);
	        sort(arr, j + 1, hi);
	    }
	
	    private static int partition(Comparable[] arr, int lo, int hi) {
	        int i = lo, j = hi + 1;
	        Comparable v = arr[lo];
	        while (true) {
	            while (less(arr[++i], v)) { // 找到一个大于v的元素
	                if (i == hi) {
	                    break;
	                }
	            }
	            while (less(v, arr[--j])) { // 找到一个小于v的元素
	                if (j == lo) {
	                    break;
	                }
	            }
	            if (i >= j) { // 两个指针相遇了，退出循环
	                break;
	            }
	            exchange(arr, i, j); // 交换i和j处的元素的位置，这样可以保证v左小于v，v右都大于v
	        }
	        exchange(arr, lo, j); // 还要注意最终要将v和a[j]交换
	        return j;
	    }
	}

### 5.3 切分

在partition()方法中，我们将使用初始值a[lo]进行切分，当指针i和j相遇的时候主循环退出。在循环中，a[i]小于v时，增大i，a[j]大于v时减小j，然后交换a[i]和a[j]来保证i左侧的元素都小于v，j右侧的元素都大于v. 当指针相遇时交换a[lo]和a[j]，切分结束，这样切分值就留在a[j]中了。

![http://algs4.cs.princeton.edu/23quicksort/images/partitioning.png](http://algs4.cs.princeton.edu/23quicksort/images/partitioning.png)

如图所示，这里假设要排序的数组是字符串数组KRATELEPUIMQCXOS，那么开始的时候v为K，这时将i向右移动，直到找到大于K的元素，或者达到数组末尾；然后，将j向左移动，直到找到小于K的元素，或者到达数组开头；如果没有到达数组的两端并且i在j的左边，我们就将i和j交换；否则，退出循环，并将K和a[j]交换位置。这样处理之后K左边的元素都小于K，K右边的元素都大于K，那么我们只要再分别将其左侧和右侧的元素进行排序即可，而且排序之后就无需再将两个分数组合并了，因为K左都是小于K，K右都是大于K的。

注意循环是一直进行的，循环退出的条件有三个：1).游标i达到数组的末尾；2).游标j达到数组的开头；3).游标i和游标j的扫描过的区域有重叠。

### 5.4 性能

1. 它的缺点在于在切分不平衡的时候程序的效率可能非常低劣：也就是在上述选择K的过程中，如果K是一个非常靠近数组的开头或者结尾位置的元素，那么在排序的时候就没有办法达到分治的目的了。
2. 快速排序平均时间复杂度也为O(nlogn)级。

## 6、堆排序

### 6.1 优先队列

普通的队列是一种**先进先出**的数据结构，元素在队列尾追加，而从队列头删除。在**优先队列**中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出的行为特征。

**堆**是具有下列性质的**完全二叉树**：每个结点的值都大于等于其两个子结点的值，或者每个结点的值都小于等于两个子结点的值，前者被称为**大顶堆**，后者被称为**小顶堆**。

1. **堆有**序：当一颗二叉树的**每个结点都大于等于它的两个子结点**时，它被称为堆有序（堆有序只能保证根大于子结点，无法保证左子树的结点都大于右子树的结点）；
2. 堆的操作会首先进行一些简单的改动，打破堆的状态，然后再遍历堆并按照要求将堆的状态恢复，这个过程叫做**堆的有序化**；
3. **上浮**：如果二叉堆的有序状态因为**某个结点变得比它的父结点大**而被打破，那么我们就通过交换它和它的父结点来修复堆，如果交换之后这个结点仍然比父结点大就继续向现在的父结点交换，直到它的父结点比它大，或者达到了堆的顶部；
4. **下沉**：如果二叉堆的有序状态因为**某个结点变得比它的子结点小**而被打破，那么我们就通过交换它和它的子结点来修复堆，如果交换之后这个结点仍然比子结点小就继续向现在的子结点交换，直到它的子结点比它小，或者达到了堆的底部。

#### 6.1.1 使用二叉堆实现优先队列

二叉堆的元素的存储方式与我们之前提到的二叉树的存储方式相同，比如如下的二叉堆

![http://algs4.cs.princeton.edu/24pq/images/heap-representations.png](http://algs4.cs.princeton.edu/24pq/images/heap-representations.png)

我们使用一个数组来存储该二叉堆的元素，空出下标为0的元素，所以在数组中以上树的实际存储结果是：- T S R P N O A E I H G. 

#### 6.1.2 二叉的堆的代码实现

一下是一份使用Java实现的二叉堆：

	public class MaxPQ<E extends Comparable<E>> {
	    private E[] elements;
	    private int size;

	    public MaxPQ() {}
	
	    public MaxPQ(int size) {
	        this.elements = (E[]) new Comparable[size + 1]; // 需要使用size+1初始化数组大小，因为第0个元素不使用
	    }
	
	    public MaxPQ(E[] elements) {
	        this.elements = elements;
	    }
	
	    public void insert(E element) {
	        elements[++size] = element;
	        swim(size);
	    }
	
	    public E delMax() {
	        E max = elements[1]; // 要被删除的元素位于下标1处
	        exchange(1, size--); // 交换1和最后一个元素
	        elements[size+1] = null; // 防止结点游离
	        sink(1); // 恢复堆的有效性
	        return max;
	    }
	
	    public int size() {
	        return size;
	    }
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	
	    private boolean less(int i, int j) {
	        return elements[i].compareTo(elements[j]) < 0;
	    }
	
	    private void exchange(int i, int j) {
	        E element = elements[i];
	        elements[i] = elements[j];
	        elements[j] = element;
	    }
	
	    /**
	     * 堆的上浮操作，即将结点k不断和父结点进行比较，如果结点k比父结点k/2大
	     * 那么就将其和父结点(k/2)的位置进行交换，直到达到了顶部结点为止
	     *
	     * @param k 要上浮的结点 */
	    private void swim(int k) {
            // while循环中为true时表示，k非根结点，并且k的父结点小于k
	        while (k > 1 && less(k / 2, k)) {
	            exchange(k / 2, k);
	            k /= 2;
	        }
	    }
	
	    /**
	     * 堆的下沉操作，将父结点k和两个子结点中较大的那个比较，如果父结点比它小就将父结点
	     * 与其进行交换，然后将被交换的那个子结点的位置作为父结点继续进行判断，直达父结点
	     * 大于子结点或者达到了堆的末尾的时候退出循环
	     *
	     * @param k 要进行比较的父结点 */
	    private void sink(int k) {
            int j;
	        while ((j = 2 * k) < size) {
                // 比较k的左结点和右结点，将j赋值两者中较大的那个的下标
	            if (j < size && less(j, j + 1)) { 
	                j++;
	            }
                // 将父结点k和子结点j进行比较，如果父结点大于子结点就结束循环
	            if (!less(k, j)) { 
	                break;
	            }
	            exchange(k, j); // 交换父结点和子结点的位置
	            k = j; // 将下次要判断的父结点定义为上述中的子结点
	        }
	    }
	}

在上述代码中我们实现了堆的上浮和下沉的两个方法，在随后的堆排序中会继续使用这两个方法来进行一些操作。

### 6.2 堆排序

#### 6.2.1 代码

	public class Heap {
	
	    public static void sort(Comparable[] pq) {
	        int n = pq.length;
            // 注意这里下沉的根结点的选择：先从最后一个子树开始，逐渐到最终的根结点
            // 不断地进行下沉操作
	        for (int k = n/2; k >= 1; k--) {
	            sink(pq, k, n);
	        }
            // 上面执行的操作只是堆有序的操作，它能够给我们的信息只有：
            // 最顶端的结点是最大的元素
            // 下面即用上述信息实现堆排序(也是堆排序的主要逻辑):
            // 这里，每次先交换1和n，然后n-1，这样下沉的时候第n个元素就不参与了，而且
            // 每次我们将n和1交换实际意味着已经得到了最大的元素，然后依次找剩下的树中
            // 的最大元素，这样依次取出最大元素放在后面就达到了排序的目的
	        while (n > 1) {
	            exch(pq, 1, n--);
	            sink(pq, 1, n);
	        }
	    }
	
        // 将数组pq[]下沉，这里k是下沉的树的根结点，n是该树中结点的总数
        // 后面两个参数的意义很重要！
	    private static void sink(Comparable[] pq, int k, int n) {
            int j;
	        while ((j = 2 * k) <= n) {
	            if (j < n && less(pq, j, j+1)) {
                    j++;
                }
	            if (!less(pq, k, j)) {
	                break;
	            }
	            exch(pq, k, j);
	            k = j;
	        }
	    }
	
	    private static boolean less(Comparable[] pq, int i, int j) {
	        return pq[i-1].compareTo(pq[j-1]) < 0;
	    }
	
	    private static void exch(Object[] pq, int i, int j) {
	        Object swap = pq[i-1];
	        pq[i-1] = pq[j-1];
	        pq[j-1] = swap;
	    }
	
	}

在上述堆排序代码中的sort()方法中，使用sink()方法将a[1]到a[N]的元素进行排序，for循环构造了堆，然后while循环将最大的元素a[1]和a[N]交换并修复了堆，如此直到堆变空。

![](http://algs4.cs.princeton.edu/24pq/images/heapsort-trace.png)

上图就是堆排序的完整的实现过程。左侧代表的是堆有序的逻辑，右侧即排序的逻辑。从这里可以看出，在执行完堆有序的逻辑之后，树的最顶端元素已经是最大的。然后我们将其与最末尾元素交换，交换之后排除最末尾的元素，然后再执行下沉逻辑。再次排序之后最顶端又是最大元素，我们再次将其与最末尾元素交换，排除，再排序……这样不断先将最大元素和最末尾元素交换，然后排除末尾的(最大的)元素之后堆排序。这样每次总是将最大元素提取出来，也就达到了排序的效果。

#### 6.2.2 性能

堆排序是一种选择排序，它的最坏，最好，平均时间复杂度均为O(nlogn)，它也是不稳定排序。

## 7、排序算法的性能

![](http://algs4.cs.princeton.edu/25applications/images/sort-characteristics.png)

![](http://upload.ouliu.net/i/20171028223546rv2ny.png)
