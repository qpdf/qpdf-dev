The current plan is to implement the following steps of
the [pages epic](https://github.com/qpdf/qpdf/discussions/1104) next. The plan is provisional and
subject to change. Comments are welcome [here](https://github.com/qpdf/qpdf/discussions/xxx) . The
nitty-gritty implementation detail is discussed
at [qpdf/qpdf-dev](https://github.com/qpdf/qpdf-dev/discussions/3) .

Further detail will be added as the work progresses.

## Step 1: add new `--input` option

The plan is to provide a new option to allow all the input files to be defined outside the
`--pages` option. So, instead of

```
qpdf main.pdf \
     --pages . --file=input1.pdf --range=3-z --file=input2.pdf --password=xyz --file=input3.pdf -- \
     out.pdf
```

it will be possible to use

```
qpdf --input --file=main.pdf --file=input1.pdf --file=input2.pdf --password=xyz --file=input3.pdf -- \
     --pages --id=0 --id=1 --range=3-z --id=2 -- \
     out.pdf
```

instead. While the new option provides no benefit in this example, it will make it easier to deal
with complex assemblies of pages where the same input file appears multiple times in the `--pages`
options. More importantly, it provides the following additional benefits:

- It will provide a convenient way to add options that apply to an entire input file. For an example
  see the `--destinations` option described under
  [step 2b](#step-2b-add-option-to-preserve-named-destinations-when-merging-files).
- It will be more convenient (and more efficient) for users of job JSON or the QPDFJob C++/C API.
  Such users are likely to have a table of inputs already, and in many cases each entry in
  the `--pages` option will be for a single page.
- It will provide an opportunity to enhance the QPDFJob interface in future to allow additional
  types of inputs to be specified in the `--files` option, including memory buffers or QPDF objects.

In order to make the commands less verbose, short-forms are planned for `--file`, `--password`,
`--range` and `--id`, allowing the examples given above to be abbreviated to

```
qpdf main.pdf \
     --pages . -f=input1.pdf -r=3-z -f=input2.pdf -p=xyz -f=input3.pdf -- \
     out.pdf
```

and

```
qpdf --input -f=main.pdf -f=input1.pdf -f=input2.pdf -p=xyz -f=input3.pdf -- \
     --pages -i=0 -i=1 -r=3-z -i=2 i=3 -- \
     out.pdf
```

## step 2a: allow rotation to be specified inside the `--pages` option

For example, to combine two input files with the last page of each file rotated by 90 degrees,
instead of writing

```
qpdf main.pdf \
     --pages . --file=input1.pdf --range=3-z-- \
     --rotate=90:3,27 \
     out.pdf
```

it will be possible to write

```
qpdf --input -f=main.pdf -f=input1.pdf -- \
     --pages
     -i=0 -r=1-r2 \
     -i=0 -r=z --rotate=90 \
     -i=1 -r=1-r2 \
     -i=1 -r=z --rotate=90 -- \
     out.pdf
```

This avoids having to work out the page number (in the output file) of the final page of each input
file. In particular, if this was a monthly job where the length of the input files may vary, it
would avoid the need to adjust the script every month.

It will also be more convenient for users of the QPDFJob interface.

The plan is for future modification options (e.g. scaling or cropping) to take the same approach.

## step 2b: add option to preserve named destinations when merging files

The plan is to have a new options that allows for named destinations to be preserved, which will
allow more hyperlinks in merged files to work correctly. Preservation of named destinations is also
a prerequisite for a future option to preserve outlines.

There is a runtime cost involved, and therefore preservation is going to be optional. It will be
possible to select the option globally or on a per input file basis.

In order to avoid name clashes, it is currently planned to prepend a unique id for each input file
to each destination name.
