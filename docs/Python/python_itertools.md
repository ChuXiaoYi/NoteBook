Python的内建模块itertools提供了非常有用的用于操作迭代对象的函数。

### **itertools.count(start=0, step=1)**
创建一个迭代器，生成从n开始的连续整数，如果忽略n，则从0开始计算（注意：此迭代器不支持长整数)

如果超出了sys.maxint，计数器将溢出并继续从-sys.maxint-1开始计算。

当使用浮点数进行计数时，有时可以通过替换乘法代码来实现更高的准确率，例如：`(start + step * i for i in count())`

该方法等价于：
```python
def count(start=0, step=1):
    # count(10) --> 10 11 12 13 14 ...
    # count(2.5, 0.5) -> 2.5 3.0 3.5 ...
    n = start
    while True:
        yield n
        n += step
```

### **itertools.cycle(iterable)**
创造一个迭代器，复制从当前迭代器返回的每一个元素并将其保存到创建的迭代器中。当当前迭代器耗尽时，从创造的迭代器循环返回元素。

简单理解就是，传入一个序列，无限循环下去

大致相当于:
```python
def cycle(iterable):
    # cycle('ABCD') --> A B C D A B C D A B C D ...
    saved = []
    for element in iterable:
        yield element
        saved.append(element)
    while saved:
        for element in saved:
              yield element
```

### **itertools.repeat(object[, times])**

让迭代器一次又一次地返回对象。无限运行，除非指定了`times`参数控制重复次数

大致相当于：
```python
def repeat(object, times=None):
    # repeat(10, 3) --> 10 10 10
    if times is None:
        while True:
            yield object
    else:
        for i in range(times):
            yield object
```

repeat的一个常见用法是提供一个用于map或zip的常数流:
```
>>> list(map(pow, range(10), repeat(2)))
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### **itertools.accumulate(iterable[, func])**

创建一个迭代器，它返回计算的累积和，或其他二进制函数的计算结果（这个二进制函数可以通过func参数指定）。如果指定了func参数，必须保证这个参数对应的函数可以接收两个参数。

大致相当于：
```python
def accumulate(iterable, func=operator.add):
    'Return running totals'
    # accumulate([1,2,3,4,5]) --> 1 3 6 10 15
    # accumulate([1,2,3,4,5], operator.mul) --> 1 2 6 24 120
    it = iter(iterable)
    try:
        total = next(it)
    except StopIteration:
        return
    yield total
    for element in it:
        total = func(total, element)
        yield total
```

如果很难理解，可以这样理解：

    对于默认func来说，结果就是[p0, p0+p1, p0+p1+p2, …]

### **itertools.chain(\*iterables)**

创建一个迭代器，该迭代器从第一个迭代返回元素，直到它被耗尽，然后继续到下一个迭代，直到所有的迭代都被耗尽。用于将连续序列视为单个序列。
大致相当于:
```python
def chain(*iterables):
    # chain('ABC', 'DEF') --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```

### **classmethod chain.from_iterable(iterable)**

改变`chain()`的结构，从一个懒加载的可迭代对象中获取输入值。大致相当于：
```python
def from_iterable(iterables):
    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F
    for it in iterables:
        for element in it:
            yield element

```
### **itertools.compress(data, selectors)**

过滤迭代器中的元素，只返回在selectors中计算为`True`的对应元素。当迭代器或选择器结束后，就停止。
大致相当于：
```python
def compress(data, selectors):
    # compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F
    return (d for d, s in zip(data, selectors) if s)
```

### <span id = "dropwhile">**itertools.dropwhile(predicate, iterable)**</span>

去除predicate为true的元素，当第一次遇到predicate为false的情况时，直接将其与后面的全都一起返回
大致相当于：
```python

def dropwhile(predicate, iterable):
    # dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1
    iterable = iter(iterable)
    for x in iterable:
        if not predicate(x):
            yield x
            break
    for x in iterable:
        yield x
```

### **itertools.filterfalse(predicate, iterable)**

从迭代器中过滤出所有使predicate为`False`的元素，并返回
大致相当于：
```python
def filterfalse(predicate, iterable):
    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8
    if predicate is None:
        predicate = bool
    for x in iterable:
        if not predicate(x):
            yield x
```

### **itertools.groupby(iterable, key=None)**
创建一个从迭代中返回连续键和组的迭代器。 关键是计算每个元素的键值的函数。 如果未指定或为None，则键默认为标识函数并返回元素不变。 通常，迭代需要在相同的键函数上排序。

groupby（）的操作类似于Unix中的uniq过滤器。 每次键函数的值发生变化时，它都会生成一个中断或新组（这就是为什么通常需要使用相同的键函数对数据进行排序）。 这种行为不同于SQL的GROUP BY，它聚合了常见元素而不管它们的输入顺序如何。

返回的组本身是一个迭代器，它与groupby（）共享底层的iterable。 由于源是共享的，因此当groupby（）对象被迭代时，前一个组被迭代的将不再可见。 因此，如果以后需要该数据，则应将其存储为列表：
```python
groups = []
uniquekeys = []
data = sorted(data, key=keyfunc)
for k, g in groupby(data, keyfunc):
    groups.append(list(g))      # Store group iterator as a list
    uniquekeys.append(k)
```
`groupby()`相当于：
```python
class groupby:
    # [k for k, g in groupby('AAAABBBCCDAABBB')] --> A B C D A B
    # [list(g) for k, g in groupby('AAAABBBCCD')] --> AAAA BBB CC D
    def __init__(self, iterable, key=None):
        if key is None:
            key = lambda x: x
        self.keyfunc = key
        self.it = iter(iterable)
        self.tgtkey = self.currkey = self.currvalue = object()
    def __iter__(self):
        return self
    def __next__(self):
        self.id = object()
        while self.currkey == self.tgtkey:
            self.currvalue = next(self.it)    # Exit on StopIteration
            self.currkey = self.keyfunc(self.currvalue)
        self.tgtkey = self.currkey
        return (self.currkey, self._grouper(self.tgtkey, self.id))
    def _grouper(self, tgtkey, id):
        while self.id is id and self.currkey == tgtkey:
            yield self.currvalue
            try:
                self.currvalue = next(self.it)
            except StopIteration:
                return
            self.currkey = self.keyfunc(self.currvalue)
```
举个例子：
```python
from itertools import groupby
qs = [{'date' : 1},{'date' : 2}]
[(name, list(group)) for name, group in itertools.groupby(qs, lambda p:p['date'])]

Out[77]: [(1, [{'date': 1}]), (2, [{'date': 2}])]


>>> from itertools import *
>>> a = ['aa', 'ab', 'abc', 'bcd', 'abcde']
>>> for i, k in groupby(a, len):
...     print i, list(k)
...
2 ['aa', 'ab']
3 ['abc', 'bcd']
5 ['abcde']
```

### **itertools.islice(iterable, start, stop[, step])**和**itertools.islice(iterable, stop)**
创建一个迭代器，该迭代器从iterable返回选中的元素。如果start是非零的，则跳过可迭代的元素，直到到达start为止。之后，元素会连续返回，除非step设置得比step高，这会导致跳过项。如果stop为None，则继续迭代，直到迭代器耗尽为止;否则，它将在指定位置停止。与常规切片不同，islice（）不支持start，stop或step的负值。可以用于从内部结构已被扁平化的数据中提取相关字段(例如，多行报告可能每隔一行列出一个name字段)。
大致相当于:
```python
def islice(iterable, *args):
    # islice('ABCDEFG', 2) --> A B
    # islice('ABCDEFG', 2, 4) --> C D
    # islice('ABCDEFG', 2, None) --> C D E F G
    # islice('ABCDEFG', 0, None, 2) --> A C E G
    s = slice(*args)
    start, stop, step = s.start or 0, s.stop or sys.maxsize, s.step or 1
    it = iter(range(start, stop, step))
    try:
        nexti = next(it)
    except StopIteration:
        # Consume *iterable* up to the *start* position.
        for i, element in zip(range(start), iterable):
            pass
        return
    try:
        for i, element in enumerate(iterable):
            if i == nexti:
                yield element
                nexti = next(it)
    except StopIteration:
        # Consume to *stop*.
        for i, element in zip(range(i + 1, stop), iterable):
            pass
```
如果start为None，那么迭代从0开始。如果step是None，那么step默认为1。

### **itertools.starmap(function, iterable)**
将`iterable`中的每一项，映射到`function`中，并执行`function`
大致相当于：
```python
def starmap(function, iterable):
    # starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000
    for args in iterable:
        yield function(*args)
```

### **itertools.takewhile(predicate, iterable)**
只要使`predicate`为true，就返回。当遇到第一个为`False`的值就停止。与[`dropwhile`](#dropwhile)相反
大致相当于：
```python
def takewhile(predicate, iterable):
    # takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4
    for x in iterable:
        if predicate(x):
            yield x
        else:
            break
```

### **itertools.tee(iterable, n=2)**
从一个可迭代对象中返回n个独立的可迭代对象。

下面的代码会解释`tee`做了什么（虽然实际的解释会更复杂一些，并且只是用了一个单独的FIFO先进先出队列）。
大致相当于：
```python
def tee(iterable, n=2):
    it = iter(iterable)
    deques = [collections.deque() for i in range(n)]
    def gen(mydeque):
        while True:
            if not mydeque:             # when the local deque is empty
                try:
                    newval = next(it)   # fetch a new value and
                except StopIteration:
                    return
                for d in deques:        # load it to all the deques
                    d.append(newval)
            yield mydeque.popleft()
    return tuple(gen(d) for d in deques)
```
实际这样用：
```python
from itertools import tee

print(tee([1,2,3], 3))  # ==>(<itertools._tee object at 0x10cf787c8>, <itertools._tee object at 0x10cf78808>, <itertools._tee object at 0x10cf78848>)

for a in tee([1,2,3], 3):
    for i in a:
        print(i, end=" ")
    print()
#    1 2 3                  
#    1 2 3 
#    1 2 3

for a, b, c in tee([1,2,3], 3):
    print(a, b, c)
#    1 2 3                  
#    1 2 3 
#    1 2 3

```
一旦tee（）进行了拆分，原始的iteable不应该在其他任何地方使用; 否则，迭代可以在没有通知tee对象的情况下进行。

这个itertool可能需要大量的辅助存储（取决于需要存储多少临时数据）。 通常，如果一个迭代器在另一个迭代器启动之前使用大部分或全部数据，则使用list（）而不是tee（）会更快。


### **itertools.zip_longest(\*iterables, fillvalue=None)**

聚合每一个可迭代对象的元素。如果迭代的长度不均匀，则使用`fillvalue`填充缺失值。 迭代继续，直到最长的可迭代用尽。 
大致相当于：
```python
def zip_longest(*args, fillvalue=None):
    # zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D-
    iterators = [iter(it) for it in args]
    num_active = len(iterators)
    if not num_active:
        return
    while True:
        values = []
        for i, it in enumerate(iterators):
            try:
                value = next(it)
            except StopIteration:
                num_active -= 1
                if not num_active:
                    return
                iterators[i] = repeat(fillvalue)
                value = fillvalue
            values.append(value)
        yield tuple(values)
```
如果其中一个iterables可能是无限的，那么`zip_longest()`函数应该包含一些限制调用次数的东西（例如`islice()`或`takewhile()`）。 如果未指定，则fillvalue默认为None。

### **itertools.product(\*iterables, repeat=1)**
对放入的可迭代对象进行笛卡尔积运算。

大致相当于生成器表达式中的嵌套for循环。例如，`product(A, B)`的返回和`((x,y) for x in A for y in B)`一样

要计算iterable与其自身的乘积，请使用可选的repeat关键字参数指定重复次数。 例如，`product(A, repeat=4)`相当于`product(A, A, A, A)`

此函数大致等同于以下代码，但实际实现不会在内存中构建中间结果：
```python
def product(*args, repeat=1):
    # product('ABCD', 'xy') --> Ax Ay Bx By Cx Cy Dx Dy
    # product(range(2), repeat=3) --> 000 001 010 011 100 101 110 111
    pools = [tuple(pool) for pool in args] * repeat
    result = [[]]
    for pool in pools:
        result = [x+[y] for x in result for y in pool]
    for prod in result:
        yield tuple(prod)
```
### <font id = 'permutations'>**itertools.permutations(iterable, r=None)**</font>
对可迭代对象进行组合排列，组合长度为`r`
大致相当于：
```python
def permutations(iterable, r=None):
    # permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC
    # permutations(range(3)) --> 012 021 102 120 201 210
    pool = tuple(iterable)
    n = len(pool)
    r = n if r is None else r
    if r > n:
        return
    indices = list(range(n))
    cycles = list(range(n, n-r, -1))
    yield tuple(pool[i] for i in indices[:r])
    while n:
        for i in reversed(range(r)):
            cycles[i] -= 1
            if cycles[i] == 0:
                indices[i:] = indices[i+1:] + indices[i:i+1]
                cycles[i] = n - i
            else:
                j = cycles[i]
                indices[i], indices[-j] = indices[-j], indices[i]
                yield tuple(pool[i] for i in indices[:r])
                break
        else:
            return
```

### **itertools.combinations(iterable, r)**
和[`permutations`](#permutations)类似。但是不同的是，不会有忽略元素顺序的相同的组合

其相当于：
```python
def combinations(iterable, r):
    # combinations('ABCD', 2) --> AB AC AD BC BD CD
    # combinations(range(4), 3) --> 012 013 023 123
    pool = tuple(iterable)
    n = len(pool)
    if r > n:
        return
    indices = list(range(r))
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != i + n - r:
                break
        else:
            return
        indices[i] += 1
        for j in range(i+1, r):
            indices[j] = indices[j-1] + 1
        yield tuple(pool[i] for i in indices)

```

### **itertools.combinations_with_replacement(iterable, r)**
和[`combinations`](combinations)相似。但不同的是，此方法返回的组合中，会有相同元素
其大致相当于：
```python
def combinations_with_replacement(iterable, r):
    # combinations_with_replacement('ABC', 2) --> AA AB AC BB BC CC
    pool = tuple(iterable)
    n = len(pool)
    if not n and r:
        return
    indices = [0] * r
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != n - 1:
                break
        else:
            return
        indices[i:] = [indices[i] + 1] * (r - i)
        yield tuple(pool[i] for i in indices)
```
















