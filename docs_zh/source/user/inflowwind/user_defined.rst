.. _ifw_user_defined:

用户自定义风场
=========================

本节介绍如何在 InflowWind 中使用 ``WindType = 6`` 实现自定义风场。

概述
--------

用户自定义风场功能允许开发者通过以下方式实现自定义风模型：

1. 定义数据结构来存储风场参数
2. 从输入文件或参数初始化该数据结构
3. 实现一个函数，返回任意位置和时刻的风速

该功能适用于：

- 解析风模型（如涡流模型、尾流模型）
- 标准格式无法提供的自定义风廓线
- 与外部风场求解器耦合
- 来自传感器的实时风速测量
- 新型风场表示方法的研究与开发

.. important::
   修改注册表文件（``.txt`` 文件）后，必须重新构建项目以重新生成类型定义文件（``*_Types.f90``）。
   对 ``.txt`` 文件的修改定义了扩展的数据结构，但只有在重新生成后才能使用。

实现步骤
--------------------

步骤 1：定义数据结构
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

编辑 ``modules/inflowwind/src/IfW_FlowField.txt``，向 ``UserFieldType`` 添加字段：

.. code-block:: text

   typedef  ^              UserFieldType          ReKi                RefHeight           -     -         -     "reference height; used to center the wind"                   meters
   typedef  ^              ^                      IntKi               NumDataLines        -     0         -     "number of data lines (for time-varying user wind)"          -
   typedef  ^              ^                      DbKi                DTime               :     -         -     "time array for user-defined wind"                           seconds
   typedef  ^              ^                      ReKi                Data                ::    -         -     "user-defined wind data array [NumDataLines, NumDataColumns]" -
   typedef  ^              ^                      CHARACTER(1024)     FileName            -     -         -     "name of user wind file (if applicable)"                     -

添加风模型实现所需的任何自定义字段。

步骤 2：定义初始化输入
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

编辑 ``modules/inflowwind/src/InflowWind_IO.txt``，向 ``User_InitInputType`` 添加字段：

.. code-block:: text

   typedef  ^              User_InitInputType    CHARACTER(1024)         WindFileName            -     -     -     "name of file containing user-defined wind data (if applicable)" -
   typedef  ^              ^                     ReKi                    RefHt                   -     -     -     "reference height for user wind field"                        meters
   typedef  ^              ^                     IntKi                   NumDataColumns          -     0     -     "number of data columns in user wind file (if applicable)"    -

添加初始化风模型所需的任何参数。

步骤 3：重新生成类型文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

修改注册表文件后，重新构建项目以重新生成类型定义。在 Windows 上使用 Visual Studio 构建 OpenFAST 时，
类型文件会自动重新生成。使用 CMake 时，运行以下命令以启用类型文件的生成。

.. code-block:: bash

   cd build
   cmake .. -DGENERATE_TYPES=ON
   make

构建过程会根据 ``.txt`` 注册表文件自动重新生成 ``*_Types.f90`` 文件。

步骤 4：实现初始化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

编辑 ``modules/inflowwind/src/InflowWind_IO.f90``，实现 ``IfW_User_Init()``：

.. code-block:: fortran

   subroutine IfW_User_Init(InitInp, SumFileUnit, UF, FileDat, ErrStat, ErrMsg)
      ! Initialize UF%RefHeight, read data files, allocate arrays
      ! Set FileDat metadata (wind type, time range, spatial extent, etc.)
      ! Write summary information to SumFileUnit if > 0
   end subroutine

该例程完成以下工作：

- 读取 ``InitInp`` 中指定的必要输入文件
- 分配并填充 ``UserFieldType``（``UF``）数据结构
- 在 ``WindFileDat`` 结构中设置适当的元数据
- 将初始化信息写入汇总文件

步骤 5：实现速度函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

编辑 ``modules/inflowwind/src/IfW_FlowField.f90``，实现 ``UserField_GetVel()``：

.. code-block:: fortran

   subroutine UserField_GetVel(UF, Time, Position, Velocity, ErrStat, ErrMsg)
      ! Use UF data to compute velocity at Position and Time
      ! Position(1) = X, Position(2) = Y, Position(3) = Z (meters)
      ! Return Velocity(1) = U, Velocity(2) = V, Velocity(3) = W (m/s)
   end subroutine

该函数在仿真过程中为每个需要风速的位置调用。

坐标系
------------------

输入坐标（Position）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **X**：下游方向（由 InflowWind 施加旋转后）
- **Y**：横向/侧风方向
- **Z**：垂直方向（从地面起算，Z=0 为地面）
- **单位**：m

输出速度（Velocity）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **U**：沿 X 方向的速度分量（正 = 顺风向）
- **V**：沿 Y 方向的速度分量（朝下游看时正 = 向左）
- **W**：沿 Z 方向的速度分量（正 = 向上）
- **单位**：m/s

.. note::
   InflowWind 处理全局坐标系与风坐标系之间的旋转。
   您的实现应在风坐标系中工作，其中 X 轴与平均风向对齐。

示例实现
----------------------

幂律风廓线
~~~~~~~~~~~~~~~~~~~~~~

本示例实现了一个简单的幂律风廓线。

**速度函数**（位于 ``IfW_FlowField.f90``）：

.. code-block:: fortran

   subroutine UserField_GetVel(UF, Time, Position, Velocity, ErrStat, ErrMsg)
      type(UserFieldType), intent(in)     :: UF
      real(DbKi), intent(in)              :: Time
      real(ReKi), intent(in)              :: Position(3)
      real(ReKi), intent(out)             :: Velocity(3)
      integer(IntKi), intent(out)         :: ErrStat
      character(*), intent(out)           :: ErrMsg

      real(ReKi)                          :: RefSpeed, Exponent, Height

      ErrStat = ErrID_None
      ErrMsg = ""

      ! Get reference speed and exponent from UF%Data
      RefSpeed = UF%Data(1, 1)  ! Reference wind speed (m/s)
      Exponent = UF%Data(1, 2)  ! Power law exponent
      Height = Position(3)       ! Height above ground

      ! Apply power law: U(z) = Uref * (z/zref)^alpha
      if (Height > 0.0_ReKi) then
         Velocity(1) = RefSpeed * (Height / UF%RefHeight)**Exponent
         Velocity(2) = 0.0_ReKi  ! No lateral wind
         Velocity(3) = 0.0_ReKi  ! No vertical wind
      else
         Velocity = 0.0_ReKi  ! Below ground
      end if

   end subroutine

**初始化**（位于 ``InflowWind_IO.f90``）：

.. code-block:: fortran

   subroutine IfW_User_Init(InitInp, SumFileUnit, UF, FileDat, ErrStat, ErrMsg)
      ! ... (declarations)

      ErrStat = ErrID_None
      ErrMsg = ""

      ! Set reference height
      UF%RefHeight = InitInp%RefHt

      ! Allocate data array for [RefSpeed, Exponent]
      UF%NumDataLines = 1
      call AllocAry(UF%Data, 1, 2, 'User wind data', TmpErrStat, TmpErrMsg)
      call SetErrStat(TmpErrStat, TmpErrMsg, ErrStat, ErrMsg, RoutineName)
      if (ErrStat >= AbortErrLev) return

      ! Set values (could read from file instead)
      UF%Data(1, 1) = 10.0_ReKi  ! 10 m/s reference speed
      UF%Data(1, 2) = 0.2_ReKi   ! Power law exponent

      ! Set metadata
      FileDat%WindType = 6
      FileDat%RefHt = UF%RefHeight
      FileDat%MWS = UF%Data(1, 1)
      FileDat%RefHt_Set = .true.
      ! ... (set other FileDat fields as needed)

   end subroutine

常见用例
----------------

稳态解析风场
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

将风定义为仅随位置变化的函数（忽略 ``Time`` 参数）。

**示例**：对数风廓线、涡流风场、带切变的均匀流。

时变风场
~~~~~~~~~~~~~~~~~~~~~~~

将风数据存储在时间序列数组中，根据 ``Time`` 参数进行插值。

**示例**：实测风数据、指定风瞬变、尾流模型。

来自外部求解器的风场
~~~~~~~~~~~~~~~~~~~~~~~~~~

调用外部函数获取瞬时风场。

**示例**：CFD 耦合、外部尾流模型、指定湍流。

实时传感器数据
~~~~~~~~~~~~~~~~~~~~~

加载来自传感器的实测风数据，并在空间/时间上进行插值。

**示例**：LIDAR 测量数据、气象桅杆数据、现场测量数据。

限制与注意事项
-------------------------------

当前限制
~~~~~~~~~~~~~~~~~~~

1. **不支持加速度**：用户自定义风场目前不支持某些模块（如 MHK 水轮机）所需的加速度计算。

性能注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``UserField_GetVel()`` 在每个时间步的每个点都会被调用
- 实现应高效；尽可能在 ``IfW_User_Init()`` 中预计算
- 对复杂计算考虑缓存或插值策略

错误处理
~~~~~~~~~~~~~~

- 始终在 ``IfW_User_Init()`` 中验证输入参数
- 在 ``UserField_GetVel()`` 中检查数组边界
- 验证 Position 和 Time 值在有效范围内
- 使用 ``SetErrStat()`` 适当报告错误

最佳实践
--------------

1. **从简单开始**：先使用解析模型，再实现复杂的风场

2. **详细文档**：添加详细注释，说明实现方式和任何文件格式

3. **使用 SI 单位**：始终使用 m、s 和 m/s

4. **预计算**：尽可能在初始化时计算，而非运行时计算

5. **验证**：在用于生产之前，先用已知的解析解进行测试

6. **处理边界**：对有效域外的点实现适当的行为

7. **报告元数据**：正确填充 ``WindFileDat`` 中的时间范围、空间范围等

文件位置
--------------

==========================================  ==========================================================
文件                                        用途
==========================================  ==========================================================
``modules/inflowwind/src/``                 源代码目录
``IfW_FlowField.txt``                       流场数据结构的类型定义
``InflowWind_IO.txt``                       初始化输入的类型定义
``IfW_FlowField.f90``                       流场实现（``UserField_GetVel()``）
``InflowWind_IO.f90``                       初始化实现（``IfW_User_Init()``）
``IfW_FlowField_Types.f90``                 自动生成的类型定义（从 .txt 重新生成）
``InflowWind_IO_Types.f90``                 自动生成的类型定义（从 .txt 重新生成）
==========================================  ==========================================================

其他资源
--------------------

- 获取一般 InflowWind 信息，请参阅原始的 :download:`InflowWind 手册 <InflowWind_Manual.pdf>`
- 参考源代码中现有的风场实现（Uniform、Grid3D）
- 查看 :ref:`ifw_appendix` 获取输入文件示例
- 关于数组分配和错误处理工具，请参阅 NWTC Library 文档
- 关于风坐标系和旋转的信息，请参阅 :ref:`ifw_angles`
