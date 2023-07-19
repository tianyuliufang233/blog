---
title: "Perf原理"
date: 2023-09-19T10:59:07+08:00
categories:
- 收件箱
- linxu相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---
perf原理
<!--more-->
# perf原理
Linux性能计数器是一个基于内核的子系统，它提供一个性能分析框架，如硬件(cpu、PMU(Performance Monitoring Unit))功能和软件（软件计数器、tracepoint）功能。

Perf可以对程序进行函数级别的采样，从而了解程序的性能瓶颈。基本原理是：每隔一个固定时间，就是cpu上产生一个中断，看当前哪个进程、哪个函数，然后给对应的进程和函数加一个统计值，这样就知道cpu有多少时间在某个进程或某个函数上。
```
sudo perf record -a --call-graph dwarf -p `ps aux | grep "xxx" | grep -v grep | cut -c 9-15` -d 1 -b
```
# perf 命令
命令结构   perf [--version] [--help] [OPTIONS] COMMAND [ARGS]
|选项	|释义|
|---|---|
|annotate|	读取 perf.data(通过perf record创建) 展示注释代码|
|bench	|基准测试的一般框架，对系统调度、内存访问、epoll、futex进行压力测试|
|buildid-cache |	管理buildid cache|
|buildid-list |	列出一个perf.data文件的buildids|
|c2c	|共享数据C2C/HITM分析|
|config	|在配置文件中获取和设置变量|
|data	|数据文件相关的处理|
|diff	|读取 perf.data文件并显示差异|
|evlist	|列出perf.data文件的事件名称|
|ftrace	|内核ftrace功能的简单包装器|
|inject	|筛选以使用其他信息扩充事件流|
|kallsyms|	在运行内核的中搜索标志|
|kmem	|用于跟踪/测量内核内存属性的工具|
|kvm	|用于跟踪/测量kvm来宾系统|
|list	|列出所有事件符号类型|
|lock	|分析事件锁|
|mem	|配置文件内存访问|
|record	|运行命令并将其配置文件记录到perf.data中|
|report	|读取perf.data(由perf记录创建)并显示配置文件|
|sched	|用于跟踪/测量调度程序属性|
|script	|读取perf.data(由perf记录创建)并显示跟踪输出|
|stat	|运行命令并收集性能计数器统计信息|
|test	|运行全流程测试|
|timechart|	可视化工作负载期间整体系统行为的工具|
|top	|系统分析工具|
|probe	|定义新的动态跟踪点|
|trace	|启动工具|

# 常用参数 stat
|参数|	含义	|
|---|---|
|-e --event |	选择事件符号名称(sys/bus/event_source/drices/<pmu>/format/*)|	
|-i --no-inherit |	子任务不继承计数器	|
|-p --pid |	在现有进程上统计事件状态	|
|-t --tid |	在现有线程上统计事件状态	|
|-a --all-cpus |	全系统范围所有cpu	|
|-c --scale |	缩放/规范化 计数器值	|
|-d --detailed |	打印更详细的统计数据，最多可指定3次  -d  详细事件、L1和LLC数据缓存。-d -d 更多详细事件，DTLB和ITLB事件。-d -d -d 非常详细的事件，添加预取事件|	
|-r --repeat= |	重复命令并打印平均值+stddev(max:100)  0表示永远	|
|-B --big-num | 根据区域设置打印带有数千个分隔符的大数字	|
|-C --cpu |	仅依靠提供的cpu列表计数。多个cpu可以以逗号分隔的形式提供，不带空格的列表：0，1.cpu的使用范围使用-表示：0-2指定。在每线程模式下，将忽略此选项。-a选项仍然是激活系统范围监视所必需的。默认值是依靠所有cpu。|	
|-A --no-aggr	| 不聚合所有受监视的cpu计数 |	
|-n --null |	不启动任何计数器 |
|-v --verbose |	更详细，显示计数器 打开错误等|	
|-x  --field-separator SEP |	使用CSV样式输出打印计数，以便轻松直接导入电子表格。列由SEP中指定的字符串分隔|	
|--table |	以表格显示每次运行的时间(-r 选项)，如：性能统计  --null -r 5 --table 性能调度管道。性能计数器统计数据(5次运行)|	
|-G  -cgroup |	仅在名为"name"的容器cgroup中监视。此选项仅在per-cpu模式。name在受监视的cpu上运行时收到监控。可以提供多个cgroups。|	
|-o  --output |	将输出打印到指定文件中	|
|--append |	追加到-o指定的文件中，如果未指定-o，则忽略|	
|--log-fd |	输出记录到fd，而不是stderr。与--output互补|	
|--pre  --post |	测量前 测量后挂钩|	
|-I  --interval-print msecs |	每N毫秒打印计数增量(最小值:  1毫秒) 开销百分比可能很高，在某些情况下，如间隔小于100毫秒。请谨慎使用|	
|--interval-count |	固定次数的打印计数增量。合格选项与"-l"选项一起使用|	
|--interval-clear |	下一个间隔之前清除屏幕。	|
|--timeout |	停止性能统计信息会话并在N毫秒后打印计数增量(最小值：10毫秒)。"-l"选项不支持此选项。|	
|--metric-only	| 仅打印计算指标。将它们打印在一行中。不显示任何原始值。不支持--per-thread	|
|--per-socket |	系统范围模式测量的每个处理器插槽的聚合计数。用于检测套接字之间不平衡的有用模式。输出包括插槽编号和该插槽上的联机处理器数量。|	
|--per-die |	用于系统范围模式测量的每个处理器芯片的总计数。这个模式检测模具之间不平衡的有用模式。	|
|--per-core |	系统范围模式测量的每个物理处理器的聚合计数。	|
|--per-thread |	监视线程(-t 选项)或进程(-p时，每个受监控线程的聚合计数选项)	|
|-D --delay msecs |	启动程序后，等待毫秒后再进行测量。过滤掉程序启动阶段。 |	
|-T, --transaction |	打印事务执行统计信息(如果支持)	|
|-i file  --input file	|input file name|	
|--per-socket	| 用于系统范围模式测量的每个处理器socket 的聚合计数|	
|--per-die |	用于系统范围模式测量的每个处理器芯片的总计数	|
|--per-core |	用于系统范围模式测量的每个物理处理器的聚合计数	|
|-M  --metrics|	打印以逗号分隔的列表中指定的衡量指标组。对于一组所有指标从组中添加。系统会自动测量指标中的事件。见性能列出可能的指标和指标组的输出。|	
|-A  --no-aggr|	不聚合所有受监控cpu的计数|	
|--topdown |	打印自上而下的 1 级指标（如果 CPU 支持）。这允许通过将消耗的周期分解为前端绑定、后端绑定、错误推测和停用来确定 CPU 密集型工作负载的 CPU 管道中的瓶颈	|

## 常用top命令
|参数 |	释义	|
|---|---|
|-a  --all-cpus |	系统范围的集和 |	
|-c  --count |	事件采样周期	|
|-C  --cpu |	仅监视体哦那个的cpu列表。多个cpu可以作为逗号分隔的列表提供：0，1.cpu的范围用-：0-2指定。默认监视所有cpu。|	
|-d --delay |	刷新之间间隔秒数	|
|-e --event |	选择pmu事件。悬着可以是符号事件名称(使用perf列表列出所有事件)或rnn形式的原始pmu事件(eventsel+umask)，其中NNN是十六进制事件描述符。 |	
|-E  --entries |	展示多功能	|
|-f  --count-filter |	只展示事件数超过此值的函数	|
|--group |	将计数器放入计数器组	|
|-F  --freq |	在此频率下配置文件。|	
|-i --inherit |	子任务不继承计数器	|
|-k  --vmlinux |	vmlinux的路径。批注功能必需|	
|--ingore-vmlinux |	忽略vmlinux文件	|
|-m --mmap-pages= |	mmap数据页数(必须是2的幂)或附加的大小规格单位字符 - B/K/M/G。大小向上舍入为最近的页幂为两个值	|
|-p  --pid |	分析现有进程ID上的事件(逗号分隔列表)	|
|-t  --tid |	分析现有线程ID上的事件(逗号分隔列表)	|
|-u --uid |	记录uid拥有的线程中的事件。姓名或者编号	|
|-r  --realtime |	使用此RT SCHED_FIFO优先级收集数据	|
|--sym-annotate |	注释此符号	|
|-K  --hide_kernel_symbols |	隐藏内核符号 |	
|-U --hide_user_symbols |	隐藏用户符号	|
|--demangle-kernel |	动态内核符号	|
|-D --dump-symtab |	转储用于分析的符号表 |	
|-v --verbose |	更详细(显示打开的错误计数) |	
|-z  --zero |	显示更新的历史记录为零	|
|-s  --sort |	按键排序: pid、comm、dso、符号、父项、srcline、权重、local_weight、终止、in_tx、事务、开销、样本、周期。|	
|--fields |	指定输出字段-可以csv格式指定多个键。以下字段可用:间接费用、overhead_sys、overhead_us、overhead_children	|


## record 参数
|参数|	注释|
|---|---|
|-e --event |	选择pmu事件。可以选择一个符号事件，一行pmu事件，符号形成的pmu事件，|
|--filter |	事件筛选器。此选项遵循-e，该选择器选择跟踪点事件或硬件跟踪pmu。|
|-a --all-cpus |	从所有cpu收集系统范围(如果未指定目标，则默认)|
|-p --pid |	记录现有进程ID上的事件(逗号分隔列表) |
|-t  --tid |	记录现有线程ID 上的事件(逗号分隔列表)。这个选项通常默认关闭，开启需要添加 --inherit |
|-r  --realtime |	使用此 RT SCHED_FIFO优先级收集数据 |
|--no-buffering |	不使用缓冲收集数据|
|-c  --count |	采样事件周期 |
|-o --output |	输出文件名|
|-i  --no-inherit |	子任务不继承计数器 |
|-F  --freq |	在此凭率下配置文件。使用当前最大频率为max，即kernel.perf_event_max_sample_rate sysctl中的值。|
|--strict-freq |	如果无法使用指定的凭率，则为失败|
|-m --mmap-pages |	mmap数据页数(必须是2的幂)或附加单位字符 - B/K/M/G的大小规范。大小向上舍入为最接近的页次方为2的值。通过添加逗号，指定AUX区域跟踪的map页数|
|--group |	将所有事件放在一个事件组中。|
|-g  --call-graph |	开启调用图(堆栈链/回溯)记录 |
|-q  --quite |	不打印任何消息，用于脚本 |
|-v    --verbose |	更详细(显示计数器打开的错误等) |
|-s  --stat |	记录每个线程的事件计数。将其与性能报告-T一起使用以查看值|
|-d  --data |	记录示例虚拟地址|
|--phys-data |	记录实例物理地址 |
|-T  --timestamp |	记录实例时间戳。将其与perf报告 -D 一起使用以查看事件戳|
|-P --period |	记录采样周期|
|--sample-cpu |	记录实例cpu |
|-n --no-samples |	不采样|
|-R  --raw-samples |	从所有打开的计数器中收集原始样本记录(跟踪点计数器的默认值)|
|-c  --cpu |	仅在提供的cpu列表中收集样本。多个cpu可以作为逗号分隔的列表提供，没有空格: 0,1。cpu的范围使用-：0-2指定。在启用继承模式的每线程模式下(默认)，仅当线程在指定的cpu上执行时，才会捕获示例|
|-B --no-buildid |	不将二进制文件生成的id保存到perf.data文件中。这会在录制过程中的最后一步花费很长时间，因为它需要处理查找MMAP记录的所有事件|
|-N  --no-buildid-cache |	不更新构建id缓存。在perf.data文件(包括构建id)中的信息足够的情况下，可以节省一些开销。还可以将"record.build-id"配置变量设置为无缓存以具有相同的效果。|
|-G  name  --cgroup  name |	 仅在cgroup中监视此选项仅在每核cpu模式下可用。必须挂载cgroup文件系统。监视属于容器"名称"的所有线程。可以提供多个cgroup |
|-b  --branch-any |	启用分支堆栈采样。可以对任何类型的分支进行采样。是 --branch-filter any的快捷方式。 |
|-j --branch-filter |	启用分支堆栈采样。每个样本捕获一系列连续的分支。每个示例捕获的分支数取决于底层硬件、感兴趣的分支类型核执行代码。any、any_call、any_ret、ind_call、call、u、k、hv、in_tx、no_tx、abort_tx、cond、save_type|
|--wight |	启用加权抽样。每个样品记录一个额外的重量，并可与重量和local_wight排序一起显示。|
|--transaction |	记录事务相关事件的事务标志。|
|--per-thread |	使用pre-thread。默认情况下，将创建每个cpu的mmap。此选项将覆盖该选项，并使用每线程mmap。这样做的副作用是继承会自动禁用。|
|-D --delay	| 启动程序后，等待几秒后再测量，对于过滤程序的启动阶段很有用。|
|-I --intr-regs |	在中断时捕获机器状态(寄存器)，每个样本的计数器溢出。捕获的寄存器列表取决于体系结构。默认情况下，此选项为关闭状态。|
|--user-regs |	在采样时捕获用户寄存器。于I相同的参数|
|--running-time |	记录读取事件的运行和启用时间 |
|-k  --clockid |	设置用于perf_event_type记录种各种时间字段的时钟id。|
|-S --snapshot |	选择aux区域跟踪快照模式。此选项仅对aux区域追踪事件有效。可以指定每个快照要捕获的字节数。|
|--proc-map-timeout |	处理预先存在的线程时，可能需要很长时间，因为文件可能很大。这种情况下，需要超时。默认500秒|
|--switch-events |	记录上下文切换事件|
|--buildid-all |	记录所有dso的构建id，无论是否实际命中。|
|--all-kernel |	将所有使用的事件配置为在内核空间种运行|
|--all-user |	将所有使用的事件配置为在用户空间种运行|
|--timestamp-filename |	将时间戳附加到输出文件名 |
|--timestamp-boundary |	记录时间戳边界(第一个/最后一个样本的时间)|
|--switch-output |	生成多个perf.data文件，时间戳前缀，根据模式值切换到新文件："信号"  -当收到SIGUSR2(默认值)或 - 当达到大小阈值时，大小应为附加单位字符的数字|
|--dry-run |	解析选项后退出。--dry-run可用于检测cmdline选项中的错误|
|--tail-synthesize |	不要在记录开始时收集非样本事件，而是在完成输出文件期间收集它们。收集的非样本事件反映了记录完成时系统的状态|
|--overwrite |	使所有事件都使用可重写的环形缓冲区。可重写环形缓冲区的工作方式类似于飞行记录器。当它变满时，内核将覆盖旧的记录因此永远捕获进入perf.data文件。|	