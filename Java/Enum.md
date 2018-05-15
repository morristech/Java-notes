# 枚举

## 1、枚举的示例

    public enum ProductType {
        NORMAL(0, "普通品"),
        SPECIAL(1, "特殊品"); // 这里的分号是必不可少的

        public final String name;
        public final int no;

        ProductType(int id, String name) {
		    this.id = id;
            this.name = name;
        }

        public static ProductType getPortraitById(String name) {
            for (ProductType type : values()){
                if (type.name.equals(name)){
                    return type;
                }
            }
            throw new IllegalArgumentException("illegal argument");
        }
    }

上面是一个枚举的示例，注意这里面的一些细节：

1. 首先，定义枚举的时候要用enum关键字，这只是替代了定义类的时候的class关键字；
2. 实际上枚举是隐式继承Enum.class的，所以，枚举本身也是一个类，并且上面用到的values()方法就来自于Enum；
3. 枚举通常用来表示那些不会进行修改的对象，所以我们可以根据需要向枚举中添加一些字段；
4. 枚举的字段可以是public的，因为它同时也是final的，所以，不用担心暴露得太多而无法控制的问题；
5. 因为所有的枚举类型都默认继承了Enum类，所以自定义枚举类型就不能再继承其他类了；
6. 枚举有一个ordinal字段，它表示的是指定的枚举值在所有枚举值中的位置（从0开始）。不过，我们通常倾向于自己实现自己的id来给枚举值标序，因为这样更有利于维护

### 1.使用接口组织枚举

虽然，枚举没有办法继承新的类，但是却可以实现接口。这里我们在接口内部定义一组枚举，用来表示同一个大类中的一些分组，然后每个枚举内部再定义一些具体的枚举值：

    public interface City {
        enum ChineseCity implements City {
            BEIJING, SHANGHAI, GUANGZHOU;
        }

        enum AmericanCity implements City {
            NEW_YORK, HAWAII, IOWA, WASHINGTON;
        }

        enum EnglishCity implements City {
            BRISTOL, CAMBRIDGE, CHESTER, LIVERPOOL;
        }
    }

这里我们定义的是一个城市的接口，借口内部定义了三个枚举，分别枚举了中国、美国和英国的城市。可以看出在这里我们使用接口定义了“城市”的抽象概念，然后在接口的内部定义了三个枚举，来对应三个不同的国家。这样就相当于在City到具体的枚举之间又增加了一个新的层次。定义完毕之后，我们可以这么使用：

    public static void main(String ...args) {
        City city = City.AmericanCity.IOWA;
        System.out.println(city);
        System.out.println(City.ChineseCity.BEIJING);
        System.out.println(City.AmericanCity.NEW_YORK);
        System.out.println(City.EnglishCity.LIVERPOOL);
    }

### 2.使用枚举增强枚举的扩展性

如下面的代码所示，我们定义了一个接口类型Operation来表示一些操作，其内部定义了一个执行的方法（需要注意的是，我们在使用enum定义枚举的时候，实现接口的方法的操作是在各个枚举值上面实现的）。我们可以先定义一些基本的枚举类型，如果我们要在原来的基础之上进行拓展的话，那么我们只需要实现Operation并添加新的枚举即可：

    public interface Operation {
        double apply(int num1, int num2);
    }

    public enum BasicOperation implements Operation {
        ADD() {
            public double apply(int num1, int num2) {
                return num1 + num2;
            }
        },
        MINUS() {
            public double apply(int num1, int num2) {
                return num1 - num2;
            }
        },
        TIMES() {
            public double apply(int num1, int num2) {
                return num1 / num2;
            }
        },
        DIVIDE() {
            public double apply(int num1, int num2) {
                return num1 * num2;
            }
        };
    }

    public enum ExtendedOperation implements Operation {
        EXP() {
            public double apply(int num1, int num2) {
                return Math.exp(num1);
            }
        };
    }

对以上定义的方法的一个调用：

    Operation operation = BasicOperation.ADD;
    System.out.println(operation.apply(7, 8));

使用上面的两行代码，我们可以轻易地得出结果为15.这是没有问题的，而且我们可以看出这里借助于枚举实现了策略模式。

### 2.EnumSet和EnumMap

这是两个适用于枚举类型的容器类型
