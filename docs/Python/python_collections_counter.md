## **Counter**

实现：
```python
class collections.Counter([iterable-or-mapping])
```

源码中，简单介绍了一些用法：

```python
>>> c = Counter('abcdeabcdabcaba')  # count elements from a string

>>> c.most_common(3)                # three most common elements
[('a', 5), ('b', 4), ('c', 3)]
>>> sorted(c)                       # list all unique elements
['a', 'b', 'c', 'd', 'e']
>>> ''.join(sorted(c.elements()))   # list elements with repetitions
'aaaaabbbbcccdde'
>>> sum(c.values())                 # total of all counts
15

>>> c['a']                          # count of letter 'a'
5
>>> for elem in 'shazam':           # update counts from an iterable
...     c[elem] += 1                # by adding 1 to each element's count
>>> c['a']                          # now there are seven 'a'
7
>>> del c['b']                      # remove all 'b'
>>> c['b']                          # now there are zero 'b'
0

>>> d = Counter('simsalabim')       # make another counter
>>> c.update(d)                     # add in the second counter
>>> c['a']                          # now there are nine 'a'
9

>>> c.clear()                       # empty the counter
>>> c
Counter()

Note:  If a count is set to zero or reduced to zero, it will remain
in the counter until the entry is deleted or the counter is cleared:

>>> c = Counter('aaabbc')
>>> c['b'] -= 2                     # reduce the count of 'b' by two
>>> c.most_common()                 # 'b' is still in, but its count is zero
[('a', 3), ('c', 1), ('b', 0)]

```

`Counter`是dict的子类，可以用来计算可哈希对象的数量。它是一个无序的集合，并且元素作为dict的key，数量作为dict的value。数量可以是任意整数值，包括0和负数。

```python
>>> c = Counter()                           # a new, empty counter
>>> Counter("adfadf")						# a new counter from an iterables
Counter({'a': 2, 'd': 2, 'f': 2})			
>>> Counter({'red': 4, 'blue': 2})			# a new counter from a mapping
Counter({'red': 4, 'blue': 2})
>>> Counter(cats=4, dogs=8)					# a new counter from keyword args
Counter({'dogs': 8, 'cats': 4})
```

对于那些不存在的元素，如果想要获取它，Counter会返回0，而不会引发`KeyError`：

```python
>>> c = Counter(['eggs', 'ham'])
>>> c['bacon']                              # count of a missing element is zero
0
```
从源码中可以看出来为什么不引发`KeyError`:

```python
def __missing__(self, key):
    'The count of elements not in the Counter is zero.'
    # Needed so that self[missing_item] does not raise KeyError
    return 0
```

如果count设置为零或减少为零，它将保留在counter中，直到删除该条目或清除计数器：

```python
>>> c['sausage'] = 0                        # counter entry with a zero count
>>> del c['sausage']  
```

由于Counter是dict的子类，因此他具备dict的方法。除此之外，它还具备以下方法：

- **elements()**

	返回每一个元素，元素会根据个数重复count次，并且是以任意顺序返回的。如果元素的个数小于1(包括负值)，那么就会被忽略不返回

		>>> c = Counter(a=4, b=2, c=0, d=-2)
		>>> sorted(c.elements())
		['a', 'a', 'a', 'a', 'b', 'b']

- **most_common([n])**

	返回n个最常见元素及其计数的列表，从最常见到最少。 如果省略n或None，则most_common（）返回计数器中的所有元素。 具有相同计数的元素是任意排序的：

		>>> Counter('abracadabra').most_common(3)  
		[('a', 5), ('r', 2), ('b', 2)]

- **subtract([iterable-or-mapping])**

	根据迭代器或映射中对当前元素进行加减操作。和`dict.update()`类似，但是注意，是操作，而不是替换。输入和输出可以为0或负数

		>>> c = Counter(a=4, b=2, c=0, d=-2)
		>>> d = Counter(a=1, b=2, c=3, d=4)
		>>> c.subtract(d)
		>>> c
		Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6}

通常情况下，Counter对象和字典具有相同的方法。但是以下两个方法会有所不同：

- **fromkeys(iterable)**

	Counter类没有实现这个方法


- **update([iterable-or-mapping])**

	和`dict.update()`相似，但是是进行加减操作，而不是替换。

	从源码看，更容易理解一些：

		    def update(*args, **kwds):
        '''Like dict.update() but add counts instead of replacing them.

        Source can be an iterable, a dictionary, or another Counter instance.

        >>> c = Counter('which')
        >>> c.update('witch')           # add elements from another iterable
        >>> d = Counter('watch')
        >>> c.update(d)                 # add elements from another counter
        >>> c['h']                      # four 'h' in which, witch, and watch
        4

        '''
        # The regular dict.update() operation makes no sense here because the
        # replace behavior results in the some of original untouched counts
        # being mixed-in with all of the other counts for a mismash that
        # doesn't have a straight-forward interpretation in most counting
        # contexts.  Instead, we implement straight-addition.  Both the inputs
        # and outputs are allowed to contain zero and negative counts.

        if not args:
            raise TypeError("descriptor 'update' of 'Counter' object "
                            "needs an argument")
        self, *args = args
        if len(args) > 1:
            raise TypeError('expected at most 1 arguments, got %d' % len(args))
        iterable = args[0] if args else None
        if iterable is not None:
            if isinstance(iterable, _collections_abc.Mapping):
                if self:
                    self_get = self.get
                    for elem, count in iterable.items():
                        self[elem] = count + self_get(elem, 0)
                else:
                    super(Counter, self).update(iterable) # fast path when counter is empty
            else:
                _count_elements(self, iterable)
        if kwds:
            self.update(kwds)

官网给出一些常见操作：
```python
sum(c.values())                 # total of all counts
c.clear()                       # reset all counts
list(c)                         # list unique elements
set(c)                          # convert to a set
dict(c)                         # convert to a regular dictionary
c.items()                       # convert to a list of (elem, cnt) pairs
Counter(dict(list_of_pairs))    # convert from a list of (elem, cnt) pairs
c.most_common()[:-n-1:-1]       # n least common elements
+c                              # remove zero and negative counts
```

提供了几个数学运算来组合Counter对象以生成多个集合（计数大于零的计数器）。 加法和减法通过添加或减去相应元素的计数来组合计数器。 &和 | 返回相应计数的最小值和最大值。 每个操作都可以接受带有符号计数的输入，但输出将排除计数为零或更少的结果。

```python
>>> c = Counter(a=3, b=1)
>>> d = Counter(a=1, b=2)
>>> c + d                       # add two counters together:  c[x] + d[x]
Counter({'a': 4, 'b': 3})
>>> c - d                       # subtract (keeping only positive counts)
Counter({'a': 2})
>>> c & d                       # intersection:  min(c[x], d[x]) 
Counter({'a': 1, 'b': 1})
>>> c | d                       # union:  max(c[x], d[x])
Counter({'a': 3, 'b': 2})
```

一元加法和减法是用于添加空计数器或从空计数器中减去的快捷方式。
```python
>>> c = Counter(a=2, b=-4)
>>> +c
Counter({'a': 2})
>>> -c
Counter({'b': 4})
```

<font color="red">注意</font> : Counter主要用于处理正整数的计数。但是也不要忘记考虑其他类型或负值的情况。Counter类继承自dict，对于key和value是没有限制的。value除了数字也可以存储其他。