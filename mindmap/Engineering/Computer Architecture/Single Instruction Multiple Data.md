Imagine you have to **add two arrays of numbers**:
```
A = [1, 2, 3, 4]
B = [5, 6, 7, 8]
```
and you want `C = A + B = [6, 8, 10, 12]`. A simple CPU without SIMD executes **one instruction at a time per data item**:
```
load A[0]
add B[0]
store C[0]
...
```
→ That’s 4 adds = 4 instructions.

With **SIMD**, the CPU can execute **one instruction that operates on multiple data elements at once**.  
So one “add” instruction might add 4 pairs of numbers **in parallel**:

Let’s step back and see where SIMD fits in the broader **parallel processing taxonomy** — this is called **Flynn’s Taxonomy**.

| Type     | Meaning                               | Example                                                         |
| -------- | ------------------------------------- | --------------------------------------------------------------- |
| **SISD** | _Single Instruction, Single Data_     | Classic single-core CPU (one thread, one instruction at a time) |
| **SIMD** | _Single Instruction, Multiple Data_   | Vector processors, GPUs                                         |
| **MISD** | _Multiple Instruction, Single Data_   | Rare (fault-tolerant systems)                                   |
| **MIMD** | _Multiple Instruction, Multiple Data_ | Multicore CPUs, clusters                                        |
### MIMD vs SIMD

- SIMD → All processing elements execute the **same** instruction.
- MIMD → Each processor executes **its own** instructions, possibly on different data.

Modern systems (like your laptop) often **combine** both:
- Each **CPU core** runs its own thread (MIMD),
- Each **core** uses SIMD instructions internally for speed.

## Example: Loop Vectorization

Here’s how SIMD is used by compilers:
### Non-SIMD (scalar)
`for (int i = 0; i < 4; i++) {     c[i] = a[i] + b[i]; }`
### SIMD (conceptually)
`load A[0..3] into vector register load B[0..3] into vector register C[0..3] = A[0..3] + B[0..3]`

That’s one vector instruction instead of 4 scalar ones.

Compilers like GCC, Clang, and MSVC can do this automatically with **auto-vectorization** — or you can use **intrinsics** to manually write SIMD code.