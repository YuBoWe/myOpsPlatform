# Python Decorators (装饰器) 指南

装饰器是 Python 中一种非常强大的工具，允许你在不修改原函数代码的情况下，动态地增加函数的功能。它的本质是一个返回函数的高阶函数。

## 1. 基础装饰器

最简单的装饰器接收一个函数并返回一个新的函数。

```python
def my_decorator(func):
    def wrapper():
        print("执行前...")
        func()
        print("执行后...")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

## 2. 处理函数参数与元数据

### 使用 `*args` 和 `**kwargs`
为了让装饰器能够通用于任何参数个数的函数，需要使用可变参数。

### 使用 `functools.wraps`
装饰器会使原函数的元数据（如 `__name__`, `__doc__`）丢失。使用 `@wraps` 可以保留这些信息。

```python
from functools import wraps

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"调用函数: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@logger
def add(a, b):
    """返回 a + b 的和"""
    return a + b

print(add(1, 2))      # 3
print(add.__name__)   # add (如果不加 @wraps，会显示 wrapper)
```

## 3. 带参数的装饰器

如果装饰器本身需要接收参数，需要再嵌套一层函数。

```python
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    print(f"Hello, {name}!")

greet("Bob")
```

## 4. 类装饰器

可以使用类来实现装饰器，通过实现 `__init__` 和 `__call__`。

```python
class CountCalls:
    def __init__(self, func):
        @wraps(func)
        self.func = func
        self.num_calls = 0

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"第 {self.num_calls} 次调用")
        return self.func(*args, **kwargs)

@CountCalls
def my_func():
    pass

my_func()
my_func()
```

## 5. 常用内置装饰器

*   `@property`: 将一个方法转换成属性调用。
*   `@classmethod`: 将方法定义为类方法（首个参数为 `cls`）。
*   `@staticmethod`: 将方法定义为静态方法。

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @classmethod
    def from_diameter(cls, diameter):
        return cls(diameter / 2)

    @staticmethod
    def is_valid_radius(r):
        return r > 0
```

## 6. 应用场景

*   **日志记录**: 记录函数调用时间、参数等。
*   **权限校验**: 在执行业务逻辑前检查用户权限。
*   **性能测试**: 计算函数运行耗时。
*   **缓存 (Memoization)**: 缓存函数结果，避免重复计算。
