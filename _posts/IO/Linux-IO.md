[TOC]

## Linux IO体系

### 读数据

#### 1. read

阻塞读取

参数：read(文件fd,，读取长度，用户态内存buffer)

缺点：

1. 不允许指定文件读取的起始位置
2. 阻塞式IO
3. 需要有一次内核态cache到用户态buffer的拷贝

#### 2. direct IO

操作系统直接把数据从磁盘读取到用户指定的用户态buffer中，不需要先读到内核cache，可以减少一次内存拷贝。但是，缺点在于，不使用内核cache，那么就不能使用操作系统的内存管理。针对同一份数据，如果需要再次读取，也只能从磁盘再进行一次IO。

#### 3. pread

解决问题：可以指定文件读取的起始位置

read调用时：

```
read
-------------
mutex_lock
lseek() // 用来跳转到指定的文件起始位置
read()
mutex_unlock
-------------
1. 需要使用lseek()系统调用来切换文件位置，所以两次系统调用
2. lseek()时，需要对整段加锁，效率不高
```

替换为pread()

```
pread(文件fd，起始位置，读取长度，用户态buffer) //等同于上述read代码，不用两次系统调用，不用加锁
```

#### 4. readahead

当我们需要进行多次IO读取时，多次调用read，会多次阻塞在IO上，性能较低。

示例：

```
while (ptr in ptrCollection) {
	read(ptr);
}
// 会导致频繁阻塞在read方法上
```

readahead()是一个异步方法，该方法只是通知操作系统，我需要读取这个文件，是一个预读操作。操作系统会把对应的数据读取到内核cache，当我们read读取时，就只需要从内核态拷贝到用户态这么一个步骤，不用再频繁阻塞在IO上。

使用readahend()进行优化

```
// 使用readahead进行预读优化，等再次调用read时，数据大部分已经在内核cache中，不用频繁IO阻塞
while (ptr in ptrCollection) {
	readahead(ptr);
}
while (ptr in ptrCollection) {
	read(ptr);
}
```

#### 5. mmap

mmap可以将文件的磁盘地址直接映射到内核态的页表上，然后将页表指针返回到用户态。所以，用户程序可以直接操作该指针进行文件读取，减少了内核cache到用户态cache的一次拷贝。

**优点：**

1. 节约一次数据拷贝

2. 不同于direct IO，mmap的内核态页表映射，也能够使用内核cache

**缺点：**

1. 由于用户直接使用地址指针操作，操作系统无法感知是否存在访问，所以针对于LRU的内存管理来说，无法更新热度，容易被挤出内存。不过可以在调用前通过readahead()方法，标记一下访问，减缓这个问题。
2. 页表缺失：如果用户访问的页面地址时，文件还没有映射上来。操作系统会对页表进行加锁，互斥比较严重

​	

### 写数据

#### 1. write()

将数据写入到磁盘，阻塞调用

**过程：**

1. 将写buffer拆分成页（一般4k）

2. 将buffer内容拷贝到内核cache

3. 将cache中对应页标记为脏页，并计算脏页比例

4. 如果脏页比例大于dirty_ratio（一般默认40%），开始***同步***写脏页（阻塞write），直到降低到dirty_ratio阈值

5. 如果脏页比例大于dirty_background_ratio（一般默认10%），唤醒后台线程异步写4MB脏页。write返回（不阻塞write）

   > Notes: 此外，系统还会以dirty_write_centisecs周期，定期唤醒kupdate线程，扫描所有打开着的inode。对变脏时间超过dirty_expire_centisecs的inode，会将该inode上所有脏页刷新到磁盘。

   





