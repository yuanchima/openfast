.. _StC-Input:

输入文件
===========

用户通过单独的输入文件配置每个 StC 实例。该输入文件定义 StC
相对于其安装位置的位置以及属性。它还可以与外部力文件一起使用，
在指定位置施加时间序列载荷（主要用于诊断目的）。


单位
-----

结构控制使用 SI 单位制（kg、m、s、N）。除非另有说明，角度默认
以弧度为单位。


.. raw:: html

   <hr>



.. _StC-Locations:

结构控制位置
----------------------------

结构控制输入文件定义 StC 实例的位置和属性。
位置相对于 ServoDyn 主输入文件中给出的 StC 类型（参见 :numref:`SrvD-StC-Inputs`）。
四种位置类型为：机舱（Nacelle）、塔筒（Tower）、叶片（Blade）和浮式平台（Platform）。

StC 的映射信息将在 OpenFAST 主汇总文件中给出。


机舱 StC
~~~~~~~~~~~

此 StC 安装位置相对于机舱参考点连接。
它将跟随所有机舱运动（包括偏航、塔筒弯曲和浮式平台运动
引起的运动）。


塔筒 StC
~~~~~~~~~

此 StC 安装位置连接到塔筒网格上，位于塔筒基底以上指定的高度。
此 StC 连接将随该高度的线网格一起运动。
例如，安装在 90 m 塔筒上 85 m 处的 StC 将跟随
塔筒中心线上对应 85 m 高度位置的网格线运动。


叶片 StC
~~~~~~~~~

此 StC 安装位置沿叶片 z 轴（IEC 叶片坐标系）在距叶片根部指定距离处
连接到叶片结构中心。此位置将跟随所有叶片变形和
运动（包括使用 BeamDyn 时的叶片扭转）。此选项适用于
BeamDyn 和 ElastoDyn 两种叶片表示形式。

使用此选项时，每个叶片上都将连接相同的 StC。
每个叶片安装的 StC 的响应是单独跟踪的，可通过
:download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>` 的 ServoDyn 选项卡中的输出通道获取。


平台 StC
~~~~~~~~~~~~

此 StC 安装位置相对于平台参考点定位。
当建模为刚体平台时（如刚性半潜式平台），它连接到平台参考点。
当建模为柔性浮式体时，StC 连接到 SubDyn 网格。


.. raw:: html

   <hr>

.. _StC-Input-File:

结构控制输入文件
-----------------------------

输入文件可以包含任意数量的注释头行，以及在输入文件任意位置
的注释行。
:download:`（NREL 5 MW TLP 塔筒调谐质量阻尼器结构控制输入文件示例）<ExampleFiles/NRELOffshrBsline5MW_StC.dat>`：

仿真控制
~~~~~~~~~~~~~~~~~~

**Echo** [flag]

   将输入数据回显到 <RootName>.ech


StC 自由度
----------------------

**StC_DOF_MODE** [switch]

   DOF 模式   {0: 无 StC 或 TLCD DOF; 1: StC_X_DOF、StC_Y_DOF 和/或 StC_Z_DOF
   （三个独立 StC DOF）; 2: StC_XY_DOF（2DOF 全向 StC）; 3:
   StC_XYZ_DOF（3DOF 全向 StC）; 5: TLCD; 6: 指定力/力矩
   时间序列; 7: 外部 DLL 确定的力}


**StC_X_DOF** [flag]

   StC X 方向 DOF 开启或关闭   *[仅当* **StC_DOF_MODE==1** *时使用]*

**StC_Y_DOF** [flag]

   StC Y 方向 DOF 开启或关闭   *[仅当* **StC_DOF_MODE==1** *时使用]*

**StC_Z_DOF** [flag]

   StC Z 方向 DOF 开启或关闭   *[仅当* **StC_DOF_MODE==1** *时使用]*


StC 位置
------------

StC 的位置相对于其所连接的部件。这在 ServoDyn 主输入文件中指定。
参见上文说明。

**StC_P_X** [m]

   StC 静止 X 位置

**StC_P_Y** [m]

   StC 静止 Y 位置

**StC_P_Z** [m]

   StC 静止 Z 位置


StC 初始条件
----------------------

*仅当* **StC_DOF_MODE==1、2 或 3** *时使用*

**StC_X_DSP** [m]

   StC X 初始位移   *[相对于静止位置]*

**StC_Y_DSP** [m]

   StC Y 初始位移   *[相对于静止位置]*

**StC_Z_DSP** [m]

   StC Z 初始位移   *[相对于静止位置；仅当*
   **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_Z_PreLd** [N]

   StC Z 弹簧预载。可以为弹簧预载的直接数值（单位牛顿），
   或 **"gravity"** 表示预载弹簧以在重力作用时偏移 StC Z 质量块的静止位置，
   使用 :math:`F_{Z_{PreLoad}} = M_Z * G`，或 **"none"** 禁用弹簧预载。
   实现细节参见 :numref:`SrvD-StCz-PreLoad`。
   *[仅当* **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或*
   **StC_DOF_MODE==3** *时使用]*


StC 配置
-----------------

*仅当* **StC_DOF_MODE==1、2 或 3** *时使用*

**StC_X_PSP** [m]

   正向止动位置——X 质量块最大位移

**StC_X_NSP** [m]

   负向止动位置——X 质量块最小位移

**StC_Y_PSP** [m]

   正向止动位置——Y 质量块最大位移

**StC_Y_NSP** [m]

   负向止动位置——Y 质量块最小位移

**StC_Z_PSP** [m]

   正向止动位置——Z 质量块最大位移 *[仅当*
   **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3**
   *时使用]*

**StC_Z_NSP** [m]

   负向止动位置——Z 质量块最小位移 *[仅当*
   **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3**
   *时使用]*

StC 质量、刚度和阻尼
------------------------------

*仅当* **StC_DOF_MODE==1、2 或 3** *时使用*

**StC_X_M** [kg]

   StC X 质量   *[仅当* **StC_DOF_MODE==1** *且* **StC_X_DOF==TRUE**
   *时使用]*

**StC_Y_M** [kg]

   StC Y 质量   *[仅当* **StC_DOF_MODE==1** *且* **StC_Y_DOF==TRUE**
   *时使用]*

**StC_Z_M** [kg]

   StC Z 质量   *[仅当* **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE**
   *时使用]*

**StC_Omni_M** [kg]

   StC 全向质量   *[仅当* **StC_DOF_MODE==2 或 3** *时使用]*

**StC_X_K** [N/m]

   StC X 刚度

**StC_Y_K** [N/m]

   StC Y 刚度

**StC_Z_K** [N/m]

   StC Z 刚度   *[仅当* **StC_DOF_MODE==1** *且*
   **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_X_C** [N/(m/s)]

   StC X 阻尼

**StC_Y_C** [N/(m/s)]

   StC Y 阻尼

**StC_Z_C** [N/(m/s)]

   StC Z 阻尼   *[仅当* **StC_DOF_MODE==1** *且*
   **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_X_KS** [N/m]

   止动弹簧 X 刚度

**StC_Y_KS** [N/m]

   止动弹簧 Y 刚度

**StC_Z_KS** [N/m]

   止动弹簧 Z 刚度   *[仅当* **StC_DOF_MODE==1** *且*
   **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_X_CS** [N/(m/s)]

   止动弹簧 X 阻尼

**StC_Y_CS** [N/(m/s)]

   止动弹簧 Y 阻尼

**StC_Z_CS** [N/(m/s)]

   止动弹簧 Z 阻尼   *[仅当* **StC_DOF_MODE==1** *且*
   **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*


StC 用户自定义弹簧力
------------------------------

*仅当* **StC_DOF_MODE==1、2 或 3** *时使用*

**Use_F_TBL** [flag]

   使用来自用户自定义表格的弹簧力

**NKInpSt** [-]

   弹簧力输入站点数量

表格应包含 6 列，分别为位移和等效弹簧力：**X**、**F_X**、**Y**、**F_Y**、**Z** 和 **F_Z**。
位移单位为米（m），力单位为牛顿（N）。

弹簧力表格示例：

.. container::
   :name: Tab:SpringForce

   .. literalinclude:: ExampleFiles/SpringForce.txt
      :language: none


StructCtrl 控制
------------------
*仅当* **StC_DOF_MODE==1、2、3 或 7** *时使用*

**StC_CMODE** [switch]

   控制模式   {0: 无; 1: 半主动控制模式; 3: 通过用户子程序的主动控制
   模式; 5: 通过 Bladed 接口的主动控制模式}。使用 StC_DOF_MODE==7 时，
   StC_CMODE 必须为 3 或 5。

**StC_CChan** [-]

   刚度和阻尼的控制通道组（1:10） *[仅当*
   **StC_DOF_MODE=1、2、3 或 7** *且* **StC_CMODE=5** *时使用]*

**StC_SA_MODE** [-]

   半主动控制模式 {1: 基于速度的天棚控制; 2: 基于逆速度
   的天棚控制; 3: 基于位移的天棚控制;
   4: 带摩擦力的相位差算法; 5: 带阻尼力的相位差算法}

**StC_X_C_HIGH** [-]

   天棚控制的 StC X 高阻尼

**StC_X_C_LOW** [-]

   天棚控制的 StC X 低阻尼

**StC_Y_C_HIGH** [-]

   天棚控制的 StC Y 高阻尼

**StC_Y_C_LOW** [-]

   天棚控制的 StC Y 低阻尼

**StC_Z_C_HIGH** [-]

   天棚控制的 StC Z 高阻尼 *[仅当*
   **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_Z_C_LOW** [-]

   天棚控制的 StC Z 低阻尼  *[仅当*
   **StC_DOF_MODE==1** *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]*

**StC_X_C_BRAKE** [-]

   制动 StC 的 StC X 高阻尼 *[当前未使用。设置为零]*

**StC_Y_C_BRAKE** [-]

   制动 StC 的 StC Y 高阻尼 *[当前未使用。设置为零]*

**StC_Z_C_BRAKE** [-]

   制动 StC 的 StC Z 高阻尼 *[仅当* **StC_DOF_MODE==1**
   *且* **StC_Z_DOF==TRUE** *或* **StC_DOF_MODE==3** *时使用]* *[当前
   未使用。设置为零]*



TLCD —— 调谐液柱阻尼器
----------------------------------

*仅当* **StC_DOF_MODE==5** *时使用*

**L_X** [m]

   X 方向 TLCD 总长度

**B_X** [m]

   X 方向 TLCD 水平段长度

**area_X** [m^2]

   X 方向 TLCD 竖直柱截面积

**area_ratio_X** [-]

   X 方向 TLCD 截面积比 *[竖直柱面积除以
   水平柱面积]*

**headLossCoeff_X** [-]

   X 方向 TLCD 水头损失系数

**rho_X** [kg/m^3]

   X 方向 TLCD 液体密度

**L_Y** [m]

   Y 方向 TLCD 总长度

**B_Y** [m]

   Y 方向 TLCD 水平段长度

**area_Y**        [m^2]

   Y 方向 TLCD 竖直柱截面积

**area_ratio_Y**  [-]

   Y 方向 TLCD 截面积比 *[竖直柱面积除以
   水平柱面积]*

**headLossCoeff_Y** [-]

   Y 方向 TLCD 水头损失系数

**rho_Y** [kg/m^3]

   Y 方向 TLCD 液体密度

指定时间序列
----------------------

可以在 StC 阻尼器的位置施加指定的力和力矩时间序列。
力和力矩可以施加在全局坐标系或局部（随动）坐标系中。此功能
*仅当* **StC_DOF_MODE==6** *时使用*。

**PrescribedForcesCoord** [switch]

   指定力采用全局还是局部坐标  {1: 全局; 2: 局部}。
   使用 StC_DOF_MODE==7 时，PrescribedForcesCoord 必须为 1。

**PrescribedForcesFile** [-]

   指定力的文件名。期望格式为 7 列：time、
   FX、FY、FZ、MX、MY、MZ。值将在文件中给定的时间步和
   值组之间进行插值。输入文件可以包含任意数量的注释头行，
   以及在输入文件任意位置的注释行。

   对于叶片安装的 StC，可以为每个叶片提供不同的时间序列输入文件
   （在新行中添加额外的文件名）。如果只提供一个文件，
   它将用于所有叶片。示例设置参见回归测试：
   `StC_test_OC4Semi_blade2`。


指定时间序列文件示例 :download:`（指定力时间序列示例）<ExampleFiles/PrescribedForce.txt>`：

.. container::
   :name: Tab:PrescribedForce

   .. literalinclude:: ExampleFiles/PrescribedForce.txt
      :language: none
