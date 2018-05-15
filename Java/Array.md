# 数组

## 1 一维数组

### 1.1 一维数组的声明

    类型[] 数组名; 或 类型 数组名[];

声明和创建分别进行

    类型[] 数组名;
    数组名 = new 类型[元素个数];

声明和创建同时进行

    类型[] 数组名 = new 类型[元素个数];

### 1.2 一维数组的实例化

    数组名 = new 类型[]{ 元素0,元素1,......,元素n };
    类型[] 数组名 = new 类型[]{ 元素0,元素1,......,元素n };
    类型[] 数组名 = { 元素0,元素1,......,元素n };

注意：

1. 如果通过`{}`初始化数组元素，则用new关键字创建数组不需要也不能指定数组的元素个数，编译器会自动推断元素个数. 即int []arr = new int[5]{1,2,3,4,5};是错误的. 
2. 就是如果指定了数组的内容就不能指定数组的大小。指定数组大小只能在仅仅定义数组的时候使用。
3. 创建一个数组而没有给其元素赋值时，数字数组所有元素初始化为0，boolean数组的所有元素初始化为false，对象数组的所有元素初始化为null.
4. 可以将“类型[]”当作一个整体，不要往[]中添加数字
5. 常见的定义数组的错误形式

        int[4][2] arr = new int[][];
        int[][] arr; arr = new int[4][2]{........};
        int[][] arr = new int[4][2]{........};

### 1.3 一维数组的访问

    数组名[下标];

可以通过数组的length属性获得数组的长度，即

    数组名.length;

## 2 二维数组

### 2.1 二维数组声明

    类型[][] 数组名; 或 类型 数组名[][];

声明和创建分别进行

    类型[][] 数组名;
    数组名 = new 类型[元素个数1][元素个数2];

声明和创建同时进行

    类型[][] 数组名 = new 类型[元素个数1][元素个数2];

### 2.2 二维数组的实例化

    数组名 = new 类型[]{ 元素0,元素1,......,元素n };
    类型[][] 数组名 = new 类型[][]{ 元素0,元素1,......,元素n };
    类型[][] 数组名 = { 元素0,元素1,......,元素n };

### 2.3 二维数组访问

    数组名[下标1][下标2];

譬如arr[2][3];可以理解为包含两个数组的数组. 故

    数组名.length         //返回 数组名 的元素个数
    数组名[下标].length   //返回 数组[下标] 的元素个数

## 3 不规则数组

### 3.1 声明并初始化

    int[][] jaggedArray = { {1,3,5,7,9}, {0,2,4,6,8}, {11,22} };
    int[][] jaggedArray = { new int[]{1,3,5,7,9}, 
        new int[]{0,2,4,6,8}, new int[]{11,22} };
    int[][] jaggedArray = new int[][]{ new int[]{1,3,5,7,9}, new int[]{0,2,4,6,8}, new int[]{11,22} };
    int[][] jaggedArray = new int[3][ ];
    jaggedArray[0] = new int[]{1,3,5,7,9}; jaggedArray[1] = new int[]{0,2,4,6,8};; jaggedArray[2] = new int[]{11,22};

## 4 数组的工具类

### 4.1 Java.util.Arrays

`Java.util.Arrays`提供了一些提供了一系列的方法，可以用来对数组进行操作：

1. `sort()`：对数组进行排序
2. `binarySearch()`：使用二分法查找指定的键值（注意查找之前需要先排序）
3. `copyOf()`和`copyOfRange()`：复制数组，截取或使用默认值填充
4. `toString()`：返回指定数组内容的字符串表示形式
5. `fill()`:使用指定的值类型填充数组
6. `hashCode()`：产生数组的散列码
7. `asList()`：将指定数组转换成List类型，注意asList()方法返回的ArrayList是Arrays的私有静态内部类，它不允许我们向返回的容器中加入或者移除元素

### 4.2 System.arraycopy

`System.arraycopy`提供了拷贝数组的高效方法，它的定义是：`System.arraycopy(src, scrPos, dest, destPos, length)`。注意，该方法不会自动包装和自动拆包，所以Integer[]和int[]是不能相互复制的。

使用示例：

    int[] arr = new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int[] dest = new int[5];
    System.arraycopy(arr, 4, dest, 2, 3);
    System.out.println(Arrays.toString(dest));

输出结果是：

    [0, 0, 4, 5, 6]
	
可以看出它的效果是：将arr中从索引4开始的连续3个元素拷贝到dest中从2开始向后的所有元素中。并且这里要求复制到dest中的元素个数必须小于dest定义的个数。

### 4.3 数组克隆

`.clone()`方法不能直接用于多维数组，要用的话也要在每一维上使用`clone()`方法

    int[] arr = {1,2,3,4,5}; int[] arr2 = arr.clone();

常用的复制数组方法总结

1. 使用循环；
2. 使用数组变量的`.clone()`方法，简单但是不灵活；
3. 使用`System.arraycopy()`方法. 简单灵活；
4. `Java,util.Arrays.copyOf`/`copyOfRange()`方法. 简单灵活。

## 5 数组与泛型

1.不能实例化具有参数化类型的数组，如下面的两种实例化方式中第一种是合法的，而第二种是不合法的。

    Bag<Integer>[] bags = new Bag[5]; // 合法
    // Bag<Integer>[] bags = new Bag<Integer>[5]; // 不合法
    for (int i=0;i<5;i++) {
        bags[i] = new Bag();
        bags[i].value = i;
    }

2.不能创建泛型数组，但是可以创建Object[]数组，然后将其强制转换成泛型数组：

    private static class Bag<T> {
        // T[] ts = new T[5]; // 非法
        T[] ts = (T[]) new Object[5]; // 合法，但是会在编译期得到“不受检查”的警告信息
    }