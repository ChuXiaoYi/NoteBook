
转载来自：https://www.cnblogs.com/ybjourney/p/8859519.html

Python2.5之后引入了上下文管理器（context manager），算是Python的黑魔法之一，它用于规定某个对象的使用范围。本文是针对于该功能的思考总结。

##为什么需要上下文管理器？
首先，需要思索下为什么需要引入上下文管理器。
在正常情况下，管理各种系统资源（如文件）、数据库连接时，通常是先打开这些资源，执行完相应的业务逻辑，最后关闭资源。
举两个例子:

> - 使用Python打开一个文件写入内容，之后需要关闭这个文件。如果不正常关闭的话可能会在文件操作时出现异常，因为系统允许你打开的文件的最大数是有限的。
> - 在数据库连接时也是存在类似问题，数据库的连接算是一种比较昂贵的资源，若连接过多而没有及时关闭的话，就可能出现不能继续连接的异常错误。

但是，很多程序员经常会忘记关闭文件，或者关闭数据库的连接。这时候就引入了上下文管理器，它可以在你不需要该对象的时候，自动关闭它。

##上下文管理器怎么使用？
上下文管理器的语法是：
```
with...as...
```
    
**实例：文件操作**
```python
print "不使用上下文管理器"
print "*" * 30
f = open('file.py', 'w')
print f.closed
f.write("# Hello World")
f.close()
print f.closed

print "\n使用上下文管理器"
print "*" * 30
with open("file.py", 'w') as f:
    print f.closed
    f.write('# Hello Python')
print f.closed
```
这里通过.closed比较，我们可以看到上下文管理器可以自动关闭文件，对于上下文管理器而言，有隶属于它的程序块，当隶属于它的程序块执行结束的时候（判断缩进），上下文管理器将自动关闭文件。
上述实例，也可以使用try...except...来实现，同样可以很直观的看到使用with...as...语句之后，代码确实相对更加简洁。

##上下文管理实现机制
因为文件对象是Python的内置对象，内置了上下文管理的特殊方法，所以它可以使用with语句。在Python中，任何对象，只要实现了上下文管理，就可以使用with语句，实现上下文管理需要通过__enter__和__exit__这两个方法来实现。
关于这两个方法：

 - enter(self)：进入该对象时调用此方法，返回值将放入with...as...语句中的as说明的变量中
 - exit(self, type, value, tb):离开上下文管理器时调用该方法，如果有异常出现，返回False，type、value和tb将分别表示异常的类型、值和追踪信息，传递出上下文显示；如果没有异常，则三个变量的值均为None。
```
with 上下文管理器：
    语法体
```
当with语句遇到上下文管理器时，就会在执行语法体之前，先执行__enter__方法，然后再执行语法体，执行完语法体之后，执行__exit__方法。

##上下文管理器实现
使用Python2.7X实现一个上下文管理器:
```python
class Context(object):

    def __init__(self):
        print "实例化一个对象"

    def __enter__(self):
        print "获取该对象"

    def __exit__(self, exc_type, exc_val, exc_tb):
        print "退出该对象"

temp = Context()

with temp:
    print "执行体"
```
这样，__enter__方法和__exit__方法的调用过程就很明晰。

##contextLib
在contextlib中，提供了contextmanager装饰器，通过yield返回函数将函数分隔为两部分，yield之前的语句在__enter__中执行，yield之后的语句在__exit__中执行，简化了上下文管理器的实现方式：

总结：通过上下文管理器，我们可以更好的控制对象在不同区间的特性，并且可以使用with语句替代try...except方法，使得代码更加的简洁，主要的使用场景是访问资源，可以保证不管过程中是否发生错误或者异常都会执行相应的清理操作，释放出访问的资源。



