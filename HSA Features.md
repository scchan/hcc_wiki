This page explains HSA-specific features implemented in C++AMP.  Please remember to use **"-Xclang -fhsa-ext"** to turn on these features.  For more information about C++AMP for HSA please refer to [HSA Support Status](HSA Support Status).

***

# Shared Virtual Memory (SVM) #

The most significant feature of HSA Architecture is Shared Virtual Memory (SVM).  Codes on GPU and CPU can now share a single memory address space.  Explicit memory copies between CPU and GPU can be eliminated.

To use SVM, the following pattern has to be employed as of now:

1. Define the array in host
1. Use a pointer to point to the array
1. In C++AMP lambda function, capture the pointer by value [=]
1. Access the array by dereferencing through the pointer

The following code snippet depicts the pattern aforementioned:
```
#!c++

  // Prepare input and output arrays on host
  int table_a[vecSize];
  int table_b[vecSize];
  int table_c[vecSize];

  // Obtain pointers to arrays
  int *p_a = &table_a[0];
  int *p_b = &table_b[0];
  int *p_c = &table_c[0];

  // launch kernel
  Concurrency::extent<1> e(vecSize);
  parallel_for_each(
    e,
    [=](Concurrency::index<1> idx) restrict(amp) {

      // p_a, p_b, p_c are captured by value [=]
      p_c[idx[0]] = p_a[idx[0]] + p_b[idx[0]];

  });
```

Since 0.5.0, it's also possible to capture host objects by reference now.  The following codes depict this:

```
#!c++

  // Prepare input and output arrays on host
  int table_a[vecSize];
  int table_b[vecSize];
  int table_c[vecSize];

  // launch kernel
  Concurrency::extent<1> e(vecSize);
  parallel_for_each(
    e,
    [&](Concurrency::index<1> idx) restrict(amp) {

      // table_a, table_b, table_c are captured by value [&]
      table_c[idx[0]] = table_a[idx[0]] + table_b[idx[0]];

  });
```


Compared to ordinary C++AMP programs, Concurrency::array or Concurrency::array_view are NOT needed anymore.  Implicit memory copies within C++AMP arrays are also NOT needed anymore.

An example of vector addition is provided as a [C++AMP Unit Test](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/RawPointer/array_add.cpp?at=master).

Other examples which depict capture host objects by reference are also provided as [C++AMP Unit Tests](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/a93a99ded71f8826b8c91a3c896cb9c26485deb2/tests/Unit/CaptureByRef/?at=master).


# Platform Atomics and Memory Order #

In C++11 atomic operations are included in C++ Standard Library.  In C++AMP for HSA we have implemented a partial mapping so atomic objects and their member functions defined in <atomic> can be used in HSA kernels.

By using platform atomics in conjunction with SVM, GPU codes can atomically read-modify-write host memory locations.  Thus it is possible to synchronize operations between CPU and GPU.

Several sample codes are implemented as [C++AMP Unit Tests](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/?at=master).

* [atomic_int.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/atomic_int.cpp?at=master) : Update the SVM example by using atomic_int as underlying data type.  I deliberately changed the calculation to a bit more complex ( table_c = (table_a + 1) + (table_b - 1) ).
* [sync_1way.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/sync_1way.cpp?at=master) : In this example I let CPU wait for GPU using atomic operations.  Each thread in GPU will atomically increase the value until 1,048,576.
* [sync_2way.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/sync_2way.cpp?at=master) : In this example I show how CPU and GPU can wait for each other.  Each thread in GPU will atomically check an atomic_int flag, which will be atomically altered by CPU by checking the status on GPU.  The result is an array with only a single "1" inside, and that "1" will gradually move toward the end of the array.
* [pingpong.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/pingpong.cpp?at=master) : A fancier version of sync_2way.  The "1" will now bounce around for some fixed amount of times before the program ends.
* [syscall.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/tests/Unit/PlatformAtomics/syscall.cpp?at=master) : Based on sync_2way, this example shows how to let GPU threads request CPU to invoke system calls and pass return values back to GPU.


## Limitation of Platform Atomics and Memory Order ##

* In the current implementation, atomic objects with width less than 32-bits can NOT be used in HSA kernels.  For example, atomic_flag, atomic_bool, atomic_char are NOT supported.
* Floating point atomic objects (atomic_float, atomic_double) are NOT supported.
* Atomic objects of unsigned types (atomic_uint, atomic_ulong) might NOT work due to bugs in HSAIL Compiler.
* Atomic objects of pointers (atomic<T*>) or atomic objects of user-defined types (atomic<UDT>) are NOT tested yet.
* In HSA, not all C++11 memory order are supported:
* * std::memory_order_consume is implemented as std::memory_order_acquire.
* * std::memory_order_seq_cst will sometimes be translated to std::memory_order_acq_rel.
* All atomic operations in HSA kernels will have system-wide memory scope (sys), and there is no way to specify memory scope in C++11 yet.

***

# Dynamic Memory Allocation and Deallocation #

On HSA Architecture, it's possible for HSA kernels to allocate / deallocate memory dynamically through C++ new / delete operators.  Memory allocated will reside in global segment so it could be shared among CPU and GPU.

Several sample codes are implemented as [C++AMP Unit Tests](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/?at=master).

## Allocation samples ##

* [new_scalar_non-tiled.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/new_scalar_non-tiled.cpp?at=master) , [new_scalar_tiled.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/new_scalar_tiled.cpp?at=master) : Demonstrate how to new scalar types.
* [new_scalar_array.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/new_scalar_array.cpp?at=master) : Demonstrate how to new scalar arrays.
* [new_obj.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/new_obj.cpp?at=master) : Demonstrate how to new objects.
* [new_obj_array.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/new_obj_array.cpp?at=master) : Demonstrate how to new object arrays.

## Deallocation samples ##

* [delete.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/delete.cpp?at=master) : Demonstrate how to use delete operator.

## More concrete samples ##

* [fibonacci.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/fibonacci.cpp?at=master) : Demonstate the full process of allocating an array in each GPU thread, and use the allocated array to do some computations, and release the array.

## How the implementation works (Outdated) ##

In Clang C++AMP frontend, each new / new[] / delete / delete[] will eventually be translated to calls to the following C++ builtin functions:

* _Znwj : new
* _Znaj : new[]
* _ZdlPv : delete
* _ZdaPv : delete[]

To actually allocate / deallocate memory, we have manually created 2 versions of HSAIL logic:
* an [very simple allocator algorithm](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/0ea1dc692164a9542fa3dc849705fb64c5a4ed13/lib/new_delete.hsail?at=master)
* use [platform atomics](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/49e72c092e2c5b9f70d780c195ada427dd8cfbee/lib/new_delete.hsail?at=master) to ask CPU to do malloc / free

In the more sophisticated (platform atomic) version, GPU threads will use atomic objects to submit requests to CPU.  A CPU service thread would be checking atomic objects for each GPU threads and call respective system calls.

Also an LLVM pass is written to ensure pointers returned to GPU kernels will be promoted to address space 1 (addrspace(1)) which means they are in HSA global segment to ensure correct HSAIL instructions will be emitted.

## Limitations of Dynamic Memory Allocation and Deallocation ##

The current implementation is still rudimentary and will be gradually improved.  We have already identified following limitations:

1. Only support 1D kernels now.
1. The platform atomic implementation need helper routines (put_ptr_a/b/c/x/y/z) to pass data structure's address into new / delete implementation.
1. CPU-GPU communication method need to be updated : We employed platform atomic to do CPU-GPU communication.  Two service threads on CPU will be needed to poll the status on GPU. Once HSA runtime support agent dispatch packets and signal/queue HSAIL instructions we will update the implementation to remove this busy-waiting service threads.
1. Can't support BRIG right now : Memory allocation/deallocation algorithms are implemented in plain-text HSAIL and will be concatenated with GPU kernels in HSAIL before finalization.  Ideally it will need to be in BRIG format and link with GPU kernels in BRIG.  We will improve it once BRIG finalization becomes available.

## Benchmarking of Xmalloc/Xfree/malloc/free (To be improved) ##

We use [MilThread.cpp](https://bitbucket.org/multicoreware/cppamp-driver-ng-hsa/src/78d2b4e2aeeb0c14799df2a713f86842cf96931b/tests/Unit/NewDelete/MilThreads.cpp?at=master) for benchmarking. tile size = 256, inner loop = 64

If we set WITH_DELETE as 1:

Algorithm \ #Threads | 256 | 1 k | 4 k | 16 k | 64 k
-- | -- | -- | -- | -- | --
Xmalloc | 2.53612  | 2.52492 | 5.18776 | 12.7675 | 48.1073
malloc | 4.95359  | 5.02993 | 9.9825 | 25.0512 | 95.3349

If we set WITH_DELETE as 0:

Algorithm \ #Threads | 256 | 1 k | 4 k | 16 k | 64 k
-- | -- | -- | -- | -- | --
Xmalloc | 2.47995  | 2.43344 | 4.93228 | 12.3311 | 44.903
malloc | 2.29558  | 2.4688 | 4.93827 | 12.7047 | 48.1328

***

# Asynchronous parallel_for_each #

New interfaces have been introduced to support asynchronously launch a kernel:
```
#!c++
template <int N, typename Kernel>
completion_future async_parallel_for_each(extent<N> compute_domain, const Kernel& f);

template <int D0, int D1, int D2, typename Kernel>
completion_future async_parallel_for_each(tiled_extent<D0,D1,D2> compute_domain, const Kernel& f);

template <int D0, int D1, typename Kernel>
completion_future async_parallel_for_each(tiled_extent<D0,D1> compute_domain, const Kernel& f);

template <int D0, typename Kernel>
completion_future async_parallel_for_each(tiled_extent<D0> compute_domain, const Kernel& f);
```

By returning completion_future object from async_parallel_for_each, we are be able to achieve the following features:
* wait for kernel completion through existing wait() member function in completion_future
* daisy chain kernel execution through existing then() member function in completion_future

An example to daisy chain kernel execution looks like:
```
#!c++
std::promise<void> all_done;

// async launch k1
completion_future fut = async_parallel_for_each(e, k1);
fut.then([&] {
   // async launch k2 after k1 is done
   completion_future fut2 = async_parallel_for_each(e, k2);
   fut2.then([&] {
     // sync launch k3 after k2 is done
     parallel_for_each(e, k3);

     all_done.set_value();
   });
});

// wait for all kernels (k1, k2, k3) finishes execution
all_done.get_future().wait();
```

***

# Relax C++ language rules for C++AMP kernels on HSA #

With HSA extension (-Xclang -fhsa-ext), it's now possible to support more C++ language rules within C++AMP kernels.  Please refer to: [Relax C++ Lanauge Rules on HSA](https://docs.google.com/a/multicorewareinc.com/spreadsheets/d/1j7PgJsZa89Z51Kp49T8-4GU2lMl8pydfwuDoXcAxgGw) for more detailed information.

***

# Auto-auto : no restriction specifiers on HSA kernels #

In HSA extension mode, lambda kernels in parallel_for_each doesn't need restrict(amp) anymore.  Please refer to [sample code](https://bitbucket.org/multicoreware/cppamp-driver-ng/src/a93a99ded71f8826b8c91a3c896cb9c26485deb2/tests/Unit/AutoRestricted/auto_auto.cpp?at=master).
