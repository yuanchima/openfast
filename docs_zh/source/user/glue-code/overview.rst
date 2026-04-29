.. _glue-code-overview:

耦合代码概述
============

OpenFAST 耦合代码是将各个物理模块（ElastoDyn、AeroDyn、HydroDyn、ServoDyn 等）连接成耦合仿真的软件层。它主要位于 ``modules/openfast-library/src/`` 目录中，并依赖 ``modules/nwtc-library/src/ModVar.f90`` 中的 ``ModVar`` 模块来抽象描述模块之间交换的每个变量。

主要职责包括：

* 初始化每个模块并将其变量注册到耦合代码
* 管理多速率子步进（时间步长为全局时间步长约数的模块）
* 将一个模块的输出映射到另一个模块的输入（运动网格、载荷网格和标量变量）
* 在松耦合或紧耦合广义-α 耦合下运行时间步进循环
* 对组装好的系统进行线性化以生成状态空间矩阵

源文件
------

.. list-table::
   :header-rows: 1
   :widths: 35 65

   * - 文件
     - 用途
   * - ``FAST_Subs.f90``
     - 顶层初始化：读取输入文件，调用每个模块的 ``_Init`` 函数，并调用 ``MV_AddModule`` 将每个模块注册到耦合代码。
   * - ``FAST_ModGlue.f90``
     - 通过 ``ModGlue_CombineModules`` 将每个模块的变量描述组合成一个整体的 ``ModGlueType`` 结构；执行线性化（``ModGlue_Linearize_OP``）和稳态调整（``ModGlue_CalcSteady``）。
   * - ``FAST_Solver.f90``
     - 实现广义-α 紧耦合求解器（``FAST_SolverStep``）、输入输出收敛循环（``FAST_CalcOutputsAndSolveForInputs``）和雅可比矩阵组装。
   * - ``FAST_Mapping.f90``
     - 网格到网格和变量到变量的传递映射。
   * - ``FAST_Funcs.f90``
     - 模块级 ``CalcOutput``、``UpdateStates``、``CalcContStateDeriv``、``GetOperatingPoint`` 和 ``SetOperatingPoint`` 的包装器，用于调度到正确的模块实例。
   * - ``ModVar.f90`` (nwtc-library)
     - ``ModVar`` 模块：数据结构（``ModVarType``、``ModVarsType``、``ModDataType``、``DatLoc``）和所有 ``MV_*`` 子程序。

模块耦合类别
------------

在 ``FAST_SolverInit`` 初始化过程中，每个模块被精确分配到一个耦合类别：

.. list-table::
   :header-rows: 1
   :widths: 20 15 65

   * - 类别
     - 标志
     - 描述
   * - 紧耦合（TC）
     - ``MC_Tight``
     - 状态和加速度通过广义-α 牛顿迭代同时求解。当 ``ModCoupling`` ≥ 2 时，ElastoDyn、BeamDyn 和 SubDyn 是紧耦合模块。
   * - 选项 1
     - ``MC_Option1``
     - 输入依赖于紧耦合输出，并且在同一个牛顿循环中收敛的模块（例如 HydroDyn、MoorDyn、带结构控制器的 ServoDyn）。
   * - 选项 2
     - ``MC_Option2``
     - 在收敛循环之前每步调用一次的松耦合模块（InflowWind、SeaState、AeroDyn 等）。
   * - 后置
     - ``MC_Post``
     - 输入求解延迟到收敛循环之后的模块（ServoDyn、ExternalInflow）。

时间步进循环（概述）
--------------------

每次调用 ``FAST_SolverStep`` 遵循以下顺序：

1. **校正迭代**（外循环）——最多 ``p%MaxConvIter`` 次迭代。
2. **选项 2**——松耦合模块的输入求解 + 状态更新 + ``CalcOutput``。
3. **选项 1**——半隐式模块的输入求解 + 状态更新。
4. **紧耦合输入求解**——收集紧耦合模块的输入。
5. **收敛迭代**（内循环）——对紧耦合状态和输入进行 Newton-Raphson 更新，直到更新范数低于 ``ConvTol`` 或达到迭代极限。
6. **求解后输入求解**——ServoDyn、ExternalInflow。

模块注册和变量排序在 :ref:`glue-code-modvar` 中有详细描述。每个模块的变量如何组装成全局数组和雅可比矩阵在 :ref:`glue-code-modglue` 中介绍。求解器算法和雅可比构造在 :ref:`glue-code-solver` 中介绍。
