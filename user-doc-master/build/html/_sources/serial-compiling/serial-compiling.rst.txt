串行及OpenMP程序编译及运行
==========================

在本超算系统上可运行C/C++、Fortran的串行程序，以及与OpenMP和MPI结合的并行程序。编译程序时，用户只需在登录节点(login01、login02)上以相应的编译命令和选项进行编译即可（用户不应到其余节点上进行编译，以免影响系统效率。其它节点一般只设置了运行作业所需要的库路径等，未必设置了编译环境）。当前安装的编译环境主要为：

-  C/C++、Fortran编译器：Intel、PGI [#pgi]_ 和GNU编译器，支持OpenMP并行。

-  MPI并行环境：HPC-X、Open MPI和Intel MPI并行环境。

安装目录为 ``/opt`` 等，系统设置采用module进行管理（参见\ `[module] <../module-environment/module-environment.html#module>`__\ ），用户可以采用下述方式之一等需设置自己所需的编译环境运行，如：

-  ``module load intel/2020``

-  在中设置（设置完成后需要\ ``source ~/.bashrc``\ 或重新登录以便设置生效）：

.. code:: bash

   . /opt/intel/2020/bin/compilervars.sh intel64

注意：在中设置的级别有可能要高于使用\ ``module load``\ 设置的，可以运行\ ``icc -v``\ 或\ ``which icc``\ 等命令查看实际使用的编译环境。

建议采用对一般程序来说性能较好的Intel编译器，用户也可以选择适合自己程序的编译器，以取得更好的性能。

本部分主要介绍串行C/C++
Fortran源程序和OpenMP并行程序的编译，MPI并行程序的编译将在后面介绍。

串行C/C++程序的编译
~~~~~~~~~~~~~~~~~~~

输入输出文件后缀与类型的关系
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

编译器默认将按照输入文件的后缀判断文件类型，见下表。

.. container::
   :name: ci

   .. table:: 输入文件后缀与类型的关系

     +------------+---------------+-----------------+
     |文件名      |解释           |动作             |
     +============+===============+=================+
     |filename.c  | C源文件       |传给编译器       |
     |filename.C  |               |                 |
     |filename.CC |               |                 |
     +------------+---------------+-----------------+
     |filename.cc | C++源文件     |传给编译器       |
     |filename.cpp|               |                 |
     |filename.cxx|               |                 |
     +------------+---------------+-----------------+
     |filename.a  | 静态链接库文件|传递给链接器     |
     |filename.so | 动态链接库文件|                 |
     +------------+---------------+-----------------+
     |filename.i  | 已预处理的文件|传递给标准输出   |
     +------------+---------------+-----------------+
     |filename.o  | 目标文件      |传递给链接器     |
     +------------+---------------+-----------------+
     |filename.s  | 汇编文件      |传递给汇编器     |
     +------------+---------------+-----------------+

编译器默认将输出按照文件类型与后缀相对应，见下表。

.. container::
   :name: co

   .. table:: 输出文件后缀与文件类型的关系

      ========== ==================================
      文件名     解释
      ========== ==================================
      filename.i 已预处理的文件，一般使用-p选项生成

      filename.o 目标文件，一般使用-c选项生成

      filename.s 汇编文件，一般使用-s选项生成

      a.out      默认生成的可执行文件
      ========== ==================================

串行C/C++程序编译举例
^^^^^^^^^^^^^^^^^^^^^

-  Intel C/C++编译器：

   -  | 将C程序yourprog.c编译为可执行文件yourprog：
      | ``icc -o yourprog yourprog.c``

   -  | 将C++程序yourprog.cpp编译为可执行文件yourprog：
      | ``icpc -o yourprog yourprog.cpp``

   -  | 将C程序yourprog.c编译为对象文件yourprog.o而不是可执行文件：
      | ``icc -c yourprog.c``

   -  | 将C程序yourprog.c编译为汇编文件yourprog.s而不是可执行文件：
      | ``icc -S yourprog.c``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``icc -g yourprog.c -o yourprog``

   -  | 指定头文件路径编译：
      | ``icc -I/alt/include -o yourprog yourprog.c``

   -  | 指定库文件路径及库名编译：
      | ``icc -L/alt/lib -lxyz -o yourprog yourprog.c``

-  PGI C/C++编译器：

   -  | 将C程序yourprog.c编译为可执行文件yourprog：
      | ``pgcc -o yourprog yourprog.c``

   -  | 将C++程序yourprog.cpp编译为可执行文件yourprog：
      | ``pgCC -o yourprog yourprog.cpp``

   -  | 将C程序yourprog.c编译为对象文件yourprog.o而不是可执行文件：
      | ``pgcc -c yourprog.c``

   -  | 将C程序yourprog.c编译为汇编文件yourprog.s而不是可执行文件：
      | ``pgcc -S yourprog.c``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``pgcc -g yourprog.c -o yourprog``

   -  | 指定头文件路径编译：
      | ``pgcc -I/alt/include -o yourprog yourprog.c``

   -  | 指定库文件路径及库名(-l)编译：
      | ``pgcc -L/alt/lib -lxyz -o yourprog yourprog.c``

-  GNU C/C++编译器：

   -  将C程序yourprog.c编译为可执行文件yourprog：
      ``gcc -o yourprog yourprog.c``

   -  | 将C++程序yourprog.cpp编译为可执行文件yourprog：
      | ``g++ -o yourprog yourprog.cpp``

   -  | 将C程序yourprog.c编译为对象文件yourprog.o而不是可执行文件：
      | ``gcc -c yourprog.c``

   -  | 将C程序yourprog.c编译为汇编文件yourprog.s而不是可执行文件：
      | ``gcc -S yourprog.c``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``gcc -g yourprog.c -o yourprog``

   -  | 指定头文件路径编译：
      | ``gcc -I/alt/include -o yourprog yourprog.c``

   -  | 指定库文件路径及库名编译：
      | ``gcc -L/alt/lib -lxyz -o yourprog yourprog.c``

串行Fortran程序的编译
~~~~~~~~~~~~~~~~~~~~~

.. _输入输出文件后缀与类型的关系-1:

输入输出文件后缀与类型的关系
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

编译器默认将按照输入文件的后缀判断文件类型，见下表。

.. container::
   :name: fi

   .. table:: 输入文件后缀与文件类型的关系

      +--------------+--------------------------+--------------------------+
      | 文件名       | 解释                     | 动作                     |
      +==============+==========================+==========================+
      | filename.a   | 静态链接库文件，多个.o   | 传给编译器               |
      |              | 文件的打包集合           |                          |
      +--------------+--------------------------+--------------------------+
      | filename.f   |                          |                          |
      | filename.for |                          |                          |
      | filename.ftn | 固定格式的Fortran源文件  | 被Fortran编译器编译      |
      | filename.i   |                          |                          |
      +--------------+--------------------------+--------------------------+
      | filename.fpp |                          |                          |
      | filename.FPP |                          |                          |
      | filename.F   | 固定格式的Fortran源文件  | 自动被Fortran编译器      |
      | filename.FOR |                          | 预处理后再被编译         |
      | filename.FTN |                          |                          |
      +--------------+--------------------------+--------------------------+
      | filename.f90 |                          |                          |
      | filename.i90 | 自由格式的Fortran源文件  | 被Fortran编译器编译      |
      +--------------+--------------------------+--------------------------+
      | filename.F90 | 自由格式的Fortran源文件  | 自动被Fortran编译器      |
      |              |                          | 预处后再被编译           |
      +--------------+--------------------------+--------------------------+
      | filename.s   | 汇编文件                 | 传递给汇编器             |
      +--------------+--------------------------+--------------------------+
      | filename.so  | 动态链接库文             | 传递给链接器             |
      |              | 件，多个.o文件的打包集合 |                          |
      +--------------+--------------------------+--------------------------+
      | filename.o   | 目标文件                 | 传递给链接器             |
      +--------------+--------------------------+--------------------------+

编译器默认将输出按照文件类型与后缀相对应，见下表。

.. container::
   :name: fo

   .. table:: 输出文件后缀与类型的关系

      +--------------+----------------------+--------------------------+
      | 文件名       | 解释                 | 生成方式                 |
      +==============+======================+==========================+
      | filename.o   | 目标文件             | 编译时添加-c选项生成     |
      +--------------+----------------------+--------------------------+
      | filename.so  | 共享库文件           | 编译时指定为共享型，     |
      |              |                      | 如添加-shared，并不含-c  |
      +--------------+----------------------+--------------------------+
      | filename.mod | 模块文件             | 编译含有                 |
      |              |                      | MODULE声明时的源文件生成 |
      +--------------+----------------------+--------------------------+
      | filename.s   | 汇编文件             | 编译时添加-S选项生成     |
      +--------------+----------------------+--------------------------+
      | a.out        | 默认生成的可执行文件 | 编译时没有指定-c时生成   |
      +--------------+----------------------+--------------------------+

串行Fortran程序编译举例
^^^^^^^^^^^^^^^^^^^^^^^

-  Intel Fortran编译器：

   -  | 将Fortran 77程序yourprog.for编译为可执行文件yourprog：
      | ``ifort -o yourprog yourprog.for``

   -  | 将Fortran 90程序yourprog.f90编译为可执行文件yourprog：
      | ``ifort -o yourprog yourprog.f90``

   -  | 将Fortran
        90程序yourprog.90编译为对象文件yourprog.o而不是可执行文件：
      | ``ifort -c yourprog.f90``

   -  | 将Fortran程序yourprog.f90编译为汇编文件yourprog.s而不是可执行文件：
      | ``ifort -S yourprog.f90``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``ifort -g yourprog.f90 -o yourprog``

   -  | 指定头文件路径编译：
      | ``ifort -I/alt/include -o yourprog yourprog.f90``

   -  | 指定库文件路径及库名编译：
      | ``ifort -L/alt/lib -lxyz -o yourprog yourprog.f90``

-  PGI Fortran编译器：

   -  | 将Fortran 77程序yourprog.for编译为可执行文件yourprog：
      | ``pgf77 -o yourprog yourprog.for``

   -  | 将Fortran 90程序yourprog.f90编译为可执行文件yourprog：
      | ``pgf90 -o yourprog yourprog.f90``

   -  | 将Fortran程序yourprog.f90编译为对象文件yourprog.o而不是可执行文件：
      | ``pgf90 -c yourprog.f90``

   -  | 将Fortran程序yourprog.f90编译为汇编文件yourprog.s而不是可执行文件：
      | ``pgf90 -S yourprog.f90``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``pgf90 -g yourprog.f90 -o yourprog``

   -  | 指定头文件路径编译：
      | ``pgf90 -I/alt/include -o yourprog yourprog.f90``

   -  | 指定库文件路径及库名编译：
      | ``pgf90 -L/alt/lib -lxyz -o yourprog yourprog.f90``

-  GNU Fortran编译器：

   -  将Fortran 77程序yourprog.for编译为可执行文件yourprog：

      -  gcc 4.x系列：\ ``gfortran -o yourprog yourprog.for``

      -  gcc 3.x系列：\ ``g77 -o yourprog yourprog.for``

   -  | 将Fortran 90程序yourprog.f90编译为可执行文件yourprog：
      | ``gfortran -o yourprog yourprog.f90``

   -  | 将Fortran程序yourprog.f90编译为对象文件yourprog.o而不是可执行文件：
      | ``gfortran -c yourprog.f90``

   -  | 将Fortran程序yourprog.f90编译为汇编文件yourprog.s而不是可执行文件：
      | ``gfortran -S yourprog.f90``

   -  | 生成带有调试信息的可执行文件以用于调试：
      | ``gfortran -g yourprog.f90 -o yourprog``

   -  | 指定头文件路径编译：
      | ``gfortran -I/alt/include -o yourprog yourprog.f90``

   -  | 指定库文件路径及库名编译：
      | ``gfortran -L/alt/lib -lxyz -o yourprog yourprog.f90``

注意：\ ``g77``\ 既不支持OpenMP，也不支持Fortran 90及之后的标准。

OpenMP程序的编译与运行
~~~~~~~~~~~~~~~~~~~~~~

OpenMP程序的编译
^^^^^^^^^^^^^^^^

Intel、PGI和GNU编译器都支持OpenMP并行，只需利用相关编译命令结合必要的OpenMP编译选项编译即可。对应此三种编译器的OpenMP编译选项：

-  Intel编译器：-qopenmp（2015及之后版）

-  PGI编译器：-mp

-  GNU编译器：-fopenmp

采用这三种编译器的OpenMP源程序编译例子如下：

-  Intel编译器：

   -  | 将C程序yourprog-omp.c编译为可执行文件yourprog-omp：
      | ``icc -qopenmp -o yourprog-omp yourprog.c``

   -  | 将Fortran 90程序yourprog-omp.f90编译为可执行文件yourprog-omp：
      | ``ifort -qopenmp -o yourprog-omp yourprog.f90``

-  PGI编译器：

   -  | 将C程序yourprog-omp.c编译为可执行文件yourprog-omp：
      | ``pgcc -mp -o yourprog-omp yourprog.c``

   -  | 将Fortran 90程序yourprog-omp.f90编译为可执行文件yourprog-omp：
      | ``pgf90 -mp -o yourprog-omp yourprog.f90``

-  GNU编译器：

   -  | 将C程序yourprog-omp.c编译为可执行文件yourprog-omp：
      | ``gcc -fopenmp -o yourprog-omp yourprog.c``

   -  | 将Fortran 90程序yourprog-omp.f90编译为可执行文件yourprog-omp：
      | ``gfortran -fopenmp -o yourprog-omp yourprog.f90``

OpenMP程序的运行
^^^^^^^^^^^^^^^^

OpenMP程序的运行一般是通过在运行前设置环境变量\ ``OMP_NUM_THREADS``\ 来控制线程数，比如在BASH中利用\ ``export OMP_NUM_THREADS=40``\ 设置使用40个线程运行。

注意，本系统为节点内共享内存节点间分布式内存的架构，因此只能在一个节点上的CPU之间运行同一个OpenMP程序作业，在提交作业时需要使用相应选项以保证在同一个节点运行。

.. [#pgi]
   目前尚未配置，等以后配置上
