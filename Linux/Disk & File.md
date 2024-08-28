### 块设备

块设备一般指磁盘。



### 缓冲区首部

Linux 使用 `struct buffer_head` 结构体来描述块设备的缓存，以管理磁盘上的数据块和与其相关的内存页。



# VFS



### page cache vs buffer cache

The page cache caches pages of files to optimize file I/O. The buffer cache caches disk blocks to optimize block I/O.

Before Linux 2.4, the two caches were distinct: Files were in the page cache, disk blocks were in the buffer cache. Given that most files are represented by a filesystem on a disk, data was represented twice.

Starting from Linux 2.4, the contents of the two caches were unified. If cached data has both a file and a block representation—as most data does—the buffer cache will simply point into the page cache. However, the buffer cache remains, as the kernel still needs to perform block I/O in terms of blocks. Because a small amount of block data isn't file backed (e.g. metadata and raw block I/O).

VFS 和 各种文件系统以 block 为单位组织磁盘数据。
在 Linux 2.4 以前，存在两种不同的 disk cache: page cache 和 buffer cache，前者存放访问**磁盘文件**时生成的磁盘数据页，后者是通过 VFS 访问 block 的数据。
为减少冗余，从 Linux 2.4.10 开始，buffer cache 就不存在了。 不再单独分配块缓冲区，相反，把它们存放在叫做 “缓冲区页” 的专门页中，而缓冲区 页保存在页高速缓存中。
缓冲区页在形式上就是与称做 “缓冲区首部”的附加描述符相关的数据页，其主要目 的是快速确定页中的一个块在磁盘中的地址。实际上，page cache 内的页中的 一大块数 据 在 磁 盘 上的 地 址 不 一定 是 相 邻 的 。



### dirty page

