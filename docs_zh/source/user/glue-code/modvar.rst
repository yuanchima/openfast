.. _glue-code-modvar:

模块变量（``ModVar``）
=====================

``ModVar`` 模块（位于 ``modules/nwtc-library/src/ModVar.f90``）提供了数据结构和子程序，允许每个物理模块声明其连续状态、输入和输出，使得耦合代码可以通用地操作这些变量，而无需了解任何特定模块的内部结构。

完整的类型层次结构——从 ``DatLoc`` 和 ``ModVarType`` 到 ``ModVarsType`` 和 ``ModDataType``，再到顶层的 ``ModGlueType``——如 :numref:`fig-modvar-types` 所示。

.. contents::
   :local:
   :depth: 2

数据结构
--------

``DatLoc``
~~~~~~~~~~

``DatLoc``（数据位置）是一个小型结构，用于唯一标识特定变量在模块派生类型层次结构中的位置：

.. code-block:: fortran

   TYPE :: DatLoc
     INTEGER :: Num   ! Data identification number (from _Types.f90 file)
     INTEGER :: i1    ! First index
     INTEGER :: i2    ! Second index
     INTEGER :: i3    ! Third index
     INTEGER :: i4
     INTEGER :: i5
   END TYPE DatLoc

每个变量的 ``DatLoc`` 值由注册表生成的 ``DatLoc()`` 构造函数在每个模块的 ``InitVars`` 子程序中创建一次，并传递给 ``MV_AddVar`` / ``MV_AddMeshVar``。耦合代码将此值存储在每个 ``ModVarType`` 中，并在设置网格映射时使用 ``MV_EqualDL`` 来匹配源模块和目标模块之间的变量。

``ModVarType``
~~~~~~~~~~~~~~

描述单个变量（或网格场的一组变量）：

.. code-block:: fortran

   TYPE :: ModVarType
     INTEGER      :: Field       ! Field type (FieldForce, FieldTransDisp, ...)
     INTEGER      :: Nodes       ! Number of nodes (mesh variables only)
     INTEGER      :: Num         ! Total number of scalar values
     INTEGER      :: Flags       ! Bit-mask of VF_* flags
     INTEGER      :: DerivOrder  ! 0=disp/orientation, 1=velocity, 2=acceleration
     INTEGER      :: iLoc(2)     ! [start, end] in module-local array
     INTEGER      :: iGlu(2)     ! [start, end] in glue-level array
     INTEGER      :: iq(2)       ! [start, end] in solver state (q) array
     INTEGER      :: iLB, iUB    ! User array bounds (for array-valued scalars)
     INTEGER      :: j, k, m, n  ! Additional user-defined indices
     REAL(R8Ki)   :: Perturb     ! Perturbation size for Jacobian finite differences
     TYPE(DatLoc) :: DL          ! Data location
     CHARACTER    :: Name        ! Human-readable variable name
     CHARACTER(:), ALLOCATABLE :: LinNames(:)  ! Per-value linearization labels
   END TYPE ModVarType

``ModVarsType``
~~~~~~~~~~~~~~~

保存一个模块的所有变量，分为三组：

.. code-block:: fortran

   TYPE :: ModVarsType
     INTEGER                             :: Nx, Nu, Ny
     TYPE(ModVarType), ALLOCATABLE :: x(:)   ! Continuous states
     TYPE(ModVarType), ALLOCATABLE :: u(:)   ! Inputs
     TYPE(ModVarType), ALLOCATABLE :: y(:)   ! Outputs
   END TYPE ModVarsType

``ModDataType``
~~~~~~~~~~~~~~~

耦合代码为每个模块实例保存的顶层容器：

.. code-block:: fortran

   TYPE :: ModDataType
     CHARACTER    :: Abbr      ! Module abbreviation ("ED", "BD", ...)
     INTEGER      :: iMod      ! Index in glue module array
     INTEGER      :: ID        ! Module_ED, Module_BD, ...
     INTEGER      :: Ins       ! Instance number
     INTEGER      :: iRotor    ! Rotor index (0 = all rotors)
     INTEGER      :: SubSteps  ! Module sub-steps per solver step
     INTEGER      :: Category  ! Bit-mask of MC_* coupling flags
     REAL(R8Ki)   :: DT        ! Module time step
     TYPE(ModVarsType)  :: Vars
     TYPE(ModLinType)   :: Lin
   END TYPE ModDataType

变量标志（``VF_*``）
---------------------

标志通过 ``IOR`` 组合，并使用 ``MV_HasFlagsAll`` / ``MV_HasFlagsAny`` 进行测试。

.. list-table::
   :header-rows: 1
   :widths: 25 10 65

   * - 标志
     - 值
     - 含义
   * - ``VF_None``
     - 0
     - 未设置标志；用作匹配任何变量的通配符。
   * - ``VF_Mesh``
     - 1
     - 变量是网格场（由 ``MV_AddMeshVar`` 自动设置）。
   * - ``VF_Line``
     - 2
     - 网格是 *线* 网格（单位长度载荷）；线性化标签会添加 ``/m`` 后缀。
   * - ``VF_RotFrame``
     - 4
     - 变量位于旋转参考系中。
   * - ``VF_Linearize``
     - 8
     - 变量包含在全系统线性化中。
   * - ``VF_ExtLin``
     - 16
     - 变量包含在扩展线性化输出中。
   * - ``VF_2PI``
     - 32
     - 范围为 [0, 2π] 的标量角度（例如发电机方位角）。
   * - ``VF_WriteOut``
     - 64
     - 与 ``WriteOutput`` 通道关联的输出变量。
   * - ``VF_Solve``
     - 256
     - 变量参与紧耦合雅可比或输入输出收敛求解。由 ``FAST_SolverInit`` 自动设置；模块开发者不应手动设置此标志。
   * - ``VF_AeroMap``
     - 512
     - 用于气动图计算的变量。
   * - ``VF_Mapping``
     - 1024
     - 变量参与模块间传递映射。
   * - ``VF_NoLin``
     - 8192
     - 明确将变量排除在线性化和求解器之外（覆盖 ``VF_Linearize`` 和 ``VF_Solve``）。

场类型（``Field*``）
--------------------

用于 ``ModVarType`` 的 ``Field`` 成员和 ``MV_AddMeshVar`` 的 ``Fields`` 参数中：

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - 常量
     - 含义
   * - ``FieldForce``
     - 节点力（每个节点 3 个分量，N）
   * - ``FieldMoment``
     - 节点力矩（每个节点 3 个分量，N·m）
   * - ``FieldTransDisp``
     - 平动位移（m）
   * - ``FieldOrientation``
     - 方向，内部存储为单位四元数参数（rad）
   * - ``FieldTransVel``
     - 平动速度（m/s）
   * - ``FieldAngularVel``
     - 角速度（rad/s）
   * - ``FieldTransAcc``
     - 平动加速度（m/s²）
   * - ``FieldAngularAcc``
     - 角加速度（rad/s²）
   * - ``FieldAngularDisp``
     - 角位移（rad）
   * - ``FieldScalar``
     - 通用标量值

在 ``ModVar.f90`` 中定义的便捷数组：

* ``LoadFields``   = ``[FieldForce, FieldMoment]``（载荷场）
* ``TransFields``  = ``[FieldTransDisp, FieldTransVel, FieldTransAcc]``（平动场）
* ``AngularFields``= ``[FieldOrientation, FieldAngularVel, FieldAngularAcc, FieldAngularDisp]``（转动场）
* ``MotionFields`` = 所有平动和转动运动场

添加模块变量
------------

每个参与耦合代码的模块必须实现一个 ``InitVars``（或等效的）子程序，通过调用下面记录的 ``MV_Add*`` 子程序来填充 ``ModVarsType`` 结构。该子程序在模块的 ``_Init`` 例程中被调用，生成的 ``Vars`` 会立即传递给 ``MV_AddModule``。

``MV_AddVar``
~~~~~~~~~~~~~

向变量数组添加单个（可能是多元素的）标量变量。

.. code-block:: fortran

   subroutine MV_AddVar(VarAry, Name, Field, DL, &
                        Num, iAry, jAry, kAry, &
                        Flags, DerivOrder, Perturb, LinNames, Active)

.. list-table::
   :header-rows: 1
   :widths: 20 10 70

   * - 参数
     - 意图
     - 描述
   * - ``VarAry``
     - ``INOUT``
     - 可分配的 ``ModVarType`` 数组；新变量会追加到其中。
   * - ``Name``
     - ``IN``
     - 用于调试输出和线性化标签的人类可读名称。
   * - ``Field``
     - ``IN``
     - 场类型常量（``FieldScalar``、``FieldTransDisp`` 等）。
   * - ``DL``
     - ``IN``
     - 标识数据在模块派生类型中位置的 ``DatLoc``。
   * - ``Num``
     - ``IN``（可选）
     - 标量值的数量。默认为 1。如果为 0，调用不执行任何操作。
   * - ``iAry``
     - ``IN``（可选）
     - 如果数据存储在数组中，起始下界索引。
   * - ``jAry``、``kAry``
     - ``IN``（可选）
     - 第二和第三数组索引（用于 2D 或 3D 数组）。
   * - ``Flags``
     - ``IN``（可选）
     - 初始 ``VF_*`` 标志位掩码。默认为 ``VF_None``。
   * - ``DerivOrder``
     - ``IN``（可选）
     - 覆盖自动推断的导数阶数（0、1 或 2）。
   * - ``Perturb``
     - ``IN``（可选）
     - 有限差分扰动幅度。一个好的默认值大约是变量预期量级的 1%。对于网格场，如果省略此参数，则使用 ``ModVarType_Init`` 内部计算的默认值。
   * - ``LinNames``
     - ``IN``（可选）
     - 每个值的线性化通道标签数组（长度 = ``Num``）。**对于非网格、非标量变量是必需的**。
   * - ``Active``
     - ``IN``（可选）
     - 设置为 ``.false.`` 时有条件地跳过添加变量。

**示例**——注册 ServoDyn 的发电机转矩输入：

.. code-block:: fortran

   call MV_AddVar(Vars%u, 'Generator torque command', FieldScalar, &
                  DatLoc(SrvD_u_GenTrq), &
                  Flags=VF_Linearize, &
                  Perturb=1.0e3_R8Ki, &        ! N*m
                  LinNames=['SrvD GenTrq, N*m'])

``MV_AddMeshVar``
~~~~~~~~~~~~~~~~~

将单个 ``MeshType`` 的所有请求网格场添加到变量数组中。它是 ``MV_AddVar`` 的便捷包装器，会遍历 ``Fields`` 参数并跳过已提交网格中不存在的场。

.. code-block:: fortran

   subroutine MV_AddMeshVar(VarAry, Name, Fields, DL, Mesh, &
                            Flags, Perturbs, Active, iVar)

.. list-table::
   :header-rows: 1
   :widths: 20 10 70

   * - 参数
     - 意图
     - 描述
   * - ``VarAry``
     - ``INOUT``
     - 要追加新变量的可分配数组。
   * - ``Name``
     - ``IN``
     - 该网格上所有场的基名。
   * - ``Fields``
     - ``IN``
     - 场类型常量的整数数组；在适当的情况下使用 ``LoadFields``、``MotionFields`` 等便捷参数。
   * - ``DL``
     - ``IN``
     - 该网格在模块数据类型中的 ``DatLoc``。
   * - ``Mesh``
     - ``INOUT``
     - 已提交的 ``MeshType``。其 ``ID`` 字段被设置以在网格映射操作中标识它。如果网格尚未提交，子程序会直接返回而不添加任何内容。
   * - ``Flags``
     - ``IN``（可选）
     - 在 ``VF_Mesh`` 基础上添加的额外 ``VF_*`` 标志。
   * - ``Perturbs``
     - ``IN``（可选）
     - 扰动值数组，每个对应 ``Fields`` 中的一个条目。
   * - ``Active``
     - ``IN``（可选）
     - 有条件地禁用整个网格变量注册。
   * - ``iVar``
     - ``OUT``（可选）
     - 返回分配的网格 ``ID``，以便调用者存储它供后续字段查找使用。

**示例**——注册 ElastoDyn 的叶根输出运动网格：

.. code-block:: fortran

   call MV_AddMeshVar(Vars%y, 'BladeRootMotion', MotionFields, &
                      DatLoc(ED_y_BladeRootMotion, i), &   ! blade i
                      p%BladeRootMotion(i), &
                      Flags=VF_Linearize + VF_RotFrame)

``MV_AddModule``
~~~~~~~~~~~~~~~~

模块的 ``InitVars`` 子程序完成后，调用者使用 ``MV_AddModule`` 将模块注册到耦合代码。

.. code-block:: fortran

   subroutine MV_AddModule(ModDataAry, ModID, ModAbbr, Instance, &
                           ModDT, SolverDT, Vars, Linearize, &
                           ErrStat, ErrMsg, iRotor)

.. list-table::
   :header-rows: 1
   :widths: 20 10 70

   * - 参数
     - 意图
     - 描述
   * - ``ModDataAry``
     - ``INOUT``
     - 可分配的 ``ModDataType`` 数组；新条目会追加到其中。
   * - ``ModID``
     - ``IN``
     - 模块标识符常量（``Module_ED``、``Module_BD`` 等）。
   * - ``ModAbbr``
     - ``IN``
     - 用于输出标签的简短缩写字符串（``"ED"``、``"BD"``）。
   * - ``Instance``
     - ``IN``
     - 实例号（从 1 开始）。大多数模块只有一个实例。
   * - ``ModDT``
     - ``IN``
     - 模块时间步长（秒）。必须是 ``SolverDT`` 的精确整数约数。
   * - ``SolverDT``
     - ``IN``
     - 求解器（全局）时间步长。
   * - ``Vars``
     - ``IN``
     - 来自模块 ``InitVars`` 调用的已填充 ``ModVarsType``。
   * - ``Linearize``
     - ``IN``
     - 是否启用线性化。当为 ``.false.`` 时，``LinNames`` 数组会被释放以节省内存。
   * - ``ErrStat`` / ``ErrMsg``
     - ``OUT``
     - 错误状态和消息。
   * - ``iRotor``
     - ``IN``（可选）
     - 多转子风机的转子号（0 = 所有转子）。

**子步进逻辑**：如果 ``ModDT < SolverDT``，``MV_AddModule`` 会计算 ``ModData%SubSteps = NINT(SolverDT/ModDT)`` 并验证模块 DT 是否能精确整除求解器 DT。如果 ``ModDT > SolverDT``，会返回错误。

**FAST_Subs.f90 中的典型调用序列**：

.. code-block:: fortran

   ! Module computes its own Vars in Init
   call ED_Init(InitInp, u, p, ..., InitOut, ErrStat, ErrMsg)
   ! Register with glue code
   call MV_AddModule(m%ModData, Module_ED, 'ED', 1, p%DT, p_FAST%DT, &
                     InitOut%Vars, p_FAST%Linearize, ErrStat, ErrMsg, iRotor=1)

``MV_InitVarsJac``
~~~~~~~~~~~~~~~~~~

在所有 ``MV_AddVar`` / ``MV_AddMeshVar`` 调用完成后，在每个模块的 ``InitVars`` 内部调用。它为每个变量分配模块本地的 ``iLoc`` 索引范围，并分配雅可比计算期间使用的 ``ModJacType`` 工作数组。

.. code-block:: fortran

   subroutine MV_InitVarsJac(Vars, Jac, Linearize, ErrStat, ErrMsg)

扰动值
-----

每个变量都带有一个 ``Perturb`` 值，用于在构建模块级雅可比时进行中心差分有限差分。选择 ``MV_AddVar`` / ``MV_AddMeshVar`` 的 ``Perturb`` 参数时，应使得产生的输出变化大到足以与数值噪声区分开，但又小到足以保持在线性范围内。典型值：

* 平动位移：``1.0e-4`` m
* 转动（方向）：``2.0e-5`` rad
* 平动速度：``1.0e-3`` m/s
* 角速度：``2.0e-4`` rad/s
* 平动加速度：``1.0e-2`` m/s²
* 力：``1.0e1`` N
* 力矩：``1.0e1`` N·m
* 通用标量：取决于上下文

``UJacSclFact`` 输入参数（参见 :ref:`glue-code-solver-inputs`）是一个全局调节因子，当力/力矩的量级与状态量级差异很大时，求解器会将其应用于雅可比中的载荷变量，以改善矩阵的条件数。

方向表示
--------

在耦合变量数组中，方向**不**以方向余弦矩阵（DCM）的形式存储或操作。相反，使用紧凑的三分量单位四元数参数化：

.. math::

   \mathbf{q}_p = [q_1, q_2, q_3] \quad \text{where } q_0 = \sqrt{1 - q_1^2 - q_2^2 - q_3^2}

这种参数化避免了完整 DCM 中的冗余，并可以通过四元数合成（``quat_compose``）实现直接的有限差分。从 ``ModVar`` 导出的转换工具包括 ``dcm_to_quat``、``quat_to_dcm``、``quat_compose``、``quat_inv``、``quat_to_rvec``、``rvec_to_quat``、``wm_to_quat`` 和 ``quat_to_wm``。

在为雅可比行计算方向差异时，``MV_ComputeDiff`` 会计算负扰动和正扰动四元数之间的相对旋转，并将其转换为旋转向量。
