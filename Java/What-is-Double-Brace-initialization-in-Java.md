# 什么是Java中的双括号初始化
## 问题

Java中的双括号初始化语法（{{...}}）是什么？

## 答案

### No.1

双括号初始化创建一个从指定类派生的匿名类（外括号），并在该类中提供一个初始化块（内括号）：

```java
new ArrayList<Integer>(){{
    add(1);
    add(2);
}};
```

### No.2

不推荐使用。除了语法不常用和别扭外，你还会不必要地引入两个棘手问题：

1. 你正在创建太多的匿名类

每当你使用双括号初始化时，都会创建一个新类，例如：

```java
Map source = new HashMap(){{
    put("firstName", "John");
    put("lastName", "Smith");
    put("organizations", new HashMap(){{
        put("0", new HashMap(){{
            put("id", "1234");
        }});
        put("abc", new HashMap(){{
            put("id", "5678");
        }});
    }});
}};
```

如上代码会产生下面一坨新类：

```shell
Test$1$1$1.class
Test$1$1$2.class
Test$1$1.class
Test$1.class
Test.class
```

这对于类加载器来说是相当大的开销！当然，如果你只是使用一次，它并不会花费太多初始化时间。但如果你在整个企业应用程序中这样做了20000次……所有那些耗费的堆内存仅仅是为了这么一点“语法糖”，不值得！

2. 你可能正在造成内存泄露！


如果你使用上面的代码并从方法返回一个map，该方法的调用者会持有外部类的引用，导致外部类无法被垃圾回收。当这个外部类占有了很多内存资源时，会导致大量内存的泄露。请考虑以下示例：

```java
public class ReallyHeavyObject {

    // Just to illustrate...
    private int[] tonsOfValues;
    private Resource[] tonsOfResources;

    // This method almost does nothing
    public Map quickHarmlessMethod() {
        Map source = new HashMap(){{
            put("firstName", "John");
            put("lastName", "Smith");
            put("organizations", new HashMap(){{
                put("0", new HashMap(){{
                    put("id", "1234");
                }});
                put("abc", new HashMap(){{
                    put("id", "5678");
                }});
            }});
        }};

        return source;
    }
}
```

返回的Map现在将包含对宿主类ReallyHeavyObject的实例引用。你可能不想冒这个风险：

![](images\1.png)

### No.3

- 第一个大括号创建一个新的匿名内部类
- 第二组大括号创建一个实例初始化器，如Class中的静态块。