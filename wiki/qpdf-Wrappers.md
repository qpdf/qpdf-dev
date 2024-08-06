# Wrappers around qpdf

This page includes a list of known wrappers around qpdf. If you have a library that enhances qpdf or
exposes its functionality for programmers of languages other than C++, let me know, and I'll add it
to this list.

## Python

* [pikepdf](https://pypi.org/project/pikepdf/): a fully featured Python PDF library build upon qpdf.
  It exposes most of the functionality of qpdf in a "Pythonic" (idiomatically Python) way and offers
  many higher-level interfaces to work with PDF files beyond what is provided directly by qpdf.

## C#

* [QPdfNet](https://github.com/Sicos1977/QPdfNet): a C# package that exposes uses qpdf's `QPDFJob`
  API to expose functionality that is available from the qpdf CLI without having to actually invoke
  the `qpdf` command.
