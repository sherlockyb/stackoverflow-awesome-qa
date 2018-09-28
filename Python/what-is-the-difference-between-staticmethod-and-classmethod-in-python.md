# Python中的@staticmethod和@classmethod有什么差异
## 问题

用@staticmethod修饰的函数和用@classmethod修饰的函数有什么区别

## 答案

### No.1

也许一些代码示例会有所帮助：注意函数**foo**、**class_foo**和**static_foo**调用签名的区别

```python
class A(object):
    def foo(self, x):
        print "executing foo(%s,%s)" %(self,x)
    
    @classmethod
    def class_foo(cls, x):
        print "executing class_foo(%s,%s)" %(cls, x)
    
    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)" %(x)
a=A()
```

下面是对象实例调用方法的常用方法。对象实例a隐式传递为第一个参数。

```py
a.foo(1)
# executing foo(<__main__.A object at 0xb7dbef0c>, 1)
```

调用**classmethods**时，对象实例的类cls将代替self作为第一个参数隐式传递给函数

```py
a.class_foo(1)
# executing class_foo(<class '__main__.A'>, 1)
```

你也可以使用类来调用class_foo。实际上，当你将一个方法定义为classmethod，那可能是因为你原本就打算用类来调用它而不是对象实例。`A.foo(1)`将抛出**TypeError**异常，而`A.class_foo(1)`则是合法的：

```py
A.class_foo(1)
# executing class_foo(<class '__main__.A'>, 1)
```

对于**staticmethods**，self（对象实例）和cls（类）都不会作为第一个参数隐式传递。它们的行为类似于普通函数，除了你可以从实例或类中调用它们。

```pyt
a.static_foo(1)
# executing static_foo(1)
A.static_foo('hi')
# executing static_foo(hi)
```

**静态方法的作用**——用于将与类有某些逻辑联系的方法组合在一块，**方便代码管理，仅此而已**。

`foo`只是一个函数，但当你调用`a.foo`时，你不仅仅得到一个函数，你还得到该函数的一个“部分应用”的版本，即对象实例a作为函数的第一个参数绑定。foo需要2个参数，而`a.foo`只需要1个参数。

a绑定到了foo，下面就是”绑定“一词的含义：

```py
print(a.foo)
# <bound method A.foo of <__main__.A object at 0xb7d52f0c>>
```

对于`a.class_foo`，a并没有绑定到`class_foo`，而是类`A`绑定到了`class_foo`。

```py
print(a.class_foo)
# <bound method type.class_foo of <class '__main__.A'>>
```

对于静态方法，`a.static_foo`仅仅返回了没有任何参数绑定的普通函数。`static_foo`需要的是1个参数，而`a.static_foo`需要的同样也是1个参数

```pyt
print(a.static_foo)
# <function static_foo at 0xb7d479cc>
```

当然，当你用类A调用`static_foo`时也是同样的结果：

```py
print(A.static_foo)
# <function static_foo at 0xb7d479cc>
```

### No.2

说一下@classmethod和@staticmethod修饰的方法之间的相似性与差异性：

**相似性**：它们都可以在类本身上调用，而不仅仅是类的实例。因此，它们在某种意义上都是Class的方法。

**差异性**：类方法将接收类本身作为第一个参数，而静态方法则不接受。

因此，静态方法在某种意义上来说并不局限于Class本身，只是因为它可能具有相关的功能而挂在那里。

```py
>>> class Klaus:
        @classmethod
        def classmthd(*args):
            return args

        @staticmethod
        def staticmthd(*args):
            return args

# 1. 不带参数调用classmethod
>>> Klaus.classmthd()  
(__main__.Klaus,)  # 类作为第一个参数隐式传递

# 2. 带一个参数调用classmethod
>>> Klaus.classmthd('chumma')
(__main__.Klaus, 'chumma')

# 3. 不带参数调用staticmethod
>>> Klaus.staticmthd()  
()

# 4. 带一个参数调用staticmethod
>>> Klaus.staticmthd('chumma')
('chumma',)
```

### No.3

要决定是否使用**@staticmethod**或**@classmethod**，得看你方法的内部实现：如果方法中访问了类中的其他变量或方法，则使用**@classmethod**。否则，如果你的方法没有接触到类的任何其他部分，那么使用**@staticmethod**。

```python
class Apple:
    _counter = 0
    @staticmethod
    def about_apple():
        print('Apple is good for you.')
        # 注意，你仍然可以访问该类的其他成员，只不过你得用class实例，这种方式毕竟不太优雅，例如：
        # @staticmethod
        #	print('Number of apples have been juiced:%s' % Apple._counter)
    @classmethod
    def make_apple_juice(cls, number_of_apples):
        print('Make juice:')
        for i in range(number_of_apples):
            cls._juice_this(i)
    @classmethod
    def _juice_this(cls, apple):
        print('Juicing %d...' % apple)
        cls._counter += 1
```





