## 虚拟地址
为了充分利用和管理系统内存资源，Linux采用虚拟内存管理技术，利用虚拟内存技术让每个进程都有`4GB` 互不干涉的虚拟地址空间。
进程初始化分配和操作的都是基于这个「虚拟地址」，只有当进程需要实际访问内存资源的时候才会建立虚拟地址和物理地址的映射，调入物理内存页。
#### 虚拟地址的好处

- 避免用户直接访问物理内存地址，防止一些破坏性操作，保护操作系统
- 每个进程都被分配了4GB的虚拟内存，用户程序可使用比实际物理内存更大的地址空间

`4GB` 的进程虚拟地址空间被分成两部分：「用户空间」和「内核空间」
![](https://cdn.nlark.com/yuque/0/2020/png/515219/1593589648246-c1e50a60-7ad7-4cce-9c35-40000b300ec3.png#align=left&display=inline&height=321&margin=%5Bobject%20Object%5D&originHeight=321&originWidth=322&size=0&status=done&style=none&width=322)
## 物理地址
不管是用户空间还是内核空间，使用的地址都是虚拟地址，当需进程要实际访问内存的时候，会由内核的「请求分页机制」产生「缺页异常」调入物理内存页。
把虚拟地址转换成内存的物理地址，这中间涉及利用mmu内存管理单元（memory management unit）对虚拟地址分段和分页（段页式）地址转换，关于分段和分页的具体流程，这里不再赘述，可以参考任何一本计算机组成原理教材。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593589966219-b9c8b907-848f-4ac3-becc-303bf1a9ec12.png#align=left&display=inline&height=88&margin=%5Bobject%20Object%5D&name=image.png&originHeight=88&originWidth=931&size=22631&status=done&style=none&width=931)
`Linux` 内核会将物理内存分为3个管理区，分别是：
### ZONE_DMA
`DMA`内存区域。包含0MB~16MB之间的内存页框，可以由老式基于`ISA`的设备通过`DMA`使用，直接映射到内核的地址空间。
### ZONE_NORMAL
普通内存区域。包含16MB~896MB之间的内存页框，常规页框，直接映射到内核的地址空间。
### ZONE_HIGHMEM
高端内存区域。包含896MB以上的内存页框，不进行直接映射，可以通过永久映射和临时映射进行这部分内存页框的访问。
![](https://cdn.nlark.com/yuque/0/2020/png/515219/1593592614169-883721f2-a527-44ca-b026-e6be2bf62625.png#align=left&display=inline&height=331&margin=%5Bobject%20Object%5D&originHeight=331&originWidth=261&size=0&status=done&style=none&width=261)
物理内存区划分
## 用户空间
用户进程能访问的是「用户空间」，每个进程都有自己独立的用户空间，虚拟地址范围从从 `0x00000000` 至 `0xBFFFFFFF` 总容量3G 。
用户进程通常只能访问用户空间的虚拟地址，只有在执行内陷操作或系统调用时才能访问内核空间。
### 进程与内存
进程（执行的程序）占用的用户空间按照「 访问属性一致的地址空间存放在一起 」的原则，划分成 `5`个不同的内存区域。访问属性指的是“可读、可写、可执行等 。

- 代码段
代码段是用来存放可执行文件的操作指令，可执行程序在内存中的镜像。代码段需要防止在运行时被非法修改，所以只准许读取操作，它是不可写的。

- 数据段
数据段用来存放可执行文件中已初始化全局变量，换句话说就是存放程序静态分配的变量和全局变量。

- BSS段
`BSS`段包含了程序中未初始化的全局变量，在内存中 `bss` 段全部置零。

- 堆 `heap`
堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。当进程调用malloc等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；当利用free等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）

- 栈 `stack`
栈是用户存放程序临时创建的局部变量，也就是函数中定义的变量（但不包括 `static` 声明的变量，static意味着在数据段中存放变量）。除此以外，在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。由于栈的先进先出特点，所以栈特别方便用来保存/恢复调用现场。从这个意义上讲，我们可以把堆栈看成一个寄存、交换临时数据的内存区。


上述几种内存区域中数据段、`BSS` 段、堆通常是被连续存储在内存中，在位置上是连续的，而代码段和栈往往会被独立存放。堆和栈两个区域在 `i386` 体系结构中栈向下扩展、堆向上扩展，相对而生。
![](https://cdn.nlark.com/yuque/0/2020/png/515219/1593592866312-a19b8f6b-4d3d-4da2-89f0-6be359b52c2f.png#align=left&display=inline&height=316&margin=%5Bobject%20Object%5D&originHeight=316&originWidth=121&size=0&status=done&style=none&width=121)
你也可以在linux下用`size` 命令查看编译后程序的各个内存区域大小：
```
[lemon ~]# size /usr/local/sbin/sshd
   text	   data	    bss	    dec	    hex	filename
1924532	  12412	 426896	2363840	 2411c0	/usr/local/sbin/sshd
```
## 内核空间
在 `x86 32` 位系统里，Linux 内核地址空间是指虚拟地址从 `0xC0000000` 开始到 `0xFFFFFFFF` 为止的高端内存地址空间，总计 `1G` 的容量， 包括了内核镜像、物理页面表、驱动程序等运行在内核空间 。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593594396235-0f59c417-b1b5-4b54-aa2e-262c1a485887.png#align=left&display=inline&height=306&margin=%5Bobject%20Object%5D&name=image.png&originHeight=409&originWidth=299&size=20853&status=done&style=none&width=224)
#### 直接映射区
直接映射区 `Direct Memory Region`：从内核空间起始地址开始，最大`896M`的内核空间地址区间，为直接内存映射区。
直接映射区的896MB的「线性地址」直接与「物理地址」的前`896MB`进行映射，也就是说线性地址和分配的物理地址都是连续的。内核地址空间的线性地址`0xC0000001`所对应的物理地址为`0x00000001`，它们之间相差一个偏移量`PAGE_OFFSET = 0xC0000000`
该区域的线性地址和物理地址存在线性转换关系「线性地址 = `PAGE_OFFSET` + 物理地址」也可以用 `virt_to_phys()`函数将内核虚拟空间中的线性地址转化为物理地址。
#### 高端内存线性地址空间
内核空间线性地址从 896M 到 1G 的区间，容量 128MB 的地址区间是高端内存线性地址空间，为什么叫高端内存线性地址空间？下面给你解释一下：
前面已经说过，内核空间的总大小 1GB，从内核空间起始地址开始的 896MB 的线性地址可以直接映射到物理地址大小为 896MB 的地址区间。
退一万步，即使内核空间的1GB线性地址都映射到物理地址，那也最多只能寻址 1GB 大小的物理内存地址范围。
请问你现在你家的内存条多大？快醒醒都 0202 年了，一般 PC 的内存都大于 1GB 了吧！
所以，内核空间拿出了最后的 128M 地址区间，划分成下面三个高端内存映射区，以达到对整个物理地址范围的寻址。而在 64 位的系统上就不存在这样的问题了，因为可用的线性地址空间远大于可安装的内存。
##### 动态内存映射区
`vmalloc Region` 该区域由内核函数`vmalloc`来分配，特点是：线性空间连续，但是对应的物理地址空间不一定连续。`vmalloc` 分配的线性地址所对应的物理页可能处于低端内存，也可能处于高端内存。
##### 永久内存映射区
`Persistent Kernel Mapping Region` 该区域可访问高端内存。访问方法是使用 `alloc_page (_GFP_HIGHMEM)` 分配高端内存页或者使用`kmap`函数将分配到的高端内存映射到该区域。
##### 固定映射区
`Fixing kernel Mapping Region` 该区域和 4G 的顶端只有 4k 的隔离带，其每个地址项都服务于特定的用途，如 `ACPI_BASE` 等。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593596212052-68c592b4-2cfa-4274-90a7-5f7157ceb812.png#align=left&display=inline&height=213&margin=%5Bobject%20Object%5D&name=image.png&originHeight=438&originWidth=753&size=43510&status=done&style=none&width=367)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593596275531-ee8f9985-d5f3-486b-9e63-877be0d6a282.png#align=left&display=inline&height=149&margin=%5Bobject%20Object%5D&name=image.png&originHeight=414&originWidth=1019&size=48480&status=done&style=none&width=367)
## 内存数据结构
要让内核管理系统中的虚拟内存，必然要从中抽象出内存管理数据结构，内存管理操作如「分配、释放等」都基于这些数据结构操作，这里列举两个管理虚拟内存区域的数据结构。
### 用户空间内存数据结构
在前面「进程与内存」章节我们提到，Linux进程可以划分为 5 个不同的内存区域，分别是：代码段、数据段、`BSS`、堆、栈，内核管理这些区域的方式是，将这些内存区域抽象成`vm_area_struct`的内存管理对象。
`vm_area_struct`是描述进程地址空间的基本管理单元，一个进程往往需要多个`vm_area_struct`来描述它的用户空间虚拟地址，需要使用「链表」和「红黑树」来组织各个`vm_area_struct`。
链表用于需要遍历全部节点的时候用，而红黑树适用于在地址空间中定位特定内存区域。内核为了内存区域上的各种不同操作都能获得高性能，所以同时使用了这两种数据结构。
用户空间进程的地址管理模型：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593596724432-dfa4a7dc-8c2c-4eec-97c2-cace8e0d996b.png#align=left&display=inline&height=271&margin=%5Bobject%20Object%5D&name=image.png&originHeight=362&originWidth=367&size=27473&status=done&style=none&width=275)
### 内核空间动态分配内存数据结构
在内核空间章节提到过「动态内存映射区」，该区域由内核函数vmalloc来分配，特点是：线性空间连续，但是对应的物理地址空间不一定连续。vmalloc分配的线性地址所对应的物理页可能处于低端内存，也可能处于高端内存。
vmalloc 分配的地址则限于vmalloc_start与vmalloc_end之间。每一块vmalloc分配的内核虚拟内存都对应一个vm_struct结构体，不同的内核空间虚拟地址之间有4k大小的防越界空闲区间隔区。
与用户空间的虚拟地址特征一样，这些虚拟地址与物理内存没有简单的映射关系，必须通过内核页表才可转换物理地址或物理页，它们有可能尚未被映射，当发生缺页时才真正分配物理页面。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593606887811-ebf196ea-ef71-471c-8c8e-193900babd8a.png#align=left&display=inline&height=295&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=787&size=38511&status=done&style=none&width=567)
前面分析了 Linux 内存管理机制，下面深入学习**物理内存管理和虚拟内存分配。**
通过前面的学习我们知道，程序可没这么好骗，任你内存管理把虚拟地址空间玩出花来，到最后还是要给程序实实在在的物理内存，不然程序就要罢工了。
所以物理内存这么重要的资源一定要好好管理起来使用（物理内存，就是你实实在在的内存条），那么内核是如何管理物理内存的呢？
## 物理内存管理
在linux系统中通过分段和分页机制，把物理内存划分4k大小的内存页page（也称作页框page frame），物理内存的分布和回收都是基于内存页进行，把物理内存分页管理的好处很多。
加入系统请求小块内存，可以预先分配一页给它，避免了反复的申请和释放小块内存带来频繁的系统开销。
假如系统需要大块内存，则可以用多页内存拼凑，而不必要求大块连续内存。你看不管内存大小都能收放自如，分页机制多么完美的解决方案！
but，理想很丰满，现实很骨感。如果就直接这样把内存分页使用，不再加额外的管理还是存在问题的，下面，我们来看下，系统在多次分配和释放物理页的时候会遇到哪些问题。
### 物理页管理面临问题
物理内存页分配会出现外部碎片和内部碎片问题，所谓的「内部」和「外部」是针对「页框内外」而言，一个页框内的内存碎片是内部碎片，多个页框间的碎片是外部碎片。
**外部碎片**
当需要分配大块内存的时候，要用好几页组合起来才够，而系统分配物理内存页的时候会尽量分配连续的内存页面，频繁的分配与回收物理页导致大量的小块内存夹杂在已分配页面中间，形成外部碎片，例如：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593608258881-f09558f5-73f2-4571-bdf6-dcef7a7271ef.png#align=left&display=inline&height=286&margin=%5Bobject%20Object%5D&name=image.png&originHeight=381&originWidth=421&size=13783&status=done&style=none&width=316)
**内部碎片**
物理内存是按页来分配的，这样当实际只需要很小内存的时候，也会分配至少4k大小的页面，而内核中有很多需要以字节为单位分配内存的场景，这样本来只想要几个字节而已却不得不分配一页内存，除去用掉的字节剩下的就形成了内部碎片。
方法总比困难多，因为存在上面的这些问题，聪明的程序员灵机一动，引入页面管理算法来解决上述的碎片问题。
**buddy（伙伴）分配算法**
linux 内核引入伙伴系统算法（buddy system），什么意识呢？就是把相同大小的页框用链表串起来，页框块就像手拉手的好伙伴，也是这个算法的名字的由来。
具体的，所有的空闲页框分组为11个块链表，每个块链表分别包含大小为1,2,4,8,16,32,64,128,256,512,1024个连续页框的页框块。最大可以申请1024个连续页框，对应4mb大小的连续内存。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593608898092-af9ec250-e8b5-402c-9431-be6d7cff9d11.png#align=left&display=inline&height=218&margin=%5Bobject%20Object%5D&name=image.png&originHeight=291&originWidth=316&size=7593&status=done&style=none&width=237)
因为任何正整数都可以由2^n的和组成，所以总能找到合适大小的内存块分配出去减少了外部碎片产生。
**分配实例**
比如：我需要申请4个页框，但是长度为4个连续页框块链表没有空闲的页框块，伙伴系统会从连续8个页框块的链表获取一个，并将其拆分为两个连续4个页框块，取其中一个，另一个放入连续4个页框块的空闲链表中。释放的时候会检查，释放的这几个页框后的页框是否空闲，能否组成下一级长度的块。
```
[lemon]]# cat /proc/buddyinfo 
Node 0, zone      DMA      1      0      0      0      2      1      1      0      1      1      3 
Node 0, zone    DMA32   3198   4108   4940   4773   4030   2184    891    180     67     32    330 
Node 0, zone   Normal  42438  37404  16035   4386    610    121     22      3      0      0      1
```
**slab分配器**
看到这里你可能会想，有了伙伴系统这下总可以管理好物理内存了吧？不，还不够，否则就没有slab分配器什么事了。
什么事slab分配器？
一般来说，内核对象的生命周期是这样的：分配内存-初始化-释放内存，内核中有大量的小对象，比如文件描述结构对象、任务描述结构对象，如果按照伙伴系统按页分配和释放内存，对小对象频繁的执行「分配内存-初始化-释放内存」会非常消耗性能。
伙伴系统分配出去的内存还是以页框为单位，而对于内核的很多场景都是分配小片内存，远不到一页内存大小的空间。slab分配器，**「通过将内存按使用对象不同再划分成不同大小的空间」**，应用于内核对象的缓存。伙伴系统和slab不是二选一的关系，slab内存分配器是对伙伴分配算法的补充。
**大白话说原理**
对于每个内核中的相同类型的对象，如：task_struct、file_struct等需要重复使用的小型内核数据对象，都会有个slab缓存池，缓存住大量常用的「已经初始化」的对象，每当要申请这种类型的对象时，就从缓存池中的slab列表中分配一个出去；而当要释放时，将其重新保存在该列表中，而不是直接返回给伙伴系统，从而避免内部碎片，同时也大大提高了内存分配性能。
**数据结构**
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614334639-a18ff7d1-8d1b-4650-bf2e-61be9d1e87f8.png#align=left&display=inline&height=188&margin=%5Bobject%20Object%5D&name=image.png&originHeight=251&originWidth=633&size=24224&status=done&style=none&width=475)
`kmem_cache` 是一个`cache_chain` 的链表组成节点，代表的是一个内核中的相同类型的「对象高速缓存」，每个`kmem_cache` 通常是一段连续的内存块，包含了三种类型的 `slabs` 链表：

- `slabs_full` (完全分配的 `slab` 链表)
- `slabs_partial` (部分分配的`slab` 链表)
- `slabs_empty` ( 没有被分配对象的`slab` 链表)

`kmem_cache` 中有个重要的结构体 `kmem_list3` 包含了以上三个数据结构的声明。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614367122-ab19023a-5482-478b-9847-a0ad4a0346c1.png#align=left&display=inline&height=292&margin=%5Bobject%20Object%5D&name=image.png&originHeight=557&originWidth=1080&size=239690&status=done&style=none&width=567)
`slab` 是`slab` 分配器的最小单位，在实现上一个 `slab` 由一个或多个连续的物理页组成（通常只有一页）。单个slab可以在 `slab` 链表之间移动，例如如果一个「半满`slabs_partial`链表」被分配了对象后变满了，就要从 `slabs_partial` 中删除，同时插入到「全满`slabs_full`链表」中去。内核`slab`对象的分配过程是这样的：

1. 如果`slabs_partial`链表还有未分配的空间，分配对象，若分配之后变满，移动 `slab` 到`slabs_full` 链表
2. 如果`slabs_partial`链表没有未分配的空间，进入下一步
3. 如果`slabs_empty` 链表还有未分配的空间，分配对象，同时移动`slab`进入`slabs_partial`链表
4. 如果`slabs_empty`为空，请求伙伴系统分页，创建一个新的空闲`slab`， 按步骤 3 分配对象

![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614396572-3f99d3ef-f777-4099-bdf9-d0f4a4dea471.png#align=left&display=inline&height=256&margin=%5Bobject%20Object%5D&name=image.png&originHeight=341&originWidth=461&size=24279&status=done&style=none&width=346)
##### 命令查看
上面说的都是理论，比较抽象，动动手来看看系统中的 slab 吧！你可以通过 `cat /proc/slabinfo` 命令，实际查看系统中`slab` 信息。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614479901-7f503632-45dc-4f5e-928f-1327c169ba71.png#align=left&display=inline&height=114&margin=%5Bobject%20Object%5D&name=image.png&originHeight=217&originWidth=1080&size=162928&status=done&style=none&width=567)
`slabtop`  实时显示内核 slab 内存缓存信息。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614559843-2f066201-2400-4c64-8b1a-228b4d5d16c9.png#align=left&display=inline&height=213&margin=%5Bobject%20Object%5D&name=image.png&originHeight=213&originWidth=577&size=35867&status=done&style=none&width=577)
#### slab高速缓存的分类
slab高速缓存分为两大类，「通用高速缓存」和「专用高速缓存」。
##### 通用高速缓存
slab分配器中用 `kmem_cache` 来描述高速缓存的结构，它本身也需要 slab 分配器对其进行高速缓存。cache_cache 保存着对「高速缓存描述符的高速缓存」，是一种通用高速缓存，保存在`cache_chain` 链表中的第一个元素。
另外，slab 分配器所提供的小块连续内存的分配，也是通用高速缓存实现的。通用高速缓存所提供的对象具有几何分布的大小，范围为32到131072字节。内核中提供了 `kmalloc()` 和 `kfree()` 两个接口分别进行内存的申请和释放。
##### 专用高速缓存
内核为专用高速缓存的申请和释放提供了一套完整的接口，根据所传入的参数为指定的对象分配slab缓存。
###### 专用高速缓存的申请和释放
kmem_cache_create() 用于对一个指定的对象创建高速缓存。它从 cache_cache 普通高速缓存中为新的专有缓存分配一个高速缓存描述符，并把这个描述符插入到高速缓存描述符形成的 cache_chain 链表中。kmem_cache_destory() 用于撤消和从 cache_chain 链表上删除高速缓存。
##### slab的申请和释放
`slab` 数据结构在内核中的定义，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614595843-a9a74383-6263-4616-8129-647d006256f2.png#align=left&display=inline&height=480&margin=%5Bobject%20Object%5D&name=image.png&originHeight=480&originWidth=1080&size=173882&status=done&style=none&width=1080)
kmem_cache_alloc() 在其参数所指定的高速缓存中分配一个slab，对应的 kmem_cache_free() 在其参数所指定的高速缓存中释放一个slab。
**虚拟内存分配**
前面讨论的都是对物理内存的管理，Linux 通过虚拟内存管理，欺骗了用户程序假装每个程序都有 4G 的虚拟内存寻址空间（如果这里不懂我说啥，建议回头看下 [别再说你不懂Linux内存管理了，10张图给你安排的明明白白！](https://mp.weixin.qq.com/s?__biz=MzIwMjM4NDE1Nw==&mid=2247483865&idx=1&sn=dfa63a467b620b6131acaef9ea6874a3&chksm=96de37aba1a9bebdcc097314f40ae633bd393253759ecd970ac84cdb41f18994da9405dbf356&token=1178579599&lang=zh_CN&scene=21#wechat_redirect)）。
所以我们来研究下虚拟内存的分配，这里包括用户空间虚拟内存和内核空间虚拟内存。
**注意，分配的虚拟内存还没有映射到物理内存，只有当访问申请的虚拟内存时，才会发生缺页异常，再通过上面介绍的伙伴系统和 slab 分配器申请物理内存。**
### 用户空间内存分配
#### malloc
`malloc` 用于申请用户空间的虚拟内存，当申请小于 `128KB` 小内存的时，`malloc`使用  `sbrk或brk` 分配内存；当申请大于 `128KB` 的内存时，使用 `mmap` 函数申请内存；
##### 存在问题
由于 `brk/sbrk/mmap` 属于系统调用，如果每次申请内存都要产生系统调用开销，`cpu` 在用户态和内核态之间频繁切换，非常影响性能。
而且，堆是从低地址往高地址增长，如果低地址的内存没有被释放，高地址的内存就不能被回收，容易产生内存碎片。
##### 解决
因此，`malloc`采用的是内存池的实现方式，先申请一大块内存，然后将内存分成不同大小的内存块，然后用户申请内存时，直接从内存池中选择一块相近的内存块分配出去。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614677161-eac265b5-3cc3-4a60-8623-be1300669bf9.png#align=left&display=inline&height=214&margin=%5Bobject%20Object%5D&name=image.png&originHeight=285&originWidth=681&size=22034&status=done&style=none&width=511)
### 内核空间内存分配
在讲内核空间内存分配之前，先来回顾一下内核地址空间。`kmalloc` 和 `vmalloc` 分别用于分配不同映射区的虚拟内存，看这张上次画的图：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593614833569-f6972b7b-1b62-4e99-99a9-1418806a4b93.png#align=left&display=inline&height=306&margin=%5Bobject%20Object%5D&name=image.png&originHeight=409&originWidth=299&size=20853&status=done&style=none&width=224)
#### kmalloc
`kmalloc()` 分配的虚拟地址范围在内核空间的「直接内存映射区」。
按字节为单位虚拟内存，一般用于分配小块内存，释放内存对应于 `kfree` ，可以分配连续的物理内存。函数原型在 `<linux/kmalloc.h>` 中声明，一般情况下在驱动程序中都是调用 `kmalloc()` 来给数据结构分配内存 。
还记得前面说的 slab 吗？`kmalloc` 是基于slab 分配器的 ，同样可以用`cat /proc/slabinfo` 命令，查看 `kmalloc` 相关 `slab`   对象信息，下面的 kmalloc-8、kmalloc-16 等等就是基于slab分配的 kmalloc 高速缓存。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593615283170-ecafc205-84a1-4697-a058-1db04fd2b947.png#align=left&display=inline&height=225&margin=%5Bobject%20Object%5D&name=image.png&originHeight=225&originWidth=764&size=42330&status=done&style=none&width=764)
#### vmalloc
`vmalloc` 分配的虚拟地址区间，位于 `vmalloc_start` 与`vmalloc_end` 之间的「动态内存映射区」。
一般用分配大块内存，释放内存对应于 `vfree`，分配的虚拟内存地址连续，物理地址上不一定连续。函数原型在 `<linux/vmalloc.h>` 中声明。一般用在为活动的交换区分配数据结构，为某些 `I/O` 驱动程序分配缓冲区，或为内核模块分配空间。
下面的图总结了上述两种内核空间虚拟内存分配方式。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/515219/1593615316409-d3ababe8-c8a1-42ca-8865-fae07b062dc6.png#align=left&display=inline&height=161&margin=%5Bobject%20Object%5D&name=image.png&originHeight=215&originWidth=747&size=21847&status=done&style=none&width=560)

原文链接：[https://mp.weixin.qq.com/s/yXbUnesmoyhDyfRrQAmKsw](https://mp.weixin.qq.com/s/yXbUnesmoyhDyfRrQAmKsw)
