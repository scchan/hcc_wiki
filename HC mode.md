# HC API : An HSA-extension to C++AMP

HC is a C++ API derived from C++AMP. It has nearly identical features with C++AMP in:

- abstract modeling of GPU devices: accelerator / acclerator_view
- multi-dimensional array / array_view
- multi-dimensional index / extent
- parallel_for_each kernel launching interface
- math & atomic functions

And it comes with a few HSA-specific features:

- relaxed rules in operations allowed in kernels
- new syntax of tiled_extent and tiled_index
- dynamic group segment memory allocation
- true asynchronous kernel launching behavior
- additional HSA-specific APIs

---

# HC API

HC comes with two header files as of now:

- <hc.hpp> : Main header file for HC
- <hc_math.hpp> : Math functions for HC

Most HC APIs are stored under "hc" namespace, and the class name is the same as their counterpart in C++AMP "Concurrency" namespace.  Users of C++AMP should find it easy to switch from C++AMP to HC.

| HC | C++AMP |
|----|--------|
| Concurrency::accelerator | hc::accelerator |
| Concurrency::accelerator_view | hc::accelerator_view |
| Concurrency::extent | hc::extent |
| Concurrency::index | hc::index |
| Concurrency::completion_future | hc::completion_future |
| Concurrency::array | hc::array |
| Concurrency::array_view | hc::array_view |

---

# How to build programs with HC API

Use "hcc-config", instead of "clamp-config", or you could manually add "-hc" when you invoke clang++. Also, "hcc" is added as an alias for "clang++".

For example:

```
hcc `hcc-config --cxxflags --ldflags` foo.cpp -o foo
```

---

# HCC built-in macros


---

# HC-specific features

- relaxed rules in operations allowed in kernels
- new syntax of tiled_extent and tiled_index
- dynamic group segment memory allocation
- true asynchronous kernel launching behavior
- additional HSA-specific APIs
