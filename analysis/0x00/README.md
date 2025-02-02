boltdb在启动时，通过mmap系统调用将数据库文件映射到进程的虚拟内存中，这样可以就可以直接
对内存进行读写实现对文件的修改，而数据的落盘工作交给OS负责，只有在事务提交或更新
元数据时，才会通过fdatasync系统调用强制将脏页落盘。

内存与磁盘的数据交互是以页为单位。为了提升读写效率，boltdb的数据库文件也是按照页
来组织，且页的大小和文件系统的页大小一致。

由于mmap与unmmap系统调用的开销成本较大，所有boltdb在每次mmap时会多申请一部分空间
(小于1GB时倍增申请，大于1GB时额为申请1GB)，这样就可以产生一些空闲页。同时，随着对数
据库的操作，在更新值或删除值时，数据库也可能会产生空闲页。这些空闲页并不会立即被释放，
而是通过一个空闲页列表进行临时保存，供其它数据使用，以此减少系统开销。