References
==========

This reader covers a variety of intermediate Python topics, but there's always
more to learn, and seeing other perspectives will help you understand the
concepts better. Many Python and data science learning resources are available
for free online or through the library.

Some references I recommend are:

* [Python for Data Analysis][pydata] (2nd edition) by McKinney. An introduction
  to using Python for data science, by the creator of the Pandas package.
* [Python Data Science Handbook][pyhandbook] by VanderPlas. 
* [Python Basics][py-basics], the reader from DataLab's introductory Python
  workshop series.
* My own [teaching notes][notes] from several years of teaching
  statistical computing.

[pydata]: https://wesmckinney.com/pages/book.html
[pyhandbook]: https://jakevdp.github.io/PythonDataScienceHandbook/
[py-basics]: https://ucdavisdatalab.github.io/workshop_python_basics/
[notes]: https://github.com/nick-ulle/teaching-notes


Section 1
---------

{numref}`chapter-indexing` is a deep dive into how indexing works in the pandas
package. While developing the chapter, I relied heavily on the excellent
[pandas documentation][pandas-docs].

[pandas-docs]: https://pandas.pydata.org/pandas-docs/stable/


Section 2
---------

{numref}`chapter-reproducible` describes strategies to improve the organization
and reproducibility of your computing projects. The chapter was informed by the
following references:

* DataLab's [README, Write Me! Reader][readme]
* [The Turing Way][turing]
* [The Hitchhiker's Guide to Python][hitchhikers]
* Jenny Bryan's presentation [How to Name Files][naming-files]
* [Cookiecutter Data Science][cookiecutter]
* [This chapter][remote-conda] of DataLab's Introduction to Remote Computing
  Reader

[readme]: https://ucdavisdatalab.github.io/workshop_how-to-data-documentation/
[turing]: https://the-turing-way.netlify.app/
[hitchhikers]: https://docs.python-guide.org/
[naming-files]: https://speakerdeck.com/jennybc/how-to-name-files
[cookiecutter]: https://drivendata.github.io/cookiecutter-data-science/
[remote-conda]: https://ngs-docs.github.io/2021-august-remote-computing/installing-software-on-remote-computers-with-conda.html


Section 3
---------

{numref}`chapter-iterators` is a deep dive into how Python's iterators and
generators work. The chapter is based on the following references:

* [Generator Tricks for Systems Programmers, v3.0][beazley] by Beazley.
* The official [Python 3 Documentation][py3-docs].
* The [Scaling to Large Datasets][scaling] chapter of the
  pandas documentation.

[beazley]: https://www.dabeaz.com/generators/
[py3-docs]: https://docs.python.org/3/
[scaling]: https://pandas.pydata.org/docs/user_guide/scale.html
