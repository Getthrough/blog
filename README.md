# Welcome to getthrough's blog
It's a repository where I write something about programming. 

## Google Protocol Buffer(protobuf) 2018/6/9



## seventy-eight rules in Effective Java(Joshua Bloch, second edition )
### One: Consider static factory methods instead of constructors
示例:
```
    public Boolean valueOf(boolean b) {
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




