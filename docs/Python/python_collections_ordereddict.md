## **OrderedDict**

OrderedDict和Dict一样，但是它记住了item插入到字典的顺序。当对有序字典进行迭代时，item会按照第一次插入到字典的顺序返回。

实现：
```python
class collections.OrderedDict([items])
```
OrderedDict是dict的子类，支持dict的方法。OrderedDict是一个能记住插入key的顺序的dict。如果有新的item覆盖现有item时，位置不变。但如果是删除该item，再次插入后，将会移到最后。

- **popitem(last=True)**

	该方法返回key-value键值对，并删除该键值对。当`last`为`True`时，是按照LIFO(后进先出)的顺序弹出；如果`last`为`False`，是按照FIFO(先进先出)的顺序弹出

		>>> from collections import OrderedDict
		>>> od = OrderedDict(a=1, b=2)
		>>> od
		OrderedDict([('a', 1), ('b', 2)])
		>>> od.popitem(last=True)
		('b', 2)
		>>> od = OrderedDict(a=1, b=2)
		>>> od
		OrderedDict([('a', 1), ('b', 2)])
		>>> od.popitem(last=False)
		('a', 1)
		>>> od
		OrderedDict([('b', 2)])

- **move_to_end(key, last=True)**

	将现有键移动到有序字典的任一端。 如果`last`为`True`（默认值），则将item移动到右端;如果`last`为`False`，则将item移动到开头。 如果key不存在，则引发`KeyError`：

		>>> d = OrderedDict.fromkeys('abcde')
		>>> d
		OrderedDict([('a', None), ('b', None), ('c', None), ('d', None), ('e', None)])
		>>> d.move_to_end('b')
		>>> d
		OrderedDict([('a', None), ('c', None), ('d', None), ('e', None), ('b', None)])
		>>> d.move_to_end('b', last=False)
		>>> d
		OrderedDict([('b', None), ('a', None), ('c', None), ('d', None), ('e', None)])

除了常用方法外，OrderedDict还支持`reversed()`来反转

```python
>>> reversed(d)
<odict_iterator object at 0x10f813888>
>>> for i in reversed(d):
	print(i)

	
e
d
c
a
b
```

由于OrderedDict会记住插入的顺序，因此它可以结合排序一起使用

```python
>>> d = {'banana': 3, 'apple': 4, 'pear': 1, 'orange': 2}
>>> t = [1, -1, 0]
>>> OrderedDict(sorted(d.items(), key=lambda t: t[0]))
OrderedDict([('apple', 4), ('banana', 3), ('orange', 2), ('pear', 1)])
>>> OrderedDict(sorted(d.items(), key=lambda t: t[1]))
OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])
>>> OrderedDict(sorted(d.items(), key=lambda t: len(t[0])))
OrderedDict([('pear', 1), ('apple', 4), ('banana', 3), ('orange', 2)])
```
当删除排序后的OrderedDict的item时，顺序不会变；但如果是插入新的item到OrderedDict中，item会直接加在最后，并不会根据排序插入。

如果想自定义一个OrderedDict，当出现key值相同的item想插入时，希望可以插入，而不是覆盖，可以这样写：
```python
class LastUpdatedOrderedDict(OrderedDict):
    'Store items in the order the keys were last added'

    def __setitem__(self, key, value):
        if key in self:
            del self[key]
        OrderedDict.__setitem__(self, key, value)
```

我们也可以同时继承`Counter`和`OrderedDict`。通过这个多继承，可以实现一个根据第一次遇到的不同元素的顺序来对元素进行计数。可能这样说起来有些绕嘴也很糊涂，还是上代码吧：
```python
from collections import ChainMap, Counter, OrderedDict

class OrderedCounter(Counter, OrderedDict):
    'Counter that remembers the order elements are first encountered'

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, OrderedDict(self))

    def __reduce__(self):
        return self.__class__, (OrderedDict(self),)

if __name__ == '__main__':
    oc = OrderedCounter('adddddbracadabra')
    print(oc)

OrderedCounter(OrderedDict([('a', 5), ('d', 6), ('b', 2), ('r', 2), ('c', 1)]))
```
原理是这样的：

调用类方法时，Python必须找到要执行的正确方法。有一个定义了搜索类顺序的一个排序，称为方法解析顺序或`mro`。`mro`可以通过`__mro__`查看：
```python
print(OrderedCounter.__mro__)

(<class '__main__.OrderedCounter'>, <class 'collections.Counter'>, <class 'collections.OrderedDict'>, <class 'dict'>, <class 'object'>)
```

当OrderedDict的实例调用`__setitem __()`时，它按顺序搜索类：`OrderedCounter`，`Counter`，`OrderedDict`(在这个类找到了调用的方法)。 所以像`oc['a'] = 0`这样的语句最终会调用`OrderedDict.__setitem__()`。

相反，`__getitem__`不会被`mro`中的任何子类覆盖，因此`count = oc['a']`由`dict.__getitem__()`处理。
```python
oc = OrderedCounter()    
oc['a'] = 1             # this call uses OrderedDict.__setitem__
count = oc['a']         # this call uses dict.__getitem__
```

对于像`oc.update('foobar')`这样的语句，会发生更有趣的调用序列。首先，调用`Counter.update()`。`Counter.update()`的代码使用`self[elem] = count + self_get(elem, 0)`，然后它变成对`OrderedDict.__setitem__()`的调用。 并且该代码调用`dict.__setitem__()`。

<font color="red">注意:</font>如果基类顺序颠倒，则不再有效。因为mro不同，从而导致调用过程中会调用错误的方法。