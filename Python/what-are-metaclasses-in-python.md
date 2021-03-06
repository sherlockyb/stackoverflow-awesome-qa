# 什么是Python中的元类

## 问题

什么是元类，我们用它们做什么？

## 答案

### No.1

元类是类的类。就像类定义了实例对象的行为一样，元类定义了类的行为。类是元类的实例。

虽然在Python中你可以为元类使用任意的callables（如[Jerub](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python/100037#100037)所示）,实际上更有用的方法是使它自身成为一个真正的类。`type`是Python中常用的元类。正如你所猜想的，`type`自身就是一个类，它是自身的类型。你将无法在Python中重新创建一个纯粹像`type`一样的东西。要在Python中创建自己的元类，其实只需要构建`type`的子类即可。

元类最常用作类工厂。就像你通过调用类名来创建类的一个实例一样，Python通过调用元类来创建一个新类（当它执行`class`语句）。结合常规的`__init__`和`__new__`方法，元类因此允许你在创建类时做“额外的事”，比如使用某个注册表注册新类，或者甚至完全用其他类替换该类。

当`class`语句被执行时，Python首先执行`class`语句的主体，就像执行正常的代码块一样。生成的命名空间（一个词典)保存了将要生成的类的属性。元类是通过在待生成类的`__metaclass__`属性或`__metaclass__`全局变量中查找待生成类的基类（继承元类）来确定的，然后调用元类并用类的名称、基类和属性来实例化它的。

但是，元类实际上定义了类的类型，而不仅仅是它的工厂，因而你可以用它们做更多的事。例如，你可以在元类上定义常规方法，这些元类方法类似于类方法，因为它们可以在没有实例的类上调用，但它们又不像类方法，因为它们不能在类的实例上调用。`type.__subclasses__()`是元类`type`中的一个方法示例，你也可以在其中定义常规的“魔法”方法，例如`__add__`，`__iter__`和`__getattr__`，以实现或改变类的行为。

下面是一些汇总示例：

```python
def make_hook(f):
    """Decorator to turn 'foo' method into '__foo__' """
    f.is_hook = 1
    return f

class MyType(type):
    def __new__(mcls, name, bases, attrs):
        if name.startswith('None'):
            return None
        # Go over attributes and see if they should be renamed
        for attrname, attrvalue in attrs.iteritems():
            if getattr(attrvalue, 'is_hook', 0):
                newattrs['__%__' % attrname] = attrvalue
            else:
                newattrs[attrname] = attrvalue
        return super(MyType, mcls).__new__(mcls, name, bases, newattrs)
    
    def __init__(self, name, bases, attrs):
        super(MyType, self).__init__(name, bases, attrs)
        # classregistry.register(self, self.interfaces)
        print "Would register class %s now." % self
    
    def __add__(self, other):
        class AutoClass(self, other):
            pass
        return AutoClass
        # Alternatively, to autogenerate the classname as well as the class
        # return type(self.__name + other.__name, (self, other), {})
       
    def unregister(self):
        # classregistry.unregister(self)
        print "Would unregister class %s now." % self

class MyObject:
    __metaclass__ = MyType

class NoneSample(MyObject):
    pass

# will print "NoneType None"
print type(NoneSamle), repr(NoneSample)

class Example(MyObject):
    def __init__(self, value):
        self.value = value
    @make_hook
    def add(self, other):
        return self.__class__(self.value + other.value)
    
# Will unregister the class
Example.unregister()

inst = Example(10)
# Will fail with an AttributeError
#inst.unregister()
print inst + inst
class Sibling(MyObject):
    pass

ExampleSibling = Example + Sibling
# ExampleSibling is now a subclass of both Example and Sibling (with no
# content of its own) although it will believe it's called 'AutoClass'
print ExampleSibling
print ExampleSibling.__mro__
```

### No.2

**类也是对象**

在理解元类之前，你需要掌握Python中的类。借鉴Smalltalk语言，Python对于类是什么有一个非常奇特的构想。

在大多数语言中，类就是描述如何生成对象的代码片段。这一点在Python中也是成立的。

```python
>>> class ObjectCreator(object):
...		pass
...
>>> my_object = ObjectCreator()
>>> print(my_object)
<__main__.ObjectCreator object at 0x8974f2c>
```

但是Python中的类远不止如此。类也是对象。只要你使用关键字`class`，Python将执行它并创建一个对象。下面的代码：

```python
>>> class ObjectCreator(object):
... 	pass
...
```

会在内存中创建一个对象，名称就是`ObjectCreator`。

该对象（类）本身具有创建对象（类实例）的能力，而这就是为什么它是一个类的原因。但是，它本质上仍然是一个对象，因而你能对其做如下操作：

- 你可以将其赋值给一个变量
- 你可以拷贝它
- 你可以为其添加属性
- 你可以将它作为函数参数传递

例如：

```python
>>> print(ObjectCreator) # 你可以打印一个类，因为它本身就是个对象
<class '__main__.ObjectCreator'>
>>> def echo(o):
...		print(o)
>>> echo(ObjectCreator) # 你能一个类作为参数传递
<class '__main__.ObjectCreator'>
>>> print(hasattr(ObjectCreator, 'new_attribute'))
False
>>> ObjectCreator.new_attribute = 'foo' # 你能向一个类添加新属性
>>> print(hasattr(ObjectCreator, 'new_attribute'))
True
>>> print(ObjectCreator.new_attribute)
foo
>>> ObjectCreatorMirror = ObjectCreator # 你能将一个类赋值给一个变量
>>> print(ObjectCreatorMirror.new_attribute)
foo
>>> print(ObjectCreatorMirror())
<__main__.ObjectCreator object at 0x8997b4c>
```

**动态地创建类**

