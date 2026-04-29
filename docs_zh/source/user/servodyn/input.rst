.. _SrvD-Input:

输入文件
===========

用户通过 ServoDyn 主输入文件，以及结构控制和控制器 DLL
的单独输入文件来配置伺服动力学模型参数。*此信息不完整，将在后续补充文档。*


单位
-----

ServoDyn 使用 SI 单位制（kg、m、s、N）。除非另有说明，角度默认
以弧度为单位。

ServoDyn 主输入文件
----------------------------

ServoDyn 主输入文件定义控制器的建模选项，
包括部分 DLL 选项和结构控制选项（通常为调谐质量阻尼器系统）。


仿真控制
~~~~~~~~~~~~~~~~~~

**Echo** [flag]

   将输入数据回显到 <RootName>.ech

**DT**   [sec]

   控制器的通信间隔（或 "default"）


变桨控制
~~~~~~~~~~~~~

**PCMode** [switch]

   变桨控制模式 {0: 无, 3: 用户自定义例程 PitchCntrl, 4:
   用户自定义 Simulink/Labview, 5: 用户自定义 Bladed 风格 DLL}

**TPCOn** [sec]

   启用主动变桨控制的时间 *[当* **PCMode==0** *时未使用]*

**PitNeut(1)** [deg]

   叶片 1 中性变桨位置——此位置下变桨弹簧力矩为零
   *[当* **PCMode>0** 且 **t>=TPCOn** *时未使用]*

**PitNeut(2)** [deg]

   叶片 2 中性变桨位置——此位置下变桨弹簧力矩为零
   *[当* **PCMode>0** 且 **t>=TPCOn** *时未使用]*

**PitNeut(3)** [deg]

   叶片 3 中性变桨位置——此位置下变桨弹簧力矩为零
   *[当* **PCMode>0** 且 **t>=TPCOn** *时未使用]* *[2 叶片时未使用]*

**PitSpr(1)** [N-m/rad]

   叶片 1 变桨弹簧常数

**PitSpr(2)** [N-m/rad]

   叶片 2 变桨弹簧常数

**PitSpr(3)** [N-m/rad]

   叶片 3 变桨弹簧常数 *[2 叶片时未使用]*

**PitDamp(1)** [N-m/(rad/s)]

   叶片 1 变桨阻尼常数

**PitDamp(2)** [N-m/(rad/s)]

   叶片 2 变桨阻尼常数

**PitDamp(3)** [N-m/(rad/s)]

   叶片 3 变桨阻尼常数 *[2 叶片时未使用]*

**TPitManS(1)** [sec]

   叶片 1 开始超控变桨机动并结束标准变桨控制的时间

**TPitManS(2)** [sec]

   叶片 2 开始超控变桨机动并结束标准变桨控制的时间

**TPitManS(3)** [sec]

   叶片 3 开始超控变桨机动并结束标准变桨控制的时间 *[2 叶片时未使用]*

**PitManRat(1)** [deg/s]

   超控变桨机动过程中叶片 1 趋向最终变桨角度的变桨速率

**PitManRat(2)** [deg/s]

   超控变桨机动过程中叶片 2 趋向最终变桨角度的变桨速率

**PitManRat(3)** [deg/s]

   超控变桨机动过程中叶片 3 趋向最终变桨角度的变桨速率 *[2 叶片时未使用]*

**BlPitchF(1)** [deg]

   变桨机动时叶片 1 的最终变桨角度

**BlPitchF(2)** [deg]

   变桨机动时叶片 2 的最终变桨角度

**BlPitchF(3)** [deg]

   变桨机动时叶片 3 的最终变桨角度 *[2 叶片时未使用]*


发电机与转矩控制
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**VSContrl** [switch]

   变速控制模式 {0: 无, 1: 简单变速, 3: 用户自定义例程 UserVSCont,
   4: 用户自定义 Simulink/Labview, 5: 用户自定义 Bladed 风格 DLL}

**GenModel** [switch]

   发电机模型 {1: 简单, 2: Thevenin, 3: 用户自定义例程
   UserGen} *[仅当* **VSContrl==0** *时使用]*

**GenEff**   [\%]

   发电机效率 *[Thevenin 和用户自定义发电机模型忽略此项]*

**GenTiStr** [flag]

   启动发电机的方法 {T: 使用 TimGenOn 定时启动, F: 使用 SpdGenOn 按发电机转速启动}

**GenTiStp** [Flag]

   停止发电机的方法 {T: 使用 TimGenOf 定时停止, F: 发电机功率为零时停止}

**SpdGenOn** [rpm]

   启动过程中开启发电机的发电机转速（HSS 转速） *[仅当*
   **GenTiStri==False** *时使用]*

**TimGenOn** [sec]

   启动过程中开启发电机的时间 *[仅当*
   **GenTiStr==True** *时使用]*

**TimGenOf** [sec]

   关闭发电机的时间 *[仅当* **GenTiStp==True** *时使用]*


简单变速转矩控制
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**VS_RtGnSp** [rpm]

   简单变速发电机控制的额定发电机转速（HSS 侧）
   *[仅当* **VSContrl==1** *时使用]*

**VS_RtTq**   [N-m]

   简单变速发电机控制在区域 3 的额定发电机转矩/恒定发电机转矩（HSS 侧） *[仅当* **VSContrl==1**
   *时使用]*

**VS_Rgn2K**  [N-m/rpm^2]

   简单变速发电机控制在区域 2 的发电机转矩常数（HSS 侧） *[仅当* **VSContrl==1** *时使用]*

**VS_SlPc**   [\%]

   简单变速发电机控制在区域 2 1/2 的额定发电机滑差百分比 *[仅当* **VSContrl==1** *时使用]*


简单感应发电机
~~~~~~~~~~~~~~~~~~~~~~~~~~

**SIG_SlPc**     [\%]

   额定发电机滑差百分比 *[仅当* **VSContrl==0** *且*
   **GenModel==1** *时使用]*

**SIG_SySp**     [rpm]

   同步（零转矩）发电机转速 *[仅当* **VSContrl==0**
   *且* **GenModel==1** *时使用]*

**SIG_RtTq**     [N-m]

   额定转矩 *[仅当* **VSContrl==0** *且* **GenModel==1** *时使用]*

**SIG_PORt**     [-]

   失步比（Tpullout/Trated） *[仅当* **VSContrl==0** *且*
   **GenModel==1** *时使用]*


Thevenin 等效感应发电机
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**TEC_Freq**     [Hz]

   电网频率 [50 或 60] *[仅当* **VSContrl==0** *且*
   **GenModel==2** *时使用]*

**TEC_NPol**     [-]

   极数 [偶数，> 0] *[仅当* **VSContrl==0** *且*
   **GenModel==2** *时使用]*

**TEC_SRes**     [ohms]

   定子电阻 *[仅当* **VSContrl==0** *且* **GenModel==2** *时使用]*

**TEC_RRes**     [ohms]

   转子电阻 *[仅当* **VSContrl==0** *且* **GenModel==2** *时使用]*

**TEC_VLL**      [volts]

   线间 RMS 电压 *[仅当* **VSContrl==0** *且*
   **GenModel==2** *时使用]*

**TEC_SLR**      [ohms]

   定子漏抗 *[仅当* **VSContrl==0** *且*
   **GenModel==2** *时使用]*

**TEC_RLR**      [ohms]

   转子漏抗 *[仅当* **VSContrl==0** *且*
   **GenModel==2** *时使用]*

**TEC_MR**       [ohms]

   励磁电抗 *[仅当* **VSContrl==0** *且* **GenModel==2**
   *时使用]*


高速轴制动
~~~~~~~~~~~~~~~~~~~~~~

**HSSBrMode**     [switch]

   HSS 制动模型 {0: 无, 1: 简单, 3: 用户自定义例程 UserHSSBr,
   4: 用户自定义 Simulink/Labview, 5: 用户自定义 Bladed 风格 DLL}

**THSSBrDp**      [sec]

   启动 HSS 制动部署的时间

**HSSBrDT**       [sec]

   HSS 制动从启动到完全部署所需的时间 *[仅当*
   **HSSBrMode==1** *时使用]*

**HSSBrTqF**      [N-m]

   完全部署时的 HSS 制动转矩


机舱偏航控制
~~~~~~~~~~~~~~~~~~~

**YCMode**        [switch]

   偏航控制模式 {0: 无, 3: 用户自定义例程 UserYawCont, 4:
   用户自定义 Simulink/Labview, 5: 用户自定义 Bladed 风格 DLL}

**TYCOn**         [sec]

   启用主动偏航控制的时间 *[当* **YCMode==0** *时未使用]*

**YawNeut**       [deg]

   中性偏航位置——此偏航角下偏航弹簧力为零

**YawSpr**        [N-m/rad]

   机舱偏航弹簧常数

**YawDamp**       [N-m/(rad/s)]

   机舱偏航阻尼常数

**TYawManS**      [sec]

   开始超控偏航机动并结束标准偏航控制的时间

**YawManRat**     [deg/s]

   偏航机动速率（绝对值）

**NacYawF**       [deg]

   超控偏航机动的最终偏航角度


.. _SrvD-AfC-inputs:

气动流动控制
~~~~~~~~~~~~~~~~~~~~~~~~

**AfCmode**       [switch]

   翼型控制模式 {0: 无, 1: 正弦波循环, 4: 用户自定义
   Simulink/Labview, 5: 用户自定义 Bladed 风格 DLL}

**AfC_Mean**      [-]

   余弦循环或稳态值的均值水平 *[仅当*
   **AfCmode==1** *时使用]*

**AfC_Amp**       [-]

   襟翼信号余弦循环的幅值 *[仅当*
   **AfCmode==1** *时使用]*

**AfC_Phase**     [deg]

   襟翼信号余弦循环相对于叶片方位角（0 为垂直）的相位 *[仅当* **AfCmode==1** *时使用]*

当 **AfCmode==1** 时，翼型流动控制信号由表达式
*AfC_Mean + p%AfC_Amp*cos( Azimuth + AfC_phase)* 设定，其中 azimuth
为该叶片的方位角（azimuth=0 视为垂直）。


.. _SrvD-CableControl-inputs:

缆索控制
~~~~~~~~~~~~~

MoorDyn 或 SubDyn 模块中指定的缆索单元可以通过 ServoDyn 由 Bladed 风格控制器进行控制。
每根缆索接收一对控制器通道，一个用于请求的缆索长度变化（DeltaL），
另一个用于缆索长度变化率（DeltaLdot）。
通道分配由包含缆索单元的模块（目前为 MoorDyn 和/或 SubDyn）请求，
并映射到相应的控制通道。
ServoDyn 输出的汇总文件中提供了请求通道的模块摘要。
连接 DLL 时最多可请求 100 个通道组，连接 Simulink 时最多 20 个通道组。

**CCmode**        [switch]

   缆索控制模式 {0: 无, 4: 用户自定义 Simulink/Labview, 5:
   用户自定义 Bladed 风格 DLL}。

   每个缆索控制通道组包含一个 DeltaL 通道（请求的
   缆索长度变化）和一个 DeltaLdot 通道（来自控制器/Simulink 接口的
   缆索长度变化率）。


.. _SrvD-StC-inputs:

结构控制
~~~~~~~~~~~~~~~~~~

以下各选项的安装位置说明参见 :numref:`StC-Locations`。

**NumBStC**      [integer]

   叶片结构控制器数量

**BStCfiles**      [-]

   叶片结构控制器文件名（一行中用引号括起的字符串）
   *[当* **NumBStC==0** *时未使用]*

**NumNStC**      [integer]

   机舱结构控制器数量

**NStCfiles**      [-]

   机舱结构控制器文件名（一行中用引号括起的字符串）
   *[当* **NumNStC==0** *时未使用]*

**NumTStC**      [integer]

   塔筒结构控制器数量

**TStCfiles**      [-]

   塔筒结构控制阻尼文件名（一行中用引号括起的字符串）
   *[当* **NumTStC==0** *时未使用]*

**NumSStC**   [integer]

   子结构结构控制器数量

**SStCfiles**   [-]

   子结构结构控制器文件名（一行中用引号括起的字符串）
   *[当* **NumSStC==0** *时未使用]*


Bladed 控制器接口
~~~~~~~~~~~~~~~~~~~~~~~~~~~

**DLL_FileName**  [-]

   Bladed-DLL 格式的动态库名称/路径 {.dll [Windows] 或 .so [Linux]}
   *[仅用于 Bladed 接口]*

**DLL_InFile**    [-]

   发送给 DLL 的输入文件名 *[仅用于 Bladed 接口]*

**DLL_ProcName**  [-]

   DLL 中待调用过程的名称 *[区分大小写；仅用于 DLL
   接口]*

**DLL_DT**        [sec]

   动态库的通信间隔（或 "default"） *[仅用于
   Bladed 接口]*

**DLL_Ramp**      [flag]

   是否在 DLL_DT 时间步之间使用线性斜坡 [为 true 时会引入
   时间偏移] *[仅用于 Bladed 接口]*

**BPCutoff**      [Hz]

   来自 DLL 的叶片变桨低通滤波器截止频率 *[仅用于
   Bladed 接口]*

**NacYaw_North**  [deg]

   当迎风端指向正北时，机舱的参考偏航角度
   *[仅用于 Bladed 接口]*

**Ptch_Cntrl**    [switch]

   记录 28：使用独立变桨控制 {0: 集中变桨; 1: 独立
   变桨控制} *[仅用于 Bladed 接口]*

**Ptch_SetPnt**   [deg]

   记录  5：低于额定风速时的变桨角度设定值 *[仅用于 Bladed
   接口]*

**Ptch_Min**      [deg]

   记录  6：最小变桨角度 *[仅用于 Bladed 接口]*

**Ptch_Max**      [deg]

   记录  7：最大变桨角度 *[仅用于 Bladed 接口]*

**PtchRate_Min**  [deg/s]

   记录  8：最小变桨速率（允许的最负值） *[仅用于
   Bladed 接口]*

**PtchRate_Max**  [deg/s]

   记录  9：最大变桨速率 *[仅用于 Bladed 接口]*

**Gain_OM**       [N-m/(rad/s)^2]

   记录 16：最优模式增益 *[仅用于 Bladed 接口]*

**GenSpd_MinOM**  [rpm]

   记录 17：最小发电机转速 *[仅用于 Bladed 接口]*

**GenSpd_MaxOM**  [rpm]

   记录 18：最优模式最大转速 *[仅用于 Bladed 接口]*

**GenSpd_Dem**    [rpm]

   记录 19：额定以上所需发电机转速 *[仅用于 Bladed
   接口]*

**GenTrq_Dem**    [N-m]

   记录 22：额定以上所需发电机转矩 *[仅用于 Bladed
   接口]*

**GenPwr_Dem**    [W]

   记录 13：需求功率 *[仅用于 Bladed 接口]*


Bladed 接口转矩-转速查表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**DLL_NumTrq**    [-]

   记录 26：转矩-转速查表中的点数
   {0 = 无，使用最优模式参数；非零 = 通过将记录 16 设为 0.0 忽略
   最优模式参数} *[仅用于 Bladed
   接口]*
   期望以下 2 列表格格式：

   +------------+------------+
   | GenSpd_TLU | GenTrq_TLU |
   |   (rpm)    |   (N-m)    |
   +------------+------------+


.. _SrvD-Outputs:

输出
~~~~~~

**SumPrint**      [flag]

   将汇总数据打印到 <RootName>.sum。该文件包含输入摘要，
   并在使用 Bladed 风格控制器时给出通信通道的详细列表。
   此信息有助于调试控制器或验证 ServoDyn 的配置方式。

**OutFile**       [-]

   决定输出位置的开关：{1: 仅输出到模块输出文件; 2: 仅输出到耦合框架输出文件;
   3: 两者都输出} *（当前未使用）*

**TabDelim**      [flag]

   在文本表格输出文件中使用制表符分隔？ *（当前未使用）*

**OutFmt**        [-]

   用于文本表格输出（时间除外）的格式。结果字段应为
   10 个字符。（引号括起的字符串） *（当前未使用）*

**TStart**        [sec]

   开始表格输出的时间 *（当前未使用）*

**OutList** 节控制 ServoDyn 生成的输出量。
输入一行或多行，包含引号括起的字符串，其中
包含一个或多个输出参数名称。用逗号、分号、空格和/或制表符的任意组合
分隔输出参数名称。如果
在参数名称前加负号 "-"、下划线 "_" 或
字符 "m" 或 "M"，ServoDyn 会在写入数据前将该通道的值乘以 -1。
参数按照输入文件中列出的顺序写入。ServoDyn 允许您使用
多行，以便将列表分为有意义的组，
使每行更短。您可以在任一行的
闭合引号后输入注释。在行首或行首引号字符串的开头输入字符串 "END"，
将使 ServoDyn 停止扫描
更多通道名称行。如果 ServoDyn 遇到
未知/无效的通道名称，它会警告用户，但会从输出文件中
移除可疑通道。请参阅 Excel 文件
:download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>`
的 ServoDyn 选项卡获取完整的输出参数列表。
