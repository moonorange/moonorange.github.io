---
title: 'OOP Random Notes'
date: '2021-10-03'
categories: ["Software Design"]
tags: ["OOP", "Python", "Design Pattern", "English Article"]

---

## Intro

Random notes of relating OOP

## Terminology

### Parameter

> A parameter is a named variable passed into a function.
> Parameter variables are used to import arguments into functions.

The difference between parameters and arguments

> Function parameters are the names listed in the function's definition.
> Function arguments are the real values passed to the function.
> Parameters are initialized to the values of the arguments supplied.

[MDN Parameter](https://developer.mozilla.org/en-US/docs/Glossary/Parameter)

### Override

Overriding a method means that a subclass redefines its inherited method(s) when it needs to change or extend the behavior.

### Overload

Overloading a method means implementing multiple methods within the same class that use the same name but a different set of parameters.

### Composition and Inheritance

There is a principle saying `Composition over inheritance` (or composite reuse principle) in object-oriented programming (OOP).

I would like to compare these two things.

#### Inheritance

Implementation Cost: Low

Name Space: Not separated

When using inheritance, the child class shares its whole state with the parent class.
So the state isn't enclosed in a name space.

```python
class Logger:
    def __init__(self, value):
		self._value = value

	def log(self):
		print(f"Parent: value is {self._value}")

class ChildLogger(Logger):
    pass

>>> cld = ChildLogger(5)
>>> print(ChildLogger._value)
>>> 5
>>> cld.log()
>>> Parent: value is 5
```

#### Composition

Implementation Cost: High

Name Space: Separated

Composition, on the other side, keeps the state completely private and makes the delegated object see only what is explicitly shared through message passing.

```python
class Logger:
    def log(self, value):
        print(f"Logger: value is {value}")


class ChildLogger(Logger):
    def __init__(self, value):
        self._value = value
        self.logger = Logger()

    def printf(self):
        self.logger.log(self._value)

>>> cld.printf()
>> Logger: value is 5
```

In the above example, state of ChildLogger can't be seen by Parent Logger class until it's shared explicitly from printf function.

### Summary

Generally, if you can find `to be(is a)` relations of classes, you can implement them using Inheritance, and if you find them `has a` relations, you should use Composition.

## References

[Overload Vs Override in Object Oriented Programming(OOP)](https://medium.com/@atandaoluchiaminat/overload-vs-override-in-object-oriented-programming-oop-a38ca0ccaf40)

[Inheritance and Composition: A Python OOP Guide](https://realpython.com/inheritance-composition-python/)

[合成と委譲](https://python.ms/composition-over-inheritance/)

[Delegation: composition and inheritance in object-oriented programming](https://www.thedigitalcatonline.com/blog/2020/08/17/delegation-composition-and-inheritance-in-object-oriented-programming/)
