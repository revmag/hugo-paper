+++
title = "SMs, Cores, Threads, Blocks, SMA, coalesced memory "
weight = 3
+++

When we use GPUs, and code it in CUDA - there are some things which we notice : threads and blocks, and how exactly are the functions running, like all the threads and all the blocks are running simultaneously or consecutively?

Start with SMs, and cores.
So Streaming Multiprocessor ( SM) - aka compute units - SIMD units. SM is like core of multi core CPU, it is a hardware building block
And Each SM contains a bunch of cores, each core 
( Like Kepler has 192 cores, Maxwell, Pascal, Ampere, Ada has 128 cores, and Turing, A100,A30 has 62 cores)

These cores are wrapped in bunches of 32, called warps, and example Kepler can dispatch 6 of these warps at once.( in parallel).
And all threads in one warp execute the same SIMD instructions, and run parallely.
And a single thread is executed on 1 CUDA core.

So each SM run in parallel, the execution order is random, and all the cores inside a SM, also run in parallel, at times using the same memory, the Shared Memory Access, near the L1 cache( each core does 1 thread at a time).

And let’s say, each thread makes 3 accesses, but they will happen in order, and for all the threads in a wrap, they will happen at the same time. 

If my matrix is laid out like this →

```css
0 1 2 3
4 5 6 7
8 9 a b
```

And we want to choose which configuration is better :
Config 1 :

```cpp
thread 0:  0, 1, 2
thread 1:  3, 4, 5
thread 2:  6, 7, 8
thread 3:  9, a, b
```

Config 2 :

```cpp
thread 0:  0, 4, 8
thread 1:  1, 5, 9
thread 2:  2, 6, a
thread 3:  3, 7, b
```

So config 2 is better, as all 4 threads will go at the same time, and memory address of 0,1,2,3 are consecutive, so it would be easier to fetch, and this is called memory coalescing.
”A coalesced memory transaction is one in which all of the threads in a half-warp access global memory at the same time. This is oversimple, but the correct way to do it is just have consecutive threads access consecutive memory addresses.”

https://www.reddit.com/r/CUDA/comments/x2f767/how_does_cuda_blockswarps_thread_works/

https://www.reddit.com/r/nvidia/comments/1celgf6/how_are_1024_threads_executed_in_a_thread_block/?rdt=54705

https://stackoverflow.com/questions/5041328/in-cuda-what-is-memory-coalescing-and-how-is-it-achieved