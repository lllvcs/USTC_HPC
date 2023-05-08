Intel C/C++ Fortran编译器
=========================

Intel C/C++ Fortran编译器简介
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Intel Parallel Studio XE Cluster <https://software.intel.com/en-us/parallel-studio-xe>`__\ 版C/C++ Fortran编译器，是一种主要针对Inetl平台的高性能编译器，可用于开发复杂且要进行大量计算的C/C++、Fortran程序。

系统当前安装目录为，其下有多种年份版本。官方手册目录为 ``/opt/intel`` 。用户可以采用molule命令来设置所需的环境，请参看 `[设置编译及运行环境] <../module-environment/module-environment.html#module>`__\ 。

Intel编译器编译C和C++源程序的编译命令分别为\ ``icc``\ 和\ ``icpc``\ ；编译Fortran源程序的命令为\ ``ifort``\ 。\ ``icpc``\ 命令使用与\ ``icc``\ 命令相同的编译器选项，利用\ ``icpc``\ 编译时将后缀为.c和.i的文件看作为C++文件；而利用\ ``icc``\ 编译时将后缀为.c和.i的文件则看作为C文件。用\ ``icpc``\ 编译时，总会链接C++库；而用\ ``icc``\ 编译时，只有在编译命令行中包含C++源文件时才链接C++库。

编译命令格式为：\ ``command [options] [@response_file] file1 [file2...]``\ ，其中\ ``response_file``\ 为文件名，此文件包含一些编译选项，请注意调用时前面有个@。

在Intel数学库(Intelmath)中的许多函数针对Intel微处理器相比针对非Intel微处理器做了非常大的优化处理。

为了使用Intel数学库中的函数，需要在程序源文件中包含头文件，例如使用实函数：

.. code:: C

    // real_math.c
    #include <stdio.h>
    #include <mathimf.h>

    int main() {
        float fp32bits;
        double fp64bits;
        long double fp80bits;
        long double pi_by_four = 3.141592653589793238/4.0;

        // pi/4 radians is about 45 degrees
        fp32bits = (float) pi_by_four; // float approximation to pi/4
        fp64bits = (double) pi_by_four; // double approximation to pi/4
        fp80bits = pi_by_four; // long double (extended) approximation to pi/4

        // The sin(pi/4) is known to be 1/sqrt(2) or approximately .7071067
        printf("When x = %8.8f, sinf(x) = %8.8f \n", fp32bits, sinf(fp32bits));
        printf("When x = %16.16f, sin(x) = %16.16f \n", fp64bits, sin(fp64bits));
        printf("When x = %20.20Lf, sinl(x) = %20.20f \n", fp80bits, sinl(fp80bits));

        return 0;
    }

编译：\ ``icc real_math.c``

编译错误
^^^^^^^^

C/C++程序编译时的出错信息类似以下：

::

    netlog.c(140): error: identifier "hhh" is undefined
                        for(int hhh=domain_cnt+1;hhh>TMP;hhh--){
                                                 ^
    netlog.c(156): error: expected an expression
    for(int i=0;i<32;i++)for(int j=0;j<256;j++)if(ip1[i][j]!=0)fprintf(fin);
        ^

Fortran程序编译时的出错信息类似以下：

::

    NOlihm.f90(146): error #6404: This name does not have a type, and must 
    have an explicit type.   [NPR]
      n2nd=0;  npr=0
    -----------^
    NOlihm.f90(542): remark #8290: Recommended relationship between field width
    'W' and the number of fractional digits 'D' in this edit descriptor is 'W>=D+3'.
    6060 format(/i2,'-th layer',i2,'-th element: z=',i3,' a=',f9.5/' Ef=',f7.5)
    -----------------------------------------------------------------------^

编译错误的格式为：

-  源文件名(行数): 错误类型:具体说明

-  源代码，^指示出错位置

错误类型可以为：

-  Warning：警告，报告对编译有效但也许存在问题的语法，请根据信息及程序本身判断，不一定需要处理。

-  Error：存在语法或语义问题，必须要处理。

-  Fatal Error：报告环境错误，如磁盘空间没有了。

Fortran程序运行错误
^^^^^^^^^^^^^^^^^^^

根据运行时错误代码可以在\ `官方手册 <https://software.intel.com/en-us/fortran-compiler-developer-guide-and-reference-list-of-run-time-error-messages>`__\ 中查找对应错误解释。

Intel Parallel Studio XE版重要编译选项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Intel编译器选项分为几类，可以用\ ``icc -help 类别``\ 查看对应的选项，类别与选项对应关系如下：

advanced
   - Advanced Optimizations，高级优化

codegen
   - Code Generation，代码生成

compatibility
   - Compatibility，兼容性

component
   - Component Control，组件控制

data
   - Data，数据

deprecated
   - Deprecated Options，过时选项

diagnostics
   - Compiler Diagnostics，编译器诊断

float
   - Floating Point，浮点

help
   - Help，帮助

inline
   - Inlining，内联

ipo
   - Interprocedural Optimization (IPO)，过程间优化

language
   - Language，语言

link
   - Linking/Linker，链接/链接器

misc
   - Miscellaneous，杂项

opt
   - Optimization，优化

output
   - Output，输出

pgo
   - Profile Guided Optimization (PGO)，概要导向优化

preproc
   - Preprocessor，预处理

reports
   - Optimization Reports，优化报告

openmp
   - OpenMP and Parallel Processing，OpenMP和并行处理

可以运行\ ``icc -help help``\ 查看选项分类情况。

优化选项
''''''''

-  -fast：最大化整个程序的速度，相当于设置-ipo、-O3、-no-prec-div、-static、-fp-model fast=2和-xHost。这里是所谓的最大化，还是需要结合程序本身使用合适的选项，默认不使用此选项。

-  -nolib-inline：取消标准库和内在函数的内联展开。

-  -O\ *n*\ ：设定优化级别，默认为O2。O与O2相同，推荐使用；O3为在O2基础之上增加更激进的优化，比如包含循环和内存读取转换和预取等，但在有些情况下速度反而慢，建议在具有大量浮点计算和大数据处理的循环时的程序使用。

-  -Ofast：设定一定的优化选项提高程序性能，设定-O3, -no-prec-div和-fp-model fast=2。在Linux系统上提供与gcc的兼容。

-  -Os：启用优化，但不增加代码大小，并且产生比-O2优化小的代码。它取消了一些优化不明显却增大了代码的优化选项。

代码生成选项
''''''''''''

-  -ax\ *code*\ ：在有性能提高时，生成针对Intel处理器的多特征面向的自动调度代码路径。\ *code*\ 可为：

   -  COMMON-AVX512：生成Intel(R) Advanced Vector Extensions 512 (Intel(R) AVX-512)基础指令。

   -  CORE-AVX2：生成IntelAdvanced Vector Extensions 2 (IntelAVX2)、IntelAVX、SSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  CORE-AVX-I：生成Float-16转换指令和RDRND（随机数）指令、IntelAdvanced Vector Extensions (IntelAVX)、IntelSSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  AVX：生成IntelAdvanced Vector Extensions (IntelAVX)、IntelSSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  SSE4.2：生成IntelSSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  SSE4.1：生成IntelSSE4.1、SSE3、SSE2、SSE和SSSE3指令

   -  SSSE3：生成SSSE3指令和IntelSSE3、SSE2和SSE指令。

   -  SSE3：生成IntelSSE3、SSE2和SSE指令。

   -  SSE2：生成IntelSSE2和SSE指令。

-  -fexceptions、-fno-exceptions：是否生成异常处理表。

-  -x\ *code*\ ：设置启用编译目标的特征，包含采取何种指令集和优化。

   -  COMMON-AVX512

   -  CORE-AVX2

   -  CORE-AVX-I

   -  AVX

   -  SSE4.2

   -  SSE4.1

   -  SSSE3

   -  SSE3

   -  SSE2

-  -m\ *code*\ ：需要生成目标特征的指令集。\ *code*\ 可为：

   -  avx：生成IntelAdvanced Vector Extensions (IntelAVX)、IntelSSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  sse4.2：生成IntelSSE4.2、SSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  sse4.1：生成IntelSSE4.1、SSE3、SSE2、SSE和SSSE3指令。

   -  ssse3：生成SSSE3指令和IntelSSE3、SSE2和SSE指令。

   -  sse3：生成IntelSSE3、SSE2和SSE指令。

   -  sse2：生成IntelSSE2和SSE指令。

   -  sse：已过时，现在与ia32一样。

   -  ia32：生成与IA-32架构兼容的x86/x87通用代码。取消任何默认扩展指令集，任何之前的扩展指令集。并且取消所有面向特征的优化及指令。此值仅在Linux系统上使用IA-32架构时有效。

-  -m32和-m64：生成IA-32或Intel64位代码，默认由主机系统设定。

-  -march=\ *processor*\ ：生成支持某种处理器特定特征的代码。\ *processor*\ 可为：

   -  generic

   -  core-avx2

   -  core-avx-i

-  -mtune=\ *processor*\ ：针对特定处理器优化。\ *processor*\ 可为：

   -  generic（默认）

   -  core-avx2

   -  core-avx-i

-  -xHost：生成编译主机处理器能支持的最高指令集。

过程间优化(IPO)选项
'''''''''''''''''''

-  -ip：在单个文件中进行过程间优化(Interprocedural Optimizations-IPO)。

-  -ip-no-inlining：禁止过程间优化时启用的全部和部分内联。

-  -ip-no-pinlining：禁止过程间优化时启用的部分内联。

-  -ipo\ *[n]*\ 、-no-ipo：是否在多文件中进行过程间优化，非负整数\ *n*\ 为可生成的对象文件数。

-  -ipo-c：在多文件中进行过程间优化，并生成一个对象文件。

-  -ipo-jobs\ *n*\ ：指定在过程间优化的链接阶段时的命令（作业）数。

-  -ipo-S：在多文件中进行过程间优化，并生成一个汇编文件。

-  -ipo-separate：在多文件中进行过程间优化，并为每个文件分别生成一个对象文件。

高级优化选项
''''''''''''

-  -funroll-all-loops：即使在循环次数不确定的情况下也展开所有循环。默认为否。

-  -guide\ *[=n]*\ ：设置自动向量化、自动并行及数据变换的指导级别。\ *n*\ 为1到4，1为标准指导，4为最高指导，如果\ *n*\ 忽略，则默认为4。默认为不启用。

-  -guide-data-trans\ *[=n]*\ ：设置数据变换时的指导级别。\ *n*\ 为1到4，1为标准指导，4为最高指导，如果\ *n*\ 忽略，则默认为4。默认为不启用。

-  -guide-file\ *[=filename]*\ ：将自动并行的结果输出到文件\ *filename*\ 中。

-  -guide-file-append\ *[=filename]*\ ：将自动并行的结果追加到文件\ *filename*\ 中。

-  -guide-par\ *[=n]*\ ：设置自动并行的指导级别。\ *n*\ 为1到4，1为标准指导，4为最高指导，如果\ *n*\ 忽略，则默认为4。默认为不启用。

-  -guide-vec\ *[=n]*\ ：设置自动向量化的指导级别。\ *n*\ 为1到4，1为标准指导，4为最高指导，如果\ *n*\ 忽略，则默认为4。默认为不启用。

-  -mkl\ *[=lib]*\ ：链接时自动链接Intel MKL库，默认为不启用。\ *lib*\ 可以为：

   -  parallel：采用线程化部分的MKL库链接，此为\ *lib*\ 如果没指明时的默认选项。

   -  sequential：采用未线程化的串行MKL库链接。

   -  cluster：采用集群部分和串行部分MKL链接。

-  -simd、-no-simd：是否启用SIMD编译指示的编译器解释。

-  -unroll\ *[=n]*\ ：设置循环展开的最大层级。

-  -unroll-aggressive、-no-unroll-aggressive：设置对某些循环执行激进展开。默认不启用。

-  -vec、-no-vec：是否启用向量化。默认启用。

概要导向优化(PGO)选项
'''''''''''''''''''''

-  -p：使用gprof编译和链接函数。

-  -prof-dir *dir*\ ：设定存储概要导向优化信息的文件目录。

-  -prof-file *filename*\ ：设定概要摘要文件名。

优化报告选项
''''''''''''

-  -qopt-report\ *[=n]*\ ：设定显示优化报告信息的级别，为每个对象文件生成一个对应的文件。\ *n*\ 为0（不显示）到5（最详细）。

-  -qopqopt-report-file\ *=keyword*\ ：设定报告文件名。\ *keyword*\ 可以为：

   -  filename：保存输出的文件名。

   -  stderr：输出到标准错误输出。

   -  stdout：输出到标准输出。

-  -qopt-report-filter\ *=string*\ ：设置报告的过滤器。\ *string*\ 可以为filename、routine、range等。

-  -qopt-report-format\ *=keyword*\ ：设置报告的格式。\ *keyword*\ 可以为text和vs，分别对应纯文本和Visual Studio格式。

-  -qopt-report-help：显示使用-qopt-report-phase选项时可用于报告生成的各优化阶段，并显示各级别报告的简短描述。

-  -qopt-report-per-object：为各对象文件生成独立的报告文件。

-  -qopt-report-phase：对生成的优化报告指明一个或多个优化阶段。\ *phase*\ 可以为：cg、ipo、loop、openmp、par、pgo、tcollect、vec和all等。

-  -qopt-report-routine\ *=substring*\ ：让编译器对含有\ *substring*\ 的子程序生成优化报告。

-  -qopt-report-names\ *=keyword*\ ：是否在优化报告中显示重整的或未重整的名字。\ *keyword*\ 可以为：mangled和unmangled。

-  -tcheck：对线程应用启用分析。

-  -tcollect\ *[lib]*\ ：插入测试探测调用Intel Trace Collector API。\ *lib*\ 为一种Intel Trace Collector库，例如：VT、VTcs、VTmc或VTfs。

-  -tcollect-filter\ *filename*\ ：对特定的函数启用或禁止测试。

-  -vec-report\ *[=n]*\ ：设置向量化诊断信息详细程度。\ *n*\ 为0（不显示）到7（最详细）。

OpenMP和并行处理选项
''''''''''''''''''''

-  -fmpc-privatize、-fno-mpc-privatize：是否启用针对多处理器计算环境(MPC)所有静态数据私有。

-  -par-affinity=\ *[modifier,…]type[,permute][,offset]*\ ：设定线程亲和性。

   -  modifier：可以为以下值之一：granularity=fine|thread|core、[no]respect、[no]verbose、[no]warnings、proclist=proc_list。默认为granularity=core, respect, noverbose。

   -  type：指示线程亲和性。此选项是必需的，并且需为以下之一：compact、disabled、explicit、none、scatter、logical、physical。默认为none。logical和physical已经过时。分别使用compact和scatter，并且没有permute值。

   -  permute：非负整数。当type设置为explicit、none或disabled时，不能使用此选项。默认为0。

   -  offset：非负整数。当type设置为explicit、none或disabled时，不能使用此选项。默认为0。

-  -par-num-threads=\ *n*\ ：设定并行区域内的线程数。

-  -par-report\ *n*\ ：设定自动并行时诊断信息的显示级别。\ *n*\ 可以为0到5。

-  -par-runtime-control\ *n*\ 、-no-par-runtime-control：设定是否对符号循环边界的循环执行运行时检查代码。

-  -par-schedule\ *-keyword[=n]*\ ：设定循环迭代的调度算法。\ *keyword*\ 可以为：

   -  auto：由编译器或者运行时系统设定调度算法。

   -  static：将迭代分割成连续块。

   -  static-balanced：将迭代分割成偶数大小的块。

   -  static-steal：将迭代分割成偶数大小的块，但允许线程从临近线程窃取部分块。

   -  dynamic：动态获取迭代集。

   -  guided：设定迭代的最小值。

   -  guided-analytical：使用指数分布或动态分布分割迭代。

   -  runtime：直到运行时才设定调度分割。

   *n*\ 为每个迭代数或块大小。此设置，只能配合static、dynamic和guided使用。

-  -par-threshold\ *n*\ ：设定针对循环自动并行的阈值。\ *n*\ 为一个在0到100间的整数，限定针对循环自动并行的阈值：

   -  如\ *n*\ 为0，则循环总会被并行。

   -  如\ *n*\ 为100，则循环只有在基于编译器分析应用的数据能达到预期收益时才并行。

   -  1到99为预期可能的循环加速百分比。

-  -parallel：让自动并行器针对可以安全并行执行的循环生成多线程代码。

-  -parallel-source-info\ *=n*\ 、-no-parallel-source-info：设定当生成OpenMP或自动并行代码是否显示源位置。\ *n*\ 为显示级别：

   -  0：禁止显示源位置信息。

   -  1：显示子程序名和行信息。

   -  2：显示路径、文件名、子程序名和行信息。

-  -qopenmp：编译OpenMP程序。注意：在一般只能在同一个节点内的CPU上运行OpenMP程序。

-  -qopenmp-lib\ *=type*\ ：设定链接时使用的OpenMP运行时库。当前\ *type*\ 只能设定为compat。

-  -qopenmp-link\ *=library*\ ：设定采用动态还是静态链接OpenMP运行时库。\ *library*\ 可以为static和dynamic，分别表示静态和动态链接OpenMP运行时库。

-  -qopenmp-report\ *n*\ ：设定OpenMP并行器的诊断信息的显示级别。\ *n*\ 可以为0、1和2。

-  -qopenmp-simd、-no-qopenmp-simd：设定是否启用OpenMP SIMD编译。

-  -qopenmp-stubs：使用串行模式编译OpenMP程序。

-  -qopenmp-task\ *=model*\ ：设定OpenMP的任务模型。\ *model*\ 可以为：

   -  intel：让编译接受Intel任务序列指导指令（#pragma intel_omp_taskq和#pragma intel_omp_task）。OpenMP API 3.0将被忽略。

   -  omp：让编译接受OpenMP API 3.0任务序列指导指令（#pragma omp_task）。Intel任务序列指导指令将被忽略。

-  -qopenmp-threadprivate\ *=type*\ ：设定OpenMP线程私有的实现。\ *type*\ 可以为：

   -  legacy：让编译器继承使用以前Intel编译器使用的OpenMP线程私有实现。

   -  compat：让编译器使用基于对每个私有线程变量应用__declspec(thread)属性的兼容OpenMP线程私有实现。

浮点选项
''''''''

-  -fast-transcendentals：让编译器使用超越函数代替，超越函数是较快但精度较低的实现。

-  -fimf-absolute-error\ *=value[:funclist]*\ ：定义对于数学函数返回值允许的最大绝对误差的值。\ *value*\ 为正浮点数，\ *funclist*\ 为函数名列表。如：-fimf-absolute-error=0.00001:sin,sinf。

-  -fimf-accuracy-bits=\ *bits[:funclist]*\ ：定义数学函数返回值的相对误差，包含除法及开方。\ *bits*\ 为正浮点数，指明编译器应该使用的正确位数，\ *funclist*\ 为函数名列表。如：-fimf-accuracy-bits=23:sin,sinf。bits与ulps之间的变换关系为：\ :math:`ulps=2^{p-1-bits}`\ ，其中\ :math:`p`\ 为目标格式尾数\ :math:`bits`\ 的位数（对应单精度、双精度和长双精度分别为23、53和64）。

-  -fimf-max-error\ *=ulps[:funclist]*\ ：定义对于数学函数返回值的最大允许相对误差，包含除法及开方。\ *value*\ 为正浮点数，指定编译器可以使用的最大相对误差，\ *funclist*\ 为函数名列表，如：-fimf-max-error=4.0:sin,sinf。

-  -fimf-precision\ *[=value[:funclist]]*\ ：当设定使用何种数学库函数时，定义编译器应该使用的精度。\ *value*\ 可以为：

   -  high：等价于max-error = 0.6

   -  medium：等价于max-error = 4

   -  low：等价于accuracy-bits = 11（对单精度）和accuracy-bits = 26（对双精度）

   *funclist*\ 为函数名列表，如： -fimf-precision=high:sin,sinf。

-  -fma、-no-fma：是否对存在融合乘加(fused multiply-add-FMA)的目标处理器启用融合乘加。此选项只有在-x或-march参数设定CORE-AVX2或更高时才有效。

-  -fp-model *keyword*\ ：控制浮点计算的语义，\ *keyword*\ 可以为：

   -  precise：取消浮点数据的非值安全优化。

   -  fast[=1|2]：对浮点数据启用更加激进的优化。

   -  strict：启用精度和异常，禁止收缩，启用编译指示stdc和fenv_access。

   -  source：四舍五入中间结果到源定义精度。

   -  double：四舍五入中间结果到53-bit（双）精度。

   -  extended：四舍五入中间结果到64-bit（扩展）精度。

   -  [no-]except：定义严格浮点异常编译指令是否启用。

   *keyword*\ 可以分成以下三组使用：

   -  precise, fast, strict

   -  source, double, extended

   -  except

-  -fp-port、-no-fp-port：是否对浮点操作启用四舍五入。

-  -fp-speculation\ *=mode*\ ：设定推测浮点操作时使用的模式。\ *mode*\ 可以为：

   -  fast：让编译器推测浮点操作。

   -  safe：让编译器在推测浮点操作有可能存在浮点异常时停止推测。

   -  strict：让编译器禁止浮点操作时推测。

   -  off：与strict相同。

-  -fp-trap\ *=mode[,mode,...]*\ ：设置主函数的浮点异常捕获模式。\ *mode*\ 可以为：

   -  [no]divzero：是否启用被0除时的IEEE捕获。

   -  [no]inexact：是否启用不精确结果时的IEEE捕获。

   -  [no]invalid：是否启用无效操作时的IEEE捕获。

   -  [no]overflow：是否启用上溢时的IEEE捕获。

   -  [no]underflow：是否启用下溢时的IEEE捕获。

   -  [no]denormal：是否启用非正规时的IEEE捕获。

   -  all：启用上述所有的IEEE捕获。

   -  none：禁止启用上述所有的IEEE捕获。

   -  common：启用最常见的IEEE捕获：被0除、无效操作和上溢。

-  -fp-trap-all\ *=mode[,mode,...]*\ ：设置所有函数的浮点异常捕获模式。\ *mode*\ 可以为：

   -  [no]divzero：是否启用被0除时的IEEE捕获。

   -  [no]inexact：是否启用不精确结果时的IEEE捕获。

   -  [no]invalid：是否启用无效操作时的IEEE捕获。

   -  [no]overflow：是否启用上溢时的IEEE捕获。

   -  [no]underflow：是否启用下溢时的IEEE捕获。

   -  [no]denormal：是否启用非正规时的IEEE捕获。

   -  all：启用上述所有的IEEE捕获。

   -  none：禁止启用上述所有的IEEE捕获。

   -  common：启用最常见的IEEE捕获：被0除、无效操作和上溢。

-  -ftz：赋值非常规操作结果为0。

-  -mp1：提高浮点操作的精度和一致性。

-  -pc\ *n*\ ：设定浮点尾数精度。\ *n*\ 可以为：

   -  32：四舍五入尾数到24位（单精度）。

   -  64：四舍五入尾数到53位（双精度）。

   -  80：四舍五入尾数到64位（扩展精度）。

-  -prec-div、-no-prec-div：是否提高浮点除的精度。

-  -prec-sqrt、-no-prec-sqrt：是否提高开根的精度。

-  -rcd：启用快速浮点数到整数转换。

内联选项
''''''''

-  -gnu89-inline：设定编译器在C99模式时使用C89定义处理内联函数。

-  -finline、-fno-inline：是否对__inline声明的函数进行内联，并执行C++内联。

-  -finline-functions、-fno-inline-functions：对单个文件编译时启用函数内联。

-  -finline-limit\ *=n*\ ：设定内联函数的最大数。\ *n*\ 为非负整数。

-  -inline-calloc、-no-inline-calloc：是否设定编译器内联调用calloc()为调用malloc()和memset()。

-  -inline-factor、-no-inline-factor：是否设定适用于所有内联选项定义的上限的比例乘法器。

-  -inline-level\ *=n*\ ：设定内联函数的展开级别。\ *n*\ 可以为0、1、2.

输出、调试及预编译头文件(PCH)选项
'''''''''''''''''''''''''''''''''

-  -c：仅编译成对象文件（.o文件）。

-  -debug *[keyword]*\ ：设定是否生成调试信息。\ *keyword*\ 可以为：

   -  none：不生成调试信息。

   -  full或all：生成完全调试信息。

   -  minimal：生成最少调试信息。

   -  [no]emit_column：设定是否针对调试生成列号信息。

   -  [no]expr-source-pos：设定是否在表达式粒度级别生成源位置信息。

   -  [no]inline-debug-info：设定是否针对内联代码生成增强调试信息。

   -  [no]macros：设定是否针对C/C++宏生成调试信息。

   -  [no]pubnames：设定是否生成DWARF debug_pubnames节。

   -  [no]semantic-stepping：设定是否生成针对断点和单步的增强调试信息。

   -  [no]variable-locations：设定是否编译器生成有助于寻找标量局部变量的增强型调试信息。

   -  extended：设定关键字值semantic-stepping和variable-locations。

   -  [no]parallel：设定是否编译器生成并行调试代码指令以有助于线程数据共享和可重入调用探测。

-  -g：包含调试信息。

-  -g0：禁止生成符号调试信息。

-  -gdwarf-*n*\ ：设定生成调试信息时的DWARF版本，\ *n*\ 可以为2、3、4。

-  -o file：指定生成的文件名。

-  -pch：设定编译器使用适当的预编译头文件。

-  -pch-create *filename*\ ：设定生成预编译头文件。

-  -pch-dir *dir*\ ：设定搜索预编译头文件的目录。

-  -pch-use *filename*\ ：设定使用的预编译头文件。

-  -print-multi-lib：打印哪里系统库文件应该被发现。

-  -S：设定编译器只是生成汇编文件但并不进行链接。

预处理选项
''''''''''

-  -B\ *dir*\ ：设定头文件、库文件及可执行文件的搜索路径。

-  -D\ *name[=value]*\ ：设定编译时的宏及其值。

-  -dD：输出预处理的源文件中的#define指令。

-  -dM：输出预处理后的宏定义。

-  -dN：与-dD类似，但只输出的#define指令的宏名。

-  -E：设定预处理时输出到标注输出。

-  -EP：设定预处理时输出到标注输出，忽略#line指令。

-  -gcc、-no-gcc、-gcc-sys：判定确定的GNU宏(__GNUC__、__GNUC_MINOR__和\__GNUC_PATCHLEVEL__)是否定义。

-  -gcc-include-dir、-no-gcc-include-dir：设定是否将gcc设定的头文件路径加入到头文件路径中。

-  -H：编译时显示头文件顺序并继续编译。

-  -I：设定头文件附加搜索路径。

-  -icc、-no-icc：设定Intel宏(__INTEL_COMPILER)是否定义。

-  -idirafter\ *dir*\ ：设定\ *dir*\ 路径到第二个头文件搜索路径中。

-  -imacros *filename*\ ：允许一个头文件在编译时在其它头文件前面。

-  -iprefix *prefix*\ ：指定包含头文件的参考目录的前缀。

-  -iquote *dir*\ ：在搜索的头文件路径前面增加\ *dir*\ 目录以供那些使用引号而不是尖括号的文件使用。

-  -isystem\ *dir*\ ：附加\ *dir*\ 目录到系统头文件的开始。

-  -iwithprefix\ *dir*\ ：附加\ *dir*\ 目录到通过-iprefix引入的前缀后，并将其放在头文件目录末尾的头文件搜索路径中。

-  -iwithprefixbeforex\ *dir*\ ：除头文件目录\ *dir*\ 放置的位置与-I声明的一样外，与-iwithprefix类似。

-  -M：让编译器针对各源文件生成makefile依赖行。

-  -MD：预处理和编译，生成后缀为.d包含依赖关系的输出文件。

-  -MF\ *filename*\ ：让编译器在一个文件中生成makefile依赖信息。

-  -MG：让编译器针对各源文件生成makefile依赖行。与-M类似，但将缺失的头文件作为生成的文件。

-  -MM：让编译器针对各源文件生成makefile依赖行。与-M类似，但不包含系统头文件。

-  -MMD：预处理和编译，生成后缀为.d包含依赖关系的输出文件。与-M类似，但不包含系统头文件。

-  -MP：让编译器对每个依赖生成伪目标。

-  -MQ\ *target*\ ：对依赖生成改变默认目标规则。\ *target*\ 是要使用的目标规则。与-MT类似，但引用特定Make字符。

-  -MT\ *target*\ ：对依赖生成改变默认目标规则。\ *target*\ 是要使用的目标规则。

-  -nostdinc++：对C++不搜索标准目录下的头文件，而搜索其它标准目录。

-  -P：停止编译处理，并将结果写入文件。

-  -pragma-optimization-level\ *=interpretation*\ ：指定如没有前缀指定时，采用何种优化级别编译指令解释。\ *interpretation*\ 可以为：

   -  Intel：Intel解释。

   -  GCC：GCC解释。

-  -U\ *name*\ ：取消某个宏的预定义。

-  -undef：取消所有宏的预定义。

-  -X：从搜索路径中去除标准搜索路径。

C/C++语言选项
'''''''''''''

-  -ansi：与gcc的-ansi选项兼容。

-  -check\ *=keyword[, keyword...]*\ ：设定在运行时检查某些条件。\ *keyword*\ 可以为：

   -  [no]conversions：设定是否在转换成较小类型时进行检查。

   -  [no]stack：设定是否在堆栈帧检查。

   -  [no]uninit：设定是否对未初始化变量进行检查。

-  -fno-gnu-keywords：让编译器不将typeof作为一个关键字。

-  -fpermissive：让编译器允许非一致性代码。

-  -fsyntax-only：让编译器仅作语法检查，不生成目标代码。

-  -funsigned-char：将默认字符类型变为无符号类型。

-  -help-pragma：显示所有支持的编译指令。

-  -intel-extensions、-no-intel-extensions：是否启用Intel
   C和C++语言扩展。

-  -restrict、-no-restrict：设定是否采用约束限定进行指针消岐。

-  -std\ *=val*\ ：\ *val*\ 可以为c89、c99、gnu89、gnu++89或c++0x，分别对应相应标准。

-  -stdlib\ *[=keyword]*\ ：设定链接时使用的C++库。\ *keyword*\ 可以为：

   -  libstdc++：链接使用GNU libstdc++库。

   -  libc++：链接使用libc++库。

-  -strict-ansi：让编译器采用严格的ANSI一致性语法。

-  -x *type*\ ：\ *type*\ 可以为c、c++、c-header、cpp-output、c++-cpp-output、assembler、assembler-with-cpp或none，分别表示c源文件等，以使所有源文件都被认为是此类型的。

-  -Zp\ *[n]*\ ：设定结构体在字节边界的对齐。n是字节大小边界，可以为1、2、4、8和16。

Fortran语言选项
'''''''''''''''

-  -auto-scalar：INTEGER、REAL、COMPLEX和LOGICAL内在类型变量，如未声明有SAVE属性，将分配到运行时堆栈中，下次调用此函数时变量赋值。

-  -allow
   *keyword*\ ：设定编译器是否允许某些行为。\ *keyword*\ 可以为[no]fpp_comments，声明fpp预处理器如何处理在预处理指令行中的Fortran行尾注释。

-  -altparam、-noaltparam：设定是否允许不同的语法（不带括号）PARAMETER声明。

-   -assume *keyword[, keyword...]*\ ：设定某些假设。\ *keyword*\ 可以为：none、[no]bscc、[no]buffered_io、[no]buffered_stdout、[no]byterecl、[no]cc_omp、[no]dummy_aliases、[no]fpe_summary、[no]ieee_fpe_flags、[no]minus0、[no]old_boz、[no]old_ldout_format、[no]old_logical_ldio、[no]old_maxminloc、[no]old_unit_star、[no]old_xor、[no]protect_constants、[no]protect_parens、[no]realloc_lhs、[no]source_include、[no]std_intent_in、[no]std_minus0_rounding、[no]std_mod_proc_name、[no]std_value、[no]underscore、[no]2underscores、[no]writeable-strings等

-  -ccdefault *keyword*\ ：设置文件显示在终端上时的回车类型。\ *keyword*\ 可以为：

   -  none：设定编译器使用无回车控制预处理。

   -  default：设定编译器使用默认回车控制设定。

   -  fortran：设定编译器使用通常的第一个字符的Fortran解释。如字符0使得在输出一个记录时先输出一个空行。

   -  list：设定编译器记录之间出输出换行。

-  -check\ *=keyword[, keyword...]*\ ：设定在运行时检查某些条件。\ *keyword*\ 可以为：

   -  none：禁止所有检查。

   -  [no]arg_temp_created：设定是否在子函数调用前检查实参。

   -  [no]assume：设定是否在测试在ASSUME指令中的标量布尔表达式为真，或在ASSUME_ALIGNED指令中的地址对齐声明的类型边界时进行检查。

   -  [no]bounds：设定是否对数组下标和字符子字符串表达式进行检查。

   -  [no]format：设定是否对格式化输出的数据类型进行检查。

   -  [no]output_conversion：设定是否对在指定的格式描述域内的数据拟合进行检查。

   -  [no]pointers：设定是否对存在一些分离的或未初始化的指针或为分配的可分配目标时进行检查。

   -  [no]stack：设定是否在堆栈帧检查。

   -  [no]uninit：设定是否对未初始化变量进行检查。

   -  all：启用所有检查。

-  -cpp：对源代码进行预处理，等价于-fpp。

-  -extend-source\ *[size]*\ ：指明固定格式的Fortran源代码宽度，\ *size*\ 可为72、80和132。也可直接用-72、-80和-132指定，默认为72字符。

-  -fixed：指明Fortran源代码为固定格式，默认由文件后缀设定格式类别。

-  -free：指明Fortran源程序为自由格式，默认由文件后缀设定格式类别。

-  -nofree：指明Fortran源程序为固定格式。

-  -implicitnone：指明默认变量名为未定义。建议在写程序时添加implicit none语句，以避免出现由于默认类型造成的错误。

-  -names *keyword*\ ：设定如何解释源代码的标志符和外部名。\ *keyword*\ 可以为：

   -  lowercase：让编译器忽略标识符的大小写不同，并转换外部名为小写。

   -  uppercase：让编译器忽略标识符的大小写不同，并转换外部名为大写。

   -  as_is：让编译器区分标识符的大小写，并保留外部名的大小写。

-  -pad-source、-nopad-source：对固定格式的源码记录是否采用空白填充行尾。

-  -stand *keyword*\ ：以指定Fortran标准进行编译，编译时显示源文件中不符合此标准的信息。\ *keyword*\ 可为f03、f90、f95和none，分别对应显示不符合Fortran
   2003、90、95的代码信息和不显示任何非标准的代码信息，也可写为-std\ *keyword*\ ，此时\ *keyword*\ 不带f，可为03、90、95。

-  -standard-semantics：设定编译器的当前Fortran标准行为是否完全实现。

-  -syntax-only：仅仅检查代码的语法错误，并不进行其它操作。

-  -wrap-margin、-no-wrap-margin：提供一种在Fortran列表输出时禁止右边缘包装。

-  -us：编译时给外部用户定义的函数名添加一个下划线，等价于-assume
   underscore，如果编译时显示_函数找不到时也许添加此选项即可解决。

数据选项
''''''''

-  共有选项

   -  -fcommon、-fno-common：设定编译器是否将common符号作为全局定义。

   -  -fpic、-fno-pic：是否生成位置无关代码。

   -  -fpie：类似-fpic生成位置无关代码，但生成的代码只能链接到可执行程序。

      -  -gcc：定义GNU宏。

      -  -no-gcc：取消定义GNU宏。

      -  -gcc-sys：只有在编译系统头文件时定义GNU宏。

   -  -mcmodel=\ *mem_model*\ ：设定生成代码和存储数据时的内存模型。\ *mem_model*\ 可以为：

      -  small：让编译器限制代码和数据使用最开始的2GB地址空间。对所有代码和数据的访问可以使用指令指针(IP)相对地址。

      -  medium：让编译器限制代码使用最开始的2GB地址空间，对数据没有内存限制。对所有代码的访问可以使用指令指针（IP）相对地址，但对数据的访问必须采用绝对地址。

      -  large：对代码和数据不做内存限制。所有访问都得使用绝对地址。

   -  -mlong-double-*n*\ ：覆盖掉默认的长双精度数据类型配置。\ *n*\ 可以为：

      -  64：设定长双精度数据为64位。

      -  80：设定长双精度数据为80位。

-  C/C++专有选项

   -  -auto-ilp32：让编译器分析程序设定能否将64位指针缩成32位指针，能否将64位长整数缩成32位长整数。

   -  -auto-p32：让编译器分析程序设定能否将64位指针缩成32位指针。

   -  -check-pointers=\ *keyword*\ ：设定编译器是否检查使用指针访问的内存边界。\ *keyword*\ 可以为：

      -  none：禁止检查，此为默认选项。

      -  rw：检查通过指针读写的内存边界。

      -  write：只检查通过内存写的内存边界。

   -  -check-pointers-dangling\ *keyword*\ ：设定编译器是否对悬挂（dangling）指针参考进行检查。\ *keyword*\ 可以为：

      -  none：禁止检查悬挂指针参考，此为默认选项。

      -  heap：检查heap的悬挂指针参考。

      -  stack：检查stack的悬挂指针参考。

      -  all：检查上述所有的悬挂指针参考。

   -  -fkeep-static-consts、-fno-keep-static-consts：设定编译器是否保留在源文件中没有参考的变量分配。

-  Fortran专有选项

   -  -convert *[keyword]*\ ：转换无格式数据的类型，比如\ *keyword*\ 为big_endian和little_endian时，分别表示无格式的输入输出为big_endian和little_endian格式，更多格式类型，请看编译器手册。

   -  -double-size *size*\ ：设定DOUBLE PRECISION和DOUBLE COMPLEX声明、常数、函数和内部函数的默认KIND。\ *size*\ 可以为64或128，分别对应KIND=8和KIND=16。

   -  -dyncom *"common1,common2,..."*\ ：对指定的common块启用运行时动态分配。

   -  -fzero-initialized-in-bss、-fno-zero-initialized-in-bss：设定编译器是否将数据显式赋值为0的变量放置在DATA块内。

   -  -intconstant、-nointconstant：让编译器使用FORTRAN 77语法设定整型常数的KIND参数。

   -  -integer-size *size*\ ：设定整型和逻辑变量的默认KIND。\ *size*\ 可以为16、32或64，分别对应KIND=2、KIND=4或KIND=8。

   -  -no-bss-init：让编译器将任何未初始化变量和显式初始化为0的变量放置在DATA块。默认不启用，放置在BSS块。

   -  -real-size *size*\ ：设定实型变量的默认KIND。\ *size*\ 可以为32、64或18，分别对应KIND=4、KIND=8或KIND=16。

   -   -save：强制变量值存储在静态内存中。此选项保存递归函数和用AUTOMATIC声明的所有变量（除本地变量外）在静态分配中，下次调用时可继续用。默认为-auto-scalar，内在类型INTEGER、REAL、COMPLEX和LOGICAL变量分配到运行时堆栈中。

   -  -zero、-nozero：是否将所有保存的但未初始化的内在类型INTEGER、REAL、COMPLEX或LOGICAL的局部变量值初始为0。

编译器诊断选项
''''''''''''''

-  -diag-*type=diag-list*\ ：控制显示的诊断信息。\ *type*\ 可以为：

   -  enable：启用一个或一组诊断信息。

   -  disable：禁用一个或一组诊断信息。

   -  error：让编译器将诊断信息变为错误。

   -  warning：让编译器将诊断信息变成警告

   -  remark：让编译器将诊断信息变为备注。

   *diag-list*\ 可为：driver、port-win、thread、vec、par 、openmp 、warn、error 、remark 、cpu-dispatch 、id[,id,...] 、tag[,tag,...]等。

-  -traceback、-notraceback：编译时在对象文件中生成额外的信息使得在运行出错时可以提供源文件回朔信息。

-  -w：编译时不显示任何警告，只显示错误。

-  -w\ *n*\ ：设置编译器生成的诊断信息级别。\ *n*\ 可以为：

   -  0：对错误生成诊断信息，屏蔽掉警告信息。

   -  1：对错误和警告生成诊断信息。此为默认选项。

   -  2：对错误和警告生成诊断信息，并增加些额外的警告信息。

   -  3：对备注、错误和警告生成诊断信息，并在级别2的基础上再增加额外警告信息。建议对产品使用此级别。

   -  4：在级别3的基础上再增加一些警告和备注信息，这些增加的信息一般可以安全忽略。

-  -Wabi、-Wno-abi：设定生成的代码不是C++ ABI兼容时是否显示警告信息。

-  -Wall：编译时显示警告和错误信息。

-  -Wbrief：采用简短方式显示诊断信息。

-  -Wcheck：让编译器在对特定代码在编译时进行检查。

-  -Werror：将所有警告信息变为错误信息。

-  -Werror-all将所有警告和备注信息变为错误信息。

-  -Winline：设定编译器显示哪些函数被内联，哪些未被内联。

-  -Wunused-function、-Wno-unused-functio：设定是否在声明的函数未使用时显示警告信息。

-  -Wunused-variable、-Wno-unused-variable：设定是否在声明的变量未使用时显示警告信息。

兼容性选项
''''''''''

-  -f66：使用FORTRAN 66标准，默认为使用 Fortran 95标准。

-  -f77rtl、-nof77rtl： 是否使用FORTRAN 77运行时行为，默认为使用Intel Fortran运行时行为。控制以下行为：

   -  当unit没有与一个文件对应时，一些INQUIRE说明符将返回不同的值：

      -  NUMBER= 返回0；

      -  ACCESS= 返回’UNKNOWN’；

      -  BLANK= 返回’UNKNOWN’；

      -  FORM= 返回’UNKNOWN’。

   -  PAD= 对格式化输入默认为"NO’。

   -  NAMELIST和列表输入的字符串必需用单引号或双引号分隔。

   -  当处理NAMELIST输入时：

      -  每个记录的第一列被忽略。

      -  出现在组名前的’$’或’&’必须在输入记录的第二列。

   -  -fpscomp [keyword[, keyword...]]、-nofpscomp：设定是否某些特征与IntelFortran或Microsoft\* Fortran PowerStation兼容。\ *keyword*\ 可以为：

      -  none：没有选项需要用于兼容性。

      -  [no]filesfromcmd：设定当OPEN声明中FILE=说明符为空时的兼容性。

      -  [no]general：设定当Fortran PowerStation和IntelFortran语法存在不同时的兼容性。

      -  [no]ioformat：设定列表格式和无格式IO时的兼容性。

      -  [no]libs：设定可移植性库是否传递给链接器。

      -  [no]ldio_spacing：设定是否在运行时在数值量后字符值前插入一个空白。

      -  [no]logicals：设定代表LOGICAL值的兼容性。

      -  all：设定所有选项用于兼容性。

-  -fabi-version=\ *n*\ ：设定使用指定版本的ABI实现。\ *n*\ 可以为：

   -  0：使用最新的ABI实现。

   -  1：使用gcc 3.2和gcc 3.3使用的ABI实现。

   -  2：使用gcc 3.4及更高的gcc中使用的ABI实现。

-  -gcc-name=\ *name*\ ：设定使用的gcc编译器的名字。

-  -gxx-name\ *name*\ ：设定使用的g++编译器的名字。

链接和链接器选项
''''''''''''''''

-  -Bdynamic：在运行时动态链接所需要的库。

-  -Bstatic ：静态链接用户生成的库。

-  -cxxlib\ *[=dir]*\ 、-cxxlib-nostd、-no-cxxlib：设定是否使用gcc提供的C++运行时库及头文件。\ *dir*\ 为gcc二进制及库文件的顶层目录。

-  -I\ *dir*\ ：指明头文件的搜索路径。

-  -L\ *dir*\ ：指明库的搜索路径。

-  -l\ *string*\ ：指明所需链接的库名，如库名为libxyz.a，则可用-lxyz指定。

-  -no-libgcc：禁止使用特定gcc库链接。

-  -nodefaultlibs：禁止使用默认库链接。

-  -nostartfiles：禁止使用标准启动文件链接。

-  -nostdlib：禁止使用标准启动和库文件链接。

-  -pie、-no-pie：设定编译器是否生成需要链接进可执行程序的位置独立代码

-  -pthread：对多线程启用pthreads库。

-  -shared：生成共享对象文件而不是可执行文件，必须在编译每个对象文件时使用-fpic选项。

-  -shared-intel：动态链接Intel库。

-  -shared-libgcc：动态链接GNU libgcc库。

-  -static：静态链接所有库。

-  -static-intel：静态链接Intel库。

-  -static-libgcc：静态链接GNU libgcc库。

-  -static-libstdc++：静态链接GNU libstdc++库。

-  -u *symbol*\ ：设定指定的符号未定义。

-  -v：显示驱动工具编译信息。

-  -Wa\ *,option1[,option2,...]*\ ：传递参数给汇编器进行处理。

-  -Wl\ *,option1[,option2,...]*\ ：传递参数给链接器进行处理。

-  -Wp\ *,option1[,option2,...]*\ ：传递参数给预处理器。

-  -Xlinker *option*\ ：将option信息传递给链接器。

其它选项
''''''''

-  -help *[category]*\ ：显示帮助。

-  -sox\ *[=keyword[,keyword]]*\ 、-no-sox：设定是否让编译时在生成的可执行文件中保存编译选项和版本等信息，也可以指定是否保存子程序等信息。

   -  inline：包含在各目标文件中的内联子程序名。

   -  profile：包含编译时采用-prof-use的子程序列表，以及存储概要信息的.dpi文件名和指明使用的和忽略的概要信息。

   存储的信息可以使用以下方法查看：

   -  ``objdump -sj .comment a.out``

   -  ``strings -a a.out | grep comment:``

-  -V：显示版本信息。

-  –version：显示版本信息。

-  -watch\ *[=keyword[, keyword...]]*\ 、-nowatch：设定是否在控制台显示特定信息。\ *keyword*\ 可以为：

   -  none：禁止cmd和source。

   -  [no]cmd：设定是否显示驱动工具命令及执行。

   -  [no]source：设定是否显示编译的文件名。

   -  all：启用cmd和source。
