## **ChainMap**

`ChainMap`类提供一个快速链接多个映射（字典）的操作。通常情况下，他会比创建字典然后调用`update()`快。

该类可用于模拟嵌套作用域，在模板中很有用。

实现：
```python
class collections.ChainMap(*maps)
```

`ChainMap`类组合多个字典或其他映射到一个可更新的、单一的对象中。如果没有指定`maps`，就会提供一个空字典，以此来保证每个新链中都会有至少一个字典（映射）

底层映射存储在列表中。 该列表是公共的，可以使用`maps`属性访问或更新。

```python
>>> from collections import ChainMap
>>> m1 = {'color': 'red', 'user': 'guest'}
>>> m2 = {'name': 'drfish', 'age': '18'}
>>> chain_map = ChainMap(m1, m2)
>>> chain_map
ChainMap({'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
>>> print(chain_map.get('name'))
drfish
```

支持所有常用的字典方法。除此之外，还支持以下属性：

- **maps**

	返回一个用户可以更新的映射列表。他是按照搜索顺序排序的。

		>>> chain_map.maps
		[{'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'}]
	

- **new_child(m=None)**
	
	返回一个新的ChainMap，这个新ChainMap包含新添加的map，并且这个map在首位。如果没有指定m，那么就会在最前面添加一个空dict。因此`d.new_child()`相当于`ChainMap({}, *d.maps)`。

	需要注意的是，这将产生一个全新的ChainMap，和之前的互不干扰

	读一下源码会更容易理解：

		def new_child(self, m=None):                # like Django's Context.push()
	        '''New ChainMap with a new map followed by all previous maps.
	        If no map is provided, an empty dict is used.
	        '''
	        if m is None:
	            m = {}
	        return self.__class__(m, *self.maps)


	示例：

		>>> m3 = {'data': '1-6'}
		>>> chain_map.new_child(m=m3)
		ChainMap({'data': '1-6'}, {'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
		>>> chain_map
		ChainMap({'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
		>>> id(chain_map.new_child(m=m3))
		4496700080
		>>> id(chain_map)
		4496631176


- **parents**

		@property
	    def parents(self):                          # like Django's Context.pop()
	        'New ChainMap from maps[1:].'
	        return self.__class__(*self.maps[1:])

	返回一个新的ChainMap，这个新ChainMap不包括第一个dict。这个对于跳过第一个map搜索很有用。`d.parents`大致相当于`ChainMap(*d.maps[1:])`

		>>> chain_map.parents
		ChainMap({'name': 'drfish', 'age': '18'})
		>>> id(chain_map.parents)
		4492113680
		>>> id(chain_map)
		4496631176
