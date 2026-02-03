# Python Magic Methods (魔术方法) 指南

魔术方法（Magic Methods）是 Python 中以双下划线开头和结尾的特殊方法，例如 `__init__`。它们允许你重写 Python 的内置行为，使你的类能够像内置类型一样工作。

## 1. 生命周期管理

### `__init__(self, ...)`
最常用的方法，在对象创建后进行初始化。
```python
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")
```

### `__new__(cls, ...)`
在 `__init__` 之前调用，负责创建并返回实例。通常在继承不可变类型（如 `str`, `int`）或实现单例模式时使用。

### `__del__(self)`
析构方法，当对象被垃圾回收时调用。

---

## 2. 对象表示

### `__str__(self)`
由 `str(obj)` 和 `print(obj)` 调用，返回面向用户的友好字符串描述。

### `__repr__(self)`
由 `repr(obj)` 调用，返回面向开发者的准确字符串描述（通常可以用来重新创建该对象）。

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    
    def __str__(self):
        return f"Point({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"
```

---

## 3. 容器与序列

### `__len__(self)`
由 `len(obj)` 调用。

### `__getitem__(self, key)`
实现索引访问 `obj[key]`。

### `__setitem__(self, key, value)`
实现索引赋值 `obj[key] = value`。

```python
class MyList:
    def __init__(self, data):
        self.data = data
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, index):
        return self.data[index]

m = MyList([1, 2, 3])
print(len(m))    # 3
print(m[1])      # 2
```

---

## 4. 上下文管理器

### `__enter__(self)` 和 `__exit__(self, ...)`
用于实现 `with` 语句。

```python
class FileOpener:
    def __init__(self, filename):
        self.filename = filename
    
    def __enter__(self):
        self.file = open(self.filename, 'r')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()

with FileOpener("test.txt") as f:
    content = f.read()
```

---

## 5. 可调用对象

### `__call__(self, ...)`
允许像函数一样调用对象。

```python
class Adder:
    def __init__(self, n):
        self.n = n
    def __call__(self, x):
        return self.n + x

add5 = Adder(5)
print(add5(10))  # 15
```

---

## 6. 属性访问控制

### `__getattr__(self, name)`
当访问不存在的属性时调用。

### `__setattr__(self, name, value)`
在对任何属性赋值时调用。

---

## 常见魔术方法一览表

| 分类 | 方法 | 描述 |
| :--- | :--- | :--- |
| **比较** | `__lt__`, `__le__`, `__eq__`, `__ne__`, `__gt__`, `__ge__` | <, <=, ==, !=, >, >= |
| **算术** | `__add__`, `__sub__`, `__mul__`, `__truediv__` | +, -, *, / |
| **类型转换** | `__int__`, `__float__`, `__bool__` | 转换为对应类型 |
