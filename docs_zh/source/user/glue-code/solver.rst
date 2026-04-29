.. _glue-code-solver:

求解器
======

OpenFAST 紧耦合求解器实现于 ``modules/openfast-library/src/FAST_Solver.f90``。它使用带 Newton-Raphson 收敛迭代的广义-α 方案，积分连续状态并解析模块间的输入-输出耦合。

.. contents::
   :local:
   :depth: 2

.. _glue-code-solver-inputs:

用户输入参数
---------------------

所有求解器参数都在主 OpenFAST 输入文件（``*.fst``）的**功能开关与标志**和**紧耦合/求解器**部分设置。

.. list-table::
   :header-rows: 1
   :widths: 22 12 66

   * - 参数
     - 类型
     - 描述
   * - ``DT``
     - 实数
     - 全局（求解器）时间步长，单位为秒。所有模块的时间步长必须等于 ``DT`` 或为其整数约数。
   * - ``ModCoupling``
     - 整数
     - 耦合方法。

       * ``1``——松耦合：结构模块（ED/BD/SD）被视为选项 1，**不**参与紧牛顿循环。
       * ``2``——带固定雅可比更新的紧耦合（``DT_UJac`` 控制更新频率）。
       * ``3``——带自适应雅可比更新的紧耦合（每当牛顿循环在迭代预算内无法收敛时，重建雅可比矩阵）。
   * - ``RhoInf``
     - 实数
     - 广义-α 积分器的数值阻尼参数 ρ∞。范围 [0, 1]；1 = 无数值阻尼（二阶精度），0 = 最大阻尼（一阶精度）。典型值：**0.9**。将 ``RhoInf`` 降低到 1 以下会阻尼高频数值噪声，但代价是精度略有降低。
   * - ``MaxConvIter``
     - 整数
     - 每个时间步的最大牛顿收敛迭代次数，超过则求解器宣告收敛失败。典型值：**20**。当 ``ModCoupling=2`` 或 ``1`` 时，失败会触发致命错误；当 ``ModCoupling=3`` 时，会先重建雅可比矩阵并重试该步，然后发出警告。
   * - ``ConvTol``
     - 实数
     - 收敛容差。当牛顿更新向量的平均 `L2` 范数低于此值时，迭代停止。典型值：``1.0e-4``。更严格的容差会增加计算成本，但对于刚性问题可能需要。
   * - ``DT_UJac``
     - 实数
     - 当 ``ModCoupling=2`` 时，雅可比矩阵重建的时间间隔（秒）。

       * 如果 ``DT_UJac < DT``：在收敛迭代预算的一部分时重建雅可比矩阵。
       * 如果 ``DT_UJac ≥ DT``：每 ``CEILING(DT_UJac/DT)`` 个时间步重建一次雅可比矩阵。
       * 将 ``DT_UJac`` 设置为非常大的值（例如 ``9999``）会在整个仿真期间冻结雅可比矩阵；这对于性能分析或系统接近线性且雅可比计算代价高昂时很有用。
   * - ``UJacSclFact``
     - 实数
     - 应用于雅可比矩阵载荷行和列的条件缩放因子。力和力矩变量在线性求解前除以该因子，求解后乘回，使载荷条目的量级与位移/速度条目相等。典型值：海上系统为 **1.0e5**；对于非常大或非常小的风机可能需要调整。
   * - ``CompElast``
     - 整数
     - 选择结构动力学模块：``1`` = ElastoDyn，``2`` = BeamDyn（仅叶片，ElastoDyn 仍处理塔筒/平台），``3`` = 简化 ElastoDyn。当 ``ModCoupling ≥ 2`` 时，所选模块成为 TC 成员。
   * - ``CompSub``
     - 整数
     - 子结构模块：``0`` = 无，``1`` = SubDyn，``2`` = ExtPtfm，``3`` = SlD（SoilDyn）。当 ``ModCoupling ≥ 2`` 时，SubDyn 加入 TC 集合。
   * - ``CompHydro``
     - 整数
     - ``0`` = 无，``1`` = HydroDyn。HydroDyn 始终为选项 1。
   * - ``CompMooring``
     - 整数
     - ``0`` = 无，``1`` = MAP++，``2`` = FEAMooring，``3`` = MoorDyn，``4`` = OrcaFlex。系泊模块始终为选项 1。
   * - ``CompAero``
     - 整数
     - 空气动力学模块：``0`` = 无，``1`` = AeroDisk，``2`` = AeroDyn。对于陆基风机，AeroDyn 为选项 2；对于 MHK，为选项 1。
   * - ``CompServo``
     - 整数
     - 控制器模块：``0`` = 无，``1`` = ServoDyn。ServoDyn 默认为求解后阶段，但当结构控制器（塔筒、叶片、机舱 StC）激活时成为选项 1。

广义-α 积分
------------------------------

紧耦合求解器积分以下形式的二阶 ODE：

.. math::

   \mathbf{M}\,\ddot{\mathbf{q}} + \mathbf{f}(\mathbf{q}, \dot{\mathbf{q}}, t) = 0

使用**广义-α 方法**（Chung & Hulbert, 1993）。根据 ``RhoInf`` 指定的谱半径 ρ∞，方法参数为：

.. math::

   \alpha_m &= \frac{2\rho_\infty - 1}{\rho_\infty + 1} \\
   \alpha_f &= \frac{\rho_\infty}{\rho_\infty + 1} \\
   \gamma   &= \tfrac{1}{2} - \alpha_m + \alpha_f \\
   \beta    &= \tfrac{1}{4}(1 - \alpha_m + \alpha_f)^2

收敛循环中使用的两个导出系数为：

.. math::

   \beta'   &= h^2 \beta \frac{1 - \alpha_f}{1 - \alpha_m} \\
   \gamma'  &= h \gamma  \frac{1 - \alpha_f}{1 - \alpha_m}

其中 *h* = ``DT``。

**状态向量布局**——求解器为每个模块维护一个*广义坐标*（q）向量，包含四列：

.. list-table::
   :header-rows: 1
   :widths: 15 85

   * - 列
     - 含义
   * - ``q``
     - 位移/方向状态（``DerivOrder = 0``）
   * - ``v``
     - 速度状态（``DerivOrder = 1``）
   * - ``vd``
     - 加速度（物理值，来自模块的 ``CalcContStateDeriv``）
   * - ``a``
     - 算法加速度（广义-α 内部变量）

每个时间步开始时的状态预测：

.. math::

   q_{n+1}^{\rm pred}  &= q_n + h v_n + h^2[(\tfrac{1}{2} - \beta)a_n + \beta\, a_{n+1}] \\
   v_{n+1}^{\rm pred}  &= v_n + h[(1-\gamma)a_n + \gamma\, a_{n+1}]

模块排序
---------------

在 ``FAST_SolverInit`` 期间，每个模块根据 ``ModCoupling`` 及其自身物理类型进行分类，并分配到 ``Glue_TCParam`` 结构中的一个有序索引数组：

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 数组
     - 模块（按顺序）
   * - ``iModTC``
     - ElastoDyn、BeamDyn、SubDyn（当 ``ModCoupling ≥ 2`` 时）
   * - ``iModOpt1``
     - ServoDyn（当 StC 激活时）、SED、AD（MHK）、ExtPtfm、HydroDyn、OrcaFlex、MoorDyn；当 ``ModCoupling = 1`` 时，ED/BD/SD 也出现在这里
   * - ``iModOpt2``
     - ServoDyn、SED、ED、BD、SD、InflowWind、SeaState、AeroDyn（陆基）、AeroDisk、ExtLoads、MAP++、FEAMooring、IceDyn、IceFloe、SoilDyn
   * - ``iModPost``
     - ServoDyn、ExternalInflow
   * - ``iModInit``
     - SED、ED、BD、SD、InflowWind、ExtLoads（仅第 0 步初始化）

雅可比矩阵构建
---------------------

需要组装两个独立的雅可比矩阵：

1. **TC/选项1 雅可比矩阵**（``BuildJacobianTC``）——用于主时间步进收敛循环。
2. **IO 雅可比矩阵**（``BuildJacobianIO``）——用于初始和线性化输入-输出求解。

变量选择（``VF_Solve`` 标志）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在 ``FAST_SolverInit → SetVarSolveFlags`` 期间，必须出现在雅可比矩阵中的变量会被设置 ``VF_Solve`` 标志：

* 所有 TC 模块的**连续状态**（自动设置）。
* TC 到 TC 映射的**运动网格**输入/输出（所有场）。
* TC 到选项1 或选项1 到 TC 映射的**运动网格**输入加速度。
* 参与任何 TC/选项1 映射的**载荷网格**输入和输出。
* 当映射传递力矩时，目标模块的**载荷网格**位移输出（力矩臂雅可比项需要）。
* TC/选项1 模块的**变量到变量**映射输入/输出。
* 任何带有 ``VF_NoLin`` 的变量会被排除在 ``VF_Solve`` 之外。

雅可比结构（TC 雅可比）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

组装后的 TC 雅可比矩阵 **J** 大小为 ``NumJ × NumJ``，其中：

.. math::

   N_J = \underbrace{N_Q}_{\text{TC 状态}} +
         \underbrace{N_{U_T}}_{\text{TC 输入}} +
         \underbrace{N_{U_1}}_{\text{选项1 输入}}

列和行的分区如下：

.. math::

   \mathbf{J} = \begin{bmatrix}
     J_{11} & J_{12} \\
     J_{21} & J_{22}
   \end{bmatrix}

其中：

* **J₁₁**（``NumQ × NumQ``）——加速度残差相对于 TC 位移/速度状态的导数（由模块 ``dXdx`` 子雅可比加广义-α 切线组成）。
* **J₁₂**（``NumQ × NumU_T``）——加速度残差相对于 TC 输入的导数（来自 ``dXdu``）。
* **J₂₁**（``NumU_T × NumQ``）——输入残差相对于 TC 状态的导数（来自 ``dUdx = dUdy · dydx``）。
* **J₂₂**（``NumU × NumU``）——输入残差相对于输入的导数，包括载荷条件行/列。

右侧向量（XB）包含残差：

* **状态残差**（行 ``iJX``）：预测的速度导数与模块计算的加速度之间的差值。
* **输入残差**（行 ``iJU``）：从网格映射计算的输入（``FAST_InputSolve``）与当前迭代值之间的差值。

载荷部分（行 ``iJL``）在分解前会预先除以 ``UJacSclFact`` 以改善条件数。

雅可比更新策略
~~~~~~~~~~~~~~~~~~~~~~~~

``ModCoupling = 2``（固定更新）
   当以下任一计数器归零时，重建雅可比矩阵：

   * ``UJacStepsRemain``——剩余步数；每次重建雅可比矩阵时初始化为 ``CEILING(DT_UJac/DT)``。
   * ``UJacIterRemain``——迭代预算；当 ``DT_UJac < DT`` 时初始化为 ``CEILING(DT_UJac/DT · MaxConvIter)``。

   收敛失败时，求解器会立即返回致命错误。

``ModCoupling = 3``（自适应更新）
   收敛循环第一次失败时重建雅可比矩阵。如果强制重建后该步仍不收敛，发出非致命警告并继续仿真。

各模块雅可比贡献
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

模块级雅可比子矩阵在 ``BuildJacobianTC`` 和 ``BuildJacobianIO`` 中通过有限差分计算，使用 ``ModVar`` 中的 ``MV_Perturb`` / ``MV_ComputeDiff`` / ``MV_ComputeCentralDiff`` 工具。对于每个标记为 ``VF_Solve`` 的变量：

1. 对工作状态/输入数组施加大小为 ``Var%Perturb`` 的正扰动。
2. 调用 ``FAST_CalcOutput``（或 ``FAST_GetContStateDeriv``）。
3. 施加相等的负扰动。
4. 再次调用。
5. 计算中心差分：``(y_plus - y_minus) / (2·Perturb)``。

对于方向变量（``FieldOrientation``），扰动通过四元数合成而非直接加法施加（``MV_Perturb``），差值作为旋转向量提取（``MV_ComputeDiff``）。

线性求解
~~~~~~~~~~~~

**J** 的 LU 分解使用 ``LAPACK_getrf`` 计算，系统求解使用 ``LAPACK_getrs``（封装在 ``NWTC_LAPACK`` 中）。相同的分解矩阵会在收敛迭代中重用，直到更新策略触发重建。

收敛检查
~~~~~~~~~~~~~~~~~

每次牛顿步后，收敛误差为更新向量的平均 `L2` 范数：

.. math::

   e = \frac{\|\Delta \mathbf{z}\|_2}{N_J}

其中 :math:`\Delta \mathbf{z}` 组合了状态和输入更新。如果 ``e < ConvTol``（``ErrID_None``）或迭代次数达到 ``MaxConvIter``（根据 ``ModCoupling`` 为 ``ErrID_Fatal`` 或 ``ErrID_Warn``），循环退出。

求解器输出通道
--------------------------------

每个时间步会将三个输出通道写入 ``DriverWriteOutput``，启用后会出现在输出文件中：

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 索引
     - 内容
   * - 1
     - 该步的总收敛迭代次数（``TotalIter``）
   * - 2
     - 最终收敛误差（``ConvError``）
   * - 3
     - 该步的雅可比矩阵重建次数（``NumUJac``）
