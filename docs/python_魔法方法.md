# python的魔法方法

## `__new__`

​	当实例化对象时，最先调用的方法，用于给对象开辟内存空间。

​	虽然python天然就是单例，但是我们也可以使用new方法自己实现单例。

```python
from threading import Lock
class Singleton:
    __instance_lock = Lock() # 加线程锁
    
    def __new__(cls, *args, **kwargs):
        if not hasattr(Singleton, "__instance"):
            with Singleton.__instance_lock:
                if not hasattr(singleton, "__instance"):
                    Singleton.__instance = super().__new__(cls, *args, **kwargs)
        return Singleton.__instance
    
# new方法找内存地址，需要找的是同一个类
# 继承type实现metaclass时，它调用的是call方法，找的是当前类。锁要求一致。
```

## `__init__`

​	对象实例化时对类属性进行初始化的地方，用于接收参数，指定对象的属性。

## `__iter__()`
  如果一个对象内部实现了该方法，那么这个对象就是可迭代对象。

```python
class MyList:
    my_list = [1,2,3]
    def __iter__(self):
        return iter(self.my_list)
```

## `__getitem__()`

​	当未实现 `__iter__`方法时，python解释器尝试使用 `__getitem__`方法获取元素，如果可行，那么这个对象也是一个可迭代对象

```python
class MyList:
    my_list = [1,2,3]
    def __getitem__(self, item):
        return self.my_list[item]
```

​	但是此时使用 isinstance(my_list, Iterator)进行判定的时候，结果为False。因为 isinstance仅检测 `__iter__` 方法。

## `__next__()`

​	在可迭代对象的基础上，用于实现一个迭代器。然后使用 next() 获取对象元素。

```python
class MyList:
    index = 0
    my_list = [1,2,3]
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.index <= len(self.my_list) - 1:
            value = self.my_list[self.index]
            self.index += 1
            return value
        raise StopIteration
```

​	迭代器之后是生成器，为了实现延时计算，缓解大量数据下内存消耗过猛的问题。

## `__doc__`

​	使用doc这个魔法方法，可以查询该模块的说明文档，内容同 help() 方法输出一致。

```python
import json
print(json.__doc__)
print(help("json"))
```

## `__name__`

​	始终是定义时的模块名，即使使用 import .. as .. 取了别名，或是赋值给了另一个变量名。

```python
import json
>> json.__name__

>> my_json = json
>> my_json.__name__

import json as js
>> js.__name__

# 输出内容都是一致的
```

​	python文件既可以直接作为模块被其他python文件调用，也可以直接运行。

我们可以使用 `if __name__ == "__main__"` 来告诉python解释器这是直接运行的。

## `__file__`

​	包含了该模块的文件路径。需要注意的是内建的模块没有这个属性，访问它回抛出异常。

## `__dict__`

​	包含了模块里可用的属性名-属性的字典；也就是可以使用`模块名.属性名`访问的数据。

## `__module__`

​	包含该类的定义的模块名；需要注意：是字符串形式的模块名而不是模块对象。

```python
class People:
    """
       people python class
    """
	
# 命令行或自执行
>> p = people()
>> print(p.__module__)
"main"

# 被其他模块调用时
-test.py
from people import People
p = People()
print(p.__module__)
"People"
```

## `__bases__`

​	直接父类对象的元组；但不包含继承树更上层的其他类，比如父类的父类。 

```python
>> class People: pass
...
>>> class Teenager(People): pass
...
>>> class Student(Teenager): pass
...
>>> Student.__bases__
(<class '__main__.Teenager'>,)

```

## `__enter__`

​	python中有上下文管理器的概念，即会自动帮助我们释放资源。比如打开文件`with open(file_path, 'w') as fp` ; 打开一个异步线程池 `with concurrent.futures.thread.ThreadPoolExecutor(max_workers=5) as pool` 等。我们只需要实现管理器协议，就可以实现一个上下文管理器。

​	以下是 `ThreadPoolExecutor` 的父类中的 管理器实现方法:

```python
class Executor(object):
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.shutdown(wait=True)
        return False
```

## `__exit__`

​	与 `__enter__` 共同组成实现了一个上下文管理器协议，当逻辑体执行结束，会自动帮助我们释放资源。

​	使用`__exit__` 方法可以在执行过程中捕获异常，return True，就告诉解释器该异常我们已经捕获了，不需要再向外抛出。

​	`__exit__` 有三个参数：

+ exc_type: 异常类型

+ exc_val: 异常值

+ exc_tb: 异常的错误栈信息

  当主逻辑没有报错时，这三个值都是 None。

## `__call__`

​	该方法的功能类似于在类中重载 `()` 运算符，是的类实例对象可以像调用普通函数一样，通过 `对象名()` 的形式进行调用。

​	举个最简单的例子：

```python
class Test:
    def __call__(self, name, add):
        print("调用 __call__ 方法:", name, add)
    
>> test = Test()
>> test("name", "work")
调用 __call__ 方法: name work
```

​	通过call方法补足 hasattr 函数的缺陷，即 hasattr 无法判定指定的名称是属性还是方法。

```python
class Test:
    def __init__(self):
        self.name = "hello"
    def say(self):
        print(self.name)
>> test = Test()
>> if hasattr(test, "name"):
>>     print(hasattr(test.name, "__call__"))
False
>> if hasattr(test, "say"):
>>     print(hasattr(test.say, "__call__"))
True
```

​	使用call 方法实现普通装饰器

```python
# 日志打印
class Logger:
    
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print("[INFO]: 函数 {} 正在运行".format(self.func.__name__))
        return self.func(*args, **kwargs)

@Logger
def say(something):
    print("say {}!".format(something))
    
say("hello")
```

​	同样也可以使用 `call` 方法实现一个单例 (基于metaclass 元类)

```python
from threading import Lock
class Singleton(type):
    __instance_lock = Lock()
        
    def __call__(self, *args, **kwargs):
        if not hasattr(self, "__instance"):
            # 锁是关键，必须指定的是相同的锁：即类属性
            with SingletonType__instance_lock:
                if not hasattr(self, "__instance"):
                    self.__instance = super(SingletonType, self).__call__(*args, **kwargs)
        return self.__instance
    
class Foo(metaclass=SingletonType):
    def __init__(self,name):
        self.name = name
obj1 = Foo('name')
obj2 = Foo('name')
```







2 个单例







































