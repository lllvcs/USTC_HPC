.. _module:

设置编译及运行环境
==================

本超算系统安装了多种编译环境及应用等，为方便用户使用，采用\ `Environment Modules <http://modules.sourceforge.net/>`__\ 工具对其进行了封装，用户可以利用\ ``module``\ 命令设置、查看所需要的环境等。编译和运行程序时在命令行可用\ ``module load modulefile``\ 加载对应的模块（仅对该次登录生效），如不想每次都手动设置，可将其设置在 或 文件中：

-  ``~/.bashrc`` ，只Bash启动时设置：

.. code:: bash

   module load intel/2020

-  ``~/.modulerc`` ，每次module命令启动时都设置：

.. code :: bash

   #%Module1.0
   module load intel/2020

注意第一行#%Module1.0是必需的。

module基本语法为：\ ``module [ switches ] [ sub-command ] [ sub-command-args ]``

常用开关参数(switches)：

-  -\-help, -H：显示帮助。

-  -\-force, -f：强制激活依赖解决。

-  -\-terse, -t：以短格式显示。

-  -\-long, -l：以长格式显示。

-  -\-human, -h：以易读方式显示。

-  -\-verbose, -v：显示module命令执行时的详细信息。

-  -\-silent, -s：静默模式，不显示出错信息等。

-  -\-icase, -i：搜索时不区分大小写。

常用子命令(sub-command)：

-  ``avail [path...]``\ ：显示MODULEPATH环境变量中设置的目录中的某个目录下可用的模块，如有参数指定，则显示MODULEPATH中符合这个参数的路径。如\ ``module avail``\ ：

::

   ------------------------ /opt/Modules/app -----------------------
   gaussian/g16.C01 vasp/5.4.4/intel-2020 vasp/5.4.4/vtst/intel-2020
   matlab/2019b vasp/5.4.4/intelmpimkl2018u4
   vasp/5.4.4/hpcx-intel-2019.update5-novtst
   vasp/5.4.4/vtst/hpcx-intel-2019.update5

   ------------------------ /opt/Modules/compiler -------------------
   cuda/10.2.89 gcc/7.5.0 gcc/8.3.0 gcc/9.2.0 intel/2018.update4
   intel/2019.update5 intel/2020

   ------------------------- /opt/Modules/lib -----------------------
   mkl/2018.update4 mkl/2019.update5 mkl/2020

   ------------------------- /opt/Modules/mpi ----------------------- 
   hpcx/hpcx hpcx/hpcx-mt hpcx/hpcx-prof-ompi intelmpi/2020
   openmpi/4.0.2/intel/2020 hpcx/hpcx-debug hpcx/hpcx-mt-ompi
   hpcx/hpcx-stack openmpi/3.0.5/gcc/9.2.0 hpcx/hpcx-debug-ompi
   hpcx/hpcx-ompi intelmpi/2018.update4 openmpi/3.0.5/intel/2020
   hpcx/hpcx-intel-2019.update5 hpcx/hpcx-prof intelmpi/2019.update5
   openmpi/4.0.2/gcc/9.2.0

   ------------------------ /opt/Modules/python --------------------- 
   anaconda3 python/3.8.1

上面输出：

   -  /opt/Modules/mpi：模块所在的目录，由\ ``MODULEPATH``\ 环境变量中设定。

   -  openmpi/3.0.5/intel/2020：模块名或模块文件modulefile，此表示此为3.0.5版本Open
      MPI，而且是采用2020版本Intel编译器编译的。

-  ``help [modulefile...]``\ ：显示每个子命令的用法，如给定modulefile参数，则显示modulefile中的帮助信息。

-  ``add|load modulefile...``\ ：加载modulefile中设定的环境，如\ ``module load openmpi/3.0.5/intel/2020``\ 。

-  ``rm|unload modulefile...``\ ：卸载已加载的环境modulefile，如\ ``module unload openmpi/3.0.5/intel/2020``\ 。

-  ``swap|switch [modulefile1] modulefile2``\ ：用modulefile2替换当前已加载的modulefile1，如modulefile1没指定，则交换与modulefile2同样根目录下的当前已加载modulefile。

-  | ``show|display  modulefile...``\ ：显示modulefile环境变量信息。如
   | ``module show openmpi/3.0.5/intel/2020``

   上面输出：

   -  第一行是modulefile具体路径。

   -  module-whatis：模块说明，后面可用子命令whatis、apropos、keyword等显示或搜索。

   -  module load：表示自动加载的模块。

   -  prepend-path：表示将对应目录加到对应环境变量的前面。

-  ``clear``\ ：强制module软件相信当前没有加载任何modulefiles。

-  ``purge``\ ：卸载所有加载的modulefiles。

-  ``refresh``\ ：强制刷新所有当前加载的不安定的组件。一般用于aliases需要重新初始化，但环境比那两已经被当前加载的模块设置了的派生shell中。

-  ``whatis [modulefile...]``\ ：显示modulefile中module-whatis命令指明的关于此modulefile的说明，如果没有指定modulefile，则显示所有modulefile的。

-  ``apropos|keyword string``\ ：在modulefile中module-whatis命令指明的关于此modulefile的说明中搜索关键字，显示符合的modulefile。

其它一些不常用命令及参数，请\ ``man module``\ 查看。

用户也可自己生成自己所需要的modulefile文件，用\ ``MODULEFILE``\ 和环境变量来指该modulefile文件所在目录，用\ ``module``\ 来设置，具体请\ ``man module``\ 及\ ``man modulefile``\ 。
