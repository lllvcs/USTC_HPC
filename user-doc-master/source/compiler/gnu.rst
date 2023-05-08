GNU C/C++ Fortran编译器
=======================

GNU C/C++ Fortran编译器简介
^^^^^^^^^^^^^^^^^^^^^^^^^^^

`GNU C/C++ Fortran(GCC)编译器 <http://gcc.gnu.org/>`__ 为系统自带的编译器，当前自带的版本为4.8.5，还装有9.2.0等。默认为4.8.5版本，用户无需特殊设置即可使用。GNU编译器编译C、C++源程序的命令分别 ``gcc`` 和 ``g++`` ； ``gfortran`` 可以直接编译Fortran 77、90源程序。

编译错误
^^^^^^^^

C/C++程序编译时错误信息类似：

::

    netlog.c: In function 'main':
    netlog.c:84:7: error: 'for' loop initial declarations are only allowed in C99 mode
    netlog.c:84:7: note: use option -std=c99 or -std=gnu99 to compile your code

编译错误的格式为：

- 源文件名: 函数中
- 源文件名:行数:列数:错误类型:具体说明
- 源文件名:行数:列数:注解:解决办法

Fortran程序编译时错误信息类似：

::

    NOlihm.f90:146.14:

      n2nd=0;  npr=0
               1
    Error: Symbol 'npr' at (1) has no IMPLICIT type

编译错误的格式为：

- 源文件名:行数:列数:
- 源文件代码
- 1指示出错位置
- 错误类型: 具体说明

GNU C/C++编译器GCC重要编译选项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GNU编译器GCC是Linux系统自带的编译器，系统自带的版本为4.8.5，另外还装有9.2.0等，选项非常多，下面仅仅是列出一些针对4.8.5本人认为常用的重要选项，建议仔细看GCC相关资料。

建议仔细查看编译器手册中关于程序优化的部分，多加测试，选择适合自己程序的编译选项以提高性能。

控制文件类型的选项
''''''''''''''''''

-  -x language：明确指定而非让编译器判断输入文件的类型。language可为：

   -  c、c-header、c-cpp-output

   -  c++、c++-header、c++-cpp-output

   -  objective-c、objective-c-header、objective-c-cpp-output

   -  objective-c++、objective-c++-header、objective-c++-cpp-output

   -  assembler、assembler-with-cpp

   -  ada

   -  f95、f95-cpp-input

   -  java

   -  treelang

   当language为none时，禁止任何明确指定的类型，其类型由文件名后缀设定。

-  -c：仅编译成对象文件（.o文件），并不进行链接。

-  -o file：指定生成的文件名。

-  -v：详细模式，显示在每个命令执行前显示其命令行。

-  -###：显示编译器、汇编器、链接器的调用信息但并不进行实际编译，在脚本中可以用于捕获驱动器生成的命令行。

-  –help：显示帮助信息。

-  –target-help：显示目标平台的帮助信息。

-  –version：显示编译器版本信息。

.. _cc语言选项-2:

C/C++语言选项
'''''''''''''

-  -ansi：C模式时，支持所有ISO C90指令。在C++模式时，去除与ISO
   C++冲突的GNU扩展。

-  -std\ *=val*\ ：控制语言标准，可以为c89、iso9899:1990、iso9899:199409、c99、c9x、iso9899:1999、iso9899:199x、gnu89、gnu99、gnu9x、c++98、gnu++98。

-  -B：在源文件中允许C++风格的注释，指的是以//开始到行尾内容为注释。除非指定-C选项，否则这些注释被去除。

-  -c8x或-c89：对C源文件采用C89标准。

-  -c9x或-c99：对C源文件采用C99标准。

.. _fortran语言选项-2:

Fortran语言选项
'''''''''''''''

-  -fconvert\ *=conversion*\ ：指定对无格式Fortran数据文件表示方式，其值可以为：native，默认值；swap，在输入输出时从大端（big-endian）到小端（little-endian）交换比特，或者相反；big-endian，用大端方式读写；little-endian，用小端方式读写。

-  -ff2c：与\ ``g77``\ 和\ ``f2c``\ 命令生成的代码兼容。

-  -ffree-form和-ffixed-form：声明源文件是自由格式还是固定格式，默认从Fortran
   90起的源文件为自由格式，之前的Fortran 77等的源文件为固定格式。

-  -fdefault-double-8：设置DOUBLE PRECISION类型为8比特。

-  -fdefault-integer-8：设置INTEGER和LOGICAL类型为8比特。

-  -fdefault-real-8：设置REAL类型为8比特。

-  -fno-backslash：将反斜线(\)当作正常字符（非转义符）处理。

-  -fno-underscoring：不在名字后添加\_。注意：\ ``gfortran``\ 默认行为与\ ``g77``\ 和\ ``f2c``\ 不兼容，为了兼容需要加-ff2c选项。除非使用者了解与现有系统环境的集成，否则不建议使用-fno-underscoring选项。

-  -ffixed-line-length-*n*\ ：设置固定格式源代码的行宽为n。

-  -ffree-line-length-*n*\ ：设置自由格式源代码的行宽为n。

-  -fimplicit-none：禁止变量的隐式声明，所有变量都需要显式声明。

-  -fmax-identifier-length=\ *n*\ ：设置名称的最大字符长度为n，Fortran 95和200x的长度分别为31和65。

-  -fno-automatic：将每个程序单元的本地变量和数组声明具有SAVE属性。

-  -fcray-pointer：支持Cray指针扩展。

-  -fopenmp：编译OpenMP并行程序。

-  -M\ *dir*\ 和-J\ *dir*\ ：指定编译时保存生成的模块文件目录。

-  -fsecond-underscore：默认\ ``gfortran``\ 对外部函数名添加一个\_，如果使用此选项，那么将添加两个\_。此选项当使用-fno-underscoring选项时无效。此选项当使用-ff2c时默认启用。

-  -std\ *=val*\ ：指明Fortran标准，val可以为f95、f2003、legacy。

-  -funderscoring：对外部函数名没有\_的加\_，以与一些Fortran编译器兼容。

警告选项
''''''''

-  -fsyntax-only：仅仅检查代码的语法错误，并不进行其它操作。

-  -w：编译时不显示任何警告，只显示错误。

-  -Wfatal-errors：遇到第一个错误就停止，而不尝试继续运行显示更多错误信息。

.. _调试选项-1:

调试选项
''''''''

-  -g：包含调试信息。

-  -ggdb：包含利用gbd调试时所需要的信息。

.. _优化选项-2:

优化选项
''''''''

-  -O\ *[level]*\ ：设置优化级别。优化级别level可以设置为0、1、2、3、s。

.. _预处理选项-2:

预处理选项
''''''''''

-  -C：预处理时保留C源文件中的注释。

-  -D *name*\ ：预处理时定义宏name的值为1。

-  -D *name=def*\ ：预处理时定义name为def。

-  -U *name*\ ：预处理时去除的任何name初始定义。

-  -undef：不预定义系统或GCC声明的宏，但标准预定义的宏仍旧被定义。

-  -dD：显示源文件中定义的宏及其值到标准输出。

-  -dI：显示预处理中包含的所有文件，包括文件名和定义时的行号信息。

-  -dM：显示预处理时源文件中定义的宏及其值，包括定义时文件名和行号。

-  -dN：与-dD类似，但只显示源文件中定义的宏，而不显示宏值。

-  -E：预处理各.c文件，将结果发给标准输出，不进行编译、汇编或链接。

-  -I\ *dir*\ ：指明头文件的搜索路径。

-  -M：打印make的依赖关系到标准输出。

-  -MD：打印make的依赖关系到文件file.d，其中file是编译文件的根名字。

-  -MM：打印make的依赖关系到标准输出，但忽略系统头文件。

-  -MMD：打印make的依赖关系到文件file.d，其中file是编译的文件的根名字，但忽略系统头文件。

-  -P：预处理每个文件，并保留每个file.c文件预处理后的结果到file.i。

.. _链接选项-1:

链接选项
''''''''

-  -pie：在支持的目标上生成地址无关的可执行文件。

-  -s：从可执行文件中去除所有符号表。

-  -rdynamic：添加所有符号表到动态符号表中。

-  -static：静态链接所需的库。

-  -shared：生成共享对象文件而不是可执行文件，必须在编译每个对象文件时使用-fpic选项。

-  -shared-libgcc：使用共享libgcc库。

-  -static-libgcc：使用静态libgcc库。

-  -u *symbol*\ ：确保符号symbol未定义，强制链接一个库模块来定义它。

-  -I\ *dir*\ ：指明头文件的搜索路径。

-  -l\ *string*\ ：指明所需链接的库名，如库为libxyz.a，则可用-lxyz指定。

-  -L\ *dir*\ ：指明库的搜索路径。

-  -B\ *dir*\ ：设置寻找可执行文件、库、头文件、数据文件等路径。

i386和x86-64平台相关选项
''''''''''''''''''''''''

-  -mtune\ *=cpu-type*\ ：设置优化针对的CPU类型，可为：generic、core2、opteron、opteron-sse3、bdver1、bdver2等，bdver1为针对本系统AMD
   Opteron CPU的。

-  -march\ *=cpu-type*\ ：设置指令针对的CPU类型，CPU类型与上行中一样。

-  -mieee-fp和-mno-ieee-fp：浮点操作是否严格按照IEEE标准。

约定成俗选项
''''''''''''

-  -fpic：生成地址无关的代码以用于共享库。

-  -fPIC：如果目标机器支持，将生成地址无关的代码。

-  -fopenmp：编译OpenMP并行程序。

-  -fpie和-fPIE：与-fpic和-fPIC类似，但生成的地址无关代码，只能链接到可执行文件中。
