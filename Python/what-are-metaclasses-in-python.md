# 什么是Python中的元类

## 问题

什么是元类，我们用它们做什么？

## 答案

### No.1

元类是类的类。就像类定义了实例对象的行为一样，元类定义了类的行为。类是元类的实例。

虽然在Python中你可以为元类使用任意的callables（如[Jerub](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python/100037#100037)所示）,实际上更有用的方法是使它自身成为一个真正的类。`type`是Python中常用的元类。正如你所猜想的，`type`自身就是一个类，它是自身的类型。你将无法在Python中重新创建一个纯粹像`type`一样的东西。要在Python中创建自己的元类，其实只需要构建`type`的子类即可。