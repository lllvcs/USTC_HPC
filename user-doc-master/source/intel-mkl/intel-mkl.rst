Intel MKL数值函数库
===================

本系统上安装的数值函数库主要有Intel核心数学库(Math Kernel Library, MKL)，用户可以直接调用，以提高性能、加快开发。

当前安装的Intel MKL版本为Intel Parallel Studio XE 2018、2019和2020版编译器自带的，安装在。在BASH下可以通过运行\ ``module load``\ 选择Intel编译器时设置，或者在之类的环境变量设置文件中添加类似下面代码设置Intel MKL所需的环境变量\ ``INCLUDE``\ 、\ ``LD_LIBRARY_PATH``\ 和\ ``MANPATH``\ 等：

.. code:: bash

    . /opt/intel/2020/mkl/bin/mklvars.sh intel64

Intel MKL主要内容
~~~~~~~~~~~~~~~~~

Intel MKL主要包含如下内容：

-  基本线性代数子系统库（BLAS, level 1, 2, 3）和线性代数库（LAPACK）：提供向量、向量-矩阵、矩阵-矩阵操作。

-  ScaLAPACK分布式线性代数库：含基础线性代数通信子程序（Basic Linear Algebra Communications Subprograms, BLACS）和并行基础线性代数子程序（Parallel Basic Linear Algebra Subprograms, PBLAS）

-  PARDISO直接离散算子：一种迭代离散算子，支持用于求解方程的离散系统的离散BLAS (level 1, 2, and 3)子函数，并提供可用于集群系统的分布式版本的PARDISO。

-  快速傅立叶变换方程（Fast Fourier transform, FFT）：支持1、2或3维，支持混合基数（不局限与2的次方），并有分布式版本。

-  向量数学库（Vector Math Library, VML）： 提供针对向量优化的数学操作。

-  向量统计库（Vector Statistical Library, VSL）：提供高性能的向量化随机数生成算子，可用于一些几率分布、剪辑和相关例程和汇总统计功能。

-  数据拟合库（Data Fitting Library）：提供基于样条函数逼近、函数的导数和积分，及搜索。

-  扩展本征解算子（Extended Eigensolver）：基于FEAST的本征值解算子的共享内存版本的本征解算子。

Intel MKL目录内容
~~~~~~~~~~~~~~~~~

Intel MKL的主要目录内容见下表 。

.. container::
   :name: mo

   .. table:: Intel MKL目录内容

      +--------------------------------------+------------------------------------------------------------------+
      | 目录                                 | 内容                                                             |
      +======================================+==================================================================+
      | <mkl_dir>                            | MKL主目录，如 ``/opt/intel/2020/mkl``                            |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/benchmarks/linpack         | 包含OpenMP版的LINPACK的基准程序                                  |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/benchmarks/mp_linpack      | 包含MPI版的LINPACK的基准程序                                     |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/bin                        | 包含设置MKL环境变量的脚本                                        |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/bin/ia32                   | 包含针对IA-32架构设置MKL环境变量的脚本                           |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/bin/intel64                | 包含针对Intel64架构设置MKL环境变量的脚本                         |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/examples                   | 一些例子，可以参考学习                                           | 
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/include                    | 含有INCLUDE文件                                                  |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/include/ia32               | 含有针对ia32 Intel编译器的Fortran 95 .mod文件                    | 
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir >/include/intel64/ilp64     | 含有针对Intel64 Intel编译器ILP64接口 [3]_ 的Fortran  95 .mod文件 |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/include/intel64/lp64       | 含有针对Intel64 Intel编译器LP64接口的Fortran 95 .mod文件         |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/include/mic/ilp64          | 含针对MIC架构ILP64接口的Fortran 95 .mod文件，本系统未配置MIC     |
      +--------------------------------------+------------------------------------------------------------------+
      | <mk _dir>/include/mic/lp64l          | 含针对MIC架构LP64接口的Fortran 95 .mod文件，本系统未配置MIC      |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/include/fftw               | 含有FFTW2和3的INCLUDE文件                                        |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/blas95          | 包含BLAS的Fortran90封装及用于编译成库的makefile                  |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/LAPACK95        | 包含LAPACK的Fortran 90封装及用于编译成库的makefile               |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/fftw2xc         | 包含2.x版FFTW(C接口)封装及用于编译成库的makefile                 |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/fftw2xf         | 包含2.x版FFTW(Fortran接口)封装及用于编译成库的makefile           |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/fftw2x_cdft     | 包含2.x版集群FFTW(MPI接口)封装及用于编译成库的makefile           |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/fftw3xc         | 包含3.x版FFTW(C接口)封装及用于编译成库的makefile                 |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/interfaces/fftw3xf         | 包含3.x版FFTW(Fortran接口)封装及用于编译成库的makefile           |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir >/interfaces/fftw3x_cdft    | 包含3.x版集群FFTW(MPI接口)封装及用于编译成库的makefile           |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir >/interfaces/fftw2x_cdft    | 包含2.x版MPIFFTW(集群FFT)封装及用于编译成库的makefile            |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/lib/ia32                   | 包含IA32架构的静态库和共享目标文件                               |
      +--------------------------------------+------------------------------------------------------------------+
      |  <mkl_dir>/lib/intel64               | 包含EM64T架构的静态库和共享目标文件                              |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/lib/mic                    | 用于MIC协处理器，本系统未配置MIC                                 |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/tests                      | 一些测试文件                                                     |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/tools                      | 工具及插件                                                       |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/tools/builder              | 包含用于生成定制动态可链接库的工具                               |
      +--------------------------------------+------------------------------------------------------------------+
      | <mkl_dir>/../Documentation/en_US/mkl | MKL文档目录                                                      |
      +--------------------------------------+------------------------------------------------------------------+

链接Intel MKL
~~~~~~~~~~~~~

快速入门
^^^^^^^^

利用-mkl编译器参数
''''''''''''''''''

Intel Composer XE编译器支持采用-mkl [4]_ 参数链接Intel MKL：

-  -mkl或-mkl=parallel：采用标准线程Intel MKL库链接；

-  -mkl=sequential：采用串行Intel MKL库链接；

-  -mkl=cluster：采用Intel MPI和串行MKL库链接；

-  对Intel 64架构的系统，默认使用LP64接口链接程序。

使用单一动态库
''''''''''''''

可以通过使用Intel MKL Single Dynamic Library(SDL)来简化链接行。

为了使用SDL库，请在链接行上添加libmkl_rt.so。例如

``icc application.c -lmkl_rt``

SDL使得可以在运行时选择Intel MKL的接口和线程。默认使用SDL链接时提供：

-  对Intel 64架构的系统，使用LP64接口链接程序；

-  Intel线程。

如需要使用其它接口或改变线程性质，含使用串行版本Intel MKL等，需要使用函数或环境变量来指定选择，参见\ `4.3.2 <#dsi>`__\ 动态选择接口和线程层部分。

选择所需库进行链接
''''''''''''''''''

选择所需库进行链接，一般需要：

-  从接口层(Interface layer)和线程层(Threading layer)各选择一个库；

-  从计算层(Computational layer)和运行时库(run-time libraries, RTL)添加仅需的库。

链接应用程序时的对应库参见下表。

+------------------------+----------------------+------------------------+----------------+-------------+
|                        | 接口层               | 线程层                 | 计算层         | 运行库      |
+========================+======================+========================+================+=============+
| IA-32架构，静态链接    | libmkl_intel.a       | libmkl_intel_thread.a  | libmkl_core.a  | libiomp5.so |
+------------------------+----------------------+------------------------+----------------+-------------+
| IA-32架构，动态链接    | libmkl_intel.so      | libmkl_intel_thread.so | libmkl_core.so | libiomp5.so |
+------------------------+----------------------+------------------------+----------------+-------------+
| Intel 64架构，静态链接 | libmkl_intel_lp64.a  | libmkl_intel_thread.a  | libmkl_core.a  | libiomp5.so |
+------------------------+----------------------+------------------------+----------------+-------------+
| Intel64架构，动态链接  | libmkl_intel_lp64.so | libmkl_intel_thread.so | libmkl_core.so | libiomp5.so |
+------------------------+----------------------+------------------------+----------------+-------------+

SDL会自动链接接口、线程和计算库，简化了链接处理。下表列出的是采用SDL动态链接时的Intel MKL库。参见\ `4.3.2 <#dsi>`__\ 动态选择接口和线程层部分，了解如何在运行时利用函数调用或环境变量设置接口和线程层。

=================== ============ ===========
\                   SDL          运行时库
=================== ============ ===========
IA-32和Intel 64架构 libmkl_rt.so libiomp5.so
=================== ============ ===========

使用链接行顾问
''''''''''''''

Intel提供了网页方式的链接行顾问帮助用户设置所需要的MKL链接参数。访问\ http://software.intel.com/en-us/articles/intel-mkl-link-line-advisor\ ，按照提示输入所需要信息即可获取链接Intel MKL时所需要的参数。

使用命令行链接工具
''''''''''''''''''

使用Intel MKL提供的命令行链接工具可以简化使用Intel MKL编译程序。本工具不仅可以给出所需的选项、库和环境变量，还可执行编译和生成可执行程序。\ ``mkl_link_tool``\ 命令安装在，主要有三种模式：

-  查询模式：返回所需的编译器参数、库或环境变量等：

   -  获取Intel MKL库：\ ``mkl_link_tool -libs [Intel MKL Link Tool options]``

   -  获取编译参数：\ ``mkl_link_tool -opts [Intel MKL Link Tool options]``

   -  获取编译环境变量：\ ``mkl_link_tool -env [Intel MKL Link Tool options]``

-  编译模式：可编译程序。

   -  用法：\ ``mkl_link_tool [options] <compiler> [options2] file1 [file2 ...]``

-  交互模式：采用交互式获取所需要的参数等。

   -  用法：\ ``mkl_link_tool -interactive``

参见\ http://software.intel.com/en-us/articles/mkl-command-line-link-tool\ 。

链接举例
^^^^^^^^

在Intel 64架构上链接
''''''''''''''''''''

在这些例子中：

-  MKLPATH=$MKLROOT/lib/intel64

-  MKLINCLUDE=$MKLROOT/include

如果已经设置好环境变量，那么在所有例子中可以略去-I$MKLINCLUDE，在所有动态链接的例子中可以略去-L$MKLPATH。

-  使用LP64接口的并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -Wl,--start-group $MKLPATH/libmkl_intel_lp64.a $MKLPATH/libmkl_intel_thread.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -liomp5 -lpthread -lm

-  使用LP64接口的并行Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -liomp5 -lpthread -lm

-  使用LP64接口的串行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -Wl,--start-group $MKLPATH/libmkl_intel_lp64.a $MKLPATH/libmkl_sequential.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -lpthread -lm

-  使用LP64接口的串行Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lpthread -lm

-  使用ILP64接口的并行Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -Wl,--start-group $MKLPATH/libmkl_intel_ilp64.a $MKLPATH/libmkl_intel_thread.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -liomp5 -lpthread -lm

-  使用ILP64接口的并行Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -lmkl_intel_ilp64 -lmkl_intel_thread -lmkl_core -liomp5 -lpthread -lm

-  使用串行或并行（调用函数或设置环境变量选择线程或串行模式，并设置接口）Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -lmkl_rt

-  使用Fortran 95 LAPACK接口和LP64接口的并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE -I$MKLINCLUDE/intel64/lp64 \
    -lmkl_lapack95_lp64 -Wl,--start-group $MKLPATH/libmkl_intel_lp64.a \
    $MKLPATH/libmkl_intel_thread.a $MKLPATH/libmkl_core.a \
    -Wl,--end-group -liomp5 -lpthread -lm

-  使用Fortran 95 BLAS接口和LP64接口的并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE -I$MKLINCLUDE/intel64/lp64 \
    -lmkl_blas95_lp64 -Wl,--start-group $MKLPATH/libmkl_intel_lp64.a \
    $MKLPATH/libmkl_intel_thread.a $MKLPATH/libmkl_core.a \
    -Wl,--end-group -liomp5 -lpthread -lm


在IA-32架构上链接
'''''''''''''''''

在这些例子中：

-  MKLPATH=$MKLROOT/lib/ia32

-  MKLINCLUDE=$MKLROOT/include

如果已经设置好环境变量，那么在所有例子中可以略去-I$MKLINCLUDE，在所有动态链接的例子中可以略去-L$MKLPATH。

-  使用并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -Wl,--start-group $MKLPATH/libmkl_intel.a $MKLPATH/libmkl_intel_thread.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -liomp5 -lpthread -lm

-  使用并行Intel MKL库动态链接myprog.f：

.. code:: bash

   ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
   -lmkl_intel -lmkl_intel_thread -lmkl_core -liomp5 -lpthread -lm

-  使用串行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE \
    -Wl,--start-group $MKLPATH/libmkl_intel.a $MKLPATH/libmkl_sequential.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -lpthread -lm


-  使用串行Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE -lmkl_intel -lmkl_sequential -lmkl_core -lpthread -lm

-  使用串行或并行（调用mkl_set_threading_layer函数或设置环境变量MKL\_THREADING_LAYER选择线程或串行模式）Intel MKL库动态链接myprog.f：

.. code:: bash

    ifort myprog.f -lmkl_rt

-  使用Fortran 95 LAPACK接口和并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE -I$MKLINCLUDE/ia32 -lmkl_lapack95 \
    -Wl,--start-group $MKLPATH/libmkl_intel.a $MKLPATH/libmkl_intel_thread.a

-  使用Fortran 95 BLAS接口和并行Intel MKL库静态链接myprog.f：

.. code:: bash

    ifort myprog.f -L$MKLPATH -I$MKLINCLUDE -I$MKLINCLUDE/ia32 -lmkl_blas95 \
    -Wl,--start-group $MKLPATH/libmkl_intel.a $MKLPATH/libmkl_intel_thread.a \
    $MKLPATH/libmkl_core.a -Wl,--end-group -liomp5 -lpthread -lm


链接细节
^^^^^^^^

在命令行上列出所需库链接
''''''''''''''''''''''''

注意：下面是动态链接的命令，如果想静态链接，需要将含有-l的库名用含有库文件的路径来代替，比如用$MKLPATH/libmkl_core.a代替-lmkl_core，其中$MKLPATH为用户定义的指向MKL库目录的环境变量。

| 注：\ ``[]``\ 内的表示可选，\ ``|``\ 表示其中之一、\ ``{}``\ 表示含有。
  在静态链接时，在分组符号（如，\ ``-Wl,--start-group $MKLPATH/libmkl_cdft_core.a $MKLPATH/libmkl_blacs_``
| ``intelmpi_ilp64.a $MKLPATH/libmkl_intel_ilp64.a $MKLPATH/libmkl_intel_thread.a\``
| ``$MKLPATH/libmkl_core.a -Wl,--end-group``\ ）封装集群组件、接口、线程和计算库。

列出库的顺序是有要求的，除非是封装在上面分组符号中的。

.. _dsi:

动态选择接口和线程层链接
''''''''''''''''''''''''

SDL接口使得用户可以动态选择Intel MKL的接口和线程层。

-  设置接口层

   可用的接口与系统架构有关，对于Intel
   64架构，可使用LP64和ILP64接口。在运行时设置接口，可调用mkl_set_interface_layer函数或设置\ ``MKL_INTERFACE_LAYER``\ 环境变量。下表为可用的接口层的值。

   ====== ============================= ===============================
   接口层 ``MKL_INTERFACE_LAYER``\ 的值 mkl_set_interface_layer的参数值
   ====== ============================= ===============================
   LP64   LP64                          MKL_INTERFACE_LP64
   ILP64  ILP64                         MKL_INTERFACE_ILP64
   ====== ============================= ===============================

   如果调用了mkl_set_interface_layer函数，那么环境变量\ ``MKL_INTERFACE_LAYER``\ 的值将被忽略。默认使用LP64接口。

-  设置线程层

   在运行时设置线程层，可以调用mkl_set_threading_layer函数或者设置环境变量\ ``MKL_THREADING_LAYER``\ 。下表为可用的线程层的值。

   ========= ============================= ===============================
   线程层    ``MKL_INTERFACE_LAYER``\ 的值 mkl_set_interface_layer的参数值
   ========= ============================= ===============================
   Intel线程 INTEL                         MKL_THREADING_INTEL
   串行线程  SEQUENTIAL                    MKL_THREADING_SEQUENTIAL
   GNU线程   GNU                           MKL_THREADING_GNU
   PGI线程   PGI                           MKL_THREADING_PGI
   ========= ============================= ===============================

   如果调用了mkl_set_threading_layer函数，那么环境变量MKL_THREADING_LAYER的值被忽略。默认使用Intel线程。

.. _lp:

使用接口库链接
''''''''''''''

-  使用ILP64接口 vs. LP64接口

   Intel MKL ILP64库采用64-bit整数（索引超过含有\ :math:`2^{31}-1`\ 个元素的大数组时使用），而LP64库采用32-bit整数索引数组。

   LP64和ILP64接口在接口层实现，分别采用下面接口层链接使用LP64或ILP64：

   -  静态链接：libmkl_intel_lp64.a或libmkl_intel_ilp64.a

   -  动态链接：libmkl_intel_lp64.so或libmkl_intel_ilp64.so

   ILP64接口提供以下功能：

   -  支持大数据数组（具有超过\ :math:`2^{31}-1`\ 个元素）；

   -  添加-i8编译器参数编译Fortran程序。

   LP64接口提供与以前Intel MKL版本的兼容，因为LP64对于仅提供一种接口的版本低于9.1的Intel MKL来说是一个新名字。如果用户的应用采用Intel MKL计算大数据数组或此库也许在将来会用到时请选择使用ILP64接口。

   Intel MKL提供的ILP64和LP64头文件路径是相同的。

   -  采用LP64/ILP64编译

      下面显示如何采用ILP64和LP64接口进行编译：

      -  | Fortran：
         | ILP64：\ ``ifort -i8 -I<mkl directory>/include ...``
         | LP64：\ ``ifort -I<mkl directory>/include ...``

      -  | C/C++：
         | ILP64：\ ``icc -DMKL_ILP64 -I<mkl directory>/include ...``
         | LP64：\ ``icc -I<mkl directory>/include ...``

      注意，采用-i8或-DMKL_ILP64选项链接LP64接口库时也许将会产生预想不到的错误。

   -  编写代码

      如果不使用ILP64接口，无需修改代码。

      为了移植或者新写代码使用ILP64接口，需要使用正确的Intel MKL函数和子程序的参数类型：

      +--------------------------------------------------------+----------------------------+-----------+
      | 整数类型                                               | Fortran                    | C/C++     |
      +========================================================+============================+===========+
      | 32-bit整数                                             | INTEGER*4或INTEGER(KIND=4) | int       |
      +--------------------------------------------------------+----------------------------+-----------+
      | 针对ILP64/LP64的通用整数（ILP64使用64-bit，其余32-bit）| INTEGER，不指明KIND        | MKL_INT   |
      +--------------------------------------------------------+----------------------------+-----------+
      | 针对ILP64/LP64的通用整数（64-bit整数）                 | INTEGER*8或INTEGER(KIND=8) | MKL_INT64 |
      +--------------------------------------------------------+----------------------------+-----------+
      | 针对ILP64/LP64的FFT接口                                | INTEGER，不指明KIND        | MKL_LONG  |
      +--------------------------------------------------------+----------------------------+-----------+

   -  局限性

      所有Intel MKL函数都支持ILP64编程，但是针对Intel MKL的FFTW接口：

      -  FFTW 2.x封装不支持ILP64；

      -  FFTW 3.2封装通过专用功能函数plan_guru64支持ILP64。

-  使用Fortran 95接口库

   libmkl_blas95*.a和libmkl_lapack95*.a库分别含有BLAS和LAPACK所需的Fortran 95接口，并且是与编译器无关。在Intel MKL包中，已经为Intel Fortran编译器预编译了，如果使用其它编译器，请在使用前先编译。

使用线程库链接
''''''''''''''

-  串行库模式

   采用Intel MKL串行（非线程化）模式时，Intel MKL运行非线程化代码。它是线程安全的（除了LAPACK已过时的子程序?lacon），即可以在用户程序的OpenMP代码部分使用。串行模式不要求与OpenMP运行时库的兼容，环境变量\ ``OMP_NUM_THREADS``\ 或其Intel MKL等价变量对其也无影响。

   只有在不需要使用Intel MKL线程时才应使用串行模式。当使用一些非Intel编译器线程化程序或在需要非线程化库（比如使用MPI的一些情况时）的情形使用Intel MKL时，串行模式也许有用。为了使用串行模式，请选择*sequential.*库。

   对于串行模式，由于*sequential.*依赖于pthread，请在链接行添加POSIX线程库(pthread)。

-  选择线程库层

   一些Intel MKL支持的编译器使用OpenMP线程技术。Intel MKL支持这些编译器提供OpenMP技术实现，为了使用这些支持，需要采用正确的线程层和编译器支持运行库进行链接。

   -  线程层

      每个Intel MKL线程库包含针对同样的代码采用不同编译器（Intel、GNU和PGI编译器）分别编译的库。

   -  运行时库

      此层包含Intel编译器兼容的OpenMP运行时库libiomp。在Intel编译器之外，libiomp提供在Linux操作系统上对更多线程编译器的支持。即，采用GNU编译器线程化的程序可以安全地采用intel MKL和libiomp链接。

      下表有助于解释在不同情形下使用Intel MKL时选择线程库和运行时库（仅静态链接情形）：

      +--------+----------------+-----------------------+--------------------+----------------------------+
      | 编译器 | 应用是否线程化 | 线程层                | 推荐的运行时库     | 备注                       |
      +========+================+=======================+====================+============================+
      | Intel  | 无所谓         | libmkl_intel_thread.a | libiomp5.so        |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | PGI    | Yes            | libmkl_pgi_thread.a或 | 由PGI*提供         | 使用libmkl_sequential.a    |
      |        |                | libmkl_sequential.a   |                    | 从IntelMKL调用中去除线程化 |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | PGI    | No             | libmkl_intel_thread.a | libiomp5.so        |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | PGI    | No             | libmkl_pgi_thread.a   | 由PGI*提供         |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | PGI    | No             | libmkl_sequential.a   | None               |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | GNU    | Yes            | libmkl_gnu_thread.a   | libiomp5.so或库    | libiomp5提供监控缩放性能   |
      |        |                |                       | GNU OpenMP运行时库 |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | GNU    | Yes            | libmkl_sequential.a   | None               |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | GNU    | No             | libmkl_intel_thread.a | libiomp5.so        |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | other  | Yes            | libmkl_sequential.a   | None               |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+
      | other  | No             | libmkl_intel_thread.a | libiomp5.so        |                            |
      +--------+----------------+-----------------------+--------------------+----------------------------+

使用计算库链接
''''''''''''''

-  如不使用Intel MKL集群软件在链接应用程序时只需要一个计算库即可，其依赖于链接方式：

   -  静态链接： libmkl_core.a

   -  动态链接： libmkl_core.so

-  采用Intel MKL集群软件的计算库

   ScaLAPACK和集群Fourier变换函数(Cluster FFTs)要求更多的计算库，其也许依赖于架构。下表为列出的针对Intel 64架构的使用ScaLAPACK或集群FFTs的计算库：

   +----------------------+-----------------------------------------+-------------------------------------------+
   | 函数域               | 静态链接                                | 动态链接                                  |
   +======================+=========================================+===========================================+
   | ScaLAPACK，LP64接口  | libmkl_scalapack_lp64.a和libmkl_core.a  | libmkl_scalapack_lp64.so和libmkl_core.so  |
   +----------------------+-----------------------------------------+-------------------------------------------+
   | ScaLAPACK，ILP64接口 | libmkl_scalapack_ilp64.a和libmkl_core.a | libmkl_scalapack_ilp64.so和libmkl_core.so |
   +----------------------+-----------------------------------------+-------------------------------------------+
   | 集群FFTs             | libmkl_cdft_core.a和libmkl_core.a       | libmkl_cdft_core.so和libmkl_core.so       |
   +----------------------+-----------------------------------------+-------------------------------------------+

下表为列出的针对IA-32架构的使用 ScaLAPACK或集群FFTs的计算库：

+-----------+----------------------------------------+------------------------------------------+
| 函数域    | 静态链接                               | 动态链接                                 |
+===========+========================================+==========================================+
| ScaLAPACK | libmkl_scalapack_core.a和libmkl_core.a | libmkl_scalapack_core.so和libmkl_core.so |
+-----------+----------------------------------------+------------------------------------------+
| 集群FFTs  | libmkl_cdft_core.a和libmkl_core.a      | libmkl_cdft_core.so和libmkl_core.so      |
+-----------+----------------------------------------+------------------------------------------+

注意：对于ScaLAPACK和集群FFTs，当在MPI程序中使用时，还需要添加BLACS库。

使用编译器运行库链接
''''''''''''''''''''

甚至在静态链接其它库时，也可动态链接libiomp5、兼容的OpenMP运行时库。

静态链接libiomp5也许会存在问题，其原因由于操作环境或应用越复杂，将会包含更多多余的库的复本。这将不仅会导致性能问题，甚至导致不正确的结果。

动态链接libiomp5时，需确保\ ``LD_LIBRARY_PATH``\ 环境变量设置正确。

使用系统库链接
''''''''''''''

使用Intel MKL的FFT、Trigonometric Transforn或Poisson、Laplace和Helmholtz求解程序时，需要通过在链接行添加-lm参数链接数学支持系统库。

在Linux系统上，由于多线程libiomp5库依赖于原生的pthread库，因此，在任何时候，libiomp5要求在链接行随后添加-lpthread参数（列出的库的顺序非常重要）。

冗长（Verbose）启用模式链接
'''''''''''''''''''''''''''

如果应用调用了MKL函数，您也许希望知道调用了哪些计算函数，传递给它们什么参数，并且花费多久执行这些函数。当启用Intel MKL冗长（Verbose）模式时，您的应用可以打印出这些信息。可以打印出这些信息的函数称为冗长启用函数。并不是所有Intel MKL函数都是冗长启用的，请查看Intel MKL发布说明。

在冗长模式下，每个冗长启用函数的调用都将打印人性化可读行描述此调用。如果此应用在此函数调用中终止，不会有针对此函数的信息打印出来。第一个冗长启用函数调用将打印一版本信息行。

为了对应用启用Intel MKL冗长模式，需要执行以下两者之一：

-  设置环境变量MKL_VERBOSE为1，在bash下可以执行\ ``export MKL_VERBOSE=1``

-  调用支持函数mkl_verbose(1)

函数调用mkl_verbose(0)将停止冗长模式。调用启用或禁止冗长模式的函数将覆盖掉环境变量设置。关于mkl_verbose函数，请参看Intel MKL Reference Manual。

Intel MKL冗长模式不是线程局域的，而是全局状态。这意味着如果一个应用从多线程中改变模式，其结果将是未定义的。

性能优化等
~~~~~~~~~~

请参见Intel MKL官方手册。

.. [3]
   ILP64接口和LP64接口的区别参见\ \ `4.3.3 <#lp>`__\ \ 使用接口库链接。

.. [4]
   是-mkl，不是-lmkl，其它编译器未必支持此-mkl选项。
