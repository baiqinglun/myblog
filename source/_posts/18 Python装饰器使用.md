---
title: Python装饰器
excerpt: python装饰器使用
# sticky: 100
published: true
date: "2024/04/25 16:41:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/%E5%BE%AE%E8%AF%8D%E4%BA%91.jpg
category: 
- [编程,Python]
tags:
- Python
- 装饰器
---

# 1、装饰器定义

函数本身也是对象，能赋值给变量，通过变量名称可以调用函数的功能。此外能通过`__name__`属性拿到函数的名称。

```python
def fun1():
    print("fun1")
    
f1 = fun1
f1.__name__
# 'fun1'
```

装饰器是在不改变原函数条件下，为原函数调用前增加新的功能。装饰器其实是一个高阶函数，高阶函数`hightfunc`接受一个函数对象`func`，在高阶函数内部定义一个新的函数`wrapper`，新的函数`wrapper`在调用`func`函数对象前后执行一定的操作，之后返回`func`，整个告诫高阶函数再返回`wrapper`

```python
def log(func):
    def wrapper():
        print(f"call {func.__name__}")
        return func
    return wrapper

def fun1():
    print("fun1")
    
log(fun1)()
```

写成语法糖的形式

```python
def log(func):
    def wrapper():
        print(f"call {func.__name__}")
        return func
    return wrapper

@log
def fun1():
    print("fun1")
    
fun1()
```

# 2、装饰器有参函数无参

使用装饰器时，传入变量

```python
import logging

def log(level='debug'):
    def decorator(func):
        def wrapper():
            func()
            if level == 'warning':
                logging.warning("{} is called".format(func.__name__))
            else:
                logging.debug("{} is called".format(func.__name__))
            return func
        return wrapper
    return decorator

@log(level="warning") # 添加带参数的装饰器 log()
def func():
    print("func()调用了")

func()

# WARNING:root:func is called
# func()调用了
```

# 3、装饰器和函数均有参

需要使用`*args, **kwargs`

```python
import logging

def log(level='debug'):
    def decorator(func):
        def wrapper(*args, **kwargs):
            ret = func(*args, **kwargs)
            if level == 'warning':
                logging.warning("{} is called".format(func.__name__))
            else:
                logging.debug("{} is called".format(func.__name__))
            return ret
        return wrapper
    return decorator

@log(level="warning") # 添加带参数的装饰器 log()
def func(n,m):
    print(f"from func(), n={n},m={m}")

func(0,2)
```

对于`*args, **kwargs`我了解的还不多，这里不做过多介绍。

- `*args (Non-Keyword Arguments)`相当于列表`['Hello'``, ``'Welcome'``, ``'to'``, ``'GeeksforGeeks']`
- `**kwargs (Keyword Arguments)`相当于字典`{first='Geeks', mid='for', last='Geeks'}`

```python
# *args
list = ["apple","banana","Pomelo"]
def func(*argv):
    for arg in argv:
        print(arg+" ")
func(*list)
# apple 
# banana 
# Pomelo

# **kwargs 
dic = {"1":"apple","2":"banana","3":"Pomelo"}
def func(**kwargs):
    for key,value in kwargs.items():
        print(key+"="+value)
func(**dic)
# 1=apple
# 2=banana
# 3=Pomelo


# *args,**kwargs 
list = ["apple","banana","Pomelo"]
dic = {"1":"apple","2":"banana","3":"Pomelo"}
def func(*argv,**kwargs):
    for arg in argv:
        print(arg+" ")
    for key,value in kwargs.items():
        print(key+"="+value)
func(*list,**dic)
# apple 
# banana 
# Pomelo
# 1=apple
# 2=banana
# 3=Pomelo
```

# 4、类方法装饰器

```python
import time
def decorator(func):
    def wrapper(self, *args, **kwargs):
        start_time = time.time()
        ret = func(self, *args, **kwargs)
        end_time = time.time()
        print("%s.%s() cost %f second!" % (self.__class__,
              func.__name__, end_time - start_time))
        return ret
    return wrapper

class TestDecorator():
    @decorator
    def mysleep(self, n):
        time.sleep(n)

obj = TestDecorator()
obj.mysleep(1)
```

# 5、装饰器类

会调用`__init__`和`__call__`函数

```python
class Tracer():
    def __init__(self, func):
        self.func = func
        self.calls = 0
    def __call__(self, *args, **kwargs):
        self.calls += 1
        print("call %s() %d times" % (self.func.__name__, self.calls))
        return self.func(*args, **kwargs)

@Tracer
def test_tracer(val, name="default"):
    print("func() name:%s, val: %d" % (name, val))

for i in range(2):
    test_tracer(i, name=("name" + str(i)))

>>>
call test_tracer() 1 times
func() name:name0, val: 0
call test_tracer() 2 times
func() name:name1, val: 1
```

# 6、 带参数装饰器类

```python
class Tracer():
    def __init__(self, arg0): # 可支持任意参数
        self.arg0 = arg0
        self.calls = 0
    def __call__(self, func):
        def wrapper(*args, **kwargs):
            self.calls += 1
            print("arg0:%d call %s() %d times" % (self.arg0, func.__name__, self.calls))
            return func(*args, **kwargs)
        return wrapper

@Tracer(arg0=0)
def test_tracer(val, name="default"):
    print("func() name:%s, val: %d" % (name, val))

for i in range(2):
    test_tracer(i, name=("name" + str(i)))

>>>
arg0:0 call test_tracer() 1 times
func() name:name0, val: 0
arg0:0 call test_tracer() 2 times
func() name:name1, val: 1
```

# 7、内置装饰器

## 7.1 staticmethod

类的静态函数装饰器，可通过类名直接调用

```python
class C():
    @staticmethod
    def static_method():
        print("This is a static method!")

C.static_method()     # 类名直接调用

c = C()
c.static_method()     # 类对象调用

>>>
This is a static method!
This is a static method!
```

## 7.2 classmethod

```python
class C():
    @classmethod
    def class_method(cls):
        print("This is ", cls)

class B(C):
  pass

C.class_method()  # 类名直接调用
c = C()
c.class_method()  # 类对象调用

B.class_method()  # 继承类调用

>>>
This is  <class '__main__.C'>
This is  <class '__main__.C'>
This is  <class '__main__.B'>
```

