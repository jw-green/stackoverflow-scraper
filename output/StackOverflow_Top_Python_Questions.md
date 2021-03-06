# StackOverflow Top Python Questions


## [What does the yield keyword do?](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do)

**8064 Votes**, Alex. S.

To understand what `yield` does, you must understand what generators are. And before generators come iterables.

### Iterables

When you create a list, you can read its items one by one. Reading its items one by one is called iteration:

```python
>>> mylist = [1, 2, 3]
>>> for i in mylist:
...    print(i)
1
2
3
```

`mylist` is an iterable. When you use a list comprehension, you create a list, and so an iterable:

```python
>>> mylist = [x*x for x in range(3)]
>>> for i in mylist:
...    print(i)
0
1
4
```

Everything you can use "`for... in...`" on is an iterable; `lists`, `strings`, files...
These iterables are handy because you can read them as much as you wish, but you store all the values in memory and this is not always what you want when you have a lot of values.

### Generators

Generators are iterators, a kind of iterable you can only iterate over once. Generators do not store all the values in memory, they generate the values on the fly:

```python
>>> mygenerator = (x*x for x in range(3))
>>> for i in mygenerator:
...    print(i)
0
1
4
```

It is just the same except you used `()` instead of `[]`. BUT, you cannot perform `for i in mygenerator` a second time since generators can only be used once: they calculate 0, then forget about it and calculate 1, and end calculating 4, one by one.

### Yield

`yield` is a keyword that is used like `return`, except the function will return a generator.

```python
>>> def createGenerator():
...    mylist = range(3)
...    for i in mylist:
...        yield i*i
...
>>> mygenerator = createGenerator() # create a generator
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```

Here it's a useless example, but it's handy when you know your function will return a huge set of values that you will only need to read once.
To master `yield`, you must understand that when you call the function, the code you have written in the function body does not run. The function only returns the generator object, this is a bit tricky :-)
Then, your code will be run each time the `for` uses the generator.
Now the hard part:
The first time the `for` calls the generator object created from your function, it will run the code in your function from the beginning until it hits `yield`, then it'll return the first value of the loop. Then, each other call will run the loop you have written in the function one more time, and return the next value, until there is no value to return.
The generator is considered empty once the function runs, but does not hit `yield` anymore. It can be because the loop had come to an end, or because you do not satisfy an `"if/else"` anymore.


### Your code explained

Generator:

```python
# Here you create the method of the node object that will return the generator
def _get_child_candidates(self, distance, min_dist, max_dist):

    # Here is the code that will be called each time you use the generator object:

    # If there is still a child of the node object on its left
    # AND if distance is ok, return the next child
    if self._leftchild and distance - max_dist < self._median:
        yield self._leftchild

    # If there is still a child of the node object on its right
    # AND if distance is ok, return the next child
    if self._rightchild and distance + max_dist >= self._median:
        yield self._rightchild

    # If the function arrives here, the generator will be considered empty
    # there is no more than two values: the left and the right children
```

Caller:

```python
# Create an empty list and a list with the current object reference
result, candidates = list(), [self]

# Loop on candidates (they contain only one element at the beginning)
while candidates:

    # Get the last candidate and remove it from the list
    node = candidates.pop()

    # Get the distance between obj and the candidate
    distance = node._get_dist(obj)

    # If distance is ok, then you can fill the result
    if distance <= max_dist and distance >= min_dist:
        result.extend(node._values)

    # Add the children of the candidate in the candidates list
    # so the loop will keep running until it will have looked
    # at all the children of the children of the children, etc. of the candidate
    candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))

return result
```

This code contains several smart parts:

The loop iterates on a list, but the list expands while the loop is being iterated :-) It's a concise way to go through all these nested data even if it's a bit dangerous since you can end up with an infinite loop. In this case, `candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))` exhausts all the values of the generator, but `while` keeps creating new generator objects which will produce different values from the previous ones since it's not applied on the same node.
The `extend()` method is a list object method that expects an iterable and adds its values to the list.

Usually we pass a list to it:

```python
>>> a = [1, 2]
>>> b = [3, 4]
>>> a.extend(b)
>>> print(a)
[1, 2, 3, 4]
```

But in your code it gets a generator, which is good because:

You don't need to read the values twice.
You may have a lot of children and you don't want them all stored in memory.

And it works because Python does not care if the argument of a method is a list or not. Python expects iterables so it will work with strings, lists, tuples and generators! This is called duck typing and is one of the reason why Python is so cool. But this is another story, for another question...
You can stop here, or read a little bit to see an advanced use of a generator:

### Controlling a generator exhaustion


```python
>>> class Bank(): # Let's create a bank, building ATMs
...    crisis = False
...    def create_atm(self):
...        while not self.crisis:
...            yield "$100"
>>> hsbc = Bank() # When everything's ok the ATM gives you as much as you want
>>> corner_street_atm = hsbc.create_atm()
>>> print(corner_street_atm.next())
$100
>>> print(corner_street_atm.next())
$100
>>> print([corner_street_atm.next() for cash in range(5)])
['$100', '$100', '$100', '$100', '$100']
>>> hsbc.crisis = True # Crisis is coming, no more money!
>>> print(corner_street_atm.next())
<type 'exceptions.StopIteration'>
>>> wall_street_atm = hsbc.create_atm() # It's even true for new ATMs
>>> print(wall_street_atm.next())
<type 'exceptions.StopIteration'>
>>> hsbc.crisis = False # The trouble is, even post-crisis the ATM remains empty
>>> print(corner_street_atm.next())
<type 'exceptions.StopIteration'>
>>> brand_new_atm = hsbc.create_atm() # Build a new one to get back in business
>>> for cash in brand_new_atm:
...    print cash
$100
$100
$100
$100
$100
$100
$100
$100
$100
...
```

Note: For Python 3, use`print(corner_street_atm.__next__())` or `print(next(corner_street_atm))`
It can be useful for various things like controlling access to a resource.

### Itertools, your best friend

The itertools module contains special functions to manipulate iterables. Ever wish to duplicate a generator?
Chain two generators? Group values in a nested list with a one-liner? `Map / Zip` without creating another list?
Then just `import itertools`.
An example? Let's see the possible orders of arrival for a four-horse race:

```python
>>> horses = [1, 2, 3, 4]
>>> races = itertools.permutations(horses)
>>> print(races)
<itertools.permutations object at 0xb754f1dc>
>>> print(list(itertools.permutations(horses)))
[(1, 2, 3, 4),
 (1, 2, 4, 3),
 (1, 3, 2, 4),
 (1, 3, 4, 2),
 (1, 4, 2, 3),
 (1, 4, 3, 2),
 (2, 1, 3, 4),
 (2, 1, 4, 3),
 (2, 3, 1, 4),
 (2, 3, 4, 1),
 (2, 4, 1, 3),
 (2, 4, 3, 1),
 (3, 1, 2, 4),
 (3, 1, 4, 2),
 (3, 2, 1, 4),
 (3, 2, 4, 1),
 (3, 4, 1, 2),
 (3, 4, 2, 1),
 (4, 1, 2, 3),
 (4, 1, 3, 2),
 (4, 2, 1, 3),
 (4, 2, 3, 1),
 (4, 3, 1, 2),
 (4, 3, 2, 1)]
```


### Understanding the inner mechanisms of iteration

Iteration is a process implying iterables (implementing the `__iter__()` method) and iterators (implementing the `__next__()` method).
Iterables are any objects you can get an iterator from. Iterators are objects that let you iterate on iterables.
There is more about it in this article about how `for` loops work.

## [What are metaclasses in Python?](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)

**4439 Votes**, e-satis

A metaclass is the class of a class. Like a class defines how an instance of the class behaves, a metaclass defines how a class behaves. A class is an instance of a metaclass.

While in Python you can use arbitrary callables for metaclasses (like Jerub shows), the more useful approach is actually to make it an actual class itself. `type` is the usual metaclass in Python. In case you're wondering, yes, `type` is itself a class, and it is its own type. You won't be able to recreate something like `type` purely in Python, but Python cheats a little. To create your own metaclass in Python you really just want to subclass `type`.
A metaclass is most commonly used as a class-factory. Like you create an instance of the class by calling the class, Python creates a new class (when it executes the 'class' statement) by calling the metaclass. Combined with the normal `__init__` and `__new__` methods, metaclasses therefore allow you to do 'extra things' when creating a class, like registering the new class with some registry, or even replace the class with something else entirely.
When the `class` statement is executed, Python first executes the body of the `class` statement as a normal block of code. The resulting namespace (a dict) holds the attributes of the class-to-be. The metaclass is determined by looking at the baseclasses of the class-to-be (metaclasses are inherited), at the `__metaclass__` attribute of the class-to-be (if any) or the `__metaclass__` global variable. The metaclass is then called with the name, bases and attributes of the class to instantiate it.
However, metaclasses actually define the type of a class, not just a factory for it, so you can do much more with them. You can, for instance, define normal methods on the metaclass. These metaclass-methods are like classmethods, in that they can be called on the class without an instance, but they are also not like classmethods in that they cannot be called on an instance of the class. `type.__subclasses__()` is an example of a method on the `type` metaclass. You can also define the normal 'magic' methods, like `__add__`, `__iter__` and `__getattr__`, to implement or change how the class behaves.
Here's an aggregated example of the bits and pieces:

```python
def make_hook(f):
    """Decorator to turn 'foo' method into '__foo__'"""
    f.is_hook = 1
    return f

class MyType(type):
    def __new__(mcls, name, bases, attrs):

        if name.startswith('None'):
            return None

        # Go over attributes and see if they should be renamed.
        newattrs = {}
        for attrname, attrvalue in attrs.iteritems():
            if getattr(attrvalue, 'is_hook', 0):
                newattrs['__%s__' % attrname] = attrvalue
            else:
                newattrs[attrname] = attrvalue

        return super(MyType, mcls).__new__(mcls, name, bases, newattrs)

    def __init__(self, name, bases, attrs):
        super(MyType, self).__init__(name, bases, attrs)

        # classregistry.register(self, self.interfaces)
        print "Would register class %s now." % self

    def __add__(self, other):
        class AutoClass(self, other):
            pass
        return AutoClass
        # Alternatively, to autogenerate the classname as well as the class:
        # return type(self.__name__ + other.__name__, (self, other), {})

    def unregister(self):
        # classregistry.unregister(self)
        print "Would unregister class %s now." % self

class MyObject:
    __metaclass__ = MyType


class NoneSample(MyObject):
    pass

# Will print "NoneType None"
print type(NoneSample), repr(NoneSample)

class Example(MyObject):
    def __init__(self, value):
        self.value = value
    @make_hook
    def add(self, other):
        return self.__class__(self.value + other.value)

# Will unregister the class
Example.unregister()

inst = Example(10)
# Will fail with an AttributeError
#inst.unregister()

print inst + inst
class Sibling(MyObject):
    pass

ExampleSibling = Example + Sibling
# ExampleSibling is now a subclass of both Example and Sibling (with no
# content of its own) although it will believe it's called 'AutoClass'
print ExampleSibling
print ExampleSibling.__mro__
```

## [Does Python have a ternary conditional operator?](https://stackoverflow.com/questions/394809/does-python-have-a-ternary-conditional-operator)

**4241 Votes**, community-wiki

Yes, it was added in version 2.5.
The syntax is:

```python
a if condition else b
```

First `condition` is evaluated, then either ``a or ``b is returned based on the Boolean value of `condition`
If `condition` evaluates to True ``a is returned, else ``b is returned. 
For example:

```python
>>> 'true' if True else 'false'
'true'
>>> 'true' if False else 'false'
'false'
```

Note that conditionals are an expression, not a statement. This means you can't use assignments or `pass` or other statements in a conditional:

```python
>>> pass if False else x = 3
  File "<stdin>", line 1
    pass if False else x = 3
          ^
SyntaxError: invalid syntax
```

In such a case, you have to use a normal `if` statement instead of a conditional.

Keep in mind that it's frowned upon by some Pythonistas for several reasons:

The order of the arguments is different from many other languages (such as C, Ruby, Java, etc.), which may lead to bugs when people unfamiliar with Python's "surprising" behaviour use it (they may reverse the order).
Some find it "unwieldy", since it goes contrary to the normal flow of thought (thinking of the condition first and then the effects).
Stylistic reasons.

If you're having trouble remembering the order, then remember that if you read it out loud, you (almost) say what you mean. For example, `x = 4 if b > 8 else 9` is read aloud as `x will be 4 if b is greater than 8 otherwise 9`.
Official documentation:

Conditional expressions
Is there an equivalent of Cs ?: ternary operator?

## [How to check whether a file exists?](https://stackoverflow.com/questions/82831/how-to-check-whether-a-file-exists)

**4190 Votes**, spence91

If the reason you're checking is so you can do something like `if file_exists: open_it()`, it's safer to use a `try` around the attempt to open it. Checking and then opening risks the file being deleted or moved or something between when you check and when you try to open it.
If you're not planning to open the file immediately, you can use `os.path.isfile`

Return `True` if path is an existing regular file. This follows symbolic links, so both islink() and isfile() can be true for the same path.


```python
import os.path
os.path.isfile(fname) 
```

if you need to be sure it's a file.
Starting with Python 3.4, the `pathlib` module offers an object-oriented approach (backported to `pathlib2` in Python 2.7):

```python
from pathlib import Path

my_file = Path("/path/to/file")
if my_file.is_file():
    # file exists
```

To check a directory, do:

```python
if my_file.is_dir():
    # directory exists
```

To check whether a `Path` object exists independently of whether is it a file or directory, use `exists()`:

```python
if my_file.exists():
    # path exists
```

You can also use `resolve()` in a `try` block:

```python
try:
    my_abs_path = my_file.resolve():
except FileNotFoundError:
    # doesn't exist
else:
    # exists
```

## [What does if __name__ == __main__: do?](https://stackoverflow.com/questions/419163/what-does-if-name-main-do)

**3891 Votes**, Devoted

When the Python interpreter reads a source file, it executes all of the code found in it.
Before executing the code, it will define a few special variables. For example, if the Python interpreter is running that module (the source file) as the main program, it sets the special `__name__` variable to have a value `"__main__"`.  If this file is being imported from another module, `__name__` will be set to the module's name.
In the case of your script, let's assume that it's executing as the main function, e.g. you said something like

```python
python threading_example.py
```

on the command line. After setting up the special variables, it will execute the `import` statement and load those modules. It will then evaluate the `def` block, creating a function object and creating a variable called `myfunction` that points to the function object. It will then read the `if` statement and see that `__name__` does equal `"__main__"`, so it will execute the block shown there.
One reason for doing this is that sometimes you write a module (a `.py` file) where it can be executed directly. Alternatively, it can also be imported and used in another module. By doing the main check, you can have that code only execute when you want to run the module as a program and not have it execute when someone just wants to import your module and call your functions themselves.
See this page for some extra details.

## [Calling an external command in Python](https://stackoverflow.com/questions/89228/calling-an-external-command-in-python)

**3497 Votes**, freshWoWer

Look at the subprocess module in the standard library:

```python
from subprocess import call
call(["ls", "-l"])
```

The advantage of subprocess vs system is that it is more flexible (you can get the stdout, stderr, the "real" status code, better error handling, etc...). 
The official docs recommend the subprocess module over the alternative os.system():

The subprocess module provides more powerful facilities for spawning new processes and retrieving their results; using that module is preferable to using this function [`os.system()`].

The "Replacing Older Functions with the subprocess Module" section in the subprocess documentation may have some helpful recipes.
Official documentation on the subprocess module:

Python 2 - subprocess
Python 3 - subprocess

## [How to merge two dictionaries in a single expression?](https://stackoverflow.com/questions/38987/how-to-merge-two-dictionaries-in-a-single-expression)

**3064 Votes**, Carl Meyer

How can I merge two Python dictionaries in a single expression?

For dictionaries ``x and ``y, ``z becomes a merged dictionary with values from ``y replacing those from ``x.

In Python 3.5 or greater, :

```python
z = {**x, **y}
```

In Python 2, (or 3.4 or lower) write a function:

```python
def merge_two_dicts(x, y):
    z = x.copy()   # start with x's keys and values
    z.update(y)    # modifies z with y's keys and values & returns None
    return z
```

and

```python
z = merge_two_dicts(x, y)
```



### Explanation

Say you have two dicts and you want to merge them into a new dict without altering the original dicts:

```python
x = {'a': 1, 'b': 2}
y = {'b': 3, 'c': 4}
```

The desired result is to get a new dictionary (``z) with the values merged, and the second dict's values overwriting those from the first.

```python
>>> z
{'a': 1, 'b': 3, 'c': 4}
```

A new syntax for this, proposed in PEP 448 and available as of Python 3.5, is 

```python
z = {**x, **y}
```

And it is indeed a single expression. It is now showing as implemented in the release schedule for 3.5, PEP 478, and it has now made its way into What's New in Python 3.5 document.
However, since many organizations are still on Python 2, you may wish to do this in a backwards compatible way. The classically Pythonic way, available in Python 2 and Python 3.0-3.4, is to do this as a two-step process:

```python
z = x.copy()
z.update(y) # which returns None since it mutates z
```

In both approaches, ``y will come second and its values will replace ``x's values, thus `'b'` will point to ``3 in our final result.
Not yet on Python 3.5, but want a single expression
If you are not yet on Python 3.5, or need to write backward-compatible code, and you want this in a single expression, the most performant while correct approach is to put it in a function:

```python
def merge_two_dicts(x, y):
    """Given two dicts, merge them into a new dict as a shallow copy."""
    z = x.copy()
    z.update(y)
    return z
```

and then you have a single expression:

```python
z = merge_two_dicts(x, y)
```

You can also make a function to merge an undefined number of dicts, from zero to a very large number:

```python
def merge_dicts(*dict_args):
    """
    Given any number of dicts, shallow copy and merge into a new dict,
    precedence goes to key value pairs in latter dicts.
    """
    result = {}
    for dictionary in dict_args:
        result.update(dictionary)
    return result
```

This function will work in Python 2 and 3 for all dicts. e.g. given dicts ``a to ``g:

```python
z = merge_dicts(a, b, c, d, e, f, g) 
```

and key value pairs in ``g will take precedence over dicts ``a to ``f, and so on.
Critiques of Other Answers
Don't use what you see in the formerly accepted answer:

```python
z = dict(x.items() + y.items())
```

In Python 2, you create two lists in memory for each dict, create a third list in memory with length equal to the length of the first two put together, and then discard all three lists to create the dict. In Python 3, this will fail because you're adding two `dict_items` objects together, not two lists - 

```python
>>> c = dict(a.items() + b.items())
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'dict_items' and 'dict_items'
```

and you would have to explicitly create them as lists, e.g. `z = dict(list(x.items()) + list(y.items()))`. This is a waste of resources and computation power. 
Similarly, taking the union of `items()` in Python 3 (`viewitems()` in Python 2.7) will also fail when values are unhashable objects (like lists, for example). Even if your values are hashable, since sets are semantically unordered, the behavior is undefined in regards to precedence. So don't do this:

```python
>>> c = dict(a.items() | b.items())
```

This example demonstrates what happens when values are unhashable:

```python
>>> x = {'a': []}
>>> y = {'b': []}
>>> dict(x.items() | y.items())
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
```

Here's an example where y should have precedence, but instead the value from x is retained due to the arbitrary order of sets:

```python
>>> x = {'a': 2}
>>> y = {'a': 1}
>>> dict(x.items() | y.items())
{'a': 2}
```

Another hack you should not use:

```python
z = dict(x, **y)
```

This uses the `dict` constructor, and is very fast and memory efficient (even slightly more-so than our two-step process) but unless you know precisely what is happening here (that is, the second dict is being passed as keyword arguments to the dict constructor), it's difficult to read, it's not the intended usage, and so it is not Pythonic. 
Here's an example of the usage being remediated in django.
Dicts are intended to take hashable keys (e.g. frozensets or tuples), but this method fails in Python 3 when keys are not strings.

```python
>>> c = dict(a, **b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: keyword arguments must be strings
```

From the mailing list, Guido van Rossum, the creator of the language, wrote:

I am fine with
  declaring dict({}, **{1:3}) illegal, since after all it is abuse of
  the ** mechanism.

and 

Apparently dict(x, **y) is going around as "cool hack" for "call
  x.update(y) and return x". Personally I find it more despicable than
  cool.

It is my understanding (as well as the understanding of the creator of the language) that the intended usage for `dict(**y)` is for creating dicts for readability purposes, e.g.:

```python
dict(a=1, b=10, c=11)
```

instead of 

```python
{'a': 1, 'b': 10, 'c': 11}
```


### Response to comments


Despite what Guido says, `dict(x, **y)` is in line with the dict specification, which btw. works for both Python 2 and 3. The fact that this only works for string keys is a direct consequence of how keyword parameters work and not a short-comming of dict. Nor is using the ** operator in this place an abuse of the mechanism, in fact ** was designed precisely to pass dicts as keywords. 

Again, it doesn't work for 3 when keys are non-strings. The implicit calling contract is that namespaces take ordinary dicts, while users must only pass keyword arguments that are strings. All other callables enforced it. `dict` broke this consistency in Python 2:

```python
>>> foo(**{('a', 'b'): None})
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: foo() keywords must be strings
>>> dict(**{('a', 'b'): None})
{('a', 'b'): None}
```

This inconsistency was bad given other implementations of Python (Pypy, Jython, IronPython). Thus it was fixed in Python 3, as this usage could be a breaking change.
I submit to you that it is malicious incompetence to intentionally write code that only works in one version of a language or that only works given certain arbitrary constraints.
Another comment:

`dict(x.items() + y.items())` is still the most readable solution for Python 2. Readability counts. 

My response: `merge_two_dicts(x, y)` actually seems much clearer to me, if we're actually concerned about readability. And it is not forward compatible, as Python 2 is increasingly deprecated.
Less Performant But Correct Ad-hocs
These approaches are less performant, but they will provide correct behavior.
They will be much less performant than `copy` and `update` or the new unpacking because they iterate through each key-value pair at a higher level of abstraction, but they do respect the order of precedence (latter dicts have precedence)
You can also chain the dicts manually inside a dict comprehension:

```python
{k: v for d in dicts for k, v in d.items()} # iteritems in Python 2.7
```

or in python 2.6 (and perhaps as early as 2.4 when generator expressions were introduced):

```python
dict((k, v) for d in dicts for k, v in d.items())
```

`itertools.chain` will chain the iterators over the key-value pairs in the correct order:

```python
import itertools
z = dict(itertools.chain(x.iteritems(), y.iteritems()))
```

Performance Analysis
I'm only going to do the performance analysis of the usages known to behave correctly. 

```python
import timeit
```

The following is done on Ubuntu 14.04
In Python 2.7 (system Python):

```python
>>> min(timeit.repeat(lambda: merge_two_dicts(x, y)))
0.5726828575134277
>>> min(timeit.repeat(lambda: {k: v for d in (x, y) for k, v in d.items()} ))
1.163769006729126
>>> min(timeit.repeat(lambda: dict(itertools.chain(x.iteritems(), y.iteritems()))))
1.1614501476287842
>>> min(timeit.repeat(lambda: dict((k, v) for d in (x, y) for k, v in d.items())))
2.2345519065856934
```

In Python 3.5 (deadsnakes PPA):

```python
>>> min(timeit.repeat(lambda: {**x, **y}))
0.4094954460160807
>>> min(timeit.repeat(lambda: merge_two_dicts(x, y)))
0.7881555100320838
>>> min(timeit.repeat(lambda: {k: v for d in (x, y) for k, v in d.items()} ))
1.4525277839857154
>>> min(timeit.repeat(lambda: dict(itertools.chain(x.items(), y.items()))))
2.3143140770262107
>>> min(timeit.repeat(lambda: dict((k, v) for d in (x, y) for k, v in d.items())))
3.2069112799945287
```


### Resources on Dictionaries


My explanation of Python's dictionary implementation, updated for 3.6.
Answer on how to add new keys to a dictionary
Mapping two lists into a dictionary
The official Python docs on dictionaries 
The Dictionary Even Mightier - talk by Brandon Rhodes at Pycon 2017
Modern Python Dictionaries, A Confluence of Great Ideas - talk by Raymond Hettinger at Pycon 2017

## [How can I create a directory if it does not exist?](https://stackoverflow.com/questions/273192/how-can-i-create-a-directory-if-it-does-not-exist)

**2841 Votes**, Parand

I see two answers with good qualities, each with a small flaw, so I will give my take on it:
Try `os.path.exists`, and consider `os.makedirs` for the creation.

```python
import os
if not os.path.exists(directory):
    os.makedirs(directory)
```

As noted in comments and elsewhere, there's a race condition - if the directory is created between the `os.path.exists` and the `os.makedirs` calls, the `os.makedirs` will fail with an `OSError`. Unfortunately, blanket-catching `OSError` and continuing is not foolproof, as it will ignore a failure to create the directory due to other factors, such as insufficient permissions, full disk, etc.
One option would be to trap the `OSError` and examine the embedded error code (see Is there a cross-platform way of getting information from Pythons OSError):

```python
import os, errno

try:
    os.makedirs(directory)
except OSError as e:
    if e.errno != errno.EEXIST:
        raise
```

Alternatively, there could be a second `os.path.exists`, but suppose another created the directory after the first check, then removed it before the second one - we could still be fooled. 
Depending on the application, the danger of concurrent operations may be more or less than the danger posed by other factors such as file permissions. The developer would have to know more about the particular application being developed and its expected environment before choosing an implementation.

## [How do I sort a dictionary by value?](https://stackoverflow.com/questions/613183/how-do-i-sort-a-dictionary-by-value)

**2804 Votes**, Gern Blanston

It is not possible to sort a dictionary, only to get a representation of a dictionary that is sorted. Dictionaries are inherently orderless, but other types, such as lists and tuples, are not. So you need an ordered data type to represent sorted values, which will be a listprobably a list of tuples.
For instance,

```python
import operator
x = {1: 2, 3: 4, 4: 3, 2: 1, 0: 0}
sorted_x = sorted(x.items(), key=operator.itemgetter(1))
```

`sorted_x` will be a list of tuples sorted by the second element in each tuple. `dict(sorted_x) == x`.
And for those wishing to sort on keys instead of values:

```python
import operator
x = {1: 2, 3: 4, 4: 3, 2: 1, 0: 0}
sorted_x = sorted(x.items(), key=operator.itemgetter(0))
```

## [Does Python have a string 'contains' substring method?](https://stackoverflow.com/questions/3437059/does-python-have-a-string-contains-substring-method)

**2717 Votes**, Blankman

You can use the `in` operator:

```python
if "blah" not in somestring: 
    continue
```

## [How do I list all files of a directory?](https://stackoverflow.com/questions/3207219/how-do-i-list-all-files-of-a-directory)

**2654 Votes**, duhhunjonn

`os.listdir()` will get you everything that's in a directory - files and directories.
If you want just files, you could either filter this down using `os.path`:

```python
from os import listdir
from os.path import isfile, join
onlyfiles = [f for f in listdir(mypath) if isfile(join(mypath, f))]
```

or you could use `os.walk()` which will yield two lists for each directory it visits - splitting into files and dirs for you. If you only want the top directory you can just break the first time it yields

```python
from os import walk

f = []
for (dirpath, dirnames, filenames) in walk(mypath):
    f.extend(filenames)
    break
```

And lastly, as that example shows, adding one list to another you can either use `.extend()` or 

```python
>>> q = [1, 2, 3]
>>> w = [4, 5, 6]
>>> q = q + w
>>> q
[1, 2, 3, 4, 5, 6]
```

Personally, I prefer `.extend()`

## [How do I check if a list is empty?](https://stackoverflow.com/questions/53513/how-do-i-check-if-a-list-is-empty)

**2603 Votes**, Ray Vega

```python
if not a:
  print("List is empty")
```

Using the implicit booleanness of the empty list is quite pythonic.

## [Difference between append vs. extend list methods in Python](https://stackoverflow.com/questions/252703/difference-between-append-vs-extend-list-methods-in-python)

**2586 Votes**, Claudiu

`append`: Appends object at end.

```python
x = [1, 2, 3]
x.append([4, 5])
print (x)
```

gives you: `[1, 2, 3, [4, 5]]`

`extend`: Extends list by appending elements from the iterable.

```python
x = [1, 2, 3]
x.extend([4, 5])
print (x)
```

gives you: `[1, 2, 3, 4, 5]`

## [What is the difference between @staticmethod and @classmethod in Python?](https://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python)

**2547 Votes**, Daryl Spitzer

Maybe a bit of example code will help: Notice the difference in the call signatures of `foo`, `class_foo` and `static_foo`:

```python
class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)"%(self,x)

    @classmethod
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)"%(cls,x)

    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)"%x    

a=A()
```

Below is the usual way an object instance calls a method. The object instance, ``a, is implicitly passed as the first argument.

```python
a.foo(1)
# executing foo(<__main__.A object at 0xb7dbef0c>,1)
```


With classmethods, the class of the object instance is implicitly passed as the first argument instead of `self`.

```python
a.class_foo(1)
# executing class_foo(<class '__main__.A'>,1)
```

You can also call `class_foo` using the class. In fact, if you define something to be
a classmethod, it is probably because you intend to call it from the class rather than from a class instance. `A.foo(1)` would have raised a TypeError, but `A.class_foo(1)` works just fine:

```python
A.class_foo(1)
# executing class_foo(<class '__main__.A'>,1)
```

One use people have found for class methods is to create inheritable alternative constructors.

With staticmethods, neither `self` (the object instance) nor  `cls` (the class) is implicitly passed as the first argument. They behave like plain functions except that you can call them from an instance or the class:

```python
a.static_foo(1)
# executing static_foo(1)

A.static_foo('hi')
# executing static_foo(hi)
```

Staticmethods are used to group functions which have some logical connection with a class to the class.

`foo` is just a function, but when you call `a.foo` you don't just get the function,
you get a "partially applied" version of the function with the object instance ``a bound as the first argument to the function. `foo` expects 2 arguments, while `a.foo` only expects 1 argument.
``a is bound to `foo`. That is what is meant by the term "bound" below:

```python
print(a.foo)
# <bound method A.foo of <__main__.A object at 0xb7d52f0c>>
```

With `a.class_foo`, ``a is not bound to `class_foo`, rather the class ``A is bound to `class_foo`.

```python
print(a.class_foo)
# <bound method type.class_foo of <class '__main__.A'>>
```

Here, with a staticmethod, even though it is a method, `a.static_foo` just returns
a good 'ole function with no arguments bound. `static_foo` expects 1 argument, and
`a.static_foo` expects 1 argument too.

```python
print(a.static_foo)
# <function static_foo at 0xb7d479cc>
```

And of course the same thing happens when you call `static_foo` with the class ``A instead.

```python
print(A.static_foo)
# <function static_foo at 0xb7d479cc>
```

## [Accessing the index in 'for' loops?](https://stackoverflow.com/questions/522563/accessing-the-index-in-for-loops)

**2438 Votes**, Joan Venge

Using an additional state variable, such as an index variable (which you would normally use in languages such as C or PHP), is considered non-pythonic.
The better option is to use the built-in function `enumerate()`, available in both Python 2 and 3:

```python
for idx, val in enumerate(ints):
    print(idx, val)
```

Check out PEP 279 for more.

## [Using global variables in a function](https://stackoverflow.com/questions/423379/using-global-variables-in-a-function)

**2388 Votes**, user46646

You can use a global variable in other functions by declaring it as `global` in each function that assigns to it:

```python
globvar = 0

def set_globvar_to_one():
    global globvar    # Needed to modify global copy of globvar
    globvar = 1

def print_globvar():
    print(globvar)     # No need for global declaration to read value of globvar

set_globvar_to_one()
print_globvar()       # Prints 1
```

I imagine the reason for it is that, since global variables are so dangerous, Python wants to make sure that you really know that's what you're playing with by explicitly requiring the `global` keyword.
See other answers if you want to share a global variable across modules.

## [How to make a chain of function decorators?](https://stackoverflow.com/questions/739654/how-to-make-a-chain-of-function-decorators)

**2351 Votes**, Imran

Check out the documentation to see how decorators work. Here is what you asked for:

```python
def makebold(fn):
    def wrapped():
        return "<b>" + fn() + "</b>"
    return wrapped

def makeitalic(fn):
    def wrapped():
        return "<i>" + fn() + "</i>"
    return wrapped

@makebold
@makeitalic
def hello():
    return "hello world"

print hello() ## returns "<b><i>hello world</i></b>"
```

## [Understanding Python's slice notation](https://stackoverflow.com/questions/509211/understanding-pythons-slice-notation)

**2183 Votes**, Simon

It's pretty simple really:

```python
a[start:end] # items start through end-1
a[start:]    # items start through the rest of the array
a[:end]      # items from the beginning through end-1
a[:]         # a copy of the whole array
```

There is also the `step` value, which can be used with any of the above:

```python
a[start:end:step] # start through not past end, by step
```

The key point to remember is that the `:end` value represents the first value that is not in the selected slice. So, the difference beween `end` and `start` is the number of elements selected (if `step` is 1, the default).
The other feature is that `start` or `end` may be a negative number, which means it counts from the end of the array instead of the beginning. So:

```python
a[-1]    # last item in the array
a[-2:]   # last two items in the array
a[:-2]   # everything except the last two items
```

Similarly, `step` may be a negative number:

```python
a[::-1]    # all items in the array, reversed
a[1::-1]   # the first two items, reversed
a[:-3:-1]  # the last two items, reversed
a[-3::-1]  # everything except the last two items, reversed
```

Python is kind to the programmer if there are fewer items than you ask for. For example, if you ask for `a[:-2]` and ``a only contains one element, you get an empty list instead of an error. Sometimes you would prefer the error, so you have to be aware that this may happen.

## [How do I install pip on Windows?](https://stackoverflow.com/questions/4750806/how-do-i-install-pip-on-windows)

**2173 Votes**, community-wiki

### Python 2.7.9+ and 3.4+

Good news! Python 3.4 (released March 2014) and Python 2.7.9 (released December 2014) ship with Pip. This is the best feature of any Python release. It makes the community's wealth of libraries accessible to everyone. Newbies are no longer excluded from using community libraries by the prohibitive difficulty of setup. In shipping with a package manager, Python joins Ruby, Node.js, Haskell, Perl, Go--almost every other contemporary language with a majority open-source community. Thank you Python.
Of course, that doesn't mean Python packaging is problem solved. The experience remains frustrating. I discuss this in Stack Overflow question Does Python have a package/module management system?.
And, alas for everyone using Python 2.7.8 or earlier (a sizable portion of the community). There's no plan to ship Pip to you. Manual instructions follow.

### Python 2  2.7.8 and Python 3  3.3

Flying in the face of its 'batteries included' motto, Python ships without a package manager. To make matters worse, Pip was--until recently--ironically difficult to install.
Official instructions
Per https://pip.pypa.io/en/stable/installing/#do-i-need-to-install-pip:
Download `get-pip.py`, being careful to save it as a `.py` file rather than `.txt`. Then, run it from the command prompt:

```python
python get-pip.py
```

You possibly need an administrator command prompt to do this. Follow Start a Command Prompt as an Administrator (Microsoft TechNet).
This installs the pip package, which (in Windows) contains ...\Scripts\pip.exe that path must be in PATH environment variable to use pip from the command line (see the second part of 'Alternative Instructions' for adding it to your PATH,
Alternative instructions
The official documentation tells users to install Pip and each of its dependencies from source. That's tedious for the experienced and prohibitively difficult for newbies.
For our sake, Christoph Gohlke prepares Windows installers (`.msi`) for popular Python packages. He builds installers for all Python versions, both 32 and 64 bit. You need to:

Install setuptools
Install pip

For me, this installed Pip at `C:\Python27\Scripts\pip.exe`. Find `pip.exe` on your computer, then add its folder (for example, `C:\Python27\Scripts`) to your path (Start / Edit environment variables). Now you should be able to run `pip` from the command line. Try installing a package:

```python
pip install httpie
```

There you go (hopefully)! Solutions for common problems are given below:
Proxy problems
If you work in an office, you might be behind an HTTP proxy. If so, set the environment variables `http_proxy` and `https_proxy`. Most Python applications (and other free software) respect these. Example syntax:

```python
http://proxy_url:port
http://username:password@proxy_url:port
```

If you're really unlucky, your proxy might be a Microsoft NTLM proxy. Free software can't cope. The only solution is to install a free software friendly proxy that forwards to the nasty proxy. http://cntlm.sourceforge.net/
Unable to find vcvarsall.bat
Python modules can be partly written in C or C++. Pip tries to compile from source. If you don't have a C/C++ compiler installed and configured, you'll see this cryptic error message.

Error: Unable to find vcvarsall.bat

You can fix that by installing a C++ compiler such as MinGW or Visual C++. Microsoft actually ships one specifically for use with Python. Or try Microsoft Visual C++ Compiler for Python 2.7.
Often though it's easier to check Christoph's site for your package.

## [Finding the index of an item given a list containing it in Python](https://stackoverflow.com/questions/176918/finding-the-index-of-an-item-given-a-list-containing-it-in-python)

**2132 Votes**, Eugene M

```python
>>> ["foo", "bar", "baz"].index("bar")
1
```

Reference: [Data Structures > More on Lists][http://docs.python.org/2/tutorial/datastructures.html#more-on-lists]
Note that this only returns the index of the first instance of the item, and results in a `ValueError` if it's not present.

```python
>>> [1, 1].index(1)
0
>>> [1, 1].index(2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: 2 is not in list
```

If the item might not be present in the list, you should either 

check for it first with `item in my_list` (clean, readable approach), or
wrap the `index` call in a `try/except` block which catches `ValueError` (probably faster, at least when the list to search is long and the item is usually present.)

## [Check if a given key already exists in a dictionary](https://stackoverflow.com/questions/1602934/check-if-a-given-key-already-exists-in-a-dictionary)

**2070 Votes**, Mohan Gulati

`in` is the intended way to test for the existence of a key in a `dict`.

```python
d = dict()

for i in xrange(100):
    key = i % 10
    if key in d:
        d[key] += 1
    else:
        d[key] = 1
```

If you wanted a default, you can always use `dict.get()`:

```python
d = dict()

for i in xrange(100):
    key = i % 10
    d[key] = d.get(key, 0) + 1
```

... and if you wanted to always ensure a default value for any key you can use `defaultdict` from the `collections` module, like so:

```python
from collections import defaultdict

d = defaultdict(lambda: 0)

for i in xrange(100):
    d[i % 10] += 1
```

... but in general, the `in` keyword is the best way to do it.

## [Difference between __str__ and __repr__?](https://stackoverflow.com/questions/1436703/difference-between-str-and-repr)

**2030 Votes**, Casebash

Alex summarized well but, surprisingly, was too succinct.
First, let me reiterate the main points in Alexs post:

The default implementation is useless (its hard to think of one which wouldnt be, but yeah)
`__repr__` goal is to be unambiguous
`__str__` goal is to be readable
Containers `__str__` uses contained objects `__repr__`

Default implementation is useless
This is mostly a surprise because Pythons defaults tend to be fairly useful. However, in this case, having a default for `__repr__` which would act like:

```python
return "%s(%r)" % (self.__class__, self.__dict__)
```

would have been too dangerous (for example, too easy to get into infinite recursion if objects reference each other). So Python cops out. Note that there is one default which is true: if `__repr__` is defined, and `__str__` is not, the object will behave as though `__str__=__repr__`.
This means, in simple terms: almost every object you implement should have a functional `__repr__` thats usable for understanding the object. Implementing `__str__` is optional: do that if you need a pretty print functionality (for example, used by a report generator).
The goal of `__repr__` is to be unambiguous
Let me come right out and say it  I do not believe in debuggers. I dont really know how to use any debugger, and have never used one seriously. Furthermore, I believe that the big fault in debuggers is their basic nature  most failures I debug happened a long long time ago, in a galaxy far far away. This means that I do believe, with religious fervor, in logging. Logging is the lifeblood of any decent fire-and-forget server system. Python makes it easy to log: with maybe some project specific wrappers, all you need is a

```python
log(INFO, "I am in the weird function and a is", a, "and b is", b, "but I got a null C  using default", default_c)
```

But you have to do the last step  make sure every object you implement has a useful repr, so code like that can just work. This is why the eval thing comes up: if you have enough information so `eval(repr(c))==c`, that means you know everything there is to know about ``c. If thats easy enough, at least in a fuzzy way, do it. If not, make sure you have enough information about ``c anyway. I usually use an eval-like format: `"MyClass(this=%r,that=%r)" % (self.this,self.that)`. It does not mean that you can actually construct MyClass, or that those are the right constructor arguments  but it is a useful form to express this is everything you need to know about this instance.
Note: I used `%r` above, not `%s`. You always want to use `repr()` [or `%r` formatting character, equivalently] inside `__repr__` implementation, or youre defeating the goal of repr. You want to be able to differentiate `MyClass(3)` and `MyClass("3")`.
The goal of `__str__` is to be readable
Specifically, it is not intended to be unambiguous  notice that `str(3)==str("3")`. Likewise, if you implement an IP abstraction, having the str of it look like 192.168.1.1 is just fine. When implementing a date/time abstraction, the str can be "2010/4/12 15:35:22", etc. The goal is to represent it in a way that a user, not a programmer, would want to read it. Chop off useless digits, pretend to be some other class  as long is it supports readability, it is an improvement.
Containers `__str__` uses contained objects `__repr__`
This seems surprising, doesnt it? It is a little, but how readable would

```python
[moshe is, 3, hello
world, this is a list, oh I don't know, containing just 4 elements]
```

be? Not very. Specifically, the strings in a container would find it way too easy to disturb its string representation. In the face of ambiguity, remember, Python resists the temptation to guess. If you want the above behavior when youre printing a list, just

```python
print "[" + ", ".join(l) + "]"
```

(you can probably also figure out what to do about dictionaries.
Summary
Implement `__repr__` for any class you implement. This should be second nature. Implement `__str__` if you think it would be useful to have a string version which errs on the side of more readability in favor of more ambiguity.

## [Least Astonishment and the Mutable Default Argument](https://stackoverflow.com/questions/1132941/least-astonishment-and-the-mutable-default-argument)

**2010 Votes**, Stefano Borini

Actually, this is not a design flaw, and it is not because of internals, or performance.
It comes simply from the fact that functions in Python are first-class objects, and not only a piece of code.
As soon as you get to think into this way, then it completely makes sense: a function is an object being evaluated on its definition; default parameters are kind of "member data" and therefore their state may change from one call to the other - exactly as in any other object.
In any case, Effbot has a very nice explanation of the reasons for this behavior in Default Parameter Values in Python.
I found it very clear, and I really suggest reading it for a better knowledge of how function objects work.

## [How do I pass a variable by reference?](https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference)

**2006 Votes**, David Sykes

Arguments are passed by assignment. The rationale behind this is twofold:

the parameter passed in is actually a reference to an object (but the reference is passed by value)
some data types are mutable, but others aren't

So:

If you pass a mutable object into a method, the method gets a reference to that same object and you can mutate it to your heart's delight, but if you rebind the reference in the method, the outer scope will know nothing about it, and after you're done, the outer reference will still point at the original object. 
If you pass an immutable object to a method, you still can't rebind the outer reference, and you can't even mutate the object.

To make it even more clear, let's have some examples. 

### List - a mutable type

Let's try to modify the list that was passed to a method:

```python
def try_to_change_list_contents(the_list):
    print('got', the_list)
    the_list.append('four')
    print('changed to', the_list)

outer_list = ['one', 'two', 'three']

print('before, outer_list =', outer_list)
try_to_change_list_contents(outer_list)
print('after, outer_list =', outer_list)
```

Output:

```python
before, outer_list = ['one', 'two', 'three']
got ['one', 'two', 'three']
changed to ['one', 'two', 'three', 'four']
after, outer_list = ['one', 'two', 'three', 'four']
```

Since the parameter passed in is a reference to `outer_list`, not a copy of it, we can use the mutating list methods to change it and have the changes reflected in the outer scope.
Now let's see what happens when we try to change the reference that was passed in as a parameter:

```python
def try_to_change_list_reference(the_list):
    print('got', the_list)
    the_list = ['and', 'we', 'can', 'not', 'lie']
    print('set to', the_list)

outer_list = ['we', 'like', 'proper', 'English']

print('before, outer_list =', outer_list)
try_to_change_list_reference(outer_list)
print('after, outer_list =', outer_list)
```

Output:

```python
before, outer_list = ['we', 'like', 'proper', 'English']
got ['we', 'like', 'proper', 'English']
set to ['and', 'we', 'can', 'not', 'lie']
after, outer_list = ['we', 'like', 'proper', 'English']
```

Since the `the_list` parameter was passed by value, assigning a new list to it had no effect that the code outside the method could see. The `the_list` was a copy of the `outer_list` reference, and we had `the_list` point to a new list, but there was no way to change where `outer_list` pointed.

### String - an immutable type

It's immutable, so there's nothing we can do to change the contents of the string
Now, let's try to change the reference

```python
def try_to_change_string_reference(the_string):
    print('got', the_string)
    the_string = 'In a kingdom by the sea'
    print('set to', the_string)

outer_string = 'It was many and many a year ago'

print('before, outer_string =', outer_string)
try_to_change_string_reference(outer_string)
print('after, outer_string =', outer_string)
```

Output:

```python
before, outer_string = It was many and many a year ago
got It was many and many a year ago
set to In a kingdom by the sea
after, outer_string = It was many and many a year ago
```

Again, since the `the_string` parameter was passed by value, assigning a new string to it had no effect that the code outside the method could see. The `the_string` was a copy of the `outer_string` reference, and we had `the_string` point to a new string, but there was no way to change where `outer_string` pointed.
I hope this clears things up a little.
EDIT: It's been noted that this doesn't answer the question that @David originally asked, "Is there something I can do to pass the variable by actual reference?". Let's work on that.

### How do we get around this?

As @Andrea's answer shows, you could return the new value. This doesn't change the way things are passed in, but does let you get the information you want back out:

```python
def return_a_whole_new_string(the_string):
    new_string = something_to_do_with_the_old_string(the_string)
    return new_string

# then you could call it like
my_string = return_a_whole_new_string(my_string)
```

If you really wanted to avoid using a return value, you could create a class to hold your value and pass it into the function or use an existing class, like a list:

```python
def use_a_wrapper_to_simulate_pass_by_reference(stuff_to_change):
    new_string = something_to_do_with_the_old_string(stuff_to_change[0])
    stuff_to_change[0] = new_string

# then you could call it like
wrapper = [my_string]
use_a_wrapper_to_simulate_pass_by_reference(wrapper)

do_something_with(wrapper[0])
```

Although this seems a little cumbersome.

## [Iterating over dictionaries using 'for' loops](https://stackoverflow.com/questions/3294889/iterating-over-dictionaries-using-for-loops)

**1979 Votes**, TopChef

`key` is just a variable name.  

```python
for key in d:
```

will simply loop over the keys in the dictionary, rather than the keys and values.  To loop over both key and value you can use the following:
For Python 2.x:

```python
for key, value in d.iteritems():
```

For Python 3.x:

```python
for key, value in d.items():
```

To test for yourself, change the word `key` to `poop`.
For Python 3.x, `iteritems()` has been replaced with simply `items()`, which returns a set-like view backed by the dict, like `iteritems()` but even better. 
This is also available in 2.7 as `viewitems()`. 
The operation `items()` will work for both 2 and 3, but in 2 it will return a list of the dictionary's `(key, value)` pairs, which will not reflect changes to the dict that happen after the `items()` call. If you want the 2.x behavior in 3.x, you can call `list(d.items())`.

## [Making a flat list out of list of lists in Python](https://stackoverflow.com/questions/952914/making-a-flat-list-out-of-list-of-lists-in-python)

**1959 Votes**, Emma

```python
flat_list = [item for sublist in l for item in sublist]
```

which means:

```python
for sublist in l:
    for item in sublist:
        flat_list.append(item)
```

is faster than the shortcuts posted so far. (``l is the list to flatten.)
Here is a the corresponding function:

```python
flatten = lambda l: [item for sublist in l for item in sublist]
```

For evidence, as always, you can use the `timeit` module in the standard library:

```python
$ python -mtimeit -s'l=[[1,2,3],[4,5,6], [7], [8,9]]*99' '[item for sublist in l for item in sublist]'
10000 loops, best of 3: 143 usec per loop
$ python -mtimeit -s'l=[[1,2,3],[4,5,6], [7], [8,9]]*99' 'sum(l, [])'
1000 loops, best of 3: 969 usec per loop
$ python -mtimeit -s'l=[[1,2,3],[4,5,6], [7], [8,9]]*99' 'reduce(lambda x,y: x+y,l)'
1000 loops, best of 3: 1.1 msec per loop
```

Explanation: the shortcuts based on ``+ (including the implied use in `sum`) are, of necessity, `O(L**2)` when there are L sublists -- as the intermediate result list keeps getting longer, at each step a new intermediate result list object gets allocated, and all the items in the previous intermediate result must be copied over (as well as a few new ones added at the end). So (for simplicity and without actual loss of generality) say you have L sublists of I items each: the first I items are copied back and forth L-1 times, the second I items L-2 times, and so on; total number of copies is I times the sum of x for x from 1 to L excluded, i.e., `I * (L**2)/2`.
The list comprehension just generates one list, once, and copies each item over (from its original place of residence to the result list) also exactly once.

## [How can I make a time delay in Python?](https://stackoverflow.com/questions/510348/how-can-i-make-a-time-delay-in-python)

**1919 Votes**, user46646

```python
import time
time.sleep(5)   # delays for 5 seconds. You can Also Use Float Value.
```

Here is another example where something is run approximately once a minute:

```python
import time 
while True:
    print("This prints once a minute.")
    time.sleep(60)   # Delay for 1 minute (60 seconds).
```

## [Understanding Python super() with __init__() methods [duplicate]](https://stackoverflow.com/questions/576169/understanding-python-super-with-init-methods)

**1901 Votes**, Mizipzor

`super()` lets you avoid referring to the base class explicitly, which can be nice. But the main advantage comes with multiple inheritance, where all sorts of fun stuff can happen. See the standard docs on super if you haven't already.
Note that the syntax changed in Python 3.0: you can just say `super().__init__()` instead of `super(ChildB, self).__init__()` which IMO is quite a bit nicer.

## [Catch multiple exceptions in one line (except block)](https://stackoverflow.com/questions/6470428/catch-multiple-exceptions-in-one-line-except-block)

**1876 Votes**, inspectorG4dget

From Python Documentation:

An except clause may name multiple exceptions as a parenthesized tuple, for example


```python
except (IDontLikeYouException, YouAreBeingMeanException) as e:
    pass
```

Or, for Python 2 only:

```python
except (IDontLikeYouException, YouAreBeingMeanException), e:
    pass
```

Separating the exception from the variable with a comma will still work in Python 2.6 and 2.7, but is now deprecated and does not work in Python 3; now you should be using `as`.

## [How to get current time in Python?](https://stackoverflow.com/questions/415511/how-to-get-current-time-in-python)

**1863 Votes**, user46646

```python
>>> import datetime
>>> datetime.datetime.now()
datetime(2009, 1, 6, 15, 8, 24, 78915)
```

And just the time:

```python
>>> datetime.datetime.time(datetime.datetime.now())
datetime.time(15, 8, 24, 78915)
```

The same but slightly more compact:

```python
>>> datetime.datetime.now().time()
```

See the documentation for more info.
To save typing, you can import the `datetime` object from the `datetime` module:

```python
>>> from datetime import datetime
```

Then remove the leading `datetime.` from all the above.

## [Is there a way to run Python on Android? [closed]](https://stackoverflow.com/questions/101754/is-there-a-way-to-run-python-on-android)

**1841 Votes**, e-satis

One way is to use Kivy:

Open source Python library for rapid development of applications
  that make use of innovative user interfaces, such as multi-touch apps.



Kivy runs on Linux, Windows, OS X, Android and iOS. You can run the same [python] code on all supported platforms.

Kivy Showcase app

## [Add new keys to a dictionary?](https://stackoverflow.com/questions/1024847/add-new-keys-to-a-dictionary)

**1831 Votes**, lfaraone

```python
>>> d = {'key':'value'}
>>> print(d)
{'key': 'value'}
>>> d['mynewkey'] = 'mynewvalue'
>>> print(d)
{'mynewkey': 'mynewvalue', 'key': 'value'}
```

## [How do I parse a string to a float or int in Python?](https://stackoverflow.com/questions/379906/how-do-i-parse-a-string-to-a-float-or-int-in-python)

**1708 Votes**, Tristan Havelick

```python
>>> a = "545.2222"
>>> float(a)
545.22220000000004
>>> int(float(a))
545
```

## [How do I install pip on macOS or OS X?](https://stackoverflow.com/questions/17271319/how-do-i-install-pip-on-macos-or-os-x)

**1677 Votes**, The System

All you need to do is

```python
sudo easy_install pip
```

## [In Python, how do I read a file line-by-line into a list?](https://stackoverflow.com/questions/3277503/in-python-how-do-i-read-a-file-line-by-line-into-a-list)

**1630 Votes**, Julie Raswick

```python
with open(fname) as f:
    content = f.readlines()
# you may also want to remove whitespace characters like `\n` at the end of each line
content = [x.strip() for x in content] 
```

I'm guessing that you meant `list` and not array.

## [How to clone or copy a list?](https://stackoverflow.com/questions/2612802/how-to-clone-or-copy-a-list)

**1612 Votes**, aF.

With `new_list = my_list`, you don't actually have two lists. The assignment just copies the reference to the list, not the actual list, so both `new_list` and `my_list` refer to the same list after the assignment.
To actually copy the list, you have various possibilities:

You can slice it: 

```python
new_list = old_list[:]
```

Alex Martelli's opinion (at least back in 2007) about this is, that it is a weird syntax and it does not make sense to use it ever. ;) (In his opinion, the next one is more readable).
You can use the built in `list()` function:

```python
new_list = list(old_list)
```

You can use generic `copy.copy()`:

```python
import copy
new_list = copy.copy(old_list)
```

This is a little slower than `list()` because it has to find out the datatype of `old_list` first.
If the list contains objects and you want to copy them as well, use generic `copy.deepcopy()`:

```python
import copy
new_list = copy.deepcopy(old_list)
```

Obviously the slowest and most memory-needing method, but sometimes unavoidable.

Example:

```python
import copy

class Foo(object):
    def __init__(self, val):
         self.val = val

    def __repr__(self):
        return str(self.val)

foo = Foo(1)

a = ['foo', foo]
b = a[:]
c = list(a)
d = copy.copy(a)
e = copy.deepcopy(a)

# edit orignal list and instance 
a.append('baz')
foo.val = 5

print('original: %r\n slice: %r\n list(): %r\n copy: %r\n deepcopy: %r'
      % (a, b, c, d, e))
```

Result:

```python
original: ['foo', 5, 'baz']
slice: ['foo', 5]
list(): ['foo', 5]
copy: ['foo', 5]
deepcopy: ['foo', 1]
```

## [How to concatenate two lists in Python?](https://stackoverflow.com/questions/1720421/how-to-concatenate-two-lists-in-python)

**1573 Votes**, y2k

You can use the ``+ operator to combine them:

```python
listone = [1,2,3]
listtwo = [4,5,6]

mergedlist = listone + listtwo
```

Output:

```python
>>> mergedlist
[1,2,3,4,5,6]
```

## [Is there a way to substring a string in Python?](https://stackoverflow.com/questions/663171/is-there-a-way-to-substring-a-string-in-python)

**1531 Votes**, Joan Venge

```python
>>> x = "Hello World!"
>>> x[2:]
'llo World!'
>>> x[:2]
'He'
>>> x[:-2]
'Hello Worl'
>>> x[-2:]
'd!'
>>> x[2:-2]
'llo Worl'
```

Python calls this concept "slicing" and it works on more than just strings. Take a look here for a comprehensive introduction.

## [How do you split a list into evenly sized chunks?](https://stackoverflow.com/questions/312443/how-do-you-split-a-list-into-evenly-sized-chunks)

**1523 Votes**, jespern

Here's a generator that yields the chunks you want:

```python
def chunks(l, n):
    """Yield successive n-sized chunks from l."""
    for i in range(0, len(l), n):
        yield l[i:i + n]
```



```python
import pprint
pprint.pprint(list(chunks(range(10, 75), 10)))
[[10, 11, 12, 13, 14, 15, 16, 17, 18, 19],
 [20, 21, 22, 23, 24, 25, 26, 27, 28, 29],
 [30, 31, 32, 33, 34, 35, 36, 37, 38, 39],
 [40, 41, 42, 43, 44, 45, 46, 47, 48, 49],
 [50, 51, 52, 53, 54, 55, 56, 57, 58, 59],
 [60, 61, 62, 63, 64, 65, 66, 67, 68, 69],
 [70, 71, 72, 73, 74]]
```


If you're using Python 2, you should use `xrange()` instead of `range()`:

```python
def chunks(l, n):
    """Yield successive n-sized chunks from l."""
    for i in xrange(0, len(l), n):
        yield l[i:i + n]
```


Also you can simply use list comprehension instead of writing a function. Python 3:

```python
[l[i:i + n] for i in range(0, len(l), n)]
```

Python 2 version:

```python
[l[i:i + n] for i in xrange(0, len(l), n)]
```

## [What does ** (double star/asterisk) and * (star/asterisk) do for parameters?](https://stackoverflow.com/questions/36901/what-does-double-star-asterisk-and-star-asterisk-do-for-parameters)

**1496 Votes**, Todd

The `*args` and `**kwargs` is a common idiom to allow arbitrary number of arguments to functions as described in the section more on defining functions in the Python documentation.
The `*args` will give you all function parameters as a tuple:

```python
In [1]: def foo(*args):
   ...:     for a in args:
   ...:         print a
   ...:         
   ...:         

In [2]: foo(1)
1


In [4]: foo(1,2,3)
1
2
3
```

The `**kwargs` will give you all 
keyword arguments except for those corresponding to a formal parameter as a dictionary.

```python
In [5]: def bar(**kwargs):
   ...:     for a in kwargs:
   ...:         print a, kwargs[a]
   ...:         
   ...:         

In [6]: bar(name='one', age=27)
age 27
name one
```

Both idioms can be mixed with normal arguments to allow a set of fixed and some variable arguments:

```python
def foo(kind, *args, **kwargs):
   pass
```

Another usage of the `*l` idiom is to unpack argument lists when calling a function.

```python
In [9]: def foo(bar, lee):
   ...:     print bar, lee
   ...:     
   ...:     

In [10]: l = [1,2]

In [11]: foo(*l)
1 2
```

In Python 3 it is possible to use `*l` on the left side of an assignment (Extended Iterable Unpacking), though it gives a list instead of a tuple in this context:

```python
first, *rest = [1,2,3,4]
first, *l, last = [1,2,3,4]
```

Also Python 3 adds new semantic (refer PEP 3102):

```python
def func(arg1, arg2, arg3, *, kwarg1, kwarg2):
    pass
```

Such function accepts only 3 positional arguments, and everything after ``* can only be passed as keyword arguments.

## [Print in terminal with colors?](https://stackoverflow.com/questions/287871/print-in-terminal-with-colors)

**1475 Votes**, aboSamoor

This somewhat depends on what platform you are on. The most common way to do this is by printing ANSI escape sequences. For a simple example, here's some python code from the blender build scripts:

```python
class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'
```

To use code like this, you can do something like 

```python
print bcolors.WARNING + "Warning: No active frommets remain. Continue?" 
      + bcolors.ENDC
```

This will work on unixes including OS X, linux and windows (provided you use ANSICON, or in Windows 10 provided you enable VT100 emulation). There are ansi codes for setting the color, moving the cursor, and more.
If you are going to get complicated with this (and it sounds like you are if you are writing a game), you should look into the "curses" module, which handles a lot of the complicated parts of this for you. The Python Curses HowTO is a good introduction.
If you are not using extended ASCII (i.e. not on a PC), you are stuck with the ascii characters below 127, and '#' or '@' is probably your best bet for a block. If you can ensure your terminal is using a IBM extended ascii character set, you have many more options. Characters 176, 177, 178 and 219 are the "block characters".
Some modern text-based programs, such as "Dwarf Fortress", emulate text mode in a graphical mode, and use images of the classic PC font. You can find some of these bitmaps that you can use on the Dwarf Fortress Wiki see (user-made tilesets).
The Text Mode Demo Contest has more resources for doing graphics in text mode.
Hmm.. I think got a little carried away on this answer. I am in the midst of planning an epic text-based adventure game, though. Good luck with your colored text!

## [How to get the number of elements in a list in Python?](https://stackoverflow.com/questions/1712227/how-to-get-the-number-of-elements-in-a-list-in-python)

**1464 Votes**, y2k

The `len()` function can be used with a lot of types in Python - both built-in types and library types.

```python
>>> len([1,2,3])
3
```

## [Are static class variables possible?](https://stackoverflow.com/questions/68645/are-static-class-variables-possible)

**1459 Votes**, Andrew Walker

Variables declared inside the class definition, but not inside a method are class or static variables:

```python
>>> class MyClass:
...     i = 3
...
>>> MyClass.i
3 
```

As @millerdev points out, this creates a class-level ``i variable, but this is distinct from any instance-level ``i variable, so you could have

```python
>>> m = MyClass()
>>> m.i = 4
>>> MyClass.i, m.i
>>> (3, 4)
```

This is different from C++ and Java, but not so different from C#, where a static member can't be accessed using a reference to an instance.
See what the Python tutorial has to say on the subject of classes and class objects.
@Steve Johnson has already answered regarding static methods, also documented under "Built-in Functions" in the Python Library Reference.

```python
class C:
    @staticmethod
    def f(arg1, arg2, ...): ...
```

@beidy recommends classmethods over staticmethod, as the method then receives the class type as the first argument, but I'm still a little fuzzy on the advantages of this approach over staticmethod. If you are too, then it probably doesn't matter.

## [Hidden features of Python [closed]](https://stackoverflow.com/questions/101268/hidden-features-of-python)

**1420 Votes**, community-wiki

### Chaining comparison operators:


```python
>>> x = 5
>>> 1 < x < 10
True
>>> 10 < x < 20 
False
>>> x < 10 < x*10 < 100
True
>>> 10 > x <= 9
True
>>> 5 == x > 4
True
```

In case you're thinking it's doing `1 < x`, which comes out as `True`, and then comparing `True < 10`, which is also `True`, then no, that's really not what happens (see the last example.) It's really translating into `1 < x and x < 10`, and `x < 10 and 10 < x * 10 and x*10 < 100`, but with less typing and each term is only evaluated once.

## [Manually raising (throwing) an exception in Python](https://stackoverflow.com/questions/2052390/manually-raising-throwing-an-exception-in-python)

**1420 Votes**, TIMEX

### How do I manually throw/raise an exception in Python?


Use the most specific Exception constructor that semantically fits your issue.  
Be specific in your message, e.g.:

```python
raise ValueError('A very specific bad thing happened.')
```


### Don't raise generic exceptions

Avoid raising a generic Exception. To catch it, you'll have to catch all other more specific exceptions that subclass it.
Problem 1: Hiding bugs

```python
raise Exception('I know Python!') # Don't! If you catch, likely to hide bugs.
```

For example:

```python
def demo_bad_catch():
    try:
        raise ValueError('Represents a hidden bug, do not catch this')
        raise Exception('This is the exception you expect to handle')
    except Exception as error:
        print('Caught this error: ' + repr(error))

>>> demo_bad_catch()
Caught this error: ValueError('Represents a hidden bug, do not catch this',)
```

Problem 2: Won't catch
and more specific catches won't catch the general exception:

```python
def demo_no_catch():
    try:
        raise Exception('general exceptions not caught by specific handling')
    except ValueError as e:
        print('we will not catch exception: Exception')


>>> demo_no_catch()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in demo_no_catch
Exception: general exceptions not caught by specific handling
```


### Best Practices: `raise` statement

Instead, use the most specific Exception constructor that semantically fits your issue.

```python
raise ValueError('A very specific bad thing happened')
```

which also handily allows an arbitrary number of arguments to be passed to the constructor:

```python
raise ValueError('A very specific bad thing happened', 'foo', 'bar', 'baz') 
```

These arguments are accessed by the `args` attribute on the Exception object. For example:

```python
try:
    some_code_that_may_raise_our_value_error()
except ValueError as err:
    print(err.args)
```

prints 

```python
('message', 'foo', 'bar', 'baz')    
```

In Python 2.5, an actual `message` attribute was added to BaseException in favor of encouraging users to subclass Exceptions and stop using `args`, but the introduction of `message` and the original deprecation of args has been retracted.

### Best Practices: `except` clause

When inside an except clause, you might want to, for example, log that a specific type of error happened, and then re-raise. The best way to do this while preserving the stack trace is to use a bare raise statement. For example:

```python
logger = logging.getLogger(__name__)

try:
    do_something_in_app_that_breaks_easily()
except AppError as error:
    logger.error(error)
    raise                 # just this!
    # raise AppError      # Don't do this, you'll lose the stack trace!
```

Don't modify your errors... but if you insist.
You can preserve the stacktrace (and error value) with `sys.exc_info()`, but this is way more error prone and has compatibility problems between Python 2 and 3, prefer to use a bare `raise` to re-raise. 
To explain - the `sys.exc_info()` returns the type, value, and traceback. 

```python
type, value, traceback = sys.exc_info()
```

This is the syntax in Python 2 - note this is not compatible with Python 3:

```python
    raise AppError, error, sys.exc_info()[2] # avoid this.
    # Equivalently, as error *is* the second object:
    raise sys.exc_info()[0], sys.exc_info()[1], sys.exc_info()[2]
```

If you want to, you can modify what happens with your new raise - e.g. setting new args for the instance:

```python
def error():
    raise ValueError('oops!')

def catch_error_modify_message():
    try:
        error()
    except ValueError:
        error_type, error_instance, traceback = sys.exc_info()
        error_instance.args = (error_instance.args[0] + ' <modification>',)
        raise error_type, error_instance, traceback
```

And we have preserved the whole traceback while modifying the args. Note that this is not a best practice and it is invalid syntax in Python 3 (making keeping compatibility much harder to work around).

```python
>>> catch_error_modify_message()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in catch_error_modify_message
  File "<stdin>", line 2, in error
ValueError: oops! <modification>
```

In Python 3:

```python
    raise error.with_traceback(sys.exc_info()[2])
```

Again: avoid manually manipulating tracebacks. It's less efficient and more error prone. And if you're using threading and `sys.exc_info` you may even get the wrong traceback (especially if you're using exception handling for control flow - which I'd personally tend to avoid.)
Python 3, Exception chaining
In Python 3, you can chain Exceptions, which preserve tracebacks:

```python
    raise RuntimeError('specific message') from error
```

Be aware:

this does allow changing the error type raised, and
this is not compatible with Python 2.

Deprecated Methods:
These can easily hide and even get into production code. You want to raise an exception, and doing them will raise an exception, but not the one intended!
Valid in Python 2, but not in Python 3 is the following:

```python
raise ValueError, 'message' # Don't do this, it's deprecated!
```

Only valid in much older versions of Python (2.4 and lower), you may still see people raising strings:

```python
raise 'message' # really really wrong. don't do this.
```

In all modern versions, this will actually raise a TypeError, because you're not raising a BaseException type. If you're not checking for the right exception and don't have a reviewer that's aware of the issue, it could get into production.

### Example Usage

I raise Exceptions to warn consumers of my API if they're using it incorrectly:

```python
def api_func(foo):
    '''foo should be either 'baz' or 'bar'. returns something very useful.'''
    if foo not in _ALLOWED_ARGS:
        raise ValueError('{foo} wrong, use "baz" or "bar"'.format(foo=repr(foo)))
```


### Create your own error types when apropos


"I want to make an error on purpose, so that it would go into the except"

You can create your own error types, if you want to indicate something specific is wrong with your application, just subclass the appropriate point in the exception hierarchy:

```python
class MyAppLookupError(LookupError):
    '''raise this when there's a lookup error for my app'''
```

and usage:

```python
if important_key not in resource_dict and not ok_to_be_missing:
    raise MyAppLookupError('resource is missing, and that is not ok.')
```

## [How to convert string to lowercase in Python](https://stackoverflow.com/questions/6797984/how-to-convert-string-to-lowercase-in-python)

**1412 Votes**, Benjamin Didur

```python
s = "Kilometer"
print(s.lower())
```

The official documentation is `str.lower()`.

## [Find current directory and file's directory [duplicate]](https://stackoverflow.com/questions/5137497/find-current-directory-and-files-directory)

**1409 Votes**, John Howard

To get the full path to the directory a Python file is contained in, write this in that file:

```python
import os 
dir_path = os.path.dirname(os.path.realpath(__file__))
```

(Note that the incantation above won't work if you've already used `os.chdir()` to change your current working directory, since the value of the `__file__` constant is relative to the current working directory and is not changed by an `os.chdir()` call.)

To get the current working directory use 

```python
import os
cwd = os.getcwd()
```


Documentation references for the modules, constants and functions used above:

The `os` and `os.path` modules.
The `__file__` constant
`os.path.realpath(path)` (returns "the canonical path of the specified filename, eliminating any symbolic links encountered in the path")
`os.path.dirname(path)` (returns "the directory name of pathname `path`")
`os.getcwd()` (returns "a string representing the current working directory")
`os.chdir(path)` ("change the current working directory to `path`")

## [Converting string into datetime](https://stackoverflow.com/questions/466345/converting-string-into-datetime)

**1389 Votes**, Oli

`datetime.strptime` is the main routine for parsing strings into datetimes. It can handle all sorts of formats, with the format determined by a format string you give it:

```python
from datetime import datetime

datetime_object = datetime.strptime('Jun 1 2005  1:33PM', '%b %d %Y %I:%M%p')
```

The resulting `datetime` object is timezone-naive.
Links:

Python documentation for `strptime`: Python 2, Python 3
Python documentation for `strptime`/`strftime` format strings: Python 2, Python 3
strftime.org is also a really nice reference for strftime

Notes:

`strptime` = "string parse time"
`strftime` = "string format time"
Pronounce it out loud today & you won't have to search for it again in 6 months.

## [How do I copy a file in python?](https://stackoverflow.com/questions/123198/how-do-i-copy-a-file-in-python)

**1380 Votes**, Matt

`shutil` has many methods you can use. One of which is:

```python
from shutil import copyfile

copyfile(src, dst)
```

Copy the contents of the file named `src` to a file named `dst`. The destination location must be writable; otherwise, an `IOError` exception will be raised. If `dst` already exists, it will be replaced. Special files such as character or block devices and pipes cannot be copied with this function. `src` and `dst` are path names given as strings.

## [Replacements for switch statement in Python?](https://stackoverflow.com/questions/60208/replacements-for-switch-statement-in-python)

**1376 Votes**, Michael Schneider

You could use a dictionary:

```python
def f(x):
    return {
        'a': 1,
        'b': 2,
    }[x]
```
