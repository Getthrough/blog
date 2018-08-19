# Welcome to getthrough's blog
It's a repository where I write something about programming. 

## Google Protocol Buffer(protobuf) 2018/6/9



## seventy-eight rules in Effective Java(Joshua Bloch, second edition )
### One: Consider static factory methods instead of constructors
示例:
```
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
```
* 静态工厂方法的优点：
    1. **命名上的优势**。使用公共的构造方法创建对象，方法名只能与类名相同，有时候并不见名知意，使用静态工厂方法在命名上可以灵活控制。我们知道，两个方法同名并且参数相同时，可以通过调换参数的顺序实现方法签名的不同。但是对于方法的调用方来说是十分困惑的，而且容易发生错误调用，而静态工厂方法在方法参数相同的情况下，可以通过不同的命名进行明显区分。
    2. **并不一定每一次调用都创建一个新的对象**。如果静态工厂方法每次返回的都是同一个对象，那么在比较使用该方法返回的对象时，可以直接使用 == 符号而不需要使用 equals() 方法，== 符号在性能上更有优势。
    3. **可以返回返回值类型的任意子类型**。这意味着，只要返回的接口类型没有变化，那么不管服务方的代码逻辑如何变化，对于调用方来说都是无感知的。并且这也提高了服务方代码的可维护性。
    4. **可以减少创建对象时需要多个(层)参数情况的代码冗余**。比如：
    ```
        // 不使用静态工厂方法时，每次需要这样创建对象
        Map<String, List<String>> m = new HashMap<String, List<String>>();
        
        // 使用静态工厂方法(截止 JDK 1.8.0_171 版本未支持)：
        public static <K, V> HashMap<K, V> getInstance() {
            return new HashMap<K, V>();
        }
        // 后续在使用时
        Map<String, List<String>> m = HashMap.getInstance();
    ```

* 静态工厂方法的缺点：
    1. **只有静态方法的类无法被继承**。
    2. **浏览API文档时，难以发现它与普通静态方法的不同，而构造器比较明显**。

### two: Consider a builder when face with many constructor parameters
示例：
```
// 手机类，有多个参数（必须的和可选的）
public class Phone {
    private String brand;   //required
    private String model;   //required
    private Double price;   //required

    private String  color;      //optional
    private Integer capacity;   //optional
    private Double  weight;     //optional
    private Double  height;     //optional
    private Double  width;      //optional
    private Double  depth;      //optional

    // 静态成员类 Builder，拥有和外部类相同的参数
    public static class Builder {
        // required parameter
        private String brand;
        private String model;
        private Double price;
        // optional parameter, init to default value
        private String  color       = "";
        private Integer capacity    = 256;
        private Double  weight      = 6.14d;
        private Double  height      = 5.65;
        private Double  width       = 2.79d;
        private Double  depth       = 0.3;

        // 构造方法(构造必须的参数)
        public Builder(String brand, String model, Double price) {
            this.brand = brand;
            this.model = model;
            this.price = price;
        }

        public Builder color(String color) {
            this.color = color;
            return this;
        }
        public Builder capacity(Integer capacity) {
            this.capacity = capacity;
            return this;
        }
        public Builder weight(Double weight) {
            this.weight = weight;
            return this;
        }
        public Builder height(Double height) {
            this.height = height;
            return this;
        }
        public Builder width(Double width) {
            this.width = width;
            return this;
        }
        public Builder depth(Double depth) {
            this.depth = depth;
            return this;
        }
        // 构建对象的方法，调用私有的构造器，进行属性的复制
        public Phone build() {
            return new Phone(this);
        }
    }
    // 私有的构造方法
    private Phone(Builder builder) {
        // 拷贝字段值
        this.brand    = builder.brand;
        this.model    = builder.model;
        this.price    = builder.price;
        this.color    = builder.color;
        this.capacity = builder.capacity;
        this.weight   = builder.weight;
        this.height   = builder.height;
        this.width    = builder.width;
        this.depth    = builder.depth;
    }
}

// 客户端调用方式
Phone iPhoneX = new Phone.Builder("iPhone", "X", 999d)
                .capacity(64)
                .color("Silver")
                .depth(0.3d)
                .height(5.65d)
                .weight(6.14d)
                .width(2.79)
                .build();
```
Builder 模式的优点在于创建对象时可以很明确的看到哪些参数被设置，同时编写代码也十分简洁和顺畅。如果使用重叠的构造器，对于多个参数的对象而言，调用构造器时传入多个参数往往令人困惑，不像 builder 一样一眼可以看出来设置了哪些参数。如果使用 JavaBean 的构建方式（无参构造+getter/setter），其构造过程被分到了几个 setter 方法调用中，因此在构造过程中，对象可能会处于不一致的状态，builder 模式创建的则是完整的对象，过多考虑线程安全性。
### three: Enforce the singleton property with a private constructor or an enum type
使用私有构造器或者枚举类型来强化“单例”属性。
单例模式想必是程序们最熟悉的一个设计模式了。它有几种实现方式，比如 public static final 修饰的对象引用：
```
public class FileSystem {
    // for every call, always get the same reference
    public static final FileSystem INSTANCE = new FileSystem();
    private FileSystem(){}
}
```
或者是静态工厂模式：
```
public class FileSystem {
    private static FileSystem INSTANCE = new FileSystem();
    private FileSystem(){}
    // static factory
    public static FileSystem getINSTANCE() {
        return INSTANCE;
    }
}
```
以上两种方式在对象序列化时，或者使用反射创建对象时，都无法保证单例。为保证序列化单例，需要在该对象中添加一个 readResolve() 方法：
```
private Object readResolve() {
        return INSTANCE;
    }
```
那么在 JDK 1.5 之后，可以使用枚举（enmu）创建保证序列化单例的单例模式：
```
public enum FileSystem {
    INSTANCE;
}
```
其实，对于枚举类，使用反射创建对象是会报“枚举类无法实例化的异常”，也就是说，枚举类的单例既保证了序列化时单例也保证了使用反射时的单例。
“使用枚举类创建单例是最好的方式”，Joshua Bloch 说到。
### four: Enforce noninstantiability with a private constructor
对于一些工具类而言，它所拥有的都是静态方法和字段，比如 java.lang.Math, java.util.Arrays，这些工具类在使用时并不希望使用者去创建实例。对于一个类，如果不去手动编写它的构造方法，那么编译器会提供一个无参的构造用于类的初始化。那如何防止调用者创建该类的实例呢？将该类设计成抽象类吗？不是。抽象类的一大用意是让子类继承，而子类在正常情况下是可以被实例化的。既然在不手动提供构造器时，编译器会提供默认无参构造器，那么我们手动提供一个**私有的构造器**就可以了。
```
// Noninstantiable utility class
public class UtilityClass {
    // 私有构造，为了防止在该类中无意调用了构造，可抛出异常提示（并不是严格要求需要）
    // 最好写上注释，指出私有属性
    private UtilityClass() {
    throw new AssertionError();
    }
}
```
### five: Avoid creating unnecessary objects
避免创建不必要的对象。
书上一个例子：
```
String s = new String("stringette");// Don't do this!
```
这是非常不明智的做法，因为"stringette"本身就是一个字符串，它存在于方法区(method area)的常量池。当使用 new 关键字创建对象，那么该语句所在的类**被类加载器加载时（解析阶段）**会在常量池创建一个字符串字面量（literal），当 new 语句执行时，会在堆内存（heap）又创建一个对象，那么可以理解为：当该语句存在于一个 n 次的循环中，运行时阶段将会创建 n+1 个对象。而`String s = "stringette";`则只会在类加载时创建一次对象，后续对于该字符串的引用将返回同一个指向常量池中的内存地址。
对于一些比较重量级的对象，考虑是否能减少创建的次数，比如可否在类加载时作为静态字段加载，这样既减少了内存占用，也提高了运行效率。
同样的，对于自动拆装箱（autoboxing）的类型转换，如果在循环中进行，同样耗时也是可观的。
需要注意的是，这条需要结合**第 39 条**理解，因为创建不必要的对象也仅仅是影响了性能和代码的风格，并不会造成错误的结果。

### six: Eliminate obsolete object references
删除“过期”的对象引用。(过期引用：再也不会被取消引用的引用)
书中举了一个栈（Stack）的模型结构：
```
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
        throw new EmptyStackException();
        return elements[--size];
    }
    /**
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
pop() 方法将栈顶的元素获取后指针往后退一位，此时 pop 出来的对象存在着过期的引用无法被垃圾回收期回收。当元素不断的进栈出栈，愈来愈多的过期对象可能会造成内存溢出（out of memory OOM）的风险。

`elements[size] = null;` 将数组栈顶位置指向 null 即可消除过期引用了。
在使用缓存时，也比较容易发生内存泄漏。将对象引用放入缓存中，它就容易被遗忘掉，导致在很长的一段时间里它不被使用但也仍然存在与缓存中。如果想要实现一种这样的缓存：在缓存之外有指向缓存键值对中的键，那么该键值对就不会失效。那么可以使用 `WeakHashMap` 实现。

如果一个客户端在一个 API 中注册了回调，但是没有显式地取消注册，那么很有可能它们会积聚起来。想要确保它们被垃圾回收，最好保存它们地弱引用，如保存成 `WeakHashMap` 中地键。

### Item 7 Avoid finalizers


### Item 8 Obey the genernal contract when  overriding **equals**




