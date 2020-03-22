---
title: mypy static type checking in Python
date: '2017-11-01'
categories: ['dev']
tags: ['python']
---

**Static type checking** could be one of the next feature to be included **in the Python standard library**. One of the main reason is that the lack of static typing is sometimes cool — you do not want to take care of it when you write small scripts — but you would be sometimes happy when your code base is growing to ensure that everything is fine without writing a test case for each line of code. So **[mypy](http://mypy-lang.org)** is here for that.

> Mypy is an experimental optional static type checker for Python that aims to combine the benefits of dynamic (or "duck") typing and static typing.[^1]

The second is that one of the login of the one of the main committer is *gvanrossum* [^2]--yes, **Guido van Rossum** himself the "Benevolent Dictator For Life" (BDFL) of Python. The project is developed at Dropbox with a secret plan in mind.

> My secret plan at Dropbox is actually that once we have a large enough fraction of the code base annotated, we can start converting into Python 3 in a semi-automated fashion, in a way that would not be possible without those annotations. We're not there yet, but that's my secret plan. [^3]
What does it look like?

```python
# Performing a specific import
from typing import Dict

d = {} # type: Dict[str, int]

d['foo'] = 1
d['bar'] = 'one'

result = d['foo'] + d['bar']
```

The only changes in the code above is the import and what looks like a standard comment `# type: Dict[str, int]`. It seems a very good practice in this case to simply say in a comment what will be accepted by the `dict d`. For example in order to avoid this kind of error at runtime when you do not expect it.

```
$ python mp.py
Traceback (most recent call last):
File "mp.py", line 9, in <module>
    result = d['foo'] + d['bar']
TypeError: unsupported operand type(s) for +: 'int' and 'str’
```

It’s better and more clear **to detect it before** by performing this simple call.

```
$ mypy mp.py
mp.py:7: error: Incompatible types in assignment (expression has type "str", target has type "int")
```

The mypy package also provides other features--like the declaration of types for function parameters and outputs--that I will not talk about here. One of the best things is that this feature is not constraining. It can be used **only where it makes sense without impacting the rest of the code**. So it **can live with existing part of code without any problem** and be progressively integrated every time is meaningful or every time the code evolve or is reviewed.

> You can freely mix static and dynamic typing within a program, within a module or within an expression. No need to give up dynamic typing — use static typing when it makes sense.[^1]

Maybe I will dive in more details in a next post, but if you are curious about that just give it a try!

`$ pip install mypy`

[^1]: http://mypy-lang.org/
[^2]: https://github.com/python/mypy/graphs/contributors3
[^3]: https://talkpython.fm/episodes/transcript/100/python-past-present-and-future-with-guido-van-rossum

