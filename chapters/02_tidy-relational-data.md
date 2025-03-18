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

# Untidy & Relational Data


:::{admonition} Learning Goals
:class: note
After this lesson, you should be able to:

* Explain what it means for data to be tidy
* Use Polars to reshape data
* Explain what a relational data set is
* Use Polars to join data based on common columns
* Describe the different types of joins
* Identify which types of joins to use when faced with a relational data set
:::

This chapter is part 2 (of 2) of _Cleaning & Reshaping Data_, a workshop series
about how to prepare data for analysis. The major topics of this chapter are
how to reshape untidy data sets with pivots and how to combine related data
sets with joins.


## Reshaping Untidy Data

The structure of a data set---its shape and organization---has enormous
influence on how difficult it will be to analyze, so making structural changes
is an important part of the cleaning process. Researchers conventionally
arrange tabular data so that each row contains a single **observation** or
case, and each column contains a single kind of measurement or identifier,
called a **feature**.

:::{important}
What constitutes an observation depends on what questions you want to answer
with the data. Sometimes you might need to work with a data set at several
different levels of observational unit to answer all of your questions. We'll
see an example of this soon.
:::

In 2014, [Hadley Wickham][hadley] refined and formalized the conventions for
tabular data by [introducing the concept of **tidy data**][tidy]. Paraphrasing
Wickham, the rules for tidy data are:

[hadley]: https://hadley.nz/
[tidy]: https://vita.had.co.nz/papers/tidy-data.html

> 1. Every column is a single feature.
> 2. Every row is a single observation.
> 3. Every cell is a single value.

These rules ensure that all of the values in a data set are visually organized
and are easy to access with indexing operations. They're also specific enough
to make tidiness a convenient standard for functions that operate on tabular
data, so many packages for Python and other programming languages are designed
from the ground up for working with tidy data. 

This section explains how to **reshape** untidy data into tidy data. While
reshaping can seem tricky at first, making sure your data set has the right
structure before you begin analysis saves time and frustration in the long run.

### An Untidy Data Set

The City of Davis has two bike counters: one is at the intersection of 3rd
Street and University Avenue (the 3rd Street bike obelisk) and the other is at
the intersection of Loyola Drive and Pole Line Road. The City publishes data
from the bike counters online. DataLab combined the City's 2020 bike counts,
aggregated to the day level, with precipitation and wind data from the U.S.
National Oceanic and Atmospheric Administration's weather station at Sacramento
Metropolitan Airport (this was the nearest weather station with complete
records for 2020). We'll use this data set to demonstrate how to transform
untidy data.

:::{important}
[Click here][davis-bikes] to download the 2020 Davis bike counts data set.

[davis-bikes]: https://ucdavis.box.com/s/q9oox9arov97dvsocrsx6gghh7283z3i

If you haven't already, we recommend you create a directory for this workshop.
In your workshop directory, create a `data/` subdirectory. Download and save
the data set in the `data/` subdirectory.
:::

::::{admonition} Documentation for the 2020 Davis Bike Counts Data Set
:class: note, dropdown
Each row in the data set contains measurements from one date-variable
combination.

:::{list-table}
:header-rows: 1

* - Column
  - Description
* - `date`
  - The date of measurement
* - `variable`
  - What was measured: `third` and `loyola` are bike counts, `prcp` is total
    precipitation in millimeters, `awnd` is average daily wind speed in meters
    per second
* - `value`
  - The measured value
:::

The source for the bike counts is the City of Davis' [Bike and Pedestrian
Statistics][bike-ped-stats] web page. The source for the total precipitation
and average wind speed is NOAA's [weather station at Sacramento Metropolitan
Airport][noaa-sac].

[bike-ped-stats]: https://www.cityofdavis.org/city-hall/public-works-engineering-and-transportation/bike-pedestrian-program/bike-and-pedestrian-data-statistics
[noaa-sac]: https://www.ncdc.noaa.gov/cdo-web/datasets/GHCND/stations/GHCND:USW00093225/detail
::::

The data set is saved in a Parquet file, which you can use the
`pl.read_parquet` function to read:

```{code-cell}
import polars as pl

bikes = pl.read_parquet("data/2020_davis_bikes.parquet")
bikes.head()
```

This data is not tidy, because it breaks rule 1. The `value` column contains
many different features---they even have different units! Soon we'll reshape
the data set to make it tidy.

Before you reshape a data set, you should also think about what role each
column serves:

* **Identifiers** (or indexes) are labels that distinguish observations from
  one another. They're often but not always categorical. Examples include names
  or identification numbers, treatment groups, and dates or times. In the bike
  counts data, the `date` column is an identifier.

* **Measurements** are the values collected for each observation and typically
  the values of research interest. For the tuberculosis data set, the `value`
  column is a measurement.

A clear understanding of which columns are identifiers and which are
measurements makes it easier to write the code to reshape.


### Rows into Columns

In order to make the Davis bike counts data tidy, the measurements in the
`value` column need to be moved into two separate columns, one for each of the
categories in the `variable` column.

You can use the `.pivot` method to **pivot** a data frame, creating new columns
from values in the rows. This makes the data frame wider (and shorter), so some
data frame packages call this operation `pivot_wider`. Let's pivot the `bikes`
data frame on the `variable` column to create four new columns filled with
values from the `values` column.

The `.pivot` method's most important parameters are:

* `on` -- The column that contains names for the new columns.
* `values` -- The column(s) that contains values for the new columns.
* `index` -- The identifier columns, which are not pivoted. This defaults to
  all columns except those in `on` and `values`.

Here's how to use the method to make `bikes` tidy:

```{code-cell}
bikes2 = bikes.pivot(on = "variable", values = "value")
bikes2.head()
```

The method automatically removes values from the `date` column as needed to
maintain the original correspondence with the pivoted values.

The new `bikes2` data frame contains all of the data from `bikes`, but now the
measurements for each date share a row. In other words, the observational units
for `bikes2` are dates, which is convenient for investigating how individual
features change over time, as well as making same-time comparisons between
features. To illustrate this, we can use Polars' `.plot.scatter` method to make
a scatter plot of the bike counts at 3rd Street against the counts at Loyola
Drive:

```{code-cell}
bikes2.plot.scatter(x = "third", y = "loyola")
```

The plot shows that Loyola Drive occasionally has days with much higher traffic
than 3rd Street, but it's difficult to tell whether 3rd or Loyola is typically
busier (that is, whether there are more points above or below the $y = x$
line).


(columns-into-rows)=
### Columns into Rows

Suppose we want to try to get a better answer to whether Loyola or 3rd tends to
be busier. One way we can do it is by making a line plot with the counts for
each site over time. This way we're treating each site as a group within the
data and making a comparison between groups.

Comparing groups in a data frame is generally easier when each row corresponds
to an observation from one group. Let's reshape the `bikes2` data frame so that
the observational units are date-site combinations. To do this, the `third` and
`loyola` columns need to be transformed into two new columns: one for
measurements (the counts) and one for identifiers (the sites). It might help to
visualize this as stacking the two separate columns `third` and `loyola`
together, one on top of the other, and then adding a second column with the
corresponding site names.

You can use the `.unpivot` method to **unpivot** a data frame, creating new
rows from values in the columns. This is the inverse of a pivot. It makes the
data frame longer (and narrower), so some data frame packages call this
operation `pivot_longer`. We'll unpivot the `bikes2` data frame on the `third`
and `loyola` columns.

The `.unpivot` method's parameters are:

* `on` -- The columns to stack into a new column; the names of these columns
  will also go into a new column.
* `index` -- The identifier columns, which are not unpivoted.
* `variable_name` -- Name(s) for the new identifier column(s)
* `value_name` -- Name(s) for the new measurement column(s)

For `.unpivot`, it's important to set both `on` and `index`, since any columns
you don't include in one or the other will be dropped.

The code to unpivot `bikes2` is:

```{code-cell}
bikes3 = bikes2.unpivot(
    on = ["third", "loyola"],
    index = ["date", "prcp", "awnd"],
    variable_name = "site",
    value_name = "count"
)
bikes3.head()
```

For `bikes3`, the observational units are date-site combinations, as planned.
This is convenient for comparing the two sites to each other with statistics
and visualizations. We can use Polars' `.plot.line` method to make a line plot
of the counts for the two sites:

```{code-cell}
plot = bikes3.plot.line(x = "date", y = "count", color = "site")
# Make the plot 600 pixels wide.
plot.properties(width = 600)
```

From this plot, we can see that Loyola Drive was generally busier for the first
3 months of 2020. Traffic dropped at both sites in mid-March, probably due to
the COVID-19 pandemic. The drop was sharper at Loyola Drive than 3rd Street, so
3rd Street was generally busier for the remaining 9 months of 2020.

:::{note}
We didn't use `prcp` and `awnd` (and they don't differ between sites anyway), so
we could've let `.unpivot` drop them. The resulting data frame would have the
same observational units as `bikes3`, but would also be a subset of the
original `bikes` data frame (albeit with different column names).
:::


### Case Study: SMART Ridership

[Sonoma-Marin Area Rail Transit (SMART)][smart] is a relatively new single-line
passenger light rail service between the San Francisco Bay and Santa Rosa. They
publish data about monthly ridership online, but the format is slightly messy.
Let's clean and reshape the data in order to make a plot of ridership over
time.

[smart]: http://sonomamarintrain.org/

:::{important}
[Click here][smart-riders] to download the SMART Ridership data set (version
2025-02).

If you haven't already, we recommend you create a directory for this workshop.
In your workshop directory, create a `data/` subdirectory. Download and save
the data set in the `data/` subdirectory.

[smart-riders]: https://ucdavis.box.com/s/kj5ac1ptdt3lb0sfmcolnxus90bcgewm
:::

:::{admonition} Documentation for the SMART Ridership Data Set
:class: note, dropdown

The source for the data set is the [SMART Ridership Reports][smart-reports] web
page.

[smart-reports]: https://www.sonomamarintrain.org/RidershipReports
:::

The data set is saved as a Microsoft Excel file. Before reading an Excel file,
it's a good idea to manually inspect it with spreadsheet software to figure out
how the data are organized. The SMART data set contains two tables on the left
side of the first sheet: one for total monthly ridership and one for average
weekday ridership (by month). Let's focus on the total monthly ridership table.

You can use Polars' `pl.read_excel` function to read sheets from an Excel file.

:::{important}
The `pl.read_excel` function depends on the `fastexcel` package. Make sure to
install the package if you haven't already.
:::

Use `pl.read_excel` to read the SMART Ridership data:

```{code-cell}
:tags: ["remove-stderr"]
smart = pl.read_excel("data/2025-02_smart_ridership.xlsx")
smart.head()
```

The total monthly ridership table corresponds to rows `1:14` and the first 9
columns:

```{code-cell}
smart = smart[1:14, :9]
smart
```

The first row is a header, so let's use it to set the column names and remove
it from the data frame. You can get a row as a tuple (rather than a data frame)
with the `.row` method:

```{code-cell}
smart.columns = smart.row(0)
smart = smart[1:, :]
smart
```

The `FY` columns need to be cast to floats, and `FY18` uses a hyphen `-` to
indicate a missing value. We can use the `.replace` method to replace the
hyphen with a missing value and then cast the columns:

```{code-cell}
smart = smart.with_columns(
    pl.exclude("Month").replace("-", None).cast(pl.Float64)
)
smart.head()
```

There's still a lot of cleaning to do. The identifiers in this data set are the
months and years, and they're split between the row and column names. Each row
contains data from several different years, so the data set is not tidy. In
addition, the years are indicated in fiscal years (FY), which begin in July
rather than January, so some of the years need to be adjusted.

To make the data set tidy, it needs to be reshaped so that the values in the
various fiscal year columns are all in one column. In other words, the data set
needs to be unpivoted ({numref}`columns-into-rows`) on all of the `FY` columns.
Listing all of the columns would be tedious, but fortunately Polars provides a
shortcut: the `pl.selectors` namespace, which is conventionally imported as
`cs` (for "column selectors"), provides many helpful functions for selecting
columns. You can use the `cs.starts_with` function to select all columns that
start with a string. Let's try this out in the code to unpivot the `FY`
columns:

```{code-cell}
import polars.selectors as cs

smart = smart.unpivot(
    on = cs.starts_with("FY"),
    index = "Month",
    variable_name = "fiscal_year",
    value_name = "count"
)
smart.head()
```

In order to use the months and years in the data, we need to convert them to
dates. As a first step towards this, we can replace the `FY` prefix in the new
`fiscal_year` column with `20` and cast the column to integers. We can use the
`.str.replace` method to do the replacement:

```{code-cell}
smart = smart.with_columns(
    pl.col("fiscal_year").str.replace("FY", "20").cast(pl.Int64)
)
smart.head()
```

Next, we can use the `.str.to_date` and `.dt.month` methods to create a new
column of month numbers:

```{code-cell}
smart = smart.with_columns(
    month_num = pl.col("Month").str.to_date("%b").dt.month()
)
smart.head()
```

Now we need to transform the fiscal years in the `fiscal_year` column into
calendar years. A SMART fiscal year extends from July to the following June and
is named after the calendar year at the end of the fiscal year. So from July to
December, the calendar year is the fiscal year minus 1. We can 

```{code-cell}
smart = smart.with_columns(
    cal_year = 
        pl.col("fiscal_year") +
        pl.when(pl.col("month_num") >= 7).then(-1).otherwise(0)
)
smart.head()
```

Finally, we can use the `pl.date` function to construct dates from the
`cal_year` and `month_num` columns:

```{code-cell}
smart = smart.with_columns(date = pl.date("cal_year", "month_num", 1))
smart.head()
```

With the dates in the `date` column and the counts in the `count` column, we
have everything we need to make a plot of SMART ridership over time. We can use
the  `.plot.line` method to make the plot:

```{code-cell}
plot = smart.plot.line(x = "date", y = "count")
# Make the plot 600 pixels wide.
plot.properties(width = 600)
```

Notice the huge drop (more than 90%) in April 2020 due to the COVID-19
pandemic!


## Working with Relational Data

:::{important}
This section is still in development and will be posted soon.
:::

Think about how you would organize restaurant reviews data. Each restaurant has
a name, address, phone number, operating hours, and other restaurant level
details. Because the number of reviews for each restaurant will vary, we could
put the data in a single table with one row for each review. Then the table is
at the review level, and we'd have to repeat restaurant level details across
all reviews for each restaurant.

The review level table is convenient if we want to analyze reviews, but not so
convenient if we want to analyze restaurants. For instance, suppose we want to
count how many restaurants open before 10 a.m. In order to avoid counting each
restaurant multiple times, we have to reduce the table to one row per
restaurant before we compute the count.

The problem with putting the restaurant review data in a single table is that
it consists of observations at two different levels: the restaurant level and
the review level. From this perspective, an intuitive solution is to put the
data in two tables: a "restaurants" table where each row is a restaurant and a
"reviews" table where each row is a review. Then we can choose the most
suitable table for each question we want to answer.

Each review is associated with a restaurant, and each restaurant is associated
with some reviews, so the two tables are related even though they're separate.
We can keep track of the relationship by including a column for restaurant name
(or a unique restaurant identifier) in both tables. This makes the data
**relational**: multiple tables where the relationships between them are
expressed through columns in common.

:::{note}
Most database software are designed to efficiently store and query relational
data, so in computing contexts, **database** is often synonymous with
relational data.
:::

For some questions, we'll need to use both the restaurants table and the
reviews table. For example, suppose we want to count how many restaurants open
before 10 a.m. *and* have at least 1 five-star review. We can use the reviews
table to compute the number of five-star reviews for each restaurant, combine
these with the restaurants table, and then count restaurants that meet our
conditions in the resulting table. Operations that combine two related tables
based on columns they have in common are called **joins**. The two tables are
conventionally called the **left table** and **right table**.

There are a few different kinds of joins. We'll use a simplified, fictitious
version of the restaurant reviews data to demonstrate some of them. The
`restaurants` table contains names and phone numbers for three restaurants:

```{code-cell}
restaurants = pl.DataFrame({
    "id": [1, 2, 3],
    "name": ["Alice's Restaurant", "The Original Beef", "The Pie Hole"],
    "phone": ["555-3213", "555-1111", "555-9983"]
})
restaurants
```

The `reviews` table contains scores from five restaurant reviews:

```{code-cell}
reviews = pl.DataFrame({
    "id": [4, 2, 1, 2, 2],
    "score": [4.2, 3.5, 4.7, 4.8, 4.0],
})
reviews
```


### Inner Joins

An **inner join** only keeps rows from the left table if they match rows in the
right table and vice-versa.

You can use the `.join` method to join two data frames with Polars. The data
frame on the left side of `.join` is the left table. The right table is the
first argument to `.join`. The `.join` method also requires an argument for the
`on` parameter, which should be the name of the column to use to match the two
data frames. By default, `.join` does an inner join.

Try joining the `restaurants` table and the `reviews` table:

```{code-cell}
restaurants.join(reviews, on = "id")
```

The inner join keeps rows where `id` is `1` or `2`, since these values appear
in both tables. It drops the row where `id` is `3` in `restaurants` and the row
where `id` is `4` in `reviews`. The resulting table has 4 rows and the columns
from both data frames.

:::{tip}
If the column you want to use to join two data frames has a different name in
each one, set the `left_on` and `right_on` parameters instead of `on`.
:::


### Left & Right Joins

A **left join** keeps all rows from the left table and only keeps rows from the
right table if they match. Missing values fill any spaces where there was no
match.

You can do a left join with the `.join` method by setting `how = "left"`. Try a
left join on the restaurant reviews data:

```{code-cell}
restaurants.join(reviews, on = "id", how = "left")
```

The left join keeps all of the rows from `restaurants`, and matches rows from
`reviews` when possible. There are no reviews where the `id` is `3`, so `score`
is missing for that row.

A left join is asymmetric, so switching the order of the tables will generally
produce a different result:

```{code-cell}
reviews.join(restaurants, on = "id", how = "left")
```

A **right join** is equivalent to a left join with the order of the tables
switched. Because of this, some relational data tools don't have a right join
command (only a left join command). You can do a right join with the `.join`
method by setting `how = "right"`.


### Full Joins

A **full join** keeps all rows from both tables. Missing values fill any spaces
where there was no match.

You can do a full join with the `.join` method by setting `how = "full"`. Try a
full join on the restaurant reviews data:

```{code-cell}
restaurants.join(reviews, on = "id", how = "full")
```

:::{seealso}
There are a few more kinds of joins. The Polars User Guide has [a complete
list with examples.][pl-joins].

[pl-joins]: https://docs.pola.rs/user-guide/transformations/joins/
:::


### Case Study: CA Crash Reporting System

The California Highway Patrol publish data about vehicle crashes in the state
as the California Crash Reporting System (CCRS). The CCRS is a relational data
set with three tables per year which describe crash events, parties involved,
and all injuries, witnesses, and passengers. Let's use the 2024 CCRS data for
Sacramento and Yolo Counties to compute statistics about crashes in those
counties.

:::{important}
[Click here][crash-data] to download the 2024 Sacramento & Yolo Crash data set
(3 CSV files).

[crash-data]: https://ucdavis.box.com/s/kjxowylvg3cfqgnofh03gn5e3xicsobt

If you haven't already, we recommend you create a directory for this workshop.
In your workshop directory, create a `data/` subdirectory. Download and save
the data set in the `data/` subdirectory.
:::

:::{admonition} Documentation for the 2024 Sacramento & Yolo Crash Data Set
:class: note, dropdown

The data set consists of three tables:

* `2024_sac-yolo_crashes.csv`, where each row is a crash.
* `2024_sac-yolo_parties.csv`, where each row is a person directly involved in
  the crash (not a witness or passenger).
* `2024_sac-yolo_injured-witness-passenger.csv`, where each row is an injured
  person (including drivers), witness, or passenger.

[Click here][ccrs-docs] to download the documentation for the source data set.

[ccrs-docs]: https://ucdavis.box.com/s/pfgbup6fw17nq7vajo1e05eoj0w57ygi

This data set is a subset of the much larger [CA Crash Reporting System data
set][ccrs].

[ccrs]: https://data.ca.gov/dataset/ccrs
:::
