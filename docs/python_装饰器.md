# 装饰器

​	装饰器本质上是一个python函数，返回值是内部函数的引用。它可以让其他函数在不需要做任何代码变动的前提下增加额外的功能，装饰器的返回值也是一个函数对象。

​	常用的业务场景有: 插入日志，性能测试，事务处理，缓存，权限校验等。

## 普通装饰器

​	最普通的装饰器，实现的功能是:

+ 在函数执行前，记录一行日志
+ 函数执行结束，记录一行日志

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print("我准备好开始执行 {} 函数".format(func.__name__))
        func(*args, **kwargs)
        print("函数执行结束")
    return wrapper
@logger
def add(x, y):
    print(x+y)

>> add(3, 5)
我准备好执行add函数
5
函数执行结束
```

## 带参数的装饰器

​	上面的例子，装饰器是不能接受参数的，其用法只适用于一些简单的场景。只能对被装饰函数执行一些固定的逻辑。

​	创造一个伪场景，可以在装饰器里传递一个参数，指明国籍，并在函数执行前，用自己的国家的语言打招呼：

```python
def say_hello(country):
    def wrapper(func):
        def check(*args, **kwargs):
            if country == "china":
                print("你好")
            elif country == "usa":
                print("hello")
            else:
                return " - "
        	# 真正函数执行的地方
            func(*args, **kwargs)
        return check
    return wrapper

@say_hello("china")
def add(x, y):
    print(x, y, "x+y=", x+y)

>> add(1,2)
你好
1 2 x+y=3 
```

​	这样的实现比较复杂，需要进行两层嵌套。

## 不带参数的类装饰器

​	以上是基于函数实现的装饰器，还有基于类实现的装饰器。当要实现基于类的装饰器必须实现以下两个内置函数：

+ `__init__` ：接受被装饰函数

+ `__call__` ： 实现装饰逻辑 

  简单实现一个添加日志的装饰器

```python
class Logger:
    def __init__(self, func):
        self.func = func
        
    def __call__(self, *args, **kwargs):
        print("执行前置工作")
        self.func(*args, **kwargs)
        print("执行后置工作")
        
@Logger
def work(name):
    print("正在执行 {} 工作".format(name))
    
>> work("绘图")
```













































