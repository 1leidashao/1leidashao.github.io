---
  layout:     post
title:      "深入理解Python特殊属性"
subtitle:   " \"python special Attributes\""
date:       2017-05-07 14:00:00
author:     "FengZhao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - python
---


我们知道，python的特殊方法的前后各有两个下划线，这篇文章，我们来看看前后各有两个下划线的属性，同样，这也是特殊属性。

特殊方法名的前后各有两个下划线。特殊方法又被成为魔法方法(magic method)，定义了许多Python语法和表达方式，正如我们在下面的例子中将要看到的。当对象中定义了特殊方法的时候，Python也会对它们有“特殊优待”。比如定义了\_\_init\_\_()方法的类，会在创建对象的时候自动执行\_\_init\_\_()方法中的操作。

可以通过dir()来查看对象所拥有的特殊方法，比如dir(1),我们把1看作是一个整数类的实例，执行dir(1)就是查看1这个整数对象拥有的特殊方法。




## \_\_dict\_\_ 属性

  Python中的每个对象都有一个\_\_dict\_\_属性。这个字典对象包含了这个对象本身的所有属性。它存储这些属性的名称与值的键值对。

  这是一个例子：
```python
class Person(object):
x=1
a=Person()
a.y=2`  
a.__dict__     # {'y':2}
```

  有一个奇怪的现象是，当我尝试直接使用a="xyz",然后打印print(a.\_\_dict\_\_)时，竟然报str对象没有\_\_dict\_\_属性，直接字符串或整数常量调用的时候，也会报没有这个属性，不知道这里是什么原因？

  注意到x并不在中，原因很简单，因为y是对象a的属性。而x是类Person的属性，它会出现在Person.\_\_dict\_\_属性中。事实上，Person的\_\_dict\_\_也包括许多键（包括'\_\_dict\_\_')：

```python
a.__class__.__dict__        # 等价于Person.__dict__
a.__class__.__dict__['x']   # 访问Person类的x属性
```

  对象的dict属性很好理解，它就是一个dict对象:
```python
a.__dict__.__class__   	 # dict 它返回对象a的dict属性的类名。
a.__dict__.__={}  	     # 显式将这个字典设置为空，即移除这个对象的所有属性。
a.__dict__.__['y']=5     # 等价于a.y=5,即将5赋给对象a的y属性。
```
  我们为a这个对象添加的属性，都可以通过a.\_\_dict\_\_来访问，它返回一个dict,包含这些属性的键值对。

  类的dict属性就没那么好理解了，实际上，它是dictproxy的一个对象，dictproxy是一个特殊类，它的对象就像普通的字典，但是它们某些键却不太一样.

```python
Person.__dict__.__.__class__   	# 它返回dictproxy，即类的dict属性所属的类型。
Person.__dict__.__ ={}   		# error,类的\_\_dict\_\_属性是不可写的。
Person.__dict__.__['x']=4 	    # 不可写，同样也会报错
Person.x=4       				# 为类添加一个属性
```
  我们发现，类的\_\_dict\_\_属性不可写(无法为这个字典添加键)，但是可以通过类名.属性的形式添加一个属性，完成同样的功能，因为这两者内部实现实际上是不同的。

  这看起来有点不好理解，如果没有看明白，多看几遍，应该就能明白是什么意思了。




## \_\_bases\_\_和\_\_class\_\_属性

要访问一个对象object的类型，我们使用instance.\_\_class\_\_属性来获取当前对象的类型，或者直接使用type(object)。

要访问一个对象的父类，则使用class.\_\_bases\_\_（注意只能使用类名）即可获取它的父类:
```python
class Person(object):
          pass
    class Student(Person):
            __init__(self,age):
                   self.age=age
    jack=Person()
    jack.__class__      #  person        
    tom=Student(10)
    tom.__class__       #  student
    Student.__bases__   #  Person,只能用类名来获取其父类名称
```

因此，比较两个对象的类型是否相同，我们可以直接比较这两个的\_\_class\_\_属性是否相等或者调用type()方法获取属性：
```python
jack.__class__ == tom.__class__  # False
type(jack) == type(tom)          # False
```

显然，这个方法和属性，只能返回当前对象的类，所以对于class的继承关系，使用type()就很不方便，我们要判断class的类型，可以使用isinstance()函数，它接受两个参数，第一个参数是object，即被判断的对象，第二个参数是classinfo，即被比较的类。如果object是这个类的实例，或者它的子类的实例，则返回True,否则返回False。看下面的例子：
```python
isinstace(tom,student)  # True,tom是stundent类的实例  
isinstace(tom,person)   # True，student继承自person类，所以student类的对象也可以被看作是person类型
```

python文档中对这个函数的描述是这样的：

> isinstance(object, classinfo)
> Return true if the object argument is an instance of the classinfo argument, or of a (direct, indirect or virtual) subclass thereof. If object is not an object of the given type, the function always returns false. If classinfo is a tuple of type objects (or recursively, other such tuples), return true if object is an instance of any of the types. If classinfo is not a type or tuple of types and such tuples, a TypeError exception is raised.

想一想，isinstance函数是怎么定义的？ 提示:先获取第一个参数的类名，然后用循环遍历出它的所有父类的类型，然后与第二个参数逐一比较。

## 特性

同一个对象的不同属性可能存在依赖关系，当其中某一个属性被修改时，我们希望依赖于该属性的其他属性也要变化。这时，通过\_\_dict\_\_的方式显然无法做到这一点。python提供了多种即时生成属性的方法，其中一种成为“特性”，特性是特殊的属性。比如，我们为Stundet类增加一个特性，当对象的age属性大于18时，adult为True;否则adult为False:
```python
class Person(object):
      pass
class Stundet(Person):         
      __init__(self,age):     # 构造函数,
          self.age=age
      def getAdult(self):     # 判断年龄，返回True或者False        
          if self.age>18.0:
              return True
          else:
              return False
      adult = property(getAdult)  # 使用内置函数property()创建特性。
```

特性是使用内置函数property()来创建。property函数最多可接收四个参数。前面三个参数为函数，分别用于获取，修改，删除特性值，最后一个参数创建一个说明字符串。

查看一下Python手册中关于property的说明：
>class property(fget=None, fset=None, fdel=None, doc=None)
>Return a property attribute.

>fget is a function for getting an attribute value. fset is a function for setting an attribute value. fdel is a function for deleting an attribute value. And doc creates a docstring for the attribute.

下面我们看一个例子：
```python
  class num(object):
      def __init__(self,value):  #构造函数，定义一个value代表数字本身
            self.value=value
      def getNeg(self):
            return -self.value

      def setNeg(self, value):
            self.value = -value

      def delNeg(self):
            del self.value

      neg=property(getNeg, setNeg, delNeg, "this is a negetive number attribute")  #定义一个特性x接收四个函数，
        a=num(1)  # 实例化一个数字对象1，并用变量a引用它
        a.neg     # -1 调用第一个参数getNeg，获取1这个对象的负数特性
        a.neg=-22 #  调用第二个参数setNeg，为这个对象的负数特性,修改其负数特性时，也会修改其value。
        del a.neg # 删除这个对象的负数属性，调用第三个参数delNeg,也删除了其value属性。
```

上面的num是一个数字类，neg作为这个类特性，表示数字的负数，当一个数字确定，那么它的负数也确定；而当我们修改一个数的负数特性时，它本身的值也应该变化，这两点通过getNeg和setNeg来实现。delNeg表示的是，如果删除特性neg，那么应该执行的操作是删除属性value。











参考

>  [Python深入 对象的属性](http://www.cnblogs.com/vamei/archive/2012/12/11/2772448.html)
