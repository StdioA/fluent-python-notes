
# Python 数据类型
> Guido 对语言设计美学的深入理解让人震惊。我认识不少很不错的编程语言设计者，他们设计出来的东西确实很精彩，但是从来都不会有用户。Guido 知道如何在理论上做出一定妥协，设计出来的语言让使用者觉得如沐春风，这真是不可多得。  
> ——Jim Hugunin  
>   Jython 的作者，AspectJ 的作者之一，.NET DLR 架构师

Python 最好的品质之一是**一致性**：你可以轻松理解 Python 语言，并通过 Python 的语言特性在类上定义**规范的接口**，来支持 Python 的核心语言特性，从而写出具有“Python 风格”的对象。  
Python 解释器在碰到特殊的句法时，会使用特殊方法（我们称之为魔术方法）去激活一些基本的对象操作。如 `my_c[key]` 语句执行时，就会调用 `my_c.__getitem__` 函数。这些特殊方法名能让你自己的对象实现和支持一下的语言构架，并与之交互：
* 迭代
* 集合类
* 属性访问
* 运算符重载
* 函数和方法的调用
* 对象的创建和销毁
* 字符串表示形式和格式化
* 管理上下文（即 `with` 块）


```python
# 通过实现魔术方法，来让内置函数支持你的自定义对象
# https://github.com/fluentpython/example-code/blob/master/01-data-model/frenchdeck.py
import collections
import random

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

deck = FrenchDeck()
# 实现 __length__ 以支持 len
print(len(deck))
# 实现 __getitem__ 以支持下标操作
print(deck[1])
print(deck[5::13])
# 有了这些操作，我们就可以直接对这些对象使用 Python 的自带函数了
print(random.choice(deck))
```

    52
    Card(rank='3', suit='spades') [Card(rank='7', suit='spades'), Card(rank='7', suit='diamonds'), Card(rank='7', suit='clubs'), Card(rank='7', suit='hearts')]
    Card(rank='6', suit='diamonds')


Python 支持的所有魔术方法，可以参见 Python 文档 [Data Model](https://docs.python.org/3/reference/datamodel.html) 部分。

比较重要的一点：不要把 `len`，`str` 等看成一个 Python 普通方法：由于这些操作的频繁程度非常高，所以 Python 对这些方法做了特殊的实现：它可以让 Python 的内置数据结构走后门以提高效率；但对于自定义的数据结构，又可以在对象上使用通用的接口来完成相应工作。但在代码编写者看来，`len(deck)` 和 `len([1,2,3])` 两个实现可能差之千里的操作，在 Python 语法层面上是高度一致的。
