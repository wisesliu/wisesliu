# 你说你做过 JVM 调优和参数配置，那你平时工作用过的配置参数有哪些？

![](https://images.pexels.com/photos/207924/pexels-photo-207924.jpeg?cs=srgb&dl=pexels-207924.jpg&fm=jpg)

## JVM参数类型

JVM 参数类型大致分为以下几类：

- **标准参数**（-），即在 JVM 的各个版本中基本不变的，相对比较稳定的参数，向后兼容
- **非标准参数**（-X），变化比较小的参数，默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容；
- **非Stable参数**（-XX），此类参数各个 JVM 实现会有所不同，将来可能会随时取消，需要慎重使用；



### 标准参数

![](https://imgkr.cn-bj.ufileos.com/798c9e1c-5aae-4798-a2eb-7bb7e454c9e1.png)

- `-version`：输出 java 的版本信息，比如 jdk 版本、vendor、model
- `-help`：输出 java 标准参数列表及其描述
- `-showversion`：输出 java 版本信息（与-version相同）之后，继续输出 java 的标准参数列表及其描述，相当于`java -verion` 和 `java -help`
- `-client`：设置 jvm 使用 client 模式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或者PC应用开发和调试
- `-server`：设置 jvm 使 server 模式，特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。在具有64位能力的 jdk 环境下将默认启用该模式，而忽略 -client 参数
- `-agentlib:libname[=options]`：用于装载本地 lib 包。其中 libname 为本地代理库文件名，默认搜索路径为环境变量 PATH 中的路径，options 为传给本地库启动时的参数，多个参数之间用逗号分隔
- `-agentpath:pathname[=options]`：按全路径装载本地库，不再搜索PATH中的路径；其他功能和 agentlib相同
- `-Dproperty=value`
   设置系统属性名/值对，运行在此jvm之上的应用程序可用System.getProperty("property")得到value的值。
   如果value中有空格，则需要用双引号将该值括起来，如-Dname="space string"。
   该参数通常用于设置系统级全局变量值，如配置文件路径，以便该属性在程序中任何地方都可访问
- `-verbose:[class|gc|jni]`：输出每个加载类|gc|jni 的信息



### X 参数

非标准参数又称为扩展参数，其列表如下

![](https://imgkr.cn-bj.ufileos.com/52aa1112-79d8-495b-9dae-ff284aefc204.png)

- `-Xint`：设置 jvm 以解释模式运行，所有的字节码将被直接执行，而不会编译成本地码
- -Xmixed：混合模式，JVM自己来决定是否编译成本地代码，默认使用的就是混合模式 
- `-Xbatch`：关闭后台代码编译，强制在前台编译，编译完成之后才能进行代码执行。 默认情况下，jvm 在后台进行编译，若没有编译完成，则前台运行代码时以解释模式运行
- `-Xbootclasspath:bootclasspath`：让 jvm 从指定路径（可以是分号分隔的目录、jar、或者zip）中加载bootclass，用来替换 jdk 的 rt.jar；若非必要，一般不会用到
- `-Xbootclasspath/a:path` ：将指定路径的所有文件追加到默认 bootstrap 路径中
- `-Xfuture`：让jvm对类文件执行严格的格式检查（默认 jvm 不进行严格格式检查），以符合类文件格式规范，推荐开发人员使用该参数。
- `-Xincgc`：开启增量 gc（默认为关闭），这有助于减少长时间GC时应用程序出现的停顿，但由于可能和应用程序并发执行，所以会降低CPU对应用的处理能力
- **`-Xloggc:file`**： 与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。若与 verbose 命令同时出现在命令行中，则以 -Xloggc 为准
- **`-Xms`**：指定 jvm 堆的初始大小，默认为物理内存的1/64，最小为1M，可以指定单位，比如k、m，若不指定，则默认为字节
- **`-Xmx`**：指定 jvm 堆的最大值，默认为物理内存的 1/4或者1G，最小为2M；单位与`-Xms`一致
- `-Xprof`：跟踪正运行的程序，并将跟踪数据在标准输出输出；适合于开发环境调试
- **-Xss**： 设置单个线程栈的大小，一般默认为 512k




### xx参数

#### Boolean 类型

- 公式： -xx:+ 或者 - 某个属性值（+表示开启，- 表示关闭）

- Case

  - 是否打印 GC 收集细节

    - `-XX:+PrintGCDetails `
    - `-XX:-PrintGCDetails `

    ![img](https://tva1.sinaimg.cn/large/00831rSTly1gdebpozfgwj315o0sgtcy.jpg)

    添加如下参数后，重新查看，发现是 + 号了

    ![](https://tva1.sinaimg.cn/large/00831rSTly1gdebrx25moj31170u042c.jpg)

  - 是否使用串行垃圾回收器

    - `-XX:-UseSerialGC`
    - `-XX:+UseSerialGC`

#### KV 设值类型

- 公式： -XX:属性key=属性value

- Case:

  - `-XX:MetaspaceSize=128m`

  - `-xx:MaxTenuringThreshold=15`

  - 我们常见的 -Xm s和 -Xmx 也属于 KV 设值类型

    - `-Xms` 等价于 `-XX:InitialHeapSize`
    - `-Xmx` 等价于 `-XX:MaxHeapSize`

    ![](https://tva1.sinaimg.cn/large/00831rSTly1gdecj9d7z3j310202qdgb.jpg)

- jinfo 举例，如何查看当前运行程序的配置

  - jps -l
  - jinfo -flag [配置项] 进程编号
  - jinfo **-flags** 1981(打印所有)
  - jinfo -flag PrintGCDetails 1981
  - jinfo -flag MetaspaceSize 2044

这些都是命令级别的查看，我们也可以在程序运行中查看

```java
long totalMemory = Runtime.getRuntime().totalMemory();
long maxMemory = Runtime.getRuntime().maxMemory();

System.out.println("total_memory(-xms)="+totalMemory+"字节，" +(totalMemory/(double)1024/1024)+"MB");
System.out.println("max_memory(-xmx)="+maxMemory+"字节，" +(maxMemory/(double)1024/1024)+"MB");
```



## 常用配置

| **参数名称**                | **含义**                                                   | **默认值**           | 说明                                                         |
| --------------------------- | ---------------------------------------------------------- | -------------------- | ------------------------------------------------------------ |
| -Xms                        | 初始堆大小                                                 | 物理内存的1/64(<1GB) | 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制. |
| -Xmx                        | 最大堆大小                                                 | 物理内存的1/4(<1GB)  | 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制 |
| -Xmn                        | 年轻代大小(1.4or lator)                                    |                      | **注意**：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。 整个堆大小=年轻代大小 + 年老代大小 + 持久代大小. 增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8 |
| -XX:NewSize                 | 设置年轻代大小(for 1.3/1.4)                                |                      |                                                              |
| -XX:MaxNewSize              | 年轻代最大值(for 1.3/1.4)                                  |                      |                                                              |
| -XX:PermSize                | 设置持久代(perm gen)初始值                                 | 物理内存的1/64       |                                                              |
| -XX:MaxPermSize             | 设置持久代最大值                                           | 物理内存的1/4        |                                                              |
| -Xss                        | 每个线程的堆栈大小                                         |                      | JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右 一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长） 和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"” -Xss is translated in a VM flag named ThreadStackSize” 一般设置这个值就可以了。 |
| -*XX:ThreadStackSize*       | Thread Stack Size                                          |                      | (0 means use default stack size) [Sparc: 512; Solaris x86: 320 (was 256 prior in 5.0 and earlier); Sparc 64 bit: 1024; Linux amd64: 1024 (was 0 in 5.0 and earlier); all others 0.] |
| -XX:NewRatio                | 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代) |                      | -XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5 Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。 |
| -XX:SurvivorRatio           | Eden区与Survivor区的大小比值                               |                      | 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10 |
| -XX:LargePageSizeInBytes    | 内存页的大小不可设置过大， 会影响Perm的大小                |                      | =128m                                                        |
| -XX:+UseFastAccessorMethods | 原始类型的快速优化                                         |                      |                                                              |
| -XX:+DisableExplicitGC      | 关闭System.gc()                                            |                      | 这个参数需要严格的测试                                       |
| -XX:MaxTenuringThreshold    | 垃圾最大年龄                                               |                      | 如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代. 对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率 该参数只有在串行GC时才有效. |
| -XX:+AggressiveOpts         | 加快编译                                                   |                      |                                                              |
| -XX:+UseBiasedLocking       | 锁机制的性能改善                                           |                      |                                                              |
| -Xnoclassgc                 | 禁用垃圾回收                                               |                      |                                                              |
| -XX:SoftRefLRUPolicyMSPerMB | 每兆堆空闲空间中SoftReference的存活时间                    | 1s                   | softly reachable objects will remain alive for some amount of time after the last time they were referenced. The default value is one second of lifetime per free megabyte in the heap |
| -XX:PretenureSizeThreshold  | 对象超过多大是直接在旧生代分配                             | 0                    | 单位字节 新生代采用Parallel Scavenge GC时无效 另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象. |
| -XX:TLABWasteTargetPercent  | TLAB占eden区的百分比                                       | 1%                   |                                                              |
| -XX:+*CollectGen0First*     | FullGC时是否先YGC                                          | false                |                                                              |

**并行收集器相关参数**

| -XX:+UseParallelGC          | Full GC采用parallel MSC (此项待验证)              |      | 选择垃圾收集器为并行收集器.此配置仅对年轻代有效.即上述配置下,年轻代使用并发收集,而年老代仍旧使用串行收集.(此项待验证) |
| --------------------------- | ------------------------------------------------- | ---- | ------------------------------------------------------------ |
| -XX:+UseParNewGC            | 设置年轻代为并行收集                              |      | 可与CMS收集同时使用 JDK5.0以上,JVM会根据系统配置自行设置,所以无需再设置此值 |
| -XX:ParallelGCThreads       | 并行收集器的线程数                                |      | 此值最好配置与处理器数目相等 同样适用于CMS                   |
| -XX:+UseParallelOldGC       | 年老代垃圾收集方式为并行收集(Parallel Compacting) |      | 这个是JAVA 6出现的参数选项                                   |
| -XX:MaxGCPauseMillis        | 每次年轻代垃圾回收的最长时间(最大暂停时间)        |      | 如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.       |
| -XX:+UseAdaptiveSizePolicy  | 自动选择年轻代区大小和相应的Survivor区比例        |      | 设置此选项后,并行收集器会自动选择年轻代区大小和相应的Survivor区比例,以达到目标系统规定的最低相应时间或者收集频率等,此值建议使用并行收集器时,一直打开. |
| -XX:GCTimeRatio             | 设置垃圾回收时间占程序运行时间的百分比            |      | 公式为1/(1+n)                                                |
| -XX:+*ScavengeBeforeFullGC* | Full GC前调用YGC                                  | true | Do young generation GC prior to a full GC. (Introduced in 1.4.1.) |

**CMS相关参数**

| -XX:+UseConcMarkSweepGC                | 使用CMS内存收集                           |      | 测试中配置这个以后,-XX:NewRatio=4的配置失效了,原因不明.所以,此时年轻代大小最好用-Xmn设置.??? |
| -------------------------------------- | ----------------------------------------- | ---- | ------------------------------------------------------------ |
| -XX:+AggressiveHeap                    |                                           |      | 试图是使用大量的物理内存 长时间大内存使用的优化，能检查计算资源（内存， 处理器数量） 至少需要256MB内存 大量的CPU／内存， （在1.4.1在4CPU的机器上已经显示有提升） |
| -XX:CMSFullGCsBeforeCompaction         | 多少次后进行内存压缩                      |      | 由于并发收集器不对内存空间进行压缩,整理,所以运行一段时间以后会产生"碎片",使得运行效率降低.此值设置运行多少次GC以后对内存空间进行压缩,整理. |
| -XX:+CMSParallelRemarkEnabled          | 降低标记停顿                              |      |                                                              |
| -XX+UseCMSCompactAtFullCollection      | 在FULL GC的时候， 对年老代的压缩          |      | CMS是不会移动内存的， 因此， 这个非常容易产生碎片， 导致内存不够用， 因此， 内存的压缩这个时候就会被启用。 增加这个参数是个好习惯。 可能会影响性能,但是可以消除碎片 |
| -XX:+UseCMSInitiatingOccupancyOnly     | 使用手动定义初始化定义开始CMS收集         |      | 禁止hostspot自行触发CMS GC                                   |
| -XX:CMSInitiatingOccupancyFraction=70  | 使用cms作为垃圾回收 使用70％后开始CMS收集 | 92   | 为了保证不出现promotion failed(见下面介绍)错误,该值的设置需要满足以下公式**[CMSInitiatingOccupancyFraction计算公式](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html#CMSInitiatingOccupancyFraction_value)** |
| -XX:CMSInitiatingPermOccupancyFraction | 设置Perm Gen使用到达多少比率时触发        | 92   |                                                              |
| -XX:+CMSIncrementalMode                | 设置为增量模式                            |      | 用于单CPU情况                                                |
| -XX:+CMSClassUnloadingEnabled          |                                           |      |                                                              |

**辅助信息**

| -XX:+PrintGC                          |                                                          |      | 输出形式:[GC 118250K->113543K(130112K), 0.0094143 secs] [Full GC 121376K->10414K(130112K), 0.0650971 secs] |
| ------------------------------------- | -------------------------------------------------------- | ---- | ------------------------------------------------------------ |
| -XX:+PrintGCDetails                   |                                                          |      | 输出形式:[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs] [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs] |
| -XX:+PrintGCTimeStamps                |                                                          |      |                                                              |
| -XX:+PrintGC:PrintGCTimeStamps        |                                                          |      | 可与-XX:+PrintGC -XX:+PrintGCDetails混合使用 输出形式:11.851: [GC 98328K->93620K(130112K), 0.0082960 secs] |
| -XX:+PrintGCApplicationStoppedTime    | 打印垃圾回收期间程序暂停的时间.可与上面混合使用          |      | 输出形式:Total time for which application threads were stopped: 0.0468229 seconds |
| -XX:+PrintGCApplicationConcurrentTime | 打印每次垃圾回收前,程序未中断的执行时间.可与上面混合使用 |      | 输出形式:Application time: 0.5291524 seconds                 |
| -XX:+PrintHeapAtGC                    | 打印GC前后的详细堆栈信息                                 |      |                                                              |
| -Xloggc:filename                      | 把相关日志信息记录到文件以便分析. 与上面几个配合使用     |      |                                                              |
| -XX:+PrintClassHistogram              | garbage collects before printing the histogram.          |      |                                                              |
| -XX:+PrintTLAB                        | 查看TLAB空间的使用情况                                   |      |                                                              |
| XX:+PrintTenuringDistribution         | 查看每次minor GC后新的存活周期的阈值                     |      | Desired survivor size 1048576 bytes, new threshold 7 (max 15) new threshold 7即标识新的存活周期的阈值为7。 |

**GC性能方面的考虑**



## 再谈 JVM 参数设置

经过前面对 JVM 参数的介绍及相关例子的实验，相信大家对 JVM 的参数有了比较深刻的理解，接下来我们再谈谈如何设置 JVM 参数

1、首先 Oracle 官方推荐堆的初始化大小与堆可设置的最大值一般是相等的，即 Xms = Xmx，因为起始堆内存太小（Xms），会导致启动初期频繁 GC，起始堆内存较大（Xmx）有助于减少 GC 次数

2、调试的时候设置一些打印参数，如-XX:+PrintClassHistogram -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -Xloggc:log/gc.log，这样可以从gc.log里看出一些端倪出来

3、系统停顿时间过长可能是 GC 的问题也可能是程序的问题，多用 jmap 和 jstack 查看，或者killall -3 Java，然后查看 Java 控制台日志，能看出很多问题

4、 采用并发回收时，年轻代小一点，年老代要大，因为年老大用的是并发回收，即使时间长点也不会影响其他程序继续运行，网站不会停顿

5、仔细了解自己的应用，如果用了缓存，那么年老代应该大一些，缓存的 HashMap 不应该无限制长，建议采用 LRU 算法的 Map 做缓存，LRUMap 的最大长度也要根据实际情况设定

要设置好各种 JVM 参数，还可以对 server 进行压测， 预估自己的业务量，设定好一些 JVM 参数进行压测看下这些设置好的 JVM 参数是否能满足要求





## JVM 参数简介

在开始实践之前我们有必要先简单了解一下 JVM 参数配置，因为本文之后的实验中提到的 JVM 中的栈，堆大小，使用的垃圾收集器等都需要通过 JVM 参数来设置

先来看下如何运行一个 Java 程序

```java
public class Test {
    public static  void main(String[] args) {
        System.out.println("test");
    }
}
```

1. 首先我们通过 **javac Test.java** 将其转成字节码

2. 其次我们往往会输入 **java Test** 这样的命令来启动 JVM 进程来执行此程序,其实我们在启动 JVM 进程的时候，可以指定相应的 JVM 的参数,如下蓝色部分

   ![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk44ib4JwRjzBAdiaI7oY4dmXe1oNIQfRluUy9xPvjXX5ZF15XNZFKmDnxVA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

指定这些 JVM 参数我们就可以指定启动 JVM 进程以哪种模式（server 或 client），运行时分配的堆大小，栈大小，用什么垃圾收集器等等，JVM 参数主要分以下三类

1、 标准参数（-），所有的 JVM 实现都必须实现这些参数的功能，而且向后兼容；例如 **-verbose:gc**（输出每次GC的相关情况)

2、 非标准参数（-X），默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容，栈，堆大小的设置都是通过这个参数来配置的，用得最多的如下

| 参数示例 | 表示意义                          |
| :------- | :-------------------------------- |
| -Xms512m | JVM 启动时设置的初始堆大小为 512M |
| -Xmx512m | JVM 可分配的最大堆大小为 512M     |
| -Xmn200m | 设置的年轻代大小为 200M           |
| -Xss128k | 设置每个线程的栈大小为 128k       |

3、非Stable参数（-XX），此类参数各个 jvm 实现会有所不同，将来可能会随时取消，需要慎重使用, -XX:-option 代表关闭 option 参数，-XX:+option 代表要关闭 option 参数,例如要启用串行 GC，对应的 JVM 参数即为 -XX:+UseSerialGC。非 Stable 参数主要有三大类

- 行为参数（Behavioral Options）：用于改变 JVM 的一些基础行为，如启用串行/并行 GC

| 参数示例                | 表示意义                                                  |
| :---------------------- | :-------------------------------------------------------- |
| -XX:-DisableExplicitGC  | 禁止调用System.gc()；但jvm的gc仍然有效                    |
| -XX:-UseConcMarkSweepGC | 对老生代采用并发标记交换算法进行GC                        |
| -XX:-UseParallelGC      | 启用并行GC                                                |
| -XX:-UseParallelOldGC   | 对Full GC启用并行，当-XX:-UseParallelGC启用时该项自动启用 |
| -XX:-UseSerialGC        | 启用串行GC                                                |

- 性能调优（Performance Tuning）：用于 jvm 的性能调优，如设置新老生代内存容量比例

| 参数示例                      | 表示意义                              |
| :---------------------------- | :------------------------------------ |
| -XX:MaxHeapFreeRatio=70       | GC后java堆中空闲量占的最大比例        |
| -XX:NewRatio=2                | 新生代内存容量与老生代内存容量的比例  |
| -XX:NewSize=2.125m            | 新生代对象生成时占用内存的默认值      |
| -XX:ReservedCodeCacheSize=32m | 保留代码占用的内存容量                |
| -XX:ThreadStackSize=512       | 设置线程栈大小，若为0则使用系统默认值 |

- 调试参数（Debugging Options）：一般用于打开跟踪、打印、输出等 JVM 参数，用于显示 JVM 更加详细的信息

| 参数示例                          | 表示意义                            |
| :-------------------------------- | :---------------------------------- |
| -XX:HeapDumpPath=./java_pid.hprof | 指定导出堆信息时的路径或文件名      |
| -XX:-HeapDumpOnOutOfMemoryError   | 当首次遭遇OOM时导出此时堆中相关信息 |
| -XX:-PrintGC                      | 每次GC时打印相关信息                |
| -XX:-PrintGC Details              | 每次GC时打印详细信息                |

*画外音：以上只是列出了比较常用的 JVM 参数，更多的 JVM 参数介绍请查看文末的参考资料*

明白了 JVM 参数是干啥用的，接下来我们进入实战演练，下文中所有程序运行时对应的 JVM 参数都以 VM Args 的形式写在开头的注释里，读者如果在执行程序时记得要把这些 JVM 参数给带上哦



https://docs.oracle.com/javacomponents/jrockit-hotspot/migration-guide/cloptions.htm#JRHMG127

参数不懂，推荐直接去看官网，

- -Xms

  - 初始大小内存，默认为物理内存1/64
  - 等价于 -XX:InitialHeapSize

- -Xmx

  - 最大分配内存，默认为物理内存的1/4
  - 等价于 -XX:MaxHeapSize

- -Xss

  - 设置单个线程的大小，一般默认为 512k~1024k
  - 等价于 -XX:ThreadStackSize
  - 如果通过 `jinfo ThreadStackSize 线程 ID` 查看会显示为 0，指的是默认出厂设置

- -Xmn

  - 设置年轻代大小（一般不设置）

- -XX:MetaspaceSize

  - 设置元空间大小。元空间的本质和永久代类似，都是对 JMM 规范中方法区的实现。不过元空间与永久代最大的区别是，元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制
  - 但是元空间默认也很小，频繁 new 对象，也会 OOM
  - -Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX:+PrintFlagsFinal

- -XX:+PrintGCDetails

  - 输出详细的 GC 收集日志信息 

  - 测试时候，可以将参数调到最小，

    `-Xms10m -Xmx10m -XX:+PrintGCDetails`

    定义一个大对象，撑爆堆内存，

    ```
    public static void main(String[] args) throws InterruptedException {
        System.out.println("==hello gc===");
    
        //Thread.sleep(Integer.MAX_VALUE);
    
        //-Xms10m -Xmx10m -XX:PrintGCDetails
    
        byte[] bytes = new byte[11 * 1024 * 1024];
    
    }![](https://tva1.sinaimg.cn/large/007S8ZIlly1gehkvas3vzj31a90u0n7t.jpg)
    ```

  - Full GC![img](https://tva1.sinaimg.cn/large/00831rSTly1gdefrc3lmbj31hy0gk7of.jpg)

    ![img](https://tva1.sinaimg.cn/large/00831rSTly1gdefr8tvx0j31h60m41eq.jpg) 

  - GC![img](https://tva1.sinaimg.cn/large/00831rSTly1gdefrf0dfqj31fs0honjk.jpg)

- -XX:SurvivorRatio

  - 设置新生代中 eden 和S0/S1空间的比例
  - 默认 -XX:SurvivorRatio=8,Eden:S0:S1=8:1:1
  - SurvivorRatio值就是设置 Eden 区的比例占多少，S0/S1相同，如果设置  -XX:SurvivorRatio=2，那Eden:S0:S1=2:1:1

- -XX:NewRatio

  - 配置年轻代和老年代在堆结构的比例
  - 默认 -XX:NewRatio=2，新生代 1，老年代 2，年轻代占整个堆的 1/3
  - NewRatio值就是设置老年代的占比，如果设置-XX:NewRatio=4，那就表示新生代占 1，老年代占 4，年轻代占整个堆的 1/5

- -XX:MaxTenuringThreshold

  - 设置垃圾的最大年龄（java8 固定设置最大 15）
  - ![img](https://tva1.sinaimg.cn/large/00831rSTly1gdefr4xeq1j31g80lek6e.jpg)



![img](https://tva1.sinaimg.cn/large/00831rSTly1gdee0iss88j31eu0n6aqi.jpg)

## 3. 你平时工作用过的 JVM 常用基本配置参数有哪些？

- -XX:+PrintFlagsInitial

  - 主要查看初始默认值

  - java -XX:+PrintFlagsInitial

  - java -XX:+PrintFlagsInitial -version

  - ![img](https://tva1.sinaimg.cn/large/00831rSTly1gdee0ndg33j31ci0m6k5w.jpg)

    **等号前有冒号** :=  说明 jvm 参数有人为修改过或者 JVM加载修改

    false 说明是Boolean 类型 参数，数字说明是 KV 类型参数

- -XX:+PrintFlagsFinal

  - 主要查看修改更新
  - java -XX:+PrintFlagsFinal
  - java -XX:+PrintFlagsFinal -version
  - 运行java命令的同时打印出参数 java -XX:+PrintFlagsFinal -XX:MetaspaceSize=512m Hello.java

- -XX:+PrintCommondLineFlags

  - 打印命令行参数
  - java -XX:+PrintCommondLineFlags -version
  - 可以方便的看到垃圾回收器
  - ![img](https://tva1.sinaimg.cn/large/007S8ZIlly1gehf1e54soj31e006qjz6.jpg)

### 盘点家底查看JVM默认值



参数不懂，推荐直接去看官网，

https://docs.oracle.com/javacomponents/jrockit-hotspot/migration-guide/cloptions.htm#JRHMG127



https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC

https://docs.oracle.com/javase/8/

Java SE Tools Reference for UNIX](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html)







https://www.cnblogs.com/duanxz/p/3482366.html

https://www.javatt.com/p/48544