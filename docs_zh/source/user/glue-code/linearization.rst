.. _glue-code-linearization:

线性化
=============

OpenFAST 可以围绕周期性（或静态）工作点线性化整个多物理场系统，生成连续时间的一阶状态空间矩阵，形式为：

.. math::

   \dot{\mathbf{x}} &= A\,\mathbf{x} + B\,\mathbf{u} \\
   \mathbf{y}       &= C\,\mathbf{x} + D\,\mathbf{u}

以及耦合矩阵 *dUdu*（输入到输入前馈）和 *dUdy*（输出到输入耦合）。线性化引擎位于 ``modules/openfast-library/src/FAST_ModGlue.f90``。

.. contents::
   :local:
   :depth: 2

线性化用户输入
------------------------------

以下参数出现在主 OpenFAST 输入文件（``*.fst``）的**线性化**部分。

.. list-table::
   :header-rows: 1
   :widths: 25 12 63

   * - 参数
     - 类型
     - 描述
   * - ``Linearize``
     - 逻辑值
     - 主开关。设置为 ``True`` 以启用所有线性化功能。当为 ``False`` 时，所有其他线性化参数将被忽略。
   * - ``CalcSteady``
     - 逻辑值
     - 当为 ``True`` 时，OpenFAST 首先向前运行仿真，直到每个目标方位角的输出在转子旋转一周后收敛（稳态微调），然后在每个方位角执行线性化。当为 ``False`` 时，在用户指定的绝对仿真时间（``LinTimes``）执行线性化。
   * - ``TrimCase``
     - 整数
     - 在 ``CalcSteady`` 期间微调的控制器自由度，以实现周期性稳态。

       * ``1``——偏航
       * ``2``——发电机转矩
       * ``3``——叶片总距
   * - ``TrimTol``
     - 实数
     - 转子旋转一周期间归一化输出误差的 RMS 收敛容差。当误差低于此值时，微调停止。典型值：``1.0e-5``。
   * - ``TrimGain``
     - 实数
     - 内置微调控制器使用的比例增益。对于偏航/变桨情况，单位为 rad/(rad/s)；对于转矩情况，单位为 N·m/(rad/s)。
   * - ``Twr_Kdmp``
     - 实数
     - 在 ``CalcSteady`` 运行期间添加的人工塔筒阻尼系数（N/(m/s)），用于帮助阻尼瞬态并更快达到稳态。设置为 0 可禁用。
   * - ``Bld_Kdmp``
     - 实数
     - 在 ``CalcSteady`` 期间的人工叶片阻尼系数（N/(m/s)）。
   * - ``NLinTimes``
     - 整数
     - 每个转子旋转周期的线性化时间点数量（当 ``CalcSteady=False`` 时，为等间隔绝对时间瞬间的数量）。必须 ≥ 1。对于周期性模型，通常需要至少 12 个方位角来解析每转的变化。
   * - ``LinTimes``
     - 实数数组
     - 当 ``CalcSteady=False`` 时，执行线性化的绝对仿真时间（秒）。长度必须等于 ``NLinTimes``。当 ``CalcSteady=True`` 时被忽略。
   * - ``LinInputs``
     - 整数
     - 控制哪些输入变量出现在 **B** 和 **D** 矩阵中。

       * ``0``（``LIN_NONE``）——无输入；仅生成状态矩阵。
       * ``1``（``LIN_STANDARD``）——模块标记为 ``VF_Linearize`` 的输入（由每个模块的 ``InitVars`` 设置默认集）。
       * ``2``（``LIN_ALL``）——所有模块输入，包括调试输入。
   * - ``LinOutputs``
     - 整数
     - 控制哪些输出变量出现在 **C** 和 **D** 矩阵中。

       * ``0``（``LIN_NONE``）——无输出。
       * ``1``（``LIN_STANDARD``）——仅 ``WriteOutput`` 通道（``VF_WriteOut`` 标志）。
       * ``2``（``LIN_ALL``）——所有模块输出。
   * - ``LinOutJac``
     - 逻辑值
     - 当为 ``True`` 时（需要 ``LinInputs=LinOutputs=2``），完整的模块雅可比矩阵会写入线性化输出文件用于调试。
   * - ``LinOutMod``
     - 逻辑值
     - 当为 ``True`` 时，除了全系统文件外，还会写入每个模块的 ``.lin`` 文件。

线性化的模块支持
----------------------------------

出现在线性化变量排序中的模块（在 ``ModGlue_Init`` 中设置）为：

InflowWind → SeaState → ServoDyn → ElastoDyn → BeamDyn → AeroDyn → HydroDyn → SubDyn → MAP++ → MoorDyn

如果 ``Linearize=True``，而某个模块不在此有序列表中，会导致致命错误。

变量选择
------------------

在 ``ModGlue_Init`` 期间，``VF_Linearize`` 标志根据 ``LinInputs`` 和 ``LinOutputs`` 设置应用于变量：

* **状态（x）**：每个参与模块的所有连续状态变量始终设置 ``VF_Linearize`` 标志。
* **输入（u）**：

  * ``LIN_NONE`` → 所有输入变量清除该标志。
  * ``LIN_STANDARD`` → 保留模块 ``InitVars`` 中设置的 ``VF_Linearize`` 标志；模块开发者选择*标准*输入集。
  * ``LIN_ALL`` → 所有输入变量设置该标志。
  * 带有 ``VF_NoLin`` 的变量始终清除 ``VF_Linearize`` 标志，无论上述设置如何。

* **输出（y）**：

  * ``LIN_NONE`` → 所有输出变量清除该标志。
  * ``LIN_STANDARD`` → 仅对同时带有 ``VF_WriteOut`` 的输出设置该标志。
  * ``LIN_ALL`` → 所有输出变量设置该标志。
  * 带有 ``VF_NoLin`` 的变量始终被排除。

组合后的变量集通过 ``ModGlue_CombineModules`` 组装到名为 ``Lin`` 的 ``ModGlueType`` 中。

稳态微调（``CalcSteady``）
---------------------------------------

当 ``CalcSteady=True`` 时，每个时间步都会调用 ``ModGlue_CalcSteady`` 来检测周期性：

1. 标记为 ``VF_Linearize`` 的模块输出（不包括 ``VF_WriteOut``）被收集到按方位角索引的缓冲区中。
2. 每完成一周旋转后，将 ``NLinTimes`` 个方位角目标处的输出与前一周的输出通过归一化 RMS 误差进行比较：

   .. math::

      \varepsilon = \sqrt{\frac{1}{N} \sum_{i=1}^{N}
        \left(\frac{y_i^{\rm current} - y_i^{\rm previous}}{r_i}\right)^2}

   其中 :math:`r_i = \max(y_{i,\rm max} - y_{i,\rm min},\, 0.01)` 是当前旋转周的输出范围（设置下限以避免除以接近零的值）。

3. 当 :math:`\varepsilon < \texttt{TrimTol}` 时，``FoundSteady=True``，并自动开始在所有 ``NLinTimes`` 个方位角处进行线性化。

4. 如果仿真在接近 ``TMax`` 约两周内仍未收敛，发出警告并强制进行线性化。

缓冲区样本之间的方位角插值使用 ``MV_ExtrapInterp`` 中的外插例程（根据可用样本数量支持常数、线性和二次方案）。

工作点线性化
-------------------------------------

``ModGlue_Linearize_OP`` 在单个工作点（时间/方位角）组装全系统矩阵：

1. **模块雅可比**：对于每个模块，调用 ``FAST_JacobianPInput`` 和 ``FAST_JacobianPContState``，通过中心差分有限微分计算各模块子矩阵 *dYdu*、*dXdu*、*dYdx*、*dXdx*。扰动量级取自每个变量的 ``Perturb`` 字段（参见 :ref:`glue-code-modvar`）。

2. **工作点提取**：``FAST_GetOP`` 将当前状态、输入和输出打包到线性化数组中（``ModGlue%Lin%x``、``%u``、``%y``）。

3. **耦合矩阵**：输入-输出耦合矩阵 *dUdu* 和 *dUdy* 由网格映射雅可比组装而成，用于说明某些模块输入是其他模块输出的函数。

4. **全系统组装**：各模块子矩阵使用存储在每个 ``ModVarType`` 中的 ``iGlu`` 索引范围，放置到组合的耦合级矩阵中。

5. **输出**：``ModGlue_CalcWriteLinearMatrices`` 写入 ``.lin`` 文件，包含：

   * 工作点值（**x_op**、**u_op**、**y_op**）
   * 线性化通道名称（来自 ``LinNames``）
   * 导数阶数指示器（``VF_DerivOrder1``、``VF_DerivOrder2``）
   * 旋转坐标系标志（``VF_RotFrame``）
   * 全系统矩阵 **A**、**B**、**C**、**D**、**dUdu**、**dUdy**
   * 各模块矩阵（如果 ``LinOutMod=True``）
   * 完整雅可比矩阵（如果 ``LinOutJac=True``）

输出文件格式
-------------------

每次线性化调用都会生成一个名为 ``<RootName>.<N>.lin`` 的文件，其中 *N* 是线性化索引（1 … ``NLinTimes``）。该文件是纯文本 ASCII 文件，可以通过 `openfast_io <https://github.com/OpenFAST/openfast_io>`_ Python 库或 `pyFAST <https://github.com/OpenFAST/pyFAST>`_ 后处理工具读取。

文件头中的关键字段：

* ``Rotor_Speed``——线性化时的转子转速（RPM）
* ``Azimuth``——线性化时的 1 号叶片方位角（度）

变量命名约定
----------------------------

在线性化输出文件中，每个通道标签遵循以下模式：

``<ModAbbr> <MeshName> <Field> [, component [, node [, unit]]]``

示例：

* ``ED BlPitch1, rad``——ElastoDyn 独立 1 号叶片变桨状态
* ``AD B1N001Fx force, node 1, N``——AeroDyn 1 号叶片 1 号节点 X 向力输入
* ``BD_1 B1TipTDxr translation displacement, node 10, m``——BeamDyn 实例 1

模块开发者应确保 ``MV_AddVar`` / ``MV_AddMeshVar`` 的 ``Name`` 参数以及 ``LinNames`` 中的条目遵循此约定，以与后处理工具保持一致。

模块开发者职责
-----------------------------------

要参与线性化，模块必须：

1. 调用 ``MV_AddVar`` / ``MV_AddMeshVar`` 时设置适当的 ``VF_Linearize`` 标志，并为可能出现在标准线性化集中的所有变量提供 ``LinNames``。

2. 实现 ``<Mod>_JacobianPInput`` 和 ``<Mod>_JacobianPContState`` 子程序（或通过注册提供解析雅可比）。耦合代码通过 ``FAST_Funcs.f90`` 中的 ``FAST_JacobianPInput`` / ``FAST_JacobianPContState`` 包装器调用这些子程序。

3. 实现 ``<Mod>_GetOP``（通过注册）以提取状态、输入和输出的工作点值。

4. 用 ``VF_NoLin`` 标记**不**应参与线性化的变量。

5. 用 ``VF_RotFrame`` 标记旋转参考系中的变量，以便后处理工具应用的多叶片坐标（MBC）变换能够识别这些变量。
