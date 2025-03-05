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

(chapter-debugging)=
Squashing Bugs with Python's Debugging Tools
============================================

:::{admonition} Learning Objectives
* Explain the difference between syntax errors and exceptions
* Identify Python's common built-in exceptions
* Explain and use a try-catch statement
* Raise exceptions in code
* Explain the principles of defensive programming
* Use Python's built-in `logging` module to log output
* Explain what assertions are and how to use them
* Explain what unit tests are
* Explain what debugging is and what the `pdb` module is
* Use the `pdb` debugger to navigate through running code
* Explain what `ipdb` is and how to debug with IPython & Jupyter
* Explain and use IPython magic commands
* Explain what profiling is
:::

This chapter describes how to prevent, detect, and diagnose bugs in your Python
code. This includes an introduction to Python's exceptions system, a discussion
of defensive programming best practices, and an introduction to Python's
debugging tools. The chapter also describes some of the tools you can use to
measure and improve the performance of your code.


Prerequisites
-------------

This chapter assumes you already have basic familiarity with Python. DataLab's
[Python Basics Reader][py-basics] and its accompanying workshop provide a
suitable introduction.

[py-basics]: https://ucdavisdatalab.github.io/workshop_python_basics/

To follow along, you'll need the following software versions (or newer)
installed on your computer:

* [Python][] 3.10
* [ipdb][] 0.13.9

One way to install these is to install the [Anaconda][] Python distribution.
Chapter 2 provides more details about Anaconda and the `conda` package manager.

[Python]: https://www.python.org/
[Anaconda]: https://www.anaconda.com/
[ipdb]: https://github.com/gotcha/ipdb


Errors
------

When you write and run Python code, there are two different kinds of errors you
might have to deal with: syntax errors and exceptions. A **syntax error** is a
grammatical mistake in how you wrote the code, and prevents Python from running
the code. For instance, this code to construct a list has a common syntax
error&mdash;a comma `,` is missing:

```{code-cell}
:tags: [raises-exception]
x = [1, 2, 3 4]
```

Syntax errors are the simplest kind of error to resolve, because they're
usually caused by typos or a misunderstanding of the Python syntax (which you
can clarify by reviewing the documentation). If you try to run code that
contains a syntax error, Python will report the line and character where it
first detects the error.

In contrast, an **exception** is an error that occurs while your code is
running. Exceptions are usually caused by something that violates the
assumptions of the code&mdash;such as an argument with an inappropriate type or
value, a file that's in the wrong place, or an unreliable network connection.
As an example, adding an integer and a string raises an exception:

```{code-cell}
:tags: [raises-exception]
3 + "hello"
```

Many different types of exceptions are built into Python, and you can also
create your own. The code above raises a `TypeError`, an exception that means
one or more arguments have an inappropriate type.

Another kind of exception, a `NameError`, is raised if your code tries to
access a variable that doesn't exist:

```{code-cell}
:tags: [raises-exception]
aggie
```

Here are several common built-in exceptions:

Name             | Cause
---------------- | -----------
`AttributeError` | Attribute is not found or read-only
`IndexError`     | Index to sequence is out of range
`KeyError`       | Key to dictionary is not found
`NameError`      | Variable is not found
`StopIteration`  | Iterator or generator is out of items
`TypeError`      | Object has inappropriate type for an operation
`ValueError`     | Object has inappropriate value for an operation

You can find a complete list in [the Built-in Exceptions chapter][exceptions]
of the Python documentation.

[exceptions]: https://docs.python.org/3/library/exceptions.html


### Handling Exceptions

You can make your code more robust by handling exceptions that are likely to
occur or have a straightforward solution. You can use a **try-except
statement** to indicate that some code might raise an exception and specify
what should happen if it does.

As an example, consider this code to prompt the user for a number:

```
x = input("Specify a number: ")
x = float(x)
```

If the user provides an input that's not a number, the call to `float` raises a
`ValueError`:

```{code-cell}
:tags: [remove-cell]
x = "hi"
```

```{code-cell}
:tags: [remove-input, raises-exception]
x = float(x)
```

Suppose instead you'd like to use 0.0 as a default if the input isn't a number.
One way you can do this is with a try-except statement:

```
x = input("Specify a number: ")
try:
    x = float(x)
except:
    print(f"'{x}' is not a valid number, using 0 instead.")
    x = 0
```

Put code that might raise an exception in the `try` block. Code in the `except`
block only runs if an exception is actually raised.

You can specify which types of exceptions an `except` block should catch by
adding the type after the keyword:

```
x = input("Specify a number: ")
try:
    x = float(x)
except ValueError:
    print(f"'{x}' is not a valid number, using 0 instead.")
    x = 0
```

You can optionally give the exception a name as well, so that you can refer to
it in the `except` block:

```
x = input("Specify a number: ")
try:
    x = float(x)
except ValueError as e:
    print(f"'{x}' is not a valid number, using 0 instead.")
    print(f"Original error: {e}")
    x = 0
```

Specifying the type of exception serves two purposes:

1. It ensures that your code only catches exceptions it can handle.
2. It allows you to handle different types of exceptions in different ways.

The second is possible because you can have more than one `except` block in a
try-except statement.

A try-except statement can also have an `else` block. Code in the `else` block
only runs if an exception is *not* raised. So continuing the example, you could
combine a try-except statement with a while-loop to prompt the user for a
number until they enter a valid one:

```
while True:
    x = input("Specify a number: ")
    try:
        x = float(x)
    except ValueError:
        print(f"'{x}' is not a valid number.")
    else:
        # No exception, so break out of the loop.
        break
```

Finally, you can use a `finally` block in a try-except statement to specify
code that should *always* run, whether on not an exception is raised. This is
mainly useful for cleanup operations, such as closing connections to files or
network devices.



### Raising Exceptions

You can use the `raise` keyword to make your code raise an exception. For
instance, suppose you define a function to trim a specified number of elements
from each end of a sequence. The function should raise an error if the number
of elements to trim is greater than the length of the sequence:

```{code-cell}
def trim_elements(x, n):
    if len(x) < 2*n:
        raise ValueError("Too many elements to trim.")
    return x[n:-n]
```

In this case, the function raises a `ValueError` because the value of `n` is
the problem. The argument to the exception is the error message. You can use
`raise` to raise any type of exception, including user-defined exceptions.

You can find further examples of how to handle, raise, and define new types of
exceptions in the [Errors and Exceptions chapter][errors-and-exceptions] of the
Python documentation.

[errors-and-exceptions]: https://docs.python.org/3/tutorial/errors.html


Defensive Programming
---------------------

**Defensive programming** means taking steps to prevent or quickly detect and
fix bugs when writing code. Defensive programming techniques can help you
minimize time spent debugging and the number of unexpected bugs at run-time.
Some examples of how you can program defensively include:

* Format your code neatly. Use a consistent naming scheme and indentation
  style.

* Organize your code into logical steps or "paragraphs" with blank lines in
  between. This makes it convenient to review what happens in each step, and to
  convert steps into functions when appropriate.

* Add comments to your code:
    + To create a big picture plan for what to write.
    + To explain tricky code.
    + To summarize the purpose of a "paragraph" of code.

* Convert steps that you plan to use more than once into short functions.
  Think about what the expected inputs and outputs are for each function, and
  make a note of these in the docstring.

* Develop code to handle the simple cases first. Don't try to handle more
  complicated cases until you've verified that the code works for the simple
  cases.

* Test code frequently as you write it. Alternate between writing code and
  running the code to confirm it works as expected. You should have a Python
  session running to test things out almost any time you're writing code.

* Investigate unexpected behavior such as warning messages even if the code
  seems to produce correct results. Ideally, either fix the code so the
  unexpected behavior doesn't occur, or document why the behavior occurs and
  why it doesn't pose a problem.

* Follow reproducibility best practices, including those laid out in
  {numref}`chapter-reproducible`.


### Logging

**Logging** means saving the printed output of your code to a file. Logging
makes it easier to detect bugs because you can record information about the
state of your code as it runs, and because you can compare logs across
different runs.

Python's built-in `logging` module provides basic logging functionality. To set
up logging, import the module and call `logging.basicConfig`:

```{code-cell}
import logging

logging.basicConfig(filename = "logs/my_script.log", level = logging.DEBUG)

# If you also want logged messages to print to the console:
logging.getLogger().addHandler(logging.StreamHandler())
```

The `basicConfig` function has many keyword parameters that you can use to
customize how messages are logged. You can find a complete list in [this
section][basicConfig] of the Python documentation.

[basicConfig]: https://docs.python.org/3/library/logging.html#logging.basicConfig

The `filename` parameter makes the logger save log messages to a file. If you
don't set this parameter, they'll be printed to the Python console instead.

The `level` parameter controls what kinds of messages are logged. Messages
below the specified level are not logged. The logging levels are, from highest
to lowest:

Name       | Description
---------- | -----------
`CRITICAL` | Messages about errors that prevent the code from running
`ERROR`    | Messages about non-critical errors
`WARNING`  | Messages about unexpected events that don't interfere with normal operation
`INFO`     | Informational messages
`DEBUG`    | All messages (this level is usually used only for debugging)

Once you've set up logging, you can write something to the log at a specific
level using the `debug`, `info`, `warning`, `error`, `exception`, or `critical`
functions in the `logging` module.

For instance, to log a warning and an informational message:
```{code-cell}
logging.warning("This is a warning.")
logging.info("This is information.")
```

You can use these logging functions as replacements for the `print` function.
Use debug or messages to record detailed information about your code as it
runs. When you're sure the code works as intended, you can disable these
messages by changing the level in the call to `basicConfig`.

For a much more detailed introduction to the `logging` module, see the [Logging
HOWTO][logging-howto] in the Python documentation.

[logging-howto]: https://docs.python.org/3/howto/logging.html

Also see [the `loguru` package][loguru], which provides a variety of
improvements and additional features compared to the `logging` module.

[loguru]: https://github.com/Delgan/loguru


### Assertions

An **assertion** is an expression in code that checks whether some condition is
true and raises an exception if it isn't. In other words, an assertion asserts
that some assumption must be satisfied for the code to continue running.

You can use the `assert` keyword and a condition to create an assertion in
Python:

```{code-cell}
x = 1.1
x = x ** 2
assert x > 1
```

When an assertion fails, Python raises an `AssertionError`:

```{code-cell}
:tags: [raises-exception]
assert x < 0
```

Assertions provide one way to confirm that objects in your code are as you
expect. If you suspect some code might not work correctly, but you're
completely not sure, adding an assertion is a good way to check your
suspicions.

The major advantage of using assertions rather than if-statements (and `print`
or `raise`) to confirm assumptions is that once you've finished testing your
code, you can run Python with the `-O` command line argument and it will ignore
all assertions.

As a consequence, you should think of assertions as a tool for testing code
while it's in development. If you want your code to check an assumption even
after development is finished, use if-statements and `raise` exceptions
instead. For example, if you create a package and want the functions in the
package to validate their inputs, they should use if-statements and exceptions.


### Tests

As already mentioned, testing code is an important step in the development
process, and one you should do early and often. When developing single-use
scripts and notebooks (such as data analyses), it's usually sufficient to adopt
an ad-hoc testing strategy, where you test code by manually inspecting the
outputs (and possibly some intermediate values).

For projects where the code will be developed by many people, used by many
people, or used many times, such as a package, it's often a good idea to adopt
a more formal and automated testing strategy.

**Unit testing** is a testing strategy where isolated components or "units" of
code are tested. The units are typically functions or modules, and the tests
are typically automated to report correct/incorrect results from each unit.
Testing units in isolation makes it easier to assess where bugs originate.
Using automated tests makes it convenient to run the tests every time a change
is made to the code. Unit testing is a software development best practice, and
some software developers even advocate [using tests to motivate or "drive"
development][tdd].

[tdd]: https://en.wikipedia.org/wiki/Test-driven_development

Python's built-in `unittest` provides functions for creating and running unit
tests. [The `pytest` package][pytest] provides a simpler way to create unit
tests based on `assert` statements. Many other unit testing packages are also
available; the [Python Testing Tools Taxonomy][testing-taxonomy] has an
exhaustive list.

[pytest]: https://docs.pytest.org/
[testing-taxonomy]: https://wiki.python.org/moin/PythonTestingToolsTaxonomy


Debugging
---------

**Debugging** is the process of confirming, step-by-step, that what you believe
the code does is what the code actually does, in order to identify the causes
of bugs.

The key idea is to examine the output from each step in the code. There are two
ways to go about this:

1. Work forward through the code from the beginning.
2. Work backward from the source of an error (if the bug causes an error).

A **debugger** is an interactive tool that enables running code one expression
at a time. In most debuggers, you can also inspect and change the values of
objects. Some debuggers even allow evaluation of arbitrary code.

Python's built-in `pdb` module provides a debugger. You can launch the debugger
from code by calling the `pdb.set_trace` function. Python 3.7 and newer also
provide a `breakpoint` function to launch the debugger. Alternatively, you can
run the debugger on a specific function by calling the `pdb.runcall` function.

When you launch the debugger, you'll be presented with a prompt similar to the
Python prompt. Several special commands are available in the debugger. Here are
some of the most useful ones:

| Command    | Shortcut | Description
| ---------- | -------- | -----------
| `help`     | `h`      | print help for debugger commands
| `quit`     | `q`      | halt execution and quit the debugger
| `where`    | `w`      | print a stack trace
|            | `p`      | print an expression
|            | `pp`     | pretty-print an expression
| `step`     | `s`      | continue execution until the next possible stopping point
| `next`     | `n`      | continue execution until the next line
| `until`    | `unt`    | continue execution until a given line
| `continue` | `c`      | continue execution until the next breakpoint
| `break`    | `b`      | print or set breakpoints
| `up`       | `u`      | move up the stack trace
| `down`     | `d`      | move down the stack trace

Several of Python's built-in functions are also useful when debugging:

| Function  | Description
| --------- | -----------
| `dir`     | Get a list of all names in the current scope
| `locals`  | Get a dictionary of all local variables in the current scope
| `globals` | Get a dictionary of all global variables in the current scope


### In IPython & Jupyter

In IPython and Jupyter, the `pdb` module doesn't always work correctly, and the
`breakpoint` function is currently ignored (although [this will likely be fixed
in a future version][breakpoint-issue]).

[breakpoint-issue]: https://github.com/ipython/ipykernel/issues/897

Instead, you can use the `ipdb` package as a drop-in replacement for the `pdb`
module. It provides the `ipdb.set_trace` function, and the `ipdb` debugger uses
the same set of commands as the `pdb` debugger. The `ipdb` debugger also has a
few additional features familiar from IPython, such as syntax highlighting and
tab completion.

In IPython, a **magic command** is a command you can enter at the console
that's not part of the Python language. Magic commands usually begin with a
percent `%` to distinguish them from ordinary code. Other Jupyter kernels (for
languages other than Python) can also support magic commands, but this is up to
the kernel developers. You can learn more about IPython magic commands in [this
chapter][magic] of the IPython documentation.

[magic]: https://ipython.readthedocs.io/en/stable/interactive/magics.html

IPython provides a `%debug` magic command you can use to immediately run the
debugger at the last line that caused an error. It also provides a `%pdb` magic
command to make running `%debug` the default any time an exception is raised
(just run `%pdb on` at the beginning of your session).



Profiling
---------

**Profiling** is the process of measuring how much CPU time or memory each
expression in your code uses. Profiling is usually used to improve the
performance of code rather than find bugs. It's mentioned in this chapter
because it's another kind of diagnostic you can run on code.

The simplest form of profiling is **benchmarking**, where you measure the time
it takes for some code to run. You can use Python's built-in `timeit` module to
estimate how long an expression takes to run on average. For example, suppose
you want to construct a list of numbers as strings. Here are two different ways
to do it:

```
[str(i) for i in range(1000)]

list(map(str, range(1000)))
```

It's not obvious which is faster. If you want to find out, you can use the
`timeit.timeit` function:

```{code-cell}
from timeit import timeit

timeit(lambda: [str(i) for i in range(1000)], number = 100)
```

```{code-cell}
timeit(lambda: list(map(str, range(1000))), number = 100)
```

The function returns the average time in seconds over `number` runs. In this
case, the `map` function seems to be marginally faster (although to be thorough
we should examine multiple list lengths).

You might want more detailed profile information than just the benchmark for a
single expression or function. For instance, you might want to know how long
every expression in a script takes, to identify speed bottlenecks, or you might
want information about memory usage instead of timing.

Python has two built-in profilers: `cProfile` and `profile`. Unless you want to
build new functionality into the profiler, generally `cProfile` is the better
choice. You can learn more about how to use these profilers in [the Python
Profilers chapter][profilers] of the Python documentation.

[profilers]: https://docs.python.org/3/library/profile.html

Other CPU time profilers for Python include:

* [`py-spy`](https://github.com/benfred/py-spy)
* [`pyinstrument`](https://github.com/joerick/pyinstrument)
* [`line_profiler`](https://github.com/pyutils/line_profiler)

There's no built-in memory profiler for Python, but the community has developed
several:

* [`memory_profiler`](https://github.com/pythonprofilers/memory_profiler)
* [`pympler`](https://github.com/pympler/pympler)
* [`objgraph`](https://github.com/mgedmin/objgraph)
* [`filprofiler`](https://github.com/pythonspeed/filprofiler)

