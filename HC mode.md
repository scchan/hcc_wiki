# hc API: An HSA-extension to C++ AMP

hc is a C++ API that the hcc compiler provides for accelerated computing. It has some similarities to C++ AMP, so reference materials (blogs, articles and books) that describe C++ AMP are also an excellent way to become familiar with hc. For example, both APIs use a *parallel_for_each* construct to specify a parallel execution region that runs on an accelerator. But hc differs from C++ AMP in several important ways, including the removal of the “restrict” keyword for annotating device code, an explicit asynchronous launch behavior for *parallel_for_each*, support for non-constant tile size and support for memory pointers.

---

# hc API

Currently, hc comes with two header files:

- <hc.hpp>---main hc header file
- <hc_math.hpp>---hc math functions

Most hc APIs are stored under the “hc” namespace, and the class name is the same as the counterpart in the C++ AMP “Concurrency” namespace. C++ AMP users should find it easy to switch to hc.

| C++ AMP | hc |
|----|--------|
| Concurrency::accelerator | hc::accelerator |
| Concurrency::accelerator_view | hc::accelerator_view |
| Concurrency::extent | hc::extent |
| Concurrency::index | hc::index |
| Concurrency::completion_future | hc::completion_future |
| Concurrency::array | hc::array |
| Concurrency::array_view | hc::array_view |

---

# Building Programs Using the hc API

To build a program, use *hcc-config* instead of *clamp-config*; alternatively, you can manually add *-hc* when you invoke Clang++. Also, *hcc* is an alias for *Clang++*. For example,

```
hcc `hcc-config --cxxflags --ldflags` foo.cpp -o foo
```

---

# hcc Built-In Macros

The following hcc macros are built-in:

| Macro | Meaning |
|----|--------|
| ```__HCC__``` | Always 1 |
| ```__hcc_major__``` | Major hcc version number |
| ```__hcc_minor__``` | Minor hcc version number |
| ```__hcc_patchlevel__``` | hcc patch level |
| ```__hcc_version__``` | String combining ```__hcc_major__```, ```__hcc_minor__``` and ```__hcc_patchlevel__``` |

The rule for ```__hcc_patchlevel__``` is *yyWW-(HCC driver git commit #)-(HCC clang git commit #).* Here,
- *yy* stands for the last two digits of the year
- *WW* stands for the week number of the year

The following language-mode macros are available:

| Macro | Meaning |
|----|--------|
| ```__KALMAR_AMP__``` | 1 if in C++ AMP mode (*-std=c++amp*) |
| ```__KALMAR_HC__``` | 1 if in hc mode (*-hc*) |

## Compilation Mode
hcc is a single-source compiler that allows kernel codes and host codes to reside in the same file. Internally, it triggers two compilation iterations; user programs can employ the following macros to determine which mode the compiler is in.

| Macro | Meaning |
|----|--------|
| ```__KALMAR_ACCELERATOR__``` | Nonzero if the compiler runs in kernel-code compilation mode |
| ```__KALMAR_CPU__``` | Nonzero if the compiler runs in host-code compilation mode |

---

# hc-Specific Features
The following features are specific to hc:

- Relaxed operating rules allowed in kernels
- New syntax of *tiled_extent* and *tiled_index*
- Dynamic group-segment memory allocation
- True asynchronous kernel-launching behavior
- Additional HSA-specific APIs


# Differences Between HC API and C++ AMP

Although hc and C++ AMP share many similarities in programming constructs (e.g., *parallel_for_each,* *array* and *array_view*), they exhibit significant differences.

## Support for Explicit Asynchronous ```parallel_for_each```

In C++ AMP, ```parallel_for_each``` appears as a synchronous function call in a program (i.e., the host waits for the kernel to complete); the compiler, however, may optimize it to execute the kernel asynchronously. The host would then synchronize with the device on the first access of the data modified by the kernel. For example, if a ```parallel_for_each``` writes an *array_view,* the first access to this *array_view* on the host after the ```parallel_for_each``` call would be blocked until that call completes. 

hc supports the same automatic synchronization behavior as C++ AMP. In addition, its ```parallel_for_each``` function supports explicit asynchronous execution. It returns a ```completion_future``` (similar to C++ *std::future*) object that other asynchronous operations can synchronize with, providing better flexibility on task-graph construction and enabling more-precise optimization control. 

## Device-Function Annotation

C++ AMP uses the ```restrict(amp)``` keyword to annotate functions that run on the device.

```
void foo() restrict(amp) {
..
}
...
parallel_for_each(...,[=] () restrict(amp) {
 foo();
});

```

hc uses a function attribute (```[[hc]]``` or ``` __attribute__((hc))```) to annotate a device function. 

```
void foo()  [[hc]] {
..
}
...
parallel_for_each(...,[=] () [[hc]] {
 foo();
});
```

The *\[\[hc\]\]* annotation for the kernel function called by ```parallel_for_each``` is optional, since the hcc compiler automatically annotates it as a device function. The compiler also supports partial automatic *\[\[hc\]\]* annotation for functions that are called by other device functions in the same source file:

```
// Since bar is called by foo, which is a device function, the hcc compiler
// will automatically annotate bar as a device function
void bar() {
...
}

void foo() [[hc]] {
  bar();
}
```

## Dynamic Tile Size

C++ AMP doesn't support dynamic tile size. Each tile dimension must be a compile-time constant specified as template arguments to the *tile_extent* object:

```
extent<2> ex(x, y);

// Create a tile extent of 8x8 from the extent object
// Note that the tile dimensions must be constant values
tiled_extent<8,8> t_ex(ex);

parallel_for_each(t_ex, [=](tiled_index<8,8> t_id) restrict(amp) {
...
});
```
hc supports both static and dynamic tile size:
```
extent<2> ex(x,y)

// Create a tile extent from dynamically calculated values
// Note that the tiled_extent template takes the rank instead of dimensions
tx = test_x ? tx_a : tx_b;
ty = test_y ? ty_a : ty_b;
tiled_extent<2> t_ex(ex, tx, ty);

parallel_for_each(t_ex, [=](tiled_index<2> t_id) [[hc]] {
...
});

```

## Support for Memory Pointers

C++ AMP lacks support for lambda capture of memory pointers into a GPU kernel. hc allows you to capture memory pointers implemented by a GPU kernel.

```
// Allocate GPU memory through the HSA API
int* gpu_pointer;
hsa_memory_allocate(..., &gpu_pointer);
...
parallel_for_each(ext, [=](index i) [[hc]] {
  gpu_pointer[i[0]]++;
}

```
For HSA APUs that enable systemwide shared virtual memory, a GPU kernel can directly access system memory allocated by the host:
```
int* cpu_memory = (int*) malloc(...);
...
parallel_for_each(ext, [=](index i) [[hc]] {
  cpu_memory[i[0]]++;
});
```




