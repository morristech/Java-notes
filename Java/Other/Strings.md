# 字符串相关问题总结

1. String的每一个看起来会修改String的方法实际都是创建一个全新的String对象，以包含修改后的字符串的内容，而最初的String对象则丝毫未动；
2. String使用正则表达式的方式：
	1. 匹配验证操作：`"asd".matches("[0-9]")`
	2. 分割操作：`"0as0d".split("[0-9]")`
	3. 替换操作：`"0a0sd".replaceAll("[0-9]", "09")`
3. 除了调用String对象的方法，还可以使用`Pattern`和`Matcher`两个类来使用正则表达式
4. String使用`char value[]`存储字符，所以String对象在创建之后就不能修改对象存储的字符串内容，正因如此才说String类型是不可变的
5. String类有一个特殊的创建方法，就是使用""创建。比如`new String("str")`实际上创建了两个对象：一个是通过""创建的，另一个是使用new创建。只是创建的时期不同：一个是在编译器，一个是在运行期。
6. 运行期间调用String的intern方法可以向String Pool中动态添加对象。如
		
        String s1 = "strs";
        String s2 = s1.intern();
	
	这个时候，如果我们使用`s1 == s2`进行判断的话会得到什么结果呢？答案是true. 参考intern方法的注释：如果pool中存在一个与当前String相等（所谓的相等是指使用equals方法判断时相等）的对象的时候，就返回pool中的那个对象。否则，就将当前的String添加到pool中。

7. 注意字符串拼接操作和==操作的优先级：

        String s = "str";
        System.out.println("s == s " + s == s);
        System.out.println("s == s " + (s == s));

	输出的结果是：

		false
		s == s true

	这是因为不管+号是用作加法还是用作连接字符串，它的优先级都要比“==”号要高。

8. 使用字符串拼接操作符`+`来拼接字符串，不适合运用在大规模的场景中，这是因为当两个字符串拼接到一起时，它们的内容都要被拷贝。

## 其他

1. String类型为0或多个双字节Unicode字符组成的序列，默认为null，这与””不同。所以，要判断一个字符串是否为空，可以使用下面的形式if(str!=null && str.length!=0)，即说明字符串不是null也不是””. 
2. 字符串不是字符数组，不能按照数组的方式访问
3. 比较字符串是否相等要用equals方法（C++可以用==，C语言使用strcmp）
4. 要由许多小段的字符串构建字符串，使用StringBuilder效率更高
5. 如果要修改字符串指定位置的字符，使用subString截取然后再使用+或者StringBuilder进行拼接即可
6. 文件路径中的反斜号前要增加一个额外的反斜号，如”c:\\myDir\\myFile.txt”
7. Java中字符串的长度是不可变的，这样设计是为了使字符串共享
8. char类型是采用UTF-16编码的Unicode代码点，大多数Unicode字符用一个代码单元，辅助字符需要两个代码单元，使用charAt的时候获取的是指定位置的代码单元，所以当字符串中存在需要两个代码单元的字符时，就容易出现错误。一般可以使用下面的形式获取每个代码点

        int cp = sentence.codePointAt(n);
        If(Character.isSupplementaryCodePoint(cp)) i+=2;
        else i++;

9. 使用String.format()静态方法可以创建一个格式化的字符串，而不输出，比如：`String msg=Sting.format(“Age is %d”,age);`
