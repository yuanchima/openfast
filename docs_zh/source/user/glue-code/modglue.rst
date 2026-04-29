.. _glue-code-modglue:

ModGlue——将模块组合为全局数组
===============================================

``ModGlue_CombineModules`` 是将由 ``MV_AddVar`` / ``MV_AddMeshVar`` / ``MV_AddModule`` 注册的各模块变量描述（参见 :ref:`glue-code-modvar`）转换为耦合系统*整体*视图的唯一子程序。该子程序的输出是一个 ``ModGlueType`` 结构，其 ``Vars`` 成员包含属于指定模块集的每个状态、输入和输出变量的全局索引数组。这些全局数组是求解器雅可比矩阵构建（:ref:`glue-code-solver`）和线性化流程（:ref:`glue-code-linearization`）的基础。

.. _fig-modvar-types:

.. figure:: ModVar.svg
   :alt: ModVar data structure hierarchy
   :align: center
   :width: 100%

   耦合代码使用的完整数据结构层次。左侧显示的类型（``DatLoc``、``Field ID``、``Variable Flag``）被每个 ``ModVarType`` 引用。``ModDataType``（中间，每个模块）由 ``ModGlue_CombineModules`` 收集到顶层的 ``ModGlueType``（右侧）中。

.. contents::
   :local:
   :depth: 2

``ModGlueType`` 结构
------------------------------

``ModGlueType`` 定义在 ``modules/openfast-library/src/Glue_Types.f90`` 中：

.. code-block:: fortran

   type ModGlueType
      character(ChanLen)                    :: Name     ! Label (e.g. 'Solver', 'Lin')
      type(ModDataType), allocatable        :: ModData(:) ! Per-module view
      type(ModVarsType)                     :: Vars       ! Combined variable arrays
      type(ModLinType)                      :: Lin        ! Linearization matrices
      type(VarMapType),  allocatable        :: VarMaps(:) ! Relevant mappings
   end type ModGlueType

关键成员：

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 成员
     - 描述
   * - ``ModData(:)``
     - 此耦合实例中包含的每个模块对应一个条目。每个条目是原始 ``ModDataType`` 的过滤副本，仅包含由 ``FlagFilter`` 选择的变量子集。索引顺序与传递给 ``ModGlue_CombineModules`` 的 ``iModAry`` 参数一致。
   * - ``Vars%x(:)``
     - 所选模块中所有*连续状态*变量的 ``ModVarType`` 描述符拼接数组。
   * - ``Vars%u(:)``
     - 所有*输入*变量的 ``ModVarType`` 描述符拼接数组。
   * - ``Vars%y(:)``
     - 所有*输出*变量的 ``ModVarType`` 描述符拼接数组。
   * - ``Vars%Nx / %Nu / %Ny``
     - 每个组中的标量值总数（分别为 ``Vars%x / %u / %y`` 中所有变量的 ``Var%Num`` 之和）。这些是全局数据数组和雅可比矩阵的行/列维度。
   * - ``Lin``
     - 保存线性化工作点数组（``x``、``dx``、``u``、``y``）和全系统矩阵（``dXdx``、``dXdu``、``dYdx``、``dYdu``、``dUdu``、``dUdy``）。仅当 ``Linearize=.true.`` 时分配。
   * - ``VarMaps(:)``
     - 全局 ``Mappings`` 数组的过滤子集，仅包含源模块**和**目标模块都出现在 ``iModAry`` 中的映射。在雅可比有限差分期间使用，用于解释输出到输入的耦合。

``iLoc`` 索引范围
-------------------------

在 ``ModGlue_CombineModules`` 返回后，``ModGlue%Vars%x / %u / %y`` 中的每个变量都带有一个 ``iLoc(1:2)`` 范围，用于定位其在*耦合层*数据向量中的位置。具体来说，对于 ``ModGlue%Vars%x`` 中位置 *k* 处的变量：

* ``iLoc(1)``——其第一个标量值在长度为 ``Vars%Nx`` 的数组中的索引。
* ``iLoc(2)``——其最后一个标量值的索引。

这是过滤后的*耦合本地*索引；对应的各模块位置仍可通过变量的 ``iGlu`` 范围获取（之前由 ``FAST_SolverInit → CalcVarGlobalIndices`` 设置）。``iLoc / iGlu`` 的分离意味着同一个变量描述符可以同时存在于各模块的 ``ModData`` 视图、求解器的 ``m%Mod`` 视图和线性化的 ``m%ModGlue`` 视图中，并且在每个上下文中具有一致、不重叠的索引范围。

``ModGlue_CombineModules`` 的功能
--------------------------------------

子程序签名如下：

.. code-block:: fortran

   subroutine ModGlue_CombineModules(ModGlue, ModDataAry, Mappings, &
                                      iModAry, FlagFilter, Linearize, &
                                      ErrStat, ErrMsg, Name)

.. list-table::
   :header-rows: 1
   :widths: 22 12 66

   * - 参数
     - 意图
     - 描述
   * - ``ModGlue``
     - ``out``
     - 要填充的 ``ModGlueType`` 结构。
   * - ``ModDataAry(:)``
     - ``in``
     - 耦合代码注册的各模块数据完整数组。
   * - ``Mappings(:)``
     - ``in``
     - 网格和变量映射的完整数组。
   * - ``iModAry(:)``
     - ``in``
     - ``ModDataAry`` 的有序索引列表，指定要包含*哪些*模块以及*按什么顺序*。该顺序决定了全局向量和雅可比矩阵的行/列布局。
   * - ``FlagFilter``
     - ``in``
     - ``VF_*`` 标志的位掩码。只有至少设置了其中一个标志的变量（即 ``MV_HasFlagsAny(Var, FlagFilter)`` 为真）才会被复制到耦合 ``Vars`` 数组中。
   * - ``Linearize``
     - ``in``
     - 当为 ``.true.`` 时，分配 ``Lin`` 工作点数组和全系统雅可比矩阵。
   * - ``Name``
     - ``in``（可选）
     - 存储在 ``ModGlue%Name`` 中的人类可读标签（例如 ``'Solver'``、``'Lin'``）。

内部执行的四个主要步骤：

1. **计数与分配**。遍历 ``iModAry`` 中的每个模块，统计 *x*、*u*、*y* 三个组中通过 ``FlagFilter`` 测试的变量描述符数量（以及总标量值数量）。为 ``ModGlue%Vars%x / %u / %y`` 分配恰好这些大小的空间。

2. **复制与重新索引**。对于每个模块，将过滤后的 ``ModVarType`` 描述符复制到 ``ModGlue%Vars`` 中，并分配连续的 ``iLoc`` 范围，使得耦合层索引在所有模块间是连续的。各模块的 ``ModGlue%ModData(i)%Vars`` 子数组接收相同的描述符和相同的 ``iLoc``，这样分散/收集例程可以直接操作全局向量。在此步骤中，线性名称前缀（例如 ``"ED "``、``"BD_1 "``）会被添加到 ``LinNames`` 前，以生成全局唯一的通道标签。

3. **过滤映射**。遍历完整的 ``Mappings`` 数组，仅保留那些 ``iModSrc`` 和 ``iModDst`` 都出现在 ``iModAry`` 中的映射。根据本地 ``ModData`` 位置（而非全局 ``ModDataAry`` 位置）重新索引保留的条目，并存储在 ``ModGlue%VarMaps`` 中。

4. **分配线性化存储**（仅当 ``Linearize=.true.`` 时）。分配 ``Lin`` 工作点向量（``x``、``dx``、``u``、``y``）和所有六个雅可比矩阵（``dXdx``、``dXdu``、``dYdx``、``dYdu``、``dUdu``、``dUdy``），维度由 ``Vars%Nx`` 和 ``Vars%Nu / Ny`` 确定。

``ModGlue_CombineModules`` 的调用位置
------------------------------------------

OpenFAST 初始化期间会调用该例程两次，生成两个具有不同变量选择的独立 ``ModGlueType`` 实例：

``m%Mod``——求解器耦合模块（在 ``FAST_SolverInit`` 中）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: fortran

   iMod = [p%iModTC, p%iModOpt1]   ! TC + Option-1 indices

   call ModGlue_CombineModules(m%Mod, GlueModData, GlueModMaps, iMod, &
                               VF_Solve, .true., ErrStat2, ErrMsg2, &
                               Name='Solver')

* **包含模块**：紧耦合（TC）模块加选项 1 模块。
* **变量过滤器**：``VF_Solve``。该标志由 ``FAST_SolverInit → SetVarSolveFlags`` 在必须出现在牛顿迭代中的每个变量上设置：TC 连续状态、参与模块间耦合的运动/载荷网格输入/输出，以及任何变量到变量映射的输入/输出（完整选择标准参见 :ref:`glue-code-solver`）。
* **线性化**：``.true.``——在此处分配雅可比矩阵，因为求解器的牛顿线性系统重用与工作点线性化相同的存储空间。
* **结果**：``m%Mod%Vars%Nx`` 等于 TC 位移/速度标量的数量（``p%NumQ``）；``m%Mod%Vars%Nu`` 覆盖所有标记为 ``VF_Solve`` 的 TC 和选项 1 输入。雅可比维度 ``p%NumJ = p%NumQ + p%NumU`` 直接由此得到。

``m_FAST%ModGlue``——线性化耦合模块（在 ``ModGlue_Init`` 中）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: fortran

   LinFlags = VF_Linearize + VF_Mapping + VF_Mesh

   call ModGlue_CombineModules(m%ModGlue, m%ModData, m%Mappings, &
                               p%Lin%iMod, LinFlags, &
                               p_FAST%Linearize, ErrStat2, ErrMsg2, &
                               Name="Lin")

* **包含模块**：所有参与线性化的模块，按 ``p%Lin%iMod`` 设置的规范顺序排列（InflowWind → SeaState → ServoDyn → ElastoDyn → BeamDyn → AeroDyn → HydroDyn → SubDyn → MAP++ → MoorDyn）。
* **变量过滤器**：``VF_Linearize + VF_Mapping + VF_Mesh``。``VF_Linearize`` 标志根据输入文件中的 ``LinInputs`` / ``LinOutputs`` 设置应用于每个变量；``VF_Mapping`` / ``VF_Mesh`` 确保始终包含网格耦合变量，这样即使某个变量不是正式的线性化输出，也可以组装耦合雅可比矩阵。
* **线性化**：``p_FAST%Linearize``——仅当请求线性化时才分配完整的全系统矩阵。
* **结果**：``m%ModGlue%Vars%Nx / %Nu / %Ny`` 给出写入 ``.lin`` 文件的 **A**、**B**、**C**、**D**、**dUdu**、**dUdy** 矩阵的维度。

全局索引如何支持矩阵组装
--------------------------------------------

由于每个变量描述符都带有其在耦合 ``Vars`` 数组中的 ``iLoc`` 范围，全局数据向量上的分散和收集操作变成了简单的索引范围赋值。例如，将模块的状态向量打包到全局求解器向量 ``x_global`` 中：

.. code-block:: fortran

   do iVar = 1, size(ModData%Vars%x)
      associate (Var => ModData%Vars%x(iVar))
         x_global(Var%iLoc(1):Var%iLoc(2)) = x_mod(Var%iGlu(1):Var%iGlu(2))
      end associate
   end do

而从全局雅可比列收集到各模块子列的对应操作不需要偏移量计算：``iLoc`` 索引已经编码了正确的全局位置。

类似地，存储在 ``ModGlueType`` 中的 ``VarMaps`` 数组使雅可比耦合项成为自包含的。在 ``BuildJacobianTC`` / ``BuildJacobianIO``（在 ``FAST_Solver.f90`` 中）和 ``ModGlue_Linearize_OP``（在 ``FAST_ModGlue.f90`` 中）执行期间，循环很简单：

.. code-block:: fortran

   do i = 1, size(ModGlue%VarMaps)
      ! perturb source output, evaluate destination input change
      ! scatter result into J(iVarDst%iLoc, iVarSrc%iLoc)
   end do

此层级不需要特定于模块的知识——``ModGlueType`` 实例是完全自描述的。
