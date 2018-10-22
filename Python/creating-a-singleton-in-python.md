# 在Python中创建单例

## 问题

本问题不是为了讨论单例模式是否可取，而是讨论如何在Python中以最佳的且最具Python风格的方式去实现它。

最佳方法列表：

**方法1：装饰器**

```python
def singletion(class_):
    instances = {}
    def getinstance(*args, **kwargs):
        if class_ not in instances:
            instances[class_] = class_(*args, **kwargs)
        return instances[class_]
    return getinstance

@singletion
class MyClass(BaseClass):
    pass
```

优点：

* 装饰器的添加方式通常比多重继承更直观

缺点：

* 虽然使用`MyClass()`创建的对象将会是真正的单例对象，但MyClass本身是一个函数，而不是类，因此你无法从中调用类方法。即若`m=MyClass()`,`n=MyClass()`，`o=type(n)()`，则有`m==n && m!=o&&n!=o`

## 答案

### No.1