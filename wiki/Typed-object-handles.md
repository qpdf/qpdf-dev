It is planned to introduce new typed object handles in qpdf 12 such as qpdf::Integer, qpdf::Name,
etc. The new handle types will be in addition to the existing QPDFObjectHandle class rather than a
replacement for it. What is described below are plans, with the detail subject to change - both in
response to comments / feedback and as the result of new insights gained during implementation. To
comment, go to the [discussion](https://github.com/qpdf/qpdf/discussions/1xxx) .

## Typed object basics

Typed handles operate at a slightly higher level than the existing qpdf object types. For example,
there is a qpdf::Numeric typed handle but no qpdf::Real because wherever the PDF specification
permits a real object an integer object is also acceptable. Also, it is quite common in the PDF
specification for objects to be optional, in which case a null object is also an acceptable
alternative. Typed handles are intended to distinguish between those objects that are acceptable for
a particular purpose and those that are not. Typed handles that point to an acceptable object are
considered valid and if converted to a `bool` will evaluate as `true`.

Typed handles can be created directly, e.g.

```cpp
qpdf::Integer typed1{42};
```

or from an existing object handle, e.g.

```cpp
auto typed2 = oh.as_integer();
```

They can be converted to object handles, providing the typed handle is valid (i.e. it points to the
correct type, in the example an 'optional integer', i.e. an integer object or a null object)

```cpp
QPDFObjectHandle oh2 = typed2;
```

or

```cpp
some_dict.replaceKey(“/SomeKey”,  typed2);
```

If the typed handle is not valid, conversion attempts will throw an exception. This can be avoided
by testing the typed handle before use

```cpp
if (auto const typed3 = oh.as_integer()) { // Note the 'const'
    some_dict.replaceKey(“/SomeKey”,  typed3);
}
```

Typed handles also support conversion to some appropriate C++ types

```cpp
std::size_t i2 = typed2;
int i3 = some_dict.getKey(“/SomeKey”).as_integer();
```

If the typed handle points to an integer outside the limits of the C++ types (e.g. if `typed2`
is negative), a `std::range_error` will be thrown.

## Required objects

Typed object handles can also be created to represent required objects rather than optional objects
by setting the optional parameter `optional` to false:

```cpp
qpdf::Integer typed3{42, false};
auto typed4 = oh.as_integer(false);
```

In this case, a typed handle pointing to a null object will evaluate as `false` and trying to
convert it to an object handle or C++ type will throw an exception.

## Other constraints

The PDF specification frequently add other constraints (in addition to type) to the requirements for
an object. For example, the requirements for the /Annots object of /Page objects include

> (Optional) An array of annotation dictionaries that shall contain indirect references to all
> annotations associated with the page

and

> A given annotation dictionary shall be referenced from the Annots array of only one page

Typed handles are intended to accommodate such additional constraints. This could be handled by
adding additional parameters to constructors and other methods used to create typed handles
(e.g. `as_integer`), but given the number of possible combinations (and the lack of named parameters
in C++) this would quickly become impractical and error-prone. To deal with these more complex
situations, there is a third way to create typed handles using a builder style approach:

```cpp
auto typed5 = qpdf::Integer::required().indirect().create(42);
```

## Mutation

An object can be mutated in-place by simple assignment as in

```cpp
some_dict.getKey(“/SomeKey”).as_integer() = 42;
```

provided the existing value returned by `some_dict.getKey(“/SomeKey”)` itself is of the correct
type (in this case, integer or null). The effect will be roughly similar to calling

```cpp
some_dict.replaceKey(“/SomeKey”,  "42"_qpdf);
```

with a number of important differences, including

- no new object will be created, making it more efficient;
- the object will be changed rather than replaced and therefore the change affects and is visible to
  all other object handles pointing to the existing object; and,
- the object becomes typed. There will be some error checking on future modification, which will be
  discussed more fully elsewhere.

In order for a type handle to be mutated it obviously has to point to a mutable object. To
accommodate checking for this requirement, a non-constant typed handle will evaluate as `false`
if it points to an immutable `null`.

```cpp
if (auto typed6 = some_dict.getKey(“/SomeKey”).as_integer()) {
    typed6 = 42;
}
```
