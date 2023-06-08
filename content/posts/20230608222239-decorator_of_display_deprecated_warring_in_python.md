+++
title = "Decorator of Display Deprecated Warring in Python"
publishDate = 2011-12-18
categories = ["觀想"]
draft = false
+++

用途: 用來警告 function 或 class method 已過時，如果有指定取代的function的話，在runtime時改用取代的function

```python
Python 2.7.2+ (default, Oct  4 2011, 20:03:08)
[GCC 4.6.1] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from vsgui.api import input_text
>>> input_text
<function ask_text at 0x8e0009c>
>>> input_text('input it')
vsgui/api.py:45: DeprecationWarning: Call to deprecated function input_text.; use ask_text instead
  def input_text(*args, **kwargs):
```


## 用法一：  只宣告 Function 已被 depreacted， 不指定取代的 function。 {#用法一-只宣告-function-已被-depreacted-不指定取代的-function}

```python
@depreacted() # <- 這一段表示執行deprecated() 取得wrap function
def old1():   # 當old1()被呼叫時，實際上是先呼叫wrap(old1)以取得old1 function, 再執行old()
  print 1

print 'function is '+old1
old1()
```

執行結果:

```python
function is <function old1 at 0xb7317f44>
recipe-deprecated-warring.py:45: DeprecationWarning: Call to deprecated function old1.
  def old1():
called old1
```

因為有使用 functools.wraps，所以雖然 old1 這個變數實際上已繫結在 wrap function, 但 print 時還是會顯示為 old1 function。


## 用法二： 除了警告外，還會在runtime時改執行取代depreacted function的新functoin {#用法二-除了警告外-還會在runtime時改執行取代depreacted-function的新functoin}

```python
@depreacted(new1) # <- 這一段表示執行deprecated(new1) 取得wrap
def old2():       # 當old1()被呼叫時，實際上是呼叫wrap(old1) 取得new_func 再執行 new_func
  print 1   # 而new_func會再執行new1，並把*args, **kwargs全數pass給new1

print 'function is '+old2
```

注意: 目前還不能對class method 指定 replacement method。


## 源始碼 {#源始碼}

重用了 [Python Wiki - Smart deprecation warnings](https://web.archive.org/web/20120910101845/http://www.artima.com/weblogs/viewpost.jsp?thread=240808), [Active Code Stack - deprecated ](https://web.archive.org/web/20120910101845/http://www.artima.com/weblogs/viewpost.jsp?thread=240845)的部份源碼。

```python
# took codes from the following:
#
#- [Python Wiki - Smart deprecation warnings ](http://wiki.python.org/moin/PythonDecoratorLibrary#Smart_deprecation_warnings_.28with_valid_filenames.2C_line_numbers.2C_etc..29)
# - [Active Code Stack - deprecated ](http://code.activestate.com/recipes/391367-deprecated/)

import os
import warnings
import functools

# enable to show warring
warnings.simplefilter('default')

def deprecated(replacement=None):
    """This is a decorator which can be used to mark functions
    as deprecated. It will result in a warning being emitted
    when the function is used.

    ref:
        - recipe-391367-1 on active stack code
        - recipe-577819-1 on active stack code

    @replacement function replacement function
    """
    def wrapper(old_func):
        wrapped_func = replacement and replacement or old_func
        @functools.wraps(wrapped_func)
        def new_func(*args, **kwargs):
            msg = "Call to deprecated function %(funcname)s." % {
                    'funcname': old_func.__name__}
            if replacement:
                msg += "; use {} instead".format(replacement.__name__)
            warnings.warn_explicit(msg,
                category=DeprecationWarning,
                filename=old_func.func_code.co_filename,
                lineno=old_func.func_code.co_firstlineno + 1
            )
            return wrapped_func(*args, **kwargs)
        return new_func
    return wrapper

def new1():
    print 'called new1'

@deprecated()
def old1():
    print 'called old1'

@deprecated(new1)
def old2():
    print 'called old1'

if __name__ == '__main__':
    print old1
    old1()
    print old2
    old2()
```


## 運作原理 {#運作原理}

-   第一個decorator用來產生第二個 decorator wrapper
-   第二個 decorator wrapper 用來產生新的 function new_func
-   執行新的 function new_func 會先顯示 warrning, 最後再把參數傳給要 wrapped_func，最後再傳回 wrapped_func 執行結果
-   在 runtime 時，因為是閉包的關係, 要執行的 wrapped_func 根據decorator wrapper的replacement參數來決定是用來代換的function, 還是原本的function.

下面是抽象化後的源碼

```python
@deprecated()
def _sum(args):
  pass

def deprecated(replacement=None): # 第一個 decorator
  ....
  def real_decorator(original_func): # 第二個 decorator,
                     # 套用在original function 上, 也就是_sum
      ....
      def func_with_warn(*args, **kwargs): # 產生好新function
          ....
          # 顯示warring字串
          ....
          return original_func(*args, **kwargs) # 回傳_sum的執行解果
      return func_with_warn
  return real_decorator
```


## 參考資料 {#參考資料}

-   [Decorators I: Introduction to Python Decorators](https://web.archive.org/web/20120910101845/http://www.artima.com/weblogs/viewpost.jsp?thread=240808)
-   [Python Decorators II: Decorator Arguments](https://web.archive.org/web/20120910101845/http://www.artima.com/weblogs/viewpost.jsp?thread=240845)
-   [Python 2.7 Manual - warnings — Warning control](https://web.archive.org/web/20120910101845/http://docs.python.org/library/warnings.html)
-   [Python Wiki - Smart deprecation warnings](https://web.archive.org/web/20120910101845/http://wiki.python.org/moin/PythonDecoratorLibrary#Smart_deprecation_warnings_.28with_valid_filenames.2C_line_numbers.2C_etc..29)
-   [Active Code Stack - deprecated](https://web.archive.org/web/20120910101845/http://code.activestate.com/recipes/391367-deprecated/)
