+++
title = "Serialized Python Function Object"
publishDate = 2012-03-28
categories = ["觀想"]
draft = false
+++

利用marshal把python bytecode轉成string, 就可以pickled.

把Function轉成Pickled string

```python
import pickle
import marshal
maintask_bytecode = marshal.dumps(maintask_callback.func_code)
output = {
  'maintask_bytecode': maintask_bytecode,
  'maintask_defaults': maintask_defaults,
  'maintask_args': maintask_args}
print pickle.dumps(output)
```

把Pickled string轉成function

```python
import sys
import types
import marshal
import pickle
## \brief reconstruct function from bytecode
#  \param func_bytecode str marshaled bytecode of func_code
#  \param func_default dict func_defaults
#  \return Function
def reconstruct_function(func_bytecode, func_defaults):
    func_code = marshal.loads(func_bytecode)
    return types.FunctionType(func_code, globals(), func_code.co_name,
                             func_defaults)
string = sys.stdin.read()
received = pickle.loads(string)
callback = reconstruct_function(received['maintask_bytecode'],
      received['maintask_defaults'])
print callback(maintask_args)
```
