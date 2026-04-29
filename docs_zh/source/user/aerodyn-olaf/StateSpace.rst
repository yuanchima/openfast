
.. role:: raw-latex(raw)
   :format: latex
..

状态空间表示及与 OpenFAST 的集成
=================================

状态、约束、输入和输出变量
---------------------------

OLAF 模块已通过 *AeroDyn* 集成到最新版本的 OpenFAST 中，遵循 OpenFAST 模块化框架 (:cite:`olaf-Jonkman13_1,olaf-Sprague15_1`)。为符合 OpenFAST 框架，涡旋代码被编写为一个模块，其公式包含状态方程、约束方程和输出方程。模块处理的数据包括以下向量：常量参数 :math:`\vec{p}`、输入 :math:`\vec{u}`、约束状态 :math:`\vec{z}`、状态 :math:`\vec{x}` 以及输出 :math:`\vec{y}`。这些向量定义如下：

-  参数 :math:`\vec{p}` — 一组内部系统值，与状态和输入无关。参数可以在初始化时完全定义，用于表征系统状态和输出方程。

-  输入 :math:`\vec{u}` — 提供给模块的一组值，与状态一起用于计算未来状态和系统输出。

-  约束状态 :math:`\vec{z}` — 代数变量，基于当前时间步的值使用非线性求解器计算。

-  状态 :math:`\vec{x}` — 模块的一组内部值。它们受输入影响，用于计算未来状态值和输出。这里采用连续状态，意味着状态在时间上可微，并由连续时间微分方程表征。

-  输出 :math:`\vec{y}` — 模块计算并返回的一组值，通过输出方程依赖于状态、输入和/或参数。

涡旋代码的参数包括：

-  流体特性：运动粘度 :math:`\nu`。

-  翼型特性：弦长 :math:`c` 和极曲线数据 — :math:`C_l(\alpha)`、:math:`C_d(\alpha)`、:math:`C_m(\alpha)`）。

-  算法方法和参数，例如正则化、粘性扩散、离散化、尾流几何形状和加速度。

涡旋代码的输入是：

-  升力线不同节点的位置、方向、平移速度和旋转速度（分别为 :math:`\vec{r}_{ll}`、:math:`\Lambda_{ll}`、:math:`\vec{\dot{r}}_{ll}` 和 :math:`\vec{\omega}_{ll}`），为简洁起见，这些量被收集到向量 :math:`\vec{x}_{\text{elast},ll}` 中。这些量使用 OpenFAST 的网格映射功能和数据结构处理。

-  请求位置处的扰动速度场，记为 :math:`\vec{V}_0=[\vec{V}_{0,ll}, \vec{V}_{0,m}]`。请求的位置包括升力线点 :math:`\vec{r}_{ll}` 和拉格朗日标记点 :math:`\vec{r}_m`。根据参数，此扰动速度场可能包含以下影响：来流、风切变、风偏、湍流、塔筒和机舱扰动。通常请求速度场的位置是拉格朗日标记点的位置。

约束状态是：

-  沿升力线的环量强度 :math:`\Gamma_{ll}`。

连续状态是：

-  拉格朗日标记点的位置 :math:`\vec{r}_m`

-  与每个涡旋元素相关的涡量 :math:`\vec{\omega}_e`。对于涡量在涡旋段上的投影，这对应于环量 :math:`\vec{\Gamma}_e`。对于每个段，:math:`\vec{\Gamma}_e= \Gamma_e \vec{dl}_e =\vec{\omega}_e dV_e`，其中 :math:`\vec{dl}_e` 和 :math:`dV_e` 分别是涡旋段长度及其等效涡旋体积。

输出为 [1]_：

-  升力线节点处的诱导速度 :math:`\vec{v}_{i,ll}`

-  计算未扰动风的位置 :math:`\vec{r}_{r}`（通常 :math:`\vec{r_{r}}=\vec{r}_m`）。

状态、约束和输出方程
---------------------

本节概述状态、约束和输出方程。更多细节请参见 :numref:`OLAF-Theory`。约束方程用于确定每个升力线沿展向的环量分布。对于 van Garrel 方法，此环量是沿叶片的攻角和翼型系数的函数。给定升力线节点处的攻角是该点处未扰动速度 :math:`\vec{v}_{0,ll}` 和涡量诱导速度 :math:`\vec{v}_{i,ll}` 的函数。部分诱导速度由当前时间步脱落和拖出的涡量引起，而这又是沿升力线环量分布的函数。此约束方程可以写为：

.. math::
   \vec{Z} = \vec{0} = \vec{\Gamma}_{ll} - \vec{\Gamma}_p\bigg(\vec{\alpha}(\vec{x},\vec{u}),\vec{p}\bigg)

其中 :math:`\vec{\Gamma}_p` 是根据 :numref:`sec:circ` 中介绍的方法之一返回沿叶片展向环量的函数。

状态方程规定了涡量的时间演化和拉格朗日标记点的对流：

.. math::
   \begin{aligned}
       \frac{d \vec{\omega}_e}{dt} &= \bigg[(\vec{\omega}\cdot\nabla)\vec{v} + \nu\nabla^2 \vec{\omega} \bigg]_e
   \end{aligned}

.. math::
   \begin{aligned}
       \frac{d \vec{r}_m}{dt} &= \vec{V}(\vec{r}_m)
    =\vec{V}_0(\vec{r}_m)  + \vec{v}_\omega(\vec{r}_m)
    =\vec{V}_0(\vec{r}_m)  + \vec{V}_\omega(\vec{r}_m, \vec{r}_m, \vec{\omega})
   \end{aligned}
   :label: eq:Convection

这里：

-  :math:`\vec{v}_\omega` 是域内涡量诱导的速度；
-  :math:`\vec{V}_\omega(\vec{r},\vec{r}_m,\vec{\omega})` 是基于拉格朗日标记点的位置和涡旋元素的强度，计算给定点 :math:`\vec{r}` 处诱导速度的函数；
-  下标 :math:`e` 表示该量应用于元素；
-  涡量 :math:`\vec{\omega}` 通过离散卷积从涡旋元素的涡量中恢复。

对于涡旋段仿真，采用粘性分裂算法，对流步骤（公式 :eq:`eq:Convection`）是主要求解的状态方程。涡量拉伸自动考虑在内，扩散则在 *事后* 进行。速度函数 :math:`\vec{V}_\omega` 使用 Biot-Savart 定律。输出方程为：

.. math::
   \begin{aligned}
      \vec{y}_1&=\vec{v}_{i,ll} = \vec{V}_\omega ( \vec{r}_{ll}, \vec{r}_m, \vec{\omega}) \\
      \vec{y}_2&=\vec{r}_{r}
   \end{aligned}

与 AeroDyn 的集成
------------------

涡旋代码已集成为 OpenFAST 气动模块 *AeroDyn* 的子模块。OpenFAST 不同模块和子模块之间的数据流如 :numref:`AeroDyn-OLAF` 所示。使用涡旋代码时，AeroDyn 的输入如 BEM 选项（例如叶尖损失因子）、偏斜模型和动态入流将被忽略。环境条件、塔筒阴影和动态失速模型选项仍会被使用。这种集成需要对 *AeroDyn* 模块进行重构，以分离与塔筒阴影建模、诱导计算、升力线力计算和动态失速相关的代码部分。当与涡旋代码结合使用时，动态失速模型会进行调整，以确保脱落涡量的影响不会被重复计算。*AeroDyn* 与入流模块 *InflowWind* 之间的接口已进行调整，以包含涡旋代码额外请求的点。


..   _AeroDyn-OLAF:

.. figure:: Schematics/VortexCodeWorkFlow.png
   :alt: OpenFAST-FVW 代码集成工作流
   :width: 100%
   :align: center

   OpenFAST-OLAF 代码集成工作流



.. [1]
   升力线上的载荷不是涡旋代码的输出；其计算由 *AeroDyn* 的单独子模块处理。
