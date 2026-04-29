.. _ad_driver:

AeroDyn驱动程序
==============
提供了独立的AeroDyn驱动程序，用于对做刚体运动（固定、正弦运动或任意运动）的刚性风机进行气动模拟。独立AeroDyn驱动程序在原先独立的风力机转子性能工具WT_Perf的功能基础上进行了改进。该驱动程序还支持OpenFAST目前不支持的风机配置。
应用示例包括：
- 模拟水平/垂直轴风力机、风筝、四旋翼、多旋翼和机翼。
- 给定风速、变桨、偏航、转子转速时间序列的模拟。
- 组合案例分析，评估不同运行条件下的风机响应，例如计算功率系数（C_P）、推力系数（C_T）和/或扭矩系数（C_Q）作为叶尖速比（TSR）和叶片变桨角度的函数。
- 塔筒基础在不同幅值和频率下振荡运动的模拟。
详情如下。

编译驱动程序
--------------------
AeroDyn驱动程序的编译步骤与OpenFAST类似（参见:numref:`installation`）。使用CMake工作流时，运行`make`会自动编译驱动程序。如果只需要编译驱动程序，使用`make aerodyn_driver`。编译后的驱动程序位于`/build/modules/aerodyn/`文件夹中。Windows系统可使用`/vs-build/AeroDyn`文件夹中的Visual Studio解决方案。

.. _addm_driver_input_file:

驱动输入和选项
-------------------------
**主要概念**
驱动程序支持：
- 两种风机定义方式：基本（水平轴风力机，HAWT）和高级。
- 两种入流定义方式：基本（均匀风切变幂律）和高级（InflowWind）。
- 三种分析类型（`AnalysisType`）：1）一个或多个风机的单次模拟，2）单个风机在时变输入下的单次模拟，3）单个风机的组合案例。
当前限制：
- 所有叶片和所有转子的叶片节点数必须相同。
- 每个风机只能有一个（可选的）塔筒。
- 分析类型2和3仅限1个风机。
详情如下，将介绍输入文件的不同部分。

**头文件和输入配置**
输入文件以头文件开头，用户可以在第二行放置模型的相关描述。接下来是输入配置部分。用户可以切换`Echo`标志，将驱动程序解析后的输入文件回写到磁盘。`MHK`开关允许用户指定是否为MHK涡轮机。`MHK=0`表示非MHK涡轮机，`MHK=1`表示固定式MHK涡轮机，`MHK=2`表示漂浮式MHK涡轮机。驱动程序支持三种分析类型，但并非所有风机格式和入流类型都适用于每种分析类型：
- `AnalysisType=1`：模拟一个或多个基本（HAWT）或任意几何形状（HAWT/VAWT、四旋翼等）的转子，支持基本或高级风输入，以及可选的塔筒基础、机舱和独立变桨角度的时变运动。塔筒基础可做任意运动或正弦运动。
- `AnalysisType=2`：模拟单个风机，支持基本时变风、机舱运动、变桨。参见下文"时变分析"。
- `AnalysisType=3`：单个风机的组合案例分析，使用基本稳态风。驱动程序将按顺序运行案例表。参见下文"组合案例分析"。
接下来是总模拟时间（`TMax`）和模拟时间步长（`DT`）的定义。这些输入适用于`AnalysisType=1`和`AnalysisType=2`。用户通过变量`AeroFile`指定AeroDyn主文件的位置，路径可以是绝对路径，也可以是相对于AeroDyn驱动文件的相对路径。
头文件和输入配置示例如下：
.. code::
    ----- AeroDyn Driver Input File ---------------------------------------------------------
    Three bladed wind turbine, using basic geometry input
    ----- Input Configuration -------------------------------------------------------
    False           Echo         - 是否将输入参数回写到 "<rootname>.ech" 文件？
            0       MHK          - MHK涡轮机类型（开关） {0: 非MHK涡轮机, 1: 固定式MHK涡轮机, 2: 漂浮式MHK涡轮机}
            3       AnalysisType - {1: 多风机单次模拟, 2: 单个风机时变模拟, 3: 单个风机组合案例}
           11.0     TMax         - 总运行时间 [仅当AnalysisType≠3时使用] (s)
            0.5     DT           - 模拟时间步长 [仅当AnalysisType≠3时使用] (s)
    "AD.dat"        AeroFile     - AeroDyn主输入文件名

**环境条件**
环境条件在此指定并传递给AeroDyn。`FldDens`（相当于AeroDyn主输入文件中的`AirDens`）指定流体密度，必须大于零；空气（风力机）的典型值约为1.225 kg/m³，海水（MHK涡轮机）约为1025 kg/m³。`KinVisc`指定流体的运动粘度（用于雷诺数计算）；空气的典型值约为1.460E-5 m²/s，海水约为1.004E-6 m²/s。`SpdSound`是流体中的声速（用于非定常翼型气动计算中的马赫数计算）；空气的典型值约为340.3 m/s，海水约为1500 m/s。该部分的最后两个参数仅在MHK涡轮机的`CavitCheck = TRUE`时使用。`Patm`是自由液面上方的大气压，典型值约为101325 Pa。`Pvap`是流体的蒸气压，海水的典型值约为2000 Pa。`WtrDpth`是从海床到平均海平面（MSL）的水深。

**入流数据**
入流可以通过两种方式提供：
- 基本（`CompInflow=0`）：符合幂律切变的均匀风。风场使用参考高度（`RefHt`）、幂律指数（`PLExp`）和参考高度处的风速（`HWindSpeed`）定义。对于`AnalysisType=2`，参考风速和幂律指数作为时间序列单独定义（参见"时变分析"）。对于`AnalysisType=3`，这些参数在单独的表中提供（参见"组合案例分析"）。所有分析类型都使用参考高度，因为该高度可能与轮毂高度不同。给定节点处的风速使用以下公式确定，其中:math:`Z`是陆基风机节点相对于地面的瞬时高程，海上风机相对于平均海平面（MSL）的瞬时高程，或固定和漂浮式MHK涡轮机相对于海床的瞬时高程。
.. math::
   U(Z) = \mathrm{HWindSpeed} \left( \frac{Z}{\mathrm{RefHt}} \right)^\mathrm{PLExp}
- 高级（`CompInflow=1`）：使用InflowWind模块计算入流，支持InflowWind的所有可用选项。用户需要提供InflowWind输入文件的（相对或绝对）路径（`InflowFile`）。该功能仅限`AnalysisType=1`使用。
输入示例如下：
.. code::
    ----- Inflow Data ---------------------------------------------------------------
              0   CompInflow  - 计算入流风速（开关） {0: 稳态风; 1: 使用InflowWind模块}
    "unused"      InflowFile  - InflowWind输入文件名 [仅当CompInflow=1时使用]
            9.0   HWindSpeed  - 水平风速   [仅当CompInflow=0且AnalysisType=1时使用] (m/s)
            140   RefHt       - 水平风速参考高度 [仅当CompInflow=0时使用]  (m)
           0.10   PLExp       - 幂律指数   [仅当CompInflow=0且AnalysisType=1时使用]                        (-)

**海况数据**
AeroDyn驱动程序可以调用SeaState来定义波场，作为入流信息的一部分。对于有波浪和海流的MHK涡轮机，SeaState将查询InflowWind，并将波浪和海流场的速度和加速度相加。如果激活SeaState，也必须激活InflowWind，不过如果需要可以将海流设为0。该部分的输入示例如下：
.. code::
    ----- SeaState Data ---------------------------------------------------------------------
    1                               CompSeaSt   - 计算波浪速度（开关） {0: 无波浪; 1: 使用SeaState模块}
    "MHK_RM1_Floating_SeaState.dat" SeaStFile   - SeaState输入文件名 [仅当CompSeaSt=1时使用]

**风机数据**
用户按如下方式指定风机数量：
.. code::
    ----- Turbine Data --------------------------------------------------------------
    1   NumTurbines  - 风机数量（AnalysisType=2或AnalysisType=3时必须为1）
如注释所述，`AnalysisType=2`和`AnalysisType=3`的风机数量应为1。该部分之后，提供每个风机的几何形状和运动。每个风机的输入必须带有后缀`(i)`，其中`i`是风机编号（即使`NumTurbines=1`，`i=1`）。每个风机的输出将写入不同的文件，后缀为`.Ti`，其中`i`是风机编号（仅使用1个风机时不添加后缀）。
图:numref:`fig:MultiRotor`展示了双风力机的配置示例。该图定义了与每个风机相关的不同坐标系和原点：风机基础坐标系（t）、机舱坐标系（n）、轮毂坐标系（h）和叶片坐标系（b）。符号和约定遵循OpenFAST坐标系，不同之处在于风机坐标系的原点不在塔筒底部。
风机的预定运动施加在风机原点。偏航绕:math:`z_n`轴进行，转子绕:math:`x_h`轴旋转，叶片变桨绕各自的:math:`z_b`轴进行。使用基本（HAWT）输入格式时，不同坐标系的定义是标准化的；使用高级输入格式时，可任意定义。更多详情见下一段。
.. figure:: figs/MultiRotor.png
   :width: 80%
   :name: fig:MultiRotor
   多转子定义示意图。

**风机几何定义**
支持两种风机输入格式：
- 基本（`BasicHAWTFormat=True`）：基本水平轴风力机（HAWT）格式。在该格式中，风机几何形状完全由叶片数（`NumBlades`）、轮毂半径（`HubRad`）、轮毂高度（`HubHt`）、悬伸量（`Overhang`）、轴倾角（`ShftTilt`）、预锥角（`Precone`）和塔筒顶部到转子轴的垂直距离（`Twr2Shft`）决定，如图:numref:`fig:BasicGeometry`所示。每个参数的定义遵循ElastoDyn约定。例如，`HubRad`指定从旋转中心到沿（可能带预锥的）叶片变桨轴的叶根的半径，必须大于零。`HubHt`指定轮毂中心相对于陆基风机地面的高度，海上风机和漂浮式MHK涡轮机相对于平均海平面（MSL）的高度，或固定式MHK涡轮机相对于海床的高度。对于轮毂位于MSL以下的漂浮式MHK涡轮机，`HubHt`应为负值。`Overhang`指定塔筒中心线和轮毂中心之间沿（可能倾斜的）转子轴的距离，向下游为正（迎风式风机使用负数）。`ShftTilt`是转子轴与水平面之间的夹角（度），`ShftTilt`为正表示轴的下游端最高（迎风式风机使用负的`ShftTilt`以增加塔筒净空）。对于漂浮式MHK涡轮机，应翻转`ShftTilt`的符号以获得等效的轴倾角。例如，漂浮式迎风MHK涡轮机使用正的`ShftTilt`以增加塔筒净空。`Precone`是平转子盘和叶片扫掠锥面之间的夹角（度），向下游为正（迎风式风机使用负的`Precone`以增加塔筒净空）。`Twr2Shft`是塔筒顶部到转子轴的垂直距离。对于转子在塔筒顶部下方的漂浮式MHK涡轮机，该值应为负值。
.. figure:: figs/aerodyn_driver_geom.png
   :width: 60%
   :name: fig:BasicGeometry
   基本风机几何定义示意图。
此外，用户需要提供`t=0`时风机基础的原点（`BaseOriginInit`）。对于固定式MHK涡轮机，`BaseOriginInit`相对于海床输入。对于漂浮式MHK涡轮机，`BaseOriginInit`相对于MSL输入，如果风机基础在MSL以下，垂直分量为负。基本输入示例如下：
.. code::
    ----- Turbine(1) Geometry -------------------------------------------------------
            True    BasicHAWTFormat(1) - 输入格式切换开关 {True: 下7行为基本输入, False: 需提供基础/塔筒/机舱/轮毂/叶片几何和运动}
           0,0,0    BaseOriginInit(1) - 全局坐标系下风机基础坐标 (m)
               3    NumBlades(1)    - 叶片数 (-)
              3.    HubRad(1)       - 轮毂半径 (m)
          140.82513 HubHt(1)        - 轮毂高度 (m)
              -7    Overhang(1)     - 悬伸量 (m)
              -6    ShftTilt(1)     - 轴倾角 (deg)
              -4    Precone(1)      - 叶片预锥角 (deg)
         3.09343    Twr2Shft(1)     - 塔筒顶部到转子轴的垂直距离 (m)
- 高级（`BasicHAWTFormat=False`）：塔筒基础、机舱、轮毂和单个叶片的位置和方向可任意定义。可用于HAWT和任何其他风机概念。不同坐标系的定义见图:numref:`fig:MultiRotor`。风机基础坐标系的位置（`BaseOriginInit`）和方向（`BaseOrientationInit`）相对于全局坐标系定义。`BaseOriginInit`的垂直分量对于固定式MHK涡轮机相对于海床定义，对于漂浮式MHK涡轮机相对于MSL定义。方向使用三个连续旋转值（x-y-z欧拉角序列）给出。如果基础发生运动，基础坐标系的方向将由时变旋转加上这些初始旋转组成。
下一行给出风机是否有塔筒的标志（`HasTower`）。该标志目前仅影响VTK输出，尚未对AeroDyn产生影响。用户仍需要在AeroDyn中为每个风机提供塔筒输入数据（参见:numref:`ad_inputs_multirot`）。下一行指定AeroDyn计算中使用的投影方式。建议HAWT使用`HAWTprojection=True`，这是AeroDyn中使用的默认投影方式（投影到带预锥的变桨轴）。对于其他转子概念，设置`HAWTprojection=False`。后续几行指定塔筒、机舱和轮毂的位置和方向。
塔筒和机舱相对于风机基础（t）原点和坐标系定义。假设塔筒顶部与机舱原点重合。AeroDyn输入文件中定义的塔筒站相对于塔筒原点给出，这与OpenFAST使用地面/MSL作为参考不同（参见:numref:`ad_inputs_multirot`）。对于漂浮式MHK涡轮机，如果塔筒原点和机舱原点低于风机基础，则`TwrOrigin_t`和`NacOrigin_t`的垂直分量为负。
轮毂相对于机舱原点和坐标系（n）定义。对于漂浮式MHK涡轮机，如果轮毂原点低于机舱原点（即塔筒顶部），则`HubOrigin_n`的垂直分量为负。
接下来是叶片定义，从叶片数`NumBlades`开始。支持零叶片转子，可用于模拟孤立塔筒。如果在AeroDyn中使用塔筒阴影/势流模型，当使用OLAF时，孤立塔筒会扰乱涡尾流。使用BEM时，给定风机的叶片流动仅受该风机塔筒的扰动。风机`i`和叶片`j`的输入标记为`(i_j)`。每个叶片的原点（`BldOrigin_h`）和方向（`BldOrientation_h`）相对于轮毂原点和坐标系（h）给出。提供轮毂半径输入（`BldHubRad_Bl`）是为了方便，它们将有效地沿:math:`z_b`轴偏移叶片原点。高级几何定义的输入示例如下。该示例对应典型的3叶片迎风HAWT，倾角为6度（OpenFAST中为-6），预锥角为-4度（叶片向上游倾斜）。
.. code::
    ----- Turbine(1) Geometry -------------------------------------------------------
         False      BasicHAWTFormat(1) - 输入格式切换开关 {True: 下7行为基本输入, False: 需提供基础/塔筒/机舱/轮毂/叶片几何和运动}
    0,0,0           BaseOriginInit(1)      - 全局坐标系下风机基础原点坐标 (m)
    0,0,0           BaseOrientationInit(1) - 基础坐标系相对于全局坐标系的初始方向，使用连续三次旋转角(theta_x, theta_y, theta_z)表示，即横摇、纵摇、偏航 (deg)
    True            HasTower(1)            - 风机是否有塔筒（标志）
    True            HAWTprojection(1)      - 是否为水平轴风机（用于AeroDyn投影，标志）
    0,0,0           TwrOrigin_t(1)         - 基础坐标系下塔筒原点坐标 [仅当HasTower为True时使用] (m)
    0,0,137         NacOrigin_t(1)         - 基础坐标系下机舱原点（及塔筒顶部）坐标 (m)
    -6.96,0.,3.82   HubOrigin_n(1)         - 机舱坐标系下轮毂原点坐标 (m)
    0,6,0           HubOrientation_n(1)    - 轮毂坐标系相对于机舱坐标系的初始方向，使用连续三次旋转角(theta_x, theta_y, theta_z)表示。x轴需与旋转轴对齐。(deg)
    ----- Turbine(1) Blades -----------------------------------------------------------------
    3               NumBlades(1)          - 当前转子的叶片数 (-)
    0,0,0           BldOrigin_h(1_1)      - 轮毂坐标系下叶片1原点坐标 (m)
    0,0,0           BldOrigin_h(1_2)      - 轮毂坐标系下叶片2原点坐标 (m)
    0,0,0           BldOrigin_h(1_3)      - 轮毂坐标系下叶片3原点坐标 (m)
    0  ,-4,0        BldOrientation_h(1_1) - 叶片坐标系相对于轮毂坐标系的初始方向，使用连续三次旋转角(theta_x, theta_y, theta_z)表示，需保证z轴沿展向，y轴沿后缘（无变桨时），参数为方位角、预锥角、变桨角 (deg)
    120,-4,0        BldOrientation_h(1_2) - 叶片坐标系相对于轮毂坐标系的初始方向，使用连续三次旋转角(theta_x, theta_y, theta_z)表示，需保证z轴沿展向，y轴沿后缘（无变桨时），参数为方位角、预锥角、变桨角 (deg)
    240,-4,0        BldOrientation_h(1_3) - 叶片坐标系相对于轮毂坐标系的初始方向，使用连续三次旋转角(theta_x, theta_y, theta_z)表示，需保证z轴沿展向，y轴沿后缘（无变桨时），参数为方位角、预锥角、变桨角 (deg)
    3.0             BldHubRad_bl(1_1)     - 径向输入数据起始处的叶片坐标系z向偏移量，即轮毂半径 (m)
    3.0             BldHubRad_bl(1_2)     - 径向输入数据起始处的叶片坐标系z向偏移量，即轮毂半径 (m)
    3.0             BldHubRad_bl(1_3)     - 径向输入数据起始处的叶片坐标系z向偏移量，即轮毂半径 (m)

**风机运动定义**
风机运动的定义仅在`AnalysisType=1`时使用，但必须始终出现在输入文件中。
基础运动的定义对于基本几何和高级几何是相同的。基础运动可以是：固定（`BaseMotionType=0`）、正弦（`BaseMotionType=1`）或任意（`BaseMotionType=2`）。在应用风机基础的初始位置和方向之前，每个时间步都会施加风机基础运动。正弦运动意味着风机基础的一个自由度（`DegreeOfFreedom`）按照给定幅值（`Amplitude`）和频率（`Frequency`，单位Hz）、零相位的正弦函数运动。6个可能的自由度对应于基础坐标系在全局坐标系（g）中的平动或转动（例如纵荡、横荡、垂荡、横摇、纵摇、艏摇）。任意运动通过CSV文件（`BaseMotionFileName`）指定，文件包含19列：时间、3个平动位移（全局）、三个连续旋转角（全局）、3个平动速度、3个转动速度（omega，全局坐标系下）、3个平动加速度和3个转动加速度（alpha，全局坐标系下）。任意输入文件的示例参见:numref:`ad_inputfiles_examples`。运动文件中的时间向量必须是升序的，但不需要是线性的。驱动程序使用线性插值来确定给定时间的输入。内部不会检查位移/方向、速度和加速度的一致性。
正弦纵荡运动的输入示例如下：
.. code::
    ----- Turbine(1) Motion [仅当AnalysisType=1时使用] --------------------------
    1         BaseMotionType(1)      - 基础运动类型 {0: 固定, 1: 正弦运动, 2: 任意运动} (开关)
    1         DegreeOfFreedom(1)     - 运动自由度 {1:xg, 2:yg, 3:zg, 4:theta_xg, 5:theta_yg, 6:theta_zg} [仅当BaseMotionType=1时使用] (开关)
    5.0       Amplitude(1)           - 正弦运动幅值  [仅当BaseMotionType=1时使用] (m 或 rad)
    0.1       Frequency(1)           - 正弦运动频率  [仅当BaseMotionType=1时使用] (Hz)
    "unused"  BaseMotionFileName(1)  - 任意基础运动文件名（19列：时间、x、y、z、theta_x...alpha_z） [仅当BaseMotionType=2时使用]
基本几何和高级几何的不同输入如下：
- 基本：基本风机的运动包括恒定的机舱偏航（`NacYaw`，机舱绕垂直塔筒轴的正旋转，向下看时逆时针方向）、转子转速（`RotSpeed`，向下游看时顺时针为正）和叶片变桨（`BldPitch`，绕:math:`z_b`轴负方向）。对于漂浮式MHK涡轮机，应翻转`NacYaw`的符号以获得相同的全局偏航方向（即机舱绕垂直塔筒轴的正旋转，向下看时为顺时针方向）。示例如下：
.. code::
    0         NacYaw(1)        - 机舱偏航角（绕z_t轴） (deg)
    7         RotSpeed(1)      - 转子坐标系下的转子转速 (rpm)
    1         BldPitch(1)      - 叶片变桨角 (deg)
- 高级：提供高级几何且叶片数非零时，运动部分包含机舱运动、转子运动和独立叶片变桨运动的选项。每个运动的语法包括定义类型（固定或时变）、固定情况的值或时变情况的文件。输入文件是包含时间、位置、速度和加速度的CSV文件。文件示例参见:numref:`ad_inputfiles_examples`。内部不会检查位移/方向、速度和加速度的一致性。运动文件中的时间向量必须是升序的，但不需要是线性的。驱动程序使用线性插值来确定给定时间的输入。CSV文件中的角度和旋转数据以rad和rad/s定义，而驱动输入文件中的角度和旋转数据以deg和rpm定义。固定转速的示例如下：
.. code::
    0         NacMotionType(1)       - 机舱运动类型 {0: 固定偏航, 1: 时变偏航角} (开关)
    0         NacYaw(1)              - 机舱偏航角（绕z_t轴） [仅当NacMotionType=0时使用] (deg)
    "unused"  NacMotionFileName(1)   - 偏航运动文件名 [仅当NacMotionType=1时使用]
    0         RotMotionType(1)       - 转子运动类型 {0: 恒定转速, 1: 时变转速} (开关)
    6.0       RotSpeed(1)            - 转子坐标系下的转速 [仅当RotorMotionType=0时使用] (rpm)
    "unused"  RotMotionFileName(1)   - 转子运动文件名 [仅当RotorMotionType=1时使用]
    0         BldMotionType(1)       - 叶片变桨运动类型 {0: 固定, 1: 时变变桨} (开关)
    0         BldPitch(1_1)          - 叶片1变桨角 [仅当BldMotionType=0时使用] (deg)
    0         BldPitch(1_2)          - 叶片2变桨角 [仅当BldMotionType=0时使用] (deg)
    0         BldPitch(1_3)          - 叶片3变桨角 [仅当BldMotionType=0时使用] (deg)
    "unused"  BldMotionFileName(1_1) - 叶片1变桨运动文件名 [仅当BldMotionType=1时使用]
    "unused"  BldMotionFileName(1_2) - 叶片2变桨运动文件名 [仅当BldMotionType=1时使用]
    "unused"  BldMotionFileName(1_3) - 叶片3变桨运动文件名 [仅当BldMotionType=1时使用]

**时变分析**
时变分析用于在模拟过程中改变几个标准变量。这些变量包括：参考风速（`HWndSpeed`）、幂律指数（`PLExp`）、转子转速（`RotSpd`）、集体变桨（`Pitch`）和机舱偏航（`Yaw`）。每个变量的时间序列在CSV文件（`TimeAnalysisFileName`）中提供。时变分析通过`AnalysisType=2`选择，仅限单个风机（`numTurbines=1`）。
.. code::
    ----- 时变分析 [仅当AnalysisType=2且numTurbines=1时使用] ------------
    "TimeSeries.csv" TimeAnalysisFileName - 时间序列文件名（6列：时间、HWndSpeed、PLExp、RotSpd、Pitch、Yaw）。

**组合案例分析**
组合案例分析用于在一次运行中完成参数化研究。通过`AnalysisType=3`选择，仅限单个风机（`numTurbines=1`）。每个模拟可改变的变量包括：参考风速（`HWndSpeed`）、幂律指数（`PLExp`）、转子转速（`RotSpd`）、集体变桨（`Pitch`）、机舱偏航（`Yaw`）、时间步长（`dT`）、模拟时间（`Tmax`）和正弦运动参数（自由度`DOF`、幅值和频率）。当`DOF=0`时，风机基础固定。
.. code::
    ----- 组合案例分析 [仅当AnalysisType=3且numTubrines=1时使用] ------
             4  NumCases     - 运行案例数
    HWndSpeed  PLExp   RotSpd   Pitch   Yaw    dT      Tmax   DOF   Amplitude  Frequency
    (m/s)      (-)     (rpm)    (deg)  (deg)   (s)     (s)    (-)  (m or rad)  (Hz)
       8.      0.0       6.     0.      0.     1.0     100     0      0         0.0
       8.      0.0       6.     0.      0.     1.0     100     0      0         0.0
       9.      0.1       7.     1.      0.     0.5      50     1      5.0       0.1
       9.      0.2       8.     2.      0.     0.5      50     1      2.0       0.2

**输出**
输出部分控制表格输出文件和VTK文件的格式，类似于OpenFAST输出。用户可以控制VTK可视化的轮毂半径和机舱尺寸。轮毂表示为半径为（`VTKHubRad`）的球体，机舱表示为使用原点和三个平行于机舱坐标系的长度定义的平行六面体（`VTKNacDim`）。
.. code::
    ----- 输出设置 -------------------------------------------------------------------
      "ES15.8E2"     OutFmt      - 文本表格输出格式（不含时间列）。结果字段应为10字符。（带引号字符串）
    2                OutFileFmt  - 时程表格输出文件格式（开关） {1: 文本文件 [<RootName>.out], 2: 二进制文件 [<RootName>.outb], 3: 两者都有}
    0                WrVTK       - VTK可视化数据输出：（开关） {0: 不输出; 1: 仅初始时刻; 2: 动画输出}
    2                VTKHubRad   - VTK可视化轮毂半径 (m)
    -1,-1,-1,2,2,2   VTKNacDim   - VTK可视化机舱尺寸参数，x0,y0,z0为原点坐标，Lx,Ly,Lz为三个方向长度 (m)

.. _ad_inputs_multirot:

多风机的AeroDyn输入
------------------------------------
使用单个风机时，无需修改AeroDyn输入文件。为了最小化多风机实现的影响，驱动程序目前对所有风机仅使用一个AeroDyn输入文件。这意味着当前所有转子的AeroDyn选项都相同。
当使用超过三个叶片和多个风机时，需要调整叶片文件的定义以及塔筒、轮毂和机舱输入。

**叶片文件**
传统AeroDyn格式要求至少三个叶片文件名。因此，目前所有转子的叶片在`ADBlFile`列表中连续列出。列表按风机和风机叶片循环填充，叶片索引是最快变化的索引。目前，所有叶片的站数必须相同。
两个风机的示例如下，第一个风机有3个叶片，第二个有2个叶片：
.. code::
    ====== 转子/叶片属性 =====================================================================
    True                   UseBlCm     - 计算中是否包含气动俯仰力矩？(标志)
    "AD_Turbine1_blade1.dat" ADBlFile(1) - 叶片1分布式气动属性文件名 (-)
    "AD_Turbine1_blade1.dat" ADBlFile(2) - 叶片2分布式气动属性文件名 (-)
    "AD_Turbine1_blade3.dat" ADBlFile(3) - 叶片3分布式气动属性文件名 (-)
    "AD_Turbine2_blade1.dat" ADBlFile(4) - 叶片4分布式气动属性文件名 (-)
    "AD_Turbine2_blade2.dat" ADBlFile(5) - 叶片5分布式气动属性文件名 (-)

**轮毂和机舱输入**
定义轮毂和机舱参数的部分也必须为每个风机重复。
两个风机的示例如下：
.. code::
    ====== 轮毂属性 ============================================================================= [仅当MHK=1或2时使用]
    7.0   VolHub                - 轮毂体积 (m^3)
    0.0   HubCenBx              - 轮毂浮力中心x向偏移量 (m)
    ====== 轮毂属性 ============================================================================= [仅当MHK=1或2时使用]
    5.0   VolHub                - 轮毂体积 (m^3)
    0.2   HubCenBx              - 轮毂浮力中心x向偏移量 (m)
    ====== 机舱属性 ============================================================================= [仅当MHK=1或2，或NacelleDrag=True时使用]
    32.0            VolNac      - 机舱体积 (m^3)
    0.3, 0.0, 0.05  NacCenB     - 机舱坐标系下，偏航轴承到机舱浮力中心的位置 (m)
    4.67, 20.15, 20.15 NacArea  - 机舱坐标系下，机舱在x、y、z方向的投影面积 (m^2)
    0.5, 0.5, 0.5   NacCd       - 上述三个方向的机舱阻力系数 (-)
    0.43, 0, 0      NacDragAC   - 机舱坐标系下，机舱阻力的气动中心位置 (m)
    ====== 机舱属性 ============================================================================= [仅当MHK=1或2，或NacelleDrag=True时使用]
    32.0            VolNac      - 机舱体积 (m^3)
    0.3, 0.0, 0.05  NacCenB     - 机舱坐标系下，偏航轴承到机舱浮力中心的位置 (m)
    4.67, 20.15, 20.15 NacArea  - 机舱坐标系下，机舱在x、y、z方向的投影面积 (m^2)
    0.5, 0.5, 0.5   NacCd       - 上述三个方向的机舱阻力系数 (-)
    0.43, 0, 0      NacDragAC   - 机舱坐标系下，机舱阻力的气动中心位置 (m)

**气动塔筒输入**
AeroDyn的整个塔筒输入部分必须为每个风机重复，包括设置为没有塔筒的风机（`hasTower=False`）。每个风机的站数可以不同。AeroDyn输入文件中定义的塔筒站相对于塔筒原点给出，这与OpenFAST使用地面/MSL作为参考不同。
两个风机的示例如下：
.. code::
    ====== 风机(1)塔筒影响和气动 ============================================================ [仅当TwrPotent≠0, TwrShadow≠0, TwrAero=True, 或MHK=1或2时使用]
    2   NumTwrNds   - 分析中使用的塔筒节点数 (-) [仅当TwrPotent≠0, TwrShadow≠0, TwrAero=True, 或MHK=1或2时使用]
    TwrElev TwrDiam  TwrCd    TwrTI   TwrCb
    (m)       (m)     (-)     (-)     (-)
     0.0      2.0     1.0    0.1      0.0
    10.0      1.0     1.0    0.1      0.0
    ====== 风机(2)塔筒影响和气动 ============================================================ [仅当TwrPotent≠0, TwrShadow≠0, TwrAero=True, 或MHK=1或2时使用]
    3   NumTwrNds   - 分析中使用的塔筒节点数 (-) [仅当TwrPotent≠0, TwrShadow≠0, TwrAero=True, 或MHK=1或2时使用]
    TwrElev TwrDiam  TwrCd   TwrTI   TwrCb
    (m)       (m)     (-)    (-)     (-)
     0.0      4.0     1.0    0.1     0.0
    15.0      3.0     1.0    0.1     0.0
    30.0      2.0     1.0    0.1     0.0

.. _ad_inputfiles_examples:

驱动输入文件示例
------------------------------
r-test仓库中提供了使用驱动程序不同功能的可运行示例：
- `开发分支 <https://github.com/OpenFAST/r-test/tree/dev/modules/aerodyn/>`_ 。
- `主分支 <https://github.com/OpenFAST/r-test/tree/main/modules/aerodyn/>`_ 。

主驱动输入文件
~~~~~~~~~~~~~~~~~~~~~~~
以下是基本入流、基本HAWT和组合案例分析的AeroDyn驱动程序示例：
.. code::
    ----- AeroDyn驱动输入文件 ---------------------------------------------------------
    三叶片风力机，使用基本几何输入
    ----- 输入配置 -------------------------------------------------------------------
    False           Echo         - 是否将输入参数回写到 "<rootname>.ech" 文件？
            0       MHK          - MHK涡轮机类型（开关） {0: 非MHK涡轮机, 1: 固定式MHK涡轮机, 2: 漂浮式MHK涡轮机}
            3       AnalysisType - {1: 多风机单次模拟, 2: 单个风机时变模拟, 3: 单个风机组合案例}
           11.0     TMax         - 总运行时间 [仅当AnalysisType≠3时使用] (s)
            0.5     DT           - 模拟时间步长 [仅当AnalysisType≠3时使用] (s)
    "./AD.dat"      AeroFile - AeroDyn主输入文件名
    ----- 环境条件 ------------------------------------------------------------------
    1.225000000000000e+00     FldDens      - 工作流体密度 (kg/m^3)
    1.477551020408163e-05     KinVisc      - 工作流体运动粘度 (m^2/s)
    3.350000000000000e+02     SpdSound     - 工作流体中的声速 (m/s)
    1.035000000000000e+05     Patm         - 大气压 (Pa) [仅MHK涡轮机空化检查时使用]
    1.700000000000000e+03     Pvap         - 工作流体蒸气压 (Pa) [仅MHK涡轮机空化检查时使用]
                            0     WtrDpth      - 水深 (m)
    ----- 入流数据 --------------------------------------------------------------------
              0     CompInflow  - 计算入流风速（开关） {0: 稳态风; 1: 使用InflowWind模块}
    "unused"        InflowFile  - InflowWind输入文件名 [仅当CompInflow=1时使用]
            9.0     HWindSpeed  - 水平风速   [仅当CompInflow=0且AnalysisType=1时使用] (m/s)
            140     RefHt       - 水平风速参考高度 [仅当CompInflow=0时使用]  (m)
           0.10     PLExp       - 幂律指数   [仅当CompInflow=0且AnalysisType=1时使用]                        (-)
    ----- 海况数据 --------------------------------------------------------------------
              0     CompSeaSt   - 计算波浪速度（开关） {0: 无波浪; 1: 使用SeaState模块}
    "unused"        SeaStFile   - SeaState输入文件名 [仅当CompSeaSt=1时使用]
    ----- 风机数据 --------------------------------------------------------------------
    1               NumTurbines - 风机数量
    ----- 风机(1)几何 ---------------------------------------------------------------
            True    BasicHAWTFormat(1) - 输入格式切换开关 {True: 下7行为基本输入, False: 需提供基础/塔筒/机舱/轮毂/叶片几何和运动}
           0,0,0    BaseOriginInit(1) - 基础坐标系下塔筒底部坐标 (m)
               3    NumBlades(1)    - 叶片数 (-)
              3.    HubRad(1)       - 轮毂半径 (m)
          140.82513 HubHt(1)        - 轮毂高度 (m)
              -7    Overhang(1)     - 悬伸量 (m)
              -6    ShftTilt(1)     - 轴倾角 (deg)
              -4    Precone(1)      - 叶片预锥角 (deg)
         3.09343    Twr2Shft(1)     - 塔筒顶部到转子轴的垂直距离 (m)
    ----- 风机(1)运动 [仅当AnalysisType=1时使用] ---------------------------------
    1               BaseMotionType(1)      - 基础运动类型 {0: 固定, 1: 正弦运动, 2: 任意运动} (开关)
    1               DegreeOfFreedom(1)     - 运动自由度 {1:xg, 2:yg, 3:zg, 4:theta_xg, 5:theta_yg, 6:theta_zg} [仅当BaseMotionType=1时使用] (开关)
    5.0             Amplitude(1)           - 正弦运动幅值  [仅当BaseMotionType=1时使用] (m 或 rad)
    0.1             Frequency(1)           - 正弦运动频率  [仅当BaseMotionType=1时使用] (Hz)
    ""              BaseMotionFileName(1)  - 任意基础运动文件名（19列：时间、x、y、z、theta_x...alpha_z） [仅当BaseMotionType=2时使用]
    0               NacYaw(1)              - 机舱偏航角（绕z_t轴） (deg)
    7               RotSpeed(1)            - 转子坐标系下的转子转速 (rpm)
    1               BldPitch(1)            - 叶片1变桨角 (deg)
    ----- 时变分析 [仅当AnalysisType=2且numTurbines=1时使用] ------------
    "unused"         TimeAnalysisFileName - 时间序列文件名（6列：时间、HWndSpeed、PLExp、RotSpd、Pitch、Yaw）。
    -----  组合案例分析 [仅当AnalysisType=3且numTurbines=1时使用] -------------
             4  NumCases     - 运行案例数
    HWndSpeed  PLExp  RotSpd  Pitch   Yaw   dT    Tmax  DOF  Amplitude Frequency
    (m/s)      (-)    (rpm)   (deg)  (deg)  (s)   (s)   (-)   (-)       (Hz)
      8.0      0.0     6.      0.      0.   1.0   100    0    0          0
      8.0      0.0     6.      0.      0.   1.0   100    0    0          0
      9.0      0.1     7.      1.      0.   0.5   51     1    5.0        0.1
      9.0      0.2     8.      2.      0.   0.51  52     1    2.0        0.2
    ----- 输出设置 -------------------------------------------------------------------
    "ES15.8E2"       OutFmt      - 文本表格输出格式（不含时间列）。结果字段应为10字符。（带引号字符串）
    2                OutFileFmt  - 时程表格输出文件格式（开关） {1: 文本文件 [<RootName>.out], 2: 二进制文件 [<RootName>.outb], 3: 两者都有}
    0                WrVTK       - VTK可视化数据输出：（开关） {0: 不输出; 1: 仅初始时刻; 2: 动画输出}
    2                VTKHubRad   - VTK可视化轮毂半径 (m)
    -1,-1,-1,2,2,2   VTKNacDim   - VTK可视化机舱尺寸参数，x0,y0,z0为原点坐标，Lx,Ly,Lz为三个方向长度 (m)

运动输入文件
~~~~~~~~~~~~~~~~~~
运动文件中的时间向量必须是升序的，但不需要是线性的。驱动程序使用线性插值来确定给定时间的输入。
任意基础运动文件：
.. code::
    time_[s] , x_[m]    , y_[m]    , z_[m]    , theta_x_[rad] , theta_y_[rad] , theta_z_[rad] , xdot_[m/s] , ydot_[m/s] , zdot_[m/s] , omega_x_g_[rad/s] , omega_y_g_[rad/s] , omega_z_g_[rad/s] , xddot_[m^2/s] , yddot_[m^2/s] , zddot_[m^2/s] , alpha_x_g_[rad/s] , alpha_y_g_[rad/s] , alpha_z_g_[rad/s]
    0.000000 , 0.000000 , 0.000000 , 0.000000 , 0.000000      , 0.000000      , 0.000000      , 0.000000   , 0.000000   , 10.053096  , 0.000000          , 0.000000          , 0.000000          , 0.000000      , 0.000000      , -0.000000     , 0.000000          , 0.000000          , 0.000000
    0.100000 , 0.000000 , 0.000000 , 0.963507 , 0.000000      , 0.000000      , 0.000000      , 0.000000   , 0.000000   , 8.809596   , 0.000000          , 0.000000          , 0.000000          , 0.000000      , 0.000000      , -24.344157    , 0.000000          , 0.000000          , 0.000000
偏航运动文件：
.. code::
    time_[s] , yaw_[rad] , yaw_rate_[rad/s] , yaw_acc_[rad/s^2]
    0.000000 , 0.000000  , 0.000000         , 0.000000
    0.100000 , 0.007277  , 0.212647         , 4.029093
转子运动文件：
.. code::
    time_[s] , azimuth_[rad] , omega_[rad/s] , rotacc_[rad/s^2]
    0.000000 , 0.000000      , 0.000000      , 0.000000
    0.100000 , 0.000000      , 0.000000      , 0.000000
变桨运动文件：
.. code::
    time_[s] , pitch_[rad] , pitch_rate_[rad/s] , pitch_acc_[rad/s^2]
    0.000000 , 0.000000    , 0.000000           , 0.000000
    0.100000 , 0.000000    , 0.000000           , 0.000000
    0.200000 , 0.000000    , 0.000000           , 0.000000
