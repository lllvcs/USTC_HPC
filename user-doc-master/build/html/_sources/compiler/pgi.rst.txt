PGI C/C++ Fortran编译器
~~~~~~~~~~~~~~~~~~~~~~~

PGI C/C++ Fortran编译器简介
^^^^^^^^^^^^^^^^^^^^^^^^^^^

`PGI C/C++ Fortran编译器 <http://www.pgroup.com/products/pgiworkstation.htm>`__\ 是一种针对多种CPU与操作系统的高性能编译器，可用于开发复杂且要进行大量计算的程序。当前安装的版本为2016.7和2014.10，分别安装在、。安装在，可用\ ``module avail``\ 查看，用\ ``moudle load 模块名``\ 使用，或在自己的之类环境设置文件中添加以下代码设置：

.. code:: bash

    PATH=/opt/pgi/linux86-64/14.10/bin:$PATH
    MANPATH=$MANPATH:/opt/pgi/linux86-64/14.10/man
    export PATH MANPATH

PGI编译器编译C、C++、Fortran 77源程序的命令分别为\ ``pgcc``\ 、\ ``pgCC|pgc++``\  [1]_和\ ``pgf77``\ ，编译Fortran 90（为了描述方便，本手册中将Fortran 90、95、2003、2008标准统称为Fortran 90）的源程序的命令有\ ``pgf90``\ 、\ ``pgf901``\ 、\ ``pgf902``\ 、\ ``pgf90_ex``\ 、\ ``pgf95``\ 和\ ``pgfortran``\ 。

========= ===================================== =======================
编译工具  语言或函数                            命令
========= ===================================== =======================
PGF77     ANSI FORTRAN 77                       pgf77
PGFORTRAN ISO/ANSI Fortran 2003                 pgfortran、pgf90、pgf95
PGCC      ISO/ANSI C11 and K&R C                pgcc
PGC++     ISO/ANSI C++14 with GNU compatibility pgc++
PGDBG     Source code debugger                  pgdbg
PGPROF    Performance profiler                  pgprof
========= ===================================== =======================

官方手册目录：。

.. _编译错误-1:

编译错误
^^^^^^^^

编译时的出错信息类似以下：

::

    PGF90-S-0034-Syntax error at or near * (NOlihm.f90: 2)

编译错误的格式为：

大写的编译命令-严重级别-错误编号-解释，含指明位置(文件名: 行号)

错误严重级别分为：

-  I：信息

-  W：警告

-  S：严重

-  F：致命

-  V：其它

Fortran程序编译错误解释，参见：\ `PGI Compiler Reference Guide <http://scc.ustc.edu.cn/zlsc/tc4600/pgi/pgi14ref.pdf>`__->Chapter
9. MESSAGES->9.3. Fortran Compiler Error Messages->9.3.2. Message List

.. _fortran程序运行错误-1:

Fortran程序运行错误
^^^^^^^^^^^^^^^^^^^

Fortran程序运行时错误解释，参见：\ `PGI Compiler Reference Guide <http://scc.ustc.edu.cn/zlsc/tc4600/pgi/pgi14ref.pdf>`__->Chapter 9. MESSAGES->9.4. Fortran Run-time Error Messages->9.4.2. Message List

PGI C/C++编译器重要编译选项
^^^^^^^^^^^^^^^^^^^^^^^^^^^

PGI编译器选项非常多，下面仅仅是列出一些本人认为常用的关于编译C程序的\ ``pgcc``\ 命令的重要选项。编译C++程序的\ ``pgc++|pgCC``\ 命令有稍微不同，建议仔细查看PGI相关资料。建议仔细查看编译器手册中关于程序优化的部分，多加测试，选择适合自己程序的编译选项以提高性能。

一般选项
''''''''

-  -#：显示编译器、汇编器、链接器的调用信息。

-  -c：仅编译成对象文件（.o文件）。

-  -defaultoptions和-nodefaultoptions：是否使用默认选项，默认为使用。

-  -flags：显示所有可用的编译选项。

-  -help\ *[=option]*\ ：显示帮助信息，optio\ *n*\ 可以为groups、asm、debug、language、linker、opt、other、overall、phase、phase、prepro、suffix、switch、target和variable。

-  -Minform\ *=level*\ ：控制编译时错误信息的显示级别。level可以为fatal、file、severe、warn、inform，默认为-Minform=warn。

-  -noswitcherror：显示警告信息后，忽略未知命令行参数并继续进行编译。默认显示错误信息并且终止编译。

-  -o file：指定生成的文件名。

-  -show：显示现有pgcc命令的配置信息。

-  -silent：不显示警告信息，与-Minform=severe等同。

-  -v：详细模式，在每个命令执行前显示其命令行。

-  -V：显示编译器版本信息。

-  -w：编译时不显示任何警告，只显示错误。

.. _优化选项-1:

优化选项
''''''''

-  -fast：编译时选择针对目标平台的普通优化选项。用\ ``pgcc -fast -help``\ 可以查看等价的开关。优化级别至少为O2，参看-O选项。

-  -fastsse：对支持SSE和SSE2指令的CPU（如Intel Xeon CPU）编译时选择针对目标平台的优化选项。用\ ``pgcc -fastsse -help``\ 可以查看等价的开关，优化级别至少为O2，参看-O选项。

-  -fpic或-fPIC：编译器生成地址无关代码，以便可用于生成共享对象文件（动态链接库）。

-  -Kpic或-KPIC：与-fpic或-fPIC相同，为了与其余编译器兼容。

-  -Minfo\ *[=option[,option,…]]*\ ：显示有用信息到标准错误输出，选项可为all、autoinline、inline、ipa、loop或opt、mp、time或stat。

-  -Mipa\ *[=option[,option,…]]*\ 和-Mnoipa：启用指定选项的过程间分析优化，默认为-Mnoipa。

-  -Mneginfo\ *=option[,option…]*\ ：使编译器显示为什么特定优化没有实现的信息。选项包括concur、loop和all。

-  -Mnoopenmp：当使用-mp选项时，忽略OpenMP并行指令。

-  -Mnosgimp：当使用-mp选项时，忽略SGI并行指令。

-  -Mpfi：生成概要导向工具，此时将会包含特殊代码收集运行时的统计信息以用于子序列编译。-Mpfi必须在链接时也得使用。当程序运行时，会生成概要导向文件pgfi.out。

-  -Mpfo：启用概要导向优化，此时必须在当前目录下有概要文件pgfi.out。

-  -Mprof\ *[=option[,option,…]]*\ ：设置性能功能概要选项。此选项可使得结果执行生成性能概要，以便PGPROF性能概要器分析。

-  -mp\ *[=option]*\ ：打开对源程序中的OpenMP并行指令的支持。

-  -O\ *[level]*\ ：设置优化级别。level可设为0、1、2、3、4，其中4与3相同。

-  -pg：使用gprof风格的基于抽样的概要刨析。

调试选项
''''''''

-  -g：包含调试信息。

.. _预处理选项-1:

预处理选项
''''''''''

-  -C：预处理时保留C源文件中的注释。

-  -D\ *name[=def]*\ ：预处理时定义宏name为def。

-  -dD：打印源文件中已定义的宏及其值到标准输出。

-  -dI：打印预处理中包含的所有文件信息，含文件名和定义时的行号。

-  -dM：打印预处理时源文件已定义的宏及其值，含定义时的文件名和行号。

-  -dN：与-dD类似，但只打印源文件已定义的宏，而不打印宏值。

-  -E：预处理每个.c文件，将结果发送给标准输出，但不进行编译、汇编或链接等操作。

-  -I\ *dir*\ ：指明头文件的搜索路径。

-  -M：打印make的依赖关系到标准输出。

-  -MD：打印make的依赖关系到文件file.d，其中file是编译文件的根名字。

-  -MM：打印make的依赖关系到标准输出，但忽略系统头文件。

-  -MMD：打印make的依赖关系到文件file.d，其中file是编译的文件的根名字，但忽略系统头文件。

-  -P：预处理每个文件，并保留每个file.c文件预处理后的结果到file.i。

-  -U\ *name*\ ：去除预处理中的任何name的初始定义。

链接选项
''''''''

-  -Bdynamic：在运行时动态链接所需的库。

-  -Bstatic：静态链接所需的库。

-  -Bstatic_pgi ：动态链接系统库时静态链接PGI库。

-  -g77libs：允许链接GNU ``g77``\ 或\ ``gcc``\ 命令生成的库。

-  -l\ *string*\ ：指明所需链接的库名。如库为libxyz.a，则可用-lxyz指定。

-  -L\ *dir*\ ：指明库的搜索路径。

-  -m：显示链接拓扑。

-  -Mrpath和-Mnorpath：默认为-rpath，以给出包含PGI共享对象的路径。用-Mnorpath可以去除此路径。

-  -pgf77libs：链接时添加pgf77运行库，以允许混合编程。

-  -r：生成可以重新链接的对象文件。

-  -R\ *directory*\ ：对共享对象文件总搜索directory目录。

-  -pgf90libs：链接时添加pgf90运行库，以允许混合编程。

-  -shared：生成共享对象而不是可执行文件，必须在编译每个对象文件时使用-fpic选项。

-  -soname\ *name*\ ：生成共享对象时，用内在的DT_SONAME代替指定的name。

-  -u\ *name*\ ：传递给链接器，以生成未定义的引用。

.. _cc语言选项-1:

C/C++语言选项
'''''''''''''

-  -B：源文件中允许C++风格的注释，指的是以//开始到行尾内容为注释。除非指定-C选项，否则这些注释被去除。

-  -c8x或-c89：对C源文件采用C89标准。

-  -c9x或-c99：对C源文件采用C99标准。

.. _fortran语言选项-1:

Fortran语言选项
'''''''''''''''

-  -byteswapio或-Mbyteswapio：对无格式Fortran数据文件在输入输出时从大端（big-endian）到小端（little-endian）交换比特，或者相反。此选项可以用于读写Sun或SGI等系统中的无格式的Fortran数据文件。

-  -i2：将INTEGER变量按照2比特处理。

-  -i4：将INTEGER变量按照4比特处理。

-  -i8：将默认的INTEGER和LOGICAL变量按照4比特处理。

-  -i8storage：对INTEGER和LOGICAL变量分配8比特。

-  -Mallocatable\ *[=95|03]*\ ：按照Fortran 95或2003标准分配数组。

-  -Mbackslash和-Mnobackslash：将反斜线(\)当作正常字符（非转义符）处理，默认为-Mnobackslash。-Mnobackslash导致标准的C反斜线转义序列在引号包含的字串中重新解析。-Mbackslash则导致反斜线被认为和其它字符一样。

-  -Mextend：设置源代码的行宽为132列。

-  -Mfixed、-Mnofree和-Mnofreeform：强制对源文件按照固定格式进行语法分析，默认.f或.F文件被认为固定格式。

-  -Mfree和-Mfreeform：强制对源文件按照自由格式进行语法分析，默认.f90、.F90、.f95或.F95文件被认为自由格式。

-  -Mi4和-Mnoi4：将INTEGER看作INTEGER*4。-Mnoi4将INTEGER看作INTEGER*2。

-  -Mnomain：当链接时，不包含调用Fortran主程序的对象文件。

-  -Mr8和-Mnor8：将REAL看作DOUBLE PRECISION，将实(REAL)常数看作双精度(DOUBLE PRECISION)常数。默认为否。

-  -Mr8intrinsics *[=float]*\ 和-Mnor8intrinsics：将CMPLX看作DCMPLX，将REAL看作DBLE。添加float选项时，将FLOAT看作DBLE。

-  -Msave和-Mnosave：是否将所有局部变量添加SAVE声明，默认为否。

-  -Mupcase和-Mnoupcase：是否保留名字的大小写。-Mnoupcase导致所有名字转换成小写。注意，如果使用-Mupcase，那么变量名X与变量名x不同，并且关键字必须为小写。

-  -Mcray\ *=pointer*\ ：支持Cray指针扩展。

-  -module *directory*\ ：指定编译时保存生成的模块文件的目录。

-  -r4：将DOUBLE PRECISION变量看作REAL。

-  -r8：将REAL变量看作DOUBLE PRECISION。

平台相关选项
''''''''''''

-  -Kieee和-Knoieee：浮点操作是否严格按照IEEE
   754标准。使用-Kieee时一些优化处理将被禁止，并且使用更精确的数值库。默认为-Knoieee，将使用更快的但精确性低的方式。

-  -Ktrap=\ *[option,[option]…]*\ ：控制异常发生时CPU的操作。选项可为divz、fp、align、denorm、inexact、inv、none、ovf、unf，默认为none。

-  -Msecond_underscore和-Mnosecond_underscore：是否对已有_的Fortran函数名添加第二个\_。与\ ``g77``\ 编译命令兼容时使用，因为\ ``g77``\ 默认符号后添加第二个\_ 。

-  -mcmodel\ *=small\ :math:`|`\ medium*\ ：使内存模型是否限制对象小于2GB(small)或允许数据块大于2GB(medium)。medium时暗含-Mlarge_arrays选项。

-  -tp *target*\ ：target可以为haswell等，默认与编译时的平台一致。

