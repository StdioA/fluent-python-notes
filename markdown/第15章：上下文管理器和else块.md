
# 上下文管理和 else 块
> 最终，上下文管理器可能几乎与子程序（subroutine）本身一样重要。目前，我们只了解了上下文管理器的皮毛……Basic 语言有 with 语句，而且很多语言都有。但是，在各种语言中 with 语句的作用不同，而且做的都是简单的事，虽然可以避免不断使 用点号查找属性，但是不会做事前准备和事后清理。不要觉得名字一样，就意味着作用也一样。with 语句是非常了不起的特性。  
> ——Raymond Hettinger, 雄辩的 Python 布道者

本章讨论的特性有：
* with 语句和上下文管理器
* for、while 和 try 语句的 else 语句

## if 语句之外的 else 块
> else 子句的行为如下。
>
> for: 仅当 for 循环运行完毕时（即 for 循环没有被 break 语句中止）才运行 else 块。  
> while: 仅当 while 循环因为条件为**假值**而退出时（即 while 循环没有被 break 语句中止）才运行 else 块。  
> try: 仅当try 块中没有异常抛出时才运行else 块。官方文档（https://docs.python.org/3/ reference/compound_stmts.html）还指出：“else 子句抛出的异常不会由前面的 except 子句处理。”


## try 和 else
为了清晰和准确，`try` 块中应该之抛出预期异常的语句。因此，下面这样写更好：
```python
try：
    dangerous_call()
    # 不要把 after_call() 放在这里
    # 虽然放在这里时的代码运行逻辑是一样的
except OSError:
    log('OSError')
else:
    after_call()
```

## 上下文管理器
上下文管理器可以在 `with` 块中改变程序的上下文，并在块结束后将其还原。


```python
# 上下文管理器与 __enter__ 方法返回对象的区别

class LookingGlass:
    def __enter__(self):
        import sys
        self.original_write = sys.stdout.write
        sys.stdout.write = self.reverse_write
        return 'JABBERWOCKY'

    def reverse_write(self, text):
        self.original_write(text[::-1])
    
    def __exit__(self, exc_type, exc_value, traceback):
        import sys
        sys.stdout.write = self.original_write
        if exc_type is ZeroDivisionError:
            print('Please DO NOT divide by zero!')
            return True

with LookingGlass() as what:    # 这里的 what 是 __enter__ 的返回值
    print('Alice')
    print(what)

print(what)
print('Back to normal')
```

    ecilA
    YKCOWREBBAJ
    JABBERWOCKY
    Back to normal



```python
# contextmanager 的使用

import contextlib

@contextlib.contextmanager
def looking_glass():
    import sys
    original_write = sys.stdout.write
    sys.stdout.write = lambda text: original_write(text[::-1])
    msg = ''
    try:
        # 在当前上下文中抛出的异常，会通过 yield 语句抛出
        yield 'abcdefg'
    except ZeroDivisionError:
        msg = 'Please DO NOT divide by zero!'
    finally:
        # 为了避免上下文内抛出异常导致退出失败
        # 所以退出上下文时一定要使用 finally 语句
        sys.stdout.write = original_write
        if msg:
            print(msg)

# 写完以后感觉这个用法跟 pytest.fixture 好像啊

with looking_glass() as what:
    print('ahhhhh')
    print(what)

print(what)
```

    hhhhha
    gfedcba
    abcdefg


此外，[`contextlib.ExitStack`](https://docs.python.org/3/library/contextlib.html#contextlib.ExitStack) 在某些需要进入未知多个上下文管理器时可以方便管理所有的上下文。具体使用方法可以看文档中的示例。
