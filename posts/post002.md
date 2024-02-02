# 用静态工厂方法代替构造器

对于类而言，为了让客户端获取它自身的一个实例，最传统的方法就是提供一个公共有的构造器。

还有一种方法，也应该在每个程序员的工具箱中占有一席之地。类可以提供一个共有的静态工厂方法（static factory method），它只是一个返回类的实例的静态方法。

例如来自Boolean类的示例

```java
public static Boolean valueOf(boolean b){
	return b ? Boolean.TRUE : Boolean.False;
}
```

注意，静态工厂方法与设计模式中的工厂方法（Factory Method）模式不同。

如果不通过公有的构造器，而是通过提供的静态工厂方法。有优势也有劣势。

#### 优势

**1、它们有名称**

如果构造器的参数本身没有确切地描述正在返回的对象，那么具有适当名称的静态工厂会更容易使用，产生的客户端代码也更容易阅读。

例如，构造器`BigInteger(int,int,Random)`返回的`BigInteger`可能为素数，但如果用`BigInteger.probablePrime`的静态工厂方法表示，显然更为清楚（Java 4 版本中增加了这个方法）

一个类只能有一个带有指定签名的构造器。编程人员通常知道如何避开这一限制：通过提供俩个构造器，它们的参数列表只在参数类型的顺序上有所不同。实际上这并不是一个好主意。面对这样的API，用户永远也记不住该用哪个构造器，结果常常会调用错误的构造器。如果没有参考类的文档，往往不知所云。

**2.不必每次调用都创建一个新对象**

可以将不可变类，预先创建好实例，或者缓存起来，进行重复利用，避免不必要的重复对象。`Boolean.valueOf(boolean)`方法说明了这项技术。类似于享元模式。如果程序经常创建相同的对象，并且创建对象代价很大，则这项技术可以很好地提高性能。

还有利于严格控制在某个时刻哪些实例应该存在。

**3、可以返回原返回类型的任何子类型的对象**

这种灵活性的一种应用是，API可以返回对象，同时又不会使对象的类变为公有的。以这种方式隐藏实现类会使API变得非常简洁。这项技术适用于基于接口的框架，因为在这种框架中，接口为静态工厂方法提供了自然返回类型。

Java 8 之后接口可以提供静态方法，但Java 8 中仍要求接口的所有静态成员必须是公有的。在Java 9 中允许接口有私有的静态方法，但是静态域和静态成员类仍然需要是公有的。

（*带有static标识符，这部分变量具有独立的存储空间，与对象无关，而是与整个类相关，类的所有实例共享静态域，不属于任何独立的对象。类没有创建实例，静态域也存在。*）

**4、所返回的对象的类可以随着每次调用而发生变化，这取决于静态工厂方法的参数值。**

只要是已经声明的返回类型的子类型，都是允许的。

EnumSet没有共有构造器，只有静态工厂方法。在OpenJDK实现中，它们返回俩种子类之一的一个实例，具体取决于底层枚举类型的大小。

**5、方法返回的对象所属的类在编写包含该静态工厂方法的类时可以不存在**

这种灵活的静态工厂方法构成了服务提供者框架的基础，例如 JDBC API。

服务提供者框架是指这样一个系统：多个服务提供者提供一个实现，系统为服务提供者的客户端提供多个实现，并把他们从多个实现中解耦出来。

#### 缺点

**1、类如果不含公有的或受保护的构造器，这就不能被子类化**

例如，要想将[Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html) 中的任何便利的实现类子类化，这是不可能的。

Java Collections框架中的核心实现类,如`ArrayList`,`HashMap`等都不能被继承子类化。

主要原因有两个:

1. 这些实现类的构造方法都是私有的,不能在类的外部通过构造方法实例化或者继承。
2. 这些实现类中没有公开任何可以继承的方法。其核心逻辑都是`private`或`final`的,不允许被覆盖。

**2、程序员很难发现它们**

在 API 文档里，他们没有像构造器那样在 API 文档里明确标识出来，因此查明如何实例化一个类是非常困难的。Javadoc工具总有一天会注意到静态工厂方法。

------

**静态工厂方法的一些惯用命名：**

- from - - - 类型转换方法，它只有单个参数，返回该类型的一个相对应的实例，例如：

  ```java
  Date d = Date.from(instance);
  ```

- of - - - 聚合方法，带有多个参数，返回该类型的一个实例，把它们合并起来，例如：

  ```java
  Set<Rank> faceCards = EnumSet.of(JACK,QUEEN,KING);
  ```

- valueOf - - - 比 from 和 of 更繁琐的一种替代方法，例如：

  ```java
  BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
  ```

- instance 或者 getInstance - - - 返回的实例是通过方法的（如有）参数来描述的，但不能说与参数具有同样的值，例如：

  ```java
  StackWalker luke = StackWalker.getInstance(options);
  ```

- create 或者 newInstance - - - 像 instance 或者 getInstance 一样，但 create 或者 newInstance 能够保证，每次调用都返回一个新实例，例如：

  ```java
  Object newArray = Array.newInstance(classObject,arrayLen);
  ```

- get*Type* - - - 像 getInstance 一样，但是在工厂方法处于不同的类中的时候使用。
  Type 表示工厂方法所返回的对象类型，例如：

  ```java
  FileStore fs = Files.getFileStore(path);
  ```

- new*Type* - - - 像 newInstance 一样，但是在工厂方法处于不同的类中的时候使用。
  Type 表示工厂方法所返回的对象类型，例如：

  ```java
  BufferedReader br = Files.newBufferedReader(path);
  ```

- type - - - get*Type* 和 new*Type* 的简版，例如：

  ```java
  List<Complaint> litany = Collections.list(legacyLitany);
  ```

-end-

**简而言之，静态工厂方法和公有构造器都各有用处，我们需要理解它们各自的长处。静态工厂方法经常更加合适，因此忌惮第一反应就是提供公有的构造器，而不考虑静态工厂方法。**

