
.. _ed_input:

输入文件
========

用户通过 ElastoDyn 主输入文件以及塔筒和其他后续将在此记录的输入文件来配置结构模型参数。

不得在输入文件中添加或删除任何行。

单位
----

ElastoDyn 使用国际单位制（kg、m、s、N）。除非另有说明，角度默认以弧度为单位。

ElastoDyn 主输入文件
---------------------

ElastoDyn 主输入文件定义了 OpenFAST 结构的建模选项和几何参数，包括塔筒、机舱、传动链和叶片（如果不使用 BeamDyn）。它还设置了结构的初始条件。

仿真控制
~~~~~~~~

如果希望 ElastoDyn 回显 ElastoDyn 主输入文件、翼型输入文件和叶片输入文件的内容（对调试输入文件中的错误很有用），请将 **Echo** 标志设置为 TRUE。回显文件的命名规则为 *OutRootFile.ED.ech*。**OutRootFile** 在独立运行 ElastoDyn 时由驱动输入文件的 I/O 设置部分指定，在运行耦合仿真时由 OpenFAST 程序指定。

**Method**

**dT**

自由度
~~~~~~

**FlapDOF1**    - 第一阶叶片挥舞模式自由度（标志）
**FlapDOF2**    - 第二阶叶片挥舞模式自由度（标志）
**EdgeDOF**     - 第一阶叶片摆振模式自由度（标志）
**PitchDOF**    - 叶片变桨自由度（标志）
**TeetDOF**     - 转子跷跷板自由度（标志）[3 叶片机型不使用]
**DrTrDOF**     - 传动链旋转柔性自由度（标志）
**GenDOF**      - 发电机自由度（标志）
**YawDOF**      - 偏航自由度（标志）
**TwFADOF1**    - 第一阶塔筒前后弯曲模式自由度（标志）
**TwFADOF2**    - 第二阶塔筒前后弯曲模式自由度（标志）
**TwSSDOF1**    - 第一阶塔筒左右弯曲模式自由度（标志）
**TwSSDOF2**    - 第二阶塔筒左右弯曲模式自由度（标志）
**PtfmSgDOF**   - 平台水平纵荡平移自由度（标志）
**PtfmSwDOF**   - 平台水平横荡平移自由度（标志）
**PtfmHvDOF**   - 平台垂直垂荡平移自由度（标志）
**PtfmRDOF**    - 平台横滚倾斜旋转自由度（标志）
**PtfmPDOF**    - 平台俯仰倾斜旋转自由度（标志）
**PtfmYDOF**    - 平台偏航旋转自由度（标志）

初始条件
~~~~~~~~

**OoPDefl**     - 初始叶尖平面外位移（米）
**IPDefl**      - 初始叶尖平面内偏转（米）
**BlPitch(1)**  - 叶片 1 初始变桨角（度）
**BlPitch(2)**  - 叶片 2 初始变桨角（度）
**BlPitch(3)**  - 叶片 3 初始变桨角（度）[2 叶片机型不使用]
**TeetDefl**    - 初始或固定跷跷板角（度）[3 叶片机型不使用]
**Azimuth**     - 叶片 1 初始方位角（度）
**RotSpeed**    - 初始或固定转子转速（rpm）
**NacYaw**      - 初始或固定机舱偏航角（度）
**TTDspFA**     - 初始塔顶前后位移（米）
**TTDspSS**     - 初始塔顶左右位移（米）
**PtfmSurge**   - 平台初始或固定水平纵荡平移位移（米）
**PtfmSway**    - 平台初始或固定水平横荡平移位移（米）
**PtfmHeave**   - 平台初始或固定垂直垂荡平移位移（米）
**PtfmRoll**    - 平台初始或固定横滚倾斜旋转位移（度）
**PtfmPitch**   - 平台初始或固定俯仰倾斜旋转位移（度）
**PtfmYaw**     - 平台初始或固定偏航旋转位移（度）

风力机配置
~~~~~~~~~~

**NumBl**       - 叶片数量（-）
**TipRad**      - 从转子顶点到叶尖的距离（米）
**HubRad**      - 从转子顶点到叶根的距离（米）
**PreCone(1)**  - 叶片 1 锥角（度）
**PreCone(2)**  - 叶片 2 锥角（度）
**PreCone(3)**  - 叶片 3 锥角（度）[2 叶片机型不使用]
**HubCM**       - 从转子顶点到轮毂质心的距离[顺风方向为正]（米）
**UndSling**    - 下悬长度[从跷跷板销到转子顶点的距离]（米）[3 叶片机型不使用]
**Delta3**      - 跷跷板转子的 Delta-3 角（度）[3 叶片机型不使用]
**AzimB1Up**    - 当叶片 1 朝上时使用的 I/O 方位角值（度）；对于漂浮式 MHK 风力机，当 `AzimB1Up` = 0 时，叶片 1 将朝上（与重力方向相反）；用户可以将 `AzimB1Up` 设置为 180 度，使漂浮式 MHK 风力机相对于塔筒的方位角约定与固定式 MHK 风力机相同
**OverHang**    - 从偏航轴到转子顶点[3 叶片机型]或跷跷板销[2 叶片机型]的距离（米）
**ShftGagL**    - 从转子顶点[3 叶片机型]或跷跷板销[2 叶片机型]到轴应变片的距离[上风式转子为正]（米）
**ShftTilt**    - 转子轴倾斜角（度）
**NacCMxn**     - 从塔顶到机舱质心的顺风距离（米）
**NacCMyn**     - 从塔顶到机舱质心的侧向距离（米）
**NacCMzn**     - 从塔顶到机舱质心的垂直距离，对于漂浮式 MHK 风力机通常为负值（米）
**NcIMUxn**     - 从塔顶到机舱 IMU 的顺风距离（米）
**NcIMUyn**     - 从塔顶到机舱 IMU 的侧向距离（米）
**NcIMUzn**     - 从塔顶到机舱 IMU 的垂直距离，对于漂浮式 MHK 风力机通常为负值（米）
**Twr2Shft**    - 从塔顶到转子轴的垂直距离，对于漂浮式 MHK 风力机通常为负值（米）
**TowerHt**     - 塔筒相对于地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]的高度（米）
**TowerBsHt**   - 塔筒底部相对于地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]的高度（米）
**PtfmCMxt**    - 从地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]到平台质心的顺风距离（米）
**PtfmCMyt**    - 从地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]到平台质心的侧向距离（米）
**PtfmCMzt**    - 从地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]到平台质心的垂直距离（米）
**PtfmRefzt**   - 从地面[陆上]、平均海平面[海上风电或漂浮式 MHK]或海床[固定式 MHK]到平台参考点的垂直距离（米）

质量和惯量
~~~~~~~~~~

**TipMass(1)**  - 叶尖制动块质量，叶片 1（kg）
**TipMass(2)**  - 叶尖制动块质量，叶片 2（kg）
**TipMass(3)**  - 叶尖制动块质量，叶片 3（kg）[2 叶片机型不使用]
**PBrIner(1)**  - 变桨轴承/执行器惯量，叶片 1（kg·m²）
**PBrIner(2)**  - 变桨轴承/执行器惯量，叶片 2（kg·m²）
**PBrIner(3)**  - 变桨轴承/执行器惯量，叶片 3（kg·m²）[2 叶片机型不使用]
**BlPIner(1)**  - 无变形叶片的变桨惯量，叶片 1（kg·m²）
**BlPIner(2)**  - 无变形叶片的变桨惯量，叶片 2（kg·m²）
**BlPIner(3)**  - 无变形叶片的变桨惯量，叶片 3（kg·m²）[2 叶片机型不使用]
**HubMass**     - 轮毂质量（kg）
**HubIner**     - 轮毂绕转子轴的惯量（2 或 3 叶片机型）（kg·m²）
**HubIner_Teeter**     - 轮毂绕跷跷板轴的惯量（2 叶片机型）（kg·m²）
**GenIner**     - 发电机绕高速轴的惯量（kg·m²）
**NacMass**     - 机舱质量（kg）
**NacYIner**    - 机舱绕偏航轴的惯量（kg·m²）
**YawBrMass**   - 偏航轴承质量（kg）
**PtfmMass**    - 平台质量（kg）
**PtfmRIner**   - 平台绕质心的横滚倾斜旋转惯量（kg·m²）
**PtfmPIner**   - 平台绕质心的俯仰倾斜旋转惯量（kg·m²）
**PtfmYIner**   - 平台绕质心的偏航旋转惯量（kg·m²）
**PtfmXYIner**  - 平台横滚-俯仰惯量积（*Ixy=-∫xydm*）绕平台质心（kg·m²）
**PtfmYZIner**  - 平台俯仰-偏航惯量积（*Iyz=-∫yzdm*）绕平台质心（kg·m²）
**PtfmXZIner**  - 平台横滚-偏航惯量积（*Ixz=-∫xzdm*）绕平台质心（kg·m²）

叶片
~~~~

**BldNodes**    - 分析使用的叶片节点数（每个叶片）（-）
**BldFile(1)**  - 包含叶片 1 属性的文件名（引号字符串）
**BldFile(2)**  - 包含叶片 2 属性的文件名（引号字符串）
**BldFile(3)**  - 包含叶片 3 属性的文件名（引号字符串）[2 叶片机型不使用]

转子跷跷板
~~~~~~~~~~

**TeetMod**     - 转子跷跷板弹簧/阻尼器模型 {0: 无，1: 标准，2: 用户自定义例程 UserTeet}（开关）[3 叶片机型不使用]
**TeetDmpP**    - 转子跷跷板阻尼器位置（度）[仅适用于 2 叶片机型且 TeetMod=1 时使用]
**TeetDmp**     - 转子跷跷板阻尼常数（N·m/(rad/s)）[仅适用于 2 叶片机型且 TeetMod=1 时使用]
**TeetSStP**    - 转子跷跷板软停止位置（度）[仅适用于 2 叶片机型且 TeetMod=1 时使用]
**TeetHStP**    - 转子跷跷板硬停止位置（度）[仅适用于 2 叶片机型且 TeetMod=1 时使用]
**TeetSSSp**    - 转子跷跷板软停止线性弹簧常数（N·m/rad）[仅适用于 2 叶片机型且 TeetMod=1 时使用]
**TeetHSSp**    - 转子跷跷板硬停止线性弹簧常数（N·m/rad）[仅适用于 2 叶片机型且 TeetMod=1 时使用]

偏航摩擦
~~~~~~~~

**YawFrctMod**  - 偏航摩擦模型 {0: 无，1: 与偏航轴承力和弯矩无关的摩擦，2: 库仑项取决于偏航轴承力和弯矩的摩擦，3: 用户自定义模型}
**M_CSmax**     - 最大静库仑摩擦力矩（N·m）[YawFrctMod=1 时为 M_CSmax；YawFrctMod=2 时为 -min(0,Fz)*M_CSmax]
**M_FCSmax**    - 与偏航轴承剪切力成正比的最大静库仑摩擦力矩（N·m）[sqrt(Fx²+Fy²)*M_FCSmax；仅当 YawFrctMod=2 时使用]
**M_MCSmax**    - 与偏航轴承弯矩成正比的最大静库仑摩擦力矩（N·m）[sqrt(Mx²+My²)*M_MCSmax；仅当 YawFrctMod=2 时使用]
**M_CD**        - 动库仑摩擦力矩（N·m）[YawFrctMod=1 时为 M_CD；YawFrctMod=2 时为 -min(0,Fz)*M_CD]
**M_FCD**       - 与偏航轴承剪切力成正比的动库仑摩擦力矩（N·m）[sqrt(Fx²+Fy²)*M_FCD；仅当 YawFrctMod=2 时使用]
**M_MCD**       - 与偏航轴承弯矩成正比的动库仑摩擦力矩（N·m）[sqrt(Mx²+My²)*M_MCD；仅当 YawFrctMod=2 时使用]
**sig_v**       - 线性粘性摩擦系数（N·m/(rad/s)）
**sig_v2**      - 二次粘性摩擦系数（N·m/(rad/s)²）
**OmgCut**      - 低于该值时粘性摩擦线性化的偏航角速度截止值（rad/s）

传动链
~~~~~~

**GBoxEff**     - 齿轮箱效率（%）
**GBRatio**     - 齿轮箱速比（-）
**DTTorSpr**    - 传动链扭转弹簧刚度（N·m/rad）
**DTTorDmp**    - 传动链扭转阻尼系数（N·m/(rad/s)）

折叠
~~~~

**Furling**     - 读取折叠式风力机的附加模型属性（标志）[当前必须为 FALSE]
**FurlFile**    - 包含折叠属性的文件名（引号字符串）[当 Furling=False 时不使用]
折叠输入文件的示例见 :numref:`TF_ed_input-file`。

塔筒
~~~~

**TwrNodes**    - 分析使用的塔筒节点数（-）
**TwrFile**     - 包含塔筒属性的文件名（引号字符串）

.. _ED-Outputs:

输出
~~~~

**SumPrint** [标志] 如果希望 ElastoDyn 生成名称为 **OutFileRoot**.ED.sum* 的汇总文件，请将此值设置为 TRUE。**OutFileRoot** 在运行耦合仿真时由 OpenFAST 程序指定。

**OutFile** [开关] 当前未使用。最终目的是允许将 ElastoDyn 的输出写入模块输出文件（选项 1）、主 OpenFAST 输出文件（选项 2）或两者都写入。目前此开关被忽略。

**TabDelim** [标志] 当前未使用。将其设置为 True 将为 ElastoDyn 模块的 **OutFile** 设置文本文件的分隔符为制表符。

**OutFmt** [引号字符串] 当前未使用。ElastoDyn 将使用此字符串作为在 **OutFile** 指定的本地输出中输出浮点值的数值格式说明符。此字符串的长度不得超过 20 个字符，并且必须用单引号或双引号括起来。不得指定空字符串。为确保固定宽度的列数据与列标题正确对齐，应确保字段宽度为 10 个字符。使用 E、EN 或 ES 说明符将保证永远不会因为数字太大而溢出字段，但这些数字更难阅读。使用 F 说明符将使数字更容易阅读，但可能会溢出字段。有关格式说明符的详细信息，请参考任何 Fortran 手册。

**TStart** [秒] 设置 **OutFile** 的开始时间。当前未使用。

**DecFact** [-] 此参数设置输出的抽取因子。ElastoDyn 每 DecFact 个积分时间步才会向 **OutFile** 输出一次数据。例如，值为 5 将使 ElastoDyn 每 5 个时间步生成一次输出。此值必须是大于零的整数。

**NTwGages** [-] 沿塔筒的应变片位置数量，表示下一行的输入值数量。有效值为 0 到 5（含）的整数。

**TwrGagNd** [-] 沿塔筒的虚拟应变片位置分配给此行指定的塔筒分析节点。可能值为 1 到 TwrNodes（含），其中 1 对应最靠近塔筒底部（但不在底部）的节点，TwrNodes 对应最靠近塔顶的节点。无变形塔筒中每个分析节点相对于塔筒底部的精确高程确定如下：

   节点 J 的高程 = TwrRBHt + ( J – 1⁄2 ) • [ ( TowerHt + TwrDraft – TwrRBHt ) / TwrNodes ]
      （适用于 J = 1,2,...,TwrNodes）

此行必须至少输入 NTwGages 个值。如果 NTwGages 为 0，将跳过此行，但输入文件中必须保留一行占位。可以使用制表符、空格和逗号的组合来分隔值，但数字之间只能使用一个逗号。

**NBlGages** [-] 指定沿叶片的应变片位置数量，并表示 **BldGagNd** 中预期的输入值数量。仅当叶片结构在 ElastoDyn 中建模时使用。

**BldGagNd** [-] 指定应输出的沿叶片的虚拟应变片位置。可能值为 1 到 **BldNodes**（含），其中 1 对应最靠近叶根（但不在根部）的节点，BldNodes 对应最靠近叶尖的节点。节点位置由 ElastoDyn 叶片输入文件指定。此行必须至少输入 NBlGages 个值。如果 NBlGages 为 0，将跳过此行，但输入文件中必须保留一行占位。可以使用制表符、空格和逗号的组合来分隔值，但数字之间只能使用一个逗号。仅当叶片结构在 ElastoDyn 中建模时使用。

**OutList** 部分控制 ElastoDyn 生成的输出量。输入一行或多行包含引号字符串的内容，这些字符串又包含一个或多个输出参数名称。输出参数名称可以用逗号、分号、空格和/或制表符的任意组合分隔。如果在参数名称前加上减号“-”、下划线“_”或字符“m”或“M”，ElastoDyn 在写入数据前会将该通道的值乘以 –1。参数按照输入文件中列出的顺序写入。ElastoDyn 允许使用多行，以便将列表分成有意义的组，并使行更短。可以在任意行的右引号后输入注释。如果在行首或行首的引号字符串开头输入字符串“END”，将导致 ElastoDyn 停止扫描更多通道名称行。叶片和塔筒节点相关的量是为上述 **BldGagNd** 和 **TwrGagNd** 列表中请求的节点生成的。如果 ElastoDyn 遇到未知/无效的通道名称，会向用户发出警告，但会从输出文件中删除该可疑通道。有关可能的输出参数的完整列表，请参考 Excel 文件 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>` 中的 ElastoDyn 标签。

.. _ED-Nodal-Outputs:

.. include:: EDNodalOutputs.rst

.. _TF_ed_input-file:

ElastoDyn 折叠输入文件
----------------------

本节描述 ElastoDyn 输入文件中输入项 ``FurlFile`` 所指定的折叠输入文件。
只有当模型被指定为折叠式机型时（主输入文件中的 Furling 设置为 True），OpenFAST 才会读取此文件。
该输入文件定义了风轮折叠和尾翼折叠的几何和结构属性。

风轮折叠和尾翼折叠坐标系以及几何输入分别在 :numref:`ed_rfrl_coordsys` 和 :numref:`ed_tfrl_coordsys` 中描述。

ElastoDyn 折叠输入文件的示例如下：

.. code::

    ---------------------- FAST FURLING FILE ---------------------------------------
    Comment
    ---------------------- FEATURE FLAGS (CONT) ------------------------------------
    False       RFrlDOF     - Rotor-furl DOF (flag)
    True        TFrlDOF     - Tail-furl DOF (flag)
    ---------------------- INITIAL CONDITIONS (CONT) -------------------------------
       0.0      RotFurl     - Initial or fixed rotor-furl angle (deg)
       0.0      TailFurl    - Initial or fixed tail-furl angle (deg)
    ---------------------- TURBINE CONFIGURATION (CONT) ----------------------------
       0.1      Yaw2Shft    - Lateral distance from the yaw axis to the rotor shaft (m)
       0.0      ShftSkew    - Rotor shaft skew angle (deg)
    0., 0., 0.  RFrlCM_n    - Position of the CM of the structure that furls with the rotor [not including rotor] from the tower-top, in nacelle coordinates (m)
    1.7,0.1,0.  BoomCM_n    - Postion of the tail boom CM from the tower top, in nacelle coordinates (m)
    0., 0., 0.  TFinCM_n    - Position of tail fin CM from the tower top, in nacelle coordinates (m)
    0., 0., 0.  RFrlPnt_n   - Position of an arbitrary point on the rotor-furl axis from the tower top, in nacelle coordinates (m)
       0.0      RFrlSkew    - Rotor-furl axis skew angle (deg)
       0.0      RFrlTilt    - Rotor-furl axis tilt angle (deg)
    0.3, 0., 0. TFrlPnt_n   - Position of an arbitrary point on the tail-furl axis from the tower top, in nacelle coordinates (m)
     -45.2      TFrlSkew    - Tail-furl axis skew angle (deg)
      78.7      TFrlTilt    - Tail-furl axis tilt angle (deg)
    ---------------------- MASS AND INERTIA (CONT) ---------------------------------
       0.0      RFrlMass    - Mass of structure that furls with the rotor [not including rotor] (kg)
      86.8      BoomMass    - Tail boom mass (kg)
       0.0      TFinMass    - Tail fin mass (kg)
       0.0      RFrlIner    - Inertia of the structure that furls with the rotor about the rotor-furl axis (kg m^2) [not including rotor]
     264.7      TFrlIner    - Tail boom inertia about tail-furl axis (kg m^2)
    ---------------------- ROTOR-FURL ----------------------------------------------
       0        RFrlMod     - Rotor-furl spring/damper model {0: none, 1: standard, 2:user-defined routine} (switch)
       0.0      RFrlSpr     - Rotor-furl spring constant (N-m/rad) [used only when RFrlMod=1]
       0.0      RFrlDmp     - Rotor-furl damping constant (N-m/(rad/s)) [used only when RFrlMod=1]
       0.0      RFrlUSSP    - Rotor-furl up-stop spring position (deg) [used only when RFrlMod=1]
       0.0      RFrlDSSP    - Rotor-furl down-stop spring position (deg) [used only when RFrlMod=1]
       0.0      RFrlUSSpr   - Rotor-furl up-stop spring constant (N-m/rad) [used only when RFrlMod=1]
       0.0      RFrlDSSpr   - Rotor-furl down-stop spring constant (N-m/rad) [used only when RFrlMod=1]
       0.0      RFrlUSDP    - Rotor-furl up-stop damper position (deg) [used only when RFrlMod=1]
       0.0      RFrlDSDP    - Rotor-furl down-stop damper position (deg) [used only when RFrlMod=1]
       0.0      RFrlUSDmp   - Rotor-furl up-stop damping constant (N-m/(rad/s)) [used only when RFrlMod=1]
       0.0      RFrlDSDmp   - Rotor-furl down-stop damping constant (N-m/(rad/s)) [used only when RFrlMod=1]
    ---------------------- TAIL-FURL -----------------------------------------------
       1        TFrlMod     - Tail-furl spring/damper model {0: none, 1: standard, 2:user-defined routine} (switch)
       0.0      TFrlSpr     - Tail-furl spring constant (N-m/rad) [used only when TFrlMod=1]
      10.0      TFrlDmp     - Tail-furl damping constant (N-m/(rad/s)) [used only when TFrlMod=1]
      85.0      TFrlUSSP    - Tail-furl up-stop spring position (deg) [used only when TFrlMod=1]
       3.0      TFrlDSSP    - Tail-furl down-stop spring position (deg) [used only when TFrlMod=1]
       1.0E3    TFrlUSSpr   - Tail-furl up-stop spring constant (N-m/rad) [used only when TFrlMod=1]
       1.7E4    TFrlDSSpr   - Tail-furl down-stop spring constant (N-m/rad) [used only when TFrlMod=1]
      85.0      TFrlUSDP    - Tail-furl up-stop damper position (deg) [used only when TFrlMod=1]
       0.0      TFrlDSDP    - Tail-furl down-stop damper position (deg) [used only when TFrlMod=1]
       1.0E3    TFrlUSDmp   - Tail-furl up-stop damping constant (N-m/(rad/s)) [used only when TFrlMod=1]
     137.0      TFrlDSDmp   - Tail-furl down-stop damping constant (N-m/(rad/s)) [used only when TFrlMod=1]

*功能标志*

**RFrlDOF**
当此值为 True 时，将启用风轮折叠自由度。初始风轮折叠角由 RotFurl 指定。如果 RFrlDOF 被禁用，风轮折叠角将固定为 RotFurl。（标志）

**TFrlDOF**
当此值为 True 时，将启用尾翼折叠自由度。初始尾翼折叠角由 TailFurl 指定。如果 TFrlDOF 被禁用，尾翼折叠角将固定为 TailFurl。（标志）

*初始条件*

**RotFurl**
这是固定的或初始的风轮折叠角。绕风轮折叠轴为正，如 :numref:`figTFAxes` 所示。风轮折叠轴通过以下输入项定义：``RFrlPnt_n``、RFrlSkew 和 RFrlTilt。此值必须大于 -180 且小于等于 180 度。（度）

**TailFurl**
这是固定的或初始的尾翼折叠角。绕尾翼折叠轴为正，如 :numref:`figTFAxes` 所示。尾翼折叠轴通过以下输入项定义：``TFrlPnt_n``、``TFrlSkew`` 和 ``TFrlTilt``。此值必须大于 -180 且小于等于 180 度。（度）

*风力机配置*

输入项 ``RFrlPnt_n``、``RFrlSkew`` 和 ``RFrlTilt`` 定义了风轮折叠轴及其相关自由度 ``RFrlDOF`` 的方向。
输入项 ``TFrlPnt_n``、``TFrlSkew`` 和 ``TFrlTilt`` 定义了尾翼折叠轴及其相关自由度 ``TFrlDOF`` 的方向。
参见 :numref:`figTFAxes`。

**Yaw2Shft**
这是从偏航轴到转子轴与 yn-/zn-平面交点的侧向偏移距离。该距离平行于 yn 轴测量。顺风观看时向左为正，如 :numref:`figTFFurl` 所示。
对于具有风轮折叠功能的风力机，此距离定义了折叠角为零时的配置。（米）

**ShftSkew**
这是转子轴在名义水平面内的偏斜角。正偏斜的作用类似于正机舱偏航，如 :numref:`figTFFurl` 所示；但是，``ShftSkew`` 仅应用于将轴偏离零偏航位置几度，不得用作偏航角的替代。此值必须介于 -15 和 15 度（含）之间。
对于具有风轮折叠功能的风力机，此角度定义了折叠角为零时的配置。（度）

**RFrlCM_n**
随风轮折叠的结构的质心位置（不包括转子 - 参考输入项 ``RFrlMass``），从塔顶测量并以机舱坐标系表示。
参见 :numref:`figTFFurl`。
对于具有风轮折叠功能的风力机，此位置定义了折叠角为零时的配置。（米）

**BoomCM_n**
尾梁质心位置（参考输入项 ``BoomMass``）相对于塔顶，以机舱坐标系表示。
参见 :numref:`figTFGeom`。
对于具有尾翼折叠功能的风力机，此距离定义了折叠角为零时的配置。（米）

**TFinCM_n**
尾翼质心位置（参考输入项 ``TFinMass``）相对于塔顶，以机舱坐标系表示。
参见 :numref:`figTFGeom`。
对于具有尾翼折叠功能的风力机，此距离定义了折叠角为零时的配置。（米）

**RFrlPnt_n**
风轮折叠轴上任意点的位置，从塔顶测量并以机舱坐标系表示。
参见 :numref:`figTFAxes`。（米）

**RFrlSkew**
这是风轮折叠轴在名义水平面内的偏斜角。正偏斜使风轮折叠轴的名义水平投影绕 zn 轴定向。
参见 :numref:`figTFAxes`。
此值必须大于 -180 且小于等于 180 度。（度）

**RFrlTilt**
这是风轮折叠轴与名义水平面的倾斜角。
此值必须介于 -90 和 90 度（含）之间。
参见 :numref:`figTFAxes`。（度）

**TFrlPnt_n**
从塔顶到尾翼折叠轴上任意点的位置，以机舱坐标系表示。
参见 :numref:`figTFAxes`。（米）

**TFrlSkew**
这是尾翼折叠轴在名义水平面内的偏斜角。
正偏斜使尾翼折叠轴的名义水平投影绕 zn 轴定向。
参见 :numref:`figTFAxes`。
此值必须大于 -180 且小于等于 180 度。
（度）

**TFrlTilt**
这是尾翼折叠轴与名义水平面的倾斜角。
参见 :numref:`figTFAxes`。
此值必须介于 -90 和 90 度（含）之间。
（度）

*质量和惯量*

**RFrlMass**
这是随风轮折叠的结构（不包括转子）的质量。该质量的质心位于风轮折叠角为零时输入项 ``RFrlCM_n`` 相对于塔顶指定的点。它包括随风轮折叠的所有部件，不包括转子（叶片、轮毂和叶尖制动器）。此值不得为负。（kg）

**BoomMass**
这是尾梁的质量。尾梁质量的质心位于尾翼折叠角为零时输入项 ``BoomCM_n`` 相对于塔顶指定的点。它包括随尾翼折叠的所有部件，不包括尾翼（参见下一个输入项）。此值不得为负。（kg）

**TFinMass**
这是尾翼的质量。尾翼质量的质心位于尾翼折叠角为零时输入项 ``TFinCM_n`` 相对于塔顶指定的点。TFinMass 和 BoomMass 之和应包括随尾翼折叠的所有部件。此值不得为负。（kg）

**RFrlIner**
这是随风轮折叠的结构（不包括转子）绕风轮折叠轴的转动惯量。它包括 ``RFrlMass`` 中包含的所有质量。此值必须大于：``RFrlMass*d²``，其中 d 是风轮折叠轴与随风轮折叠的结构（不包括转子）质心之间的垂直距离。（kg·m²）

**TFrlIner**
这是尾梁绕尾翼折叠轴的转动惯量。它包括 BoomMass 中包含的所有质量。此值必须大于：``BoomMass*d²``，其中 d 是尾翼折叠轴与尾梁质心之间的垂直距离。（kg·m²）

*风轮折叠*

通过将 ``RFrlMod`` 设置为 0，风轮折叠轴承可以是无摩擦的理想轴承；通过将 ``RFrlMod`` 设置为 1，可以使用标准模型，包括线性弹簧和线性阻尼器，以及上、下止动弹簧和上、下止动阻尼器。
公式见 :numref:`ed_rtfrl_theory`。
ElastoDyn 将止动弹簧建模为风轮折叠偏转的线性函数。风轮折叠止动从指定角度开始，根据超过止动角的偏转角作为线性弹簧工作。风轮折叠阻尼器是折叠速率的线性函数，从指定的上止动和下止动角度开始工作。这些阻尼器是双向的，一旦超过止动角，对两个方向的运动都有相同的阻力。

还提供了用户自定义的风轮折叠弹簧和阻尼器模型。要使用它，请将 `RFrlMod` 设置为 2，并创建一个名为 `UserRFrl()` 的子例程，包含参数 ``RFrlDef``、``RFrlRate``、``DirRoot``、``ZTime`` 和 ``RFrlMom``：

- ``RFrlDef``: 当前风轮折叠角偏转（弧度）（输入）
- ``RFrlRate``: 当前风轮折叠角速度（rad/s）（输入）
- ``ZTime``: 当前仿真时间（秒）（输入）
- ``DirRoot``: 包含当前工作目录完整路径的仿真根名称（输入）
- ``RFrlMom``: 风轮折叠力矩（N·m）（输出）

源文件 ``ED_UserSubs.f90`` 包含一个虚拟的 ``UserRFrl()`` 例程；用你自己的例程替换它并重新编译 ElastoDyn。

**RFrlMod**
风轮折叠弹簧和阻尼器可以用三种方式建模。当 ``RFrlMod`` 为 0 时，没有风轮折叠弹簧和阻尼器，通常产生的力矩设置为零。当 ``RFrlMod`` 为 1 时，将使用以下提供的输入作为适当系数调用简单弹簧和阻尼器模型。如果 ``RFrlMod`` 设置为 2，ElastoDyn 将调用例程 ``UserRFrl()`` 来计算风轮折叠弹簧和阻尼器力矩。你应该用自己的例程替换代码提供的虚拟例程，该例程需要与 ElastoDyn 的其余部分链接。使用 0、1 或 2 以外的值将导致 ElastoDyn 中止。（开关）

**RFrlSpr**
线性风轮折叠弹簧恢复力矩通过此常数与风轮折叠偏转角成正比。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/rad）

**RFrlDmp**
线性风轮折叠阻尼力矩通过此常数与风轮折叠速率成正比。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）

**RFrlCDmp**
这种库仑摩擦阻尼力矩抵抗风轮折叠运动，但它是一个常数，与风轮折叠速率不成正比。但是，如果风轮折叠速率为零，阻尼也为零。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m）

**RFrlUSSP**
当风轮折叠偏转角超过此值时，风轮折叠上止动弹簧生效。此值必须大于 -180 且小于等于 180 度，仅当 ``RFrlMod`` 设置为 1 时使用。（度）

**RFrlDSSP**
当风轮折叠偏转角超过此值时，风轮折叠下止动弹簧生效。此值必须大于 -180 且小于等于 ``RFrlUSSP`` 度，仅当 ``RFrlMod`` 设置为 1 时使用。（度）

**RFrlUSSpr**
线性风轮折叠上止动弹簧恢复力矩通过此常数与风轮折叠上止动偏转成正比，当风轮折叠偏转角超过 ``RFrlUSSP`` 时生效。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/rad）

**RFrlDSSpr**
线性风轮折叠下止动弹簧恢复力矩通过此常数与风轮折叠下止动偏转成正比，当风轮折叠偏转角超过 ``RFrlDSSP`` 时生效。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/rad）

**RFrlUSDP**
当风轮折叠偏转角超过此值时，风轮折叠上止动阻尼器生效。此值必须大于 -180 且小于等于 180 度，仅当 ``RFrlMod`` 设置为 1 时使用。（度）

**RFrlDSDP**
当风轮折叠偏转角超过此值时，风轮折叠下止动阻尼器生效。此值必须大于 -180 且小于等于 ``RFrlUSDP`` 度，仅当 ``RFrlMod`` 设置为 1 时使用。（度）

**RFrlUSDmp**
线性风轮折叠上止动阻尼力矩通过此常数与风轮折叠速率成正比，当风轮折叠偏转角超过 ``RFrlUSDP`` 时生效。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）

**RFrlDSDmp**
线性风轮折叠下止动阻尼恢复力矩通过此常数与风轮折叠速率成正比，当风轮折叠偏转角超过 ``RFrlDSDP`` 时生效。此值不得为负，仅当 ``RFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）

*尾翼折叠*

通过将 ``TFrlMod`` 设置为 0，尾翼折叠轴承可以是无摩擦的理想轴承；通过将 ``TFrlMod`` 设置为 1，可以使用标准模型，包括线性弹簧和阻尼器，以及上、下止动弹簧和上、下止动阻尼器。
公式见 :numref:`ed_rtfrl_theory`。
ElastoDyn 将止动弹簧建模为尾翼折叠偏转的线性函数。尾翼折叠止动从指定角度开始，根据超过止动角的偏转角作为线性弹簧工作。尾翼折叠阻尼器是折叠速率的线性函数，从指定的上止动和下止动角度开始工作。这些阻尼器是双向的，一旦超过止动角，对两个方向的运动都有相同的阻力。

还提供了用户自定义的尾翼折叠弹簧和阻尼器模型。要使用它，请将 ``TFrlMod`` 设置为 2，并创建一个名为 ``UserTFrl()`` 的子例程，包含参数 ``TFrlDef``、``TFrlRate``、``ZTime``、``DirRoot`` 和 ``TFrlMom``：

- ``TFrlDef``: 当前尾翼折叠角偏转（弧度）（输入）
- ``TFrlRate``: 当前尾翼折叠角速度（rad/s）（输入）
- ``ZTime``: 当前仿真时间（秒）（输入）
- ``DirRoot``: 包含当前工作目录完整路径的仿真根名称（输入）
- ``TFrlMom``: 尾翼折叠力矩（N·m）（输出）

源文件 ``ED_UserSubs.f90`` 包含一个虚拟的 ``UserTFrl()`` 例程；用你自己的例程替换它并重新编译 ElastoDyn。

**TFrlMod**
尾翼折叠弹簧和阻尼器可以用三种方式建模。当 ``TFrlMod`` 为 0 时，没有尾翼折叠弹簧和阻尼器，通常产生的力矩设置为零。当 ``TFrlMod`` 为 1 时，将使用以下提供的输入作为适当系数调用简单弹簧和阻尼器模型。如果将 ``TFrlMod`` 设置为 2，ElastoDyn 将调用例程 ``UserTFrl()`` 来计算尾翼折叠弹簧和阻尼器力矩。你应该用自己的例程替换代码提供的虚拟例程，该例程需要与 ElastoDyn 的其余部分链接。使用 0、1 或 2 以外的值将导致 ElastoDyn 中止。（开关）

**TFrlSpr**
线性尾翼折叠弹簧恢复力矩通过此常数与尾翼折叠偏转角成正比。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/rad）

**TFrlDmp**
线性尾翼折叠阻尼力矩通过此常数与尾翼折叠速率成正比。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）

**TFrlCDmp**
这种库仑摩擦阻尼力矩抵抗尾翼折叠运动，但它是一个常数，与尾翼折叠速率不成正比。但是，如果尾翼折叠速率为零，阻尼也为零。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m）

**TFrlUSSP**
当尾翼折叠偏转角超过此值时，尾翼折叠上止动弹簧生效。此值必须大于 -180 且小于等于 180 度，仅当 ``TFrlMod`` 设置为 1 时使用。（度）

**TFrlDSSP**
当尾翼折叠偏转角超过此值时，尾翼折叠下止动弹簧生效。此值必须大于 -180 且小于等于 ``TFrlUSSP`` 度，仅当 ``TFrlMod`` 设置为 1 时使用。（度）

**TFrlUSSpr**
线性尾翼折叠上止动弹簧恢复力矩通过此常数与尾翼折叠上止动偏转成正比，当尾翼折叠偏转角超过 ``TFrlUSSP`` 时生效。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/rad）

**TFrlDSSpr**
线性尾翼折叠下止动弹簧恢复力矩通过此常数与尾翼折叠下止动偏转成正比，当尾翼折叠偏转角超过 ``TFrlDSSP`` 时生效。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/rad）

**TFrlUSDP**
当尾翼折叠偏转角超过此值时，尾翼折叠上止动阻尼器生效。此值必须大于 -180 且小于等于 180 度，仅当 ``TFrlMod`` 设置为 1 时使用。（度）

**TFrlDSDP**
当尾翼折叠偏转角超过此值时，尾翼折叠下止动阻尼器生效。此值必须大于 -180 且小于等于 ``TFrlUSDP`` 度，仅当 ``TFrlMod`` 设置为 1 时使用。（度）

**TFrlUSDmp**
线性尾翼折叠上止动阻尼力矩通过此常数与尾翼折叠速率成正比，当尾翼折叠偏转角超过 ``TFrlUSDP`` 时生效。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）

**TFrlDSDmp**
线性尾翼折叠下止动阻尼恢复力矩通过此常数与尾翼折叠速率成正比，当尾翼折叠偏转角超过 ``TFrlDSDP`` 时生效。此值不得为负，仅当 ``TFrlMod`` 设置为 1 时使用。（N·m/(rad/s)）
