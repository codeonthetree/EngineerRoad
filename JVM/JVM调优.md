一、JVM参数类型
1、标准参数
-help
-server -client
-version showversion
-cp -classpath
2、X参数 (非标准化参数)
-Xint : 解释执行
-Xcomp : 第一次使用就编译成本地代码
-Xmixed : 混合模式，JVM自己来决定是否编译成本地代码

3、XX参数（非标准化参数）



Boolean 类型

格式：-XX:[+-]<name>表示启用或禁用name属性
eg : -XX:UseConcMarkSweepGC
-XX:UseG1GC

非Boolean类型（key,value形式）

格式: -XX:<name>=<value>表示name属性的值是value
eg : -XX:MaxGCPauseMillis=500
-Xms
-XX:InitialHeapSize
-Xmx
-XX:MaxHeapSize



二：查看JVM运行时参数
1、jps
查看运行时JAVA进程 (https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html)

2、jinfo
查看运行时参数 jinfo -flags PID

3、jstat : 查看JVM统计信息

类装载信息
垃圾收集信息
JIT编译信息


option : -class -compiler -gc

S0C、S1C、S0U、S1U ： S0 S1 总量与使用量
EC、EU ： Eden区总量与使用量
OC、OU : old区总量与使用量
MC、MU ：Metaspace区总量与使用量
CCSC、CCSU ：压缩类空间总量与使用量
YGC、YGCT : youngGC次数与时间
FGC、FGCT ：fullGC次数与时间
GCT：总的GC时间

4、JVM内存结构

Metaspace=Class、Package、Method、Field、字节码、常量池、符号引用等等
            CCS ： 32位指针的class
            CodeCache : JIT编译后的本地代码、JNI使用的C代码

5、jmap

导出内存镜像文件

内存溢出自动导出

-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./

使用 jmap手动导出


6、jstack

查看线程cpu占用过高，死锁问题
        Linux top -Hp pid 命令查看cpu消耗最高的线程id printf ‘%x’ 数字 转换成16进制
        jstack pid 通过查询16进制的线程ID

三、JVM GC调优

1、JVM运行时数据区 （https://docs.oracle.com/javase/specs/jvms/se8/html/index.html）

程序计数器：JVM支持多线程同时执行，每一个线程都有自己的计数器，现成正在执行的方法叫做当前方法，如果是JAVA代码，计数器里面存放的就是当前正在执行的指令的地址，如果是C代码，则为空
虚拟机栈：JAVA虚拟机栈是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是JAVA方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程
堆：JAVA堆是JAVA虚拟机所管理的内存中最大的一块。堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。次内存区域的为一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。JAVA堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。
方法区：方法去与JAVA堆一样，是各个线程共享的内存区域，他用于存储已被虚拟机加载的累信息、常亮、静态变量、即时编译器编译后的代码等数据。最然JAVA虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫非堆，目的是与JAVA堆区分开来
本地方法栈：本地方法栈与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行JAVA方法服务，而本地方法栈则为虚拟机使用到的Native方法服务

Metaspace=Class、Package、Method、Field、字节码、常量池、符号引用等等
            CCS ： 32位指针的class
            CodeCache : JIT编译后的本地代码、JNI使用的C代码
2、垃圾回收算法
哪些内存需要回收：可达性分析算法
GCRoot:
1、虚拟机栈中引用的对象
2、方法去中类静态属性引用的对象
3、方法区中常亮引用的对象
4、本地方法栈中JNI引用的对象

标记清除 ：算法分为标记和清除两个阶段，首先标记出所有需要回收的对象，在标记后统一回收
缺点：效率不高，会产生碎片，碎片太多会导致提前GC
复制：它将可用内存按照容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活的对象复制到另一块上面，然后再把用过的内存一次清理掉
优缺点：实现简单，运行高效，但空间利用率低

标记整理：标记过程仍然与"标记-清除"算法一样，但后续步骤不是直接堆可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存
优缺点：没有了内存碎片，但是整理内存比较耗时

分代垃圾回收：young区采用复制算法，old区采用标记清除或标记整理

对象分配

对象优先在eden区分配
大对象直接进入老年带：-XX:PretenureSizeThreshold
长期存活对象进入老年带：-XX:MaxTenuringThreshold
动态年龄计算：Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值


3、垃圾收集器
串行收集器Serial：Serial、Serial old
        并行收集器Parallel：Parallel Scavenge、parallel old ，吞吐量优先
        并发收集器Current：CMS、G1，响应时间优先 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC | -XX:+UseG1GC
并行：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。适合科学计算，后台处理等弱交互请求
并发：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能是交替执行），垃圾收集线程在执行的时候不会停顿用户程序的运行。适合对响应时间有要求的场景。
停顿时间：垃圾收集器在回收垃圾的过程中中断应用的时间 -XX:MaxGCPauseMillis
吞吐量：花在垃圾收集的时间和花在应用的时间的占比 -XX:GCTimeRatio=<n> ,垃圾收集时间占:1/（1+n）

CMS垃圾收集过程：
并发收集、低停顿低延迟、老年带的垃圾收集器
        1、CMS initial park ：初始标记Root， STW
        2、CMS concurrent mark ：并发标记
        3、CMS-concureent-preclean : 并发预清理
        4、CMS remark ：重新标记，STW
        5、CMS concurrent sweep ： 并发清除
        6、CMS-concurrent-Reset ： 并发重置
        CMS的缺点：
                1、CPU敏感
                2、会产生浮动垃圾
                3、会产生空间碎片

新生代中对象的特点是“朝生夕灭”，这样如果Remark前执行一次Minor GC，大部分对象就会被回收。CMS就采用了这样的方式，在Remark前增加了一个可中断的并发预清理（CMS-concurrent-abortable-preclean），该阶段主要工作仍然是并发标记对象是否存活，只是这个过程可被中断。此阶段在Eden区使用超过2M时启动，当然2M是默认的阈值，可以通过参数修改。如果此阶段执行时等到了Minor GC，那么上述灰色对象将被回收，Reamark阶段需要扫描的对象就少了。
除此之外CMS为了避免这个阶段没有等到Minor GC而陷入无限等待，提供了参数CMSMaxAbortablePrecleanTime ，默认为5s，含义是如果可中断的预清理执行超过5s，不管发没发生Minor GC，都会中止此阶段，进入Remark。
CMS提供CMSScavengeBeforeRemark参数，用来保证Remark前强制进行一次Minor GC

更多思考
其实新生代GC存在同样的问题，即老年代可能持有新生代对象引用，所以Minor GC时也必须扫描老年代。
JVM是如何避免Minor GC时扫描全堆的？ 经过统计信息显示，老年代持有新生代对象引用的情况不足1%，根据这一特性JVM引入了卡表（card table）来实现这一目的。如下图所示：

卡表的具体策略是将老年代的空间分成大小为512B的若干张卡（card）。卡表本身是单字节数组，数组中的每个元素对应着一张卡，当发生老年代引用新生代时，虚拟机将该卡对应的卡表元素设置为适当的值。如上图所示，卡表3被标记为脏（卡表还有另外的作用，标识并发标记阶段哪些块被修改过），之后Minor GC时通过扫描卡表就可以很快的识别哪些卡中存在老年代指向新生代的引用。这样虚拟机通过空间换时间的方式，避免了全堆扫描。
总结来说，CMS的设计聚焦在获取最短的时延，为此它“不遗余力”地做了很多工作，包括尽量让应用程序和GC线程并发、增加可中断的并发预清理阶段、引入卡表等，虽然这些操作牺牲了一定吞吐量但获得了更短的回收停顿时间。