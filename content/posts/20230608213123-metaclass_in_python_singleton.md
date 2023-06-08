+++
title = "Metaclass In Python - Singleton"
publishDate = 2012-03-13
categories = ["觀想"]
draft = false
+++

Singleton 這東西大家想必不陌生，在 Python 裡實做的方式滿多, 這邊用這個當例子介紹 Metaclass 可以做什麼。

概念上很簡當, 讓Class的建構子不能產生Instance, 然後再提供一個 Classmethod 能夠取得Instance(而且只能有一個), 在Python你可以用id來檢查兩個物件 是不是相同的。原則上Singleton我只會用在不用花腦袋就會覺得這個class建出來的 instance 這個程式執行時只能有一個。

```python
class Singleton(type):
    def __init__(cls,name,bases,dic):
        super(Singleton,cls).__init__(name,bases,dic)
        cls.instance=None

    def __call__(cls, *args, **kwargs):
      # 這裡不raise Exception, 是因為doctest比較好寫
            print "please use get_instance function to get the instance"
            # 你也可讓cls()直接傳回instance, 讓class user不用在意他用的class
      # 是不是Singleton, 他只要注意class的主功能即可
            # return cls.get_instance(*args, *kw)

    def get_instance(cls,*args,**kw):
        if cls.instance is None:
            cls.instance=super(Singleton,cls).__call__(*args,**kw)
        return cls.instance
```


## 用法 {#用法}

把你想變成Singleton class 的 metaclass 設成 Singleton 就可以了， 後悔的話，把那一行註解起來，這個class就不是Singleton。

要注意的地方是這class的Singleton特性是可以被繼承的，但這也是為什麼我喜歡用 這種方式的原因，另一個好處是Class的功能跟Design Pattern的耦合度會比較低。 缺點就是會遇到metaclass衝突的狀況，但也不是不能解決。

```python
class _db(object):
  __metaclass__ = Singleton
```


## Wrap Up {#wrap-up}

```python
#!/usr/bin/env python
# vim: tabstop=4 expandtab shiftwidth=4 softtabstop=4
# -*- encoding=utf8 -*-
#
# Author 2012 Hsin-Yi Chen
class Singleton(type):
    def __init__(cls,name,bases,dic):
        super(Singleton,cls).__init__(name,bases,dic)
        cls.instance=None

    def __call__(cls, *args, **kwargs):
            print "please use get_instance function to get the instance"

    def get_instance(cls,*args,**kw):
        if cls.instance is None:
            cls.instance=super(Singleton,cls).__call__(*args,**kw)
        return cls.instance

class _db(object):
    """
    >>> obj=_db()
    please use get_instance function to get the instance
    >>> obj is None
    True
    >>> obj1 = _db.get_instance()
    connecting to 0
    >>> obj2 = _db.get_instance()
    >>> id(obj1) == id(obj2)
    True
    """
    __metaclass__ = Singleton
    session_max = 0

    def __init__(self):
        print 'connecting to {0}'.format(self.session_max)

class MySQL(_db):
    """
    >>> obj1 = MySQL.get_instance()
    connecting to 1000
    >>> obj2 = MySQL.get_instance()
    >>> id(obj1) == id(obj2)
    True
    """
    session_max = 1000

import doctest
doctest.testmod()
```
