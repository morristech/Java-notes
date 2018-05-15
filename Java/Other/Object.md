# Object类

所有类型都隐式的派生于java.lang.Object类，其主要用于两个目的：

1. 使用Object引用绑定任何数据类型的对象；
2. Object类型执行许多基本的一般用途的方法，包括equals(), finalize(), hashCode(),  getClass(), toString(), notify(), notifyAll()和wait()等. 

## 1、finilize方法

它是不可预测的，也是很危险的，一般情况下是不必要的。
它的出现只是对C++中的析构函数的一个妥协。
如果想要在类结束的时候释放占用的资源，可以使用try-catch结构来完成。

## 2、equals方法

equals方法需要遵循的规范：

1. 自反性：当x!=null,有x.equals(x)为true
2. 对称性：当x!=null且y!=null,有x.equals(y)与y.equals(x)结果相同
3. 传递性：当x!=null, y!=null, z!=null, x.equals(y)且y.equals(z)，那么x.equals(z)
4. 一致性：当x!=null，如果x和y没有修改过，那么x.equals(y)结果不变
5. 对任何x!=null，有x.equals(nyll)为false

覆写的诀窍：

1. 使用==操作检查“参数是否为这个对象的引用”
2. 使用instanceOf检查“参数是否为正确的类型”
3. 把参数转换成正确的类型
4. 对于该类中的每个“关键”域，检查参数中的域是否与该对象中的域相匹配
	1. 对于非float和double的基本类型域，使用==判断两个值是否相等
	2. 对于引用类型的域，可以使用equals方法判断两者是否相等
	3. 对于float域，可以使用Float.compare方法
	4. 对于double域，可以使用Double.compare方法
	5. 对于数组域，使用上述原则到每个元素，如果数组每个元素都很重要，可用Arrays.equals方法
5. 为了获得最佳性能，应该先比较最可能不一致的域，或者开销最低的域
6. 编写完equals方法之后，检查它们是否是：对称的、传递的、一致的
7. 覆盖equals方法时总要覆写hashCode方法
8. 不要将equals方法中的Object替换成其他类型（那就不是覆写了）

下面是一个示例程序，其中也包含了hashCode方法

    private static class Person {
        private long number;
        private int age;
        private String name;
        private float wage;
        private int[] id;

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof Person)) return false;

            Person person = (Person) o;

            if (number != person.number) return false;
            if (age != person.age) return false;
            if (Float.compare(person.wage, wage) != 0) return false;
            if (name != null ? !name.equals(person.name) : person.name != null) return false;
            return Arrays.equals(id, person.id);
        }

        @Override
        public int hashCode() {
            int result = (int) (number ^ (number >>> 32));
            result = 31 * result + age;
            result = 31 * result + (name != null ? name.hashCode() : 0);
            result = 31 * result + (wage != +0.0f ? Float.floatToIntBits(wage) : 0);
            result = 31 * result + Arrays.hashCode(id);
            return result;
        }
    }

## 3、hashCode方法

hashCode方法需要遵循的规范：

1. 只要用于equals方法比较的信息没有修改，hashCode多次调用应返回同样的值
2. 两个对象的equals方法返回结果相同，那么它们的hashCode方法的返回结果也应该相同
3. 两个对象的equals方法返回结果不同，它们的hashCode不一定要不同，但是如果不同的话，可以提高散列表的性能

计算hashCode的简单方法：

1. 把某个非零的常数值，比如17，保存在名为result的int型变量中
2. 对对象中的每个关键域f（用在equals方法中的域），完成以下步骤：
	1. 计算该域f的散列码c
		1. 若f为boolean型，计算f?1:0
		2. 若f为byte, char, short或int型，计算(int)f
		3. 若f为long型，计算(int)(f >>> 32)
		4. 若f为float型，计算Float.floatToIntBits(f)
		5. 若f为double型，计算Double.doubleToLongBits(f)，然后按long型处理
		6. 若f为对象引用，调用对象的hashCode()方法，如果对象为null返回0
		7. 若f为数组，对数组每个元素当作单独的域处理，如果数组所有元素都有意义，则用Arrays.hashCode()方法
	2. 按照下面公式将c合并到result，result = 31 * result + c
3. 返回result

示例代码可以参考equals方法，它是由IDEA自动生成的

## 4、clone方法：对象克隆

当我们使用`=`将一个引用类型赋值给另一个引用类型的时候，会使得两个引用类型共享一份资源的拷贝。
这时候修改一个引用会使的两个引用的内容都发生变化，而使用`clone()`就可以解决这个问题。
Clone之后的两个引用类型各自有自己的一份内容，不会相互影响。
但是，有一点值得注意的是，下面的代码中的CloneableClass中不存在引用类型，如果其中仍然存在引用类型的话，我们需要在clone方法中也将该引用类型clone一份。

    public static void main(String[] args) throws CloneNotSupportedException {
        // 测试1
        CloneableClass test1 = new CloneableClass("Test Class-01");
        CloneableClass cloned = test1.clone();
        cloned.inner.name = "clone2.inner.name"; // 修改了克隆对象的字段会反映到被克隆对象上
        System.out.println(test1.inner.name);
        // 测试2
        AnotherCloneableClass test2 = new AnotherCloneableClass("Test Class-02");
        AnotherCloneableClass anotherCloned = test2.clone();
        anotherCloned.inner.name = "anotherClone2.inner.name";
        System.out.println(test2.inner.name);
    }

    private static class InnerClass implements Cloneable{
        public String name;

        public InnerClass clone() throws CloneNotSupportedException{
            return (InnerClass)super.clone();
        }
    }

    private static class CloneableClass implements Cloneable{
        private String name;
        public InnerClass inner;

        public CloneableClass(String name){
            this.name = name;
            inner = new InnerClass();
        }

        public CloneableClass clone() throws CloneNotSupportedException{
            return (CloneableClass)super.clone();
        }
    }

    private static class AnotherCloneableClass extends CloneableClass{
        public AnotherCloneableClass(String name) {
            super(name);
        }

        public AnotherCloneableClass clone() throws CloneNotSupportedException{
            AnotherCloneableClass cloned = (AnotherCloneableClass) super.clone();
            cloned.inner = inner.clone();
            return cloned;
        }
    }

输出结果：

	clone2.inner.name
	null