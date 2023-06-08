+++
title = "How-To: Change string.template’s Delimiter in Runtime"
publishDate = 2012-02-13
categories = ["觀想"]
draft = false
+++

這篇文章提到Template因為用到了Metaclass, 因此改Class或是Instance的delimiter是沒作用的, 因為Template的pattern atrtibute 在 class 建構時, 就已經決定好, 於是你只能建立一個class 來繼承他, 把Template.delimiter Overwrite掉, 這樣 Template().delimter 才是你要的, 可我實在懶得為了改delimiter就去寫一個自定義class, 原文的方式我也覺得有點麻煩, 因為要組出完整的pattern

基本上這很好解, 就 runtime 時產生一個自定義 class 就好了 :p

```python
import string
def create_tpl(content, **tpl_config):
    if tpl_config:
        tplcls = type('CustomTemplate', (string.Template,), tpl_config)
    else:
        tplcls = string.Template
        return tplcls(content)

tpl = create_tpl('#aa', delimiter='#')
print tpl.substitute(aa=1)
```
