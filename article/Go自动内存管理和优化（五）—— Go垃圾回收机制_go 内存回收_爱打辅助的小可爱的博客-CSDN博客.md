# Go自动内存管理和优化（五）—— Go垃圾回收机制_go 内存回收_爱打辅助的小可爱的博客-CSDN博客
[Go 自动内存管理和优化（五）—— Go 垃圾回收机制_go 内存回收_爱打辅助的小可爱的博客 - CSDN 博客](https://blog.csdn.net/qq_54353206/article/details/128908583) 

## 一、本次学习重点内容：

#### 1、Go 的自动[内存管理](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86&spm=1001.2101.3001.7020)

#### 2、Go 的分代 GC

## 二、详细知识点介绍：

### 1、自动内存管理——简介

#### [动态内存](https://so.csdn.net/so/search?q=%E5%8A%A8%E6%80%81%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)：

> 程序在运行时根据需求动态分配的内存: malloc( )

#### 自动内存管理 (垃圾回收):

> 由程序语言的运行时系统管理动态内存
>
> **目的：** 
>
> 1、避免手动内存管理，专注于实现业务逻辑
>
> 2、保证内存使用的正确性和安全性: double-freeproblem, use-after-free problem

#### 三个任务：

> 1、为新对象分配空间
>
> 2、找到存活对象
>
> 3、回收死亡对象的内存空间

### 2、自动内存管理——相关概念：

#### [GC 算法](https://so.csdn.net/so/search?q=GC%E7%AE%97%E6%B3%95&spm=1001.2101.3001.7020)：

_Mutator_: 业务线程，分配新对象，修改对象指向关系

_Collector_: GC 线程，找到存活对象，回收死亡对象的内存空间

_Serial GC_: 只有一个 collector

_Parallel GC_: 支持多个 collectors 同时回收的 GC 算法

_Concurrent GC_: mutator(s) 和 collector(s) 可以同时执行，**Collectors 必须感知对象指向关系的改变!**

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/6da76df5-4320-4c4e-918b-5bf6eaa65474.png?raw=true)

#### GC 算法评价：

_安全性 (Safety)_：不能回收存活的对象 **基本要求**

_吞吐率 (Throughput)_：**花在 GC 上的时间**

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/753dfc83-f0e7-44c1-a7e0-033e542a32df.png?raw=true)

_暂停时间 (Pause time)_：stop the world (STW) **业务是否感知**

_内存开销 (Space overhead)_ **GC 元数据开销**

#### 跟踪垃圾回收：

对象被回收的条件：**指针指向关系不可达的对象**

标记根对象：

> 根对象：静态变量、全局变量、常量、线程栈等

标记：**找到可达对象**

> 求指针指向关系的传递闭包: 从根对象出发，找到所有可达对象

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/7dd47555-1a44-482c-8be6-76f507095356.png?raw=true)

清理：**所有不可达对象**

> 步骤：
>
> 1、将存活对象复制到另外的内存空间 (Copying GC)
>
> 2、将死亡对象的内存标记为 “可分配”(Mark-sweep GC)
>
> 3、移动并整理存活对象 (Mark-compact GC)

copying GC：![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/4862e877-aedc-4d8b-9891-9d0d970ad611.png?raw=true)

Mark-sweep GC: **使用 free list 管理空闲内存**

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/401e2af3-1385-462b-95ae-8461608c09d8.png?raw=true)

Mark-compact GC: **原地整理**

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/75588823-0cba-4b5f-9449-36e76036fb6a.png?raw=true)

### 3、分代 GC

分代假说 (Generational hypothesis): _most objects die young_

现状：_很多对象在分配出来后很快就不再使用了_

每个对象都有年龄：_经历过 GC 的次数_

目的：

> 对年轻和老年的对象，制定不同的 GC 策略，降低整体内存管理的开销

#### 不同年龄的对象处于 heap 的不同区域：

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/16a71821-8620-41a7-86fb-b16e9b05fa44.png?raw=true)

年轻代 (Young generation):

> 常规的对象分配
>
> 由于存活对象很少，可以采用 copying collection
>
> GC 吞吐率很高

老年代 (Old generation):

> 对象趋向于一直活着, 反复复制开销较大
>
> 可以采用 mark-sweep collection

#### 引用计数：

每个对象都有一个与之关联的引用数目。

对象存活的条件：**当且仅当引用数大于 0**

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/619fac1c-f4a6-4442-9a8b-f5e8715c512b.png?raw=true)

**优点：** 

1、内存管理的操作被平摊到程序执行过程中。

2、内存管理不需要了解 runtime 的实现细节: C+＋智能指针 (smart pointer)。

**缺点：** 

1、维护引用计数的开销较大：_通过原子操作保证对引用计数操作的原子性和可见性_。

2、无法回收**环形数据结构**——weak reference

3、内存开销: 每个对象都引入的额外内存空间存储引用数目。

4、回收内存时依然可能引发暂停。

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-6-30%2009-59-45/9243c184-09ef-424c-a99b-efd45a576b3d.png?raw=true)
