在线上处理问题的时候，知识，经验是关键基础，数据是依据，工具是知识处理数据的手段，这里说的数据包括但不限于运行日志、异常堆栈、GC日志、线程快照（threaddump/javacore 文件）、堆转存快照（heapdump/hprof 文件）等。

在本文中，工具主要是指 JDK 自带的工具，都位于 JDK 的 bin 目录下

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggnywp7ha2j31540aqgs3.jpg)



## 基础故障处理工具

### jps：进程监控工具

该命令很像 UNIX 的 ps 命令，列出当前用户正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main() 函数所在的类）名称以及这些进程的本地虚拟机唯一 ID（LVMID，Local Virtual Machine Identifier）。

功能虽单一，但使用频率超级高。

jps 命令格式：

```shell
jps [options] [hostid]
```

jps 工具的主要选项：

- -q：只输出 LVMID，不输出类名称、Jar 名称和传入 main 方法的参数；
- -l：输出 main 类或 Jar 的 全限定名称；
- -m：输出传入 main 方法的参数；
- -v：输出虚拟机进程启动时传入的 JVM 参数

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggo0kxc742j30xc072jv1.jpg)



### jstat：统计信息监控工具

jstat（JVM Statistics Monitoring Tool） 是用于识别虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、jit 编译等运行时数据。

jstat 命令格式：

```shell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggoa5sdezvj312s0hutko.jpg)

- option：参数选项

- -t：可以在打印的列加上 timestamp 列，用于显示系统运行的时间
- -h：可以在周期性数据的时候，可以在指定输出多少行以后输出一次表头
- vmid：进程ID
- lines：表头与表头的间隔行数
- interval：执行每次的间隔时间，单位为毫秒
- count：用于指定输出记录的 次数，缺省则会一直打印

option 值的选项：

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggoa8xbu5lj30oq0bead0.jpg)

- -class：显示类加载 ClassLoad 的相关信息；
- -compiler:：显示 JIT 编译的相关信息；
- -gc：显示和 gc相关的堆信息；
- -gccapacity：显示各个代的容量以及使用情况；
- -gcmetacapacity：显示元空间 metaspace 的大小；
- -gcnew：显示新生代信息； 
- -gcnewcapacity: 显示新生代大小和使用情况；
- -gcold：显示老年代垃圾收集状况；
- -gcoldcapacity：显示老年代的大小；
- -gcutil：显示垃圾回收信息；
- -gccause：显示垃圾回收的相关信息（同 -gcutil），同时显示最后一次或当前正在发生的垃圾回收的原因；
- -printcompilation：输出 JIT 编译的方法信息

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggoa2djmknj31hy0c4wlo.jpg)

如上表示每 5 秒输出一次 gc 信息，且输出 3 次

其中S0，S1表示新生代两个 Survivor 区，E代表的是新生代的 Eden区，C的意思是容量，U表示已经使用的意思，O表示的老年代，M表示方法区。F和Y则表示 fullGC 和 minorGC。GCT表示GC Time。



### jinfo：配置信息工具

jinfo（Configuration Info for Java）的作用是实时地查看和调整虚拟机各项参数，使用 jps 命令的 -v 参数可以查看虚拟机启动时显式指定的参数列表。但如果想知道未被显式指定的参数的系统默认值，除了去找资料外，就只能使用 jinfo 的 -flag 选项进行查询。

- no option：输出全部的参数和系统属性
- -flag name：输出对应名称的参数
- -flag [+|-]name：开启或者关闭对应名称的参数

我们可以通过 jinfo 实时的修改虚拟机的参数，但是不是任何命令都可以修改，可以修改的参数我们先来执行这个命令：`java -XX:+PrintFlagsFinal -version`，会列出当前机器支持的所有参数，那么用 jinfo 可以修改的参数是什么呢？只有最后一列显示 `manageable` 的这一列才能进行修改。

仔细查看发现可修改的参数其实并不多，jvm 的运行内存一旦在运行时确定下来，那么就无法修改。但是无法一些错误信息没有记录，或者是处于关闭状态，还是可以修改的。

jinfo 命令格式：

```shell
jinfo [-options] <pid>
```

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggoakmoq93j30qy07qacs.jpg)

查看某一进程是否开启了GC日志信息的详细打印，`jinfo -flag PrintGCDetails pid`

查看堆内存大小 `jinfo -flag MaxHeapSize pid`



### jmp：内存映像工具

jamp（Memory Map for Java）命令用于生成堆转储快照（一般称为 heapdump 或 dump 文件）。如果不使用jmap 命令，要想获取 Java 堆转储快照，还有一些比较“暴力”的手段：譬如 `-XX : + HeapDumpOnOutOfMemoryError` 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，通过  `-XX : +HeapDumpOnCtrlBreak` 参数则可以使用 `[Ctrl]+[Break]` 键让虚拟机生成 dump 文件，又或者在 Linux 系统下通过 `Kill-3` 命令发送进程退出信号“吓唬”一下虚拟机，也能拿到 dump 文件。

jmap 的作用并不仅仅是为了获取 dump 文件，他还可以查询 finalize 执行队列、Java 堆和方法区的详细信息，如空间使用率、当前用的是哪种收集器等。

和 jinfo 命令一样，jmap 有不少功能在 Windows 平台下都是受限的，除了生成 dump 文件的 `-dump` 选项和用于查看每个类的实例、空间占用统计的 `-histo` 选项在所有操作系统都提供之外，其余选项都只能在 Linux/Solaris下使用。

jmap 命令格式：

```shell
jmap [-options]  vmid
```

option 值的选项：

- dump：生成Java堆转储快照。格式为：`-dump:[live, ]format=b, file=<filename>`，其中 live 子参数说明是否只 dump 出存活的对象
- -finalizerinfo：显示在 F-Queue 中等待 Finalizer 线程执行 finalize 方法的对象。只在 Linux/Solaris 平台下有效
- -heap：显示 Java 堆详细信息，如使用哪种回收器、参数配置、分代状况等。只在 Linux/Solaris 平台下有效
- -histo：显示堆中对象统计信息，包括类、实例数量、合计容量
- -permstat：以 ClassLoader 为统计口径显示永久代内存状态。只在 Linux/Solaris 平台下有效
- -F：当虚拟机进程对 -dump 选项没有响应时，可使用这个选项强制生成 dump 快照。只在 Linux/Solaris 平台下有效

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggoc50uza1j30zk0a0gq9.jpg)



### jhat：堆转储快照分析工具

jhat（JVM Heap Analysis Tool）命令与 jmap 搭配使用，来分析 jmap 生成的堆转储快照。jhat 内置了一个微型的 HTTP/HTML 服务器，生成 dump 文件的分析结果后，可以在浏览器中查看。

不过实事求是地说，在实际工作中，除非手上真的没有别的工具可用，否则一般都不会去直接使用 jhat 命令来分析 dump 文件，主要原因有两个：

1. 一般不会在部署应用程序的服务器上直接分析 dump 文件，即使可以这样做，也会尽量将 dump 文件复制到其他机器。分析工作是一个耗时而且消耗硬件资源的过程，既然都要在其他机器进行，就没有必要受到命令行工具的限制了；
2. jhat 的分析功能相对来说比较简陋，VisualVM，以及专业用于分析 dump 文件的 Eclipse Memory Analyzer、IBM HeapAnalyzer 等工具，都能实现比 jhat 更强大更专业的分析功能。



### jstack：堆栈跟踪工具

jstack 用于生成虚拟机当前时刻的线程快照。线程快照是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过 jstack 来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

jstack 命令格式：

```shell
jstack [-options] vmid
```

option 值的选项：

- -F：当正常输出请求不被响应时，强制输出线程堆栈
- -l：输出锁信息
- -m：如果调用到本地方法的话，可以显示 C/C++ 堆栈

JDK5 之后，`java.lang.Thread` 类的 `getAllStackTraces()` 方法可以获取虚拟机中所有线程的StackTraceElement 对象，可以查看堆栈信息。

```java
for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread.getAllStackTraces().entrySet()) {
			Thread thread = (Thread) stackTrace.getKey();
			StackTraceElement[] stack = (StackTraceElement[]) stackTrace.getValue();
			if (thread.equals(Thread.currentThread())) {
				continue;
			}
			System.out.println("Thread name is :" + thread.getName());
			for (StackTraceElement stackTraceElement : stack) {
				System.out.println("\t" + stackTraceElement);
			}
		}
```

统计线程数：`jstack -l 28367 | grep 'java.lang.Thread.State' | wc -l`

**jstack 检测 CPU 过高**

1. `top`（`top -H -p pid`）：查看cpu占用高线程的 tid
2. `printf "%x\n" tid`，将线程 ID 转换为 16进制格式的 nid
3. `jstack pid|grep nid -A 100`：定位cpu占用线程



## 可视化故障处理工具

JDK 中除了附带大量的命令行工具外，还提供了几个功能集成度更高的可视化工具，用户可以使用这些可视化工具以更加便捷的方式进行进程故障诊断和调试工作。这类工具主要包括 JConsole、JHSDB、VisualVM 和 JMC 四个。

### Jconsole

从Java 5开始 引入了 JConsole。JConsole 是一个内置 Java 性能分析器，可以从命令行或在 GUI shell 中运行。您可以轻松地使用 JConsole（或者，它更高端的 “近亲” VisualVM ）来监控 Java 应用程序性能和跟踪 Java 中的代码。    



### VisualVM

VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优。这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作。本文主要介绍如何使用 VisualVM 进行性能分析及调优。









恰当的使用虚拟机故障处理、分析工具可以提升我们分析数据、定位并解决问题的效率，但我们也要知道工具永远都是知识技能的一层包装，没有什么工具是"秘密武器"。





## OOM 问题排查的一些常用工具

接下来我们来看下如何排查造成 OOM 的原因，内存泄漏是最常见的造成 OOM 的一种原因，所以接下来我们以来看看怎么使用工具来排查这种问题,使用到的工具主要有两大类

**1、使用 mat（Eclipse Memory Analyzer） 来分析 dump（堆转储快照） 文件**

主要步骤如下

- 运行 Java 时添加 「-XX:+HeapDumpOnOutOfMemoryError」 参数来导出内存溢出时的堆信息，生成 hrof 文件, 添加 「-XX:HeapDumpPath」可以指定 hrof 文件的生成路径,如果不指定则 hrof 文件生成在与字节码文件相同的目录下
- 使用 MAT（Eclipse Memory Analyzer）来分析 hrof 文件，查出内存泄漏的原因

接下来我们就来看看如何用以上的工具查看如下内存泄漏案例

```
/**
* VM Args:-Xmx10m
 */
import java.util.ArrayList;
import java.util.List;
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        while (true) {
            list.add("OutOfMemoryError soon");
        }
    }
}
```

为了让以上程序快速产生 OOM, 我把堆大小设置成了 10M, 这样执行 「java -Xmx10m -XX:+HeapDumpOnOutOfMemoryError Main」后很快就发生了 OOM，此时我们就拿到了 hrof 文件，下载 MAT 工具，打开 hrof,进行分析，打开之后选择 「Leak Suspects Report」进行分析，可以看到发生 OOM 的线程的堆栈信息，明确定位到是哪一行造成的

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk44QWer8TLAhny1tOZcS0gYVI8CElnkwoHxQxqxdVMic4hQBYompTKO6ag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



*如图示，可以看到 Main.java 文件的第 12 行导致了这次的 OOM*

**2、使用 jvisualvm 来分析**

用第一种方式必须等 OOM 后才能 dump 出 hprof 文件，但如果我们想在运行中观察堆的使用情况以便查出可能的内存泄漏代码就无能为力了，这时我们可以借助 **jvisualvm** 这款工具, jvisualvm 的功能强大，除了可以实时监控堆内存的使用情况，还可以跟踪垃圾回收，运行中 dump 中堆内存使用情况、cpu分析，线程分析等，是查找分析问题的利器，更骚的是它不光能分析本地的 Java 程序，还可以分析线上的 Java 程序运行情况, 本身这款工具也是随 JDK 发布的，是官方力推的一款运行监视，故障处理的神器。我们来看看如何用 jvisualvm 来分析上文所述的存在内存泄漏的如下代码

```
import java.util.Map;
import java.util.HashMap;

public class KeylessEntry {
    static class Key {
        Integer id; 
        Key(Integer id) {
            this.id = id;
        }  
        @Override
        public int hashCode() {
            return id.hashCode();
        }
    }

    public static void main(String[] args) {
        Map<Key,String> m = new HashMap<Key,String>();
        while(true) {
            for(int i=0;i<10000;i++) {
                if(!m.containsKey(new Key(i))) {
                    m.put(new Key(i), "Number:" + i);
                }
            }
        }
    }
}
```

打开 jvisualvm （终端输入 jvisualvm 执行即可），打开后，将堆大小设置为 500M，执行命令 **java Xms500m -Xmx500m KeylessEntry**，此时可以观察到左边出现了对应的应用 KeylessEntry,双击点击 open

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk445FOl7e3SFFMlmBibKvluGrMlzjIicIoLtfX2p5iaqPYpBbBjYOLzBAmZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

打开之后可以看到展示了 CPU，堆内存使用，加载类及线程的情况

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk4489ratobztWxsIIjyJbWFoWIg8ic6ZUERibC81pc35wuqW55kgar3uUfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注意看堆（Heap）的使用情况，一直在上涨

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk44hIU8l1X70hqXbhfHUPNKF2ERsjSwT54icia1vSIAqAWBibIQG3dCBX9Wg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时我们再点击 「Heap Dump」

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk44BVxVwoN99jHhhK97AAQnicXt7tlOERWH7V6RJqCDjWZXJ2TlfcklH2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

过一会儿即可看到内存中对象的使用情况

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk448z3z6bu76NkrDehR3PxKd2tPNRCfhZ2ObMFRX7jDJa7AylGc0Z2LWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到相关的 TreeNode 有 291w 个，远超正常情况下的 10000 个！说明 HashMap 一直在增长，自此我们可以定位出问题代码所在！

**3、使用 jps + jmap 来获取 dump 文件**

jps 可以列出正在运行的虚拟机进程，并显示执行虚拟机主类及这些进程的本地虚拟机唯一 ID，如图示

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk449ic7ZBCWWca88nzQrkvWTfUw7kuhDXtKLjnllSDXHrTFPgAFUEKQMug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

拿到进程的 pid 后，我们就可以用 jmap 来 dump 出堆转储文件了，执行命令如下

```
jmap -dump:format=b,file=heapdump.phrof pid
```

拿到 dump 文件后我们就可以用 MAT 工具来分析了。
但这个命令在生产上一定要慎用！因为JVM 会将整个 heap 的信息 dump 写入到一个文件，heap 比较大的话会导致这个过程比较耗时，并且执行过程中为了保证 dump 的信息是可靠的，会暂停应用！



## jstat 与可视化 APM 工具构建

jstat 是用于监视虚拟机各种运行状态信息的命令行工具，可以显示本地或者远程虚拟机进程中的类加载，内存，垃圾收集，JIT 编译等运行数据，jstat 支持定时查询相应的指标，如下

```
jstat -gc 2764 250 22
```

定时针对 2764 进程输出堆的垃圾收集情况的统计，可以显示 gc 的信息，查看gc的次数及时间,利用这些指标，把它们可视化，对分析问题会有很大的帮助,如图示，下图就是我司根据 jstat 做的一部分 gc 的可视化报表，能快速定位发生问题的问题点，如果大家需要做 APM 可视化工具，建议配合使用 jstat 来完成。

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLVIoXNqicyWxibebAvTuJxk44O4ltjSGfibEsaGv4OSDNv9sgcqicSwJydSHIHKVetyZk8JoPRzfoO98g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)