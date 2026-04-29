1. UTF-8 BOM是什么
2. js for in for of
# 数据类型

## 整数
## 浮点数
## 字符串
```python
s1 = "hello"  
s2 = 'hello'  
s3 = """多行  
字符串"""
```
### 字符串格式化
f-string:
```python
name = "Alice"
age = 18

print(f"我叫{name}，今年{age}岁")
```
原始字符串：
```python
path = r"C:\Users\Alice\Desktop"
print(path)
```

## 布尔
## 空值
## 常量
## list、tuple
## dict、set

# 高级特性
## 切片
## 列表生成式
## 生成器
## 迭代器
可迭代对象：`list`、`tuple`、`dict`、`set`、`str`、generator
`list`、`dict`、`str`虽然是`Iterable`，却不是`Iterator`
# 函数式编程
## 高阶函数
把函数作为参数传入，这样的函数称为高阶函数，函数式编程就是指这种高度抽象的编程范式。
## 匿名函数
匿名函数`lambda x: x * x`实际上就是：
```python
def f(x):
    return x * x
```
关键字`lambda`表示匿名函数，冒号前面的`x`表示函数参数。
## 装饰器
函数也是一个对象，而且函数对象可以被赋值给变量。
函数对象有一个`__name__`属性（注意：是前后各两个下划线），可以拿到函数的名字。
“装饰器”（Decorator）就是一个返回函数的高阶函数。
```python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```
# 面向对象编程
## 类和实例
```python
class Student(object):
    def __init__(self, name, score):
        self.name = name
        self.score = score

    def get_grade(self):
        if self.score >= 90:
            return 'A'
        elif self.score >= 60:
            return 'B'
        else:
            return 'C'

lisa = Student('Lisa', 99)
bart = Student('Bart', 59)
print(lisa.name, lisa.get_grade())
print(bart.name, bart.get_grade())
```
## 实例属性和类属性
```python
class Student(object):
    name = 'Student'
```
## 访问限制
```python
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))

```
## 使用__slots__
slots用于限制实例的属性
```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```
slots仅对当前类实例有用，对子类不起作用，除非字类也定义slots，这样就会继承。
## 使用@property
@property用于把一个方法变成属性调用
```python
class Student(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```
特别注意：属性的方法名不要和实例变量名重复，否则调用时就会陷入无限循环。
```python
class Student(object):
    # 方法名称和实例变量均为birth:
    @property
    def birth(self):
        return self.birth
```
## 多继承与Mixin
Mixin是一种基于组合思想的继承方法，多继承是实现它的语言机制。
在python中，使用多继承要注意MRO：方法解析顺序，在Python里先使用先继承的方法。