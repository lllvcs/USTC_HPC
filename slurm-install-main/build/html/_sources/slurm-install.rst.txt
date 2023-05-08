.. .. sectnum::

#########
Slurm简介
#########

.. note:: 该文档基于 **Slurm 21.08** 及 **CentOS 7.9 x86_64** ，除非特殊说明，所有命令均采用 **root** 用户在命令行终端下执行。

用途
====

Slurm(Simple Linux Utility for Resource Management， http://slurm.schedmd.com/ )是开源的、具有容错性和高度可扩展的Linux集群超级计算系统资源管理和作业调度系统。超级计算系统可利用Slurm对资源和作业进行管理，以避免相互干扰，提高运行效率。所有需运行的作业，无论是用于程序调试还是业务计算，都可以通过交互式并行 ``srun`` 、批处理式 ``sbatch`` 或分配式 ``salloc`` 等命令提交，提交后可以利用相关命令查询作业状态等。

架构
====

Slurm采用slurmctld服务（守护进程）作为中心管理器用于监测资源和作业，为了提高可用性，还可以配置另一个备份冗余管理器。各计算节点需启动slurmd守护进程，以便被用于作为远程shell使用：等待作业、执行作业、返回状态、再等待更多作业。slurmdbd(Slurm DataBase Daemon)数据库守护进程（非必需，建议采用，也可以记录到纯文本中等），可以将多个slurm管理的集群的记账信息记录在同一个数据库中。还可以启用slurmrestd(Slurm REST API Daemon)服务（非必需），该服务可以通过REST API与Slurm进行交互，所有功能都对应的API。用户工具包含 ``srun`` 运行作业、 ``scancel`` 终止排队中或运行中的作业、 ``sinfo`` 查看系统状态、 ``squeue`` 查看作业状态、 ``sacct`` 查看运行中或结束了的作业及作业步信息等命令。 ``sview`` 命令可以图形化显示系统和作业状态（可含有网络拓扑）。 ``scontrol`` 作为管理工具，可以监控、修改集群的配置和状态信息等。用于管理数据库的命令是 ``sacctmgr`` ，可认证集群、有效用户、有效记账账户等。

.. image:: img/arch.gif
   :width: 400px
   :align: center

术语
====

+ 节点
    * Hea Node：头节点、管理节点、控制节点，运行slurmctld管理服务的节点。
    * Compute Node：计算节点，运行作业计算任务的节点，需运行slurmd服务。
    * Login Node：用户登录节点，用于用户登录的节点。
    * SlurmDBD Node：SlurmDBD节点、SlurmDBD数据库节点，存储调度策略、记账和作业等信息的节点，需运行slurmdbd服务。
    * 客户节点：含计算节点和用户登录节点。
+ 用户
    * account：账户，一个账户可以含有多个用户。
    * user：用户，多个用户可以共享一个账户。
    * bank account：银行账户，对应机时费等。
+ 资源
    * GRES：Generic Resource，通用资源。
    * TRES：Trackable RESources，可追踪资源。
    * QOS：Quality of Service，服务质量，作业优先级。
    * association：关联。可利用其实现，如用户的关联不在数据库中，这将阻止用户运行作业。该选项可以阻止用户访问无效账户。
    * Partition：队列、分区。用于对计算节点、作业并行规模、作业时长、用户等进行分组管理，以合理分配资源。

插件
====

Slurm含有一些通用目的插件可以使用，采用这些插件可以方便地支持多种基础结构，允许利用构建块方式吸纳多种Slurm配置，主要包括如下插件：

    + 记账存储(Accounting Storage)：主要用于存储作业历史数据。当采用SlurmDBD时，可以支持有限的基于系统的历史系统状态。
    + 账户收集能源(Account Gather Energy)：收集系统中每个作业或节点的能源（电力）消耗，该插件与记账存储Accounting Storage和作业记账收集Job Account Gather插件一起使用。
    + 通信认证(Authentication of communications)：提供在Slurm不同组件之间进行认证机制。
    + 容器(Containers)：HPC作业负载容器支持及实现。
    + 信用(Credential，数字签名生成，Digital Signature Generation)：用于生成电子签名的机制，可用于验证作业步在某节点上具有执行权限。与用于身份验证的插件不同，因为作业步请求从用户的 ``srun`` 命令发送，而不是直接从slurmctld守护进程发送，该守护进程将生成作业步凭据及其数字签名。
    + 通用资源(Generic Resources)：提供用于控制通用资源（如GPU）的接口。
    + 作业提交(Job Submit)：该插件提供特殊控制，以允许站点覆盖作业在提交和更新时提出的需求。
    + 作业记账收集(Job Accounting Gather)：收集作业步资源使用数据。
    + 作业完成记录(Job Completion Logging)：记录作业完成数据，一般是记账存储插件的子数据集。
    + 启动器(Launchers)：控制srun启动任务时的机制。
    + MPI：针对多种MPI实现提供不同钩子，可用于设置MPI环境变量等。
    + 抢占(Preempt)：决定哪些作业可以抢占其它作业以及所采用的抢占机制。
    + 优先级(Priority)：在作业提交时赋予作业优先级，且在运行中生效（如，它们生命期）。
    + 进程追踪(Process tracking，为了信号控制)：提供用于确认各作业关联的进程，可用于作业记账及信号控制。
    + 调度器(Scheduler)：用于决定Slurm何时及如何调度作业的插件。
    + 节点选择(Node selection)：用于决定作业分配的资源插件。
    + 站点因子(Site Factor，站点优先级)：将作业多因子组件中的特殊的site_factor优先级在作业提交时赋予作业，且在运行中生效（如，它们生命期）。
    + 交换及互联(Switch or interconnect)：用于网络交换和互联接口的插件。对于多数网络系统（以太网或InifiniBand）并不需要。
    + 作业亲和性(Task Affinity)：提供一种用于将作业和其独立的任务绑定到特定处理器的机制。
    + 网络拓扑(Network Topology)：基于网络拓扑提供资源选择优化，用于作业分配和提前预留资源。

配置模式
=========

Slurm客户节点配置，有两种模式：

    + 传统模式：客户节点采用 **/etc/slurm/** 目录下的 **slurm.conf** 等配置文件进行配置。
    + 无配置(configless)模式：客户节点无需配置 **/etc/slurm** 目录下相应的配置文件。

无配置模式是Slurm的一项新特性（从20.02版起支持），可以允许计算节点和用户登录节点从slurmctld守护进程获取配置而无需采用 **/etc/slurm** 等目录下的本地配置文件。集群在Slurm控制节点上统一控制配置文件，计算节点、登录节点和其它集群节点只需通过 **/lib/systemd/system/slurmd.service** 文件配置slurmd服务启动参数，利用启动后的slurmd服务获取所需配置信息即可，而无需复制管理节点上的这些文件成为本地文件（降低文件配置不一样的风险）。支持的配置文件有：

    - slurm.conf
    - acct_gather.conf
    - cgroup.conf
    - cgroup_allowed_devices_file.conf
    - cli_filter.lua
    - ext_sensors.conf
    - gres.conf
    - helpers.conf
    - job_container.conf
    - knl_cray.conf
    - knl_generic.conf
    - oci.conf
    - plugstack.conf
    - topology.conf

slurmd服务启动时将从指定的slurmctld节点获取配置文件，slurmctld节点可以采用 ``--conf-server`` 参数准确指定或利用DNS SRV记录指定，采用 ``--conf-server`` 参数指定的优先级高于采用DNS SRV记录指定：

    + 采用 ``--conf-server`` 参数指定（默认端口6817可省略）：
        - 仅一个管理节点slurmctl-primary： ``slurmd --conf-server slurmctl-primary:6817``
        - 一个管理节点slurmctl-primary和一个备份节点slurmctl-secondary： ``slurmd --conf-server slurmctl-primary:6817,slurmctl-secondary`` 
    + 采用DNS SRV记录：
        - _slurmctld._tcp 3600 IN SRV 10 0 6817 slurmctl-backup
        - _slurmctld._tcp 3600 IN SRV  0 0 6817 slurmctl-primary

参见： `无配置(configless)模式 <https://slurm.schedmd.com/configless_slurm.html>`_ 

########
规划准备
########

* 集群名：MyCluster

* 管理节点admin：
    - 内网IP：191.168.1.254
    - **/opt/** 目录：通过NFS网络共享给其它节点使用
    - 配置文件： **/etc/slurm/** 目录下的 **cgroup.conf** 、 **slurm.conf** 、 **slurmdbd.conf** 等文件
    - 需要启动（按顺序）的守护进程服务：

         #. 通信认证：munge
         #. 系统数据库：mariadb（也可采用文本保存，更简单，本文不涉及）
         #. Slurm数据库：slurmdbd
         #. 主控管理器：slurmctld

* 数据库节点（运行slurmdbd服务）admin：
    - 可与管理节点共用，本文档与管理节点共用

* 用户登录节点login：
    - 内网IP：191.168.1.250
    - **/opt/** 目录：通过NFS服务共享管理节点上的 **/opt/** 目录

* 计算节点node[1-10]：
    - 内网IP：191.168.1.[1-10]
    - **/opt/** 目录：通过NFS服务共享管理节点上的 **/opt/** 目录
    - 需要启动（按顺序）的服务：
        #. 通信认证：munge
        #. Slurm数据库：slurmdbd

* 各节点node[1-10],login：
    - admin节点root用户可以通过密钥无需输入密码ssh进入各节点
    - 安装好munge包
    - 配置有NIS或LDAP等用户信息服务同步admin节点用户信息（管理节点建立slurm用户后，各节点执行 ``id slurm`` 可确认其slurm用户是否存在）

* 并行操作：

    各节点执行同样命令可以利用 `pdsh <https://computing.llnl.gov/linux/pdsh.html>`_ 命令或for循环处理：

    + 如需安装PDSH并行shell包，可利用源 http://mirrors.ustc.edu.cn/epel/ 进行安装配置。
    + 在node[1-10]节点执行 `id slurm` 可用下述命令之一：

        - pdsh： ``pdsh -w node[1-10] id slurm``
        - for循环： ``for i in `seq 1 10`; do ssh node$i id slurm; done``

    + 复制 `/etc/hosts` 文件到node[1-3,5,7-10]节点 `/etc` 目录下可执行下述命令之一：

        - pdsh： ``pdcp -w node[1-3,5,7-10] /etc/hosts /etc``
        - for循环： ``for i in `seq 1 10`; do scp -a /etc/hosts node$i:/etc/; done``

* 管理服务的常用命令（以slurmd为例）：

    + 设置开机自启动服务： ``systemctl enable slurmd``
    + 启动服务： ``systemctl start slurmd``
    + 重新启动服务： ``systemctl restart slurmd``
    + 停止服务： ``systemctl stop slurmd``
    + 查看服务状态及出错信息： ``systemctl status slurmd``
    + 查看服务日志： ``journalctl -xe``

##############
编译安装slurm
##############

可以采用第三方已编译好的RPM或DEB包进行安装，也可以采用源码编译方式。

可以在安装有所需的依赖包的任何节点处理，本例在管理节点进行。

安装编译slurm所需的依赖包
=========================

安装编译Slurm时所需软件包，执行：
::

    yum -y install mariadb mariadb-devel mariadb-server munge munge-libs munge-devel hwloc-libs hwloc-devel hdf5-devel pam-devel perl-ExtUtils-MakeMaker python3 readline-devel kernel-headers dbus-devel rpm-build


下载Slurm源码包
===============

访问 https://www.schedmd.com/downloads.php 复制所需版本下载源码包链接后，执行：
::

    wget https://download.schedmd.com/slurm/slurm-21.08.2.tar.bz2

编译成RPM包
===========

在 **slurm-21.08.2.tar.bz2** 文件所在目录执行（如有必要改变生成包的内容，可以提前设置 **~/.rpmmacros** 文件）：
::

    rpmbuild -ta slurm-21.08.2.tar.bz2

.. note::
    + 如提示缺少库、包等，请安装对应的包后再执行上面命令。
    + 如所依赖的包安装的不全，上述命令即使没有报错，能生成RPM包，也有可能所需的功能没有。

成功后将在 **/root/rpmbuild/RPMS/x86_64/** 目录下生成类似如下RPM文件：

::

    slurm-21.08.2-1.el7.x86_64.rpm
    slurm-libpmi-21.08.2-1.el7.x86_64.rpm
    slurm-slurmctld-21.08.2-1.el7.x86_64.rpm
    slurm-contribs-21.08.2-1.el7.x86_64.rpm
    slurm-openlava-21.08.2-1.el7.x86_64.rpm
    slurm-slurmd-21.08.2-1.el7.x86_64.rpm
    slurm-devel-21.08.2-1.el7.x86_64.rpm
    slurm-pam_slurm-21.08.2-1.el7.x86_64.rpm
    slurm-slurmdbd-21.08.2-1.el7.x86_64.rpm
    slurm-example-configs-21.08.2-1.el7.x86_64.rpm
    slurm-perlapi-21.08.2-1.el7.x86_64.rpm
    slurm-torque-21.08.2-1.el7.x86_64.rpm

不同节点类型所需安装包
======================

* 管理节点(Head Node，运行slurmctld服务)、计算节点(Compute Node)和用户登录节点(Login Node)：
   - slurm
   - slurm-perlapi
   - slurm-slurmctld（仅管理节点需要）
   - slurm-slurmd（仅计算节点需要，用户登录节点如采用无配置模式，则也需要）
* SlurmDBD节点：
   - slurm
   - slurm-slurmdbd

################################
管理节点操作（也为其它节点配置）
################################

设置Slurm的YUM软件仓库
======================
    
    可以将前面生成的RPM通过NFS服务共享或直接复制到各节点，然后执行 ``yum localinstall 包文件名`` 命令安装，或采用下面建立YUM软件仓库后直接用包名方式安装。

    * 建立YUM仓库目录：

    ::

         mkdir -p /opt/src/slurm

    * 复制前面生成的RPM文件到 **/opt/src/slurm/** 目录：

    ::

        cp /root/rpmbuild/RPMS/x86_64/*.rpm /opt/src/slurm/

    * 建立YUM仓库RPM文件索引：

        - 在 **/opt/src/slurm/** 目录下运行 ``createrepo .`` 命令生成仓库索引。

        - 生成repo配置文件，执行命令：

        ::

            cat >/etc/yum.repos.d/slurm.repo<<EOF
            [slurm]
            name=slurm
            baseurl=file:///opt/src/slurm
            gpgcheck=0
            enable=1
            EOF

    * 为其它节点设置YUM仓库

    将上述 **/etc/yum.repos.d/slurm.repo** 文件复制到所有需安装slurm的节点 **/etc/yum.repos.d/** 目录下，执行命令：

    ::

        pdcp -w node[1-10],login /etc/yum.repos.d/slurm.repo /etc/yum.repos.d/

安装所需slurm包
===============

以下命令仍旧在管理节点执行。

    + 管理节点，执行：

    ::

        yum -y install slurm slurm-perlapi slurm-slurmdbd slurm-slurmctld
        

    + 为计算节点和用户登录节点，执行：

    ::

        pdsh -w node[1-10],login yum -y install slurm slurm-perlapi slurm-slurmd 

生成slurm用户
==============

Slurm需要有一个用户用于进行通信、认证等操作，这里用户名为slurm（也可为其它名字）：

    * 生成用户，执行：

    ::

        useradd slurm


    * 其它节点配置有NIS或LDAP等用户信息管理系统，自动从管理节点获取slurm用户信息（无需再单独生成slurm用户），执行以下命令确认是否存在（如未存在，请检查相关服务）：

    ::

        pdsh -w node[1-10],login id slurm

设置通讯所需的munge服务
=======================

munge为slurm默认的验证机制，各节点 **/etc/munge/munge.key** 的内容需要一致，且所有者都为munge用户。

    * 为各节点安装munge包：

    ::
        
        pdsh -w node[1-10],login yum -y install munge

    * 生成 **/etc/munge/munge.key** ，执行命令：

    ::

        create-munge-key

    * 将生成的 **/etc/munge/munge.key** 复制成各节点 **/etc/munge/munge.key** ：

    ::

        pdcp -w node[1-10],login /etc/munge/munge.key /etc/munge/

    * 设置各节点 ``/etc/munge/munge.key`` 的所有者为munge用户：

    ::

        pdsh -w node[1-10],login chown munge.munge /etc/munge/munge.key

   * 设置各节点开机自动启动munge服务（enable选项），且现在就启动服务（--now参数）：

    ::

        pdsh -w node[1-10],login systemctl enable --now munge

    以上也可以分成两个命令执行：
        
        + 设置开机自启动

        ::
            
            pdsh -w node[1-10],login systemctl enable munge

        + 重新启动

        ::
            
            pdsh -w node[1-10],login systemctl restart munge

设置主配置文件
==============

Slurm配置文件主要在 **/etc/slurm/** 目录下。

+ 主配置文件：**/etc/slurm/slurm.conf** ：

内容模板可访问 https://slurm.schedmd.com/configurator.html 填写相应信息生成，然后修改：

::

   # Cluster Name：集群名
    ClusterName=MyCluster # 集群名，任意英文和数字名字

   # Control Machines：Slurmctld控制进程节点
    SlurmctldHost=admin # 启动slurmctld进程的节点名，如这里的admin
    BackupController=   # 冗余备份节点，可空着
    SlurmctldParameters=enable_configless # 采用无配置模式

    # Slurm User：Slurm用户
    SlurmUser=slurm # slurmctld启动时采用的用户名

    # Slurm Port Numbers：Slurm服务通信端口
    SlurmctldPort=6817 # Slurmctld服务端口，设为6817，如不设置，默认为6817号端口
    SlurmdPort=6818    # Slurmd服务端口，设为6818，如不设置，默认为6818号端口

    # State Preservation：状态保持
    StateSaveLocation=/var/spool/slurmctld # 存储slurmctld服务状态的目录，如有备份控制节点，则需要所有SlurmctldHost节点都能共享读写该目录
    SlurmdSpoolDir=/var/spool/slurmd # Slurmd服务所需要的目录，为各节点各自私有目录，不得多个slurmd节点共享
    
    ReturnToService=1 #设定当DOWN（失去响应）状态节点如何恢复服务，默认为0。
        # 0: 节点状态保持DOWN状态，只有当管理员明确使其恢复服务时才恢复
        # 1: 仅当由于无响应而将DOWN节点设置为DOWN状态时，才可以当有效配置注册后使DOWN节点恢复服务。如节点由于任何其它原因（内存不足、意外重启等）被设置为DOWN，其状态将不会自动更改。当节点的内存、GRES、CPU计数等等于或大于slurm.conf中配置的值时，该节点才注册为有效配置。
        # 2: 使用有效配置注册后，DOWN节点将可供使用。该节点可能因任何原因被设置为DOWN状态。当节点的内存、GRES、CPU计数等等于或大于slurm.conf 中配置的值，该节点才注册为有效配置。￼

    # Default MPI Type：默认MPI类型
    MPIDefault=None
        # MPI-PMI2: 对支持PMI2的MPI实现
        # MPI-PMIx: Exascale PMI实现
        # None: 对于大多数其它MPI，建议设置

    # Process Tracking：进程追踪，定义用于确定特定的作业所对应的进程的算法，它使用信号、杀死和记账与作业步相关联的进程
    ProctrackType=proctrack/cgroup
        # Cgroup: 采用Linux cgroup来生成作业容器并追踪进程，需要设定/etc/slurm/cgroup.conf文件
        # Cray XC: 采用Cray XC专有进程追踪
        # LinuxProc: 采用父进程IP记录，进程可以脱离Slurm控制
        # Pgid: 采用Unix进程组ID(Process Group ID)，进程如改变了其进程组ID则可以脱离Slurm控制

    # Scheduling：调度
    # DefMemPerCPU=0 # 默认每颗CPU可用内存，以MB为单位，0为不限制。如果将单个处理器分配给作业（SelectType=select/cons_res 或 SelectType=select/cons_tres），通常会使用DefMemPerCPU
    # MaxMemPerCPU=0 # 最大每颗CPU可用内存，以MB为单位，0为不限制。如果将单个处理器分配给作业（SelectType=select/cons_res 或 SelectType=select/cons_tres），通常会使用MaxMemPerCPU
    # SchedulerTimeSlice=30 # 当GANG调度启用时的时间片长度，以秒为单位
    SchedulerType=sched/backfill # 要使用的调度程序的类型。注意，slurmctld守护程序必须重新启动才能使调度程序类型的更改生效（重新配置正在运行的守护程序对此参数无效）。如果需要，可以使用scontrol命令手动更改作业优先级。可接受的类型为：
        # sched/backfill # 用于回填调度模块以增加默认FIFO调度。如这样做不会延迟任何较高优先级作业的预期启动时间，则回填调度将启动较低优先级作业。回填调度的有效性取决于用户指定的作业时间限制，否则所有作业将具有相同的时间限制，并且回填是不可能的。注意上面SchedulerParameters选项的文档。这是默认配置
        # sched/builtin # 按优先级顺序启动作业的FIFO调度程序。如队列中的任何作业无法调度，则不会调度该队列中优先级较低的作业。对于作业的一个例外是由于队列限制（如时间限制）或关闭/耗尽节点而无法运行。在这种情况下，可以启动较低优先级的作业，而不会影响较高优先级的作业。
        # sched/hold # 如果 /etc/slurm.hold 文件存在，则暂停所有新提交的作业，否则使用内置的FIFO调度程序。

    # Resource Selection：资源选择，定义作业资源（节点）选择算法
    SelectType=select/cons_tres
        # select/cons_tres: 单个的CPU核、内存、GPU及其它可追踪资源作为可消费资源（消费及分配），建议设置
        # select/cons_res: 单个的CPU核和内存作为可消费资源
        # select/cray_aries: 对于Cray系统
        # select/linear: 基于主机的作为可消费资源，不管理单个CPU等的分配

    # SelectTypeParameters：资源选择类型参数，当SelectType=select/linear时仅支持CR_ONE_TASK_PER_CORE和CR_Memory；当SelectType=select/cons_res、SelectType=select/cray_aries和SelectType=select/cons_tres时，默认采用CR_Core_Memory
    SelectTypeParameters=CR_Core_Memory
        # CR_CPU: CPU核数作为可消费资源
        # CR_Socket: 整颗CPU作为可消费资源
        # CR_Core: CPU核作为可消费资源，默认
        # CR_Memory: 内存作为可消费资源，CR_Memory假定MaxShare大于等于1
        # CR_CPU_Memory: CPU和内存作为可消费资源
        # CR_Socket_Memory: 整颗CPU和内存作为可消费资源
        # CR_Core_Memory: CPU和和内存作为可消费资源

    # Task Launch：任务启动
    TaskPlugin=task/cgroup,task/affinity #设定任务启动插件。可被用于提供节点内的资源管理（如绑定任务到特定处理器），TaskPlugin值可为:
        # task/affinity: CPU亲和支持（man srun查看其中--cpu-bind、--mem-bind和-E选项）
        # task/cgroup: 强制采用Linux控制组cgroup分配资源（man group.conf查看帮助）
        # task/none: #无任务启动动作

    # Prolog and Epilog：前处理及后处理
    # Prolog/Epilog: 完整的绝对路径，在用户作业开始前(Prolog)或结束后(Epilog)在其每个运行节点上都采用root用户执行，可用于初始化某些参数、清理作业运行后的可删除文件等
    # Prolog=/opt/bin/prolog.sh # 作业开始运行前需要执行的文件，采用root用户执行
    # Epilog=/opt/bin/epilog.sh # 作业结束运行后需要执行的文件，采用root用户执行

    # SrunProlog/Epilog # 完整的绝对路径，在用户作业步开始前(SrunProlog)或结束后(Epilog)在其每个运行节点上都被srun执行，这些参数可以被srun的--prolog和--epilog选项覆盖
    # SrunProlog=/opt/bin/srunprolog.sh # 在srun作业开始运行前需要执行的文件，采用运行srun命令的用户执行
    # SrunEpilog=/opt/bin/srunepilog.sh # 在srun作业结束运行后需要执行的文件，采用运行srun命令的用户执行

    # TaskProlog/Epilog: 绝对路径，在用户任务开始前(Prolog)和结束后(Epilog)在其每个运行节点上都采用运行作业的用户身份执行
    # TaskProlog=/opt/bin/taskprolog.sh # 作业开始运行前需要执行的文件，采用运行作业的用户执行
    # TaskEpilog=/opt/bin/taskepilog.sh # 作业结束后需要执行的文件，采用运行作业的用户执行行

    # 顺序：
       # 1. pre_launch_priv()：TaskPlugin内部函数
       # 2. pre_launch()：TaskPlugin内部函数
       # 3. TaskProlog：slurm.conf中定义的系统范围每个任务
       # 4. User prolog：作业步指定的，采用srun命令的--task-prolog参数或SLURM_TASK_PROLOG环境变量指定
       # 5. Task：作业步任务中执行
       # 6. User epilog：作业步指定的，采用srun命令的--task-epilog参数或SLURM_TASK_EPILOG环境变量指定
       # 7. TaskEpilog：slurm.conf中定义的系统范围每个任务
       # 8. post_term()：TaskPlugin内部函数

    # Event Logging：事件记录
    # Slurmctld和slurmd守护进程可以配置为采用不同级别的详细度记录，从0（不记录）到7（极度详细）
    SlurmctldDebug=info # 默认为info
    SlurmctldLogFile=/var/log/slurm/slurmctld.log # 如是空白，则记录到syslog
    SlurmdDebug=info # 默认为info
    SlurmdLogFile=/var/log/slurm/slurmd.log # 如为空白，则记录到syslog，如名字中的有字符串"%h"，则"%h"将被替换为节点名

    # Job Completion Logging：作业完成记录
    JobCompType=jobcomp/mysql
    # 指定作业完成是采用的记录机制，默认为None，可为以下值之一:
       # None: 不记录作业完成信息
       # Elasticsearch: 将作业完成信息记录到Elasticsearch服务器
       # FileTxt: 将作业完成信息记录在一个纯文本文件中
       # Lua: 利用名为jobcomp.lua的文件记录作业完成信息
       # Script: 采用任意脚本对原始作业完成信息进行处理后记录
       # MySQL: 将完成状态写入MySQL或MariaDB数据库

    # JobCompLoc= # 设定记录作业完成信息的文本文件位置（若JobCompType=filetxt），或将要运行的脚本（若JobCompType=script），或Elasticsearch服务器的URL（若JobCompType=elasticsearch），或数据库名字（JobCompType为其它值时）

    # 设定数据库在哪里运行，且如何连接
    JobCompHost=localhost # 存储作业完成信息的数据库主机名
    # JobCompPort= # 存储作业完成信息的数据库服务器监听端口
    JobCompUser=slurm # 用于与存储作业完成信息数据库进行对话的用户名
    JobCompPass=SomePassWD # 用于与存储作业完成信息数据库进行对话的用户密码

    # Job Accounting Gather：作业记账收集
    JobAcctGatherType=jobacct_gather/linux # Slurm记录每个作业消耗的资源，JobAcctGatherType值可为以下之一：
       # jobacct_gather/none: 不对作业记账
       # jobacct_gather/cgroup: 收集Linux cgroup信息
       # jobacct_gather/linux: 收集Linux进程表信息，建议
    JobAcctGatherFrequency=30 # 设定轮寻间隔，以秒为单位。若为-，则禁止周期性抽样

    # Job Accounting Storage：作业记账存储
    AccountingStorageType=accounting_storage/slurmdbd # 与作业记账收集一起，Slurm可以采用不同风格存储可以以许多不同的方式存储会计信息，可为以下值之一：
        # accounting_storage/none: 不记录记账信息
        # accounting_storage/slurmdbd: 将作业记账信息写入Slurm DBD数据库
    # AccountingStorageLoc: 设定文件位置或数据库名，为完整绝对路径或为数据库的数据库名，当采用slurmdb时默认为slurm_acct_db

    # 设定记账数据库信息，及如何连接
    AccountingStorageHost=localhost # 记账数据库主机名
    # AccountingStoragePort= # 记账数据库服务监听端口
    # AccountingStorageUser=slurm # 记账数据库用户名
    # AccountingStoragePass=SomePassWD # 记账数据库用户密码。对于SlurmDBD，提供企业范围的身份验证，如采用于Munge守护进程，则这是应该用munge套接字socket名（/var/run/munge/global.socket.2）代替。默认不设置
    # AccountingStoreFlags= # 以逗号（,）分割的列表。选项是：
        # job_comment：在数据库中存储作业说明域
        # job_script：在数据库中存储脚本
        # job_env：存储批处理作业的环境变量
    # AccountingStorageTRES=gres/gpu # 设置GPU时需要
    # GresTypes=gpu # 设置GPU时需要

    # Process ID Logging：进程ID记录，定义记录守护进程的进程ID的位置
    SlurmctldPidFile=/var/run/slurmctld.pid # 存储slurmctld进程号PID的文件
    SlurmdPidFile=/var/run/slurmd.pid # 存储slurmd进程号PID的文件

    # Timers：定时器
    SlurmctldTimeout=120 # 设定备份控制器在主控制器等待多少秒后成为激活的控制器
    SlurmdTimeout=300 # Slurm控制器等待slurmd未响应请求多少秒后将该节点状态设置为DOWN
    InactiveLimit=0 # 潜伏期控制器等待srun命令响应多少秒后，将在考虑作业或作业步骤不活动并终止它之前。0表示无限长等待
    MinJobAge=300 # Slurm控制器在等待作业结束多少秒后清理其记录
    KillWait=30 # 在作业到达其时间限制前等待多少秒后在发送SIGKILLL信号之前发送TERM信号以优雅地终止
    WaitTime=0 # 在一个作业步的第一个任务结束后等待多少秒后结束所有其它任务，0表示无限长等待

    # Compute Machines：计算节点
    NodeName=node[1-10] NodeAddr=192.168.1.[1-8] CPUs=48 RealMemory=192000 Sockets=2 CoresPerSocket=24 ThreadsPerCore=1 State=UNKNOWN
    # NodeName=gnode[01-10] Gres=gpu:v100:2 CPUs=40 RealMemory=385560 Sockets=2 CoresPerSocket=20 ThreadsPerCore=1 State=UNKNOWN #GPU节点例子，主要为Gres=gpu:v100:2
        # NodeName=node[1-10] # 计算节点名，node[1-10]表示为从node1、node2连续编号到node10，其余类似
        # NodeAddr=192.168.1.[1-10] # 计算节点IP
        # CPUs=48 # 节点内CPU核数，如开着超线程，则按照2倍核数计算，其值为：Sockets*CoresPerSocket*ThreadsPerCore
        # RealMemory=192000 # 节点内作业可用内存数(MB)，一般不大于free -m的输出，当启用select/cons_res插件限制内存时使用
        # Sockets=2 # 节点内CPU颗数
        # CoresPerSocket=24 # 每颗CPU核数
        # ThreadsPerCore=1 # 每核逻辑线程数，如开了超线程，则为2
        # State=UNKNOWN # 状态，是否启用，State可以为以下之一：
            # CLOUD   # 在云上存在
            # DOWN    # 节点失效，不能分配给在作业
            # DRAIN   # 节点不能分配给作业
            # FAIL    # 节点即将失效，不能接受分配新作业
            # FAILING # 节点即将失效，但上面有作业未完成，不能接收新作业
            # FUTURE  # 节点为了将来使用，当Slurm守护进程启动时设置为不存在，可以之后采用scontrol命令简单地改变其状态，而不是需要重启slurmctld守护进程。当这些节点有效后，修改slurm.conf中它们的State。在它们被设置为有效前，采用Slurm看不到它们，也尝试与其联系。
                  # 动态未来节点(Dynamic Future Nodes)：
                     # slurmd启动时如有-F[<feature>]参数，将关联到一个与slurmd -C命令显示配置(sockets、cores、threads)相同的配置的FUTURE节点。节点的NodeAddr和NodeHostname从slurmd守护进程自动获取，并且当被设置为FUTURE状态后自动清除。动态未来节点在重启时保持non-FUTURE状态。利用scontrol可以将其设置为FUTURE状态。
                     # 若NodeName与slurmd的HostName映射未通过DNS更新，动态未来节点不知道在之间如何进行通信，其原因在于NodeAddr和NodeHostName未在slurm.conf被定义，而且扇出通信(fanout communication)需要通过将TreeWidth设置为一个较高的数字（如65533）来使其无效。若做了DNS映射，则可以使用cloud_dns SlurmctldParameter。
             # UNKNOWN # 节点状态未被定义，但将在节点上启动slurmd进程后设置为BUSY或IDLE，该为默认值。

    PartitionName=batch Nodes=node[1-10] Default=YES MaxTime=INFINITE State=UP
        # PartitionName=batch # 队列分区名
        # Nodes=node[1-10] # 节点名
        # Default=Yes # 作为默认队列，运行作业不知明队列名时采用的队列
        # MaxTime=INFINITE # 作业最大运行时间，以分钟为单位，INFINITE表示为无限制
        # State=UP # 状态，是否启用
        # Gres=gpu:v100:2 # 设置节点有两块v100 GPU卡，需要在GPU节点 /etc/slum/gres.conf 文件中有类似下面配置：
            #AutoDetect=nvml
            Name=gpu Type=v100 File=/dev/nvidia[0-1] #设置资源的名称Name是gpu，类型Type为v100，名称与类型可以任意取，但需要与其它方面配置对应，File=/dev/nvidia[0-1]指明了使用的GPU设备。
            #Name=mps Count=100

+ 数据存储方式配置文件 **slurmdbd.conf** ：

仅启动slurmdbd节点需要。

::

    # 认证信息
    AuthType=auth/munge # 认证方式，该处采用munge进行认证
    AuthInfo=/var/run/munge/munge.socket.2 # 为了与slurmctld控制节点通信的其它认证信息
    #
    # slurmDBD信息
    DbdHost=localhost # 数据库节点名
    DbdAddr=127.0.0.1 # 数据库IP地址
    # DbdBackupHost=admin2 # 数据库冗余备份节点
    # DbdPort=7031 # 数据库端口号，默认为7031
    SlurmUser=slurm # 用户数据库操作的用户
    MessageTimeout=60 # 允许以秒为单位完成往返通信的时间，默认为10秒

    DebugLevel=debug5 # 调试信息级别，quiet：无调试信息；fatal：仅严重错误信息；error：仅错误信息； info：错误与通常信息；verbose：错误和详细信息；debug：错误、详细和调试信息；debug2：错误、详细和更多调试信息；debug3：错误、详细和甚至更多调试信息；debug4：错误、详细和甚至更多调试信息；debug5：错误、详细和甚至更多调试信息。debug数字越大，信息越详细
    DefaultQOS=normal # 默认QOS
    LogFile=/var/log/slurm/slurmdbd.log # slurmdbd守护进程日志文件绝对路径
    PidFile=/var/run/slurmdbd.pid # slurmdbd守护进程存储进程号文件绝对路径
    # PrivateData=accounts,users,usage,jobs # 对于普通用户隐藏的数据。默认所有信息对所有用户开放，SlurmUser、root和AdminLevel=Admin用户可以查看所有信息。多个值可以采用逗号（,）分割：
        # accounts：阻止用户查看账户信息，除非该用户是他们的协调人
        # events：阻止用户查看事件信息，除非该用户具有操作员或更高级身份
        # jobs：阻止普户查看其他用户的作业信息，除非该用户是使用 sacct 时运行作业的帐户的协调员。
        # reservations：限制具有操作员及以上身份的用户获取资源预留信息。￼
        # usage：阻止用户查看其他用户利用率。适用于sreport命令
        # users：阻止用户查看除自己以外的任何用户的信息，使得用户只能看到他们处理的关联。协调人可以看到他们作为协调人的帐户中所有用户的关联，但只有在列出用户时才能看到自己。

    #TrackWCKey=yes # 工作负载特征键。用于设置Workload Characterization Key的显示和跟踪。必须设置为跟踪wckey的使用。这必须设置为从WCKeys生成汇总使用表。注意：如果在此处设置TrackWCKey而不是在您的各种slurm.conf文件中，则所有作业都将归因于它们的默认WCKey。
    #
    # Database信息，详细解释参见前面slurm.conf中的
    StorageType=accounting_storage/mysql # 数据存储类型
    StorageHost=localhost # 存储数据库节点名
    StorageLoc=slurm_acct_db # 存储位置
    StoragePort=3306 # 存储数据库服务端口号
    StorageUser=slurm # 存储数据库用户名
    StoragePass=SomePassWD # 存储数据库密码

设置cgroup限制
==============

启用cgroup资源限制，可以防止用户实际使用的资源超过用户为该作业通过作业调度系统申请到的资源。

如不需限制，不要在 **/etc/slurm/slurm.conf** 中设定 `ProctrackType=proctrack/cgroup` 及 `TaskPlugin=task/cgroup` 参数。如设定了 `ProctrackType=proctrack/cgroup` 和 `TaskPlugin=task/cgroup` 参数，还需要设定 **/etc/slurm/cgroup.conf** 文件内容类似：

::

    ###
    #
    # Slurm cgroup support configuration file
    #
    # See man slurm.conf and man cgroup.conf for further
    # information on cgroup configuration parameters
    #--
    CgroupAutomount=yes # Cgroup自动挂载。Slurm cgroup插件需要挂载有效且功能正常的cgroup子系统于 /sys/fs/cgroup/<subsystem_name>。当启动时，插件检查该子系统是否可用。如不可用，该插件将启动失败，直到CgroupAutomount设置为yes。在此情形侠，插件首先尝试挂载所需的子系统。
    CgroupMountpoint=/sys/fs/cgroup # 设置cgroup挂载点，该目录应该是可写的，可以含有每个子系统挂载的cgroups。默认在/sys/fs/cgroup。
    # CgroupPlugin= # 设置与cgroup子系统交互采用的插件。其值可以为cgroup/v1（支持传统的cgroup v1接口）或autodetect（根据系统提供的cgroup版本自动选择）。默认为autodetect。

    ConstrainCores=yes # 如设为yes，则容器允许将CPU核作为可分配资源子集，该项功能使用cpuset子系统。由于HWLOC 1.11.5版本中修复的错误，除了task/cgroup外，可能还需要task/affinity插件才能正常运行。默认为no。
    ConstrainDevices=yes # 如设为yes，则容器允许将基于GRES的设备作为可分配资源，这使用设备子系统。默认为no。
    ConstrainRAMSpace=yes # 如设为yes，则通过将内存软限制设置为分配的内存，并将硬限制设置为分配的内存AllowedRAMSpace来限制作业的内存使用。默认值为no，在这种情况下，如ConstrainSwapSpace设为“yes”，则作业的内存限制将设置为其交换空间(SWAP)限制。
    #注意：在使用ConstrainRAMSpace时，如果一个作业步中所有进程使用的总内存大于限制，那么内核将触发内存不足(Out Of Memory，OOM)事件，将杀死作业步中的一个或多个进程。作业步状态将被标记为OOM，但作业步本身将继续运行，作业步中的其它进程也可能继续运行。这与OverMemoryKill的行为不同，后者将终止/取消整个作业步。不同之处还在于，JobAcctGather轮询系统在每个进程的基础上检查内存使用情况。

    # MaxRAMPercent=98 #运行作业使用的最大内存百分比。将应用于Slurm未显式分配内存的作业的内存约束（如，Slurm的选择插件未配置为管理内存分配）。该百分比可能是一个任意的浮点数。默认值为100。
    # AllowedRAMSpace=96 #运行作业/作业步使用的最大cgroup内存百分比。所提供的百分比可以用浮点数表示，例如101.5。在分配的内存大小下设置cgroup软内存限制，然后在分配的内存(AllowedRAMSpace/100)处设置作业/作业步硬内存限制。如果作业/作业步超出硬限制，则可能触发内存不足(OOM)事件（包括内存关闭），这些事件将记录到内核日志环缓冲区（Linux中的dmesg）。设置AllowedRAMSpace超过100可能会导致系统内存不足(OOM)事件，因为它允许作业/作业步分配比配置给节点更多的内存。建议减少已配置的节点可用内存，以避免系统内存不足(OOM)事件。将AllowedRAMSpace设置为低于100将导致作业接收的内存少于分配的内存，软内存限制将设置为与硬内存限制相同的值。默认值为100。

+ 设置Slurm文件、目录、权限等：

    - /etc/slurm/slurmdbd.conf文件所有者须为slurm用户：

    ::

        chown slurm.slurm /etc/slurm/slurmdbd.conf

    - /etc/slurm/slurmdbd.conf文件权限须为600：

    ::

        chmod 600 /etc/slurm/slurmdbd.conf

    - /etc/slurm/slurm.conf文件所有者须为root用户：

    ::

        chown root /etc/slurm/slurm.conf

    - 建立slurmctld服务存储其状态等的目录，由slurm.conf中StateSaveLocation参数定义：

    ::

        mkdir /var/spool/slurmctld

    - 设置/var/spool/slurmctld目录所有者为slurm用户：

    ::

        chown slurm.slurm /var/spool/slurmctld

+ 设置slurmdbd服务：

    - 设置slurmdbd服务为开机自启动：

    ::

        systemctl enable slurmdbd

    - 重启slurmdbd服务：

    ::

        systemctl restart slurmdbd

+ 设置Slurm中定义的集群名：

    设置Slurm中定义的集群名为MyCluster：

    ::

         sacctmgr add cluster MyCluster

设置mariadb(MySQL)数据库
========================

Slurm支持将账户信息等记录到简单的纯文本文件中、或直接存入数据库（MySQL或MariaDB等）、或对于多集群的某个安全管理账户信息的服务。该文档采用将记账信息等存储到数据库方式。

    - 设置数据库权限

    执行 ``mysql`` 命令进入MariaDB控制台，然后在MariaDB控制台中执行下面命令（注意#开始的是注释，不要执行）：

    ::

        # 生成slurm用户，以便该用户操作slurm_acct_db数据库，其密码是SomePassWD
        create user 'slurm'@'localhost' identified by 'SomePassWD';

        # 生成账户数据库slurm_acct_db
        create database slurm_acct_db;
        # 赋予slurm从本机localhost采用密码SomePassWD登录具备操作slurm_acct_db数据下所有表的全部权限
        grant all on slurm_acct_db.* TO 'slurm'@'localhost' identified by 'SomePassWD' with grant option;
        # 赋予slurm从system0采用密码SomePassWD登录具备操作slurm_acct_db数据下所有表的全部权限
        grant all on slurm_acct_db.* TO 'slurm'@'system0' identified by 'SomePassWD' with grant option;

        # 生成作业信息数据库slurm_jobcomp_db
        create database slurm_jobcomp_db;
        # 赋予slurm从本机localhost采用密码SomePassWD登录具备操作slurm_jobcomp_db数据下所有表的全部权限
        grant all on slurm_jobcomp_db.* TO 'slurm'@'localhost' identified by 'SomePassWD' with grant option;
        # 赋予slurm从system0采用密码SomePassWD登录具备操作slurm_jobcomp_db数据下所有表的全部权限
        grant all on slurm_jobcomp_db.* TO 'slurm'@'system0' identified by 'SomePassWD' with grant option;

重启slurmctld服务
=================

+ 重启slurmctld服务

::

    systemctl restart slurmctld

+ 设置开机自动启用mariadb服务，且现在就启动服务：

::

    systemctl enable --now mariadb 

计算节点和用户登录节点设置slurmd服务
====================================

建立 **slurmd.service** 文件，其内容为：

::

    [Unit]
    Description=Slurm node daemon
    After=munge.service network-online.target remote-fs.target
    Wants=network-online.target
    # ConditionPathExists=/etc/slurm/slurm.conf

    [Service]
    Type=simple
    EnvironmentFile=-/etc/sysconfig/slurmd

    # 增加--conf-server admin1:6817设定slurmctld服务节点
    ExecStart=/usr/sbin/slurmd --conf-server admin1:6817 -D -s $SLURMD_OPTIONS
    ExecReload=/bin/kill -HUP $MAINPID
    KillMode=process
    LimitNOFILE=131072
    LimitMEMLOCK=infinity
    LimitSTACK=infinity
    Delegate=yes

    [Install]
    WantedBy=multi-user.target

将该文件复制到各节点：

    ::
        
        pdcp -w node[1-10],login slurmd.service /lib/systemd/system/

systemd的服务配置文件变更后，各节点重新刷新服务内容后才能利用 **systemctl** 命令重启slurmd服务等：

    ::

        pdsh -w node[1-10],login "systemctl daemon-reload; systemctl restart slurmd"


设置记账账户和用户
==================

::

    # 增加none和test账户并赋予相应权限
    sacctmgr add account none,test Cluster=MyCluster Description="My slurm cluster" Organization="USTC"

    # 增加test1用户属于test账户
    sacctmgr -i add user test1 account=test

########
高级功能
########

GPU等资源配置
=============

各GPU节点 **/etc/slurm/gres.conf** 文件内容：

::

    #AutoDetect=nvml
    Name=gpu Type=v100 File=/dev/nvidia[0-1] # Type设定卡类型，如V100、A100等，需与队列中的gres选项对应，File对应GPU设备名，[0-1]表示有两个GPU设备/dev/nvidia0与/dev/nvidia1
    #Name=mps Count=100

服务质量(QOS)配置
=================

对资源进行限制，比如限制用户的单个作业CPU核数、运行中作业总CPU核数、作业时长等，除了利用队列分区等外，还可以利用服务质量(Quality of Service, QOS)。利用Slurm提交作业时，可以为每个作业赋予一个QOS，与作业相关的QOS有三种途径：

+ 作业调度优先级：Job Scheduling Priority
+ 作业抢占：Job Preemption
+ 作业限制：Job Limits

QOS通过 ``sacctmgr`` 工具在Slurm数据库中进行设定。

作业提交时，需要通过给 ``sbatch`` 、 ``salloc`` 和 ``srun`` 命令添加 ``--qos=`` 选项来设定需要的QOS。

作业调度优先级
^^^^^^^^^^^^^^^

作业调度优先级是在 **priority/multifactor** 插件中设定的一个数字因子。其中一个因子为QOS优先级，每个QOS被定义在Slurm数据库中，且含有与之关联的优先级。请求且获得认可的QOS的作业将在作业多因子优先级计算中包含该QOS相关联的优先级。

为了在多因子优先级计算中启用QOS优先级组件， **PriorityWeightQOS** 配置参数必需在 **slurm.conf** 文件中已被定义，且被赋予一个大于0的整数。

作业的QOS仅影响多因子插件被加载时调度的优先级。

作业抢占
^^^^^^^^

Slurm提供了两种途径用于排队中的作业抢占运行中的作业，释放运行中作业的资源，并且将其分配给排队中作业。

抢占方式是由 **slurm.conf** 文件中 **PreemptType** 配置参数决定的。当 **PreemptType** 设定为 *preempt/qos* 时，一个排队中的作业的QOS将被用于决定其是否可以抢占一个运行中的作业。

QOS可以被赋予（利用 ``sacctmgr`` 命令）一些可以被其抢占的其它QOS的列表。当有一个排队中的具有的QOS允许其抢占一个具有其它QOS运行中的作业，Slurm调度器将抢占该运行中的作业。

QOS **PreemptExemptTime** 参数设定了在作业将抢占之前最小运行时间。QOS选项优先于同名的全局选项。

作业限制
^^^^^^^^

每个QOS被赋予 **OverPartQOS** 一系列的限制，这些限制将应用于作业。这些限制反映了在Slurm数据库中定义的并在下面\ 资源限制_\ 部分中描述的用户/帐户/集群/队列(user/account/cluster/partition)关联所施加的限制。当定义了QOS的限制时，它们将优先于关联的限制。

资源限制
--------

层级
++++

Slurm的层级限制按照下述顺序强制执行，作业QOS和队列QOS顺序如采用了QOS标记 ``OverPartQOS`` 参数则可逆。

    1. 队列QOS限制
    #. 作业QOS限制
    #. 用户关联
    #. 帐户关联，升序排列
    #. Root/Cluster关联
    #. 队列限制
    #. 无

.. note:: 若在层级中定义了多个限制，则在这个列表中的最先定义的限制有效。如下面例子：

    + **MaxJobs=20** ，且 **MaxSubmitJobs** 未在队列QOS中被定义
    + 在作业QOS中没有任何限制
    + **MaxJobs=4** 和 **MaxSubmitJobs=50** 在用户关联中定义

    上面限制实际生效的是 **MaxJobs=20** 和 **MaxSubmitJobs=50** 。

.. note:: 应遵守上述规定的优先顺序，除下列限制外： **Max[Time|Wall]** 、 **[Min|Max]Nodes** 。对这些限制，即使这些队列层级限制在最底部列出，当作业被QOS和/或联合限制时，也无法超过队列层级限制 。因此，对这三种限制的默认是它们是队列限制的上限。若QOS中的 **PartitionTimeLimit** 和/或 **Partition[Max|Min]Nodes** 标记被设置，则该队列层级边界可被忽略，那么该作业将根据上述命令执行QOS和/或关联层级的限制。

设置
++++

调度策略信息必需被存储在数据库中，该数据库由 **slurm.conf** 配置文件中的 **AccountingStorageType** 配置参数定义。调度策略信息可以记录在MySQL或MariaDB数据库中。为了安全和性能，强烈推荐采用SlurmDBD作为数据库的前端。SlurmDBD使用Slurm认证插件（如MUNGE）进行认证，还使用一个已有的Slurm记账存储插件以最大化代码重用。SlurmDBD利用排队中请求的数据缓存和优先化以便优化性能。因为SlurmDBD依赖于现有的Slurm插件用于验证和数据库使用，仅安装SlurmDBD的节点不再需要其它Slurm命令和守护进程，仅需要安装slurmdbd和slurm-plugins RPM包。

记账和调度策略都被配置基于某个关联。每个关联是由集群名、银行账户、用户和Slurm队列（可选）组成的4元组。为了强制使用调度策略，需在 **slurm.conf** 配置文件中设置 **AccountingStorageEnforce** 选项的值。该选项含有采用逗号（,）分隔的一些需要强制的选项。有效值如下：

    + *associations* ：关联。如用户的关联不在数据库中，这将阻止用户运行作业。该参数可以阻止用户访问无效账户。
    + *limits* ：限制。为关联设置强制限制。如设置了该值， *associations* 值也被自动设置。
    + *qos* ：QOS。要求所有作业都将（显式或默认）指定一个有效的QOS。针对每个关联的QOS值都在数据库中定义。如设置了该值， *associations* 值也被自动设置。
    + *safe* ：安全。确保作业只有在采用某具有 **GrpTRESMins** 限制的关联或QOS时才能被启动，且前提是作业能运行完。如未设置该选项，只要作业的使用量未达到cpu-minutes限制，就会启动作业，可能导致作业虽然被启动但在达到限制时即使未完成也被终止。如设置了该值， *associations* 和 *limits* 值也被自动设置。
    + *wckeys* ：工作负载特征键。将阻止用户在它们无权访问的wckey下运行作业。如设置了该值， *associations* 值也被设置，且 **TrackWCKey** 选项被设置为true。

.. note::  关联是由集群名、银行账户、用户名和Slurm队列名（可选）组成的4元组。

           未设置 **AccountingStorageEnforce** （默认行为），作业将基于每个集群中配置的上述slurm策略运行。

工具
++++

用于管理账户策略的工具是 ``sacctmgr`` 。该工具可以被用于创建和删除集群、用户、银行账户和队列记录及其组合关联记录等。

策略的改变将被告知不同集群上的Slurm控制守护进程，且立即生效。当某个关联被删除，属于该关联的所有运行中的和排队中的作业将立即被取消。当限制降低，运行中的作业不会被取消以满足心的限制，但新的较低限制将会被强制生效。

关配置联和QOS中的限制
+++++++++++++++++++++

当处理关联时。多数这些限制都不仅是对某一用户的关联，而且是对每个集群和账户的。如新创建一个针对某用户关联被创建，但调度策略的选项未被指定，那么默认将是：集群/账户对的选项，且如两者都未被指定，则该选项是针对集群的，如果仍未被指定，则限制不生效。

.. note:: 除非特别说明，如果作业请求本身违反了给定的限制，该作业将暂停，除非作业的QOS设置了拒绝限制标志，该标志将使用作业在提交时被拒绝。当根据此标志考虑Grp限制时，Grp限制被视为最大限制。

.. note:: 当采用 ``sacctmgr`` 命令更新一个TRES域，必须指定具体哪个TRES，如：

  设置：

  ::

      sacctmgr modify user bob set GrpTRES=cpu=1500,mem=200,gres/gpu=50

  取消设置：

  ::

      sacctmgr modify user bob set GrpTRES=cpu=-1,mem=-1,gres/gpu=-1

+ **GrpTRESMins** = ：能从关联及其子项或QOS运行的过去、现在和未来作业可能使用的TRES分钟总数。当该限制到达时，所有依赖于该限制的作业将被杀掉，且没有新的作业被允许运行。该用法被衰减（以 **PriorityDecayHalfLife** 设定的衰减速率）。 该选项可以被重置（取决于 **PriorityUsageResetPeriod** 选项）以便允许作业重新基于这些关联树或QOS运行。QOS，如没有 *NoDecay* 被设置，那么 **GrpTRESMins** 将不被衰减。这仅在启用优先级多因子插件时生效。
  *
+ **GrpTRESRunMins** = ：用于限制所有同一个关联和其子项或QOS组合的总TRES分钟数。考虑到运行作业的时间限制并使用它，如果达到限制，在其它作业完成之前不会启动新作业，以允许时间释放作业。
+ **GrpTRES** = ：在给定的任意时间，能从关联及其子项或QOS运行作业的TRES总数。如达到该限制，新作业将处于排队状态，仅当资源被从该组释放时才能开始运行。
+ **GrpJobs** = ：在给定的任意时间，能从关联及其子项或QOS运行作业总数。如达到该限制，新作业将处于排队状态，仅当之前作业结束时才能开始运行。
+ **GrpJobsAccrue** = ：在给定的任意时间，能从关联及其子项QOS中获得年龄优先级的排队中作业的总数。如达到此限制，新作业将排队，但在此组中等待的先前作业删除之前不会累积年龄优先级。此限制并不决定作业是否可以运行，它只限制了优先级的年龄因素。当在QOS上设置时，此限制仅适用于作业QOS，而不适用于队列QOS。
+ **GrpSubmitJobs** = ：在给定的任意时间，能从关联及其子项QOS中提交到系统中的作业总数。如达到该限制，提交新作业将被拒绝，仅当之前作业结束时才能提交。
+ **GrpWall** ：能从关联及其子项或QOS运行作业的墙上时钟最大值。当该限制到达时，在该QOS或关联的未来作业将处于排队状态，直到期能在该限制内运行。该使用量会被衰减（以 **PriorityDecayHalfLife** 选项规定的比率）。它也可以被重置（取决于 **PriorityUsageResetPeriod** 选项），以便允许作业基于该关联数或QOS运行。具有 *NoDecay* 值的QOS不会衰减 **GrpWall** 。
+ **MaxTRESMinsPerJob** = ：作业占用TRES分钟的限制。如达到该限制，该作业如没有运行在Safe模式下将被杀掉，否则该作业将排队直到被给予足够的时间以完成该作业。
+ **MaxTRESPerJob** = ：每个作业能从关联/QOST获取的TRES最大数。
+ **MaxTRESPerNode** = ：每个作业能从关联/QOST获取的每个节点的TRES最大数。
+ **MaxWallDurationPerJob** = ：每个作业能从关联/QOST获取的最大墙上时钟时长。如达到该限制，作业在提交时将被拒绝。
+ **MinPrioThreshold** = ：从关联/QOST获取的用于预留资源的最小优先级。可用于覆盖 **bf_min_prio_reserve** 选项设置。

支持的关联指定调度策略
++++++++++++++++++++++

这些策略代表了关联所特有的调度策略，已在上面列出了QOS共同具有的共享策略和限制。

+ **Fairshare** =：用于决定公平共享优先级的整数。本质上，这是对上述系统针对该关联和其子项的请求总数。也可以使用字符串"parent"，当被用户使用时，意味着针对公平共享采用其父关联。如在该账户设置 **Fairshare=parent** ，该账户的子成员将被有效地利用它们的第一个不是 **Fairshare=parent** 的父母重新支付公平共享计算。限制保持不变，仅影响其公平共享值￼
+ **MaxJobs** =：在给定的任意时间，针对该关联能同时运行的作业总数。如该达到限制，新作业将处于排队状态，只有当该关联的作业有结束时才能运行。
+ **MaxJobsAccrue** =：在给定的任意时间，能从关联允许的排队中作业累计年龄优先级的最大数。当该达到限制，新作业将处于排队中状态，但不累计年龄优先级，直到有关联的作业从排队状态有退出排队。该限制不决定作业是否能运行，它只限制优先级的年龄因子。
+ **MaxSubmitJobs** =：在给定的任意时间，能从该关联提交到系统中的作业最大数。如达到该限制，新提交申请将被拒绝，直到存在该关联的作业退出。
+ **QOS** =：以逗号（,）分隔的能运行的QOS列表。

支持的QOS特定限制
+++++++++++++++++
.. note:: 每个账户(account)可以有多个用户(user)。

+ **MaxJobsAccruePerAccount** =：在任意时间，一个账户（或子账户）能从关联允许的排队中作业累计年龄优先级的最大数。该限制不决定作业是否能运行，它只限制优先级的年龄因子。
+ **MaxJobsAccruePerUser** =： 在任意时间，一个用户能从关联允许的排队中作业累计年龄优先级的最大数。该限制不决定作业是否能运行，它只限制优先级的年龄因子。
+ **MaxJobsPerAccount** =：一个账户（或子账户）能允许的最大同时运行作业数。
+ **MaxJobsPerUser** = 每个用户能同时运行的最大作业数。
+ **MaxSubmitJobsPerAccount** = 每个账户（或子账户）能同时运行和排队等待运行的最大作业数。
+ **MaxSubmitJobsPerUser** = 每个用户能同时运行和排队等待运行的最大作业数。
+ **MaxTRESPerAccount** = 每个账户能同时分的配最大TRES数。
+ **MaxTRESPerUser** = 每个用户能同时分配的最大TRES数。
+ **MinTRESPerJob** = 每个作业能申请的最小TRES尺寸。

**MaxNodes** 和 **MaxWall** 选项已存在于每个队列的基础Slurm配置中，但上述选项提供了基于每个用户基础的限制能力。**MaxJobs** 选项为Slurm提供了一种新机制用于控制集群个人工作负载，以确保在用户中获取某种平衡。

当给一个QOS赋予限制以用于队列QOS，请务必注意这些限制是在QOS层级，而不是对每个队列层级单独实行的。例如，某个QOS具有 **GrpTRES=cpu=20** 限制，且该QOS被赋予两个独立队列，用户将因该QOS被限制到20颗CPU而不是每个队列允许20颗CPU。

公平共享调度是基于Slurm数据库中保持的层级银行账户数据。具体参见 `priority/multifactor <https://slurm.schedmd.com/priority_multifactor.html>`_ 插件。

具体GRES限制
+++++++++++++

当一个GRES具有一种类型与其关联，且针对该类型有一个限制（如， **MaxTRESPerUser=gres/gpu:tesla=1** ），如一个用户请求一个通用gres，则不会强制执行该类型限制。在该情形下，一个附加的用于检查用户请求的lua作业提交插件将非常有用。例如，如果一个请求 **--gres=gpu:2** 受到 **MaxTRESPerUser=gres/gpu:tesla=1** 限制，该限制将不被强制执行，因此还可能得到两个tesla卡资源。

这是缘于设计限制。强制执行该限制的唯一法子是利用作业提交插件组合这些限制，以强制用户请求特定类型模型。

下面是一个基础的lua作业提交插件函数：

.. code:: lua

    function slurm_job_submit(job_desc, part_list, submit_uid)
       if (job_desc.gres ~= nil)
       then
          for g in job_desc.gres:gmatch("[^,]+")
          do
         bad = string.match(g,'^gpu[:]*[0-9]*$')
         if (bad ~= nil)
         then
            slurm.log_info("User specified gpu GRES without type: %s", bad)
            slurm.user_msg("You must always specify a type when requesting gpu GRES")
            return slurm.ERROR
         end
          end
       end
    end

在合适位置含有该脚本和该限制将强制用户总是指明一个GPU具有其类型，因此针对每个特定模型强制该限制。

总是建议针对通用和特定GRES类型设置 **AccountingStorageTRES** ，否则将不会考虑要求提供gres通用实例的请求。例如，可追踪通用GPU和Tesla GPU，将在 **slurm.conf** 中设置：

::

  AccountingStorageTRES=gres/gpu,gres/gpu:tesla
 
参见： `Trackable Resources TRES <https://slurm.schedmd.com/tres.html>`_ 。

队列QOS
^^^^^^^

QOS可与队列相关联，这意味着一个队列将具有相同的限制作为一种QOS。这也将提供一个真正的“浮动”分区的能力，意味着如果你分配所有节点队列和队列的QOS限制Grpcpus或GrpNodes队列将访问所有节点，但只能运行在其内的资源数量。

队列QOS优先级高于作业QOS，将覆盖掉作业QOS。如想相反，在需要在作业QOS中设定OverPartQOS标记，这将反转优先顺序。

其它QOS选项
^^^^^^^^^^^

+ **Flags** ： 被slurmctld用于覆盖或者加强特定特性。有效的值可为：
   - *DenyOnLimit* ：受限时拒绝。如设置，采用该QOS的作业，如果不符合QOS **MAX** 的限制作为独立的作业，将在提交时被拒绝。那些在考虑其它作业时超过这些限制，但在单独考虑时符合这些限制的作业将不会被拒绝。相反，它们将等待到资源直到可用（默认没有 **DenyOnLimit** ）。组限制（如 **GrpTRES** ）也将被视为 **MAX** 限制（如 **MaxTRESPerNode** ），如果这些限制将作为独立作业生效，这些作业将被拒绝。这目前只适用于QOS和关联限制。
   - *EnforceUsageThreshold* ：增强使用量阈值。如设置，且QOS有一个 **UsageThreshold** ，任何关联该QOS提交的作业如低于该 **UsageThresholdany** ，那么直到它们的Fairshare Usage超过该阈值时才运行。
   - *NoReserve* ：不预留。如该标记被设置，且采用回填调度，则采用该QOS的作业在回填调度时不预留资源。该标记初衷为用于关联某QOS可以被作业关联的所有其它QOS（如关联的standby QOS）可以抢占。如该标记被用于某个能被其它所有QOS抢占的QOS，这可能会导致更大的作业饥饿。
   - *PartitionMaxNodes* ：队列最大节点数。如设置，使用此QOS的作业将能够覆盖所请求的分区的MaxNodes限制
   - *PartitionMinNodes* ：队列最小节点数。如设置，使用此QOS的作业将能够覆盖所请求的分区的MinNodes限制
   - *OverPartQOS* ：覆盖队列QOS。如设置，采用该QOS的作业可以覆盖其使用的队列关联的任何QOS限制。
   - *PartitionTimeLimit* ：队列时间限制。如设置，采用该QOS的作业可以覆盖掉其使用的队列的TimeLimit限制。
   - *RequiresReservation* ：请求预留。如设置，采用该QOS的作业提交作业时必须指定一个预留。该选项可用于限制可能具有更大抢占能力的QOS的使用，或仅在预留范围内允许的额外资源。
   - *NoDecay* ：无衰减。如设置，该QOS将不会使它的 **GrpTRESMins** 、 **GrpWall** 和 **UsageRaw** 由于 **slurm.conf** 文件中设定了 **PriorityDecayHalfLife** 或 **PriorityUsageResetPeriod** 而衰减。这将允许该QOS提供那些一旦消耗，将不会自动补充的集合限制。这样的QOS将作为一个可以访问它的关联的一个有时间限制的资源配额。对于使用QOS进行的关联，帐户/用户(Account/user)的使用量仍将衰减。 **QOSGrpTRESMin** 和 **GrpWall** 限制可以增加，或将QOS RawUsage重置为0，以再次允许使用此QOS提交的作业运行（如果作业处于等待中是因为使用 **QOSGrp{TRES}MinutesLimit** 或 **QOSGrpWallLimit** ，其中TRES是某种类型的可跟踪资源）。
   - *UsageFactorSafe* ：使用量因子安全。如设置， **slurm.conf** 文件中 **AccountingStorageEnforce** 选项含有Safe，作业只有在该作业具有 **UsageFactor** 时能完成时才运行。
+ **GraceTime** ：优雅时间。当抢占时，该抢占优雅时间将被扩展到作业。
+ **UsageFactor** ：使用量因子。浮点数，其考虑了作业的TRES使用量（如， **RawUsage** 、 **TRESMins** 和 **TRESRunMins** ）。例如，如 **UsageFactor** 为2，那么每 **TRESBillingUnit** 秒时间运行作业，它将计数为2。如果 **UsageFactor** 为0.5，则每 **TRESBillingUnit** 秒钟只占一半的时间。设置为0，将不会增加作业的时间利用。

    该使用量因子仅对作业QOS有效，对于队列QOS无效。

    如设置了 **UsageFactorSafe** 标记且 **AccountingStorageEnforce** 选项含有 *Safe* ，作业只有在该作业具有 **UsageFactor** 时能完成时才运行。

    如未设置 **UsageFactorSafe** 标记且 **AccountingStorageEnforce** 选项含有 *Safe* ，作业将能够在不应用 **UsageFactor** 的情况下被调度，且不会因为限制而被杀掉。

    如未设置 **UsageFactorSafe** 标记且 **AccountingStorageEnforce** 选项不含有 *Safe* ，一个作业将能够在不应用 **UsageFactor** 的情况下被调度，并可能由于限制而被杀死。

    ``man slurm.conf`` 查看AccountingStorageEnforce。

    默认为1。为了清除先前设置的值，可使用修改命令设置新值为-1。

+ **UsageThreshold** ：使用量阈值。 代表关联中允许运行作业的最低公平份额的一个浮点数。如关联低于此阈值，并且有挂起作业或提交新作业，则这些作业将被保留，直到使用量恢复到高于此阈值为止。使用 ``sshare`` 命令来查看系统上的当前共享。

配置
^^^^

总结上述，QOS和其关联的限制被利用 ``sacctmgr`` 工具命令定义在Slurm数据库中。QOS仅影响启用多因子优先级插件的作业调度的优先级，且非0的 **PriorityWeightQOS** 已经被定义在 **slurm.conf** 文件中。当在 **slurm.conf** 文件中 **PreemptType** 被定义为 *preempt/qos* 时，QOS将仅决定作业抢占。QOS定义的限制将覆盖掉user/account/cluster/partition关联的限制。

QOS常见操作例子
^^^^^^^^^^^^^^^

所有QOS操作都使用 ``sacctmgr`` 命令。 ``sacctmgr show qos`` 命令默认输出非常长，给出了大量限制和选项，为此最好采用格式化选项进行过滤和显示。

默认，当一个集群被加入到数据库中时，会自动生成一个叫normal的QOS。

+ 显示现有的QOS：

::

    sacctmgr show qos format=name,priority

将输出：

::

    Name        Priority
    ---------- ----------
    normal          0

+ 增加一个新QOS：

::

    sacctmgr add qos 128cpu

+ 设置QOS优先级：

::

    sacctmgr modify qos 128cpu set priority=10


+ 更新信息，设置最小CPU核为1：

::

    sacctmgr update qos normal set MinTRES=cpu=1

+ 设置最小CPU核为120

::

    sacctmgr add qos long set MinTRES=cpu=120

+ 增加单用户最大资源MaxTRESPU为144核CPU，标记为144cpu：

::

    sacctmgr add qos 144cpu set MaxTRESPU=cpu=144

+ 设置其它限制：

::

    sacctmgr modify qos 128cpu set grpcpus=128

+ 为用户增加QOS：

::

    sacctmgr modify user user1 set qos=144cpu

+ 用户可以有多个QOS，利用 ``set qos+=`` 设置：

::

    sacctmgr modify user user1 set qos+=Mem64G

+ 查看账户关联的QOS：

::

    sacctmgr show assoc
    sacctmgr show assoc where name=hmli
    sacctmgr show assoc format=cluster,user,qos

将显示：

::

       Cluster       User                  QOS
    ---------- ---------- --------------------
      MyCluster                          normal
      MyCluster      root                normal
      MyCluster                          normal
      MyCluster     user1                144cpu

+ 删除某个QOS：

::

    sacctmgr delete qos Mem64G

########
日常操作
########

+ 如执行 ``sinfo`` 命令显示节点STATE为down、drain等不正常状态：

::

    PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
    batch*       up   infinite      1  down  node1
    batch*       up   infinite      1  drain node2

可先执行 ``scontrol show node 节点名`` 之类的命令查看具体原因（Reason=对应的）：
::

    NodeName=node1 Arch=x86_64 CoresPerSocket=24
       CPUAlloc=0 CPUTot=48 CPULoad=0.01
       AvailableFeatures=(null)
       ActiveFeatures=(null)
       Gres=(null)
       NodeAddr=192.168.1.1 NodeHostName=node1 Version=21.08.3
       OS=Linux 3.10.0-1160.45.1.el7.x86_64 #1 SMP Wed Oct 13 17:20:51 UTC 2021
       RealMemory=192000 AllocMem=0 FreeMem=189885 Sockets=2 Boards=1
       State=DOWN+DRAIN ThreadsPerCore=1 TmpDisk=0 Weight=1 Owner=N/A MCS_label=N/A
       Partitions=batch
       BootTime=2021-11-10T09:56:45 SlurmdStartTime=2021-11-10T15:57:12
       LastBusyTime=2021-11-10T15:57:15
       CfgTRES=cpu=48,mem=187.50G,billing=48
       AllocTRES=
       CapWatts=n/a
       CurrentWatts=0 AveWatts=0
       ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s
       Reason=Not responding [slurm@2021-10-19T12:56:19]


必要时需要到对应计算节点执行 ``systemctl restart slurmd`` 命令。

如果计算节点slurmd服务正常，但执行 ``sinfo`` 命令显示：drain，也许要在管理节点执行 ``scontrol update NodeName=节点名 State=resume`` 命令。

+ 设置某个节点状态为DRAIN，执行：

::
    
    scontrol update NodeName=节点名 State=Drain Reason='Memomry Lost'


+ 设置队列状态，执行：

::
    
    scontrol update Partition=队列名 State=[UP|DOWN|DRAIN|INACTIVE]

状态：
     - UP：设置该队列允许新作业可以在该队列排队，且将会被分配到该队列对应的节点上运行
     - DOWN：设置该队列允许新作业可以在该队列排队，但排队中的作业不会分配到该队列对应的节点上运行。已经在该队列上运行的作业将继续运行。
     - DRAIN：设置该队列不允许新作业在该队列排队（提交时显示被拒绝信息）。已在排队的作业将被分配到该队列对应节点上运行。
     - INACTIVE：设置该队列运行新的作业在该队列排队，已经在排队的老作业将不会被分配到对应节点运行。

