性能剖析与调优
================================
OpenFAST 团队的一个主要关注点是对 OpenFAST 软件进行性能剖析和优化，
目标是提高计算开销最大的用例的
求解时间性能。该过程通常涉及
初步剖析和热点分析，然后识别特定的子程序
以针对物理模块和耦合框架代码进行优化。

这项工作的一部分得到了 Intel® 的支持，通过将 NLR 指定为
`Intel® Parallel Computing Center (IPCC) <https://software.intel.com/en-us/ipcc>`_。

此处介绍过程、发现和建议的编程实践。
这是一份持续更新的文档，将随着更多研究的完成而更新。

高性能编程技术
---------------------------------------
作为一种为科学和工程应用设计的编译语言，
Fortran 非常适合生成非常高效的
代码。然而，调优代码的过程需要开发者
理解语言以及可用的工具（编译器、
性能库等），以生成最高
性能。本节确定了有效使用
Fortran 和 Intel® Fortran 编译器的编程模式。开发者
还应参考 Intel® Fortran 编译器文档中关于
`优化 <https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-oneapi-dev-guide-and-reference/top/optimization-and-programming-guide.html>`_
的一般内容，特别是关于
`自动矢量化 <https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-oneapi-dev-guide-and-reference/top/optimization-and-programming-guide/vectorization/automatic-vectorization.html>`_
和 `coarrays <https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-oneapi-dev-guide-and-reference/top/optimization-and-programming-guide/coarrays-1.html>`_ 的章节。

优化报告
~~~~~~~~~~~~~~~~~~~
在评估复杂软件中的编译器优化性能时，
仅靠计时测试不能很好地反映
编译器优化特定代码行的能力。对于
编译器优化尝试的低级信息，
开发者应生成优化报告以获取
关于各种指标的逐行报告，如矢量化、
并行化、内存和缓存使用、线程化等。
开发者应参考 Intel® Fortran 编译器文档中
关于 `优化报告 <https://software.intel.com/content/www/us/en/develop/articles/vectorization-and-optimization-reports.html>`_ 的部分。

对于 Linux 和 macOS，OpenFAST CMake 配置具有用于生成优化报告的编译器
标志，这些标志可用但被注释在
``openfast/cmake/OpenfastFortranOptions.cmake`` 中的 ``set_fast_intel_fortran_posix`` 宏中。
主要应使用 ``qopt-report-phase`` 和 ``qopt-report`` 标志。
有关其他标志和配置的更多信息，
请参阅优化报告选项 `文档 <https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-developer-guide-and-reference/top/compiler-reference/compiler-options/compiler-option-details/optimization-report-options/qopt-report-qopt-report.html>`_。

正确配置编译器标志后，编译器将
输出扩展名为 ``.optrpt`` 的文件，伴随中间编译
产物如 ``.o`` 文件。编译过程将说明
正在生成附加文件：

.. code-block::

  ifort: remark #10397: optimization reports are generated in *.optrpt files in the output location

附加文件应位于每个编译目标的相应
``CMakeFiles`` 目录中。例如，
OpenFAST 中 VersionInfo 模块的优化报告
位于：

.. code-block::

  >> ls -l openfast/build/modules/version/CMakeFiles/versioninfolib.dir/src/
  -rw-r--r--  2740 May 12 23:10 VersionInfo.f90.o
  -rw-r--r--     0 May 12 23:10 VersionInfo.f90.o.provides.build
  -rw-r--r--   668 May 12 23:10 VersionInfo.f90.optrpt

运算符强度削减
~~~~~~~~~~~~~~~~~~~~~~~~~~~
每种数学运算都有一个有效的"强度"，一些
运算可以等效地表示为多个
强度削减运算的组合，这些运算比
原始运算具有更好的性能。作为代码优化步骤的一部分，编译器可能
能够识别数学运算的强度可以
被削减的区域。编译器不能优化所有昂贵的运算。例如，
当昂贵数学运算的一部分
是包含在派生数据类型中的变量时，经常被跳过。
因此，建议对昂贵的子程序进行剖析
并搜索可能的强度削减机会。

运算符强度削减的一个具体示例是将
数组中的许多元素除以一个常数。

.. code-block:: fortran

  module_type%scale_factor = 10.0

  do i = 1
    if array(i) < 30.0
      array(i) = array(i) / module_type%scale_factor
    end if
  end do

在这种情况下，实数乘法比
实数除法开销更小。可以重构代码，
在循环外计算比例因子的倒数，
并将循环中的数学运算转换为
乘法。

.. code-block:: fortran

  module_type%scale_factor = 10.0
  inverse_scale_factor = 1.0 / module_type%scale_factor

  do i = 1
    if array(i) < 30.0
      array(i) = array(i) * inverse_scale_factor
    end if
  end do

Coarrays
~~~~~~~~
Coarrays 是 Fortran 语言在 2008 标准中引入的一项特性，
用于在单程序多数据（SPMD）编程范式下
为数组操作提供语言级别的并行化。
Fortran 标准将并行化方法留给
编译器决定，Intel® Fortran 编译器使用 MPI。

Coarrays 用于将数组操作拆分到程序的多个副本
上。这些副本称为"镜像"（image）。每个镜像有它
自己的局部变量，以及任何 coarrays 作为共享
变量的一部分。coarray 可以被认为有额外的维度，
称为"协维度"（codimension）。单个镜像（通常是第 1
索引）控制 I/O 和问题设置，镜像可以从
其他镜像的内存中读取。

对于大型数组的操作，例如从许多子数组
构造一个超级数组（如 Jacobian 矩阵的构造），
Fortran 08 的 coarray 特性可以并行化该过程，
提高整体计算效率。

.. TODO: 添加 Fortran 中 coarray 实现的示例

数据建模和访问规则
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Fortran 以列优先顺序表示数组。这意味着
多维数组在内存中以列元素
相邻的方式表示。如果数组中的某个元素位于内存中的某个位置，
则内存中前一个元素对应于
其列中上方的元素。

为了利用现代处理器的单指令多数据
特性，数组的构造和访问
应按列优先顺序进行。也就是说，循环应
最快地遍历最左边的索引。切片应尽可能
以最左边的索引上的 ``:`` 进行。

考虑到这一点，数据应表示为数组结构体
而非结构体数组。具体而言，这意味着
OpenFAST 中的数据类型应包含底层数组，数组
通常应仅包含数值类型。

下面的简短程序显示了数组元素与相邻元素之间
以字节为单位的内存距离。

.. code-block:: fortran

  program memloc

  implicit none

  integer(kind=8), dimension(3, 3) :: r, distance_up, distance_left

  ! Take the element values as their "ID"
  ! r(row, column)
  r(1,:) = (/ 1, 2, 3 /)
  r(2,:) = (/ 4, 5, 6 /)
  r(3,:) = (/ 7, 8, 9 /)
  print *, "Reference array:"
  call pretty_print_array(r)

  ! Compute the distance between matrix elements. Inputs to the `calculate_distance` function
  ! are indices for elements in the equation loc(this_element) - loc(other_element)
  distance_up(1,:) = (/ calculate_distance( 1,1 , 1,1), calculate_distance( 1,2 , 1,2), calculate_distance( 1,3 , 1,3) /)
  distance_up(2,:) = (/ calculate_distance( 2,1 , 1,1), calculate_distance( 2,2 , 1,2), calculate_distance( 2,3 , 1,3) /)
  distance_up(3,:) = (/ calculate_distance( 3,1 , 2,1), calculate_distance( 3,2 , 2,2), calculate_distance( 3,3 , 2,3) /)
  print *, "Distance in memory (bytes) for between an element and the one above it (top row zeroed):"
  call pretty_print_array(distance_up)

  distance_left(1,:) = (/ calculate_distance( 1,1 , 1,1), calculate_distance( 1,2 , 1,1), calculate_distance( 1,3 , 1,2) /)
  distance_left(2,:) = (/ calculate_distance( 2,1 , 2,1), calculate_distance( 2,2 , 2,1), calculate_distance( 2,3 , 2,2) /)
  distance_left(3,:) = (/ calculate_distance( 3,1 , 3,1), calculate_distance( 3,2 , 3,1), calculate_distance( 3,3 , 3,2) /)
  print *, "Distance in memory (bytes) for between an element and the one to the its left (first column zeroed):"
  call pretty_print_array(distance_left)

  contains

  integer(8) function calculate_distance(c1, r1, c2, r2)

      integer, intent(in) :: c1, r1, c2, r2
      calculate_distance = loc(r(c1, r1)) - loc(r(c2, r2))

  end function

  subroutine pretty_print_array(array)

      integer(8), dimension(3,3), intent(in) :: array
      print *, "["
      print '(I4, I4, I4)', array(1,1), array(1,2), array(1,3)
      print '(I4, I4, I4)', array(2,1), array(2,2), array(2,3)
      print '(I4, I4, I4)', array(3,1), array(3,2), array(3,3)
      print *, "]"

  end subroutine

  end program

优化研究
--------------------
本节描述了使用 Intel® 编译器套件对 OpenFAST
各部分进行剖析和提高性能的具体工作。

BeamDyn 性能剖析与优化（IPCC 第 1 年和第 2 年）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
已识别的 OpenFAST 性能改进的一般机制有：

* Intel® 编译器套件和 Intel® Math Kernel Library（Intel® MKL）
* 算法改进
* 内存访问优化以实现更高效的缓存使用
* 数据类型对齐以允许 SIMD 矢量化
* 使用 OpenMP 进行多线程

为了确定这些选项的前进路径，首先使用
Intel® VTune™ Amplifier 对 OpenFAST 进行剖析，以清晰了解
仿真中花费的时间。然后，分析 Intel®
Fortran 编译器生成的优化报告，以确定未
自动矢量化的区域。最后，使用 Intel® Advisor 突出显示编译器
识别为可以通过多线程改进的代码区域。

选择了两个 OpenFAST 测试用例来提供有意义且
现实的计时基准。除了真实的风力机和
大气模型外，这些用例计算开销大，并暴露出
性能改进能产生影响的区域。

**5MW_Land_BD_DLL_WTurb**

在 `此处 <https://github.com/OpenFAST/r-test/tree/dev/glue-codes/openfast/5MW_Land_BD_DLL_WTurb>`_ 下载用例文件。

此用例中使用的物理模块有：

* BeamDyn
* InflowWind
* AeroDyn 15
* ServoDyn

这是一个陆上 Jonkman 5-MW（以前称为 NREL 5-MW）风力机仿真，使用 BeamDyn 作为
结构模块。它以 0.001 秒的时间步长仿真 20 秒，
在 NLR 以前的 Peregrine
超级计算机（已退役）上执行时间为 `3 分 55 秒 <https://my.cdash.org/testDetails.php?test=40171217&build=1649048>`__。

**5MW_OC4Jckt_DLL_WTurb_WavesIrr_MGrowth**

在 `此处 <https://github.com/OpenFAST/r-test/tree/dev/glue-codes/openfast/5MW_OC4Jckt_DLL_WTurb_WavesIrr_MGrowth>`__ 下载用例文件。

这是一个海上固定式基础 Jonkman 5-MW（以前称为 NREL 5-MW）风力机仿真，
大部分计算开销发生在 HydroDyn 波浪动力学
计算中。

此用例中使用的物理模块有：

* ElastoDyn
* InflowWind
* AeroDyn 15
* ServoDyn
* HydroDyn
* SubDyn

它以 0.01 秒的时间步长仿真 60 秒，
在 NLR 以前的 Peregrine
超级计算机（已退役）上执行时间为
`20 分 27 秒 <https://my.cdash.org/testDetails.php?test=40171219&build=1649048>`__。

剖析
+++++++++
使用 Intel® VTune™ Amplifier 对 OpenFAST 测试用例进行剖析，以
识别性能热点。由于两个测试用例运行
OpenFAST 软件的不同部分，识别出了不同的热点。
在所有情况和环境设置下，大部分
CPU 时间都花在 ``fast_solution`` 循环中，这是一个
协调每个物理模块求解计算的高层子程序。

LAPACK
......
在海上用例中，LAPACK 的使用被识别为一个性能负载。
在 ``fast_solution`` 循环中，对 LAPACK 函数 ``dgetrs`` 的调用
消耗了总 CPU 时间的 3.3%。

.. figure:: images/offshore_lapack.png
   :width: 100%
   :align: center

BeamDyn
.......
虽然 BeamDyn 提供高保真度叶片响应计算，但它是一个
计算开销很大的模块。初步剖析突出显示
``bd_elementmatrixga2`` 子程序为热点。然而，最初
在 BeamDyn 中提高性能的尝试揭示了需要算法
改进和模块数据结构的优化。

结果
+++++++
虽然工作仍在进行中，OpenFAST 的求解时间性能已有改进，
性能潜力也得到了更好的理解。

IPCC 项目第一年的一些关键成果如下：

* 使用 Intel® 编译器和 MKL 库相比 GCC
  和 LAPACK 提供了显著的加速

  * 通过 MKL 线程化，海上仿真
    可获得额外的显著收益

* 海上风力机仿真在模块间的负载均衡
  较差

  * 陆上风力机配置均衡较好
  * 使用 OpenMP Tasks 实现更好的负载均衡

* OpenMP 模块级并行提供了显著但有限的速度
  提升，原因是不同模块任务之间的不均衡
* 核心算法需要重大修改才能获得 OpenMP 和 SIMD
  效益

调优 Intel® 工具以在 NLR 硬件上发挥最佳性能并添加高层
多线程，其中一个基准用例获得了
最大 3.8 倍的求解时间改进。

加速比 - Intel® 编译器和 MKL
.................................
通过使用标准的 Intel® 开发者工具技术栈，展示了
相对于 GNU 工具的性能改进：

========= ================= ===================== ======================================
编译器    数学库            5MW_Land_BD_DLL_WTurb 5MW_OC4Jckt_DLL_WTurb_WavesIrr_MGrowth
========= ================= ===================== ======================================
GNU       LAPACK            2265 s (1.0x)         673 s (1.0x)
Intel® 17 LAPACK            1650 s (1.4x)         251 s (2.7x)
Intel® 17 MKL               1235 s (1.8x)         ---
Intel® 17 MKL 多线程        722 s (3.1x)          ---
========= ================= ===================== ======================================


加速比 - OpenMP 在 FAST_Solver 中
...............................
通过向 ``FAST_Solver`` 模块添加 OpenMP 指令，展示了
性能改进。虽然求解方案不均衡，
但并行化网格映射和计算例程产生了以下
加速比：

========= =============== ===================== ======================================
编译器       数学库             5MW_Land_BD_DLL_WTurb 5MW_OC4Jckt_DLL_WTurb_WavesIrr_MGrowth
========= =============== ===================== ======================================
Intel® 17 MKL - 1 线程   1073 s (2.1x)         100 s (6.7x)
Intel® 17 MKL - 8 线程    597 s (3.8x)          ---
========= =============== ===================== ======================================

正在进行的工作
++++++++++++
OpenFAST 性能改进的下一阶段集中在两个关键
领域：

1. 将先前工作的成果实施到 OpenFAST 模块和
   耦合框架代码中
2. 为 OpenFAST 在 Intel® 下一代平台上的高效执行做准备

.. Year 2 stuff:

.. Further, `Envision Energy USA, Ltd <http://www.envision-group.com/en/energy.html>`_
.. has continuously contributed code and expertise in this area.


.. Furthermore, NLR is optimizing OpenFAST for the future through profiling on
.. Intel next generation platform (NGP) simulators.

.. bd_5MW_dynamic
.. ~~~~~~~~~~~~~~
.. Download files `here <https://github.com/OpenFAST/r-test/tree/dev/modules/beamdyn/bd_5MW_dynamic>`__.

.. This is a standalone BeamDyn case of the Jonkman 5-MW (formerly called the NREL 5-MW) wind turbine. It simulates 30
.. seconds with a time step size of 0.002 seconds and executes in 24s on NREL's
.. Peregrine supercomputer.

.. BeamDyn dynamic solve

.. Performance Improvements
.. ------------------------
.. BeamDyn chosen as the module to improve from year 1

.. How to improve vectorization

.. BeamDyn Memory Alignment
.. ~~~~~~~~~~~~~~~~~~~~~~~~
.. Work accomplished to align beamdyn types in the dervive types module
.. - Ultimately, this needs to be done in the registry

.. Multithreading
.. ~~~~~~~~~~~~~~
.. OpenMP at the highest level
.. OpenMP added to BeamDyn dynamic solve

.. Speedup
.. -------

.. These are the areas where we have demonstrated performance improvements

.. BeamDyn Dynamic
.. ---------------
.. This improved beamdyn's time to solution by XX%

.. - VTune / Advisor
.. - Vectorization report
.. - SIMD report

.. Optimization Reports
.. The optimization reports provided by the Intel fortran compiler give a static
.. analysis of code optimization. Specifically, the vectorization and openmp
.. reports were analyzed to determine


线性化例程剖析
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
作为 `ARPA-E WEIS <https://arpa-e.energy.gov/technologies/projects/wind-energy-integrated-servo-control-weis-toolset-enable-controls-co-design>`_
项目的一部分，对 OpenFAST 中的线性化能力进行了剖析，
以描述性能特征和当前瓶颈。
这项工作专门针对 FAST 库中的线性化例程，主要位于
`FAST_Lin.f90 <https://github.com/OpenFAST/openfast/blob/main/modules/openfast-library/src/FAST_Lin.f90>`_，
以及各个物理模块中构造 Jacobian 矩阵的例程。
由于这些例程需要
构造大型矩阵，这是一个计算密集型过程，
内存访问率很高。

``FAST_Linearize_OP`` 子程序中线性化算法的高层
数据流如下。

.. mermaid::

  graph TD;
    D[Construct Module Jacobian]-->A[Calculate Module OP];
    A[Calculate Module OP]-->B[Construct Glue Code Jacobians];
    A[Calculate Module OP]-->C[Construct Glue Code State Matrices];

每个启用的物理模块在其相应的
``<Module>_Jacobian`` 和 ``<Module>_GetOP`` 例程中构造模块级矩阵，这些
矩阵的集合在 ``Glue_Jacobians`` 和 ``Glue_StateMatrices`` 中被组装成全局矩阵。
在对 ``FAST_Linearize_OP`` 中总 CPU 时间的自顶向下比较中，我们看到
耦合框架代码状态矩阵的构造是最昂贵的步骤。
HydroDyn Jacobian 计算相对于其他模块
Jacobian 计算也显得突出。

.. figure:: images/TopDown_FAST_LinearizeOP.jpg
   :width: 100%
   :align: center


Jacobian 和状态矩阵基于输入、输出
和连续状态的总数确定大小。虽然大小各不相同，但这些矩阵通常在
每个维度上包含数千个元素，且大多数为零。也就是说，Jacobian
和状态矩阵是大型且稀疏的。为了减少内存
分配和访问的开销，建议使用稀疏矩阵表示。
