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

# Date & String Processing

:::{admonition} Learning Goals
After this lesson, you should be able to:

* Explain why we use special data structures for dates & times
* Identify the correct data structure for a given date/time
* Use Polars and the `datetime` module to parse dates
* Use the date format string mini-language
* Use escape codes in strings to represent non-keyboard characters
* Explain what a text encoding is
* Use Polars  to detect, extract, and change patterns in strings
* Use the regular expressions mini-language
:::

This chapter is part 1 (of 2) of _Cleaning & Reshaping Data_, a workshop series
about how to prepare data for analysis. The major topics of this chapter are
how to convert dates and times into appropriate data types and how to extract
and clean up data from strings (including numbers with non-numeric characters
such as `$`, `%`, and `,`).


## Parsing Dates & Times

Given a data set with dates or times, you might want to:

* Sort the observations chronologically
* Compute the times between observations
* Add or subtract a time offset
* Get just the month, year, hour, etc. of each observation

You can do all of these things and more with Polars, but only if your dates and
times are represented by appropriate data types. Many popular file formats for
data, such as CSV, *do not* store metadata about which values are dates and
times, so these values get typed as strings. Dates and times extracted from
text are also naturally strings.

The first step to working with dates and times typed as strings is to **parse**
them: break them down into their components and cast them to more suitable data
types. To demonstrate parsing dates, consider a series of date strings:

```{code-cell}
import polars as pl

date_strings = pl.Series(["Jan 10, 2021", "Sep 3, 2018", "Feb 28, 1982"])
date_strings
```

We can use Polars to parse the strings into dates. For series (including
columns), string methods are listed under the `.str` namespace, and the
`.str.to_date` method parses strings as dates. In some cases, the method can
infer the format of the dates automatically, but that's not the case for our
dates:

```{code-cell}
:tags: ["raises-exception"]
date_strings.str.to_date()
```

We need to supply the method with a **format string** that describes the format
of the dates. In a format string, a percent sign `%` followed by a character is
called a specifier and represents a component of a date or time. Here are some
useful specifiers:

| Specifier | Description      | 2015-01-29 21:32:55
|:--------- |:---------------- |:-------------------
| `%Y`      | 4-digit year     | 2015
| `%m`      | 2-digit month    | 01
| `%d`      | 2-digit day      | 29
| `%H`      | 2-digit hour     | 21
| `%M`      | 2-digit minute   | 32
| `%S`      | 2-digit second   | 55
| `%%`      | literal %        | %
| `%y`      | 2-digit year     | 15
| `%B`      | full month name  | January
| `%b`      | short month name | Jan

:::{note}
Most programming languages use format strings to parse and format dates and
times. There isn't a standard, but the idea seems to have originated with the
`strptime` and `strftime` functions in the C programming language. Polars uses
[the same specifiers as Rust's `chrono` crate][pl-strptime].

[pl-strptime]: https://docs.rs/chrono/latest/chrono/format/strftime/index.html
:::

Our date strings have a short month name (`%b`), a 2-digit day (`%d`), and a
4-digit year (`%Y`). The format string must match the format of the dates
exactly, so we also need to include whitespace and punctuation. Thus the format
string is `%b %d, %Y`. Use this to parse the dates:

```{code-cell}
dates = date_strings.str.to_date("%b %d, %Y")
dates
```

Polars represents dates with the `Date` type, which you can see by inspecting
the series' `.dtype` attribute:

```{code-cell}
dates.dtype
```

Polars can also parse times and datetimes (which combine a date and a time).
The respective methods are `.str.to_time` and `.str.to_datetime`. Let's try
parsing an unusual time format:

```{code-cell}
time_string = pl.Series(["6 minutes, 32 seconds after 10 o'clock"])
time = time_string.str.to_time("%M minutes, %S seconds after %H o'clock")
time
```

Since this is a time, the data type is `Time`:

```{code-cell}
time.dtype
```

Polars is compatible with Python's built-in `datetime` module, which provides
similar data types and functions for working with dates and times. The
equivalents of Polars' `Date`, `Time`, and `Datetime` types are Python's
`date`, `time`, and `datetime` types. In fact, Polars returns these types if
you access the elements of a series individually. For example:

```{code-cell}
type(time[0])
```

The `datetime` module is also the recommended way to create literal date and
time values for Polars. For instance, you can use the `dt.datetime` function to
explicitly create datetimes for a series:

```{code-cell}
import datetime as dt

events = pl.Series([
    dt.datetime(year = 2023, month = 1, day = 10, hour = 8, minute = 3),
    dt.datetime(2002, 8, 16, 14, 59)
])
events
```

The module also provides `dt.date` and `dt.time` functions to create dates and
times.

:::{caution}
The similarly-named Polars functions `pl.date`, `pl.time`, and `pl.datetime` do
not create dates and times. Make sure to use the functions from the `datetime`
module!
:::

### Extracting Components

Once you have dates or times, you might want to extract components from them.
For instance, suppose we want to get the years from the dates we parsed
earlier. For series (including columns), date and time methods are listed under
the `.dt` namespace. There are methods for a variety of different date and time
components. For instance, the `.dt.year` method gets the year:

```{code-cell}
dates.dt.year()
```

Similarly, the `.dt.month` method gets the month:

```{code-cell}
dates.dt.month()
```

The Polars documentation provides [a complete list][pl-dt] of `.dt` methods.

[pl-dt]: https://docs.pola.rs/api/python/stable/reference/series/temporal.html

:::{note}
Python's date and time types (from the `datetime` module) expose their
components as attributes rather than methods, and don't use the `.dt` prefix.

For example, you can get the year of a `date` called `x` with `x.year`. No
parentheses `()` are necessary, since `.year` is an attribute.

See [the documentation for the `datetime` module][datetime] for more details.

[datetime]: https://docs.python.org/3/library/datetime.html
:::


### Durations

Occasionally, you might need to adjust dates or times by adding an offset. For
example, suppose we want to add 30 days to the dates from earlier. The
`dt.timedelta` function creates a `timedelta` object, which represents a fixed
period of time. So to add 30 days:

```{code-cell}
dates + dt.timedelta(days = 30)
```

What if we want to add 1 month to the dates instead of 30 days? The length of a
month varies, so we can't use a `timedelta`. Polars provides a solution: the
`.dt.offset_by` method. The method takes a string argument that specifies the
period of time by which to offset. The string for 1 month is `"1m"`, so:

```{code-cell}
dates.dt.offset_by("1m")
```

This increments each date by exactly 1 month, regardless of how many days that
is. The documentation for `.dt.offset_by` describes [how to specify other
periods of time][offset_by]. 

[offset_by]: https://docs.pola.rs/api/python/stable/reference/series/api/polars.Series.dt.offset_by.html


### Case Study: CA Parks & Recreation Fleet

The government of California publishes data about its fleet of vehicles on the
[California Open Data portal][data.ca.gov]. As of March 2025, the data set
includes all non-confidential vehicles owned by agencies from 2015-2023. We'll
use a subset of this data to compute how many vehicles the CA Department of
Parks and Recreation purchased each year from 2019 to 2023. The data set is
published as a messy CSV, so we'll need to do some cleaning and parse the dates
in order to use it.

[data.ca.gov]: https://data.ca.gov/

:::{important}
[Click here][ca-parks-fleet] to download the CA Parks & Recreation
Fleet data set.

[ca-parks-fleet]: https://ucdavis.box.com/s/6j40xd44miz7x6qlvgga9eieu7ya6xo3

If you haven't already, we recommend you create a directory for this workshop.
In your workshop directory, create a `data/` subdirectory. Download and save
the data set in the `data/` subdirectory.
:::

:::{admonition} Documentation for the CA Parks & Recreation Fleet Data Set
:class: note, dropdown
Each row in the data set contains measurements from one vehicle-year
combination. 

[Click here][ca-fleet-docs] to download the documentation for the columns.

[ca-fleet-docs]: https://ucdavis.box.com/s/v6agya4830tz1lkgq8toej5q1b9j5ntf

This data set is a subset of the much larger [CA State Fleet data
set][ca-fleet].

[ca-fleet]: https://data.ca.gov/dataset/california-state-fleet
:::

To get started, read the data set from wherever you saved it:

```{code-cell}
fleet = pl.read_csv("data/2015-2023_ca_parks_fleet.csv")
fleet.head()
```

Since we want to compute the number of acquisitions each year, we'll focus on
two columns: `acquisition_method` and `acquisition_delivery_date`. The
`acquisition_method` column indicates whether the vehicle was purchased,
donated, transferred from a different agency, or something else. We're only
interested in purchases, so we can filter the others out:

```{code-cell}
fleet = fleet.filter(pl.col("acquisition_method") == "Purchase")
```

In order to determine how many vehicles were purchased each year, we need to
extract the year from `acquisition_delivery_date`. Let's take a look at this
column:

```{code-cell}
fleet["acquisition_delivery_date"].head()
```

It looks like the dates are in month-day-year format. The format string for
this is `"%m/%d/%Y"`, so:

```{code-cell}
:tags: ["raises-exception"]

fleet.select(
    pl.col("acquisition_delivery_date").str.to_date("%m/%d/%Y")
)
```

Uh-oh. According to the error message from Polars, some of the date values
don't follow the month-day-year format. Instead, they're numbers like `44832`,
`44462`, and `43451`. These numbers probably seem inscrutable, but there is an
explanation for them: Microsoft Excel, perhaps the most popular tool for
working with tabular data, stores dates by counting days from 31 December 1899.
So 1 is 1 January 1900, 32 is 1 February 1900, and so on. Unfortunately, the
Excel developers incorrectly assumed 1900 was a leap year, so all of the counts
were off by 1 after 28 February 1900. Most modern Excel-compatible spreadsheet
programs fix this by counting from 30 December 1899 rather than 31 December, so
that only dates before 28 February have different numbers. We can convert the
numbers in the `acquisition_delivery_date` column into dates by treating them
as offsets to 30 December 1899.

Transforming a series in two different ways at once is not (yet) an easy task
with Polars, but it is possible. We'll do it by splitting the series into two
series: one for the month-day-year dates and one for the Excel dates. The
former are easy to identify because they contain slashes (`/`). We can use the
`.str.contains` method, which tests whether a string contains a character, to
test for slashes. So a condition for the month-day-year dates of the
`acquisition_delivery_date` column is:

```{code-cell}
is_mdy = pl.col("acquisition_delivery_date").str.contains("/")
```

We can use the condition with the `pl.when` function to get the month-day-year
dates and the Excel dates. The `pl.when` function returns elements of a series
specified in `.then` when its condition is true and returns `null` otherwise.
So to create a new data frame `dates` with columns `mdy` and `excel` for the
month-day-year and Excel dates, respectively:

```{code-cell}
dates = fleet.select(
    mdy = pl.when(is_mdy).then(pl.col("acquisition_delivery_date")),
    excel = pl.when(~is_mdy).then(pl.col("acquisition_delivery_date"))
)
dates.head()
```

We can convert the `mdy` column to dates with `.str.to_date` and the
`"%m/%d/%Y"` format string from earlier.

To convert the `excel` column to dates, we have to start with the literal date
30 December 1899 and offset it by the value of the column. We can use `dt.date`
to create the date and `.dt.offset_by` to offset it. The code to transform both
the `mdy` and `excel` columns is:

```{code-cell}
start_date = dt.date(1899, 12, 30)

dates = dates.with_columns(
    mdy = pl.col("mdy").str.to_date("%m/%d/%Y"),
    excel = pl.lit(start_date).dt.offset_by(pl.col("excel") + "d")
)
dates.head()
```

We can combine the two columns by replacing all of the `null` values in `mdy`
with values from `excel`. This becomes the new `acquisition_delivery_date`
column:

```{code-cell}
fleet = fleet.with_columns(
    acquisition_delivery_date = dates["mdy"].fill_null(dates["excel"])
)
fleet.head()
```

With the dates in hand, we can compute the number of purchases from 2019 to
2023. Since each row in the data set represents a vehicle-year, and we just
want to count vehicles, we must first get a subset of unique vehicles. We can
do this by grouping on the `equipment_number` column, which is a unique
identifier for each vehicle:

```{code-cell}
vehicles = fleet.group_by("equipment_number").first()
```

We can compute the number of purchases by extracting the years with the
`.dt.year` method and then counting them with the `.value_counts` method:

```{code-cell}
year = vehicles["acquisition_delivery_date"].dt.year().alias("year")
year.value_counts().sort("year").tail()
```

It's unclear whether the data set contains complete records for 2023, so the
count for that year might be too small. It's also likely that the agency
purchases vehicles months or years in advance of their delivery, so these
counts are more accurately described as deliveries rather than purchases.


## String Fundamentals

Strings represent text, but even if your data sets are composed entirely of
numbers, you'll need to know how to work with strings. Text formats for data
are widespread: comma-separated values (CSV), tab-separated values (TSV),
JavaScript object notation (JSON), a panopoly of markup languages (HTML, XML,
YAML, TOML), and more. When you read data in these formats, sometimes Python
will correctly convert the values to appropriate non-string types. The rest of
the time, you need to know how to work with strings so that you can fix
whatever went wrong and convert the data yourself.

This section introduces fundamental concepts for computing on strings. The next
section, {numref}`processing-strings`, describes how to find, extract, and
replace patterns in strings.


(escape-sequences)=
### Escape Sequences

In a string, an **escape sequence** or **escape code** consists of a backslash
followed by one or more characters. Escape sequences make it possible to:

1. Write quotes or backslashes within a string
2. Write characters that don't appear on your keyboard (for example, characters
   in a foreign language)

For example, the escape sequence `\n` corresponds to the newline character.
Notice that the `print` function prints `\n` as a literal new line:

```{code-cell}
x = "Hello\nNick"
print(x)
```

On the other hand, when you let Python print a string or other object
automatically (without explicitly calling `print`), it prints a
programmer-friendly representation. For strings, the representation shows
escape codes:

```{code-cell}
x
```

This makes it easy to identify characters that are not normally visible, such
as whitespace, and also makes it easy to copy the string. You can use the
built-in `repr` function to get a representation for any object.

:::{important}
Make sure to call `print` explicitly if you want to see what a string actually
looks like.
:::

As another example, suppose we want to put literal quotation marks in a string.
We can either enclose the string in the other kind of quotation marks (single
versus double), or escape the quotation marks in the string:

```{code-cell}
x = 'She said, "Hi"'
print(x)

y = "She said, \"Hi\""
print(y)
```

Since escape sequences begin with backslash, we also need to use an escape
sequence to write a literal backslash. The escape sequence for a literal
backslash is two backslashes:

```{code-cell}
x = "\\"
print(x)
```

The Python documentation provides a [complete list of escape sequences][esc].
Other programming languages also use escape sequences, and many of them are the
same.

[esc]: https://docs.python.org/3/reference/lexical_analysis.html#escape-sequences

:::{tip}
You can mark a string as **raw** by prefixing it with `r` or `R` (lowercase `r`
is more common) to make Python ignore escape sequences. For example,
`r"backslashes \ are okay"` is a raw string.

Raw strings are especially useful for writing regular expressions
({numref}`processing-strings`).

A quirk of raw strings is that they can't end with an odd number of
backslashes, since the next character after a backslash is treated as part of
the string.
:::


### Character Encodings

Computers store data as numbers. In order to store text on a computer, people
have to agree on a **character encoding**, a system for mapping characters to
numbers. For example, in [ASCII](https://en.wikipedia.org/wiki/ASCII), one of
the most popular encodings in the United States, the character `a` maps to the
number 97.

Many different character encodings exist, and sharing text used to be an
inconvenient process of asking or trying to guess the correct encoding. This
was so inconvenient that in the 1980s, software engineers around the world
united to create the [Unicode](https://home.unicode.org/) standard. Unicode
[includes symbols](http://unicode.org/charts/) for nearly all languages in use
today, as well as emoji and many ancient languages (such as Egyptian
hieroglyphs).

Unicode maps characters to numbers, but unlike a character encoding, it
doesn't dictate how those numbers should be mapped to bytes (sequences of ones
and zeroes). As a result, there are several different character encodings that
support and are synonymous with Unicode. The most popular of these is UTF-8.

Python 3 uses Unicode by default. You can write Unicode characters with the
escape sequence `\u` (or `\U`) followed by a 4-digit (or 8-digit) number for
the character in [base 16][]. For instance, the number for `a` in Unicode is 97
(the same as in ASCII). In base 16, 97 is `61`. So you can write an `a` as:

```{code-cell}
"\u0061"
```

Unicode escape sequences are usually only used for characters that are not easy
to type. For example, the cat emoji is number `1f408` (in base 16) in Unicode.
So `"\U0001f408"` is the cat emoji:

```{code-cell}
"\U0001f408"
```

[base 16]: https://en.wikipedia.org/wiki/Hexadecimal

:::{tip}
Whether you can see a Unicode character depends on whether the current font has
a glyph (image representation) for that character. Most fonts only have glyphs
for a few languages.

Make sure to use a font with good Unicode coverage if you love emoji or expect
to work with many different languages. The [Noto Fonts][] project aims to
create a collection of fonts with a common style and complete language
coverage. The [NerdFonts][] project patches fonts commonly used for programming
so that they have better coverage of symbols. 

[Noto Font]: https://notofonts.github.io/
[NerdFonts]: https://www.nerdfonts.com/
:::


:::{note}
If you ever read or write a text file (including CSV and other formats) and the
text [looks like gibberish][mojibake], it might be an encoding problem. This is
especially true on Windows, the only modern operating system that does not
(yet) use UTF-8 as the default encoding.

Encoding problems when reading a file can usually be fixed by passing the
encoding to the function doing the reading. For instance, the code to read an
ISO-8859-1 encoded CSV file with Polars is:

```python
pl.read_csv("my_data.csv", encoding = "iso-8859-1")
```

Other reader functions might have a different parameter for the encoding, so
always check the documentation.

[mojibake]: https://en.wikipedia.org/wiki/Mojibake
:::


(processing-strings)=
## Processing Strings

String processing encompasses a variety of tasks such as searching for patterns
within strings, extracting data from within strings, splitting strings into
component parts, and removing or replacing unwanted characters (excess
whitespace, punctuation, and so on). If you work with data, sooner or later
you'll run into a data set in text format that needs a few corrections, and for
that you'll find familiarity with string processing invaluable.


### Strings in Polars

We recommend using Polars for most string processing, although Python's
built-in `str` data type and `re` module are also an excellent option. Polars
is slightly more convenient and concise for handling many strings at once (in a
data frame or series). It is also more efficient in some cases.

:::{admonition} See Also
:class: note
The Polars user guide has [a section about strings][pl-str].

[pl-str]: https://docs.pola.rs/user-guide/expressions/strings/
:::

For series (including columns), string methods are listed under the `.str`
namespace. For example, the `.str.contains` method tests whether a pattern
appears within a string. The method returns `True` if the pattern is found and
`False` if it isn't. Try it out on a sample series:

```{code-cell}
words = pl.Series(["help", "kelp", "grow", "tall"])
words.str.contains("el")
```

```{code-cell}
words.str.contains("g")
```

As another example, the `.str.slice` method extracts a substring from a string,
given the substring's position and length. So to extract 9 characters beginning
at position 5:

```{code-cell}
quotes = pl.Series([
    "It's dangerous to go alone!", "Tombs with piped in music."
])

quotes.str.slice(5, 9)
```

The `.str.slice` method is especially useful for extracting data from strings
that have a fixed width.

There are a lot of methods in the `.str` namespace, but five that are
especially important (which we will use) are:

* `.str.contains`, to test whether a string contains a pattern
* `.str.slice`, to extract a substring at a given position
* `.str.replace`, to replace or remove part of a string
* `.str.split`, to split a string into parts
* `.str.extract_groups` to extract parts of a string that match a pattern

You can find [a complete list of methods][pl-str-ref] with examples in the
Polars API reference.

[pl-str-ref]: https://docs.pola.rs/api/python/stable/reference/expressions/string.html


### Regular Expressions

Many of Polars' `.str` methods use a special language called **regular
expressions** or **regex** to describe patterns in strings. Python's `re`
module and many other programming languages also have string processing tools
that use regular expressions, so fluency with regular expressions is a
valuable, transferable skill.

You can use a regular expression to describe a complicated pattern in just a
few characters because some characters, called **metacharacters**, have special
meanings. Metacharacters are usually punctuation characters. They are *never*
letters or numbers, which always have their literal meaning.

This table lists some of the most useful metacharacters:

:::{list-table}
:header-rows: 1

* - Metacharacter
  - Meaning
* - `.`
  - any one character (wildcard)
* - ``\``
  - escape character (in both Python and regex), see {numref}`escape-sequences`
* - `^`
  - the beginning of string (not a character)
* - `$`
  - the end of string (not a character)
* - `[ab]`
  - one character, either `'a'` or `'b'`
* - `[^ab]`
  - one character, anything except `'a'` or `'b'`
* - `?`
  - the previous character appears 0 or 1 times
* - `*`
  - the previous character appears 0 or more times
* - `+`
  - the previous character appears 1 or more times
* - `()`
  - make a group
* - `|`
  - match left OR right side (not a character)
:::

{numref}`regular-expression-examples` provides examples of how most of the
metacharacters work.


:::{admonition} See Also
:class: note
You can find even more examples and a complete listing of regex metacharacters
in [the documentation for the `re` module][re] and [the documentation for
Rust's regex crate][regex]. The former is more approachable if you're not
comfortable reading Rust code, but the latter is what Polars actually uses.
Differences between the two are minor.

[re]: https://docs.python.org/3/library/re.html
[regex]: https://docs.rs/regex/latest/regex/
:::

For Polars methods that support regex, you can disable them by setting the
`literal` parameter to `True`. For example, to test whether a string contains a
literal dot `.`:

```{code-cell}
x = pl.Series(["No dot", "Lotsa dots..."])
x.str.contains(".", literal = True)
```

:::{tip}
Make it a habit to set `literal = True` anytime you use a pattern that doesn't
contain regex metacharacters. Doing so communicates to the reader that you're
not using regex, prevents bugs, and makes your code more efficient.
:::


### Replacing Parts of Strings

Replacing part of a string is a common string processing task. For instance,
quantitative data often contain non-numeric characters such as commas, currency
symbols, and percent signs. These must be removed before converting to numeric
data types. Replacement and removal go hand-in-hand, since removal is
equivalent to replacing part of a string with the **empty string** `""`.

The `.str.replace` method replaces the *first* part of a string that matches a
pattern (from left to right), while the related `.str.replace_all` method
replaces *every* part of a string that matches a pattern. Several Polars `.str`
methods that do pattern matching come in a pair like this: one to process only
the first match and one to process every match.

As an example, suppose you want to remove commas from a series of numbers (in
strings) so that you can cast them to a numeric data type. You want to remove
*all* of the commas, so `.str.replace_all` is the method to use. The first
argument is the pattern and the second argument is the replacement. In this
case, the pattern is `","` and the replacement is the empty string `""`. So the
code to remove the commas and cast the numbers is:

```{code-cell}
x = pl.Series(["1,000,000", "525,600", "42"])
x.str.replace_all(",", "", literal = True).cast(pl.Int64)
```

The `.str.replace` method wouldn't work as well for this task, since it only
replaces the first match to the pattern:

```{code-cell}
x.str.replace(",", "", literal = True)
```

You can also use these methods to replace or remove longer patterns within
words. For instance, suppose you want to change the word `"dog"` to `"cat"`:

```{code-cell}
x = pl.Series(["dogs are great, dogs are fun", "dogs are fluffy"])
x.str.replace("dog", "cat", literal = True)
```

```{code-cell}
x.str.replace_all("dog", "cat", literal = True)
```

As a final example, you can use the replacement methods and a regex pattern to
replace repeated spaces with a single space. This is a good standardization
step if you're working with text. The key is to use the regex quantifier `+`,
which means a character "repeats one or more times" in the pattern, and to use
a single space `" "` as the replacement:

```{code-cell}
x = pl.Series(["This    sentence  has  extra      space."])
x.str.replace_all(" +", " ")
```

If you just want to remove all whitespace (or other characters) from the
beginning and end of a string, you can use the `.str.strip_chars` method
instead.


### Splitting Strings

Distinct data in a text are generally separated by a character like a space or
a comma, to make them easy for people to read. Often these separators also make
the data easy to parse with Polars. The idea is to split the string into a
separate value at each separator.

The `.str.split` method splits a string at each match to a pattern. The
matching characters---that is, the separators---are discarded.

:::{important}
The `.str.split` method and Polars' other methods for splitting strings don't
support regular expressions yet (but this is a planned feature).
:::

For example, suppose you want to split some sentences into words:

```{code-cell}
sentences = pl.Series([
    "The wonderful, wonderful cat!",
    "Who is this duke of zill anyway?"
])

words = sentences.str.split(" ")
words
```

The `.str.split` method returns the splits for each string in a list. You can
use the `.list.get` method to get all of the splits at a particular position:

```{code-cell}
words.list.get(3)
```

For lists with different lengths, you can set `null_on_oob = True` in
`.list.get` to get splits that are beyond the end of the shortest list. The
method will return `null` for any list shorter than the requested position. For
example:

```{code-cell}
words.list.get(5, null_on_oob = True)
```

:::{note}
The `.list` namespace has other methods for working with series of lists as
well.
:::

:::{tip} When you know exactly how many splits you expect a string to have, use
the `.str.splitn` method instead of `.str.split`. It requires a second argument
for the maximum number of splits. It returns the results in structs
(fixed-length data structures) rather than lists. You can expand the structs
into columns with the `.struct.unnest` method.

<!--
For example:

```{code-cell}
x = pl.Series(["1, 2, 3", "10, 20, 30"])
x.str.splitn(", ", 3).struct.unnest()
```
-->

Specifying the number of splits can help prevent bugs in your code and make
splitting more efficient. In some cases it's also more convenient to work with
a data frame (from chaining `.struct.unnest`) than a series of lists.

<!--
For example, suppose you want to get the area codes from some phone numbers:

```{code-cell}
phones = pl.Series(["717-555-3421", "629-555-8902", "903-555-6781"])
result = phones.str.splitn("-", 3).struct.unnest()

result[:, 0]
```
-->
:::


### Extracting Matches

Occasionally, you might need to extract parts of a string in a more complicated
way than string splitting allows. One solution is to write a regular expression
that will match all of the data you want to capture, with parentheses `( )`,
the regex metacharacter for a group, around each distinct value. Then you can
use the `.str.extract_groups` method to extract the groups. {numref}`groups`
presents some examples of regex groups.

For example, suppose you want to split an email address into three parts: the
user name, the domain name, and the [top-level domain][tld]. To create a
regular expression that matches email addresses, you can use the `@` and `.` in
the address as anchors. The surrounding characters are generally alphanumeric,
which you can represent with the "word" metacharacter `\w`:

```text
\w+@\w+[.]\w+
```

Next, put parentheses `( )` around each part that you want to extract:

```text
(\w+)@(\w+)[.](\w+)
```

Finally, use this pattern in `.str.extract_groups`:

```{code-cell}
x = pl.Series(["datalab@ucdavis.edu"])

# Note this is a raw string:
pattern = r"(\w+)@(\w+)[.](\w+)"

result = x.str.extract_groups(pattern)
result
```

The method returns a series of structs (fixed-length data structures) with one
field for each group. You can use the `.struct.unnest` method to expand the
result into a data frame with a column for each group:

```{code-cell}
result.struct.unnest()
```

:::{note}
The `.struct` namespace has other methods for working with series of structs as
well.
:::

The pattern in this example doesn't work for all possible email addresses,
since user names can contain dots and other characters that are not
alphanumeric. You could generalize the pattern if necessary. The point is that
the `.str.extract_groups` method and groups provide an extremely flexible way
to extract data from strings.


(regular-expression-examples)=
## Regular Expression Examples

:::{important}
This section is intended as a reference and is not taught in the workshop.
:::
