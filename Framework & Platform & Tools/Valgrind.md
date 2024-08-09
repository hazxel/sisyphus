Valgrind 默认运行 `memcheck` 工具，它是用于检测内存错误的主要工具：(默认 `--tool=memcheck`, 其他工具，如 `cachegrind` 和 `callgrind`，需要在命令行中指定)

> 编译时需要开启 debug 模式



# memcheck

- 基本内存检查: `valgrind ./myprogram`
- 内存泄漏检查: `valgrind --leak-check=yes ./myprogram`
- 详细的内存泄漏检查: `valgrind --leak-check=full --show-leak-kinds=all ./myprogram`



# cachegrind 

缓存性能检查: `valgrind --tool=cachegrind ./myprogram`

> 现代 CPU 多使用**Harvard 架构**，指令缓存（I-cache）和数据缓存（D-cache）在硬件上分离，提高指令和数据访问的并行性和效率。CPU 可以同时从 I-cache 取指令和从 D-cache 取数据，减少冲突和等待时间。此外指令和数据的访问路径独立，可以优化各自的缓存层级和大小。

- I-cache 指令缓存性能
- D-cache 数据缓存性能
- LL-cache 最后一级缓存 (Last Level Cache)，一般指 L3 cache (两级缓存时指 L2)

example output

```
==711969== I   refs:      128,300,433
==711969== I1  misses:      1,775,683
==711969== LLi misses:          2,513
==711969== I1  miss rate:        1.38%
==711969== LLi miss rate:        0.00%
==711969== 
==711969== D   refs:       65,949,943  (39,591,854 rd   + 26,358,089 wr)
==711969== D1  misses:         35,025  (    27,684 rd   +      7,341 wr)
==711969== LLd misses:         14,951  (     9,709 rd   +      5,242 wr)
==711969== D1  miss rate:         0.1% (       0.1%     +        0.0%  )
==711969== LLd miss rate:         0.0% (       0.0%     +        0.0%  )
==711969== 
==711969== LL refs:         1,810,708  ( 1,803,367 rd   +      7,341 wr)
==711969== LL misses:          17,464  (    12,222 rd   +      5,242 wr)
==711969== LL miss rate:          0.0% (       0.0%     +        0.0%  )
```

Cachegrind 同时生成一个包含详细分析数据的文件 `cachegrind.out.<pid>`。可用 `cg_annotate` 工具查看,显示**每个函数的**缓存使用统计数据，包括指令缓存、数据缓存和最后一级缓存的命中和未命中情况。



# callgrind

用于分析程序的调用图和性能

- 运行: `valgrind --tool=callgrind ./myprogram`
- 查看报告: `callgrind_annotate callgrind.out.<pid>`
- 图形化查看: `kcachegrind callgrind.out.<pid>`



# Massif

用于分析程序的堆内存使用情况。



# Helgrind

用于检测多线程程序中的数据竞争。



# DRD

另一个用于检测多线程程序中的数据竞争的工具。