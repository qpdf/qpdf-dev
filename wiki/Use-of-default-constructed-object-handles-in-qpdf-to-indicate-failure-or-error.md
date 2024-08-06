It is expected that qpdf 11.10 will include an operator to convert object handles to bool.
Uninitialized / default constructed object handles will evaluate to false. This note outlines how
this operator can be used.

For the basic scenario, take `some_method` that returns an object handle:

```c++
QPDFObjectHandle some_method()
{
    // do some calculations
    if (success) {
        return some_valid_oh;
    } else {
        return {};
    }
}
```

The operator can now be used for error handling:

```c++
if (auto result = some_method()) {
    // handle success
} else {
    // handle failure
}
```

This can be expanded to multiple method calls:

```c++
auto part1 = some_method();
auto part2 = some_other_method();
if (part1 && part2 && other_relevant_conditions) {
    // handle success
} else {
    // handle failure
}
```

If the method is run for side effects only, we can use it to initialize a bool variable:

```c++
bool success{some_method()};
```

Note however that in many other situation it will be necessary to explicitly convert the object
handle to bool, notably when assigning to an existing variable or using it as a bool return value

```c++
existing_flag = static_cast<bool>(some_method());
```

**Important** Methods do not normally return null objects when successful. If it is possible that a
null object will be returned, it is important to be aware of planned changes to null objects
described at [[PDF null objects vs qpdf null objects]]
