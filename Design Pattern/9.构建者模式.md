## 构建者模式

当一个类中包含的字段比较少的时候，我们直接在构造器中要求调用者在创建类的实例的时候传入各个字段就可以了。但是当类中包含的字段比较多的时候情况就会麻烦一些。

在类中参数比较多的时候，通常有以下几种处理方法：

1. 重叠构造器
2. 使用Setter和getter在创建实例之后为每个字段赋值
3. 使用构建者模式

### 1.重叠构造器

在这种方式中，我们需要提供多个构造器：第一个是只有必要参数的构造器，第二个是包含两个参数的构造器，第三个是包含三个参数的构造器……最后一个构造器包含所有可选参数。

但是，当类中的字段比较多的时候，这显然会使得构造器数量也变得很多，代码难以阅读。如果两个参数的类型相同，又容易出现错误。所以，当参数较少的时候，这种方式是可行的。对于参数比较多的情况，仍然不能解决问题。

### 2.JavaBeans模式

这种模式下，我们使用类的无参的构造函数创建类的实例，然后调用类的setter方法，为类的各个字段赋值。这种方式的问题是：因为构建类的实例的过程被分到了几个调用中，所以在构建过程中，JavaBeans可能处于不一致状态。它必须对外提供setter方法，所以我们无法将其做成一个不可变的类（外部总能修改类的字段）。

### 3.构建者模式

以下是构建者模式的一个使用的例子，这里本质上：我们先创建一个Builder，并在创建的时候为其各个字段赋值，在赋值的时候我们只可以选择需要的字段进行赋值。创建完毕之后将其赋值给Student，并作为构造器的参数传入，然后Student从Builder上取相应的字段赋值给自己对应的字段。

    public static void main(String ...args) {
        Student kmst = new Student.Builder()
                .firstName("K")
                .lastName("Mst")
                .age(5)
                .gender("male")
                .build();
    }

    private static class Student {
        private String firstName;
        private String lastName;
        private int age;
        private String gender;

        public static class Builder {
            private String firstName;
            private String lastName;
            private int age;
            private String gender;

            public Builder() {}

            public Builder firstName(String firstName) {
                this.firstName = firstName;
                return this;
            }

            public Builder lastName(String lastName) {
                this.lastName = lastName;
                return this;
            }

            public Builder age(int age) {
                this.age = age;
                return this;
            }

            public Builder gender(String gender) {
                this.gender = gender;
                return this;
            }

            public Student build() {
                return new Student(this);
            }
        }

        public Student(Builder builder) {
            this.firstName = builder.firstName;
            this.lastName = builder.lastName;
            this.age = builder.age;
            this.gender = builder.gender;
        }
    }

构建者模式的不足之处在于，我们需要先创建Builder，然后再创建需要的类的实例。虽然构建器的开销在实践中可能并不明显，但是在某些非常注重性能的情况下，可能就会有问题。另外，Builder模式使得代码非常冗长，因此只有在很多参数的时候才使用。







