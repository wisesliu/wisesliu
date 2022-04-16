## 1.1 .线程


​	这里所说的线程指程序执行过程中的一个线程实体。JVM 允许一个应用并发执行多个线程。Hotspot JVM 中的 Java 线程与原生操作系统线程有直接的映射关系。当线程本地存储、缓冲区分配、同步对象、栈、程序计数器等准备好以后，就会创建一个操作系统原生线程。Java 线程结束，原生线程随之被回收。操作系统负责调度所有线程，并把它们分配到任何可用的 CPU 上。当原生线程初始化完毕，就会调用 Java 线程的 run() 方法。当线程结束时，会释放原生线程和 Java 线程的所有资源。

Hotspot JVM 后台运行的系统线程主要有下面几个：

| 类型                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| 虚拟机线程 （VM thread） | 这个线程等待 JVM 到达安全点操作出现。这些操作必须要在独立的线程里执行，因为当 堆修改无法进行时，线程都需要 JVM 位于安全点。这些操作的类型有：stop-theworld 垃圾回收、线程栈 dump、线程暂停、线程偏向锁（biased locking）解除。 |
| 周期性任务线程           | 这线程负责定时器事件（也就是中断），用来调度周期性操作的执行。 |
| GC 线程                  | 这些线程支持 JVM 中不同的垃圾回收活动。                      |
| 编译器线程               | 这些线程在运行时将字节码动态编译成本地平台相关的机器码。     |
| 信号分发线程             | 这个线程接收发送到 JVM 的信号并调用适当的 JVM 方法处理。     |

## 	1.2.JVM内存区域

![](../images/image-20220306175928192.png)



​	JVM 内存区域主要分为线程私有区域【程序计数器、虚拟机栈、本地方法区】、线程共享区 域【JAVA 堆、方法区】、直接内存。 线程私有数据区域生命周期与线程相同, 依赖用户线程的启动/结束 而 创建/销毁(在 Hotspot VM 内, 每个线程都与操作系统的本地线程直接映射, 因此这部分内存区域的存/否跟随本地线程的 生/死对应)。 13/04/2018 Page 22 of 283 线程共享区域随虚拟机的启动/关闭而创建/销毁。 直接内存并不是 JVM 运行时数据区的一部分, 但也会被频繁的使用: 在 JDK 1.4 引入的 NIO 提 供了基于 Channel 与 Buffer 的 IO 方式, 它可以使用 Native 函数库直接分配堆外内存, 然后使用 DirectByteBuffer 对象作为这块内存的引用进行操作(详见: Java I/O 扩展), 这样就避免了在 Java 堆和 Native 堆中来回复制数据, 因此在一些场景中可以显著提高性能。

![](../images/1.png)



### 1.2.1.程序计数器（线程私有）

​	一块较小的内存空间, 是当前线程所执行的字节码的行号指示器，每条线程都要有一个独立的程序计数器，这类内存也称为“线程私有”的内存。正在执行 java 方法的话，计数器记录的是虚拟机字节码指令的地址（当前指令的地址）。如果还是 Native 方法，则为空。

这个内存区域是唯一一个在虚拟机中没有规定任何OutOfMemoryError 情况的区域。

### 1.2.2.虚拟机栈（线程私有）

​	是描述java 方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧（Stack Frame） 用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

栈帧（ Frame）是用来存储数据和部分过程结果的数据结构，同时也被用来处理动态链接(Dynamic Linking)、 方法返回值和异常分派（ Dispatch Exception）。栈帧随着方法调用而创建，随着方法结束而销毁——无论方法是正常完成还是异常完成（抛出了在方法内未被捕获的异常）都算作方法结束。

![](../images/2.png)

### 1.2.3.本地方法区（线程私有）

​	本地方法区和Java Stack作用类似, 区别是虚拟机栈为执行Java方法服务, 而本地方法栈则为Native方法服务, 如果一个VM实现使用C-linkage模型来支持Native调用, 那么该栈将会是一个C栈，但HotSpot VM直接就把本地方法栈和虚拟机栈合二为一。

### 1.2.4.堆（Heap-线程共享）-运行时数据区

​	是被线程共享的一块内存区域，创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要的内存区域。由于现代VM采用分代收集算法, 因此Java堆从GC的角度还可以细分为:新生代(Eden区、From Survivor区和To Survivor区)和老年代。

### 1.2.5.方法区/永久代（线程共享）

​	即我们常说的永久代(Permanent  Generation), 用于存储被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据. HotSpot VM把GC分代收集扩展至方法区, 即使用Java堆的永久代来实现方法区, 这样HotSpot的垃圾收集器就可以像管理Java堆一样管理这部分内存, 而不必为方法区开发专门的内存管理器(永久带的内存回收的主要目标是针对常量池的回收和类型的卸载, 因此收益一般很小)。运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。Java虚拟机对Class文件的每一部分（自然也包括常量池）的格式都有严格的规定，每一个字节用于存储哪种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。

## 1.3.JVM运行时内存

​	Java堆从GC的角度还可以细分为:新生代(Eden区、From Survivor区和To Survivor区)和老年代。

![](../images/3.png)



### 1.3.1.新生代

​	是用来存放新生的对象。一般占据堆的 1/3 空间。由于频繁创建对象，所以新生代会频繁触发MinorGC 进行垃圾回收。新生代又分为 Eden 区、ServivorFrom、ServivorTo 三个区。

#### 1.3.1.Eden区

​	Java 新对象的出生地（如果新创建的对象占用内存很大，则直接分配到老年代）。当Eden 区内存不够的时候就会触发 MinorGC，对新生代区进行一次垃圾回收。

 ####  1.3.2.ServivorFrom

​	上一次 GC 的幸存者，作为这一次 GC 的被扫描者。

####  1.3.3.ServivorTo

​	保留了一次 MinorGC 过程中的幸存者。

#### 1.3.4.MinorGC 的过程（复制->清空->互换）

​	MinorGC 采用复制算法。

1：eden、servicorFrom 复制到ServicorTo，年龄+1首先，把Eden和ServivorFrom区域中存活的对象复制到ServicorTo区域（如果有对象的年龄以及达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1（如果ServicorTo不够位置了就放到老年区）；

2：清空eden、servicorFrom然后，清空Eden和ServicorFrom中的对象；

3：ServicorTo和ServicorFrom互换最后，ServicorTo和ServicorFrom互换，原ServicorTo成为下一次GC时的ServicorFrom区。



### 1.3.2.老年代

​	主要存放应用程序中生命周期长的内存对象。老年代的对象比较稳定，所以MajorGC不会频繁执行。在进行MajorGC前一般都先进行了一次MinorGC，使得有新生代的对象晋身入老年代，导致空间不够用时才触发。当无法找到足够大的连续空间分配给新创建的较大对象时也会提前触发一次MajorGC进行垃圾回收腾出空间。MajorGC采用标记清除算法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC的耗时比较长，因为要扫描再回收。MajorGC会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常。

### 1.3.3.永久代

​	指内存的永久保存区域，主要存放Class和Meta（元数据）的信息,Class在被加载的时候被放入永久区域，它和和存放实例的区域不同,GC不会在主程序运行期对永久区域进行清理。所以这也导致了永久代的区域会随着加载的Class的增多而胀满，最终抛出OOM异常。

#### 1.3.3.1.JAVA8与元数据

​	在Java8中，永久代已经被移除，被一个称为“元数据区”（元空间）的区域所取代。元空间的本质和永久代类似，元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本 地 内 存。因此，默认情况下，元空间的大小仅受本地内存限制。类 的 元 数 据 放 入native memory, 字符串池和类的静态变量放入java堆中，这样可以加载多少类的元数据就不再由MaxPermSize控制, 而由系统的实际可用空间来控制。



## 1.4.垃圾回收与算法

![](../images/4.png)

### 1.4.1.如何确定垃圾

#### 1.4.1.1.引用计数法

​	在Java中，引用和对象是有关联的。如果要操作对象则必须用引用进行。因此，很显然一个简单的办法是通过引用计数来判断一个对象是否可以回收。简单说，即一个对象如果没有任何与之关联的引用，即他们的引用计数都不为0，则说明对象不太可能再被用到，那么这个对象就是可回收对象。

#### 1.4.1.2.可达性分析

​	为了解决引用计数法的循环引用问题，Java使用了可达性分析的方法。通过一系列的“GC  roots”对象作为起点搜索。如果在“GC  roots”和一个对象之间没有可达路径，则称该对象是不可达的。要注意的是，不可达对象不等价于可回收对象，不可达对象变为可回收对象至少要经过两次标记过程。两次标记后仍然是可回收对象，则将面临回收。

### 1.4.2.标记清除算法（Mark-Sweep）

​	最基础的垃圾回收算法，分为两个阶段，标注和清除。标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。如图从图中我们就可以发现，该算法最大的问题是内存碎片化严重，后续可能发生大对象不能找到可利用空间的问题。2.4.3.复制算法（copying）为了解决Mark-Sweep算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清掉，如图：

![](../images/5.png)



从图中我们就可以发现，该算法最大的问题是内存碎片化严重，后续可能发生大对象不能找到可利用空间的问题。

 

### 1.4.3复制算法（copying）

 	为了解决 Mark-Sweep 算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清掉，如图：

![](../images/6.png)

​	这种算法虽然实现简单，内存效率高，不易产生碎片，但是最大的问题是可用内存被压缩到了原本的一半。且存活对象增多的话，Copying算法的效率会大大降低。

### 1.4.4.标记整理算法(Mark-Compact)

​	结合了以上两个算法，为了避免缺陷而提出。标记阶段和Mark-Sweep算法相同，标记后不是清理对象，而是将存活对象移向内存的一端。然后清除端边界外的对象。如图：

![](../images/7.png)



### 1.4.5.分代收集算法

​	分代收集法是目前大部分JVM 所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将 GC 堆划分为老生代(Tenured/Old Generation)和新生代(Young Generation)。老生代的特点是每次垃圾回收时只有少量对象需要被回收，新生代的特点是每次垃圾回收时都有大量垃圾需要被回收，因此可以根据不同区域选择不同的算法。

#### 1.4.1.1. 新生代与复制算法

​	目前大部分 JVM 的 GC 对于新生代都采取Copying 算法，因为新生代中每次垃圾回收都要回收大部分对象，即要复制的操作比较少，但通常并不是按照 1：1 来划分新生代。一般将新生代划分为一块较大的Eden 空间和两个较小的 Survivor 空间(From Space, To Space)，每次使用Eden 空间和其中的一块Survivor 空间，当进行回收时，将该两块空间中还存活的对象复制到另一块 Survivor 空间中。

![](../images/8.png)

#### 1.4.5.2.老年代与标记复制算法

​	而老年代因为每次只回收少量对象，因而采用Mark-Compact算法。

​	1.JAVA虚拟机提到过的处于方法区的永生代(Permanet Generation)，它用来存储class类，常量，方法描述等。对永生代的回收主要包括废弃常量和无用的类。

​	2.对象的内存分配主要在新生代的Eden Space和Survivor Space的From Space(Survivor目前存放对象的那一块)，少数情况会直接分配到老生代。

​	3.当新生代的Eden Space和From Space空间不足时就会发生一次GC，进行GC后，Eden Space和From Space区的存活对象会被挪到To Space，然后将Eden Space和From Space进行清理。

​	4.如果To Space无法足够存储某个对象，则将这个对象存储到老生代。

​	5.在进行GC后，使用的便是Eden Space和To Space了，如此反复循环。

​	6.当对象在Survivor区躲过一次GC后，其年龄就会+1。默认情况下年龄到达15的对象会被移到老生代中。

## 1.5.JAVA 四中引用类型

### 1.5.1.强引用

​	在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到JVM也不会回收。因此强引用是造成Java内存泄漏的主要原因之一。

### 1.5.2.软引用软

​	引用需要用SoftReference类来实现，对于只有软引用的对象来说，当系统内存足够时它不会被回收，当系统内存空间不足时它会被回收。软引用通常用在对内存敏感的程序中。

### 1.5.3.弱引用

​	弱引用需要用WeakReference类来实现，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，总会回收该对象占用的内存。

### 1.5.4.虚引用

​	虚引用需要PhantomReference类来实现，它不能单独使用，必须和引用队列联合使用。虚引用的主要作用是跟踪对象被垃圾回收的状态。



## 1.6.GC分代收集算法VS 分区收集算法

### 1.6.1.分代收集算法

​	当前主流VM垃圾收集都采用”分代收集”(Generational Collection)算法, 这种算法会根据对象存活周期的不同将内存划分为几块, 如JVM中的新生代、老年代、永久代，这样就可以根据各年代特点分别采用最适当的GC算法

#### 1.6.1.1.在新生代-复制算法

​	每次垃圾收集都能发现大批对象已死, 只有少量存活. 因此选用复制算法,只需要付出少量存活对象的复制成本就可以完成收集.

#### 1.6.1.2.在老年代-标记整理算法

​	因为对象存活率高、没有额外空间对它进行分配担保, 就必须采用“标记—清理”或“标记—整理”算法来进行回收,不必进行内存复制, 且直接腾出空闲内存.

### 1.6.2.分区收集算法

​	分区算法则将整个堆空间划分为连续的不同小区间, 每个小区间独立使用, 独立回收. 这样做的好处是可以控制一次回收多少个小区间, 根据目标停顿时间, 每次合理地回收若干个小区间(而不是整个堆), 从而减少一次GC所产生的停顿。

## 1.7.GC垃圾收集器

​	Java堆内存被划分为新生代和年老代两部分，新生代主要使用复制和标记-清除垃圾回收算法；年老代主要使用标记-整理垃圾回收算法，因此java虚拟中针对新生代和年老代分别提供了多种不同的垃圾收集器，JDK1.6中Sun HotSpot虚拟机的垃圾收集器如下：

![](../images/9.png)

### 1.7.1.Serial垃圾收集器（单线程、复制算法）

​	Serial（英文连续）是最基本垃圾收集器，使用复制算法，曾经是JDK1.3.1之前新生代唯一的垃圾收集器。Serial是一个单线程的收集器，它不但只会使用一个CPU或一条线程去完成垃圾收集工作，并且在进行垃圾收集的同时，必须暂停其他所有的工作线程，直到垃圾收集结束。Serial垃圾收集器虽然在收集垃圾过程中需要暂停所有其他的工作线程，但是它简单高效，对于限定单个CPU环境来说，没有线程交互的开销，可以获得最高的单线程垃圾收集效率，因此Serial垃圾收集器依然是java虚拟机运行在Client模式下默认的新生代垃圾收集器。

### 1.7.2.ParNew垃圾收集器（Serial+多线程）

​	ParNew垃圾收集器其实是Serial收集器的多线程版本，也使用复制算法，除了使用多线程进行垃圾收集之外，其余的行为和Serial收集器完全一样，ParNew垃圾收集器在垃圾收集过程中同样也要暂停所有其他的工作线程。ParNew收集器默认开启和CPU数目相同的线程数，可以通过-XX:ParallelGCThreads参数来限制垃圾收集器的线程数。【Parallel：平行的】ParNew虽然是除了多线程外和Serial收集器几乎完全一样，但是ParNew垃圾收集器是很多java虚拟机运行在Server模式下新生代的默认垃圾收集器。

### 1.7.3.Parallel Scavenge收集器（多线程复制算法、高效）

​	Parallel  Scavenge收集器也是一个新生代垃圾收集器，同样使用复制算法，也是一个多线程的垃圾收集器，它重点关注的是程序达到一个可控制的吞吐量（Thoughput，CPU用于运行用户代码的时间/CPU总消耗时间，即吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)），高吞吐量可以最高效率地利用CPU时间，尽快地完成程序的运算任务，主要适用于在后台运算而不需要太多交互的任务。自适应调节策略也是ParallelScavenge收集器与ParNew收集器的一个重要区别。

### 1.7.4.Serial Old收集器（单线程标记整理算法）

​	Serial  Old是Serial垃圾收集器年老代版本，它同样是个单线程的收集器，使用标记-整理算法，这个收集器也主要是运行在Client默认的java虚拟机默认的年老代垃圾收集器。在Server模式下，主要有两个用途：1.在JDK1.5之前版本中与新生代的Parallel Scavenge收集器搭配使用。2.作为年老代中使用CMS收集器的后备垃圾收集方案。新生代Serial与年老代Serial Old搭配垃圾收集过程图：新生代Parallel Scavenge收集器与ParNew收集器工作原理类似，都是多线程的收集器，都使用的是复制算法，在垃圾收集过程中都需要暂停所有的工作线程。新生代Parallel Scavenge/ParNew与年老代Serial Old搭配垃圾收集过程图：

![](../images/10.png)

​	新生代 Parallel Scavenge 收集器与 ParNew 收集器工作原理类似，都是多线程的收集器，都使用的是复制算法，在垃圾收集过程中都需要暂停所有的工作线程。新生代Parallel Scavenge/ParNew 与年老代 Serial Old 搭配垃圾收集过程图：

![](../images/11.png)

### 1.7.5.Parallel Old收集器（多线程标记整理算法）

​	Parallel Old收集器是Parallel Scavenge的年老代版本，使用多线程的标记-整理算法，在JDK1.6才开始提供。在JDK1.6之前，新生代使用ParallelScavenge收集器只能搭配年老代的Serial  Old收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量，Parallel  Old正是为了在年老代同样提供吞吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，可以优先考虑新生代Parallel  Scavenge和年老代Parallel Old收集器的搭配策略。新生代ParallelScavenge和年老代Parallel Old收集器搭配运行过程图：

![](../images/11.1.png)

### 1.7.6.CMS收集器（多线程标记清除算法）

​	Concurrent  mark  sweep(CMS)收集器是一种年老代垃圾收集器，其最主要目标是获取最短垃圾回收停顿时间，和其他年老代使用标记-整理算法不同，它使用多线程的标记-清除算法。最短的垃圾收集停顿时间可以为交互比较高的程序提高用户体验。CMS工作机制相比其他的垃圾收集器来说更复杂，整个过程分为以下4个阶段：

#### 1.7.6.1.初始标记

​	只是标记一下GC Roots能直接关联的对象，速度很快，仍然需要暂停所有的工作线程。

#### 1.7.6.2.并发标记

​	进行GC Roots跟踪的过程，和用户线程一起工作，不需要暂停工作线程。

#### 1.7.6.3.重新标记

​	为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。

#### 1.7.6.4.并发清除

​	清除GC Roots不可达对象，和用户线程一起工作，不需要暂停工作线程。由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户现在一起并发工作，所以总体上来看CMS收集器的内存回收和用户线程是一起并发地执行。CMS收集器工作过程：

![](../images/13.png)

### 1.7.7.G1收集器

​	Garbage  first垃圾收集器是目前垃圾收集器理论发展的最前沿成果，相比与CMS收集器，G1收集器两个最突出的改进是：

1.基于标记-整理算法，不产生内存碎片。

2.可以非常精确控制停顿时间，在不牺牲吞吐量前提下，实现低停顿垃圾回收。G1收集器避免全区域垃圾收集，它把堆内存划分为大小固定的几个独立区域，并且跟踪这些区域的垃圾收集进度，同时在后台维护一个优先级列表，每次根据所允许的收集时间，优先回收垃圾最多的区域。区域划分和优先级区域回收机制，确保G1收集器可以在有限时间获得最高的垃圾收集效率。

## 1.8.JAVA IO/NIO

### 1.8.1.阻塞IO模型

​	最传统的一种IO模型，即在读写数据过程中会发生阻塞现象。当用户线程发出IO请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU。当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。典型的阻塞IO模型的例子为：data = socket.read();如果数据没有就绪，就会一直阻塞在read方法。

### 1.8.2.非阻塞IO模型

​	当用户线程发起一个read操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。所以事实上，在非阻塞IO模型中，用户线程需要不断地询问内核数据是否就绪，也就说非阻塞IO不会交出CPU，而会一直占用CPU。典型的非阻塞IO模型一般如下：

```java
while(true){
    data = socket.read();
    if(data!= error){
        处理数据
            break;
    }
}
```

但是对于非阻塞IO 就有一个非常严重的问题，在 while 循环中需要不断地去询问内核数据是否就绪，这样会导致CPU 占用率非常高，因此一般情况下很少使用 while 循环这种方式来读取数据。



### 1.8.3.多路复用IO模型

​	多路复用IO模型是目前使用得比较多的模型。Java NIO实际上就是多路复用IO。在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。因为在多路复用IO模型中，只需要使用一个线程就可以管理多个socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。在Java NIO中，是通过selector.select()去查询每个通道是否有到达事件，如果没有事件，则一直阻塞在那里，因此这种方式会导致用户线程的阻塞。多路复用IO模式，通过一个线程就可以管理多个socket，只有当socket真正有读写事件发生才会占用资源来进行实际的读写操作。因此，多路复用IO比较适合连接数比较多的情况。另外多路复用IO为何比非阻塞IO模型的效率高是因为在非阻塞IO中，不断地询问socket状态时通过用户线程去进行的，而在多路复用IO中，轮询每个socket状态是内核在进行的，这个效率要比用户线程要高的多。不过要注意的是，多路复用IO模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件逐一进行响应。因此对于多路复用IO模型来说，一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询。

### 1.8.4.信号驱动IO模型

​	在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。

### 1.8.5.异步IO模型

​	异步IO模型才是最理想的IO模型，在异步IO模型中，当用户线程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从内核的角度，当它受到一个asynchronous read之后，它会立刻返回，说明read请求已经成功发起了，因此不会对用户线程产生任何block。然后，内核会等待数据准备完成，然后将数据拷贝到用户线程，当这一切都完成之后，内核会给用户线程发送一个信号，告诉它read操作完成了。也就说用户线程完全不需要实际的整个IO操作是如何进行的，只需要先发起一个请求，当接收内核返回的成功信号时表示IO操作已经完成，可以直接去使用数据了。也就说在异步IO模型中，IO操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。用户线程中不需要再次调用IO函数进行具体的读写。这点是和信号驱动模型有所不同的，在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用IO函数进行实际的读写操作；而在异步IO模型中，收到信号表示IO操作已经完成，不需要再在用户线程中调用IO函数进行实际的读写操作。注意，异步IO是需要操作系统的底层支持，在Java 7中，提供了Asynchronous IO。更多参考：http://www.importnew.com/19816.html

### 1.8.6.JAVA IO包

![](../images/14.png)

### 1.8.7.JAVA NIO

​	NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

![](../images/15.png)

#### 1.8.2.1.NIO的缓冲区

​	Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。NIO的缓冲导向方法不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

#### 1.8.2.2.NIO的非阻塞

​	IO的各种流是阻塞的。这意味着，当一个线程调用read() 或write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

![](../images/17.png)

### 1.8.8.Channel

​	首先说一下Channel，国内大多翻译成“通道”。Channel和IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：InputStream, OutputStream，而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。

NIO中的Channel的主要实现有：

​	1.FileChannel

​	2.DatagramChannel

​	3.SocketChannel

​	4.ServerSocketChannel

这里看名字就可以猜出个所以然来：分别可以对应文件IO、UDP和TCP（Server和Client）。下面演示的案例基本上就是围绕这4个类型的Channel进行陈述的。

### 1.8.9.Buffer

​	Buffer，故名思意，缓冲区，实际上是一个容器，是一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。

![](../images/16.png)

上面的图描述了从一个客户端向服务端发送数据，然后服务端接收数据的过程。客户端发送数据时，必须先将数据存入Buffer中，然后将Buffer中的内容写入通道。服务端这边接收数据必须通过Channel将数据读入到Buffer中，然后再从Buffer中取出数据来处理。在NIO中，Buffer是一个顶层父类，它是一个抽象类，常用的Buffer的子类有：ByteBuffer、IntBuffer、CharBuffer、LongBuffer、DoubleBuffer、FloatBuffer、ShortBuffer

### 1.8.10.Selector

​	Selector类是NIO的核心类，Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

## 1.9.JVM 类加载机制

JVM 类加载机制分为五个部分：加载，验证，准备，解析，初始化，下面我们就分别来看一下这 五个过程。

![](../images/18.png)

### 1.9.1.加载

​		加载是类加载过程中的一个阶段，这个阶段会在内存中生成一个代表这个类的 java.lang.Class  对象，作为方法区这个类的各种数据的入口。注意这里不一定非得要从一个 Class  文件获取，这里既可以从 ZIP 包中读取（比如从 jar 包和 war  包中读取），也可以在运行时计算生成（动态代理），也可以由其它文件生成（比如将 JSP 文件转换成对应的 Class  类）。

### 1.9.2.验证

​		这一阶段的主要目的是为了确保 Class  文件的字节流中包含的信息是否符合当前虚拟机的要求，并

且不会危害虚拟机自身的安全。



### 1.9.3.准备

​		准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，即在方法区中分配这些变量所使

用的内存空间。注意这里所说的初始值概念，比如一个类变量定义为：

```java
public static int v = 8080;
```

​		实际上变量 v 在准备阶段过后的初始值为 0 而不是 8080，将 v 赋值为 8080 的 put static  指令是程序被编译后，存放于类构造器<client>方法之中。但是注意如果声明为：

```java
public static final int v = 8080;
```

​		在编译阶段会为 v 生成 ConstantValue 属性，在准备阶段虚拟机会根据 ConstantValue 属性将  v赋值为 8080。



### 1.9.4. 解析 

​		解析阶段是指虚拟机将常量池中的符号引用替换为直接引用的过程。符号引用就是 class 文件中 的以下类型的常量。

```java
1.   CONSTANT_Class_info
2.   CONSTANT_Field_info
3.   CONSTANT_Method_info
```



### 1.9.5. 符号引用 

​		符号引用与虚拟机实现的布局无关，引用的目标并不一定要已经加载到内存中。各种虚拟 机实现的内存布局可以各不相同，但是它们能接受的符号引用必须是一致的，因为符号引 用的字面量形式明确定义在 Java 虚拟机规范的 Class 文件格式中。

###  1.9.6. 直接引用 

​		直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有 了直接引用，那引用的目标必定已经在内存中存在。 

### 1.9.7. 初始化 

​		初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载 器以外，其它操作都由 JVM 主导。到了初始阶段，才开始真正执行类中定义的 Java 程序代码。 

### 1.9.8. 类构造器 

​		初始化阶段是执行类构造器方法的过程。方法是由编译器自动收集类中的类变 量的赋值操作和静态语句块中的语句合并而成的。虚拟机会保证子方法执行之前，父类 的方法已经执行完毕，如果一个类中没有对静态变量赋值也没有静态语句块，那么编译 器可以不为这个类生成()方法。 

​		注意以下几种情况不会执行类初始化： 

```java
1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。 
2. 定义对象数组，不会触发该类的初始化。 
3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触 发定义常量所在的类。
4. 通过类名获取 Class 对象，不会触发类的初始化。 
5. 通过 Class.forName 加载指定类时，如果指定参数 initialize 为 false 时，也不会触发类初 始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。 
6. 通过 ClassLoader 默认的 loadClass 方法，也不会触发初始化动作。
```



### 1.9.9. 类加载器 

​		虚拟机设计团队把加载动作放到 JVM 外部实现，以便让应用程序决定如何获取所需的类，JVM 提 供了 3 种类加载器：

#### 1.9.9.1. 启动类加载器(Bootstrap ClassLoader) 

​		1. 负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath 参数指定路径中的，且被 虚拟机认可（按文件名识别，如 rt.jar）的类。

#### 1.9.9.2. 扩展类加载器(Extension ClassLoader)

​		 2. 负责加载 JAVA_HOME\lib\ext 目录中的，或通过 java.ext.dirs 系统变量指定路径中的类 库。

 #### 1.9.9.3. 应用程序类加载器(Application ClassLoader)：

		 3. 负责加载用户路径（classpath）上的类库。 JVM 通过双亲委派模型进行类的加载，当然我们也可以通过继承 java.lang.ClassLoader 实现自定义的类加载器。

![](../images/19.png)



### 1.9.10.双亲委派

​		当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载其中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的Class），子类加载器才会尝试自己去加载。采用双亲委派的一个好处是比如加载位于 rt.jar  包中的类 java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个 Object  对象。

![](../images/20.png)

### 1.9.11. OSGI（动态模型系统） 

​		OSGi(Open Service Gateway Initiative)，是面向 Java 的动态模型系统，是 Java 动态化模块化系 统的一系列规范。 

#### 1.9.11.1. 动态改变构造 

​		OSGi 服务平台提供在多种网络设备上无需重启的动态改变构造的功能。为了最小化耦合度和促使 这些耦合度可管理，OSGi 技术提供一种面向服务的架构，它能使这些组件动态地发现对方。 

#### 1.9.11.2. 模块化编程与热插拔 

​		OSGi 旨在为实现 Java 程序的模块化编程提供基础条件，基于 OSGi 的程序很可能可以实现模块级 的热插拔功能，当程序升级更新时，可以只停用、重新安装然后启动程序的其中一部分，这对企 业级程序开发来说是非常具有诱惑力的特性。 OSGi 描绘了一个很美好的模块化开发目标，而且定义了实现这个目标的所需要服务与架构，同时 也有成熟的框架进行实现支持。但并非所有的应用都适合采用 OSGi 作为基础架构，它在提供强大 功能同时，也引入了额外的复杂度，因为它不遵守了类加载的双亲委托模型。
