产品出了一个奇怪的需求，想通过字符串相似度取匹配城市= =（当然，最后证实通过字符串相似度取判断两个字符串是不是一个城市是不对的！！！）

这里就记录一下我计算字符串(英文字符串)相似度的方法吧～

!!! quote "参考文档"

	- [python_levenshtein 的安装和使用](https://www.jianshu.com/p/06370a33e1ee)
	- [相似度算法之余弦相似度](https://blog.csdn.net/zz_dd_yy/article/details/51926305)

### **Levenshtein**

- Levenshtein.hamming(str1, str2)

	计算汉明距离。要求str1和str2必须长度一致。是描述两个等长字串之间对应位置上不同字符的个数。
	
	用法：
	```shell
	>>> import Levenshtein     
	>>> Levenshtein.hamming('abc', 'cba')
	2
	>>> Levenshtein.hamming('abc', 'def')
	3
	```

- Levenshtein.distance(str1, str2)

	计算编辑距离（也成Levenshtein距离）。是描述由一个字串转化成另一个字串最少的操作次数，在其中的操作包括插入、删除、替换。

	用法：
	```shell
	>>> Levenshtein.distance('abc', 'ab')
	1
	>>> Levenshtein.distance('cxy', 'ab')
	3
	```

- Levenshtein.ratio(str1, str2)
	
	计算莱文斯坦比。计算公式 r = (sum - ldist) / sum, 其中sum是指str1 和 str2 字串的长度总和，ldist是类编辑距离

	<font color="red">注意：</font>这里的类编辑距离不是`Levenshtein.distance(str1, str2)`所说的编辑距离，`Levenshtein.distance(str1, str2)`中三种操作中每个操作+1，而在此处，删除、插入依然+1，但是替换+2
	这样设计的目的：ratio('a', 'c')，sum=2,按2中计算为（2-1）/2 = 0.5,’a','c'没有重合，显然不合算，但是替换操作+2，就可以解决这个问题。

	用法：
	```shell
	>>> Levenshtein.ratio('a,cdsf', 'abcd')		      
	0.6
	```

### **difflib**

我主要用的是`SequenceMatcher`， 因此，本次只介绍`SequenceMatcher`.

`SequenceMatcher`是可以对两个可序列化的对象进行比较的类

官网上的用法是：

```shell
>>> s = SequenceMatcher(lambda x: x == " ",
   ...                     "private Thread currentThread;",
   ...                     "private volatile Thread currentThread;")
   >>> print(round(s.ratio(), 3))
   0.866
   ```

   第一个参数为一个函数，主要用来去掉自己不想算在内的元素；如果没有，可以写`None`
   后面两个参数就是需要比较的两个对象了


### **余弦定理**

[相似度算法之余弦相似度](https://blog.csdn.net/zz_dd_yy/article/details/51926305)

通过阅读上面的文章，我们可以简单总结计算相似度的几个步骤：

1. 列出所有出现的字母，并分别统计两个字符串出现这些字母的次数。这里我是这样写的，利用`from collections import Counter, OrderedDict`

	方法：
	```shell
	>>> from collections import Counter, OrderedDict
	>>> from copy import deepcopy
	>>> a = 'abc'
	>>> b = 'bcde'
	>>> item = set(a) | set(b)
	>>> item
	{'b', 'c', 'e', 'd', 'a'}
	>>> model = OrderedDict().fromkeys(item)
	>>> model
	OrderedDict([('b', None), ('c', None), ('e', None), ('d', None), ('a', None)])
	>>> model1 = deepcopy(model)
	>>> model2 = deepcopy(model)
	>>> model1.update(Counter(a))
	>>> model1
	OrderedDict([('b', 1), ('c', 1), ('e', None), ('d', None), ('a', 1)])
	>>> model2.update(Counter(b))
	>>> model2
	OrderedDict([('b', 1), ('c', 1), ('e', 1), ('d', 1), ('a', None)])
	```
	这样写的原因是，在比较词频的时候，要保证每个字母的顺序是一样的～

2. 利用余弦公式计算相似度

	方法：
	```shell
	>>> import math
	>>> sum = 0	#分子
	>>> q1 = 0	#分母
	>>> q2 = 0	#分母
	>>> for i in item:
		a = model1[i] if type(model1[i]) != type(None) else 0
		b = model2[i] if type(model2[i]) != type(None) else 0
		sum += a * b
		q1 += pow(a, 2)
		q2 += pow(b, 2)
	>>> sum
	2
	>>> q1
	3
	>>> q2
	4
	>>> result = float(sum) / (math.sqrt(q1) * math.sqrt(q2))
	>>> result
	0.5773502691896258
	```
这样就算出相似度啦～

<font color="red">ps：本文说的计算的字符串，全是英文字符串～～</font>









































