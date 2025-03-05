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

(chapter-indexing)=
Slicing & Dicing Data with pandas
=================================

:::{admonition} Learning Objectives
* Explain what pandas indexes are and how they're used
* Get, modify, and set indexes on series and data frames
* Explain and use pandas' index alignment
* Describe and use the 4 major modes of indexing in pandas
* Explain the difference between `.loc` and `.at`
* Identify problems where using `.set_index` to make a column an index is
  helpful
* Explain and use pandas' query mini-language
:::

This chapter is a deep dive into how **indexing**&mdash;the process of getting
or setting elements of a data structure&mdash;works in the [pandas][] package.
Indexing is sometimes also called **subsetting** or (element) extraction, and
is a fundamental operation when using a programming language to solve problems
in data science.

[pandas]: https://pandas.pydata.org/


Prerequisites
-------------

This chapter assumes you already have basic familiarity with Python and pandas.
In particular, you should be comfortable indexing Python lists and dictionaries
with the square bracket operator `[ ]`, and should be able to explain what a
`Series` and `DataFrame` are and how the they differ. DataLab's [Python Basics
Reader][py-basics] and its accompanying workshop provide a suitable
introduction to these topics.

[py-basics]: https://ucdavisdatalab.github.io/workshop_python_basics/

To follow along, you'll need the following software versions (or newer)
installed on your computer:

* [Python][] 3.10
* [NumPy][] 1.22
* [pandas][] 1.4

One way to install these is to install the [Anaconda][] Python distribution.
Chapter 2 provides more details about Anaconda and the `conda` package manager.

[Python]: https://www.python.org/
[NumPy]: https://numpy.org/
[Anaconda]: https://www.anaconda.com/

[CLICK HERE][data-download] to get the data set used in the examples for this
chapter. After clicking the link, you'll need to right-click on the page and
select "Save Page As...". Make sure to save the page somewhere you can easily
access from your Python session.

[data-download]: https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-03-29/sports.csv


What's an Index?
----------------

In pandas, an `Index` is a Python object that stores a sequence of labels or
names. The labels usually correspond to the elements of a data structure along
a given dimension or **axis**. Indexes are a key feature that differentiates
pandas from other software for programming with tabular data (such as R).

According to the pandas documentation, indexes serve three important roles:

1. As metadata to provide additional context about a data set
2. As a way to explicitly and automatically align data, avoiding bugs due to
   misalignment (see {numref}`alignment`)
3. As a convenience for getting and setting subsets of the data (see
   {numref}`column-indexes`)


### Indexes on Series

As an example of where pandas uses indexes, `Series` are 1-dimensional and
always have exactly one index. You can see the index just to the left of the
elements when you print out a series:

```{code-cell}
import pandas as pd

x = pd.Series([4, 5, 3, 1])
x
```

You can get or set the index on a series with the `.index` attribute:

```{code-cell}
x.index
```

The labels in an index can be numbers, strings, dates, or other values. Pandas
provides subclasses of `Index` for specific purposes, including:

* `RangeIndex`, where the labels are a monotonic sequence of integers with a
  fixed beginning, end, and step size
* `CategoricalIndex`, where the labels are categories (and not necessarily
  unique)
* `IntervalIndex`, where the labels are non-overlapping intervals of the real
  number line
* `DateTimeIndex`, where the labels are dates and times
* `PeriodIndex`, where each label is a date and time span

You may also encounter `Int64Index`, `UInt64Index`, and `Float64Index`, which
represent an arbitrary list of labels of the respective type. For example:

```{code-cell}
x[[1, 3]].index
```

Beginning in pandas 2.0, these three subclasses (but not subclasses in the
bulleted list above) will be replaced with a generic `Index` class. See [this
warning][index-deprecated] in the pandas documentation for details.

[index-deprecated]: https://pandas.pydata.org/docs/user_guide/advanced.html#int64index-and-rangeindex

You can access the labels in an index with standard Python indexing operations:

```{code-cell}
idx = x.index
# Get the 2nd label
idx[1]
```

Indexes are **immutable**, which means the labels in an index can't be changed.
For instance:

```{code-cell}
:tags: [raises-exception]
idx[0] = 5
```

Although labels in an index can't be changed, every index also has a name,
which can be changed. Think of the name as a name for the dimension or axis to
which the index corresponds. You can get or set the name of an index with the
`.name` attribute (on the index itself):

```{code-cell}
idx.name
```

The default value of `.name` is `None`, which means the index doesn't have a
name. When pandas prints a series or data frame, it will also print the names
of any attached indexes. For example:

```{code-cell}
x.index.name = "labels"
x
```

Pandas automatically creates indexes whenever you create (or load) a series or
data frame. Pandas also provides functions to construct indexes manually. These
are usually named after the type of index. For instance, you can create an
ordinary `Index` from a list or NumPy array:

```{code-cell}
new_idx = pd.Index(["a", "b", "c", "d"])
new_idx
```

You can use assignment to replace the index on a series. For example:
```{code-cell}
x.index = new_idx
x
```

Note that the new index must have the same length as the index it replaces (and
the series itself).

:::{admonition} Checkpoint Exercise
:class: important

Replace the index on `x` with one like the original (in variable `idx`), but
where the first label is 5 instead of 0. Try to do this *without* typing out a
list of all 4 labels.

Hint: the Python `list` function can convert many types of objects into lists,
including pandas indexes.
:::


### Indexes on Data Frames

Pandas `DataFrame`s are 2-dimensional and always have exactly two indexes. The
indexes correspond to the rows and the columns, respectively. To see this, use
pandas to load the data from the `sports.csv` file:

```{code-cell}
dtype = {"classification_other": str}
sports = pd.read_csv("data/sports.csv", dtype = dtype)
sports.head()
```

This data set contains information about funding and staffing of athletics
programs at U.S. universities from 2015 to 2019 (inclusive). The source for the
data set is the U.S. Department of Education's [Equity in Athletics Data
Analysis project][EADA], and the data was cleaned by the [Tidy Tuesday][tidy]
community. [This article][usafacts] describes some of the conclusions that can
be drawn from the data set.

[EADA]: https://ope.ed.gov/athletics/#/datafile/list
[tidy]: https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-03-29
[usafacts]: https://usafacts.org/articles/coronavirus-college-football-profit-sec-acc-pac-12-big-ten-millions-fall-2020/

On data frames, you can use the `.index` attribute to get and set the *row*
index. For example:

```{code-cell}
sports.index
```

The default index created by the `read_csv` function counts from 0 to the
number of rows.

The other index on data frames corresponds to the columns. You can use the
`.columns` attribute to get and set the column index. For instance:

```{code-cell}
sports.columns
```

The indexes stored in the `.index` and `.columns` attributes of a data frame
have the same classes and basic properties as indexes on series.

(alignment)=
### Alignment

For arithmetic operations that involve more than one series or data frame,
pandas automatically **aligns** the elements based on the indexes. As an
example, consider these two series:

```{code-cell}
u = pd.Series([1, 2, 3], index = ["a", "b", "c"])
v = pd.Series([1, 2, 3], index = ["c", "b", "a"])
```

The elements of the series are identical, but the indexes differ. If you're an
experienced NumPy (or R) user, the result of adding these two series together
might surprise you:

```{code-cell}
u + v
```

This is the result because pandas aligns the elements, adding element `a` from
`u` to element `a` from `v`, element `b` to element `b`, and so on. 

Sometimes you might want to do arithmetic on series or data frames without any
alignment. The canonical solution in this case is to use the `.to_numpy` method
to convert one or both of the objects into a NumPy array:

```{code-cell}
u + v.to_numpy()
```

Pandas tries to align elements and carry out arithmetic operations even for
objects with mismatched shapes and labels. Pandas fills elements for which the
result is unknown with missing values. For example:

```{code-cell}
w = pd.Series([-10, 0], index = ["z", "a"])

u - w
```

In this example, only element `a` has two operands; for each other element, the
result is a missing value because only one operand is known.

:::{admonition} Checkpoint Exercise
:class: important

What happens if you use an arithmetic operation on two series or data frames
where one has repeated/duplicated labels in its index?

Hint: construct an example and see what happens!
:::

Pandas does *not* align series and data frames in comparison operations, and
raises an error if they're not already aligned:

```{code-cell}
:tags: [raises-exception]
u > v
```

See [this GitHub issue][compare-align] for a developer discussion of whether
this is a helpful feature for preventing bugs or a nuisance.

[compare-align]: https://github.com/pandas-dev/pandas/issues/1134

Finally, the `align` method manually aligns two series or data frames. The
method returns two objects of the same type, but with elements sorted so that
the indexes match. For example:

```{code-cell}
u.align(v)
```

For objects with mismatched shapes, the `align` method inserts missing values
to make their shapes and indexes match:

```{code-cell}
u.align(w)
```

The `align` method provides a way to align to objects so that they can be
compared:

```{code-cell}
ua, va = u.align(v)
ua > va
```

Moreover, concatenating the two results from the `align` method is the same as
performing a [join][]. The `align` method even provides a parameter `join` to
select the type of join (`"outer"`, `"left"`, `"right"`, or `"inner"`). For
relational series and data frames, you can break a join into smaller steps by
using a combination of the `align` function , column indexing, and the pandas
`concat` function.

[join]: https://pandas.pydata.org/docs/user_guide/merging.html#database-style-dataframe-or-named-series-joining-merging


Indexing Operators
------------------

Pandas provides several different operators for indexing series and data
frames, as well as several different ways to use each. The primary indexing
operator is the square bracket `[ ]`.

For a `Series`, the square bracket operator *usually* selects elements by
label. For instance, integer arguments are treated as labels, not positions, if
the labels in the index are integers:

```{code-cell}
x = pd.Series([1, 2, 3], index = [1, 0, 3])
x[0]
```

You can use a list to select multiple elements at once or repeat elements:

```{code-cell}
x[[0, 3, 0]]
```

You can also use the square bracket operator to select elements by label when
the labels are strings:

```{code-cell}
y = pd.Series([10, 20, 30], index = ["a", "b", "c"])
y["a"]
```

In this case, passing an integer argument selects an element by position:

```{code-cell}
y[0]
```

Using the square bracket operator this way in non-interactive code is risky,
because it can cause bugs if some of the series or data frames your code
operates on unexpectedly have integer labels.

The dot `.` serves as a secondary indexing operator for series and data frames
with string indexes. For instance, to get the element `b`:

```{code-cell}
y.b
```

Since the dot `.` is also used to access attributes and methods, it cannot be
used to access elements with a label that's the same as the name of an
attribute or method. When in doubt, it's safer to use `[ ]` to access elements
by label.


### Slices

A **slice** selects a range of elements. The syntax is `start:stop:step`, with
the second colon `:` and arguments optional, the same as for Python's built-in
slicing. The default start value is the beginning of the series, the default
stop value is one past the end, and the default step value is 1.

Slices can be by position or by label. With `[ ]`, when the start or stop value
is an integer, pandas assumes you want to slice by position:

```{code-cell}
x[1:3]
```

In a slice by position, the element at the stop position is not included.

Slicing with only the step can be useful in a variety of situations, such as
getting every second element:

```{code-cell}
x[::2]
```

Negative values in a slice are counted backward from the end of the object. For
example, this code gets the *last* two elements of the series:

```{code-cell}
x[-2:]
```

:::{admonition} Checkpoint Exercise
:class: important

What happens if you use a negative step value in a slice?
:::


When the start or stop value is a string, pandas slices by label:

```{code-cell}
y["b":]
```

Be aware that when you slice by label, the element at the stop position is
always included:

```{code-cell}
y["b":"c"]
```

This is a case where the pandas developers decided convenience outweighs
consistency. See [this section][endpoints] of the pandas documentation for more
details.

[endpoints]: https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html#advanced-endpoints-are-inclusive


:::{admonition} Checkpoint Exercise
:class: important

What happens if you include an integer step value in a slice by label?
:::


### By Position with `.iloc`

When you want to index a series or data frame by position, use `[ ]` on the
`.iloc` attribute. Then any integer arguments are assumed to be positions,
regardless of the type of index on the object.

For example:

```{code-cell}
x.iloc[0]
```

```{code-cell}
x.iloc[[1, 0, 2]]
```

```{code-cell}
x.iloc[0:]
```

You can use this attribute regardless of the index type:

```{code-cell}
y.iloc[0:]
```

As a result, the `.iloc` attribute is a more reliable way to index by position
than using `[ ]` alone.

Pandas also provides an `.iat` attribute to get or set scalar values in a series
or data frame by position:

```{code-cell}
x.iat[0]
```

Compared to `.iloc`, the `.iat` attribute is typically much faster (in terms of
CPU time), so it's important to use `.iat` in performance-critical sections of
code. In addition, `.iat` raises an error if your code attempts to get or set
more than one value, which can potentially alert you to bugs that would go
undetected with `.iloc`.


### By Label with `.loc`

When you want to index a series or data frame by label, use `[ ]` on the
`.loc` attribute. Then any integer arguments are assumed to be labels.

For example:

```{code-cell}
x.loc[0]
```

```{code-cell}
x.loc[0]
```

```{code-cell}
x.loc[[0, 1]]
```

```{code-cell}
y.loc["b"]
```

```{code-cell}
y.loc[["b", "a", "b"]]
```

Pandas also provides an `.at` attribute to get or set scalar values in a series
or data frame by label:

```{code-cell}
x.at[0]
```

The advantages of `.at` compared to `.loc` are the same as `.iat` compared to
`.iloc`: better run-time speed and stricter requirements on the argument.

When setting elements by label with `[ ]`, `.loc`, or `.at`, if you use a
label that's not already in the series index, pandas will append the label and
value. For instance:

```{code-cell}
x.loc[10] = -1
x
```

This doesn't work when setting elements by position.


### By a Condition

The square bracket operator `[ ]`, the `.iloc` attribute, and the `.loc`
attribute all support indexing a series by a condition. The condition must be
represented by a Boolean array with the same length as the series.

You can use comparison operators to get a suitable Boolean array. For example:

```{code-cell}
is_positive = x > 0
is_positive
```

```{code-cell}
x[is_positive]
```

```{code-cell}
x.loc[is_positive]
```

If the condition has an index, the square bracket operator and `.loc` operator
will try to align the condition with the series being indexed:

```{code-cell}
z = x.iloc[[1, 0, 3]]
z
```

```{code-cell}
z[is_positive]
```

The `.iloc` operator raises an error if the Boolean array has an index:

```{code-cell}
:tags: [raises-exception]
x.iloc[is_positive]
```

As described in {numref}`alignment`, the canonical way to avoid index alignment
is to use the `.to_numpy` method to convert to a NumPy array:

```{code-cell}
x.iloc[is_positive.to_numpy()]
```

The logic operators for inverting and combining conditions in pandas differ
from Python's usual logic operators:

* `|` is logical or
* `&` is logical and
* `~` is logical not

For example:

```{code-cell}
x[is_positive & (x % 2 == 1)]
```

In compound conditions, each sub-condition must be enclosed in parentheses
`()`, because in Python's [order of operations][oo], the `|`, `&`, and `~`
operators are evaluated before comparison operators such as `==`, `>`, and `<`.

[oo]: https://docs.python.org/3/reference/expressions.html#operator-precedence


### By a Callable

The square bracket operator `[ ]`, the `.iloc` attribute, and the `.loc`
attribute also all support indexing a series by a function (or **callable**
object defined with `def` or `lambda`. The function must accept one argument:
the series itself. The function must return a valid argument for the indexing
operator (positions, labels, or a Boolean array).

The primary use case for this form of indexing is chaining together multiple
operations without defining intermediate variables. For example, this code adds
each element at an even-numbered position to the next element, and then gets
only the results greater than 2:

```{code-cell}
(x.iloc[::2] + x.iloc[1::2].to_numpy())[lambda a: a > 2]
```

As you can probably see, chaining operations makes code relatively difficult to
understand. Whenever possible, it's better to use simple indexing strategies
rather than indexing by a callable.

:::{admonition} Checkpoint Exercise
:class: important

Use indexing by a callable to set the elements of `x` with even values (not
positions) to twice their current value.

What's a simpler way to do this using other indexing modes?
:::


### Data Frames

The indexing operators `[ ]`, `.iloc`, and `.loc` can all be used with data
frames in addition to series. Since data frames are 2-dimensional, you can pass
the indexing operators a separate argument for each dimension. The rows come
first.

For instance, to get (1st row, 2nd and 3rd columns) of the athletics data frame
the code is:

```{code-cell}
sports.iloc[0, [1, 2]]
```

When the result has a lower dimension than the object being indexed, pandas
automatically converts to an appropriate data type. In this case, the result is
converted to a series with the names of the two columns as its index.

As another example, the code to get (rows labeled 10 and 11, column labeled
`"year"`) is:

```{code-cell}
sports.loc[[10, 11], "year"]
```

With `.iloc` and `.loc`, you can select all of the elements along a given
dimension by passing a colon `:` as the argument for that dimension. For
instance, this code gets all rows in the columns labeled `"year"` and
`"institution_name"`:

```{code-cell}
sports.loc[:, ["year", "institution_name"]]
```

You can mix indexing by condition with indexing by position or label. The most
common way to do this is to use a condition to select rows and labels to select
columns. For example:

```{code-cell}
sports.loc[sports.year == 2018, ["year", "institution_name", "sports"]]
```

:::{admonition} Checkpoint Exercise
:class: important

Use indexing to get only the rows where `year` is `2016` and `institution_name`
is `"University of California-Davis"`. Limit the columns to `year`,
`institution_name`, `sports`, and `total_rev_menwomen` (the total revenue from
the program in USD).
:::


Indexing a data frame with only one argument generally accesses rows. For
example:

```{code-cell}
sports.iloc[0]
```

As an exception, using the square bracket operator `[ ]` to index a data frame
with a string argument (or list of strings) accesses columns:

```{code-cell}
sports[["year", "institution_name"]]
```


(column-indexes)=
Columns as Indexes
------------------

An important feature of pandas is that you can use arbitrary columns in a data
frame as indexes. Generally the columns should contain identifiers for the rows
or other categorical data. For example, in the `sports` data frame, the column
`institution_name` is an identifier.

You can use the `.set_index` method to set a column as an index:

```{code-cell}
sports = sports.set_index("institution_name")
sports.head()
```

After setting a categorical column as an index, you can use indexing by label
as a convenient way to get rows in specific categories:

```{code-cell}
sports.loc["University of California-Davis"]
```

You can use the `.reset_index` method to restore the `institution_name` column
to the data frame and reset the index to a range:

```{code-cell}
sports = sports.reset_index()
sports.head()
```

Setting indexes appropriately makes it much more convenient to filter the rows
of a data frame.


### MultiIndexes

One of pandas' most powerful features is the ability to set multiple columns as
the index at the same time. This creates a `MultiIndex`, a hierarchical index
with multiple levels. For example, this code creates a MultiIndex from `year`,
`institution_name` and `sports`:

```{code-cell}
sports = sports.set_index(["year", "institution_name", "sports"])
```

The MultiIndex makes it easy to get all of the information for a specific year,
institution, and sport. When working with a MultiIndex, the corresponding
argument to the indexing operator should be a tuple with one element for each
level of the MultiIndex:

```{code-cell}
sports.loc[(2015, "Stanford University", "Football")]
```

You can use the `.xs` (read: "cross section") method to select rows from only
one particular level of a MultiIndex. For instance, to get all rows for Harvard
university:

```{code-cell}
sports.xs("Harvard University", level = 1)
```

The level argument is `1` because `institution_name` is the 2nd level of the
MultiIndex, and indexing starts from 0. You can also use the name of the level
instead:

```{code-cell}
sports.xs("Princeton University", level = "institution_name")
```

The [pandas documentation on MultiIndexes][multiindex-docs] describes a wide
variety of ways MultiIndexes can be used to slice and dice data frames.

[multiindex-docs]: https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html

:::{admonition} Checkpoint Exercise
:class: important

Use a MultiIndex to get a series that lists the sports for which there were
university athletics programs in Aberdeen, WA in 2018.
:::


The `.query` Method
-----------------

The `.query` method is another powerful way to extract information from a data
frame. The idea of `.query` is that you write a query string using a query
mini-language to select a subset of the rows. This has several advantages over
other ways of indexing:

* For large data sets, the `.query` method is [slightly faster][query-perf]
  than other ways of indexing
* You don't have to repeat the name of the data frame every time you refer to a
  column in a condition
* You can reuse a single query across multiple data frames, provided they all
  have the queried columns
* Conditions are easier to read:
    + You can use `not`, `or` and `and` to invert and combine conditions
      instead of `~`, `|`, and `&`
    * You can use `in` and `not in` in conditions
    * You can write `a < b < c` rather than `(a < b) & (b < c)`
    + It's not necessary to enclose sub-conditions in parentheses, since the
      query mini-language defines its own order of operations

[query-perf]: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#performance-of-query

Here's an example of a query:

```{code-cell}
sports = sports.reset_index()

qs = "city_txt == 'Davis' and state_cd == 'CA' and sports == 'Basketball'"
ucd_bball = sports.query(qs)
ucd_bball[["institution_name", "sports", "year", "total_rev_menwomen"]]
```


:::{admonition} Checkpoint Exercise
:class: important

Use a query string to get a data frame that lists the sports for which there
were university athletics programs in Aberdeen, WA in 2018.
:::

The pandas documentation provides [further details about how to write query
strings][query-strings].

[query-strings]: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#the-query-method
