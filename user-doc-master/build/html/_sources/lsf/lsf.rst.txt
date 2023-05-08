LSF作业调度系统
================

曙光TC4600百万亿次超级计算系统利用IBM Spectrum LSF 10.1.0进行资源和作业调度管理，所有需要运行的作业均必须通过作业提交命令\ ``bsub``\ 提交，提交后可利用相关命令查询作业状态等。为了利用\ ``bsub``\ 提交作业，需要在\ ``bsub``\ 中指定各选项和需要执行的程序。注意：

-  不要在登录节点(tc4600)上不通过作业调度管理系统直接运行作业（编译等日常操作除外），以免影响其余用户的正常使用。

-  如果不通过作业调度管理系统直接在计算节点上运行将会被监护进程直接杀掉。

作业运行的条件
~~~~~~~~~~~~~~

作业提交后需要一段时间等待作业调度系统调度运行，一般为先提交的先运行，并且作业运行需要满足多个基本条件：

-  系统有空闲资源，满足程序运行需要。可以利用\ ``bhosts``\ 命令查看，ok状态的才可以接受作业运行。

-  用户作业没有超过系统设置的允许用户运行的作业数，可以利用\ ``busers``\ 命令查看。

-  用户作业没有超过所使用的作业队列的允许作业核数的限制，可以利用\ ``bqueues``\ 命令查看。

-  用户作业没有被挂起等。利用\ ``bjobs``\ 命令查看。

-  作业调度管理系统工作正常。

当系统作业繁忙时，如果提交需要核数很多的作业，也许需要长时间才可以运行甚至根本无法获取到足够资源来运行，请考虑选择合适的并行规模。

作业如不运行，请先请运行\ ``bjobs -l JobID``\ 或\ ``bjobs -p JobID``\ 查看输出信息中PENDING
REASONS部分及提交时设置的参数等，并结合运行上述几个命令查看原因。

如果上述条件都符合，也许作业调度管理系统存在问题，请与超算中心工作人员联系处理。

查看队列情况：bqueues
~~~~~~~~~~~~~~~~~~~~~

用户在使用时，首先需要了解哪些队列可以使用，利用\ ``bqueues``\ 可以查看现有队列信息。

具体队列会根据需要更改，请注意登录系统后的提示，或运行\ ``bqueues -l``\ 查看。

``bqueues``

将输出：

::
    
    QUEUE_NAME PRIO STATUS       MAX  JL/U  JL/P JL/H NJOBS  PEND   RUN  SUSP
    long        59  Open:Active  1200 1200    -    -     0     0     0     0
    normal      58  Open:Inact   1440  192    -    -   360     0   360     0
    small       57  Closed:Active 720  120    -    -  1976  1476   500     0

其中，主要列的含义为：

-  QUEUE_NAME：队列名

-  PRIO：优先级，数字越大优先级越高

-  STATUS：状态

   -  Open：队列开放，可以接受提交新作业

   -  Active：队列已激活，队列中未开始运行的作业可以开始运行

   -  Closed：队列已关闭，不接受提交新作业

   -  Inact：队列未激活，可接受提交新作业，但队列中的等待运行的作业不会开始运行

-  MAX：队列对应的最大作业槽数（Job
   Slot，一般与CPU核数一致，以下通称CPU核数），-表示无限

-  JL/U：单个用户同时可以使用的CPU核数

-  JL/P：每个处理器可以接受的CPU核数

-  JL/H：每个节点可以接受的CPU核数

-  NJOBS：排队、运行和被挂起的总作业所占CPU核数

-  PEND：排队中的作业所需CPU核数

-  RUN：运行中的作业所占CPU核数

-  SUSP：被挂起的作业所占CPU核数

-  RSV：为排队作业预留的CPU核数

查看队列详细情况：bqueues -l
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

队列策略也许会调整，利用\ ``bqueues -l [队列名]``\ 查看各队列的详细情况：

::
    
    QUEUE: small
     -- Maximum number of job slots that each user can use in this queue is 720.
     on nodes: node1~node67

    PARAMETERS/STATISTICS
    PRIO NICE STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN SSUSP USUSP  RSV
     57   20  Open:Active     720  120    -    -  1964  1464   500     0     0    0
    Interval for a host to accept two jobs is 0 seconds

     PROCLIMIT
     48

    SCHEDULING PARAMETERS
               r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
     loadSched   -     -     -     -       -     -    -     -     -      -      -
     loadStop    -     -     -     -       -     -    -     -     -      -      -

              adapter_windows     poe nrt_windows
     loadSched             -       -           -
     loadStop              -       -           -

    SCHEDULING POLICIES:  EXCLUSIVE

    USERS: paiduigroup/ scc_group/
    HOSTS:  node1 node2 node3 node4 node5 node6 node7 node8 node9 node10 node11 node12
    RES_REQ:  span[ptile=24]


-  QUEUE：队列名，跟着的下一行是描述

-  PRIO：优先值，越大越优先

-  NICE：作业运行时的nice值，即作业运行时的操作系统调度优先值，从-20到19，越小越优先

-  RUNLIMIT：作业单CPU运行时间限制，以系统中的某个节点为基准，如为1440.0 min则表示可以运行一天（60*24）

-  CPULIMIT：单个作业运行时间限制，以系统中的某个节点E5410作为参考，运行机时（核数*墙上时间）为345600.0 CPU分钟，即如用8 CPU核计算，允许运行30天

-  PROCLIMIT：单个作业核数限制，\ ``2 4 8``\ ，表示使用此队列时，最少使用2个核，最多使用8核，如提交时没用-n指定具体核数，则使用默认4核

-  PROCESSLIMIT：单个作业最大核数限制，为8

-  MEMLIMIT：每个进程能使用的内存数，默认以KB为单位

-  THREADLIMIT：作业最大线程数

-  USERS：有权使用的用户

-  HOSTS：对应的节点

查看各节点的运行情况：lsload
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``lsload``\ 命令可查看当前各节点的运行情况，例如：

``lsload``

::

    HOST_NAME     status  r15s   r1m  r15m    ut   pg   ls  it   tmp   swp   mem
    node1         ok       0.0   0.0   0.0   0%   0.5   0   25  227G   32G   62G
    node2         locku    0.1   0.0   0.0   0%   0.4   0   60  227G   32G   62G
    node4         unavail   -     -     -    -      -   -    -     -     -     -

-  HOST_NAME：节点名。

-  status：状态。

-  status列：

   -  ok：表示可以接受新作业，只有这种状态可以接受新作业

   -  closed：表示系统在运行，但已被调度系统关闭，不接受新作业

   -  locku：表示在进行排他性运行

   -  busy：表示负载超过限定

   -  unavail、-ok：作业调度系统服务有问题

-  r15s、r1m、r15m列：分别表示15秒、1分钟、15分钟平均负载

-  ut：利用率

-  tmp：目录大小

-  swp：swp虚拟内存大小

-  mem：内存大小

-  io：硬盘读写（-l选项时才出现）

查看node2节点：\ ``lsload node2``

查看各节点的空闲情况：bhosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bhosts``\ 命令可查看当前各节点的空闲情况，例如：

``bhosts``

::

    HOST_NAME  STATUS  JL/U  MAX  NJOBS  RUN  SSUSP  USUSP  RSV
    node12         closed   -    16       2    2      0      0    0
    node10         ok       -    16       2    1      0      0    0
    node14         ok       -    16       2    1      0      0    0

-  HOST_NAME：节点名。

-  STATUS：状态。

   -  ok：表示可以接收新作业，只有这种状态可以接受新作业

   -  closed：表示已被作业占满，不接受新作业

   -  unavail和unreach：系统停机或作业调度系统服务有问题

-  JL/U：允许每个用户的作业核数。-表示未限制。

-  MAX：允许最大作业核数。

-  NJOBS：当前运行的作业数。

-  RUN：当前运行作业占据的核数。

-  SSUSP：被系统挂起的作业占据的核数。

-  USUSP：被用户挂起的作业占据的核数。

-  RSV：预留的核数。

查看node1节点：\ ``bhosts node1``

``bhosts -l``\ 会显示节点详细信息，其中slots表示目前最大可以接受作业槽数（默认一般与CPU核数一致）

即使有节点状态为ok状态，也不一定表示您的作业可以运行，具体运行条件参见\ `[run] <#run>`__\ 作业运行条件。

查看用户信息：busers
~~~~~~~~~~~~~~~~~~~~

利用\ ``busers``\ 可以查看用户信息，例如：

``busers hmli``

::

    USER/GROUP JL/P MAX NJOBS PEND RUN SSUSP USUSP RSV
    hmli           -     320     40     32    8      0     0    0

其中：

-  USER/GROUP：用户或组名。

-  JL/U：允许每个用户的作业核数。-表示未限制。

-  MAX：允许最大作业核数。

-  NJOBS：当前运行的作业数。

-  PEND：当前挂起作业占据的核数。

-  RUN：当前运行作业占据的核数。

-  SSUSP：被系统挂起的作业占据的核数。

-  USUSP：被用户挂起的作业占据的核数。

-  RSV：预留的核数。

提交作业：bsub
~~~~~~~~~~~~~~

用户需要利用\ ``bsub``\ 提交作业，其基本格式为\ ``bsub [options] command [arguments]``\ 。其中options设置队列、CPU核数等的选项，必须在command之前，否则将作为command的参数；arguments为设置作业的可执行程序本身所需要的参数，必须在command之后，否则将作为设置队列等的选项。下面将给出常用的几种提交方式。

注意：

-  作业提交后，应经常检查一下作业的CPU、内存等利用率，判断实际运行效率：

   -  可以\ ``ssh``\ 到对应运行作业的节点运行\ ``top``\ 命令；

   -  查看Ganglia系统监控：\ http://scc.ustc.edu.cn/ganglia\ 。

-  请不要ssh到节点后直接运行作业，以免影响作业调度系统分配到此节点的作业。

提交到特定队列：bsub -q
^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-q``\ 选项可以指定提交到哪个队列，，请注意参看登录后的提示或运行\ ``bqueues -l``\ 命令查看，现有的队列为：

-  normal：所需要的CPU核数大于1个且不超过16个时

-  long：所需要的CPU核数超过32个时

-  serial：所需要的CPU核数为一个时

比如想提交到normal队列使用2个CPU核运行程序executable1，可以：

``bsub -q normal -n 2 executable1``

如果提交成功，将显示类似下面的输出：

Job <79722> is submitted to queue <normal>.

其中79722为此作业的作业号，以后可利用此作业号来进行查询及终止等操作。

如提交不成功，则显示相应提示。

提交串行作业：bsub -n 1
^^^^^^^^^^^^^^^^^^^^^^^

我校超算系统鼓励运行并行作业，允许运行的串行作业资源较少，有些系统不允许运行串行作业。

运行串行作业，请使用支持串行队列的队列（假如serial队列支持串行），比如：

``bsub -q serial -n 1 executable-serial``

指明所需要的CPU核数：bsub -n
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-n min\_proc[,max\_proc]``\ 选项指定所需要的CPU核数（一般来说核数和进程数一致），比如下面指定利用24个CPU核（由\ ``-n 24``\ 指定）运行MPI（由\ ``mpijob``\ 指明为运行MPI程序）程序：

``bsub -q small -n 24 mpijob executable-mpi1``

最少采用24最大采用72 CPU核数运行：\ ``bsub -n 24,72``

当作业调度尝试开始运行时：

-  如空闲资源满足72 CPU核，那么将采用72 CPU核运行；

-  如空闲资源不满足72 CPU核，但满足24 CPU核，则用24 CPU核运行。

由于每个节点的CPU核数为24，建议单个作业所使用的核数最好为24或12的整数倍，以尽量保证自己的程序占据独立的节点或半个节点，尽量避免相互影响。以上仅仅是建议，具体申请核数应考虑作业实际情况。即使同一个计算软件，在计算不同条件时，也有可能不一样，请务必仔细研究自己所使用的软件。

提交到特定节点：bsub -m “host_name”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-m``\ 选项可以指定作业在特定节点上运行，如：

:literal:`bsub -m ``node1 node3''`

如非必要，建议不要添加此选项，以免导致作业无法及时运行。

提交MPI作业：bsub -n NUM mpijob
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果需要运行MPI作业，需要利用\ ``mpijob``\ 调用MPI可执行程序，并用-n选项指定所需的CPU核数，比如下面指定利用64颗CPU核运行MPI程序executable-mpi1：

``bsub -q long -n 64 mpijob executable-mpi1``

注意：

-  ``mpijob``\ 命令已经封装了Intel MPI和Open
   MPI的运行MPI作业的脚本，用户一般无需再特别按照Intel MPI和Open
   MPI的原始\ ``mpirun``\ 、\ ``mpiexec``\ 等命令使用。

-  | 提交作业时在\ ``mpijob``\ 之前的参数将传递给LSF，之后的参数将传递给原始的\ ``mpirun``\ 或
   | ``mpiexec``\ 等。

-  如用户对LSF和Intel MPI、Open
   MPI等不熟悉，请勿擅自添加其它参数，如有需要请与超算中心联系。

-  如您了解所使用的MPI环境与LSF的配合，也可不用\ ``mpijob``\ 提交，如直接用\ ``mpirun``\ 等命令运行。

指明需要某种资源作业提交：bsub -R
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:literal:`-R ``res_req'' [-R ``res_req'' ...]`\ 可以使得作业在需要满足某种条件的节点上运行，如：

-  :literal:`-R ``span[hosts=1]''`\ ：指定需要在同一个节点内运行；

-  | :literal:`-R ``span[ptile=8]''`\ ：指定需要在每一个节点内运行多少核，如\ :literal:`bsub -n 16 -R ``span[ptile=8]''`
   | 则会分给2个节点，每个节点8核；

-  :literal:`-R ``1*{mem>5000} + 9*{mem>1000}''`\ ：一个CPU核需要至少5GB内存，另外9个每个至少需要1GB内存。

提交OpenMP等共享内存作业：bsub -R “span[hosts=1]” OMP_NUM_THREADS=
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

由于只能在同一个节点内部运行OpenMP共享内存的作业，如Gaussian程序，此时需要添加利用\ :literal:`-R ``span[hosts=1]''`\ 参数指定使用一个节点，并用OMP_NUM_THREADS设定指定的线程数，一般应与申请的核数一致 [1]_：

指定利用8 CPU核运行OpenMP程序：

:literal:`bsub -q normal -n 8 -R ``span[hosts=1]'' OMP_NUM_THREADS=8 executable-omp1`

MPI和OpenMP共享内存混合并行作业
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

需要针对需求利用LSF环境变量特殊处理，如果自己不清楚怎么处理，请联系管理人员。

给作业起个名字：bsub -P project_name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

提交时可以利用\ ``-P``\ 选项给作业起个名字方便查看，如：

``bsub -P VASPJOB``

运行排他性作业：bsub -x
^^^^^^^^^^^^^^^^^^^^^^^

如需要独占节点运行，此时需要添加-x选项：

``bsub -x -q normal -n 8 executable-omp1``

注意：

-  排他性运行在运行期间，不允许其余的作业提交到运行此作业的节点，并且只有在某节点没有任何其余的作业在运行时才会提交到此节点上运行；

-  如果不需要采用排他性运行，请不要使用此选项，否则将导致作业必须等待完全空闲的节点才会运行，也许将增加等待时间；

-  另外使用排他性运行时，哪怕只使用某节点内的一个CPU核，也将按照此节点内的所有CPU核数进行机时计算。

指明输出、输出文件运行：bsub -i -o -e
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作业的正常屏幕输入文件（指的是类似
方式的文件）、正常屏幕输出到的文件和错误屏幕输出的文件可以利用-i、-o和-e选项来分别指定，运行后可以通过查看指定的这些输出文件来查看运行状态，文件名可利用%J与作业号挂钩。比如指定executable1的输入、正常和错误屏幕输出文件分别为：、和：

``bsub -i executable1.input -o executable1-\%J.log -e executable1-\%J.err executable1``

``-o``\ 和\ ``-e``\ 及变种\ ``-oo``\ 和\ ``-eo``\ ：

-  -o：如日志原文件存在，正常屏幕输出将追加到原文件后

-  -oo：如日志原文件存在，正常屏幕输出将覆盖原文件

-  -e：如日志原文件存在，出错时屏幕输出将追加到原文件后

-  -eo：如日志原文件存在，出错时屏幕输出将覆盖原文件

建议打开-o和-e参数，以便查看作业为什么出问题等。如果需要管理人员协助解决，请告知这些输出，以及运行目录，怎么运行的等，以便管理人员能获取足够的信息及时处理。

指明输出目录提交：bsub -outdir output_directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认情况下，屏幕输出等存放在提交作业时的目录下，如想将其放到其它目录可以采用\ ``bsub -outdir output\_directory``\ 方式，如：

`bsub -outdir %U/%J_%I myprog`

结合以下常用变量，可以方便结合作业号等设置输出目录及日志文件名等：

-  %J：作业号

-  %JG：作业组

-  %I：作业组中的索引

-  %EJ：执行作业号

-  %EI：作业组中的执行作业索引

-  %P：作业名

-  %U：用户名

-  %G：用户组名

交互式运行作业：bsub -I
^^^^^^^^^^^^^^^^^^^^^^^

如果需要运行交互式的作业（如在运行期间需要手动输入参数或利用调试器手动调试程序等需要进行交互时），需要结合-I参数。建议只是在调试期间使用，一般作业还是尽量不要使用此选项，类似选项还有-Ip和-Is：

``bsub -I executable1``

满足依赖关系运行作业：bsub -w
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-w``\ 选项可以使得新提交的作业在满足一定条件时才运行，比如与其它作业的关联：

-  done(job_ID \|“job_name” ...)：作业结束时状态为DONE时运行

-  ended(job_ID \| “job_name”)：作业结束时状态为DONE或EXIT时运行

-  exit(job_ID \| “job_name” [,[operator]
   exit_code])：作业结束时状态为EXIT，且退出代码满足一定条件时运行

-  external(job_ID \| “job_name”,
   “status_text”)：作业状态变为某状态时运行，如变为SUSP

-  支持的条件之间的条件表达式：&&（和）、||（或）、!（否）

-  支持的条件内的条件算子：>、 >=、 <、 <=、==、!=

如：\ :literal:`bsub -w ``done(1456)''`

指定时间运行：bsub -b time
^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-b [[year:][month:]day:]hour:minute``\ 可以使得新提交的作业在特定时间运行，系统会预留资源给此作业，如果在时间到达时所需资源满足则会运行，不满足则继续等待。如：

``bsub -b 2016:01:09:09:20``

指定运行时长：bsub -W time
^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ ``-W [hour:]minute``\ 可以使得提交的作业在运行超过设定时长后终止，如：

``bsub -W 1:30``

在运行前执行特定命令：bsub -E “pre_command”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ :literal:`-E ``pre\_exec\_command [arguments ...]''`\ 可以使得作业在运行前，在所分配的节点上运行特定命令。

如利用modinput.sh此修改程序输入参数：\ :literal:`bsub -E ``./modinput.sh 10'' -n 2 exec1`

在运行后执行特定命令：bsub -Ep “post_command”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

利用\ :literal:`-Ep ``post\_exec\_command [arguments ...]''`\ 可以使得作业在运行结束时，在所分配的节点上运行特定命令。

如利用此删除core文件：\ :literal:`bsub -Ep ``bin/rm -f core.*'' -n 2 exec1`

LSF作业脚本
^^^^^^^^^^^

如果作业比较复杂，还需要设置环境变量，做其它处理等，可用以在LSF脚本中设置队列等参数方式提交，如

.. code:: bash

    #!/bin/sh
    #BSUB -q long
    #BSUB -o %J.log -e %J.err
    #BSUB -n 64
    source my.sh
    mpijob ./mympi-prog1
    cd newworkdir
    mpijob ./mympi-prog2

注意，采用此方式时：

-  不得以直接\ ``./my_script.lsf``\ 等常规脚本运行方式运行。

-  需要传递给\ ``bsub``\ 命令运行：\ ``bsub < my_script.lsf``\ 。

-  如果bsub后面更-q等LSF参数，将会覆盖掉LSF脚本中的设置。

-  一般用户，没必要写此类脚本，直接通过命令行传递LSF参数即可。

-  对于当前设置满足不了作业需求，且用户比较了解LSF中的各规定，对shell脚本编写比较在行，那么用户完全可自己编写脚本提交作业，比如提交特殊需求的MPI与OpenMP结合的作业。

LSF作业脚本主要有以下常见变量比较常用，在作业运行后，这些变量存储对应的作业信息，具体的请参看LSF官方手册：

-  ``LS_JOBPID``\ ：作业进程号

-  ``LSB_HOSTS``\ ：存储系统分配的节点名

-  ``LSB_JOBFILENAME``\ ：作业脚本文件名

-  ``LSB_JOBID``\ ：作业号

-  ``LSB_QUEUE``\ ：作业队列

-  ``LSB_JOBPGIDS``\ ：作业进程组号组

-  ``LSB_JOBPIDS``\ ：作业进程号组

终止作业：bkill
~~~~~~~~~~~~~~~

利用\ ``bkill``\ 命令可以终止某个运行中或者排队中的作业，比如：

``bkill 79722``

运行成功后，将显示类似下面的输出：

::

    Job <79722> is being terminated

挂起作业：bstop
~~~~~~~~~~~~~~~

利用\ ``bstop``\ 命令可临时挂起某个作业以让别的作业先运行，例如：

``bstop 79727``

运行成功后，将显示类似下面的输出：

::

    Job <79727> is being stopped.

此命令可以将排在队列前面的作业临时挂起，以让后面的作业先运行。虽然也可以作用于运行中的作业，但并不会因为此作业被挂起而允许其余作业占用此作业所占用的CPU运行，实际资源不会释放，因此建议不要随便对运行中的作业进行挂起操作，如果运行中的作业不再想继续运行，请用\ ``bkill``\ 终止。

继续运行被挂起的作业：bresume
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bresume``\ 命令可继续运行某个挂起某个作业，例如：

``bresume 79727``

运行成功后，将显示类似下面的输出：

::

    Job <79727> is being resumed.

设置作业最先运行：btop
~~~~~~~~~~~~~~~~~~~~~~

利用\ ``btop``\ 命令可最先运行排队中的某个作业，例如：

``btop 79727``

运行成功后，将显示类似下面的输出：

::

    Job <79727> has been moved to position 1 from top.

设置作业最后运行：bbot
~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bbot``\ 命令可设定最后运行排队中的某个作业，例如：

``bbot 79727``

运行成功后，将显示类似下面的输出：

::

    Job <79727> has been moved to position 1 from bottom.

修改排队中的作业选项：bmod
~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bmod``\ 命令可修改排队中的某个作业的选项，比如想将排队中的运行作业号为79727的作业的执行命令修改为executable2并且换到long队列，可以：

``bmod -Z executable2 -q long 79727``

::

    Parameters of job <79727> are being changed.

查看作业的排队和运行情况：bjobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bjobs``\ 可以查看作业的运行情况，比如有哪些作业在运行，哪些在排队，某个作业运行在哪个节点上，以及为什么没有运行等，例如：

``bjobs``

::

    JOBID USER  STAT  QUEUE    FROM_HOST   EXEC_HOST JOB_NAME   SUBMIT_TIME
    79726 hmli  RUN   normal   tc4600      2*node31  *executab1 Mar 12 19:20
                                           1*node18
                                           1*node4
    79727 hmli  PEND  long     tc4600                *executab2 Mar 12 19:20

上面显示作业79726在运行，分别在node31、node18和node4上运行2、1、1个进程；而作业79727处于排队中尚未运行，查看未运行的原因可以利用：

``bjobs -l 79727``

::

    Job Id <79727>, tc4600 <hmli>, Project <default>, Status <PEND>,
    Queue <long> , Command <executab2>
    Sun Mar 12 14:15:07: Submitted from host <tc4600>,
    CWD <$HOME>, Requested Resources <type==any && swp>35>;
    PENDING REASONS:
    SCHEDULING PARAMETERS:
                r15s r1m r15m  ut pg   io  ls   it   tmp   swp   mem
    loadSched   -    0.7 1.0   - 4.0   -   -    -    -     -     -
    loadStop    -    1.5 2.5   - 8.0   -   -    -    -     -     -


以下为另外几个常用参数：

-  -u
   username：查看某用户的作业，如username为all，则查看所有用户的作业。

-  -q queuename：查看某队列上的作业。

-  -m hostname：查看某节点上的作业。

查看作业负载：checkjob
~~~~~~~~~~~~~~~~~~~~~~

利用\ ``/opt/bin/monitor/checkjob 作业号``\ 可以查看作业各节点负载等信息，例如：

``/opt/bin/monitor/checkjob 799374``

::

    user: hmli, jobid: 799374
    Note: cpu_load near cpu_num and swap_used smaller or zero is better.
    node_name  cpu_num  cpu_load  memory(%)  swap_used(GB)
    node491     144      109.03    23.00          0
    node493     144      115.52    23.00          0

查看运行中作业的屏幕正常输出：bpeek
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用\ ``bpeek``\ 命令可查看运行中作业的屏幕正常输出，例如：

``bpeek 79727``

::

    << output from stdout >>
    Energy: 3.0keV
    Angles: 13.0, 0.0



如果在运行中用-o和-e分别指定了正常和错误屏幕输出，也可以通过直接查看指定的文件的内容来查看屏幕输出。

如果想连续查看某个作业的输出，请添加-f参数。

.. [1]
   有些程序可以在输入文件中设置，那么可以不填加OMP_NUM_THREADS
