## **deque**
实现：
```python
class collections.deque([iterable[, maxlen]])
```
返回一个新的deque（双端队列）对象，它初始化自`iterable`。 如果未指定iterable，则新的deque为空。

Deques是堆栈和队列的泛化(名称发音为“deck”，是“双端队列”的缩写)。Deques支持从deque的任意一侧线程安全、内存高效的`appends`和`pop`，在任何方向上的性能都大致相同都是O(1)。

尽管`list`对象支持类似的操作，但它们针对快速固定长度操作进行了优化，并导致pop(0)和insert(0，v)操作有O(n)内存移动成本，这些操作改变了底层数据表示的大小和位置。

如果未指定`maxlen`或为None，则deques可能会增长到任意长度。 否则，双端队列限制为指定的最大长度。 一旦有界长度双端队列已满，当添加新项时，则会从对方端丢弃相应数量的项。 有界长度deques提供类似于Unix中的`tail`过滤器的功能。 它们还可用于跟踪仅涉及最近活动的事务和其他数据池。

Deque对象支持以下方法：
```python
append(x)	# 从deque的右边加入x

appendleft(x)	#从deque的左边加入x

clear()		#清除deque中的每一个元素，使其长度为0

copy()		# 创建一个deque的浅拷贝

count(x)	# deque中元素等于x的数量

extend(iterable)	# 从右侧扩展deque

extendleft(iterable)	# 从左侧扩展deque。但是，左边扩展的序列是反转iterable的顺序

index(x[, start[, stop]])	# 返回deque中的x位置（在索引开始时或索引停止之前）。返回第一个匹配的对象，如果没找到，会抛出`ValueError`

insert(i, x)	# 将x插入到deque的位置i。如果插入后导致deque超过`maxlen`，会抛出`IndexError`

pop()	# 从deque的右侧移除并返回一个元素。 如果没有元素，则会抛出IndexError。

popleft()	# 从deque的左侧移除并返回一个元素。 如果没有元素，则会抛出IndexError。

remove(value)	# 删除第一次出现的值。 如果未找到，则会抛出ValueError

reverse()		# 在原位反转deque的元素，然后返回None

rotate(n=1)		# 向右旋转deque n步。 如果n为负数，则向左旋转。
				# 当双端队列不为空时，向右旋转一步相当于d.appendleft(d.pop())，向左旋转一步相当于d.append(d.popleft())。
```

Deque对象还提供一个只读属性：
```python
maxlen	# deque的大小，如果无界，则为None
```

除上述之外，deques支持迭代，pickling, len(d), reverse(d), copy.copy(d), copy.deepcopy(d), 使用`in`运算符进行成员资格测试，以及下标引用，例如d[-1]。 索引访问在两端都是O(1), 但在中间减慢到O(n)。 对于快速随机访问，请改用list。

栗子：
```python
>>> from collections import deque
>>> d = deque('ghi')                 # make a new deque with three items
>>> for elem in d:                   # iterate over the deque's elements
...     print(elem.upper())
G
H
I

>>> d.append('j')                    # add a new entry to the right side
>>> d.appendleft('f')                # add a new entry to the left side
>>> d                                # show the representation of the deque
deque(['f', 'g', 'h', 'i', 'j'])

>>> d.pop()                          # return and remove the rightmost item
'j'
>>> d.popleft()                      # return and remove the leftmost item
'f'
>>> list(d)                          # list the contents of the deque
['g', 'h', 'i']
>>> d[0]                             # peek at leftmost item
'g'
>>> d[-1]                            # peek at rightmost item
'i'

>>> list(reversed(d))                # list the contents of a deque in reverse
['i', 'h', 'g']
>>> 'h' in d                         # search the deque
True
>>> d.extend('jkl')                  # add multiple elements at once
>>> d
deque(['g', 'h', 'i', 'j', 'k', 'l'])
>>> d.rotate(1)                      # right rotation
>>> d
deque(['l', 'g', 'h', 'i', 'j', 'k'])
>>> d.rotate(-1)                     # left rotation
>>> d
deque(['g', 'h', 'i', 'j', 'k', 'l'])

>>> deque(reversed(d))               # make a new deque in reverse order
deque(['l', 'k', 'j', 'i', 'h', 'g'])
>>> d.clear()                        # empty the deque
>>> d.pop()                          # cannot pop from an empty deque
Traceback (most recent call last):
    File "<pyshell#6>", line 1, in -toplevel-
        d.pop()
IndexError: pop from an empty deque

>>> d.extendleft('abc')              # extendleft() reverses the input order
>>> d
deque(['c', 'b', 'a'])
```

接下来，介绍一些deque的使用方法

有界长度deques提供类似于Unix中的`tail`过滤器的功能：
```python
def tail(filename, n=10):
    'Return the last n lines of a file'
    with open(filename) as f:
        return deque(f, n)
```

使用deques的另一种方法是通过向右追加并弹出到左侧来维护一系列最近添加的元素：
```python
from collections import deque
import itertools

def moving_average(iterable, n=3):
    # moving_average([40, 30, 50, 46, 39, 44]) --> 40.0 42.0 45.0 43.0
    # http://en.wikipedia.org/wiki/Moving_average
    it = iter(iterable)
    d = deque(itertools.islice(it, n-1))
    d.appendleft(0)
    s = sum(d)
    for elem in it:
        s += elem - d.popleft()
        d.append(elem)
        yield s / n

if __name__ == '__main__':
    for i in moving_average([40, 30, 50, 46, 39, 44]):
        print(i)

结果：
40.0
42.0
45.0
43.0

```

可以使用存储在双端队列中的输入迭代器来实现循环调度程序。 值从位置零处的活动迭代器产生。 如果该迭代器耗尽，可以使用popleft()删除它; 否则，它可以使用rotate()方法循环回到最后：
```python
def roundrobin(*iterables):
    "roundrobin('ABC', 'D', 'EF') --> A D E B F C"
    iterators = deque(map(iter, iterables))
    while iterators:
        try:
            while True:
                yield next(iterators[0])
                iterators.rotate(-1)
        except StopIteration:
            # Remove an exhausted iterator.
            iterators.popleft()
```

`rotate()`方法提供了一种实现双端切片和删除的方法。 例如，`del d[n]`的纯Python实现依赖于`rotate()`方法来定位要弹出的元素：
```python
def delete_nth(d, n):
    d.rotate(-n)
    d.popleft()
    d.rotate(n)
```
要实现双端切片，请使用类似的方法应用`rotate()`将目标元素置于双端队列的左侧。 使用`popleft()`删除旧条目，使用`extend()`添加新条目，然后反转旋转。 通过该方法的微小变化，可以轻松实现Forth样式堆栈操作，例如dup，drop，swap，over，pick，rot和roll。