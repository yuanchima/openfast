
.. _AA-usage:

在 AeroDyn 中使用气动噪声模型
==============================

本文档的实时版本可在 https://openfast.readthedocs.io/ 获取。要运行气动噪声模型，需要在 AeroDyn 主输入文件第 14 行的 **General Options** 输入块中将 **CompAA** 标志设置为 **True**。当该标志设置为 **True** 时，下一行必须包含气动噪声模型输入文件的名称，这在 :numref:`aa-sec-MainInput` 中讨论。目前，此模块不能用于 MHK（海洋水电）涡轮机。


.. container::
   :name: aa-tab:AeroDyn

   .. literalinclude:: example/AeroDyn.ipt
      :linenos:
      :language: none


.. _aa-sec-MainInput:

主输入文件
-----------

气动噪声主输入文件包含一系列输入和标志，应根据要运行的分析进行适当设置。这些分为子字段：常规选项、气动噪声模型、观察者输入和输出。

从常规选项开始，这些是：

-  **Echo** – True/False：选择是否用正确的模板重写输入文件

-  **DT_AA** – 浮点数：气动噪声计算的时间步长。只能使用 AeroDyn 时间步长 **DTAero** 的倍数。如果设置为默认值，则采用 DTAero 时间步长。

-  **AAStart** – 浮点数：气动噪声模块开始运行的时间。

-  **BldPrcnt** – 浮点数：从叶尖测量的叶片展向百分比，该部分贡献噪声排放；100% 对应从叶尖到叶根的整个叶片。

气动噪声模型字段列出了实际噪声模型的所有标志：

-  **TIMod** – 整数 0/1/2：设置湍流来流噪声模型的标志；0 关闭该模型，1 对应 :numref:`aa-amiet` 中讨论的 Amiet 模型，2 对应 :numref:`aa-guidati` 中介绍的简化 Guidati 模型。

-  **TICalcMeth** – 整数 1/2：设置入射湍流强度计算方法的标志。设置为 1 时，入射湍流强度由用户定义。设置为 2 时，入射湍流强度由入射流的时间历史估算。

-  **TI** – 浮点数：用户定义的 :math:`TI` 值，即 Amiet 模型中使用的转子入射湍流强度。

-  **avgV** – 浮点数：用于缩放 :math:`TI` 并将其转换为叶片截面入射湍流强度的平均风速值。

-  **Lturb** – 浮点数：用于估算 Amiet 模型中使用的湍流长度尺度的 :math:`L_{turb}` 值。

-  **TBLTEMod** – 整数 0/1/2：设置 TBL-TE 噪声模型的标志；0 关闭该模型，1 使用 Brooks-Pope-Marcolini (BPM) 翼型噪声模型（见 :numref:`aa-turb-TE-bpm`），2 使用 :numref:`aa-turb-TE-tno` 中描述的 TNO 模型。

-  **BLMod** – 整数 1/2：设置边界层特性计算方法的标志；1 使用 BPM 模型的简化方程，2 加载 :numref:`aa-sec-BLinputs` 中描述的文件。仅当 **TBLTEMod** 不为零时使用。

-  **TripMod** – 整数 0/1/2：如果 BLMod 设置为 1，对于非转捩边界层（**TRipMod=0**）、强转捩边界层（**TRipMod=1**）或弱转捩边界层（**TRipMod=2**）使用不同的半经验参数；2 通常用于运行中的风力机，而 1 常用于风洞翼型模型。

-  **LamMod** – 整数 0/1：激活层流边界层-涡脱落模型的标志，见 :numref:`aa-laminar-vortex`。

-  **TipMod** – 整数 0/1：激活叶尖涡模型的标志，见 :numref:`aa-tip-vortex`。

-  **RoundedTip** – True/False：如果 **TipMod=1**，此标志在圆形叶尖（True）和方形叶尖（False）之间切换，见 :numref:`aa-tip-vortex`。

-  **Alprat** – 浮点数：叶尖处升力系数曲线的斜率值；见 :numref:`aa-tip-vortex`。

-  **BluntMod** – 整数 0/1：激活（**BluntMod=1**）后缘钝度-涡脱落模型的标志，见 :numref:`aa-TE-vortex`。如果该标志设置为 1，必须按照 :numref:`aa-sec-BLinputs` 中的描述在文件中指定后缘几何形状。

观察者位置字段包含文件路径，该文件指定了观察者数量（NrObsLoc）和各自的位置；见 :numref:`aa-sec-ObsPos`。

最后，输出集包含几个输出数据选项：

-  **AWeighting** – True/False：设置声压级是否采用（True）或不采用（False）A 计权修正的标志；见 :numref:`aa-sec-ModelUsage`。

-  **NAAOutFile** – 整数 1/2/3/4：设置所需输出文件的标志。设置为 1 时，每个观察者每个 **DT_AA** 时间步的总声压级值打印到文件。设置为 2 时，在第一个输出之外还有第二个文件，打印每个观察者每个时间步的总声压级谱。设置为 3 时，在前两个输出文件之外还有第三个文件，打印每个观察者每个时间步每个噪声机制的声压级谱。设置为 4 时，生成第四个文件，包含每个节点、每个叶片、每个观察者、每个时间步的总声压级值。

-  下一行 **AAOutFile** 包含用于存储输出的文件的根名称。如果设置为 "default"，将使用默认输出文件根名称。文件名会根据 **NAAOutFile** 选项附加 1、2、3 和 4 标志。

文件必须以 END 命令结束。

.. container::
   :name: aa-tab:AeroAcousticsInput

   .. literalinclude:: example/AeroAcousticsInput.dat
      :linenos:
      :language: none


.. _aa-sec-BLinputs:

边界层输入和后缘几何形状
-------------------------

当标志 **BLMod** 设置为 2 时，必须提供预制表的边界层特性，供湍流边界层-后缘噪声模型使用。文件名应在翼型极坐标系数文件输入的 BL_file 字段中指定。每个气动站位必须指定一个翼型文件。

.. container::
   :name: aa-tab:AFtab

   .. literalinclude:: example/AFtab.dat
      :linenos:
      :language: none


在这个示例中名为 **AF20_BL.txt** 的文件包含 8 个输入，这些输入针对给定数量的雷诺数 ReListBL 和给定数量的攻角 aoaListBL 制表。这些输入以无量纲形式定义，必须为后缘上下的翼型吸力面和压力面提供，它们是：

-  **Ue_Vinf** – 边界层顶部的流动速度

-  **Dstar** – :math:`\delta^{*}`，边界层位移厚度

-  **Delta** – :math:`\delta`，名义边界层厚度

-  **Cf** – 摩擦系数。

在以下示例中，文件是通过运行边界层求解器 XFoil 的 Python 脚本 [4]_ 生成的。注意，XFoil 默认不返回 :math:`\delta`，而是返回边界层动量厚度 :math:`\theta`。:math:`\delta` 可以使用 :cite:`aa-Drela:1987` 的表达式重建：

.. math::
   \delta = \theta \bullet \left( 3.15 + \frac{1.72}{H - 1} \right) + \delta^{*}
   :label:  eq:35

其中 :math:`H` 是运动形状因子，这也是 XFoil 的标准输出之一。由于通常不可能在整个雷诺数和攻角范围内获得这些值，代码设置为采用最后可用的值并在屏幕上打印警告。

当标志 **BluntMod** 设置为 1 时，还必须沿展向定义后缘的详细几何形状。需要提供两个输入，即刚好在后缘点之前的剖面吸力面和压力面之间的角度 :math:`\Psi`，以及后缘高度 :math:`h`。:math:`\Psi` 必须以度为单位定义，而 :math:`h` 以米为单位。注意，BPM 后缘钝度模型对这两个参数非常敏感，然而对于实际叶片来说，这两个参数通常不容易确定。:numref:`aa-fig:GeomParamTE` 展示了这两个输入。

.. figure:: media/NoiseN011.png
   :alt:    后缘钝度的几何参数
   :name:   aa-fig:GeomParamTE
   :width:  100.0%

   后缘钝度的几何参数 :math:`\mathbf{\Psi}` 和 :math:`\mathbf{h}`

每个文件必须定义一个 :math:`\Psi` 值和一个 :math:`h` 值。如果标志 **BluntMod** 设置为 0，则不使用这些值。

.. container::
   :name: aa-tab:AF20_BL

   .. literalinclude:: example/AF20_BL.txt
      :linenos:
      :language: none


.. _aa-sec-ObsPos:

观察者位置
-----------

观察者的数量和位置在 ObserverLocations 文件中设置，这在 :numref:`aa-sec-MainInput` 中解释。位置必须在 OpenFAST 全局惯性坐标系中指定，该坐标系位于塔筒底部，x 轴指向下风方向，y 轴指向侧向，z 轴垂直向上。观察者坐标系的示意图如 :numref:`aa-fig:ObsRefSys` 所示。

.. figure:: media/NoiseN010.png
   :alt:    观察者参考系
   :name:   aa-fig:ObsRefSys
   :align:  center
   :width:  40.0%

   观察者参考系

:numref:`tab:ref-turb` 中展示的 IEA 风能任务 37 陆上参考风力机轮毂高度为 110 米，转子半径为 65 米，其符合国际电工委员会 61400-11 标准的观察者位于：

x = 175 [m]

y = 0 [m]

z = 0 [m].

这里展示了一个列出四个位于 2 米高度观察者的文件示例：

.. container::
   :name: aa-tab:observer

   .. literalinclude:: example/Observer.txt
      :linenos:
      :language: none


.. [4]
   https://github.com/OpenFAST/python-toolbox
