# 理解Python的装饰器

```python
@decorator
def foo():
	pass
```

实际上等于 `foo=decorator(foo)` 

装饰器的返回值应该是一个`callable`

```python
def decorator(func):
    def warp(*args,**kw):
        print("in decorator")
        return func(*args,*kw)
    return warp
s
@decorator
def foo():
    print("in foo")

foo()

# in decorator
# in foo
```

`foo=decorator(foo)`执行于`foo`的定义完成时（python中函数定义也是一种语句）

利用装饰器可以很方便的实现闭包，闭包可以理解为具有上下文的函数

```python
def print_message(msg):
    def closeure(func):
        def wrap(*args, **kw):
            print(msg)
            return func(*args, **kw)

        return wrap

    return closeure

@print_message("msg1")
def foo1():
    pass

@print_message("msg2")
def foo2():
    pass

foo1() # msg1
foo2() # msg2
```

装饰器可以嵌套，注意执行顺序是装饰器的使用顺序

```python
def print_message(msg):
    def closeure(func):
        def wrap(*args, **kw):
            print(msg)
            return func(*args, **kw)

        return wrap

    return closeure

@print_message("msg1")
@print_message("msg2")
def foo1():
    pass

@print_message("msg2")
@print_message("msg1")
def foo2():
    pass

foo1()

# msg1
# msg2

foo2()

# msg2
# msg1
```

因为装饰器实际上是语法糖，所以有一个问题，装饰器返回的函数并不是原本的函数，原本的函数的一些属性（如文档字符串）可能会丢失。

解决方法是使用`functools.wraps`，它会将原来函数的属性复制到装饰器返回的函数中。

```python
from functools import wraps

def my_decorator(f):
    @wraps(f)
    def wrapper(*args, **kw):
        print("Calling decorated function")
        return f(*args, **kw)

    return wrapper

@my_decorator
def example():
    """Docstring"""
    print("Called example function")

example()  # Calling decorated functionCalled example function
example.__name__  # 'example'
example.__doc__  # 'Docstring'
```
