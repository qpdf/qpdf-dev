PDF null objects vs qpdf null objects
=====================================

The PDF specification states

> The null object has a type and value that are unequal to those of any other object. There shall be
> only one object of type null, denoted by the keyword null.

This is straightforward - there is a single object and, by implication, it is immutable. In other
words, all nulls encountered are equal.

Unfortunately, in qpdf the situation is not quite as straightforward. There are a couple of reasons
for this:

- qpdf objects have a variety of diagnostic information attached to them. For example, an attempt to
  retrieve a object with `getKey("/Blah")` from an array will return a null object that carries the
  description " -> null returned from getting key /Blah from non-Dictionary".

- For performance reasons, qpdf does not have a distinct 'indirect reference' type. Under the hood,
  all object handles are references (or rather shared pointers) to the actual object. To avoid a
  further level of indirection, qpdf objects have an id and generation attached; if id and
  generation are both zero, the object is a direct objects. Otherwise, it is an indirect reference.
  As a result, in qpdf each indirect null is a distinct object. Furthermore, qpdf allows indirect
  objects to be updated with the `replaceObject` method. It also allows direct objects, including
  direct nulls, to be made indirect.

Therefore, in qpdf null objects are not all the same, nor are they immutable. This is particularly
true for indirect nulls. For example, if a foreign object (i.e. an object from a different PDF
file / QPDF object) is copied using the `copyForeignObject` method, any references to pages that
have not already been copied will appear as an indirect null. If the page is subsequently copied,
those indirect nulls will be transparently updated to become the copied page.

On the other hand, qpdf has to create a large number of nulls behind the scenes, most of which are
short-lived and never seen by users of the qpdf library. To avoid the overhead of creating these
nulls, qpdf uses references to a single shared null object. This does not normally cause any
problems, but it could if an attempt was made to mutate such a shared null.

qpdf 12 will make a clearer distinction between these two types of null objects - there will be
**mutable nulls** and **shared nulls**. In allmost all existing situations, both types of null
object will behave identically. The main differences are that

- shared nulls cannot be made direct,
- shared nulls never have an owner (i.e. `getOwningQPDF` will always return a nullptr), and
- attempts to set descriptions, offsets, etc will be ignored by shared nulls.

To implement shared nulls, qpdf 12 will use default constructed or uninitialized object handles. The
`isInitialized` method will be removed, and for other methods such as `isNull`, `unparse`,
`shallowCopy`, etc, uninitialized object handles will be treated as null objects.

Shared nulls can be distinguished from mutable nulls (or any other object) by converting them to
a `bool` value - shared nulls will evaluate to `false`, all other objects to `true`.

Existing code that uses the `isInitialized` method can normaly be updated by simply updating method
calls such as `some_object_handle.isInitialized()` with the object handle
`some_object_handle` itself. This will work when used as condition in if and while statements, in
logical expressions or when initializing a bool variable. In some other situations, such as when
assigning to an existing bool variable or in a return statement it will be necessary to explicitely
cast the object handle to bool.

The only complication arises when `some_object_handle` may be a null object that needs to be treated
differently from an uninitialized object. This is an unusual situation and the only circumstance
where this is likely to occur is when a user method returns an uninitialized object handle to
signal some sort of failure. In this situation it is necessary to consider whether a legitimate null
object could possibly be a shared null (objects returned by `newNull` or `getKey` will never be 
shared bulls), and if so, replace such nulls with a freshly constructed null using a call to 
`newNull()`. An alternative solution would be to use `std::optional<QPDFObjectHandle>` as the 
return type / variable type, which would distinguish between a missing object handle and an 
uninitialized object handle.

(To comment, please go to https://github.com/qpdf/qpdf/discussions/xxxx)
