collection模块实现了专门的容器数据类型，为Python的通用内置容器`dict`，`list`，`set`和`tuple`提供了替代方案。接下来，将分别介绍他们。

## **namedtuple()**
包含命名字段的元组工厂方法
命名元组为元组中的每个位置赋予含义，并允许更可读，自文档代码。 
它们可以在使用常规元组的任何地方使用，并且它们添加了按名称而不是位置索引访问字段的功能。

实现：
```python
collections.namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)
```

- 返回一个名为`typename`的新元组子类。 新子类用于创建类似元组的对象，这些对象具有可通过属性查找访问的字段以及可索引和可迭代的字段。 子类的实例还有一个有用的文档字符串（带有`typename`和`field_names`）和一个有用的`__repr __()`方法，它以`name = value`格式列出元组内容。

- `field_names`是一系列字符串，例如`['x'，'y']`。 或者，`field_names`可以是单个字符串，每个字段名由`空格`和`/`或`逗号`分隔，例如`'x y'`或`'x，y'`。

- 除了以下划线开头的名称外，任何有效的Python标识符都可用于字段名。有效标识符由字母，数字和下划线组成，但不以数字或下划线开头，也不能是类，for，return，global，pass或raise等关键字。

- 如果`rename`为`true`，则无效的字段名称将自动替换为位置名称。 例如，`['abc'，'def'，'ghi'，'abc']`被转换为`['abc'，'_1'，'ghi'，'_3']`，消除了关键字`def`和重复的字段名`abc`。

- `defaults`可以是None或可迭代的默认值。 由于具有默认值的字段必须位于没有默认值的任何字段之后，因此默认值将应用于最右侧的参数。 例如，如果字段名是['x'，'y'，'z']并且默认值是（1,2），则x将是必需参数，y将默认为1，z将默认为2。

- 如果定义了`module`，则将命名元组的`__module__`属性设置为该值。

- 命名的元组实例没有每个实例的字典，因此它们是轻量级的，并且不需要比常规元组更多的内存。

举个例子：
```python
>>> # Basic example
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(11, y=22)     # instantiate with positional or keyword arguments
>>> p[0] + p[1]             # indexable like the plain tuple (11, 22)
33
>>> x, y = p                # unpack like a regular tuple
>>> x, y
(11, 22)
>>> p.x + p.y               # fields also accessible by name
33
>>> p                       # readable __repr__ with a name=value style
Point(x=11, y=22)
```

命名元组对于将字段名称分配给csv或sqlite3模块返回的结果元组特别有用：
```python
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)
```

除了从元组继承的方法之外，命名元组还支持三个额外的方法和两个属性。 为防止与字段名称冲突，方法和属性名称以下划线开头。

- **classmethod somenamedtuple._make(iterable)**

	从现有序列或可迭代对象生成新的实例

		>>> t = [11, 22]
		>>> Point._make(t)
		Point(x=11, y=22)

- **somenamedtuple._asdict()**

	返回一个新的OrderedDict，它将字段名称映射到它们对应的值：

		>>> p = Point(x=11, y=22)
		>>> p._asdict()
		OrderedDict([('x', 11), ('y', 22)])

- **somenamedtuple._replace(**kwargs)**
	
	返回namedtuple的新实例，用新值替换特定字段：

		>>> p = Point(x=11, y=22)
		>>> p._replace(x=33)
		Point(x=33, y=22)
		>>>id(p._replace(x=66))
		4572244008
		>>>id(p)
		4569821544

- **somenamedtuple._fields**
	
	列出字段名称的字符串元组。 用于内省和从现有命名元组创建新的命名元组类型。

		>>>p._fields            # view the field names
		('x', 'y')
		>>> Color = namedtuple('Color', 'red green blue')
		>>> Pixel = namedtuple('Pixel', Point._fields + Color._fields)
		>>> Pixel(11, 22, 128, 255, 0)
		Pixel(x=11, y=22, red=128, green=255, blue=0)


- **somenamedtuple._fields_defaults**

	字典将字段名称映射到默认值。

		>>> Account = namedtuple('Account', ['type', 'balance'], defaults=[0])
		>>> Account._fields_defaults
		{'balance': 0}
		>>> Account('premium')
		Account(type='premium', balance=0)

要检索一个对象的字段，使用`getattr()`方法：
```python
>>> getattr(p, 'x')
11
```
要将字典转化为一个命名元组，使用`**x`的方式赋值:
```python
>>> d = {'x': 11, 'y': 22}
>>> Point(**d)
Point(x=11, y=22)
```

由于命名元组是常规Python类，因此很容易使用子类添加或更改功能。 以下是添加计算字段和固定宽度打印格式的方法：
```
>>> class Point(namedtuple('Point', ['x', 'y'])):
...     __slots__ = ()
...     @property
...     def hypot(self):
...         return (self.x ** 2 + self.y ** 2) ** 0.5
...     def __str__(self):
...         return 'Point: x=%6.3f  y=%6.3f  hypot=%6.3f' % (self.x, self.y, self.hypot)

>>> for p in Point(3, 4), Point(14, 5/7):
...     print(p)
Point: x= 3.000  y= 4.000  hypot= 5.000
Point: x=14.000  y= 0.714  hypot=14.018
```
上面显示的子类将__slots__设置为空元组。 这有助于防止创建实例字典，从而降低内存需求。

子类化对于添加新的存储字段没有用。 相反，只需从_fields属性创建一个新的命名元组类型：
```
>>> Point3D = namedtuple('Point3D', Point._fields + ('z',))
```

可以通过直接分配`__doc__`字段来自定义文档字符串：
```
>>> Book = namedtuple('Book', ['id', 'title', 'authors'])
>>> Book.__doc__ += ': Hardcover book in active collection'
>>> Book.id.__doc__ = '13-digit ISBN'
>>> Book.title.__doc__ = 'Title of first printing'
>>> Book.authors.__doc__ = 'List of authors sorted by last name'
```

通过使用`_replace()`对定制的已经有默认值的原型实例进行改造
```
>>> Account = namedtuple('Account', 'owner balance transaction_count')
>>> default_account = Account('<owner name>', 0.0, 0)
>>> johns_account = default_account._replace(owner='John')
>>> janes_account = default_account._replace(owner='Jane')
```

















