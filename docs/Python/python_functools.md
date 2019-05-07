!!! quote "参考文档"

    - [https://zhuanlan.zhihu.com/p/27643991](https://zhuanlan.zhihu.com/p/27643991)
    - [https://docs.python.org/3.7/library/functools.html#functools.lru_cache](https://docs.python.org/3.7/library/functools.html#functools.lru_cache)
    - [https://segmentfault.com/a/1190000009398663](https://segmentfault.com/a/1190000009398663)

### **functools.cmp_to_key(func)**
将旧式比较函数转换为关键字函数。与接受字关键函数(如sort()、min()、max()、heapq. nbiggest()、heapq.nsmallest()、itertools.groupby())的工具一起使用。该函数主要用于从Python 2转换过来的程序的转换工具，Python 2支持使用比较函数。

比较函数是任何可调用的函数，它接受两个参数，进行比较，然后返回负数(小于)、零(相等)或正数(大于)。键函数是一个可调用函数，它接受一个参数并返回另一个值作为排序键。

放上一波源码：
```python
################################################################################
### cmp_to_key() function converter
################################################################################

def cmp_to_key(mycmp):
    """Convert a cmp= function into a key= function"""
    class K(object):
        __slots__ = ['obj']
        def __init__(self, obj):
            self.obj = obj
        def __lt__(self, other):
            return mycmp(self.obj, other.obj) < 0
        def __gt__(self, other):
            return mycmp(self.obj, other.obj) > 0
        def __eq__(self, other):
            return mycmp(self.obj, other.obj) == 0
        def __le__(self, other):
            return mycmp(self.obj, other.obj) <= 0
        def __ge__(self, other):
            return mycmp(self.obj, other.obj) >= 0
        __hash__ = None
    return K
```

写一个demo，看一下运行流程：
```python
import functools


class MyObject:

    def __init__(self, val):
        self.val = val

    def __str__(self):
        return 'MyObject({})'.format(self.val)


def compare_obj(a, b):
    """Old-style comparison function.
    """
    print('comparing {} and {}'.format(a, b))
    if a.val < b.val:
        return -1
    elif a.val > b.val:
        return 1
    return 0

# 排序的key
get_key = functools.cmp_to_key(compare_obj)

def get_key_wrapper(o):
    "Wrapper function for get_key to allow for print statements."
    new_key = get_key(o)
    print('key_wrapper({}) -> {!r}'.format(o, new_key))
    return new_key

objs = [MyObject(x) for x in range(5, 0, -1)]

for o in sorted(objs, key=get_key_wrapper):
    print(o)
```
通常情况下，`cmp_to_key()` 将直接使用，但在本例中引入了一个额外的包装函数，以在调用关键函数时输出更多信息。
输出如下：
```
key_wrapper(MyObject(5)) -> <functools.KeyWrapper object at 0x10692e510>
key_wrapper(MyObject(4)) -> <functools.KeyWrapper object at 0x10692e4f0>
key_wrapper(MyObject(3)) -> <functools.KeyWrapper object at 0x10692e4d0>
key_wrapper(MyObject(2)) -> <functools.KeyWrapper object at 0x10692e470>
key_wrapper(MyObject(1)) -> <functools.KeyWrapper object at 0x10692e490>
comparing MyObject(4) and MyObject(5)
comparing MyObject(3) and MyObject(4)
comparing MyObject(2) and MyObject(3)
comparing MyObject(1) and MyObject(2)
MyObject(1)
MyObject(2)
MyObject(3)
MyObject(4)
MyObject(5)
```

### **@functools.lru_cache(maxsize=128, typed=False)**
这个装饰器实现了备忘的功能，是一项优化技术，把耗时的函数的结果保存起来，避免传入相同的参数时重复计算。lru 是（least recently used）的缩写，即最近最少使用原则。表明缓存不会无限制增长，一段时间不用的缓存条目会被扔掉。 
这个装饰器支持传入参数，还能有这种操作的？maxsize 是保存最近多少个调用的结果，最好设置为 2 的倍数，默认为 128。如果设置为 None 的话就相当于是 maxsize 为正无穷了。还有一个参数是 type，如果 type 设置为 true，即把不同参数类型得到的结果分开保存，如 f(3) 和 f(3.0) 会被区分开。 

由于字典用于缓存结果，因此函数的位置和关键字参数必须是hashable的。


如果maxsize设置为None，则禁用LRU特性，缓存可以无限制增长。当maxsize为2次方时，LRU特性表现最佳。


如果将类型设置为true，则将分别缓存不同类型的函数参数。例如，f(3)和f(3.0)将被视为具有不同结果的不同调用。


为了帮助度量缓存的有效性并调优maxsize参数，封装的函数使用cache_info()函数进行检测，该函数返回一个命名元组，显示hits(命中), misses(未命中)、maxsize和currsize。在多线程环境中，得失是近似的。


decorator还提供了一个cache_clear()函数，用于清除或使缓存失效。


原始的底层函数可以通过__wrapped__属性访问。这对于内省、绕过缓存或使用不同的缓存重新包装函数非常有用。


写了个函数追踪结果
```python
def track(func):
    @functools.wraps(func)
    def inner(*args):
        result = func(*args)
        print("{} --> ({}) --> {} ".format(func.__name__, args[0], result))
        return result
    return inner
```
递归函数适合使用这个装饰器，那就拿经典的斐波那契数列来测试吧 

不使用缓存
```python
@track
def fib(n):
    if n < 2:
        return n
    return fib(n - 2) + fib(n - 1)
```

使用缓存
```python
@functools.lru_cache()
@track
def fib_with_cache(n):
    if n < 2:
        return n
    return fib_with_cache(n - 2) + fib_with_cache(n - 1)
```
测试代码
```
fib(10)

fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (1) --> 1 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (3) --> 2 
fib --> (4) --> 3 
fib --> (1) --> 1 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (3) --> 2 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (1) --> 1 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (3) --> 2 
fib --> (4) --> 3 
fib --> (5) --> 5 
fib --> (6) --> 8 
fib --> (1) --> 1 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (3) --> 2 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (1) --> 1 
fib --> (0) --> 0 
fib --> (1) --> 1 
fib --> (2) --> 1 
fib --> (3) --> 2 
fib --> (4) --> 3 
fib --> (5) --> 5 
fib --> (0) --> 0 
····省略····

时间花费：0:00:00.001295
```
```
fib_with_cache(10)

fib_with_cache --> (0) --> 0 
fib_with_cache --> (1) --> 1 
fib_with_cache --> (2) --> 1 
fib_with_cache --> (3) --> 2 
fib_with_cache --> (4) --> 3 
fib_with_cache --> (5) --> 5 
fib_with_cache --> (6) --> 8 
fib_with_cache --> (7) --> 13 
fib_with_cache --> (8) --> 21 
fib_with_cache --> (9) --> 34 
fib_with_cache --> (10) --> 55 
时间花费：0:00:00.000117
```
可以很明显的看到，使用缓存的时候，只调用了 11 次就得出了结果，并且花费时间只为 0.000117 秒

我们再把数字调大，传入的参数改为 31
```
fib(31)

时间花费：0:00:41.323180
```

```
fib_with_cache(31)

时间花费：0:00:00.000282
```

时间相差居然如此之多！

这个装饰器还提供 cache_clear() 用于清理缓存，以及 cache_info() 用于查看缓存信息 
官方还提供了另外一个例子，用于缓存静态网页的内容

```python
@lru_cache(maxsize=32)
def get_pep(num):
    'Retrieve text of a Python Enhancement Proposal'
    resource = 'http://www.python.org/dev/peps/pep-%04d/' % num
    try:
        with urllib.request.urlopen(resource) as s:
            return s.read()
    except urllib.error.HTTPError:
        return 'Not Found'
```


### **@functools.total_ordering**
给定一个类定义一个或多个丰富的比较排序方法，这个类装饰器提供其余的方法。这简化了指定所有可能的丰富比较操作所涉及的工作:

如果你已经定义了 __eq__ 方法，以及 __lt__、__le__、__gt__ 或者 __ge__() 其中之一， 即可自动生成其它比较方法。

上一波源码：
```python
_convert = {
    '__lt__': [('__gt__', _gt_from_lt),
               ('__le__', _le_from_lt),
               ('__ge__', _ge_from_lt)],
    '__le__': [('__ge__', _ge_from_le),
               ('__lt__', _lt_from_le),
               ('__gt__', _gt_from_le)],
    '__gt__': [('__lt__', _lt_from_gt),
               ('__ge__', _ge_from_gt),
               ('__le__', _le_from_gt)],
    '__ge__': [('__le__', _le_from_ge),
               ('__gt__', _gt_from_ge),
               ('__lt__', _lt_from_ge)]
}

def total_ordering(cls):
    """Class decorator that fills in missing ordering methods"""
    # Find user-defined comparisons (not those inherited from object).
    roots = {op for op in _convert if getattr(cls, op, None) is not getattr(object, op, None)}
    if not roots:
        raise ValueError('must define at least one ordering operation: < > <= >=')
    root = max(roots)       # prefer __lt__ to __le__ to __gt__ to __ge__
    for opname, opfunc in _convert[root]:
        if opname not in roots:
            opfunc.__name__ = opname
            setattr(cls, opname, opfunc)
    return cls
```

来一个demo:
```python
from functools import total_ordering


@total_ordering
class Door(object):
    def __init__(self):
        self.value = 0
        self.first_name = ''
        self.last_name = ''

    def __eq__(self, other):
        print('=== my eq===')
        return (self.first_name, self.last_name) == (other.first_name, other.last_name)

    def __gt__(self, other):
        print('=== my total_ordering===')
        return (self.first_name, self.last_name) > (other.first_name, other.last_name)


a = Door()
b = Door()

a.first_name = 'ouyang'
a.last_name = 'guoge'

b.first_name = 'aaaa'
b.last_name = 'bbbb'

print(a == b)
print(a > b)
print(a < b)
print(a <= b)
print(a >= b)
```
结果输出为：
```
=== my eq===
False
=== my total_ordering===
True
=== my total_ordering===
False
=== my total_ordering===
False
=== my total_ordering===
True
```

### **functools.partial(func, *args, **keywords)**
该方法返回一个新的局部对象，当被调用时，它的行为将像函数调用一样，带有位置参数args和关键字参数关键字。如果向调用提供了更多的参数，它们将被附加到args中。如果提供了额外的关键字参数，它们将扩展并覆盖关键字。大致相当于:
```python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        return func(*args, *fargs, **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```
partial()用于部分函数应用程序，该应用程序“冻结”函数参数和/或关键字的一部分，从而生成一个带有简化签名的新对象。例如，可以使用partial()创建一个可调用的函数，其行为类似于int()函数，其中基参数默认为2:
```python
>>> from functools import partial
>>> basetwo = partial(int, base=2)
>>> basetwo.__doc__ = 'Convert base 2 string to an int.'
>>> basetwo('10010')
18
```
上一波源码：
```python
################################################################################
### partial() argument application
################################################################################

# Purely functional, no descriptor behaviour
class partial:
    """New function with partial application of the given arguments
    and keywords.
    """

    __slots__ = "func", "args", "keywords", "__dict__", "__weakref__"

    def __new__(*args, **keywords):
        if not args:
            raise TypeError("descriptor '__new__' of partial needs an argument")
        if len(args) < 2:
            raise TypeError("type 'partial' takes at least one argument")
        cls, func, *args = args
        if not callable(func):
            raise TypeError("the first argument must be callable")
        args = tuple(args)

        if hasattr(func, "func"):
            args = func.args + args
            tmpkw = func.keywords.copy()
            tmpkw.update(keywords)
            keywords = tmpkw
            del tmpkw
            func = func.func

        self = super(partial, cls).__new__(cls)

        self.func = func
        self.args = args
        self.keywords = keywords
        return self

    def __call__(*args, **keywords):
        if not args:
            raise TypeError("descriptor '__call__' of partial needs an argument")
        self, *args = args
        newkeywords = self.keywords.copy()
        newkeywords.update(keywords)
        return self.func(*self.args, *args, **newkeywords)

    @recursive_repr()
    def __repr__(self):
        qualname = type(self).__qualname__
        args = [repr(self.func)]
        args.extend(repr(x) for x in self.args)
        args.extend(f"{k}={v!r}" for (k, v) in self.keywords.items())
        if type(self).__module__ == "functools":
            return f"functools.{qualname}({', '.join(args)})"
        return f"{qualname}({', '.join(args)})"

    def __reduce__(self):
        return type(self), (self.func,), (self.func, self.args,
               self.keywords or None, self.__dict__ or None)

    def __setstate__(self, state):
        if not isinstance(state, tuple):
            raise TypeError("argument to __setstate__ must be a tuple")
        if len(state) != 4:
            raise TypeError(f"expected 4 items in state, got {len(state)}")
        func, args, kwds, namespace = state
        if (not callable(func) or not isinstance(args, tuple) or
           (kwds is not None and not isinstance(kwds, dict)) or
           (namespace is not None and not isinstance(namespace, dict))):
            raise TypeError("invalid partial state")

        args = tuple(args) # just in case it's a subclass
        if kwds is None:
            kwds = {}
        elif type(kwds) is not dict: # XXX does it need to be *exactly* dict?
            kwds = dict(kwds)
        if namespace is None:
            namespace = {}

        self.__dict__ = namespace
        self.func = func
        self.args = args
        self.keywords = kwds

try:
    from _functools import partial
except ImportError:
    pass
```

通过下面这个简单的demo，会更一目了然：

```python
from functools import partial

def add(x, y):
    return x + y

add_y = partial(add, 3)  # add_y 是一个函数
add_y(4)                 # 结果是 7
```

### **functools.partialmethod(func, *args, **keywords) \**
返回一个新的partialmethod描述符，它的行为类似于partial，只是它被设计为作为方法定义使用，而不是直接调用。

func必须是描述符或可调用对象(与普通函数一样，这两种对象都作为描述符处理)。

当func是一个描述符(例如正常的Python函数、classmethod()、staticmethod()、abstractmethod()或partialmethod的另一个实例)时，对__get__
的调用被委托给底层描述符，结果返回一个适当的部分对象。

当func是非描述符可调用时，会动态创建一个合适的绑定方法。当作为方法使用时，这就像一个普通的Python函数:self参数将作为第一个位置参数插入，甚至在提供给partialmethod构造函数的args和关键字之前。

举个栗子：
```
>>> class Cell(object):
...     def __init__(self):
...         self._alive = False
...     @property
...     def alive(self):
...         return self._alive
...     def set_state(self, state):
...         self._alive = bool(state)
...     set_alive = partialmethod(set_state, True)
...     set_dead = partialmethod(set_state, False)
...
>>> c = Cell()
>>> c.alive
False
>>> c.set_alive()
>>> c.alive
True
```

### **functools.reduce(function, iterable[, initializer])**
将两个参数的函数累加到序列的项上，从左到右，使序列减少到一个值。例如,`reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])`计算`((((1 + 2)+(3)+ 4)+ 5)`。左边的参数x是累计值，右边的参数y是序列的更新值。如果存在可选初始化器，则该初始化器将放置在计算中序列项的前面，并在序列为空时充当缺省值。如果初始值设定项未给定，且序列仅包含一个项，则返回第一个项。

相当于：
```python
def reduce(function, iterable, initializer=None):
    it = iter(iterable)
    if initializer is None:
        value = next(it)
    else:
        value = initializer
    for element in it:
        value = function(value, element)
    return value
```

### **@functools.singledispatch**
使用过别的面向对象语言，如 java 等，肯定熟悉各种方法的重载，但是对于 Python 来说是不支持方法的重载的，不过其为我们提供了一个装饰器，能将普通函数变为泛函数（generic function） 

比如你要针对不同类型的数据进行不同的处理，而又不想将它们写到一起，那就可以使用@singledispatch 装饰器了 

举个例子：
```python
import functools

@functools.singledispatch
def typecheck(text):
    pass

@typecheck.register(str)
def _(text):
    print(type(text))
    print("str--")

@typecheck.register(list)
def _(text):
    print(type(text))
    print("list--")

@typecheck.register(int)
def _(text):
    print(type(text))
    print("int--")

if __name__ == '__main__':
    a = [1,2,3,4]
    typecheck(a)
```
看一下结果：
```
<class 'list'>
list--
```
换几个值试一下：
```
a = 1

<class 'int'>
int--
```
```
a = "aaaa"

<class 'str'>
str--
```
singledispatch机制一个显著特征是，可以在系统的任何地方和任何模块中注册专门的函数，如果后来模块中增加了新的类型，可以轻松地添加一个新的专门函数来处理新类型。很像设计模式中的策略模式

### **functools.update_wrapper(wrapper, wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)**

这个函数就是用来更新修饰器函数的，具体更新些什么呢，我们可以直接把它的源码搬过来看一下：
```python
WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',
                       '__annotations__')
WRAPPER_UPDATES = ('__dict__',)
def update_wrapper(wrapper,
                   wrapped,
                   assigned = WRAPPER_ASSIGNMENTS,
                   updated = WRAPPER_UPDATES):
    for attr in assigned:
        try:
            value = getattr(wrapped, attr)
        except AttributeError:
            pass
        else:
            setattr(wrapper, attr, value)
    for attr in updated:
        getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
    wrapper.__wrapped__ = wrapped
    return wrapper
```
大家可以发现，这个函数的作用就是从 被修饰的函数(wrapped) 中取出一些属性值来，赋值给 修饰器函数(wrapper) 。为什么要这么做呢，我们看下面这个例子。

首先我们写个自定义的修饰器，没有任何的功能，仅有文档字符串，如下所示：
```python
def wrapper(f):
    def wrapper_function(*args, **kwargs):
        """这个是修饰函数"""
        return f(*args, **kwargs)
    return wrapper_function
    
@wrapper
def wrapped():
    """这个是被修饰的函数"""
    print('wrapped')

print(wrapped.__doc__)  # 输出`这个是修饰函数`
print(wrapped.__name__)  # 输出`wrapper_function`
```
从上面的例子我们可以看到，我想要获取`wrapped`这个被修饰函数的文档字符串，但是却获取成了`wrapper_function`的文档字符串，`wrapped`函数的名字也变成了`wrapper_function`函数的名字。这是因为给`wrapped`添加上`@wrapper`修饰器相当于执行了一句`wrapped = wrapper(wrapped)`，执行完这条语句之后，`wrapped`函数就变成了`wrapper_function`函数。遇到这种情况该怎么办呢，首先我们可以手动地在`wrapper`函数中更改`wrapper_function`的`__doc__`和`__name__`属性，但聪明的你肯定也想到了，我们可以直接用`update_wrapper`函数来实现这个功能。

我们对上面定义的修饰器稍作修改，添加了一句update_wrapper(wrapper_function, f)。
```python
from functools import update_wrapper

def wrapper(f):
    def wrapper_function(*args, **kwargs):
        """这个是修饰函数"""
        return f(*args, **kwargs)
    update_wrapper(wrapper_function, f)  # <<  添加了这条语句
    return wrapper_function
    
@wrapper
def wrapped():
    """这个是被修饰的函数"""
    print('wrapped')


print(wrapped.__doc__)  # 输出`这个是被修饰的函数`
print(wrapped.__name__)  # 输出`wrapped`

```
此时我们可以发现，`__doc__`和`__name__`属性已经能够按我们预想的那样显示了，除此之外，`update_wrapper`函数也对`__module__`和`__dict__`等属性进行了更改和更新。


### **@functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)**
看一下源码：
```python
WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',
                       '__annotations__')
WRAPPER_UPDATES = ('__dict__',)
def wraps(wrapped,
          assigned = WRAPPER_ASSIGNMENTS,
          updated = WRAPPER_UPDATES):
    return partial(update_wrapper, wrapped=wrapped,
                   assigned=assigned, updated=updated)
```

没错，就是这么的简单，只有这么一句，我们可以看出，wraps函数其实就是一个修饰器版的update_wrapper函数，它的功能和update_wrapper是一模一样的。我们可以修改我们上面的自定义修饰器的例子，做出一个更方便阅读的版本。

```python
from functools import wraps

def wrapper(f):
    @wraps(f)
    def wrapper_function(*args, **kwargs):
        """这个是修饰函数"""
        return f(*args, **kwargs)
    return wrapper_function
    
@wrapper
def wrapped():
    """这个是被修饰的函数
    """
    print('wrapped')

print(wrapped.__doc__)  # 输出`这个是被修饰的函数`
print(wrapped.__name__)  # 输出`wrapped`
```
`wraps`修饰器，其实就是将**被修饰的函数(wrapped)**的一些属性值赋值给**修饰器函数(wrapper)**，最终让属性的显示更符合我们的直觉。