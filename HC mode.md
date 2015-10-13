# HC API : An HSA-extension to C++AMP

HC is a C++ API derived from C++AMP. It has nearly identical features with C++AMP in:

- abstract modeling of GPU devices: accelerator / acclerator_view
- multi-dimensional array / array_view
- multi-dimensional index / extent
- parallel_for_each kernel launching interface
- math & atomic functions

And it comes with a few HSA-specific features:

- new syntax of tiled_extent and tiled_index
- dynamic group segment memory allocation
- true asynchronous kernel launching behavior
- additional HSA-specific APIs

