定义一个生成器的方法

- 定义一个含有`yield`语句的函数
- 定义一个实现了`__iter___`方法（迭代器协议），方法中含有yield语句的类

通过内置的`next`函数和生成器的`send`方法，可以实现调用者与生成器之间的通信和控制流转移

``` python
def gen():
    print('in gen')
    v = yield
    print('recv {}'.format(v))
    while True:
        print('yield {}'.format(v + 100))
        v = yield v + 100
        print('recv {}'.format(v))

g = gen()
next(g) # 等于调用gen.send(None)
# in gen
v = g.send(1)
# recv 1
# yield 101
print(v)
# 101
v = g.send(2)
# recv 2
# yield 102
print(v)
# 102

```

首次调用`next`函数或生成器的`send`方法时，生成器会执行到第一个`yield`语句处然后返回，返回值为`yield`后跟的表达式的值，注意首次若是调用send方法，则传入的参数必须是`None`，否则会抛出异常`TypeError: can't send non-None value to a just-started generator`。

之后调用`send`方法时，其参数会作为生成器中`yield`语句的值传入，生成器会执行到下一个`yield`表达式处后再次返回。

生成器执行到`return`语句后会抛出`StopIteration`异常，返回值会赋值给异常的`value`属性。

内置的`for`语句和一些标准库设施在迭代生成器时会自动处理该异常，这会导致一些不易察觉的bug

``` python
def gen():
    for i in range(10):
        yield i

g = gen()

for i in g:
    print(i)
# 0123456789

for i in g:
    print(i)

```

第二次迭代不会有任何输出，因为生成器已经执行到了`return`语句，换句话说，生成器只能迭代一次。若需要多次迭代同一个生成器，应先将它转换成一个迭代器（最常见的方法是转换成一个列表然后迭代）