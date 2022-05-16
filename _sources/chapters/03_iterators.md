---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

```{code-cell}
:tags: [remove-cell]
import os

os.chdir("..")
```

(chapter-iterators)=
Iterator & Generator Crash Course
=================================

:::{admonition} Learning Objectives
* Explain and use for-loops and comprehensions
* Use the `sys.getsizeof` function to get the size of an object in memory
* Explain what classes are and how they're typically used
* Create new classes
* Explain what methods and special methods are
* Identify some common special methods
* Define special methods for a class to customize behavior
* Explain what an iterator is, what an iterable is, and how they differ
* Use the `reversed`, `sorted`, `enumerate`, and `zip` functions
* Explain what the `itertools` module is
* Explain and use generator functions and generator expressions
* Explain the advantages and disadvantages of generators
:::

This chapter will help you understand how Python objects and classes work at a
lower level, particularly iterators and generators. Objects are the atoms of
Python, and knowing how to iterate over data is a fundamental programming
skill. Generators provide an elegant way to build complex data processing
pipelines that can efficiently handle big data and streaming data. Along the
way, the chapter also introduces a few of Python's advanced features.


(prerequisites)=
Prerequisites
-------------

This chapter assumes you already have basic familiarity with Python. DataLab's
[Python Basics Reader][py-basics] and its accompanying workshop provide a
suitable introduction.

[py-basics]: https://ucdavisdatalab.github.io/workshop_python_basics/

To follow along, you'll need the following software versions (or newer)
installed on your computer:

* [Python][] 3.10

One way to install these is to install the [Anaconda][] Python distribution.
Chapter 2 provides more details about Anaconda and the `conda` package manager.

[Python]: https://www.python.org/
[Anaconda]: https://www.anaconda.com/

[CLICK HERE][data-download] to get the data set used in the final example for
this chapter. The data set is in a `.tar.bz2` archive, which you'll need to
decompress. If your computer doesn't already have a program that can decompress
`.tar.bz2` files, you can use [7-Zip][7zip] on Windows or [Keka][] on Mac OS X.

[data-download]: https://spamassassin.apache.org/old/publiccorpus/20021010_easy_ham.tar.bz2
[7zip]: https://www.7-zip.org/
[Keka]: https://www.keka.io/en/

(iteration-review)=
Iteration Review
----------------

In the context of programming, **iteration** means running a block of code
repeatedly. Each repetition is a single iteration. The inputs usually change
from one iteration to the next, so that each iteration carries out a different
computation. Thus it's often useful to think about iteration in terms of
iterating over a collection of inputs, with one iteration per input value.

The primary way to iterate over a collection of values in Python is with a
**for-loop**. If you haven't used for-loops before or need more than a quick
review, see [this introduction to loops][py-basics-loops] in DataLab's Python
Basics Reader.

[py-basics-loops]: https://ucdavisdatalab.github.io/workshop_python_basics/chapters/04_non-tabular-data.html#loops

You can use for-loops with many different types of objects. For example,
for-loops iterate over the elements of a list:

```{code-cell}
for i in [10, 20, 30]:
    print(i)
```

For dictionaries, for-loops iterate over the keys:

```{code-cell}
x = {"a": 10, "b": 20, "c": 30}
for k in x:
    print(k, ":", x[k])
```

For-loops iterate over the keys rather than the values in order to be
consistent with Python's syntax for checking whether a key is present in a
dictionary:

```{code-cell}
"a" in x
```

Every dictionary has a `.values` method to provide access to just the values,
and a `.items` method to provide access to key-value pairs (as tuples).

For-loops also iterate over the characters in a string:

```{code-cell}
for i in "python":
    print(i)
```

These examples raise a question: how are for-loops able to iterate over so many
different types of objects?


### Ranges

You can also use for-loops to iterate over the numbers in a range:

```{code-cell}
x = range(5)
for i in x:
    print(i)
```

A `range` object keeps track of where the range starts and ends, but not the
values in between:

```{code-cell}
x
```

You can be sure of this by checking how much memory Python uses to store a
range compared to an equivalent list. The `sys.getsizeof` function returns the
size of an object in bytes:

```{code-cell}
import sys

sys.getsizeof(3)
```

In Python, an `int` is a fully-fledged object and uses at least 24 bytes to
store its methods, in addition to however many bytes are needed to store its
value.

All ranges use approximately the same amount of memory regardless of where they
start and end:
```{code-cell}
sys.getsizeof(x)
```

```{code-cell}
sys.getsizeof(range(1000))
```

You can use the `list` function to convert a range into a list. A list stores
all of its elements in memory. So for most ranges, the equivalent list uses far
more memory:

```{code-cell}
sys.getsizeof(list(range(1000)))
```

This raises another set of questions: What exactly is a range? When and how
does Python compute the values in a range?


(comprehensions)=
### Comprehensions

A **comprehension** is an efficient and elegant way to write a loop that
produces a value for each item over which it iterates.

For example, suppose you want to get the length of each string in a list. You
could use a for-loop:

```{code-cell}
strings = ["this", "is", "a", "python", "workshop"]

lens = []
for s in strings:
  lens.append(len(s))

lens
```

Alternatively, you can do the same thing more concisely with a **list
comprehension**:

```{code-cell}
lens = [len(s) for s in strings]
lens
```

In some cases, the comprehension is also more efficient.

Comprehensions can optionally include a condition. For example, suppose you
want to skip words that start with `a` when counting lengths:

```{code-cell}
[len(s) for s in strings if not s.startswith("a")]
```

In addition to list comprehensions, Python also supports **dictionary
comprehensions**, which produce a dictionary, and **set comprehensions**, which
produce a set.

DataLab's Python Basics Reader provides [another explanation of
comprehensions][py-basic-comprehensions].

[py-basic-comprehensions]: https://ucdavisdatalab.github.io/workshop_python_basics/chapters/04_non-tabular-data.html#comprehensions


What's a Class?
---------------

In order to understand how Python's iteration works at a lower level, first you
need to know more about classes and objects.

A **class** is a description of the data stored and operations supported by
objects of a specific type. You can think of classes as templates for different
types of objects. Every object is an instance of the class that corresponds to
its type.

You can create your own classes (and thus types) with the `class` keyword. The
keyword must be followed by a name for the class and a colon, which begins a
new block of code. Any variables or functions defined in the block become
**attributes** of objects of the class.

:::{tip}
This section explains how to create classes so that you can understand how
Python works at a lower level. You generally won't need to create classes just
to analyze data or carry out other common data science tasks.

Like functions, classes provide a way to organize and reuse code. The main
difference is that classes define reusable data structures (or types) rather
than reusable operations. For instance, `pandas.Series` and `pandas.DataFrame`
are both classes.

Classes are fundamental to a programming paradigm called **object-oriented
programming**, which is especially useful for developing large, complex
programs. You can learn much more about creating classes and object-oriented
programming by reading [this chapter][think-py-classes] of Think Python 2e.

[think-py-classes]: https://greenteapress.com/thinkpython2/html/thinkpython2016.html
:::

For example, this code creates a new class called `Temperature`, which
represents a temperature measurement with some unit. The attributes are `value`
and `unit`:

```{code-cell}
class Temperature:
    value = 0
    unit = "celsius"
```

You can **construct** or **instantiate** new objects of a specific class by
calling the class:

```{code-cell}
t = Temperature()
```

You can get or set attributes of an object with the `.` operator:

```{code-cell}
t.value
```

```{code-cell}
t.value = 20
t.value
```

### Methods

A **method** is a function attached to an object (as an attribute). When a
method is called, the object itself is implicitly passed as the first argument.
By convention, the parameter name for this argument is `self`.

As an example, here's a new definition of the `Temperature` class with a
`print` method to neatly print out information about the object:

```{code-cell}
class Temperature:
    value = 0
    unit = "celsius"

    def print(self):
        msg =  f"{self.value} degrees {self.unit}"
        print(msg)
```

To test out the method, first you need to create a new `Temperature` object.
Python will *not* automatically update `Temperature` objects you created before
redefining the class. Then call the method with no arguments:

```{code-cell}
t = Temperature()
t.print()
```

Python automatically and implicitly passes `t` as the first argument to the
`path` method.

Methods can have more than one parameter. Any parameters after the first are
assigned values from the arguments in parentheses `( )` when the method is
called, as in a regular Python function.

Here's a new version of the `Temperature` class with a `to_unit` method to
convert the temperature between Celsius and Kelvin:

```{code-cell}
class Temperature:
    value = 0
    unit = "celsius"

    def print(self):
        msg =  f"{self.value} degrees {self.unit}"
        print(msg)

    def to_unit(self, new_unit):
        if new_unit not in ["celsius", "kelvin"]:
            raise ValueError("Invalid unit.")

        if new_unit == self.unit:
            return
        elif new_unit == "kelvin":
            self.value = self.value + 273.15
        else:
            self.value = self.value - 273.15

        self.unit = new_unit
```

Once again, you must first create a new `Temperature` object to test out the
new method:

```{code-cell}
t = Temperature()
t.value = 10
t.print()

t.to_unit("kelvin")
t.print()
```


### Special Methods

The `Temperature` class would better if Python called the `print` method any
time it needed to print a `Temperature` object. Instead, Python prints the type
and memory address of the object:

```{code-cell}
t
```

Fortunately, a class can customize operations such as printing by defining
**special methods**. These methods have specific names which usually begin and
end with two underscores `__`, so they are sometimes also called **double
underscore** or **dunder** methods.

For instance, Python prints an object at the console by calling the `__repr__`
method and printing the string it returns. So here's a new version of the
`Temperature` class where the `print` method is replaced by a `__repr__`
method:

```{code-cell}
class Temperature:
    value = 0
    unit = "celsius"

    def __repr__(self):
        msg = f"{self.value} degrees {self.unit}"
        return msg

    def to_unit(self, new_unit):
        if new_unit not in ["celsius", "kelvin"]:
            raise ValueError("Invalid unit.")

        if new_unit == self.unit:
            return
        elif new_unit == "kelvin":
            self.value = self.value + 273.15
        else:
            self.value = self.value - 273.15

        self.unit = new_unit
```

Now `Temperature` objects print neatly:

```{code-cell}
t = Temperature()
t
```

Python also provides the built-in function `repr` as a standard way to call an
object's `__repr__` method directly:

```{code-cell}
repr(t)
```

A wide variety of behaviors can be customized by defining special methods. A
few examples are:

* `__init__` for object construction, as in `Temperature()`
* `__repr__` for display as a string, as in `repr(t)`
* `__str__` for conversion to a string, as in `str(t)`
* `__lt__`, `__gt__`, `__eq__`, and more for comparisons, as in `t < u`
* `__add__`, `__mul__`, and more for arithmetic, as in `t + u`
* `__getitem__` for indexing, as in `t[i]`
* `__call__` for calls, as in `t()`

You can find a complete list of special methods in [this chapter][py-datamodel]
of the Python Language Reference. In the next section, you'll learn about the
`__iter__` and `__next__` special methods, which are fundamental to how
iteration works in Python.

[py-datamodel]: https://docs.python.org/3/reference/datamodel.html


(whats-an-iterator)=
What's an Iterator?
-------------------

An **iterator** is any object which has a `__next__` method. For-loops iterate
over an iterators by calling this method at the beginning of each iteration to
get the next input. Iteration stops when the method raises a `StopIteration`
exception rather than returning a value.

None of the objects from the examples in {numref}`iteration-review` are
iterators. To check, note that you can use Python's built-in `dir` function to
get a list of an object's attributes. For example, here's code to check for a
`__next__` method on a string:

```{code-cell}
"__next__" in dir("python")
```

Since `__next__` is not present, strings are not iterators. The same is true
for lists, dictionaries, and ranges.

Instead, all of these objects are **iterable**, which means they have an
`__iter__` method. When called, the `__iter__` method returns an iterator over
the object to which it's attached. Thus for-loops call `__iter__` before the
first iteration to get an iterator, and then call `__next__` at the beginning
of each iteration as described above.

Python provides the built-in functions `next` and `iter` to call the `__next__`
and `__iter__` methods manually. For instance, here's the code to manually
iterate over and print the first three items in a list:

```{code-cell}
x = [10, 20, 30]
xiter = iter(x)
i = next(xiter)
print(i)
i = next(xiter)
print(i)
i = next(xiter)
print(i)
```

Calling `next` one more time raises a `StopIteration` exception, which
indicates the iterator has no more values to produce:

```{code-cell}
:tags: [raises-exception]
next(xiter)
```

The code above is effectively the same as this code:

```{code-cell}
for i in x:
    print(i)
```

The `next` and `iter` function are often useful when writing, testing, or
debugging code. For example, they provide an easy way to peek at the first few
elements of objects that don't support indexing (such as TensorFlow data sets).

One way to create a custom iterator is to create a class. For example, here's
an iterator (which is also iterable) that produces the specified number of
elements from an object, cycling back to the beginning if the object is not
long enough:

```{code-cell}
class Cycle:
    def __init__(self, word, n):
        self.word = word
        self.n = n
        self.i = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.n == 0:
            raise StopIteration()

        char = self.word[self.i]
        self.i = (self.i + 1) % len(self.word)
        self.n = self.n - 1
        return char
```

And here's an example of how it can be used:

```{code-cell}
"".join(Cycle("cat", 7))
```

This example also demonstrates that for-loops are not the only way to use
iterable objects. Many functions for aggregation (such as `sum` and `max`) or
instantiating data structures (such as `list` and `dict`) also accept iterable
objects as input.


Working with Iterables
----------------------

Python provides several built-in functions to make it easier to work with
iterable objects.

The `reversed` function reverses an iterable object. For instance:

```{code-cell}
for i in reversed("python"):
    print(i)
```

The `sorted` function sorts the elements of an iterable object. Here's an
example:

```{code-cell}
for i in sorted("python"):
    print(i)
```

The `enumerate` function produces tuples by combining each element of an
iterable with a 0-based index indicating its position. The index is the first
element of each tuple:

```{code-cell}
for index, i in enumerate([10, 20, 30]):
    print(i, "at position", index)
```

The `enumerate` function is especially useful when you need to iterate over the
elements of an object but also need the positions of the elements. An example
application is iterating over a list of data sets and saving them in
sequentially numbered files.

The `zip` function produces tuples by combining two or more iterables. For
instance, suppose you have a list of names and a list of ages (or two data
frame columns):

```{code-cell}
for name, age in zip(["kim", "taylor"], [33, 45]):
    print(name, "is", age)
```

The `zip` function is useful when you need to iterate over two or more
iterables at once. If the iterables have different lengths, the `zip` function
discards all elements beyond the end of the shortest iterable.

The **splat operator** `*`, which converts the elements of a container into
separate arguments, provides a way to invert the `zip` function:

```{code-cell}
zipped = zip([1, 2, 3], [10, 20, 30])
zipped = list(zipped)
zipped
```

```{code-cell}
unzipped = zip(*zipped)
list(unzipped)
```

That is, use `zip(*zipped)` to invert a zip. One application of this pattern is
separating the elements of several identical dictionaries (or records) into
tuples (or lists or columns):

```{code-cell}
records = [{"name": "doris", "age": 13}, {"name": "steve", "age": 81}]
pairs = [r.values() for r in records]
list(zip(*pairs))
```


(the-itertools-module)=
### The `itertools` Module

Python's built-in `itertools` module provides even more functions for working
with iterables. There are functions to combine, repeat, filter, group, and
more.

As an example, recall the `Cycle` class defined in {numref}`whats-an-iterator`,
which produces a given number the elements from an object, cycling back to the
beginning as necessary. You can get the same effect by combining the
`itertools` functions `cycle`, which cycles through the elements of an iterable
endlessly, and `islice`, which slices an iterable. Here's the code:

```{code-cell}
# With Cycle class
x = Cycle("cat", 7)
"".join(x)
```

```{code-cell}
# With itertools
import itertools as it

x = it.islice(it.cycle("cat"), 7)
"".join(x)
```

You can find a complete list of `itertools` functions in [the module's
documentation][itertools].

[itertools]: https://docs.python.org/3/library/itertools.html


What's a Generator?
-------------------

:::{note}
This section and the next section were informed by David Beazley's fantastic
presentation [Generator Tricks for Systems Programmers, v3.0][beazley]. If you
want to learn even more about generators, check out his slides and videos.

[beazley]: https://www.dabeaz.com/generators/
:::

A **generator function** (or **generator**) is a function which contains the
`yield` keyword and therefore produces or *generates* a sequence of values. You
can think of generators as a concise way to create custom iterators. Moreover,
generators are **lazy**, which means values in the sequence are only computed
as needed. These properties make generators ideal for creating pipelines that
process big data.

Python evaluates generator functions differently from regular functions. When
you call a generator function, Python returns a **generator iterator** (or
**generator**), but doesn't evaluate the code in the function. When you call
the iterator's `__next__` method, Python evaluates the function up to the first
`yield` and then pauses. The next time you call the `__next__` method,
evaluation resumes and then pauses at the next yield.

As an example, here's a generator which counts down from `n - 1` to 0:

```{code-cell}
def countdown(n):
    while n > 0:
        n = n - 1
        yield n
```

Calling the function returns a generator iterator:

```{code-cell}
ct = countdown(3)
ct
```

```{code-cell}
next(ct)
```

```{code-cell}
next(ct)
```

```{code-cell}
next(ct)
```

Just like an iterator, when the generator runs out of values, it raises a
`StopIteration` exception:

```{code-cell}
:tags: [raises-exception]
next(ct)
```

As another example, generators provide a third way to create an iterator which
produces a given number the elements from an object, cycling back to the
beginning as necessary. {numref}`whats-an-iterator` showed how to create this
iterator as a class, and {numref}`the-itertools-module` showed how to create
this iterator using functions from the `itertools` module. Here's the code to
create it as a generator:

```{code-cell}
def cycle(word, n):
    i = 0
    while n > 0:
        yield word[i]
        i = (i + 1) % len(word)
        n = n - 1
```

```{code-cell}
x = cycle("cat", 7)
"".join(x)
```

Generators generate values one at a time and only on demand. As a result,
generators typically use less CPU time and far less memory than doing
equivalent computations with a list. The drawback is that a generator is a
one-time operation. If you want to iterate over the generated values more than
once, you must either store them as they're produced the first time, or
recompute them by calling the generator function (and iterating) again.

You can use the `yield from` keyword to create a generator from another
generator (or iterator). For instance, here's a generator that counts down from
`n - 1` to 0 twice:

```{code-cell}
def double_countdown(n):
    yield from countdown(n)
    yield from countdown(n)
```

And here's what the output from this generator looks like:

```{code-cell}
for i in double_countdown(2):
    print(i)
```


### Generator Expressions

Another way to create generators is by writing **generator expressions**. The
syntax for a generator expression is almost the same as for a list
comprehension, but enclosed in parentheses `( )` instead of square brackets `[
]`.

For example, here's a generator expression version of the list comprehension
from {numref}`comprehensions`:

```{code-cell}
strings = ["this", "is", "a", "python", "workshop"]

lens = (len(s) for s in strings if not s.startswith("a"))
lens
```

Generator expressions have all of the same properties as generators, but are
more concise. So in the example, the lengths are not computed until they are
explicitly requested:

```{code-cell}
list(lens)
```


Data Processing Pipelines
-------------------------

This section provides a concrete example of how you can use generators to solve
a data processing problem. You'll need the Easy Ham Emails data set linked in
{numref}`prerequisites`. The data set contains a collection of email messages,
each stored in a separate file. The data processing goal is to create a
pipeline to extract all email addresses from the emails.

The first step is to define a generator to open and yield the files:

```{code-cell}
def gen_open(paths):
    for path in paths:
      yield open(path, "rt", encoding = "latin-1")
```

The `open` function returns a generator over the lines in the opened file. All
of the lines in all of the files need to be processed the same way, so the next
step is define a generator that combines the lines from all of the files into a
single sequence:

```{code-cell}
def gen_cat(files):
    for f in files:
        yield from f
```

The last step is to actually extract the emails from each line. One way to do
this is to split each line into terms, and then filter out any terms that don't
contain `@` somewhere in the middle (since these can't be email addresses):

```{code-cell}
import re

def gen_emails(lines):
    for line in lines:
        for term in re.split("[^a-zA-Z@.]", line):
            if "@" in term.strip("@"):
                yield term
```

Now that the generators are defined, putting together the pipeline is just a
sequence of calls:

```{code-cell}
from pathlib import Path

paths = Path("data/easy_ham").glob("*")
paths = list(paths)
files = gen_open(paths)
lines = gen_cat(files)
at_lines = (l for l in lines if "@" in l)
emails = gen_emails(at_lines)
```

The email addresses are only computed on request. One way to request all of
them is to generate a list:

```{code-cell}
emails = list(emails)
emails[:20]
```

You can convert the list to a set to get the total number of unique email
addresses:

```{code-cell}
len(set(emails))
```

The advantage of this approach is that it scales to any number of email
messages. Moreover, it's easy to swap out or reuse the components of the
pipeline. For instance, the `gen_open` and `gen_cat` generator functions are
essentially the same ones David Beazley uses for different tasks in his
presentation [Generator Tricks for Systems Programmers, v3.0][beazley]. The
`gen_emails` generator function could easily be replaced with a different
function to extract other information, such as phone numbers. Moreover,
additional generator functions could be added to the pipeline to further
process the results, such as extracting domain names.

The disadvantages of this approach are that it can be difficult to understand
if you're not used to programming with generators, and it can be slightly
harder to debug problems in the pipeline than it would be to debug a loop.
