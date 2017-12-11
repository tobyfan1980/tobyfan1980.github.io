---
layout: post
title:  "Singleton not good for Unit test in python"
date:   2017-09-19 11:00:00 -0800
categories:    Programming
tags:    Python
---

There are a lot of sample code online to define a singleton in python, for example.

```
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
```

Singleton make it is simple to get the instance of a class from another module. For example,
```
# my_singleton_class.py
class MySingletonClass(object):
    __metaclass__ = Singleton
    
    def __init__(self, parameters):
        self.parameters = parameters
        
my_singleton_class = MySingletonClass(my_parameters)


# from another file
from MySingletonClass import my_singleton_class
```

However, Singleton makes unit test difficult. Test usually wants to create a instance with test values. But when the singleton class module 
is imported, the instance has been created. Thus, unittest mus make sure to mock with test values before importing the module. This limitation 
adds much work in code maintenance when a group of people are working together on the project. 

Thus, instead of using Singleton, the injection is a better choice in this case. You can create an instance of the 'Singleton' class, and pass
it into other class who wants to use it. This does not break the Singleton rule, while making unit test much easier. 

```
# my_other_class.py
class MyOtherClass(object):
    def __init__(self, singleton_instance):
        self.si = singleton_instance

# test.py
# define test values and put in test_parameters
test_parameters = ...
my_singleton_class = MySingletonClass(test_parameters)

my_other_class = MyOtherClass(my_singleton_class)

my_other_class.run(...)
```

More interesting discussion is at https://testing.googleblog.com/2008/05/tott-using-dependancy-injection-to.html

