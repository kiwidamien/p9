---
title: "Setting breakpoints from the command line"
author: "Damien Martin"
date: "2025-04-08 12:00"
categories: [python, debugging, pdb]
description: "Putting breakpoints explicitly in your code has the downside they might get checked in, and break things in production. This is how to set a breakpoint from the CLI."
---

# The problem

Let's say our program isn't crashing: it is producing output, but that output is wrong. For example, we want to calculate a rolling average over 3 days, and we are getting the wrong output. 

We can put a breakpoint into the code, and use either VS Code or the command line to debug, but this raises the risk of forgetting to remove it. 

We would like a way to temporarily enter a breakpoint _from the invocation_ without touching the source file.

# Solution

You can use the `pdb` module and `b`reakpoint arguments when calling Python.

To give an explicit example:
```shell
python -m pdb -c "b 3" -c c your_script.py
```
where

* `-m pdb`: use the Python debugger module
* `-c "b 3"`: the `-c` flag means "use this command as if it were in a `.pdbrc` file". The `b 3` is breakpoint syntax, explained below. In this case `b 3` means "create a breakpoint on line 3"
* `-c c`: when starting the script, immediately "continue". Without this, `pdb` will effectively be waiting on a breakpoint at the beginning of the script.

The breakpoint syntax is
```
b(reak) [([filename:]lineno | function) [, condition]]
```

# Examples

## Single Script

We have a script for calculating Fibonacci numbers:
```python
# fib.py
def fib(n: int) -> int:
    a, b = 0, 1
    if n < 2:
        return n
    for _ in range(n):   
        a, b = b, a+b  # This is line 6
    return b   # This line is wrong, we should be returning a

print(fib(0))  # 0 -- correct
print(fib(1))  # 1 -- correct
print(fib(2))  # 2 -- should be 1
print(fib(3))  # 3 -- should be 2
```

One way to debug might be 
```shell
python -m pdb -c "b 6" -c c fib.py
```
which drops us here:
```python
Breakpoint 1 at fib.py:6
0
1
> fib.py(6)fib()
-> a, b = b, a+b
(Pdb) 
```

We can also just break at the `fib` function itself when the argument is greater than 1
```shell
python -m pdb -c "b fib,n>1" -c c fib.py
```
This drops us at 
```python 
Breakpoint 1 at fib.py:1
0
1
> fib.py(2)fib()
-> a, b = 0, 1
(Pdb) 
```

We are not going through the debugging steps here, but there are a list of useful commands for [pdb](https://docs.python.org/3.2/library/pdb.html#debugger-commands) on their site. 

**Warning for this specific example:** Annoyingly enough, one of the variables you would like to know about in this example (`n`) is being shadowed by a PDB command: (n)ext! You might want to see what the value of `n` is, but typing this into the debugger makes it go to the next line. To access the value of `n` in the debugger, you have to escape it by using `!n`.

If you are doing a lot of debugging of the same commands, you can also put your breakpoint string into a local `.pdbrc` file in the current directory, which will only get used when calling python with the `-m pdb` string. This doesn't technically fulfill the requirement of not making permanent file changes that could get pushed to prod (but you can use `.gitignore`!), but these arguments won't be used if you don't have a debugger running.


Note that `pylint` and `ruff` both have rules that can check that your code doesn't have a breakpoint as a pre-commit hook as well (it is rule [T100](https://docs.astral.sh/ruff/rules/debugger/) for `ruff`)

## Multiple Scripts

Now suppose we have two files:
```python
# fib.py
def fib_func(n: int) -> int:
    a, b = 0, 1
    if n < 2:
        return n
    for _ in range(n):   
        a, b = b, a+b  # This is line 6
    return b   # This line is wrong, we should be returning a
```

and 

```python
# driver.py
import fib

for n in range(10):
    print(f"fib({n:2d}) = {fib.fib_func(n)}")
```

You might use the debugger this way:
```shell
python -m pdb -c "b fib:3,n>1" -c c driver.py
```

or 
```shell
python -m pdb -c "b fib.fib_func,n>1" -c c driver.py 
```