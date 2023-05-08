前言
~~~~

本用户使用指南主要将对相关指示做一基本介绍，详细信息请参看相应的文档。

为了便于查看，主要排版约定如下：

-  文件名：\ `/path/file`

-  环境变量： *MKLROOT*

-  命令：\ ``command parameters``

-  脚本文件内容或长命令：

.. code:: bash

   export OPENMPI=/opt/openmpi/1.8.2_intel-compiler-2015.1.133
   export PATH=$OPENMPI/bin:$PATH
   export MANPATH=$MANPATH:$OPENMPI/share/man

-  命令输出：

::

 QUEUE_NAME      PRIO STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN  SUSP
 serial           50  Open:Active       -   16    -    -     0     0     0     0
 long             40  Open:Active       -    -    -    -     0     0     0     0
 normal           30  Open:Active       -    -    -    -     0     0     0     0

由于受水平和时间所限，错误和不妥之处在所难免，欢迎指出错误和改进意见，我们将尽力完善。
