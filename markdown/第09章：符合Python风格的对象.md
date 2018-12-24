
# 符合 Python 风格的对象
> 绝对不要使用两个前导下划线，这是很烦人的自私行为。  
> ——Ian Bicking  
>   pip, virtualenv 和 Paste 等项目的创建者

得益于 Python 数据模型，自定义类型的行为可以像内置类型那样自然。  
实现如此自然的行为，靠的不是继承，而是**鸭子类型**（duck typing）：我们只需按照预定行为实现对象所需的方法即可。

当然，这些方法不是必须要实现的，我们需要哪些功能，再去实现它就好。

书中的 `Vector` 类完整实现可见[官方 Repo](https://github.com/fluentpython/example-code/blob/master/09-pythonic-obj/vector2d_v3.py).

## 对象表现形式
Python 中，有两种获取对象的字符串表示形式的标准方式：
* `repr()`：以便于开发者理解的方式返回对象的字符串表示形式
* `str()`：以便于用户理解的方式返回对象的字符串表示形式

实现 `__repr__` 和 `__str__` 两个特殊方法，可以分别为 `repr` 和 `str` 提供支持。

## classmethod & staticmethod
`classmethod`: 定义操作**类**，而不是操作**实例**的方法。  
`classmethod` 最常见的用途是定义备选构造方法，比如 `datetime.fromordinal` 和 `datetime.fromtimestamp`.

`staticmethod` 也会改变方法的调用方式，但方法的第一个参数不是特殊的值（`self` 或 `cls`）。  
`staticmethod` 可以把一些静态函数定义在类中而不是模块中，但抛开 `staticmethod`，我们也可以用其它方法来实现相同的功能。


```python
class Demo:
    def omethod(*args):
        # 第一个参数是 Demo 对象
        return args
    @classmethod
    def cmethod(*args):
        # 第一个参数是 Demo 类
        return args
    @staticmethod
    def smethod(*args):
        # 第一个参数不是固定的，由调用者传入
        return args

print(Demo.cmethod(1), Demo.smethod(1))
demo = Demo()
print(demo.cmethod(1), demo.smethod(1), demo.omethod(1))
```

    (<class '__main__.Demo'>, 1) (1,)
    (<class '__main__.Demo'>, 1) (1,) (<__main__.Demo object at 0x05E719B0>, 1)


## 字符串模板化
`__format__` 实现了 `format` 方法的接口，它的参数为 `format` 格式声明符，格式声明符的表示法叫做[格式规范微语言](https://docs.python.org/3/library/string.html#formatspec)。  
`str.format` 的声明符表示法和格式规范微语言类似，称作[格式字符串句法](https://docs.python.org/3/library/string.html#formatstrings)。


```python
# format
num = 15
print(format(num, '2d'), format(num, '.2f'), format(num, 'X'))
```

    15 15.00 F


## 对象散列化
只有可散列的对象，才可以作为 `dict` 的 key，或者被加入 `set`.  
要想创建可散列的类型，只需要实现 `__hash__` 和 `__eq__` 即可。  
有一个要求：如果 `a == b`，那么 `hash(a) == hash(b)`.

文中推荐的 hash 实现方法是使用位运算异或各个分量的散列值：
```python
class Vector2d:
    # 其余部分省略
    def __hash__(self):
        return hash(self.x) ^ hash(self.y)
```

而最新的文章中，推荐把各个分量组成一个 `tuple`，然后对其进行散列：
```python
class Vector2d:
    def __hash__(self):
        return hash((self.x, self.y))
```

## Python 的私有属性和“受保护的”属性
Python 的“双下前导”变量：如果以 `__a` 的形式（双前导下划线，尾部最多有一个下划线）命名一个实例属性，Python 会在 `__dict__` 中将该属性进行“名称改写”为 `_Klass__a`，以防止外部对对象内部属性的直接访问。  
这样可以避免意外访问，但**不能防止故意犯错**。

有一种观点认为不应该使用“名称改写”特性，对于私有方法来说，可以使用前导单下划线 `_x` 来标注，而不应使用双下划线。

此外：在 `from mymod import *` 中，任何使用下划线前缀（无论单双）的变量，都不会被导入。


```python
class Vector2d:
    def __init__(self, x, y):
        self.__x = x
        self.__y = y
    
vector = Vector2d(1, 2)
print(vector.__dict__)
print(vector._Vector2d__x)
```

    {'_Vector2d__x': 1, '_Vector2d__y': 2}
    1


## 使用 __slot__ 类属性节省空间
默认情况下，Python 在各个实例的 `__dict__` 字典中存储实例属性。但 Python 字典使用散列表实现，会消耗大量内存。  
如果需要处理很多个属性**有限且相同**的实例，可以通过定义 `__slots__` 类属性，将实例用元组存储，以节省内存。

```python
class Vector2d:
    __slots__ = ('__x', '__y')
```

`__slots__` 的目的是优化内存占用，而不是防止别人在实例中添加属性。_所以一般也没有什么使用的必要。_

使用 `__slots__` 时需要注意的地方：
* 子类不会继承父类的 `__slots_`
* 实例只能拥有 `__slots__` 中所列出的属性
* 如果不把 `"__weakref__"` 加入 `__slots__`，实例就不能作为弱引用的目标。


```python
# 对象二进制化
from array import array
import math

class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __str__(self):
        return str(tuple(self))

    def __iter__(self):
        yield from (self.x, self.y)

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(array(self.typecode, self)))
    
    def __eq__(self, other):
        return tuple(self) == tuple(other)

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)

vector = Vector2d(1, 2)
v_bytes = bytes(vector)
vector2 = Vector2d.frombytes(v_bytes)
print(v_bytes)
print(vector, vector2)
assert vector == vector2
```

    b'd\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\x00@'
    (1, 2) (1.0, 2.0)

