# Books

## 2014

### Effective Modern C++ by Scott Meyers

#### Item 4: Know how to view deduced types - Compiler Diagnostics

I thought Meyers included a great trick here to help deduce types that might be
causing the user issues.

```cpp
template<typename T>       // declaration only for TD;
class TD;                  // TD == "Type Displayer"
```

Then instantiate TD with the type:

```cpp
TD<decltype(x)> xType;     // elicit errors containing
TD<decltype(y)> yType;     // x's and y's types
```

Very simple, very easy, great! Basically, we get compile-time errors with the
type we are looking for.

Because of this tip, I started to snoop around for other techniques used to
display the typename in question. I found a great [StackOverflow](https://stackoverflow.com/questions/81870/is-it-possible-to-print-a-variables-type-in-standard-c/56766138#56766138)
article about this exact topic. The 'last' solution was quite good (for C++17):

```cpp
#include <string_view>

template <typename T>
constexpr auto type_name() {
  std::string_view name, prefix, suffix;
#ifdef __clang__
  name = __PRETTY_FUNCTION__;
  prefix = "auto type_name() [T = ";
  suffix = "]";
#elif defined(__GNUC__)
  name = __PRETTY_FUNCTION__;
  prefix = "constexpr auto type_name() [with T = ";
  suffix = "]";
#elif defined(_MSC_VER)
  name = __FUNCSIG__;
  prefix = "auto __cdecl type_name<";
  suffix = ">(void)";
#endif
  name.remove_prefix(prefix.size());
  name.remove_suffix(suffix.size());
  return name;
}
```

There isn't an obvious way to use this function to print out during
compile-time, but it's fun to have handy.

#### Item 6: Use the explicitly typed initializer idiom whenauto deduces undesired types

There's a bit in this section that talks about 'invisible proxy' types, and how
they can screw up `auto` declarations. I didn't mind the solution (use
`static_cast` to explicitly define expected type), but I really didn't like the
idea of these invisible classes floating around, messing with `auto`.

#### Item 7: Distinguish between () and {} when creating objects

This actually made me rethink my own practices of initializing objects. I know
that I interchange usage of `()`, `{}`, and `= {}` with minimal thought. I don't
know why I do it, but I know that I'll stop that now. I think I'll go with
braced initialization, primarily because I don't tend to use
`std::initializer_list` often in my constructor signatures, so I'm not worried
about confusion between constructors.

#### Item 8: Prefer nullptr to 0 and NULL

If my project is using C++11, I will use `nullptr`, no question. After reading
this, I was surprised to see that `NULL` didn't have a pointer type. I could
have sworn that `NULL` was a `void *` but IT IS NOT!

I checked the MSVC implementation:

```cpp
#ifndef NULL
    #ifdef __cplusplus
        #define NULL 0
    #else
        #define NULL ((void *)0)
    #endif
#endif
```

What on earth! I even went to look at the C++ spec!

```text
187) Possible definitions include 0 and 0L, but not (void*)0
```

How did I never realize this? I'm happy to realize that I don't overload
function signatures with `int` and pointer-types, so I wouldn't have run into
this issue with `NULL` personally, but I'm shook that I wasn't aware that
`NULL` wasn't already a pointer in C++!

#### Item 10: Prefer scoped enums to unscoped enums

Some good tips for me in here:

* I didn't realize I could forward declare scoped enums! Will definitely take
  advantage of this!
* I've run into the exact issue of not being able to use a scoped enum values as
  an index directly without using `static_cast`. I love the little utility to
  convert to the underlying type (`toUType`).
  