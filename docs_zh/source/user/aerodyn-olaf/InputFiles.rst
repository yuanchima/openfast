.. _OLAF-Input-Files:

输入文件
===========

除非在表格中指定了行数，否则不应在输入文件中添加或删除行。

单位
-----

OLAF使用国际单位制（例如kg、m、s、N）。除非另有说明，角度单位为度。

OLAF主输入文件
-----------------------

OLAF主输入文件定义了自由尾迹的常规选项、环量模型的选择和规格、近尾迹和远尾迹长度，以及尾迹可视化选项。文件中的每个部分对应OLAF模型的一个方面。对于大多数参数，用户可以指定值为"default"（带或不带引号），在这种情况下，程序将使用下面定义的默认值。

示例OLAF主输入文件见:numref:`OLAF-Primary-Input-File`。

常规选项
~~~~~~~~~~~~~~~

**IntMethod** [开关] 指定用于对流拉格朗日标记点的积分方法。有四个选项：1) 四阶龙格-库塔 *[1]*，2) 四阶亚当斯-巴什福思 *[2]*，3) 四阶亚当斯-巴什福思-莫尔顿 *[3]*，4) 一阶前向欧拉 *[5]*。默认选项是 *[5]*。目前仅实现了选项 *[1]* 和 *[5]*。这些方法在:numref:`sec:vortconv`中说明。

**DTfvw** [秒] 指定模块更新尾迹的时间间隔。该时间间隔必须是*AeroDyn*使用的时间步长的整数倍。叶片环量在每个中间时间步根据中间叶片位置和风速进行更新。默认值为:math:`dt_{aero}`，其中:math:`dt_{aero}`是AeroDyn使用的时间步长。

**FreeWakeStart** [秒] 指定尾迹演化被归类为"自由"的时间。在达到这个时间点之前，拉格朗日标记点仅以来流速度对流。在此时间点之后，计算诱导速度并影响标记点的对流。如果给定的时间小于或等于零，则从仿真开始时尾迹就是"自由"的。默认值为:math:`0`。

**FullCircStart** [秒] 指定叶片环量达到其全部强度的时间。如果该值指定为:math:`>0`，则环量在:math:`t=0`时乘以系数:math:`0`，并线性增加到:math:`t>\textit{FullCircStart}`时的系数:math:`1`。默认值为:math:`0`。

环量规格
~~~~~~~~~~~~~~~~~~~~~~~~~~

**CircSolvMethod** [开关] 指定使用的环量方法。有三个选项：1) 基于:math:`C_l`的迭代过程 *[1]*，2) 无穿流条件 *[2]*，3)  prescribed *[3]*。默认选项是 *[1]*。这些方法在:numref:`sec:circ`中描述。

**CircSolvConvCrit** [-] 指定求解环量时使用的无量纲收敛准则。仅当*CircSolvMethod* = *[1]*时使用此变量。默认值为:math:`0.001`，对应两次迭代之间环量的:math:`0.1\%`误差。

**CircSolvRelaxation** [-] 指定求解环量时使用的松弛因子。仅当*CircSolvMethod* = *[1]*时使用此变量。默认值为:math:`0.1`。

**CircSolvMaxIter** [-] 指定求解环量时使用的最大迭代次数。仅当*CircSolvMethod* = *[1]*时使用此变量。默认值为:math:`30`。

**PrescribedCircFile** [引号字符串] 指定包含预设叶片环量的文件。仅当*CircSolvMethod* = *[3]*时使用此选项。环量文件格式是带分隔符的文件，包含一个标题行和两列。第一列是无量纲径向位置 [r/R]；第二列是附着环量值，单位为[m\ :sup:`2`/s]。径向位置不需要与AeroDyn节点位置匹配。示例预设环量文件见:numref:`Prescribed-Circulation-Input-File`。


尾迹范围和离散化选项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


**nNWPanels** [-] 指定近尾迹（NW）面板的数量（即FVW时间步长，**DTfvw**），用于定义近尾迹格架的范围。设置此参数的建议见:numref:`Guidelines-OLAF`。

**nNWPanelsFree** [-] 指定近尾迹面板的数量（以FVW时间步长为单位），在这些面板中，尾迹以"自由"方式对流。
如果*nNWPanelsFree*等于*nNWPanels*，则整个近尾迹都是自由的。否则，位于由*nNWPanelsFree*和*nNWsPanel*界定的缓冲区（"冻结近尾迹"）内的拉格朗日标记点都以共同的衰减诱导速度对流，但来流速度是变化的（见:numref:`sec:vortconvfrozen`）。
此选项可用于加快仿真速度并稳定"近尾迹"区域的末端。它有可能消除对远尾迹区域的需求。
目前，冻结近尾迹的诱导速度被任意确定为自由近尾迹最后20个面板的平均值。对流速度的衰减使得诱导速度在冻结近尾迹末端约为50%。冻结尾迹的对流速度需要额外的验证和确认，并且可能在未来版本中发生变化。
如果使用"冻结"近尾迹区域，则"自由"远尾迹区域的长度需要为零（**nFWPanelsFree=0**）。
默认情况下，此变量设置为**nNWPanels**（无冻结尾迹）。
设置此参数的建议见:numref:`Guidelines-OLAF`。

**nFWPanels** [-] 指定远尾迹使用的面板数量（FVW时间步长），远尾迹中叶尖涡和根涡被卷起以加快计算时间。设置此参数的建议见:numref:`Guidelines-OLAF`。默认值：0。


**nFWPanelsFree** [-] 指定远尾迹面板的数量（以FVW时间步长为单位），在这些面板中，尾迹以"自由"方式对流。
如果*nFWPanelsFree*等于*nFWPanels*，则整个远尾迹都是自由的。否则，位于由*nNWPanelsFree*和*nNWPanels*界定的缓冲区（"冻结远尾迹"）内的拉格朗日标记点都以共同的诱导速度对流，但来流速度是变化的。
目前，当**nNWPanelsFree=nNWPanels**（即无冻结近尾迹）时，冻结远尾迹的诱导速度被确定为自由远尾迹的平均值；否则，使用与冻结近尾迹末端相同的对流速度。
默认情况下，此变量设置为**nFWPanels**。
设置此参数的建议见:numref:`Guidelines-OLAF`。



**FWShedVorticity** [标志] 指定远尾迹中是否包含脱落涡量。默认值为*[False]*，表示远尾迹仅包含来自根涡和叶尖涡的拖曳涡量。

尾迹正则化和扩散选项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**DiffusionMethod** [开关] 指定用于考虑粘性扩散的扩散方法。有两个选项：1) 无扩散 *[0]* 和 2) 核心扩展方法 *[1]*。默认选项是 *[0]*。

**RegDeterMethod** [开关] 指定用于确定正则化参数的方法。有四个选项：1) 常数 *[0]*、2) 优化 *[1]*、3) 弦长 *[2]* 和 4) 展长 *[3]*。
优化选项为用户确定本节中的所有参数。优化选项仍在开发中，不推荐使用。
常数选项要求用户指定本节中的所有参数。
默认和推荐选项是 *[3]*。


.. math::

   r_{c,\text{wake}}(r) = \text{WakeRegParam}
   ,\quad
   r_{c,\text{blade}}(r) = \text{WingRegParam}

当**RegDeterMethod==2**时，正则化参数根据当地弦长设置：

.. math::

   r_{c,\text{wake}}(r) = \text{WakeRegParam} \cdot c(r)
   ,\quad
   r_{c,\text{blade}}(r) = \text{WingRegParam} \cdot c(r)

当**RegDeterMethod==3**时，正则化参数根据展向离散化设置：

.. math::

   r_{c,\text{wake}}(r) = \text{WakeRegParam} \cdot \Delta r(r)
   ,\quad
   r_{c,\text{blade}}(r) = \text{WingRegParam} \cdot \Delta r(r)

其中:math:`Delta r`是展向站的长度。设置此参数的建议见:numref:`Guidelines-OLAF`。



**RegFunction** [开关] 指定用于消除涡元素奇异性的正则化函数，如:numref:`sec:vortconv`中所述。有五个选项：1) 无修正 *[0]*、2) 朗肯方法 *[1]*、3) 兰姆-奥森方法 *[2]*、4) 瓦西斯塔斯方法 *[3]* 和 5) 分母偏移方法 *[4]*。
这些函数在:numref:`sec:RegularizationFunction`中给出。默认选项是 *[3]*。

**WakeRegMethod** [开关] 指定确定粘性核心半径（即正则化参数）的方法。有三个选项：1) 常数 *[1]*、2) 拉伸 *[2]* 和 3) 年龄 *[3]*。这些方法在:numref:`sec:corerad`中描述。默认选项是 *[3]*。

**WakeRegFactor** [m, 或 -] 指定尾迹正则化参数，这是涡元素初始化时使用的正则化值。如果正则化方法是"常数"，则该值在整个尾迹中使用。设置此参数的建议见:numref:`Guidelines-OLAF`。

**WingRegFactor** [m, 或 -] 指定附着涡量正则化参数，这是用于附着在叶片上的涡量元素的正则化值。设置此参数的建议见:numref:`Guidelines-OLAF`。

**CoreSpreadEddyVisc** [-] 指定涡粘性参数:math:`\delta`。该参数用于核心扩展方法（*DiffusionMethod* = *[1]*）和随年龄变化的正则化方法（*WakeRegMethod* = *[3]*）。变量:math:`\delta`在:numref:`sec:corerad`中描述。默认值为:math:`100`。

尾迹处理选项
~~~~~~~~~~~~~~~~~~~~~~

**TwrShadowOnWake** [标志] 指定塔筒势流和塔筒阴影是否对尾迹对流有影响。当在AeroDyn中激活时，塔筒阴影模型始终对升力线有影响，因此会影响叶片上的诱导和载荷。此选项仅涉及尾迹。默认选项是 *[False]*。

**ShearVorticityModel** [开关] 指定除了*InflowWind*规定的剪切入流之外，是否对剪切涡量进行建模。有两个选项：1) 不处理 *[0]* 和 2) 镜像涡量 *[1]*。镜像涡量考虑了地面效应。后续版本将实现专门的选项来考虑剪切涡量。无论此输入如何，剪切速度剖面都由*InflowWind*处理。默认选项是 *[0]*。


加速选项
~~~~~~~~~~~~~~~

**VelocityMethod** [开关] 指定用于确定速度的方法。有四个选项：
1) 涡段的:math:`N^2`毕奥-萨伐尔计算 *[1]*，
2) 粒子树公式 *[2]*，
3) 使用粒子表示的:math:`N^2`毕奥-萨伐尔计算，
4) 段树公式。
选项 *[2]* 和 *[3]* 需要指定*PartPerSegment*（见下文）。
选项 *[4]* 预计会给出与选项 *[1]* 接近的结果，同时提供显著的加速，并且此选项不需要指定*PartPerSegment*。
默认选项是 *[2]*。


**TreeBranchFactor** [-] 指定无量纲距离（以分支半径为单位），超过该距离时使用多极计算代替直接求值。仅当*VelocityMethod* = *[2,4]*时使用。默认值：1.5。

**PartPerSegment** [-] 指定当涡段由涡粒子表示时使用的粒子数量。仅当*VelocityMethod* = *[2,3]*时使用。默认值为:math:`1`。

输出选项
~~~~~~~~~~~~~~

**WrVTK** [标志] 指定是否写入可视化工具包（VTK）可视化文件。*WrVTK* = *[0]* 不写入任何VTK文件。*WrVTK* = *[1]* 按*VTK_fps*定义的时间步输出VTK文件。*WrVTK* = *[2]*，按*VTK_fps*定义的时间步输出，但确保在仿真开始和结束时都写入文件（通常与`VTK_fps=0`一起使用，仅在仿真结束时输出）。输出写入文件夹``vtk_fvw.``。参数*WrVTK*、*VTKCoord*和*VTK_fps*独立于胶合代码的VTK输出选项。默认值：0。


**nVTKBlades** [-] 指定要写入多少个叶片VTK文件。*nVTKBlades* :math:`= n` 输出:math:`n`个叶片的VTK文件，0是可接受的值。默认值为:math:`0`。

**VTKCoord** [开关] 指定写入VTK文件时使用的坐标系。有两个选项：1) 全局坐标系 *[1]* 和 2) 轮毂坐标系 *[2]*。默认选项是 *[1]*。

**VTK_fps** [:math:`1`/秒] 指定VTK文件的输出频率。提供的值被四舍五入到最接近的时间步的允许倍数。默认值为:math:`1/dt_\text{fvw}`。指定*VTK_fps* = *[all]*相当于使用值:math:`1/dt_\text{aero}`。如果*VTK_fps<0*，则不创建输出，除非*WrVTK=2*。

**nGridOut** [-] 指定网格输出的数量。默认值为0。网格输出是在规则笛卡尔网格上导出的场（速度、涡量）。它们使用后续行中的表格定义，包含两行标题。用户需要指定用于VTK输出文件名的名称（**GridName**）、网格类型（**GridType**）、开始时间（**TStart**）、结束时间（**TEnd**）、时间间隔（**DTOut**），以及每个方向上的网格范围，例如**XStart**、**XEnd**、**nX**。
当**GridType**为1时，速度场被写入磁盘；当**GridType**为2时，速度场和涡量场（使用有限差分计算）都被写入。可以在点（**nX=nY=nZ=1**）、线、平面或3D网格上导出场。
当设置为"default"时，开始时间为0，结束时间设置为仿真结束时间。输出在:math:`t_{Start}\leq t \leq t_{End}`范围内进行。
当变量**DTOut**设置为"all"时，使用AeroDyn时间步长；当设置为"default"时，使用OLAF时间步长。
输入示例如下：

.. code::

    3       nGridOut           网格输出数量
    GridName  GridType  TStart  TEnd     DTOut     XStart    XEnd   nX    YStart   YEnd    nY    ZStart   ZEnd   nZ
    (-)         (-)      (s)     (s)      (s)        (m)      (m)    (-)    (m)     (m)     (-)    (m)     (m)    (-)
    "box"        2     default default  all        -200     1000.    5    -150.   150.    20      5.     300.    30
    "vert"       1     default default  default    -200     1000.   100     0.     0.     1       5.     300.    30
    "hori"       1     default default  2.0        -200     1000.   100   -150.   150.    20     100.    100.    1

在此示例中，第一个名为"box"的网格在AeroDyn时间步导出，由形状为5x20x30、尺寸为1200x300x295的盒子组成。该网格包含速度和涡量。另外两个网格是仅包含速度的垂直和水平面。


高级选项
~~~~~~~~~~~~~~~~


高级选项（通常用于开发人员或测试版功能）可以放在OLAF输入文件的末尾：

- 这些选项使用常规格式：`Value Key - Comment`。
- 它们可以按任意顺序放置。
- 可以在行首使用`!`字符注释掉它们。
- 空行或不支持的选项将被忽略（显示到屏幕）。
- 如果未提供，将使用默认值，但高级选项不支持"DEFAULT"关键字。
- 这些是**测试版功能**，主要应由开发人员使用。


OLAF输入文件的末尾看起来如下：

.. code::

   [...]
   GridName  GridType  TStart  TEnd     DTGrid    XStart    XEnd   nX    YStart   YEnd    nY    ZStart   ZEnd   nZ
   (-)         (-)      (s)     (s)      (s)        (m)      (m)    (-)    (m)     (m)     (-)    (m)     (m)    (-)
   ===============================================================================================
   --------------------------- 高级选项 --------------------------------------------------
   ===============================================================================================
   ! 高级选项可以按任意顺序放在这里，使用常规格式：
   ! Value1   Key1            - Comment1
   ! Value2   Key2            - Comment2
   ! 以`!`开头的行被忽略，空行被忽略
   [...] 等等

目前支持的高级选项如下：

.. code::

   ===============================================================================================
   --------------------------- 高级选项 --------------------------------------------------
   ===============================================================================================
   "Panels.vtk"  SrcPnlFile     - 包含源面板的VTK文件名 {默认：""}
   1             nSrcPnlUpdate  - 源面板更新的频率（以OLAF时间步为单位），{默认：1}
   True          Induction      - 计算从尾迹到叶片和尾迹到尾迹的诱导速度，{默认：True}
   True          InductionAtCP  - 在节点（False）或控制点（CP，True）计算诱导速度，{默认：True}
   True          WakeAtTE       - 在后缘（True）或直接在LL（False，无弦向面板）处开始尾迹，{默认：True}
   False         DStallOnWake   - 动态失速对尾迹有影响，{默认：False}
   0.75          kFrozenNWStart - 冻结尾迹开始处的尾迹诱导速度分数，{默认：0.75}
   0.5           kFrozenNWEnd   - 冻结尾迹结束处的尾迹诱导速度分数，{默认：0.5}
   0.0           zGround        - 地面高度，禁止涡点低于该高度，如果存在则将它们推回**zGroundPush**定义的高度 {默认：0.0}
   0.1           zGroundPush    - 地面推回高度，见**zGround** {默认：0.1}



**SrcPanlFile** [字符串] 指定用于源面板方法的VTK文件名。VTK文件应为传统ASCII VTK文件，包含DATASET POLYDATA POINTS和POLYGONS。
下面提供了两个在XY平面上形成规则网格的面板的示例VTK文件。多边形的连接需要使得法线指向远离物体并进入流体的方向。
在下面的示例中，从上方看时多边形按顺时针定义，这导致OLAF内部定义的法线指向`+z`方向。这种配置是底部壁面、上方为流体的典型情况：OLAF法线从壁面指向流体。然而，当应用右手定则时，得到的法线从流体指向物体内部。
当使用**WrVTK**时，OLAF将写入单独的VTK文件，包含与源面板相关的各种信息，例如压力系数、面积、单位面积力、法线。查看法线的方向非常重要（在Paraview中，选择3D Glyph，Arrows，并选择`Normals`输出，按`Normals`缩放）。
BodyID CELL_DATA可用于分离不同的补丁，这有助于后处理。这是可选的，OLAF目前不使用它，但它会被写回OLAF的输出文件中。

感兴趣的读者可以查看OLAF的单元测试`Test_SrcPnl_Sphere`，该测试测试了球体周围的压力系数。

.. code::

   # vtk DataFile Version 2.0
   Comment
   ASCII
   DATASET POLYDATA
   POINTS 6 double
   0.0 0.0 0.0
   0.0 25.0 0.0
   0.0 50.0 0.0
   10.0 0.0 0.0
   10.0 25.0 0.0
   10.0 50.0 0.0

   POLYGONS 2 10
   4 0 1 4 3
   4 1 2 5 4

   CELL_DATA 2
   SCALARS BodyID int
   LOOKUP_TABLE default
   0
   0



**nSrcPnlUpdate** [整数] 定义源面板更新的频率（以OLAF时间步为单位），默认值为`1`，即每个时间步更新。

**Induction** [开关] 计算诱导速度，否则所有诱导速度都为0（无尾迹、无面板等）！默认值为`True`。

**InductionAtCP** [开关] 执行升力线计算时，在节点（False）或控制点（CP，True）计算诱导速度。默认值为`True`。

**WakeAtTE** [开关] 在后缘开始尾迹（True），或直接在LL（False，无弦向面板）处开始。默认值为`True`。

**DStallOnWake** [开关] 包括动态失速对尾迹的影响（True），即使用动态Cl更新升力线和尾迹面板的环量。默认值为`False`。

**kFrozenNWStart** [浮点数] 冻结尾迹开始处的尾迹诱导速度分数，默认值为`0.75`。见与冻结NW相关的OLAF理论，:numref:`sec:vortconvfrozen`。

**kFrozenNWEnd** [浮点数] 冻结尾迹结束处的尾迹诱导速度分数，默认值为`0.5`。见与冻结NW相关的OLAF理论，:numref:`sec:vortconvfrozen`。

**zGround** [浮点数] 禁止涡点低于该高度（以米为单位），如果存在，则将它们推回**zGroundPush**定义的高度以上（以米为单位）。默认值为`0.0`。对于MHK，海床位置基于水深添加到**zGround**中。

**zGroundPush** [浮点数] 地面推回高度，见**zGround**。默认值为`0.1`。




AeroDyn输入文件
--------------------
输入文件修改
~~~~~~~~~~~~~~~~~~~~~~~~

由于OLAF已集成到*AeroDyn*模块中，因此在*AeroDyn*输入文件中添加了尾迹计算选项和一行内容。这些添加如下：

**WakeMod** 指定使用的尾迹模型类型。已添加*WakeMod* = *[3]*，允许用户从传统BEM方法切换到OLAF方法。

**FVWFile** [字符串] 指定OLAF模块文件，路径相对于AeroDyn文件，除非提供绝对路径。


相关部分
~~~~~~~~~~~~~~~~~
当*WakeMod* = *[3]*时，BEM选项（例如叶尖损失、偏斜和动态模型）会被读取并丢弃。以下部分和参数仍然相关并被涡代码使用：

  - 常规选项（例如翼型和塔筒建模）；
  - 环境条件；
  - 动态失速模型选项；
  - 翼型和叶片信息；
  - 塔筒空气动力学；以及
  - 输出。
