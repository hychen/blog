+++
title = "在 Python 用 Data Object 傳遞資料"
publishDate = 2011-06-17
categories = ["觀想"]
draft = false
+++

當資料耦合度很高時候, 我習慣做一個data object class把他放進去, 因為python是dynamic programming language, 所以你可以在create object時, 直接設定object attribute value


## 簡單版本 {#簡單版本}

```python
class Data(object):
    def __init__(self, **kwargs):
        self.__dict__.update(kwargs)

# the result is hychen
print Data(name='hychen').name
```

這個版本則保護了built-in attributes, 避免被污染

```python
class Data(object):
    _excludes = ['__class__',
                 '__delattr__',
                 '__dict__',
                 '__doc__',
                 '__format__',
                 '__getattribute__',
                 '__hash__',
                 '__init__',
                 '__module__',
                 '__new__',
                 '__reduce__',
                 '__reduce_ex__',
                 '__repr__',
                 '__setattr__',
                 '__sizeof__',
                 '__str__',
                 '__subclasshook__',
                 '__weakref__']
    def __init__(self, **kwargs):
        for k,v in kwargs.items():
            if k in self._excludes:
                raise TypeError("{0} is not a valide keyword argument".format(k))
            self.__dict__[k] = v
# the result is hychen
print Data(name='hychen').name
```

當attribute value需要被動態產生時，像是需要被計算, 或是需要執行系統命令去獲得，則可以對getter動手腳, 例如下面這個範例, access dist_info.uname 會得到在shell執行’uname -a’一樣的結果

```python
class Data(object):
    def __init__(self, **kwargs):
        self.__dict__.update(kwargs)

    @property
    def uname(self):
        import commands
        return commands.getoutput('uname -a')

dist_info = Data(id=1)
#the result is Linux xluna 2.6.38-8-generic-pae #42-Ubuntu SMP Mon Apr 11 05:17:09 UTC 2011 i686 i686 i386 GNU/Linux
print dist_info.uname
```

setter, getter也是可以被修改的, 進階運用請在shell 執行 pydoc property 了解


## Metaclass 版本 {#metaclass-版本}

另一種實做方式 - Metaclass。

```python
class DataObjectType(type):
    def __init__(mbs, name, bases, dct):

        super(DataObjectType, mbs).__init__(name, bases, dct)
        # 將建好的class的__init__ function 設為 clsinit
        mbs.__init__ = clsinit
        # 將建好的class的__repr__ function 設為 clsrepr
        mbs.__repr__ = clsrepr

def clsinit(self, **kwargs):
    # 更新self, 也就是建構好的instance的attributes
    self.__dict__.update(kwargs)

def clsrepr(self):
    # 顯示 DataObject(a=1,b=2)這樣的字串
    return "{}({})".format(self.__class__.__name__,
            ','.join([ "{}={}".format(k,v) for k,v in self.__dict__.items()]))

# 將Value class 的 type 改成 DataObjectType, 原本應該是object的type (也就是type)
# 你可以在python interpreter裡打 type(object) 看一下結果 :p
class Value(object):
    __metaclass__ = DataObjectType

# here we go!
v = Value(x=1,y=2,z=3)
print v.x
# 1
```

而如果你想要跟collections.namedtuple 一樣可以動態產生class的話, 像這樣

```python
Value = dataobject('Value', 'x,y,z')
print Value.x
# None
```

其實做方式如下

```python
def dataobject(name, attrsstr):
    for attrname in attrsstr.split(','):
        dct[attrname] = None
    return DataObjectType(name, (), dct)
```

ok, 我們來檢查一下class的行為是不是符合我們的預期.

a: 建立名為Value的class,含有x,y屬性

```python
>>> dataobject('Value', 'x,y')
```

b: Value class 含有y 屬性, 正確

```python
>>> dataobject('Value', 'x,y').y == None
True
```

c: Value class 沒有包含z屬性(因為沒有定義), 正確

```python
>>> dataobject('Value', 'x,y').z == None
Traceback (most recent call last):
  File "", line 1, in
AttributeError: type object 'Value' has no attribute 'z'
```


### 讓 DataObject 的屬性為唯讀 {#讓-dataobject-的屬性為唯讀}

到目前為止我們已經可以動態產生data object class, 也可以動態設定data object 的值. 但卻沒辦法像namedtuple一樣, 強迫設值的動作只能在建立instance時進行. 範例如下

```python
>>> from collections import namedtuple
>>> cls = namedtuple('TestClass', 'x,y')
# 建立instance
>>> obj = cls(x=1,y=2)
# 讀取instance的x 屬性
>>> obj.x
1
# 設定instance 的 x 屬性 (喔喔, 不能設定)
>>> obj.x = 3
Traceback (most recent call last):
  File "", line 1, in
AttributeError: can't set attribute
```

要達成相似的行為非常簡單, 只要對DataObjectType 的 setattr method 動點手腳 :p 先寫一個會檢查instance有沒有readonly的變數, 若有檢查其值, 為真時, 則不允許設值。

```python
def clssetattr(self, k, v):
    try:
        # check readonly attributes
        readonly = self.readonly
    except AttributeError:
        readonly = False
    if readonly:
        raise AttributeError("can't set attribute")
    object.__setattr__(self, k, v)
```

再把 DataObjectType 產生的 class 的 setattr 換掉

```python
class DataObjectType(type):
    def __init__(mbs, name, bases, dct):
        super(DataObjectType, mbs).__init__(name, bases, dct)
        mbs.__init__ = clsinit
        mbs.__repr__ = clsrepr
        mbs.__setattr__ = clssetattr
.....
```

```python
#首先, 沒有設定readonly的狀態下
>>> obj = Value(x=3)
#成功設定x為4
>>> obj.x=4
#再來把Value設為readonly
>>> Value.readonly = True
>>> obj = Value(x=3)
# 沒辦法把x設成4, 成功!
>>> obj.x=4
Traceback (most recent call last):
  File "", line 1, in
  File "metaclass.py", line 41, in clssetattr
    raise AttributeError("can't set attribute")
AttributeError: can't set attribute
```


## 結論 {#結論}

在Refactory一書中, 建議使用data object來減少需要傳遞的參數, 而Python又可以使用property使得getter function 使用方式與讀取attribute 一模一樣, 使得data object在python裡面更powerful.
