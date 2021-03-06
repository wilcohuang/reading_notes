

# 2.1 CPU性能统计信息



## 2.1.1 运行队列统计



在`Linux`中，一个进程要么是可运行的，要么是阻塞的。阻塞进程可能等待`I/O`设备来的数据，也可能等待系统调用的结果，如果进程是可运行的，那么该进程要和其他可运行的进程竞争`CPU`时间。一个可运行的进程不一定会使用`CPU`(毕竟还没有运行)。可运行的正在等待处理器的进程构成了`运行队列`。运行队列越长，处于等待状态的进程越多。

&nbsp;

性能工具通常给我们的有两个统计维度：

1. 可运行的进程个数和等待`I/O`阻塞进程个数
2. 系统平均负载。即**正在运行**和**可运行**的进程总数。一般情况下，取平均负载时间为1分钟，5分钟和15分钟。(`top`命令的`load average`)



>`load average`的值不能超过`CPU核数 * 1.0`。以单核`CPU`为例，`load average`的值为1.0表示此时`CPU`刚好满载。如果值大于1.0，表示系统负荷过大。通常情况下，系统负荷大于0.7就要引起注意，应该调查性能问题，防止恶化。当负荷大于5.0时，表明系统长时间接近没有相应，或者接近死机了。`cat /proc/cpuinfo`命令，可以查看CPU信息。`grep -c 'model name' /proc/cpuinfo`命令，直接返回CPU的总核心数。 



## 2.1.2 上下文切换



大部分现代处理器一次只能运行一个进程或线程（**超线程处理器**实际上可以同时运行多个进程，但是`Linux`会把他们看作多个单线程处理器）。处理器有限，进程却很多，所以要不断进行上下文切换。



上下文切换开销很大，`CPU`要保存旧进程的所有上下文信息，并取出新进程的所有上下文信息。上下文中包含了`Linux`跟踪新进程的大量信息。其中包括：进程正在执行的指令，分配给进程的内存（页表信息保存），进程打开的文件等。



**进程调度（上下文切换）的原因**

> 1. 进程由于睡眠或者其他原因（阻塞）主动放弃`CPU`
>
> 2. 内核周期性的中断检测，适当情况下，内核调度器调起另外的进程



每秒定时中断的次数与架构和内核版本有关，一个检查中断频率的简单方法是用`/proc/interrupts`文件。



```shell
[wilcohuang@10_225_19_205 ~]$ cat /proc/interrupts | grep timer ; sleep 10 ; cat /proc/interrupts | grep timer
  0:         31          0          0          0          0          0          0          0   IO-APIC-edge      timer
LOC: 1323537658 1627525130  811647975  401779361 4284362526   35431888 4267725048 1204761983   Local timer interrupts
  0:         31          0          0          0          0          0          0          0   IO-APIC-edge      timer
LOC: 1323541942 1627530153  811652391  401781946 4284364903   35437433 4267726296 1204766384   Local timer interrupts

```

上面我们查看了间隔10秒的中断情况，（10秒后的中断总数目-10秒前的中断总数目）/10秒可以算出每秒中断次数。如果进程上下文接环数目明显多于定时器中断，那么可能是进程`I/O`请求很多，或者系统调用（如`sleep`）造成的。



**查看进程当前处于哪个CPU**

进程可能会被调度到不同`CPU`，查看进程当前处于的`CPU`方法有：

```shell
$ ps -o pid,psr,comm -p <pid>
  PID PSR COMMAND
 5357  10 prog
```

输出表示进程的 `pid` 为 5357（名为`prog`）目前在`CPU` 内核 10 上运行着。如果该过程没有被固定，`PSR` 列会根据内核可能调度该进程到不同内核而改变显示。



其他方式可参考：[Linux 有问必答：如何知道进程运行在哪个 CPU 内核上？](https://linux.cn/article-6307-1.html)

​			      [top查看线程所在的cpu号](https://blog.csdn.net/zgy666/article/details/78953313) (使用`top`命令查看时，需要先按`f`选择展示视图，因为`top`命令默认不展示进程处于的`CPU`)



## 2.1.3 软中断



//todo: 补全中断知识



中断分为硬终端和软中断。查看`/proc/interrupts`文件可以显示出哪些`CPU`上触发了哪些中断。



## 2.1.4 CPU使用率



CPU可以空闲，可以执行内核代码，可以执行用户代码，可以处于`iowait`状态，可以处于`irq`状态（高优先级处理硬件中断），也可以处于`softirq`模式，即系统正在执行同样由中断触发的内核代码，只不过其运行于较低优先级（下半步代码）





















