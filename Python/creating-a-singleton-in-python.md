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

**方法二：基类**

```python
class Singleton(object):
    _instance = None
    def __new__(class_, *args, **kwargs):
        if not isinstance(class_._instance, class_):
            class_._instance = object.__new__(class_, *args, **kwargs)
        return class_._instance
    
class MyClass(Singleton, BaseClass):
    pass
```

优点：

* `MyClass()`返回的确实是个类

缺点：

* 多继承！在继承第二个基类`BaseClass`的过程中，`__new__`方法是否可能被覆盖？这个得好好想想

**方法三：元类（[metaclass](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)）**

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
    
# python2
class MyClass(BaseClass):
    __metaclass__ = Singleton
    
# python3
class MyClass(BaseClass, metaclass=Singleton):
    pass
```

优点：

* `MyClass()`返回的确实是个类
* 自动神奇地覆盖了继承
* 使用`__metaclass__`用于其正确的目的

缺点：

* 暂时没想到

**方法四：装饰器返回一个同名的类**

```python
def singleton(class_):
    class class_w(class_):
        _instance = None
        def __new__(class_, *args, **kwargs):
            if class_w._instance is None:
                class_w._instance = super(class_w, class_).__new__(class_, *args, **kwargs)
            return class_w._instance
        def __init__(self, *args, **kwargs):
            if self._sealed:
                return
            super(class_w, self).__init__(*args, **kwargs)
            self._sealed = True
    class_w.__name__ = class_.__name__
    return class_w
@singleton
class MyClass(BaseClass):
    pass
```

优点：

* `MyClass()`返回的确实是个类
* 自动神奇地覆盖继承

缺点：

* 创建每个新类是否没有开销？在这里，我们为每个希望作为单例的类创建了两个类。这也许不好扩展
* 无法使用super()在基类上调用同名称的方法，因为它们会递归。这意味着你无法再自定义`__new__`并且不能将需要调用`__init__`的类子类化

## 答案

### No.1

**使用元类**

我推荐上述的方法二，但最好使用元类而不是基类。如下是一个示例实现：

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
# python2    
class Logger(object):
    __metaclass__ = Singleton
# python3
class Logger(metaclass=Singleton):
    pass
```

如果要在每次调用类时运行`__init__`，添加如下代码到`if`分支

```python
        else:
            cls._instances[cls].__init__(*args, **kwargs)
```

关于元类有几点补充。元类是类的类，也就是说，类是其元类的实例。在Python中，你可以通过`type(obj)`获取对象`obj`的元类。上面代码中的`Logger`类型为'your_module.Singleton'类型，就像`Logger`的（唯一）实例类型为'your_module.Logger'类型一样。当你使用Logger()调用logger时，Python首先会询问`Logger`，`Singleton`的元类，该怎么做，并允许实例被抢占式的创建。此过程与当你通过`myclass.attribute`引用一个类的属性时，Python通过调用`__getattr__`向该类询问该如何做，是一个道理。

元类本质上决定了**类的定义意味着什么**以及如何实现该定义。例如，参见[http://code.activestate.com/recipes/498149/](http://code.activestate.com/recipes/498149/)，它基本上使用元类在Python中重新创建了C风格的结构体（struct）。帖子[What are your (concrete) use-cases for metaclasses in Python?](https://stackoverflow.com/questions/392160/what-are-your-concrete-use-cases-for-metaclasses-in-python) 也提供了一些示例，它们似乎通常与声明式编程有关，尤其是应用在ORMs中时。

在这种情况下，如果你采用**方法2**，并且在子类中定义`__new__`方法，那么当你每次调用`SubClassOfSingleton()`时，它都会被执行一次 —— 因为它是负责调用返回存储实例的方法。**若使用元类，它将只会被调用一次**，且在创建唯一的实例时调用。

通常，使用元类来实现单例是有意义的。单例是特殊的，因为只创建一次，而元类则是自定义创建类的方式。如果需要以其他方式自定义单例类的definition，则使用元类可提供更多的控制。

你的单例**不需要多重继承**（因为元类不是基类），但是对于使用多重继承的创建类的子类，你需要确保单例类是**第一个/最左边**的一个带有重新定义`__call__`的元类的类。 这不太可能成为一个问题。实例dict不在实例的命名空间中，因此不会意外覆盖它。

你还会听说，单例模式违反了“单一责任原则” —— 每个类应该**只做一件事**。这样，如果您需要更改另一处代码，您不必担心代码会搞乱一件事，因为它们是分开的并且是封装的。元类实现通过了此测试。元类负责执行该模式，创建的类和子类不需要知道它们是单例。**方法1**未通过此测试，因为您注意到“MyClass本身是一个函数，而不是类，因此您无法从中调用类方法。”

**Python2和Python3的兼容版本**

编写同时适用于Python2和3的版本需要使用稍微复杂的方案。由于元类通常是类型`type`的子类，因此可以使用它在运行时动态创建中间基类，并将其作为元类，然后将其用作公共Singleton基类的基类。如下图：

```python
# work in Python2 & 3
class _Singleton(type):
    """ A metaclass that creates a Singleton base class when called"""
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(_Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
    
class Singleton(_Singleton('SingletonMeta', (Object,), {})):pass

class Logger(Singleton):
    pass
```

比较讽刺的一点是，该方法是使用子类来实现元类的。一个可能的优点是，与纯元类不同，`isinstance(inst, Singleton)`将返回True。

### No.2

```python
class Foo(object):
    pass
some_global_variable = Foo()
```

python的模块只导入一次，其他都是过度思考。不要使用单例并尽量不使用全局变量。

