MPI并行程序编译及运行
=====================

简介
~~~~

本系统的通信网络由Mellanox `QM8700 HDR200交换机 <https://www.mellanox.com/products/infiniband-switches/QM8700>`__\ 和\ `ConnectX-6 HDR100网卡 <https://www.mellanox.com/products/infiniband-adapters/connectx-6>`__\ 组成的100Gbps高速计算网络及1Gbps千兆以太网两套网络组成。InfiniBand网络相比千兆以太网具有高带宽、低延迟的特点，通信性能比千兆以太网要高很多，建议使用。

本系统安装有多种MPI实现，主要有：HPC-X（Mellanox官方推荐）、Intel MPI（不建议使用，特别是2019版）和Open MPI，并可与不同编译器相互配合使用，安装目录分别在 ``/opt/hpcx`` 、 ``/opt/intel`` 和 ``/opt/openmpi`` ，且具有不同版本的组合。

用户可以运行\ ``module apropos MPI``\ 或\ ``module avail``\ 查看可用MPI环境，可用类似命令设置所需的MPI环境：\ ``module load hpcx/hpcx-intel-2019.update5``\ ，使用此命令有时需要手动加载对应的编译器等版本，比如报：

::

    error while loading shared libraries: libimf.so: cannot open shared
    object file: No such file or directory

则需要加载对应的Intel编译器，比如\ ``module load intel/2019.update5``\ 。

Mellanox HDR是比较新的高速计算网络，老的MPI环境对其支持不好。

MPI并行程序的编译
~~~~~~~~~~~~~~~~~

HPC-X ScalableHPC工具集
^^^^^^^^^^^^^^^^^^^^^^^

`Mellanox HPC-X ScalableHPC工具集 <https://docs.mellanox.com/pages/viewpage.action?pageId=19798265>`__\ 是综合的软件包，含有MPI及SHMEM/PGAS通讯库。HPC-X ScalableHPC还包含这些库之上的用于提升性能和扩展性的多种加速包，包括加速点对点通信的UCX(Unified Communication X)、加速MPI/PGAS中集合操作的FCA(Fabric Collectives Accelerations)。这些全特性的、经完备测试的及打包好的工具集使得MPI和SHMEM/PGAS程序获得高性能、扩展性和效率，且保证了在Mellanox互连系统中这些通信库经过了全优化。

Mellanox HPC-X ScalableHPC工具集利用了基于Mellanox硬件的加速引擎，可以最大化基于MPI和SHMEM/PGAS的应用性能。这些应用引擎是Mellanox网卡（CORE-Direct引擎，硬件标记匹配(Tag Matching)等）和交换机（如Mellanox SHARP加速引擎）解决方案的一部分。Mellanox可扩展的分层聚合和归约协议(Scalable Hierarchical Aggregation and Reduction Protocol, SHARP)技术通过将集合操作从CPU端卸载到交换机网络端，通过去除在端到端之间发送多次数据的的需要，大幅提升了MPI操作性能。这种创新性科技显著降低了MPI操作时间，释放了重要的CPU资源使其用于计算而不是通信，且降低了到达聚合节点时通过网络的数据量。

HPC-X主要特性如下：

-  完整的MPI、PGAS/SHMEM包，且含有Mellanox UCX和FCA加速引擎

-  兼容MPI 3.2标准

-  兼容OpenSHMEM 1.4标准

-  从MPI进程将集合通信从CPU卸载到Mellanox网络硬件上

-  利用底层硬件体系结构最大化提升应用程序性能

-  针对Mellanox解决方案进行了全优化

-  提升应用的可扩展性和资源效率

-  支持RC、DC和UD等多种传输

-  节点内共享内存通信

-  带消息条带的多轨支持

-  支持GPU-direct的CUDA

HPC-X环境：

-  HPC-X CUDA支持：

   -  HPC-X默认是对于单线程模式优化的，这支持GPU和无GPU模式。

   -  HPC-X是基于CUDA 10.1编译的，由于CUDA 10.1不支持比GCC v8新的，因此对于基于v8之后的GCC编译的，不支持CUDA。

-  HPC-X多线程支持 - hpcx-mt：

   -  该选项启用所有多线程支持。

HPC-X MPI是Open MPI的一种高性能实现，利用Mellanox加速能力且无缝结合了业界领先的商业和开源应用软件包进行了优化。很多用法可参考，该部分主要介绍其不同的参数设置等。

Mellanox Fabric集合通信加速(Fabric Collective Accelerator, FCA)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

集合通信执行全体工薪操作占用系统中所有进程/节点，因此必须执行越快越高效越好。很多应用里面都含有大量的集合通讯，普通的MPI实现那会占用大量的CPU资源及产生系统噪声。Mellanox将很多类似通信从CPU卸载到Mellanox硬件网卡适配器(HCA)和交换机上及降低噪声，这种技术称之为CORE-Direct® (Collectives Offload Resource Engine)。

FCA 4.4当前支持阻塞和非阻塞的集合通信：Allgather、Allgatherv、Allreduce、AlltoAll、AlltoAllv、Barrier和Bcast。

**采用FCA v4.x (hcoll)运行MPI**
 

HPC-X默认启用FCA v4.3。

-  采用默认FCA配置参数运行：

   ``mpirun -mca coll_hcoll_enable 1 -x HCOLL_MAIN_IB=mlx5_0:1 <...>``

-  采用FCA运行：

   ``oshrun -mca scoll_mpi_enable 1 -mca scoll basic,mpi -mca coll_hcoll_enable 1 <...>``

**Open MPI中启用FCA**

在Open MPI中启用FCA v4.4，通过下述方法显式设定模块化组件架构模块化组件架构MCA(Modular Component Architecture)参数：

``mpirun -np 32 -mca coll_hcoll_enable 1 -x coll_hcoll_np=0 -x HCOLL_MAIN_IB=<device_name>:<port_num> ./a.out``

**调整FCA v4.4配置**

显示当前信息：

``/opt/mellanox/hcoll/bin/hcoll_info --all``

FCA v4.4的参数是简单的环境变量，可以通过以下方式之一设置：

-  通过mpirun命令设置：

   ``mpirun ... -x HCOLL_ML_BUFFER_SIZE=65536``

-  从SHELL设置：

   ``export HCOLL_ML_BUFFER_SIZE=65536``

   ``mpirun ...``

**选择端口及设备**

``-x HCOLL_MAIN_IB=<device_name>:<port_num>``

**启用卸载MPI非阻塞集合**
                     
``-x HCOLL_ENABLE_NBC=1``

支持以下非阻塞MPI集合：

-  MPI_Ibarrier

-  MPI_Ibcast

-  MPI_Iallgather

-  MPI_Iallreduce (4b, 8b, SUM, MIN, PROD, AND, OR, LAND, LOR)

注意：启用非阻塞MPI集合将在阻塞MPI集合中禁止多播聚合。

**启用Mellanox SHARP软件加速集合**

HPC-X支持Mellanox SHARP软件加速集合，这些集合默认是启用的。

-  启用Mellanox SHARP加速：

   ``-x HCOLL_ENABLE_SHARP=1``

-  禁止Mellanox SHARP加速

   ``-x HCOLL_ENABLE_SHARP=0``

-  更改Mellanox SHARP消息阈值（默认为256）：

   ``-x HCOLL_BCOL_P2P_ALLREDUCE_SHARP_MAX=<threshold>``

**HCOLL v4.4中的GPU缓存支持**
                         

如果CUDA运行时(runtime)是有效的，则HCOLL自动启用GPU支持。以下集合操作支持GPU缓存：

-  MPI_Allreduce

-  MPI_Bcast

-  MPI_Allgather

如果libhcoll的其它聚合操作API被启用GPU缓存调用，则会检查缓存类型后返回错误HCOLL_ERROR。

控制参数为\ ``HCOLL_GPU_ENABLE``\ ，其值可为0、1和-1：

+----+----------------------------------------------------------------+
| 值 | 含义                                                           |
+====+================================================================+
| 0  | 禁止GPU支持。不会检查用户缓存指针                              |
|    | 。此情形下，如用户提供在GPU上分配缓存，则这种行为是未定义的。  |
+----+----------------------------------------------------------------+
| 1  | 启用GPU支持。将检查缓存指针，且启用HCOLL                       |
|    | GPU聚合，这是CUDA运行时有效时的默认行为。                      |
+----+----------------------------------------------------------------+
| -1 | 部分                                                           |
|    | GPU支持。将检查缓存指针，且HCOLL回退到GPU缓存情形下的运行时。  |
+----+----------------------------------------------------------------+

*局限性*

对于MPI_Allreduce的GPU缓存支持，不是所有(OP, DTYPE)的组合都支持：

-  支持的操作

   -  SUM

   -  PROD

   -  MIN

   -  MAX

-  支持的类型

   -  INT8、INT16、INT32、INt64

   -  UINT8、UINT16、UINT32、UINT64

   -  FLOAT16、FLOAT32、FLOAT64

.. _局限性-1:

**局限性**

环境变量\ ``HCOLL_ALLREDUCE_ZCOPY_TUNE=<static/dynamic>``\ （默认为dynamic）用于设置HCOLL的大数据全归约操作算法的自动化运行优化级别。如为Static，则对运行时不优化；如是dynamic，则允许HCOLL基于性能的运行时抽样自动调节算法的基数和zero-copy [1]_ 阈值。

注：由于dynamic模式可能会导致浮点数归约结果有变化，因此不应该用于要求是数值可再现的情形下，导致该问题的原因在于非固定的归约顺序。

统一通信-X架构(Unified Communication - X Framework, UCX)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''

UCX是一种新的加速库，并且被集成到OpenMPI（作为pml层）和OpenSHMEM（作为spml层）中作为HPC-X的一部分。这是种开源的通信库，被设计为HPC应用能获得最高的性能。UCX含有广泛范围的优化，能实现在通信方面接近低级别软件开销，接近原生级别的性能。

UCX支持接收端标记匹配、单边通信语义、有效内存注册和各种增强，能有效提高HPC应用的缩放性和性能。

UCX支持：

-  InfiniBand传输：

   -  不可信数据报(Unreliable Datagram, UD)

   -  可信连接(Reliable Connected, RC)

   -  动态连接(Dynamically Connected, DC)

   -  加速verbs(Accelerated verbs)

-  Shared Memory communication with support for KNEM, CMA and XPMEM

-  RoCE

-  TCP

-  CUDA

更多信息，请参见：\ https://github.com/openucx/ucx\ 、\ http://www.openucx.org/

**OpenMPI中使用UCX**

UCX在Open MPI是默认的pml，在OpenSHMEM中是默认的spml，一般安装好设置号后无需用户自己设置就可使用，用户也可利用下面方式显式指定：

-  在Open MPI中显式指定采用UCX：

   ``mpirun --mca pml ucx --mca osc ucx ...``

-  在OpenSHMEM显示指定采用UCX：

   ``oshrun --mca spml ucx ...``

**调整UCX**
       

检查UCX的版本：

``$HPCX_UCX_DIR/bin/ucx_info -v``

UCX的参数可通过下述方法之一设置：

-  通过mpirun设置：

   ``mpirun -x UCX_RC_VERBS_RX_MAX_BUFS=128000 <...>``

-  通过SHELL设置：

   ``export UCX_RC_VERBS_RX_MAX_BUFS=128000``

   ``mpirun <...>``

-  从命令行选择采用的传输：

   ``mpirun -mca pml ucx -x UCX_TLS=sm,rc_x ...``

   上述命令设置了采用pml ucx和设定其用于使用、共享内存和加速传输verbs。

-  为了提高缩放性能，可以加大DC传输时使用的网卡的DC发起者(DCI)的QPs数

   ``mpirun -mca pml ucx -x UCX_TLS=sm,dc_x -x UCX_DC_MLX5_NUM_DCI=16``

   对于大规模系统，当DC传输不可用或者被禁用时，UCX将回退到UD传输。

   在256个连接建立后，RC传输将被禁用，该值可以利用\ ``UCX_RC_MAX_NUM_EPS``\ 环境变量加大。

-  设置UCX使用zero-copy时的阈值

   ``mpirun -mca pml ucx -x UCX_ZCOPY_THRESH=16384``

   默认UCX会自己计算优化的该阈值，需要时可利用上面环境变量覆盖掉。

-  利用\ ``UCX_DC_MLX5_TX_POLICY=<policy>``\ 环境变量设定端点如何选择DC。策略<policy>可以为：

   -  dcs：端点或者采用已指定的DCI或DCI是LIFO顺序分配的，且在不在有操作需要时释放。

   -  dcs_quota：类似dcs。另外，该DCI将在发送超过一定配额时，且有端点在等待DCI时被释放。该DCI一旦完成其所有需要的操作后就被释放。该策略确保了在端点间没有饥荒。

   -  rand：每个端点被赋予一个随机选择的DCI。多个端点有可能共享相同的DCI。

-  利用UCX
   CUDA内存钩子也许在静态编译CUDA应用时不会生效，作为一个工作区，可利用下面选项扩展配置：

   ``-x UCX_MEMTYPE_CACHE=0 -x HCOLL_GPU_CUDA_MEMTYPE_CACHE_ENABLE=0 -x HCOLL_GPU_ENABLE=1``

-  GPUDirectRDMA性能问题可以通过分离协议禁止：

   ``-x UCX_RNDV_SCHEME=get_zcopy``

-  共享内存新传输协议命名为：TBD

   可用的共享内存传输名是：posix、sysv和xpmem

   sm和mm将被包含咋以上三种方法中。

   设备‘device’名对于共享内存传输是‘memory’（在\ ``UCX_SHM_DEVICES``\ 中使用）

**UCX特性**

*硬件标识符匹配(Tag Matching)*
                            

从ConnectX-5起，在UCX中之前由软件负责的标识符匹配工作可以卸载到HCA。对于MPI应用发送消息时附带的数值标志符的加速对收到消息的处理，可以提高CPU利用率和降低等待消息的延迟。在标识符匹配中，由软件控制的匹配入口表称为匹配表。每个匹配入口含有一个标志符及对应一个应用缓存的指针。匹配表被用于根据消息标志引导到达的消息到特定的缓存。该传输匹配表和寻找匹配入口的动作被称为标识符匹配，该动作由HCA而不再由CPU实现。在当收到的消息被不按照到达顺序而是基于与发送者相关的数值标记使用时，非常有用。

硬件标识符匹配使得省下的CPU可供其它应用使用。当前硬件标识符匹配对于加速的RC和DC传输(RC_X和DC_X)是支持的，且可以在UCX中利用下面环境参数启用：

-  对RC_X传输： ``UCX_RC_MLX5_TM_ENABLE=y``

-  对DC_X传输： ``UCX_DC_MLX5_TM_ENABLE=y``

默认，只有消息大于一定阈值时才卸载到传输。该阈值由\ ``UCXTM_THRESH``\ 环境变量控制，默认是1024比特。

对于硬件标识符匹配，特定阈值时，UCX也许用回弹缓冲区(bounce
buffer)卸载内部预注册缓存代替用户缓存。该阈值由\ ``UCX_TM_MAX_BB_SIZE``\ 环境变变量控制，该值等于或小于分片大小，且必须大于\ ``UCX_TM_THRESH``\ 才能生效（默认为1024比特，即默认优化是被禁止的）。

*CUDA GPU*
        
HPC-X中的CUDA环境支持，使得HPC-X在针对点对点和集合函数的UCX和HCOLL通信库中使用各自NVIDIA GPU显存。

系统已安装了NVIDIA peer memory，支持\ `GPUDirect
RDMA <https://www.mellanox.com/products/GPUDirect-RDMA>`__\ 。

*片上内存(MEMIC)*

片上内存允许从UCX层发送消息时使用设备上的内存，该特性默认启用。它仅支持UCX中的rc_x和dc_x传输。

控制这些特性的环境变量为：

-  ``UCX_RC_MLX5_DM_SIZE``

-  ``UCX_RC_MLX5_DM_COUNT``

-  ``UCX_DC_MLX5_DM_SIZE``

-  ``UCX_DC_MLX5_DM_COUNT``

针对这些参数的更多信息，可以运行ucx_info工具查看：\ ``$HPCX_UCX_DIR/bin/ucx_info -f``\ 。

**生成Open MPI/OpenSHMEM的UCX统计信息**
                                   

为生成统计信息，需设定统计目的及触发器，它们可被选择性过滤或/和格式化。

-  统计目的可以利用\ ``UCX_STATS_DEST``\ 环境变量设置，其值可以为下列之一：

   +---------------------+-----------------------------------------------+
   | 空字符串            | 不会生成统计信息                              |
   +=====================+===============================================+
   | file:<filename>     | 存到一个文件中，具有以下替换：%h: host,       |
   |                     | %p:pid, %c:cpu, %t: time,                     |
   |                     | %e:exe，如文件名有%h，则自动替换为节点名      |
   +---------------------+-----------------------------------------------+
   | stderr              | 显示标准错误信息                              |
   +---------------------+-----------------------------------------------+
   | stdout              | 显示标准输出                                  |
   +---------------------+-----------------------------------------------+
   | udp:<host>[:<port>] | 通过UDP协议发送到host:port                    |
   +---------------------+-----------------------------------------------+

   比如：

   ``export UCX_STATS_DEST="file:ucx_%h_%e_%p.stats"``

   ``export UCX_STATS_DEST="stdout"``

-  触发器通过\ ``UCX_STATS_TRIGGER``\ 环境变量设置，其值可以为下述之一：

   ================ ============================
   exit             在程序退出前存储统计信息
   ================ ============================
   timer:<interval> 每隔一定时间存储统计信息
   signal:<signo>   当进程收到信号时存储统计信息
   ================ ============================

   比如：

   ``export UCX_STATS_TRIGGER=exit``
   ``export UCX_STATS_TRIGGER=timer:3.5``

-  利用\ ``UCX_STATS_FILTER``\ 环境变量可以过滤报告中的计数器。它接受以,分割的一组匹配项以指定显示的计数器，统计概要将包含匹配的计数器，批配项的顺序是没关系的。列表中的每个表达式可以包含任何以下选项：

   == ======================================
   \* 匹配任意字符，包含没有（显示全部报告）
   == ======================================
   ?  匹配任意单子字符
   \  匹配在括号中的一个字符
   \  匹配从括号中一定范围的字符
   == ======================================

   关于该参数的更多信息可以参见：\ https://github.com/openucx/ucx/wiki/Statistics\ 。

-  利用\ ``UCX_STATS_FORMAT``\ 环境参数可以控制统计的格式：

   ======= ======================================================
   full    每个计数器都将被在一个单行中显示
   ======= ======================================================
   agg     每个计数器将在一个单行中显示，但是将会聚合类似的计数器
   summary 所有计数器将显示在同一行
   ======= ======================================================

   注意：统计特性只有当编译安装UCX库时打开启用统计标记的时候才生效。默认为No，即不启用。因此为了使用统计特性，请重新采用文件编译UCX，或采用debug版本的UCX，可以在$HPCX_UCX_DIR/debug中找到：

   ``mpirun -mca pml ucx -x LD_PRELOAD=$HPCX_UCX_DIR/debug/lib/libucp.so ...``

   注意：采用上面提到的重新编译的UCX将会影响性能。

PGAS共享内存访问(OpenSHMEM)
'''''''''''''''''''''''''''

共享内存(SHMEM)子程序为高级并行扩展程序提供了低延迟、高带宽的通信。这些子程序在SHMEM
API中提供了用于在协作并行进程间交换数据的编程模型。SHMEM
API可在同一个并行程序中独自或与MPI子程序一起使用。

SHMEM并行编程库是一种非常简单易用的编程模型，可以使用高效的单边通讯API为共享或分布式内存系统提供直观的全局观点接口。

SHMEM程序是单程序多数据(SPMD)类型的。所有的SHMEM进程，被引用为进程单元(PEs)，同时启动且运行相同程序。通常，PEs在它们大程序中自己的子域进行计算，并且周期性与其它下次通讯依赖的PEs进行通信实现数据交换。

SHMEM子程序最小化数据传输请求、最大化带宽以及最小化数据延迟（从一个PE初始化数据传输到结束时的时间周期差）。

SHMEM子程序通过以下支持远程数据传输：

-  put操作：传递数据给一个不同PE；

-  get操作：从一个不同PE和远程指针获取数据，允许直接访问属于其它PE的数据。

其它支持的操作是集合广播和归约、栅栏同步和原子内存操作(atomic memory
operation)。原子内存操作指的是原子（不允许多个进程同时）读-更新操作，比如对远程或本地数据的获取-增加。

SHMEM库实现激活消息。源处理器将数据传输到目的处理器时仅涉及一个CPU，例如，一个处理器从另外处理器内存读取数据而无需中断远程CPU，除非编程者实现了一种机制去告知这些，否则远程处理器察觉不到其内存被读写。

**HPC-X Open MPI/OpenSHMEM**

HPC-X Open MPI/OpenSHMEM编程库是单边通信库，支持唯一的并行编程特性集合，包括并行程序应用进程间使用的点对点和集合子程序、同步、原子操作、和共享内存范式。

HPC-X OpenSHMEM基于OpenSHMEM.org协会定义的API，该库可在OFED(OpenFabrics
RDMA for Linux stack )上运行，并可使用UCX和Mellanox
FCA，为运行在InfiniBand上的SHMEM程序提供了史无前例的可扩展性级别。

**运行HPC-X OpenSHMEM**
                   
*采用UCX运行HPC-X OpenSHMEM*
                          

对于HPC-X，采用spml对程序提供服务。v2.1及之后的版本的HPC-X，ucx已是默认的spml，无需特殊指定，或者也可在oshrun命令行添加\ ``-mca spml ucx``\ 显式指定。

所有的UCX环境参数，oshrun使用时与mpirun一样，完整的列表可运行下面命令获取：

``$HPCX_UCX_DIR/bin/ucx_info -f``

*采用HPC-X OpenSHMEM与MPI一起开发应用*
                                    

SHMEM编程模型提供了一种提高延迟敏感性部分的性能的方法。通常，要求采用调用shmem_put/shmem_get和shmem_barrier来代替调用MPI中的send/recv。

SHMEM模型对于短消息来说可以相比传统的MPI调用能显著降低延时。对于MPI-2
MPI_Put/MPI_Get函数，也可以考虑替换为shmem_get/shmem_put调用。

*HPC-X OpenSHMEM调整参数*
                       
 HPC-X OpenSHMEM采用MCA参数来调整设置用户应用运行时环境。每个参数对应一种特定函数，以下为可以改变用于应用函数的参数：

-  memheap：控制内存分配策略及阈值

-  scoll：控制HPC-X OpenSHMEM集合API阈值及算法

-  spml：控制HPC-X OpenSHMEM点对点传输逻辑及阈值

-  atomic：控制HPC-X OpenSHMEM原子操作逻辑及阈值

-  shmem：控制普通HPC-X OpenSHMEM API行为

显示HPC-X OpenSHMEM参数：

-  显示所有可用参数：

   -  ``oshmem_info -a``

-  显示HPC-X OpenSHMEM特定参数：

   -  ``oshmem_info --param shmem all``

   -  ``oshmem_info --param memheap all``

   -  ``oshmem_info --param scoll all``

   -  ``oshmem_info --param spml all``

   -  ``oshmem_info --param atomic all``

：在所有节点上运行OpenSHMEM应用或性能测试时，需要执行以下命令以释放内存：

``echo 3 > /proc/sys/vm/drop_caches``

*针对对称堆(Symmetric Heap)应用的OpenSHMEM MCA参数*
                                                 

SHMEM memheap大小可以通过对oshrun命令添加\ ``SHMEM_SYMMETRIC_HEAP_SIZE``\ 参数来设置，默认为256M。

例如，采用64M memheap来运行SHMEM：

``oshrun -x SHMEM_SYMMETRIC_HEAP_SIZE=64M -np 512 -mca mpi_paffinity_alone 1 \\``

``--map-by node -display-map -hostfile myhostfile example.exe``

memheap可以采用下述方法分配：

-  sysv：system V共享内存API，目前不支持采用大页面(hugepages)分配。

-  verbs：采用IB verbs分配子。

-  mmap：采用mmap()分配内存。

-  ucx： 通过UCX库分配和注册内存

默认，HPC-X OpenSHMEM会自己寻找最好的分配子，优先级为verbs、sysv、mmap和ucx，也可以采用-mca sshmem <name>指定分配方法。

*用于强制连接生成的参数*
                      

通常，SHMEM会在PE间消极地生成连接，一般是在第一个通信发生时。

-  开始时就强制连接生成，设定MCA参数：

   ``-mca shmem_preconnect_all 1``

   内存注册器（如，infiniband rkeys）信息启动时会在进程间交换。

-  启用按需内存密钥(key)交换，可设置MCA参数：

   ``-mca shmalloc_use_modex 0``

.. _openmpi:

Open MPI库
^^^^^^^^^^

| Open MPI [2]_ 库是另一种非常优秀MPI实现，用户如需使用可以自己通过运行
| ``module load``\ 选择加载与openmpi相关的项自己设置即可。

Open MPI的安装 ``/opt/opnempi`` 目录在下。

Open MPI的编译命令主要为：

-  C程序：\ ``mpicc``

-  C++程序：\ ``mpic++``\ 、\ ``mpicxx``\ 、\ ``mpiCC``

-  Fortran 77程序：\ ``mpif77``\ 、\ ``mpif90``\ 、\ ``mpifort``

-  Fortran 90程序：\ ``mpif90``

-  Fortran程序：\ ``mpifort``\  [3]_

``mpifort``\ 为1.8系列引入的编译Fortran程序的命令。

``mpif77``\ 和\ ``mpif90``\ 为1.6系列和1.8系列的编译Fortran程序的命令。

对于MPI并行程序，对应不同类型源文件的编译命令如下：

-  | 将C语言的MPI并行程序yourprog-mpi.c编译为可执行文件yourprog-mpi：
   | ``mpicc -o yourprog-mpi yourprog-mpi.c``

-  | 将C++语言的MPI并行程序yourprog-mpi.cpp编译为可执行文件yourprog-mpi，也可换为\ ``mpic++``\ 或\ ``mpiCC``\ ：
   | ``mpicxx -o yourprog-mpi yourprog-mpi.cpp``

-  | 将Fortran
     90语言的MPI并行程序yourprog-mpi.f90编译为可执行文件yourprog-mpi：
   | ``mpifort -o yourprog-mpi yourprog-mpi.f90``

-  | 将Fortran
     77语言的MPI并行程序yourprog-mpi.f编译为可执行文件yourprog-mpi：
   | ``mpif77 -o yourprog-mpi yourprog-mpi.f``

-  | 将Fortran
     90语言的MPI并行程序yourprog-mpi.f90编译为可执行文件yourprog-mpi：
   | ``mpif90 -o yourprog-mpi yourprog-mpi.f90``

编译命令的基本语法为：\ ``\ [-showme|-showme:compile|-showme:link] ...``

编译参数可以为：

-  --showme：显示所调用的编译器所调用编译参数等信息。

-  –showme:compile：显示调用的编译器的参数

-  –showme:link：显示调用的链接器的参数

-  –showme:command：显示调用的编译命令

-  –showme:incdirs：显示调用的编译器所使用的头文件目录，以空格分隔。

-  –showme:libdirs：显示调用的编译器所使用的库文件目录，以空格分隔。

-  –showme:libs：显示调用的编译器所使用的库名，以空格分隔。

-  –showme:version：显示Open MPI的版本号。

默认使用配置Open MPI时所用的编译器及其参数，可以利用环境变量来改变。环境变量格式为\ ``OMPI_value``\ ，其\ ``value``\ 可以为：

-  ``CPPFLAGS``\ ：调用C或C++预处理器时的参数

-  ``LDFLAGS``\ ：调用链接器时的参数

-  ``LIBS``\ ：调用链接器时所添加的库

-  ``CC``\ ：C编译器

-  ``CFLAGS``\ ：C编译器参数

-  ``CXX``\ ：C++编译器

-  ``CXXFLAGS``\ ：C++编译器参数

-  ``F77``\ ：Fortran 77编译器

-  ``FFLAGS``\ ：Fortran 77编译器参数

-  ``FC``\ ：Fortran 90编译器

-  ``FCFLAGS``\ ：Fortran 90编译器参数

Intel MPI库
^^^^^^^^^^^

Intel MPI针对最新的Mellanox HDR有问题，不建议使用；如您的应用运行起来没问题，也可以使用。

Intel MPI库 [4]_ 是一种多模消息传递接口(MPI)库，所安装的5.0版本Intel MPI库实现了\ `MPI V3.0标准 <http://www.mpi-forum.org/>`__\ 。Intel MPI库可以使开发者采用新技术改变或升级其处理器和互联网络而无需改编软件或操作环境成为可能。主要包含以下内容：

-  Intel
   MPI库运行时环境(RTO)：具有运行程序所需要的工具，包含多功能守护进程(MPD)、Hydra及支持的工具、共享库(.so)和文档。

-  Intel
   MPI库开发套件(SDK)：包含所有运行时环境组件和编译工具，含编译器命令，如\ ``mpiicc``\ 、头文件和模块、静态库(.a)、调试库、追踪库和测试代码。

编译命令
''''''''

请注意，Intel MPI与Open MPI等MPI实现不同，\ ``mpicc``\ 、\ ``mpif90``\ 和\ ``mpifc``\ 命令默认使用GNU编译器，如需指定使用Intel编译器等，请使用对应的\ ``mpiicc``\ 、\ ``mpiicpc``\ 和\ ``mpiifort``\ 命令。下表为Intel MPI编译命令及其对应关系。

.. container::
   :name: impi

   .. table:: Intel MPI编译命令及其对应关系

      +----------+----------------------+-------------------------+----------------------+
      | 编译命令 | 调用的默认编译器命令 | 支持的语言              | 支持的应用二进制接口 |
      +==========+======================+=========================+======================+
      |          |                      |                         |                      |
      +----------+----------------------+-------------------------+----------------------+
      | mpicc    | gcc, cc              | C                       | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpicxx   | g++                  | C/C++                   | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpifc    | gfortran             | Fortran77*/Fortran 95\* | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      |          |                      |                         |                      |
      +----------+----------------------+-------------------------+----------------------+
      | mpigcc   | gcc                  | C                       | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpigxx   | g++                  | C/C++                   | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      |          |                      |                         |                      |
      +----------+----------------------+-------------------------+----------------------+
      | mpif77   | g77                  | Fortran 77              | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpif90   | gfortran             | Fortran 95              | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      |          |                      |                         |                      |
      +----------+----------------------+-------------------------+----------------------+
      | mpiicc   | icc                  | C                       | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpiicpc  | icpc                 | C++                     | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+
      | mpiifort | ifort                | Fortran77/Fortran 95    | 32/64 bit            |
      +----------+----------------------+-------------------------+----------------------+

其中：

-  ia32：IA-32架构。

-  intel64：Intel 64(x86_64, amd64)架构。

-  移植现有的MPI程序到Intel MPI库时，请重新编译所有源代码。

-  如需显示某命令的简要帮助，可以不带任何参数直接运行该命令。

编译命令参数
''''''''''''

-  -mt_mpi：采用以下级别链接线程安全的MPI库：MPI_THREAD_FUNNELED, MPI_THREAD_SERIALIZED或MPI_THREAD_MULTIPLE。

   Intel MPI库默认使用MPI_THREAD_FUNNELED级别线程安全库。

   注意：

   -  如使用Intel C编译器编译时添加了-openmp、-qopenmp或-parallel参数，那么使用线程安全库。

   -  如果用Intel Fortran编译器编译时添加了如下参数，那么使用线程安全库：

      -  -openmp

      -  -qopenmp

      -  -parallel

      -  -threads

      -  -reentrancy

      -  -reentrancy threaded

-  -static_mpi：静态链接Intel MPI库，并不影响其它库的链接方式。

-  -static：静态链接Intel MPI库，并将其传递给编译器，作为编译器参数。

-  -config\ *=name*\ ：使用的配置文件。

-  -profile\ *=profile_name*\ ：使用的MPI分析库文件。

-  -t或-trace：链接Intel Trace Collector库。

-  -check_mpi：链接Intel Trace Collector正确性检查库。

-  -ilp64：打开局部ILP64支持。对于Fortran程序编译时如果使用-i8选项，那么也需要此ILP64选项。

-  -dynamic_log：与-t组合使用链接Intel Trace Collector库。不影响其它库链接方式。

-  -g：采用调试模式编译程序，并针对Intel MPI调试版本生成可执行程序。可查看官方手册Environment variables部分\ ``I_MPI_DEBUG``\ 变量查看-g参数添加的调试信息。采用调试模式时不对程序进行优化，可查看\ ``I_MPI_LINK``\ 获取Intel MPI调试版本信息。

-  -link_mpi\ *=arg*\ ：指定链接MPI的具体版本，具体请查看\ ``I_MPI_LINK``\ 获取Intel MPI版本信息。此参数将覆盖掉其它参数，如-mt_mpi、-t=log、-trace=log和-g。

-  -O：启用编译器优化。

-  -fast：对整个程序进行最大化速度优化。此参数强制使用静态方法链接Intel MPI库。\ ``mpiicc``\ 、\ ``mpiicpc``\ 和\ ``mpiifort``\ 编译命令支持此参数。

-  -echo：显示所有编译命令脚本做的信息。

-  -show：仅显示编译器如何链接，但不实际执行。

-  -{cc,cxx,fc,f77,f90}\ *=compiler*\ ：选择使用的编译器。如：\ ``mpicc -cc=icc -c test.c``\ 。

-  -gcc-version\ *=nnn*\ ，设置编译命令\ ``mpicxx``\ 和\ ``mpiicpc``\ 编译时采用部分GNU C++环境的版本，如nnn的值为340，表示对应GNU C++ 3.4.x。

   ============================= =============
   :math:`<`\ nnn\ :math:`>`\ 值 GNU\* C++版本
   ============================= =============
   320                           3.2.x
   330                           3.3.x
   340                           3.4.x
   400                           4.0.x
   410                           4.1.x
   420                           4.2.x
   430                           4.3.x
   440                           4.4.x
   450                           4.5.x
   460                           4.6.x
   470                           4.7.x
   ============================= =============

-  -compchk：启用编译器设置检查，以保证调用的编译器配置正确。

-  -v：显示版本信息。

环境变量
''''''''

-  ``I_MPI_{CC,CXX,FC,F77,F90}_PROFILE``\ 和\ ``MPI{CC,CXX,FC,F77,F90}_PROFILE``\ ：

   -  默认分析库。

   -  语法：\ ``I_MPI_{CC,CXX,FC,F77,F90}_PROFILE=<profile_name>``\ 。

   -  过时语法：\ ``MPI{CC,CXX,FC,F77,F90}_PROFILE=<profile_name>``\ 。

-  ``I_MPI_TRACE_PROFILE``\ ：

   -  设定-trace参数使用的默认分析文件。

   -  语法：\ ``I_MPI_TRACE_PROFILE=<profile_name>``

   -  ``I_MPI_{CC,CXX,F77,F90}_PROFILE``\ 环境变量将覆盖掉\ ``I_MPI_TRACE_PROFILE``\ 。

-  ``I_MPI_CHECK_PROFILE``\ ：

   -  设定-check_mpi参数使用的默认分析。

   -  语法：\ ``I_MPI_CHECK_PROFILE=<profile_name>``\ 。

-  ``I_MPI_CHECK_COMPILER``\ ：

   -  设定启用或禁用编译器兼容性检查。

   -  语法：\ ``I_MPI_CHECK_COMPILER=<arg>``\ 。

      -  ``<arg>``\ 为\ ``enable | yes | on | 1``\ 时打开兼容性检查。

      -  ``<arg>``\ 为\ ``disable | no | off | 0``\ 时，关闭编译器兼容性检查，为默认值。

-  ``I_MPI_{CC,CXX,FC,F77,F90}``\ 和\ ``MPICH_{CC,CXX,FC,F77,F90}``\ ：

   -  语法：\ ``I_MPI_{CC,CXX,FC,F77,F90}=<compiler>``\ 。

   -  过时语法：\ ``MPICH_{CC,CXX,FC,F77,F90}=<compiler>``\ 。

   -  ``<compiler>``\ 为编译器的编译命令名或路径。

-  ``I_MPI_ROOT``\ ：

   -  设置Intel MPI库的安装目录路径。

   -  语法：\ ``I_MPI_ROOT=<path>``\ 。

   -  ``<path>``\ 为Intel MPI库的安装后的目录。

-  ``VT_ROOT``\ ：

   -  设置Intel Trace Collector的安装目录路径。

   -  语法：\ ``VT_ROOT=<path>``\ 。

   -  ``<path>``\ 为Intel Trace Collector的安装后的目录。

-  ``I_MPI_COMPILER_CONFIG_DIR``\ ：

   -  设置编译器配置目录路径。

   -  语法：\ ``I_MPI_COMPILER_CONFIG_DIR=<path>``\ 。

   -  ``<path>``\ 为编译器安装后的配置目录，默认值为\ ``<installdir>/<arch>/etc``\ 。

-  ``I_MPI_LINK``\ ：

   -  设置链接MPI库版本。

   -  语法：\ ``I_MPI_LINK=<arg>``\ 。

   -  ``<arg>``\ 可为：

      -  ``opt``\ ：优化的单线程版本Intel MPI库；

      -  ``opt_mt``\ ：优化的多线程版本Intel MPI库；

      -  ``dbg``\ ：调试的单线程版本Intel MPI库；

      -  ``dbg_mt``\ ：调试的多线程版本Intel MPI库；

      -  ``log``\ ：日志的单线程版本Intel MPI库；

      -  ``log_mt``\ ：日志的多线程版本Intel MPI库。

编译举例
''''''''

对于MPI并行程序，对应不同类型源文件的编译命令如下：

-  | 调用默认C编译器将C语言的MPI并行程序yourprog-mpi.c编译为可执行文件yourprog-mpi：
   | ``mpicc -o yourprog-mpi yourprog-mpi.c``

-  | 调用Intel
     C编译器将C语言的MPI并行程序yourprog-mpi.c编译为可执行文件yourprog-mpi：
   | ``mpiicc -o yourprog-mpi yourprog-mpi.cpp``

-  | 调用Intel
     C++编译器将C++语言的MPI并行程序yourprog-mpi.cpp编译为可执行文件yourprog-mpi：
   | ``mpiicxx -o yourprog-mpi yourprog-mpi.cpp``

-  | 调用GNU Forttan编译器将Fortran
     77语言的MPI并行程序yourprog-mpi.f编译为可执行文件yourprog-mpi：
   | ``mpif90 -o yourprog-mpi yourprog-mpi.f``

-  | 调用Intel Fortran编译器将Fortran
     90语言的MPI并行程序yourprog-mpi.f90编译为可执行文件yourprog-mpi：
   | ``mpiifort -o yourprog-mpi yourprog-mpi.f90``

调试
''''

使用以下命令对Intel MPI库调用GDB调试器： ``mpirun -gdb -n 4 ./testc``

可以像使用GDB调试串行程序一样调试。

也可以使用以下命令附着在一个运行中的作业上：

``mpirun -n 4 -gdba <pid>``

其中<pid>为运行中的MPI作业进程号。

环境变量I_MPI_DEBUG提供一种获得MPI应用运行时信息的方式。可以设置此变量的值从0到1000，值越大，信息量越大。

``mpirun -genv I_MPI_DEBUG 5 -n 8 ./my_application``

更多信息参见程序调试章节。

追踪
''''

使用-t或-trace选项链接调用Intel Trace Collector库生成可执行程序。此与当在mpiicc或其它编译脚本中使用-profile=vt时具有相同的效果。

``mpiicc -trace test.c -o testc``

在环境变量VT_ROOT中包含用Intel Trace Collector库路径以便使用此选项。设置I_MPI_TRACE_PROFILE为<profile_name>环境变量指定另一个概要库。如设置
I_MPI_TRACE_PROFILE为vtfs，以链接fail-safe版本的Intel Trace Collector库。

正确性检查
''''''''''

使用-check_mpi选项调用Intel Trace Collector正确性检查库生成可执行程序。此与当在mpiicc或其它编译脚本中使用-profile=vtmc时具有相同的效果。

``mpiicc -profile=vtmc test.c -o testc``

或

``mpiicc -check_mpi test.c -o testc``

在环境变量VT_ROOT中包含用Intel Trace Collector库路径以便使用此选项。设置I_MPI_CHECK_PROFILE为<profile_name>环境变量指定另一个概要库。

统计收集
''''''''

如果想收集在应用使用的MPI函数统计，可以设置I_MPI_STATS环境变量的值为1到10。设置好后再运行MPI程序，则在stats.txt文件中存储统计信息。

与编译器相关的编译选项
^^^^^^^^^^^^^^^^^^^^^^

MPI编译环境的编译命令实际上是调用Intel、PGI或GCC编译器进行编译，具体优化选项等，请参看Intel
MPI、Open MPI以及Intel、PGI和GCC编译器手册。

MPI并行程序的运行
~~~~~~~~~~~~~~~~~

MPI程序最常见的并行方式类似为：\ ``mpirun -n 40 yourmpi-prog``\ 。

在本超算系统上，MPI并行程序需结合Slurm作业调度系统的作业提交命令\ ``sbatch``\ 、\ ``srun``\ 或\ ``salloc``\ 等来调用作业脚本运行，请参看。

.. [1]
   zero copy技术就是减少不必要的内核缓冲区跟用户缓冲区间的拷贝，从而减少CPU的开销和内核态切换开销，达到性能的提升

.. [2]
   主页：\ \ http://www.open-mpi.org/

.. [3]
   注意为mpifort，而不是Intel MPI的mpifort

.. [4]
   主页：\ \ http://software.intel.com/en-us/intel-mpi-library/
