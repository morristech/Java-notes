# MyBatis映射

今天在使用MyBatis从数据库读取数据的时候遇到了这个问题：

1. 向数据库写入数据的时候没有问题，自动生成的字段都能正确写入；
2. 从数据库读取的时候某些字段为null

以下是解决问题的时候尝试过的方法：

- 读取出现错误可能是数据库的数据类型与JavaBean的数据类型不匹配造成的：

因为代码中使用了自定义的枚举，我在每个枚举中定义了整型的mId，用它作为枚举的识别标识，并将其存储到数据库。因为使用了自定义枚举，于是我相应地为每个枚举类型定义了TypeHandler并将其注册到了MyBatis中。

此外，JavaBean中还是用了Date类型，这个疑点比较多。因为，我在代码中使用的是java.util.Date类型，然后在数据库中保存的是时间的毫秒。当然，我也相应地为Date定义了TypeHandler.

几经修改数据类型，无果。于是，我决定删除一部分字段，只保留几个字段来将问题更好地定位。

首先是Date类型的问题。我在代码中是按照下面的方式配置的：

    <typeHandler handler="my.shouheng.palmcollege.dao.tools.PalmDateTypeHandler"
                 javaType="java.util.Date" jdbcType="BIGINT"/>

因为本身在MyBatis中有一个DateTypeHandler，为了将其与自定义的TypeHander区别开来，我将自定义的TypeHandler命名为PalmDateTypeHandler. 

按照上面的方式定义有一个问题，因为我是按照下面的方式来查询数据的

    <select id="selectByPrimaryKey" parameterType="long" resultType="Assignment">
        select * from gt_assignment where id=#{id}
    </select>

在这里我并没有为每个字段指定Jdbc类型，因此就有一个问题——当对代码进行调试的时候，我发现当MyBatis对指定Java类型进行处理的时候，实际上是从一个HashMap中映射出TypeHandler的。而且，对Date类型处理时，jdbcType是null的。此时，它会获取JdbcType是null的TypeHandler。因为我自定义的TypeHandler的jdbcType是BIGINT，于是就无法使用我的TypeHandler。解决这个问题的方式是，将上述配置中的jdbcType="BIGINT"去掉，然后在自定义的TypeHandler中对可能为null的情况进行处理。

- 字段映射的方法

使用了上面的方法，我成功地解决了Date类型的问题。但是，还是无法解决全部问题。因为，仍然有一些字段没有办法映射到JavaBean中。于是乎，我试图从那些成功映射的和没有映射成功的字段中寻找规律，于是我发现那些成功映射的字段在JavaBean中的名称与在数据库中的名称一样。那些没有成功映射的字段属于那些由两个或以上单词构成的，在数据库中的名称是用下划线分开的，而在JavaBean中使用的是驼峰风格。

我尝试将那些驼峰风格的字段改成下划线的形式，终于它们也能成功映射了——现在终于发现问题的原因了。


虽然发现了问题，但是这种解决问题的方式仍然不够优雅——使用下划线形式定义JavaBean字段或者使用驼峰风格定义数据库字段。于是，我想使用其他的方法来解决这个问题。

思路当然比较清晰——只要找到MyBatis映射的机制，将映射方式修改掉即可。因为，在我的JavaBean中，为每个字段都假声了@Column注解。一开始，我以为它们的用途仅仅局限于用来生产各种文件。现在我觉得它可能被用来定义数据库字段，然后将其映射到数据库中。（之所以会这么想是因为之前使用的MyBatis被二次封装过，我以为是这个原因）

于是就是漫长的寻找映射代码的过程。

开始的一段时间寻找没有什么效果。后来找到了DefaultObjectFactory这里，它是用来获取JavaBean实例的工厂。于是，我想它的将数据库字段映射到JavaBean的逻辑可能就在这里。但是，很遗憾，这里的作用仅仅是获取一个所有字段为空的JavaBean实例。其实，本来应该早就找到这里的，我试过将JavaBean的构造方法定义为私有的，企图用抛异常的方式来定位，谁知被下面这几行代码挡住了

    if (!constructor.isAccessible()) {
        constructor.setAccessible(true);
    }

找到了生成对象的问题思路就更加清晰了——我们找到它在哪里被调用，那么在获取到对象之后肯定就是往对象里面塞值了。于是，我继续在代码中寻找DefaultObjectFactory这被调用的位置。想要通过逻辑查看方法被调用的位置来寻找似乎没有那么简单，因为它被嵌套的地方太多了，如果对MyBatis的整个架构不清楚的话，肯定很容易就绕晕了。于是，我用了调试工具里的“跳出”按钮，看它一层层回退到什么位置，并注意返回值的变化.

最终,将位置锁定在FastResultSetHandler的applyAutomaticMappings方法,并确定就是在这个方法里按照指定的规则将数据库中的列映射到JavaBean中的。只是，我又注意到在这里，当使用某种规则获取属性的时候传入了一个名为mapUnderscoreToCamelCase的参数。根据它的名字，我们可以大致猜测一下它的用途：将下划线映射到驼峰。于是，我将其在浏览器中搜索一下。果然，这是在MyBatis中可以配置的参数，也就是将其设置为true之后，就可以将数据库中的下划线类型的数据映射到JavaBean中的驼峰风格的字段中。

现在，看来将数据库中的下划线风格的数据映射到JavaBean中的驼峰风格的字段只要在MyBatis中配置一下即可。

总结如下：

1. 虽然，最终的解决问题的答案很简单，但是找到这个答案并不简单。
2. 我想出现这个问题的原因可能还是对MyBatis的配置不是非常熟悉。
3. 查找问题的时候尽量缩小查找问题的范围。
4. 使用调试工具来进行跳转。
5. 分析问题的时候除了技术上的因素之外，总结产生问题的规律也很重要。

