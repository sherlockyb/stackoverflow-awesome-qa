# 用一行代码初始化ArrayList
## 问题

为了测试，我想创建一个选项列表。首先我是这样做的：

```java
ArrayList<String> places = new ArrayList<String>();
places.add("Buenos Aires");
places.add("Córdoba");
places.add("La Plata");
```

然后我将代码重构如下：

```java
ArrayList<String> places = new ArrayList<String>(
    Arrays.asList("Buenos Aires", "Córdoba", "La Plata"));
```

请问还有更好的方法吗？

## 答案

### No.1

可使用实例初始化创建一个匿名内部类的方式（也称为“双括号初始化”-**double brace initialization**）：

```java
ArrayList<String> list = new ArrayList<String>() {{
    add("A");
    add("B");
    add("C");
}};
```

然而，我并不太喜欢那种方法，因为你最终得到的是ArrayList的一个拥有实例初始化器的子类，并且该类创建的目的只是为了创建这一个对象，这对我来说似乎有点矫枉过正。【关于double brace initialization有几点需要提的：因为它的每次调用都会创建一个匿名内部类，如果使用不当，会造成类爆炸；此外，也是由于匿名内部类的原因，可能会造成内存泄露，具体可参考[What is Double Brace initialization in Java](https://stackoverflow.com/questions/1958636/what-is-double-brace-initialization-in-java)——**笔者补充**】

另外，如果[Project Coin](http://openjdk.java.net/projects/coin/)项目中的[Collection Literals](http://mail.openjdk.java.net/pipermail/coin-dev/2009-March/001193.html)提议会被JDK接受，将可直接使用集合字面的方式初始化：

```java
List<String> list = ["A", "B", "C"];
```

但不幸的是，这并不会帮到你，因为它会得到一个不可变的List而不是一个ArrayList。

### No.2

如果你只是把它声明为List，将会更简单，没必要声明为ArrayList：

```java
List<String> places = Arrays.asList("Buenos Aires", "Córdoba", "La Plata");
```

或者如果你只有一个元素的话：

```java
List<String> places = Collections.singletonList("Buenos Aires");
```

但这两种方式得到的places都是不可变的。【这里要解释下，这两个places的不可变是不一样的！前者是因为返回的ArrayList直接采用所传入的外部数组作为存储，而数组大小不可变，所以这里的ArrayList不能再增删元素，但是，列表中的元素值是可修改的，即set可用。而后者返回的，不仅不能增删元素，而且列表中唯一的元素也是不可改的，因为持有该元素的引用是final的，具体可看源码——**笔者补充**。】

想要创建一个ArrayList类型的可变列表，你可以将不可变列表用ArrayList包装得到：

```java
ArrayList<String> places = new ArrayList<>(Arrays.asList("Buenos Aires", "Córdoba", "La Plata"));
```

### No.3

简单来说，在Java8及之前：

```java
List<String> strings = Arrays.asList("foo", "bar", "baz");
```

上面将返回一个以传入数组为存储的List，因此长度不可变。但是List.set是可用的，因而它仍然是可变的。

而在Java9中：

```
List<String> strings = List.of("foo", "bar", "baz");
```

将返回一个immutable的List，它是不可变的【这里的不可变跟前面提到的Collections.singletonList类似，是指列表长度、元素都不可改，即整个列表是只读的，线程安全。就跟Scala的不可变List一个道理——**笔者补充**】。

直接用Java8提出的Stream s

```java
String<String> strings = Stream.of("foo", "bar", "baz");
```

拼接Stream s:

```java
String<String> strings = Stream.concat(Stream.of("foo", "bar",), Stream.of("baz", "qux"));
```

或者通过Stream构建List：

```java
List<String> strings = Stream.of("foo", "bar", "baz").collect(toList());
```

如果你确实需要一个java.util.ArrayList，就像[JEP 269](http://openjdk.java.net/jeps/269)中所说：

> There is a **small set** of use cases for initializing a mutable collection instance with a predefined set of values. It's usually preferable to have those predefined values be in an immutable collection, and then to initialize the mutable collection via a copy constructor.

就按下面的做吧：

```java
List<String> strings = new ArrayList<>(Arrays.asList("foo", "bar", "baz"));	// jdk <= Java8
List<String> strings = new ArrayList<>(List.of("foo", "bar", "baz"));	// jdk = java9
List<String> strings = Stream.of("foo", "bar", "baz").collect(toCollection(ArrayList::new));
```

### No.4

用Guava你可以这么写：

```java
List<String> places = Lists.newArrayList("Buenos Aires", "Córdoba", "La Plata");
```

### No.5

在Java9中可以很容易地用单行来初始化ArrayList：

```java
List<String> places = List.of("Buenos Aires", "Córdoba", "La Plata");	// immutable
List<String> places = new ArrayList<>(List.of("Buenos Aires", "Córdoba", "La Plata"));// mutable
```

Java9中新增的这些方法相比以前的有诸多优势：

1. [Space Efficiency](https://docs.oracle.com/javase/9/core/creating-immutable-lists-sets-and-maps.htm#JSCOR-GUID-6A9BAE41-A1AD-4AA1-AF1A-A8FC99A14199)
2. [Immutability](https://docs.oracle.com/javase/9/core/creating-immutable-lists-sets-and-maps.htm#JSCOR-GUID-4F3E2B7D-CE90-4862-A78A-414FC08DA6E4)
3. [Thread Safe](https://docs.oracle.com/javase/9/core/creating-immutable-lists-sets-and-maps.htm#JSCOR-GUID-DD066F67-9C9B-444E-A3CB-820503735951)

详情可参见 -> [What is the difference between List.of and Arrays.asList?](https://stackoverflow.com/questions/46579074/what-is-the-difference-between-list-of-and-arrays-aslist)

