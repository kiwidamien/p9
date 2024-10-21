---
title: "Timing imports"
author: "Damien Martin"
date: "2024-10-14 12:00"
categories: [python, package-management]
description: "Tracing particularly slow imports"
---

# Problem

When importing particular packages, they are importing slowly. You'd like to find why, and if there is a way you can refactor your code / your team's code to improve import speed.

## What slow imports usually indicate

_Too much is being done at import time_

Importing should typically not "do work" (e.g. it should not load datafiles, connect to databases, instantiate drivers, etc). The biggest exception to this is probably setting up `logger`s, because the first caller to `logging.basicConfig` can change what all the other loggers can see. 

Because logging is a [cross-cutting](https://en.wikipedia.org/wiki/Cross-cutting_concern) [concern](https://www.milanjovanovic.tech/blog/balancing-cross-cutting-concerns-in-clean-architecture), loggers are also generally made as singletons at the module level.

The reason that imports can be so slow is if we have a package structure like
```
world/
│
├── africa/
│   ├── __init__.py
│   └── zimbabwe.py
│
├── europe/
│   ├── __init__.py
│   ├── greece.py
│   ├── norway.py
│   └── spain.py
│
└── __init__.py
```
and you write `import world.europe.spain` into Python, then

1. First `world.__init__.py` gets imported (if it hasn't been imported already)
2. Then `world.europe.__init__.py` gets imported (if it hasn't been imported already)
3. Finally, `world.europe.spain.py` gets imported

If you did `import world.europe.greece` after importing `spain`, then only `world.europe.greece.py` would be imported on this second call (each module is only imported once).

But this means if your `__init__.py` files anywhere in the tree "do work", then every subpackage that you import is also going to be slow. This is one of the reasons for recommending that `__init__.py` files are kept relatively sparse.

# Profiling

There is no need to write a custom hook to see what the problematic imports are! Just use `-X importtime` on the command line, as described [here](https://realpython.com/python-import/#profile-imports)

Example:
```
$ python -X importtime -c "import datetime"
import time: self [us] | cumulative | imported package
...
import time:        87 |         87 |   time
import time:       180 |        180 |   math
import time:       234 |        234 |   _datetime
import time:       820 |       1320 | datetime
```