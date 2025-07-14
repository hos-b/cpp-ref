# Modules
modules are a way to organize code into separate units, allowing for better encapsulation and faster compilation. they provide a way to define interfaces and implementations separately, which can help reduce dependencies and improve build times.

key changes:
- replace header files
- separation into _module interface files_ and _module implementation files_ [optional]
- can be structured with submodules and module partitions
- no need for include guards
- no need to invent unique names -> only importing identical names causes a clash
- modules are processed only once -> faster build times
- preprocessor macros have no effect on and never leak from modules
- order of imports is not important

```cpp
// test_module.cppm

// module declaration
export module test_module;

namespace MyModule {
    auto GetString() { return "this is not exported"; }
    export auto GetExportedString() { return "this one is"; }
}
```

(as of yet) the standard library is not modularized but all C++ headers are importable:
- with `import <header>;`
- everything in `header` is exported, including macros (exception)
- improves build throughput; `header` will be processed only once.
- comparable to precompiled headers

this trick can also work on custom headers, but is compiler dependent.

## C++23 improvement
we now get the `std` module which imports all standard library types and functions as well as C functions under the `std` namespace.
we also get the `std.compat` module where the C functions are imported into the global namespace.
