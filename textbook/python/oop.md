---
title: ООП в python
menu: textbook-python
---

Язык python полноценно поддерживает объектно-ориентированную разработку. Минимальный класс в python можно создать следующим образом:

```py
class NewGreatType:
    pass

obj = NewGreatType()
type(obj)  # <class '__main__.NewGreatType'>
```

Любой объект в python является объектом какого-либо класса:

```py
type(1)              # <class 'int'>
type(int)            # <class 'type'>
type(NewGreatType)   # <class 'type'>
type(type)           # <class 'type'>
```

Эти примеры показывают, что типы данных сами являются объектами класса `type`. Встроенная функция `dir` позволяет получить все атрибуты (поля и методы) объекта. Выведем для примера атрибуты объекта `False` класса `bool`:

```py
dir(False)
# ['__abs__', '__add__', '__and__', '__bool__', '__ceil__', '__class__', '__delattr__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floor__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getnewargs__', '__gt__', '__hash__', '__index__', '__init__', '__init_subclass__', '__int__', '__invert__', '__le__', '__lshift__', '__lt__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__round__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'as_integer_ratio', 'bit_length', 'conjugate', 'denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']
```

Довольно много для такого простого объекта. Попробуем вызвать метод `__or__`:

```py
False.__or__(False)  # False
False.__or__(True)   # True
```

Вызов этого метода эквивалентен использованию оператора `or`. Мы обнаружили способ перегрузки операторов в python. Она выполняется с помощью определения "магических" методов, некоторые из которых мы рассмотрим ниже.

## Поля и методы

Вспомним класс `LorentzVector`, с которым мы работали в [разделе](../cpp/classes) про классы в C++ и начнем писать его аналог на python:

```py
class LorentzVector:
    """ Релятивистский вектор """

    def __init__(self, t, x):
        """ x: может быть int, float или list """
        self.t = t
        self.x = x

    def r2(self):
        """ Квадрат модуля пространственной компоненты """
        if isinstance(self.x, (int, float)):
            return self.x**2
        return sum(map(lambda a: a**2, self.x))

    def inv(self):
        """ Релятивистский инвариант """
        return self.t**2 - self.r2()
```

Метод `__init__` является конструктором. Объект `self` ссылается на сам объект класса (аналог `this` в C++) и является обязательным первым аргументом всех нестатических методов, включая конструктор. В конструкторе мы определили два поля класса: `self.x` и `self.y`. Также мы определили два метода: `r2` возвращает квадрат пространственной компоненты вектора, и `inv` возвращает релятивистский инвариант, соответствующий вектору.

Все поля и методы класса в python являются публичными. Разделение интерфейса и деталей реализации происходит на уровне соглашения об именах полей и методов: если атрибут не является частью интерфейса, то его имя должно начинаться с двух подчеркиваний, например: `__internal_variable`.

Проверим работу класса `LorentzVector`:

```py
lv1 = LorentzVector(1, 0.5)
lv1.t      # 1
lv1.x      # 0.5
lv1.r2()   # 0.25
lv1.inv()  # 0.75
```

```py
lv2 = LorentzVector(1, [0.3, 0.4, 0.0])
lv2.t      # 1
lv2.x      # [0.3, 0.4, 0.0]
lv2.r2()   # 0.25
lv2.inv()  # 0.75
```

Атрибуты могут определяться не только в конструкторе, но и в любом другом методе класса. Более того, атрибуты можно определять прямо в пользовательском коде:

```py
lv = LorentzVector(1, 0.5)
hasattr(lv, 'mass')  # False
lv.mass = 0.3
hasattr(lv, 'mass')  # True
```

Атрибуты класса (статические атрибуты) определяются сразу после названия класса:

```py
class LorentzVector:
    """ Релятивистский вектор """
    speed_of_light = 2.99792458e10  # см / с
    # ...

LorentzVector.speed_of_light  # 29979245800.0
```

Для задания статического метода необходимо использовать декоратор `staticmethod`:

```py
class LorentzVector:
    # ...
    @staticmethod
    def boost_vector(lv):
        """ Возвращает boost-вектор для данного вектора """
        if isinstance(lv.x, (int, float)):
            return lv.x / lv.t
        return list(map(lambda x: x / lv.t, lv.x))
```

Как и следовало ожидать, статический метод не имеет аргумента `self`. [Декораторы](https://habr.com/ru/post/141411/) — это инструмент python, позволяющий менять поведение функций. Технически — это функция, которая принимает на вход некоторую функцию, и возвращает новую функцию с тем же набором аргументов.

С помощью декоратора `property` можно делать вызов методов, не имеющих аргументов, похожим на обращение к полю класса:

```py
class LorentzVector:
    # ...
    @property
    def r2(self):
        """ Квадрат модуля пространственной компоненты """
        if isinstance(self.x, (int, float)):
            return self.x**2
        return sum(map(lambda a: a**2, self.x))

    @property
    def inv(self):
        """ Релятивистский инвариант """
        return self.t**2 - self.r2()
```

```py
lv1 = LorentzVector(1, 0.5)
lv1.t    # 1
lv1.x    # 0.5
lv1.r2   # 0.25
lv1.inv  # 0.75
```

## Магические методы

Интеграция пользовательских классов в среду языка выполняется посредством определения [специальных методов](https://docs.python.org/3/reference/datamodel.html#special-method-names). Начнём с рассмотрения примера перегрузки арифметического оператора:

```py
class LorentzVector:
    # ...
    def __add__(self, rhs):
        """ Сложение двух векторов """
        if isinstance(self.x, (int, float)):
            x_new = self.x + rhs.x
        else:
            x_new = list(map(lambda a: sum(a), zip(self.x, rhs.x)))
        return LorentzVector(self.t + rhs.t, x_new)
```

Аналогично выполняется перегрузка операторов вычитания (`__sub__`), умножения (`__mul__`), деления (`__div__`), круглых скобок (`__call__`), квадратных скобок (`__getitem__`) и т.д.

Методы `__str__` и `__repr__` отвечают за текстовое представление объекта. Метод `__str__` вызывается, когда объект передается в функцию `print` или в форматированную строку, и служит для "неформального" представления объекта. Метод `__repr__` должен возвращать строку, которая содержит всю информацию о состоянии объекта и по которой объект может быть восстановлен. Если определен только метод `__repr__`, то он будет вызываться в функции `print` вместо метода `__str__`.

Если передать объект `LorentzVector` в функцию `print`, не определяя специальных методов, то мы получим что-то подобное:

```py
lv = LorentzVector(1, 0.5)
print(lv)  # __main__.LorentzVector object at 0x7f50a36dfeb0>
```

Функция `print` вывела тип объекта и адрес, по которому он расположен в памяти.

Определим метод `__repr__`:

```py
class LorentzVector:
    # ...
    def __repr__(relf):
        """ Текстовое представление вектора """
        return f'LorentzVector [{self.t}, {self.x}]'
```

Получаем теперь:

```py
lv = LorentzVector(1, 0.5)
print(lv)  # LorentzVector [1, 0.5]
```

Мы рассмотрели лишь некоторые из доступных специальных методов. Рекомендуем ознакомиться с полным списком в [документации](https://docs.python.org/3/reference/datamodel.html#special-method-names).

## Наследование

Язык python позволяет выполнять наследование классов. Класс-потомок имеет доступ ко всем полям и методам класса-предка. Все классы в python являются наследниками класса `object`. Об класса `object` наш класс `LorentzVector` наследует большинство своих атрибутов:

```py
lv = LorentzVector(1, 0.5)
isinstance(lv, object)  # True
dir(lv)
# ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'inv', 'mass', 'r2', 'sum', 't', 'x']
```

Используем механизм наследования, чтобы реализовать тип релятивистского вектора иначе. Новый тип будет наследником именованного кортежа [`NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple):

```py
from typing import NamedTuple

class FourVector(NamedTuple):  # <- наследование
    """ Релятивистский вектор """

    t: float  # такой синтаксис используется для задания
    r: list   # полей кортежа и аннотации их типов

    @property
    def r2(self):
        """ Квадрат модуля пространственной компоненты """
        return sum(map(lambda a: a**2, self.r))

    @property
    def inv(self):
        """ Релятивистский инвариант """
        return self.t**2 - self.r2()

    def __repr__(relf):
        """ Текстовое представление вектора """
        return f'FourVector [{self.t}, {self.r}]'
```

Новая реализация яснее показывает структуру нашего типа данных. Для создания объекта `FourVector` необходимо задать значения полям `t` и `r`:

```py
fv1 = FourVector(1, [0.3, 0.4, 0.0])
fv2 = FourVector(t=1, r=[0.3, 0.4, 0.0])
fv3 = FourVector(t=1, r=[0.5])
fv4 = FourVector(t=1, r=0.5)

fv3.r2  # 0.25
fv4.r2  # TypeError: 'float' object is not iterable
```

При создании объекта `fv4` мы нарушили соглашение и передали в поле `r` объект `float` вместо `list`. Это привело к ошибке при вызове метода `r2`. При определении класса `FourVector` мы использовали [аннотацию типов](extra#type-ann) (`t: float`).

Больше информации о наследовании в python можно найти в [документации](https://docs.python.org/3/tutorial/classes.html#inheritance).

## Полиморфизм в python

Реализация полиморфизма в python сильно отличается от его реализации в C++. Полиморфизм в C++ реализуется с помощью инструментов наследования и шаблонов. Динамическая типизация python позволяет использовать гораздо более гибкие инструменты полиморфизма. Переменные, аргументы функций и атрибуты классов в python могут в разных контекстах иметь разные типы и даже менять тип со временем. Таким образом, все объекты в python изначально полиморфны.

В такой ситуации возникает вопрос о том как описывать ограничения на допустимые типы объектов. Здесь на помощь приходит принцип утиной типизации (duck typing), дословно состоящий в том, что "если что-то выглядит как утка и крякает как утка, значит это утка". Иными словами, если объект предоставляет необходимый интерфейс, то мы можем с ним работать вне зависимости от его типа. Для поддержки этого подхода в python реализованы инструменты для проверки свойств объектов:

```py
isinstance(obj, (int, float))       # проверяет тип obj
hasattr(obj, 'norm')                # проверяет имеет ли obj атрибут norm
issubclass(FourVector, NamedTuple)  # является ли класс FourVector подклассом NamedTuple
```

Столь гибкая типизация приводит к необходимости качественной документации кода. Хорошим стилем является описание всех контрактов функции или метода в его строке комментария. Значительно улучшает читаемость кода и аннотация типов.

## Резюме

В этом разделе мы выполнили краткий обзор инструментов python, реализующих парадигму объектно-ориентированного программирования. Обсудили создание классов; определение полей и методов; статических полей и методов; определение специальных методов, позволяющих интегрировать тип данных в среду языка; кратко рассмотрели наследование классов и принципы реализации полиморфизма в python.

Концепция ООП в python не является основной, как в C++, однако средства ООП составляют важную часть языка, и их понимание необходимо для грамотной разработки на python, поскольку все типы объектов в python являются классами.

## Источники

* [docs.python.org/3/reference/datamodel](https://docs.python.org/3/reference/datamodel.html)
* [docs.python.org/3/tutorial/classes](https://docs.python.org/3/tutorial/classes.html)
* [docs.python.org/3/library/typing](https://docs.python.org/3/library/typing.html)
* [Python Inheritance](https://www.w3schools.com/python/python_inheritance.asp)
* [Понимаем декораторы в Python'e, шаг за шагом](https://habr.com/ru/post/141411/)
* [Введение в аннотации типов Python](https://habr.com/ru/company/lamoda/blog/432656/)
