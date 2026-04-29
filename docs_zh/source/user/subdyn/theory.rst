
.. _subdyn-theory:

SubDyn理论
===========


概述
----

本节重点介绍SubDyn模块背后的理论。

SubDyn依赖于两种主要的工程方法：(1) 线性框架有限元模型(LFEM)，以及(2) 通过Craig-Bampton(C-B)方法结合静态改进方法(SIM)的动力学系统降阶，大大减少了获得精确解所需的模态数量。

海上风电子结构模型中存在许多非线性，包括材料非线性、弯曲引起的轴向缩短、大位移等。这里不考虑材料非线性，因为大多数海上多构件支撑结构设计使用钢材，且最大应力旨在低于材料的屈服强度。:cite:`damiani2013omae` 证明，在分析风机子结构时，线性有限元方法是适用的。在这项工作中，使用非线性和线性模型分析了几种不同基础几何形状、载荷路径、尺寸、支撑塔筒和风机质量的风机配置在极端载荷下的响应。结果表明，非线性行为主要由单塔筒响应引起，对多构件支撑结构影响很小。因此，子结构的LFEM模型被认为适用于风机子结构。LFEM可以容纳不同的单元类型，包括等截面或纵向变截面的欧拉-伯努利和铁木辛柯梁单元（铁木辛柯梁单元考虑剪切变形，更适合表示框架中使用的低长宽比梁，并在框架内传递载荷）。

典型多构件结构的标准有限元分析相关的大量自由度（约:math:`{10^3}`个）会阻碍风机系统动态仿真期间的计算效率。因此，采用了C-B系统降阶方法来加快处理速度，同时在整体系统响应中保持高保真性。C-B降阶用于将子结构有限元模型重新表征为降阶自由度模型，保留结构的基本低频响应模态。在SubDyn初始化步骤中，大型子结构物理自由度（位移）被降阶为少量模态自由度和界面（边界）自由度，在每个时间步中，只需要求解这些自由度的运动方程。SubDyn只求解模态自由度的运动方程，界面（边界）自由度的运动在SubDyn独立模式下运行时是指定的，或者在SubDyn与FAST耦合时通过ElastoDyn中的运动方程求解。

然而，仅保留少数自由度可能会导致轴向模态（通常频率非常高）被排除在外，而轴向模态对于捕捉静态载荷效应非常重要，例如重力和浮力引起的效应。实施了所谓的SIM方法来缓解这个问题。SIM在每个时间步计算两个静态解：一个基于全系统刚度矩阵，另一个基于降阶刚度矩阵。在每个时间步，基于C-B的时变动态解叠加在两个静态解的差值上，这相当于准静态地考虑了那些没有直接包含在动态解中的模态的贡献。

在SubDyn中，子结构被认为在底部节点（通常在海床处）被夹紧或通过线性弹簧类元件连接，并在子结构顶部节点（界面节点）与TP刚性连接。用户可以为每个底部节点提供6×6等效刚度和质量矩阵，以考虑土-桩相互作用。如本文档其他章节所述，输入文件定义了子结构几何形状、材料属性和约束。用户可以定义：单元类型；全有限元模式或C-B降阶模式；C-B降阶中要保留的模态数量；模态阻尼系数；是否利用SIM；以及每个构件的单元数量。

以下章节讨论SubDyn在FAST框架内的集成、模块中使用的主要坐标系，以及与LFEM、C-B降阶和SIM相关的理论。还介绍了用于时域仿真的状态空间公式。最后一节讨论了基础反力的计算。更多细节请参见:cite:`song2013`。

与FAST模块化框架的集成
--------------------------------------------------

基于新的模块化框架:cite:`jjonkman2013`，FAST将空气动力学模块、水动力学模块、控制和电气系统（伺服）模块以及结构动力学（弹性）模块结合在一起，实现了陆基和海上风机的耦合非线性气动-水动-伺服-弹性时域分析。:numref:`flow-chart2`展示了SubDyn模块在FAST模块化框架内的基本布局。

.. _flow-chart2:

.. figure:: figs/flowchart2.png
   :width: 70%

   SubDyn在模块化框架内的布局


在现有的松耦合时间积分方案中，耦合代码在每个时间步传递数据。这些数据包括SubDyn、HydroDyn和ElastoDyn之间的水动力载荷、子结构响应、传递到TP的载荷以及TP响应。在界面节点，TP的位移、旋转、速度和加速度是从ElastoDyn输入到SubDyn的，而TP处的反作用力是SubDyn的输出，供ElastoDyn输入。SubDyn还输出子结构的位移、速度和加速度，供HydroDyn输入，以计算成为SubDyn输入的水动力载荷。此外，SubDyn可以根据用户请求计算构件内力。在这个方案中，SubDyn跟踪其状态并通过自己的求解器集成其方程。

在紧耦合时间积分方案（尚未实现）中，SubDyn设置自己的方程，但它的状态和其他模块的状态由耦合代码中的求解器跟踪和集成，该求解器对所有模块是通用的。

SubDyn以状态空间公式实现，该公式形成子结构系统的运动方程，边界处为物理自由度，内部运动由模态自由度表示。在每个时间步，载荷和运动通过驱动代码在模块之间交换；模态响应在SubDyn的状态空间模型内部计算；下一个时间步的响应由SubDyn积分器计算（用于松耦合）或全局系统积分器计算（用于紧耦合）。

坐标系
------------------

.. _global-cs:

.. figure:: figs/global-cs.png
   :width: 40%

   全局（与子结构重合）坐标系。同时显示了与TP参考点相关的自由度。

全局和子结构坐标系：(*X*, *Y*, *Z*) 或 (:math:`{X_{SS}, Y_{SS}, Z_{SS}}`) (:numref:`global-cs`)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- 全局轴由单位向量 :math:`{\hat{I}, \hat{J}}` 和 :math:`{\hat{K}}` 表示。

- 原点设置在未偏转塔筒中心线与平均海平面(MSL)（海上系统）或地面（陆基系统）确定的水平面的交点。

- 正*Z* (:math:`{Z_{SS}}`) 轴是垂直向上的，与重力方向相反。

- 正*X* (:math:`{X_{SS}}`) 轴沿标称（零度）风向和波浪传播方向。

- *Y* (:math:`{Y_{SS}}`) 轴是横向的，假设为右手笛卡尔坐标系（沿标称下风方向看时指向左侧）。

构件或单元局部坐标系 (:math:`{x_e, y_e, z_e}`) (:numref:`element-cs`)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- 轴由单位向量 :math:`{\hat{i}_e, \hat{j}_e, \hat{k}_e}` 表示。

- 原点设置在起始节点(S, MJointID1)处截面的剪切中心。

- 局部 :math:`z_e` 轴沿构件的弹性轴，从起始节点(S)指向结束节点(E, MJointID2)。节点沿构件主轴排序，从起始节点到结束节点（根据用户输入定义）。

- 局部 :math:`x_e` 轴平行于全局 :math:`XY` 平面，其方向使得绕该轴的正旋转（小于等于180度）将使局部 :math:`z_e` 轴平行于全局 *Z* 轴。

- 局部 :math:`y_e` 轴可通过右手笛卡尔坐标系确定。

.. _element-cs:

.. figure:: figs/element-cs.png
   :width: 100%

   单元坐标系。图中所示的构件包含四个单元，第二个单元的节点S和E被标出。



局部到全局的转换
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

从局部坐标系到全局坐标系的转换可以用以下方程表示：

   .. math::  :label: loc2glb

      \begin{bmatrix} \Delta X \\ \Delta Y \\ \Delta Z \end{bmatrix}  =  [ \mathbf{D_c} ]  \begin{bmatrix} \Delta x_e \\ \Delta y_e \\ \Delta z_e \end{bmatrix}

其中 :math:`\begin{bmatrix} \Delta x_e \\ \Delta y_e \\ \Delta z_e \end{bmatrix}` 是局部坐标系中的任意向量，:math:`\begin{bmatrix} \Delta X \\ \Delta Y \\ \Delta Z \end{bmatrix}` 是同一向量在全局坐标系中的表示；:math:`[ \mathbf{D_c} ]` 是构件轴的方向余弦矩阵，可以通过以下方式获得：

   .. math::  :label: Dc

      [ \mathbf{D_c} ] = \begin{bmatrix}
                         \frac{Y_E-Y_S}{L_{exy}}  & \frac{ \left ( X_E-X_S \right ) \left ( Z_E-Z_S \right )}{L_{exy} L_e} & \frac{X_E-X_S}{L_e} \\
                         \frac{-X_E+X_S}{L_{exy}} & \frac{ \left ( Y_E-Y_S \right ) \left ( Z_E-Z_S \right )}{L_{exy} L_e} & \frac{Y_E-Y_S}{L_e} \\
                         0                        & \frac{ -L_{exy} }{L_e} & \frac{Z_E-Z_S}{L_e}
                         \end{bmatrix}

其中 :math:`{ \left ( X_S,Y_S,Z_S \right ) }` 和 :math:`{ \left ( X_E,Y_E,Z_E \right ) }` 是构件（或感兴趣单元的节点）在全局坐标系中的起始和结束节点；:math:`{L_{exy}= \sqrt{ \left ( X_E-X_S \right )^2 + \left ( Y_E-Y_S \right )^2} }`，:math:`{L_e= \sqrt{ \left ( X_E-X_S \right )^2 + \left ( Y_E-Y_S \right )^2 + \left ( Z_E-Z_S \right )^2} }`。

如果 :math:`{X_E = X_S}` 且 :math:`{Z_E = Z_S}`，则 :math:`{[ \mathbf{D_c} ]}` 矩阵可以通过以下方式获得：

如果 :math:`{Z_E >= Z_S}` 则：

 .. math::  :label: Dc_spec1

      [ \mathbf{D_c} ] = \begin{bmatrix}
                         1  & 0 & 0 \\
                         0  & 1 & 0 \\
                         0  & 0 & 1
                         \end{bmatrix}

否则：

 .. math::  :label: Dc_spec2

      [ \mathbf{D_c} ] = \begin{bmatrix}
                         1  & 0  & 0 \\
                         0  & -1 & 0 \\
                         0  & 0  & -1
                         \end{bmatrix}

在当前的SubDyn版本中，每个构件的这些方向余弦矩阵的转置（全局到局部）会在摘要文件中返回。考虑到构件截面的圆形形状，方向余弦矩阵对构件载荷验证的重要性不大。然而，要根据标准验证节点（例如:cite:`iso19902` :cite:`api2014`），弯矩需要分解为平面内和平面外分量，其中平面由一对支撑（对于X型节点）或一对支撑加支腿（对于K型节点）定义。因此，随时可以获得感兴趣构件的方向余弦，以便正确操作和转换局部剪力和弯矩，这一点很重要。

当未来版本允许圆形以外的构件截面时，用户将需要输入余弦矩阵，以指示构件主轴相对于全局参考系的最终方向。

.. _SD_FEM:

有限元模型 - 单元和约束
-----------------------------------------------

定义
~~~~~~~~~~~

图:numref:`fig:ElementsDefinitions` 用于说明所使用的一些定义。子结构的模型假设由不同的构件组成。

构件由两个节点界定。

节点由未变形结构上某点的坐标和类型(*JointType*)定义。节点类型定义了连接到该节点的所有构件的边界条件或约束。
支持以下节点类型：

- 悬臂节点 (*JointType=1*)

- 万向节 (*JointType=2*)

- 销节点 (*JointType=3*)

- 球节点 (*JointType=4*)

构件是以下四种类型之一：

- 梁 (*MType=1*)，欧拉-伯努利 (*FEMMod=1*) 或铁木辛柯 (*FEMMod=3*)

- 预紧缆索 (*MType=2*)

- 刚性连杆 (*MType=3*)

- 弹簧单元 (*MType=5*)

梁构件可以拆分为多个单元以提高模型精度（使用输入参数*NDiv*）。其他类型的构件（刚性连杆、预紧缆索和弹簧）不拆分。在本文档中，术语"单元"指的是：梁构件的子部分，或非梁类型的构件（刚性连杆、预紧缆索或弹簧）。术语"节点"指的是定义构件端点的点。有些节点是在输入文件中定义的，而其他节点则是梁构件细分产生的。单元的端点称为节点，对于所实现的单元，每个节点包含6个自由度。在当前实现中，假设节点与单元节点之间或连接单元的节点之间没有几何偏移。

.. figure:: figs/ElementsDefinitions.png
   :alt: 定义
   :name: fig:ElementsDefinitions
   :width: 80.0%

   构件、单元、节点、结点和刚性组件的定义。

.. _SD_FEM_Process:

有限元流程 - 从单元到系统矩阵
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

获得系统有限元表示的过程（在初始化时执行）如下：

- 单元：计算每个单元的质量和刚度矩阵，并使用方向余弦矩阵转换为全局坐标。

- 组装：将单元矩阵插入到全系统矩阵中。悬臂节点的自由度相互映射。对于由非悬臂节点连接的节点，其平动自由度相互映射，但每个节点的转动自由度在此系统中保留。这个全系统的自由度向量记为 :math:`\boldsymbol{x}`。

- 约束消除：使用直接消除技术来应用节点和刚性连杆引入的约束。消除过程包括形成矩阵 :math:`\boldsymbol{T}` 和一组降阶自由度 :math:`\boldsymbol{\tilde{x}}`，使得 :math:`\boldsymbol{x}=\boldsymbol{T} \boldsymbol{\tilde{x}}`。

- C-B降阶：使用Craig-Bampton降阶技术获得一组降阶自由度（界面自由度和Craig-Bampton模态）。

- 边界条件：然后应用位移边界条件（例如，对于固定基础）。

本节的其余部分重点介绍单元矩阵，以及节点和刚性连杆引入的约束说明。Craig-Bampton降阶在:numref:`GenericCBReduction`中描述。


自重载荷
~~~~~~~~~~~

自重引起的载荷在初始化时根据未变形配置预先计算。因此假设位移很小，子结构的P-Δ效应很小。

可以使用**GuyanLoadCorrection**标志来考虑"额外弯矩"，请参见:numref:`SD_ExtraMoment`节。

对于非变截面梁单元，重力引起的、施加在端节点的集中载荷如下（全局坐标系下）：

.. math::  :label: FG

	\left \{ F_G \right \} = \rho A_z g
                       \begin{bmatrix} 0 \\
                       0 \\
                       -\frac{L_e}{2} \\
		       -\frac{L_e^2}{12} D_{c,2,3} \\
		        \frac{L_e^2}{12} D_{c,1,3} \\
		       0\\
		       0\\
		       0\\
                       -\frac{L_e}{2}\\
		        \frac{L_e^2}{12} D_{c,2,3}\\
		       -\frac{L_e^2}{12} D_{c,1,3}\\
		       0
		     \end{bmatrix}

还需要注意的是，如果存在集中质量（用户在指定节点处选择），它们的贡献将作为沿全局*Z*方向的集中力包含在相关节点处。


梁单元公式
~~~~~~~~~~~~~~~~~~~~~~~~

等截面和变截面欧拉-伯努利梁单元是基于位移的，使用三阶插值函数，保证单元之间位移和旋转的连续性。等截面铁木辛柯梁单元是通过在等截面欧拉-伯努利单元中引入剪切变形导出的，因此位移也由三阶插值函数表示。

根据经典的铁木辛柯梁理论，通用两节点单元的刚度和一致质量矩阵可以写成如下形式（例如参见:cite:`panzer2009`）：


.. raw:: latex

    \renewcommand*{\arraystretch}{1.0}


.. math::  :label: ke0

    {\scriptstyle
    [k_e]=
            \begin{bmatrix}
                         \frac{12 E J_y}{L_e^3 \left (  1 + K_{sx} \right )} & 0                                                & 0                 & 0                                                                      & \frac{6 E J_y}{L_e^2 \left (  1 + K_{sx} \right )}                        & 0                 & -\frac{12 E J_y}{L_e^3 \left ( 1 + K_{sx} \right )}  & 0                                                   & 0                  & 0                                                                      & \frac{6 E J_y}{L_e^2 \left ( 1 + K_{sx} \right )}                        & 0 \\
                                                                           & \frac{12 E J_x}{L_e^3 \left (  1 + K_{sy} \right )} & 0                 & -\frac{6 E J_x}{L_e^2 \left ( 1 + K_{sy} \right )}                     & 0                                                                      & 0                 & 0                                                  & -\frac{12 E J_x}{L_e^3 \left ( 1 + K_{sy} \right )} & 0                  & -\frac{6 E J_x}{L_e^2 \left ( 1 + K_{sy} \right )}                     & 0                                                                      & 0 \\
                                                                           &                                                  & \frac{E A_z}{L_e} & 0                                                                      & 0                                                                      & 0                 & 0                                                  & 0                                                   & -\frac{E A_z}{L_e} & 0                                                                      & 0                                                                      & 0 \\
                                                                           &                                                  &                   & \frac{\left(4 + K_{sy} \right) E J_x}{L_e \left ( 1 + K_{sy} \right )} & 0                                                                      & 0                 & 0                                                  & \frac{6 E J_x}{L_e^2 \left ( 1 + K_{sy} \right )}   & 0                  & \frac{\left( 2 - K_{sy} \right) E J_x}{L_e \left ( 1 + K_{sy} \right )}  & 0                                                                      & 0  \\
                                                                           &                                                  &                   &                                                                        & \frac{\left(4 + K_{sx} \right) E J_y}{L_e \left ( 1 + K_{sx} \right )} & 0                 & -\frac{6 E J_y}{L_e^2 \left ( 1 + K_{sx} \right )} & 0                                                   & 0                  & 0                                                                      & \frac{\left( 2 - K_{sx} \right) E J_y}{L_e \left ( 1 + K_{sx} \right )}  & 0  \\
                                                                           &                                                  &                   &                                                                        &                                                                        & \frac{G J_z}{L_e} & 0                                                  & 0                                                   & 0                  & 0                                                                      & 0                                                                      & -\frac{G J_z}{L_e} \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   & \frac{12 E J_y}{L_e^3 \left ( 1 + K_{sx} \right )}  & 0                                                   & 0                  & 0                                                                      & -\frac{6 E J_y}{L_e^2 \left ( 1 + K_{sx} \right )}                     & 0  \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   &                                                    & \frac{12 E J_x}{L_e^3 \left ( 1 + K_{sy} \right )}    & 0                  & \frac{6 E J_x}{L_e^2 \left ( 1 + K_{sy} \right )}                      & 0                                                                      & 0  \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   &                                                    &                                                     & \frac{E A_z}{L_e}  & 0                                                                      & 0                                                                      & 0 \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    & \frac{\left(4 + K_{sy} \right) E J_x}{L_e \left ( 1 + K_{sy} \right )} & 0                                                                      & 0   \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    &                                                                        & \frac{\left(4 + K_{sx} \right) E J_y}{L_e \left ( 1 + K_{sx} \right )} & 0   \\
                                                                           &                                                  &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    &                                                                        &                                                                        & \frac{G J_z}{L_e} \\
                 \end{bmatrix}
     }


.. math::  :label: me0

	[m_e]= \rho \\
            \left[\begin{array}{*{12}c}
                         \frac{13 A_z L_e}{35} + \frac{6 J_y}{5 L_e} & 0                                                  & 0                 & 0                                                                      & \frac{11 A_z L_e^2}{210} + \frac{J_y}{10}                                & 0                 & \frac{9 A_z L_e}{70} - \frac{6 J_y}{5 L_e}           & 0                                                   & 0                  & 0                                                                      & -\frac{13 A_z L_e^2}{420} + \frac{J_y}{10}                               & 0 \\
                                                                           & \frac{13 A_z L_e}{35} + \frac{6 J_x}{5 L_e}          & 0                 & -\frac{11 A_z L_e^2}{210} - \frac{J_x}{10}                               & 0                                                                      & 0                 & 0                                                  & \frac{9 A_z L_e}{70} - \frac{6 J_x}{5 L_e}            & 0                  & \frac{13 A_z L_e^2}{420} - \frac{J_x}{10}                                & 0                                                                      & 0 \\
                                                                           &                                                    & \frac{A_z L_e}{3} & 0                                                                      & 0                                                                      & 0                 & 0                                                  & 0                                                   & \frac{A_z L_e}{6}  & 0                                                                      & 0                                                                      & 0 \\
                                                                           &                                                    &                   & \frac{A_z L_e^3}{105} + \frac{2 L_e J_x}{15}                             & 0                                                                      & 0                 & 0                                                  & -\frac{13 A_z L_e^2}{420} + \frac{J_x}{10}            & 0                  & -\frac{A_z L_e^3}{140} - \frac{L_e J_x}{30}                              & 0                                                                      & 0  \\
                                                                           &                                                    &                   &                                                                        & \frac{A_z L_e^3}{105} + \frac{2 L_e J_y}{15}                             & 0                 & \frac{13 A_z L_e^2}{420} - \frac{J_y}{10}            & 0                                                   & 0                  & 0                                                                      & -\frac{A_z L_e^3}{140} - \frac{L_e J_y}{30}                              & 0  \\
                                                                           &                                                    &                   &                                                                        &                                                                        & \frac{J_z L_e}{3} & 0                                                  & 0                                                   & 0                  & 0                                                                      & 0                                                                      & \frac{J_z L_e}{6} \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   & \frac{13 A_z L_e}{35} + \frac{6 J_y}{5 L_e}          & 0                                                   & 0                  & 0                                                                      & -\frac{11 A_z L_e^2}{210} - \frac{J_y}{10}                               & 0  \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   &                                                    & \frac{13 A_z L_e}{35} + \frac{6 J_x}{5 L_e}           & 0                  & \frac{11 A_z L_e^2}{210} + \frac{J_x}{10}                                & 0                                                                      & 0  \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   &                                                    &                                                     & \frac{A_z L_e}{3}  & 0                                                                      & 0                                                                      & 0 \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    & \frac{A_z L_e^3}{105} + \frac{2 L_e J_x}{15}                             & 0                                                                      & 0   \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    &                                                                        & \frac{A_z L_e^3}{105} + \frac{2 L_e J_y}{15}                             & 0   \\
                                                                           &                                                    &                   &                                                                        &                                                                        &                   &                                                    &                                                     &                    &                                                                        &                                                                        & \frac{J_z L_e}{3} \\
                 \end{array}\right]

其中矩阵是对称的，:math:`A_z` 是单元横截面积，:math:`J_x, J_y, J_z` 是截面主轴的面积二阶矩；:math:`L_e` 是未变形单元从起始节点到结束节点的长度；:math:`\rho, E,` 和 :math:`G` 分别是材料密度、杨氏模量和剪切模量；:math:`K_{sx}, K_{sy}` 是剪切修正系数，如下所示（如果选择欧拉-伯努利公式，则设置为零）：

.. math::  :label: Ksxy

	K_{sx}= \frac{12 E J_y}{G A_{sx} L_e^2}

	K_{sy}= \frac{12 E J_x}{G A_{sy} L_e^2}

其中沿局部*x*和*y*（主轴）轴的剪切面积定义为：

.. math::  :label: Asxy

	A_{sx}= k_{ax} A_z

	A_{sy}= k_{ay} A_z



其中：

.. math::  :label: kaxy

   k_{ax} = k_{ay} = \dfrac{ 6 (1 + \mu)^2 \left(1 + \left ( \frac{D_i}{D_o} \right )^2 \right)^2 }{ \left(1 + \left ( \frac{D_i}{D_o} \right )^2 \right)^2 (7 + 14 \mu + 8 \mu^2) + 4 \left ( \frac{D_i}{D_o} \right )^2 (5 + 10 \mu +4 \mu^2)}



方程:eq:`kaxy` 来自:cite:`steinboeck2013`，适用于空心圆截面，其中 :math:`\mu` 表示泊松比。

在组装全局系统刚度(*K*)和质量(*M*)矩阵之前，各个 :math:`{[k_e]}` 和 :math:`{[m_e]}` 通过 :math:`{[ \mathbf{D_c} ]}` 转换到全局坐标系，如下式所示：

.. math::  :label: ke1

	[k] =  \begin{bmatrix}
                         [\mathbf{D_c}] & 0 & 0 & 0 \\
                         & [\mathbf{D_c}] & 0 & 0  \\
                         & & [\mathbf{D_c}] & 0  \\
                         & & & [\mathbf{D_c}]
                 \end{bmatrix}  [k_e] \begin{bmatrix}
                         [\mathbf{D_c}] & 0 & 0 & 0 \\
                         & [\mathbf{D_c}] & 0 & 0  \\
                         & & [\mathbf{D_c}] & 0  \\
                         & & & [\mathbf{D_c}]
                 \end{bmatrix}^T

.. math::  :label: me1

	[m] =  \begin{bmatrix}
                         [\mathbf{D_c}] & 0 & 0 & 0 \\
                         & [\mathbf{D_c}] & 0 & 0  \\
                         & & [\mathbf{D_c}] & 0  \\
                         & & & [\mathbf{D_c}]
                 \end{bmatrix}  [m_e] \begin{bmatrix}
                         [\mathbf{D_c}] & 0 & 0 & 0 \\
                         & [\mathbf{D_c}] & 0 & 0  \\
                         & & [\mathbf{D_c}] & 0  \\
                         & & & [\mathbf{D_c}]
                 \end{bmatrix}^T

其中*m*和*k*是全局坐标系下的单元矩阵。


.. _SD_Pretension_Cable:

预紧缆索单元公式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


有限元的主刚度方程假设当所有位移都为零时力也为零，即力和位移之间的关系是线性的：:math:`\boldsymbol{f}=\boldsymbol{K}\boldsymbol{u}`。如果材料受到所谓的初始应变、初始应力或预应力，这个假设就不成立。这些效应可能由温度变化和预紧（或制造公差）产生。这些效应在Felippa的笔记中有讨论:cite:`felippa`。

预紧缆索可以通过假设桁架单元的初始伸长，并考虑这种初始伸长在纵向和正交方向上可能产生的恢复力来建模。


推导
^^^^^^^^^^

考虑沿:math:`z`方向取向的预紧缆索。为了简化推导，假设左端点固定，只有右端点发生偏转。符号说明如图:numref:`fig:FEPreTension`所示。

.. figure:: figs/FEPreTension.png
   :alt: 预紧
   :name: fig:FEPreTension
   :width: 50.0%

   预紧缆索方程推导使用的符号


预紧前单元的长度记为:math:`L_0`，其轴向刚度为:math:`k=EA/L_0`。在这个平衡位置，缆索的应力为零。该单元的用户输入选择为：未变形节点位置（预紧状态下）:math:`\boldsymbol{x}_1` 和 :math:`\boldsymbol{x}_2`、伸长刚度:math:`EA` 以及长度变化 :math:`\Delta L_0 = L_0 - L_e` (:math:`<0`)。预紧力 :math:`T_0` 是导出的输入量。定义如下：

.. math::

   \begin{aligned}
       L_e = \lVert \boldsymbol{x}_2 - \boldsymbol{x}_1 \rVert
       ,\quad
       \epsilon_0 = \frac{T_0}{EA}
       ,\quad
       L_0 = \frac{L_e}{1 + \epsilon_0}
   \end{aligned}

不同变量根据输入定义如下：

.. math::

   \begin{aligned}
       L_0 = L_e + \Delta L_0
       \qquad
       T_0 = - EA \frac{\Delta L_0}{L_0}
       ,\qquad
       \epsilon_0 = \frac{T_0}{EA} = \frac{-\Delta L_0}{L_0} = \frac{-\Delta L_0}{L_e + \Delta L_0}
   \end{aligned}

缆索偏转的自由度 :math:`(u_x, u_z)` 是从非平衡位置测量的，该位置与平衡位置有偏移，因此单元的预紧长度为 :math:`L_e > L_0`。当 :math:`u_z = 0` 时，缆索的应力为 :math:`\epsilon_0 = (L_e - L_0)/L_0`，或者说 :math:`L_e = L_0(1 + \epsilon_0)`。缆索的初始张力为 :math:`\boldsymbol{T}_0 = -k(L_e - L_0)\,\boldsymbol{e}_z = -EA\epsilon_0\,\boldsymbol{e}_z`。在偏转位置，缆索的长度为：

.. math::

   \begin{aligned}
      L_d = \sqrt{(L_e + u_z)^2 + u_x^2}
      = L_e \sqrt{1 + \frac{2 u_z}{L_e} + \frac{u_z^2}{L_e^2} + \frac{u_x^2}{L_e^2}}
      \approx L_e \left(1 + \frac{u_z}{L_e}\right)
      \label{eq:PreTensionLength}
   \end{aligned}

其中假设偏转与单元长度 :math:`L_e` 相比很小，即 :math:`u_x \ll L_e` 且 :math:`u_z \ll L_e`，并且只保留一阶项。偏转后缆索的张力为 :math:`\boldsymbol{T}_d = -k(L_d - L_0) \boldsymbol{e}_r`，其中径向向量是沿偏转缆索的向量，因此：

.. math::

   \begin{aligned}
       \boldsymbol{e}_r = \cos\theta \boldsymbol{e}_z + \sin\theta \boldsymbol{e}_x
           ,\quad
           \text{其中}
           \quad
              \cos\theta = \frac{L_e + u_z}{L_d}
               \approx 1
           ,\quad
             \sin\theta = \frac{u_x}{L_d}
               \approx \frac{u_x}{L_e}(1 - \frac{u_z}{L_e})
               \approx \frac{u_x}{L_e}
      \label{eq:PreTensionRadial}
   \end{aligned}

张力的分量为：

.. math::

   \begin{aligned}
    T_{d,z} &= -k(L_d - L_0)\cos\theta \approx -\frac{EA}{L_0}(L_e - L_0 + u_z)\, 1\,
        = -EA\epsilon_0 - \frac{EA}{L_0} u_z
        \nonumber
        \\
    T_{d,x} &= -k(L_d - L_0)\sin\theta \approx -\frac{EA}{L_0}(L_e - L_0 + u_z)\frac{u_x}{L_e}
        \approx -EA\epsilon_0 \frac{u_x}{L_e}
        = -\frac{EA\epsilon_0}{L_0(1 + \epsilon_0)}u_x
            \label{eq:PreTensionForce}
   \end{aligned}

预紧缆索的有限元公式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

为简洁起见，这里省略了转动自由度，因为缆索单元不考虑这些自由度。将线性公式应用于有限单元的两个节点，将每个节点处的力解释为单元对节点施加的内力。按照这个约定，预紧缆索单元可以用单元刚度矩阵 :math:`\boldsymbol{K}_e` 和附加节点载荷向量 :math:`\boldsymbol{f}_{e,0}` 表示，因此单元的静平衡方程可以写成 :math:`\boldsymbol{f}_e = \boldsymbol{K}_e \boldsymbol{u} + \boldsymbol{f}_{e,0}`，其中：

.. math:: :label: StiffnessMatrixCable

   \begin{aligned}
     \begin{bmatrix}
       f_{x,1}\\
       f_{y,1}\\
       f_{z,1}\\
       f_{x,2}\\
       f_{y,2}\\
       f_{z,2}\\
     \end{bmatrix}
       =
           \frac{EA}{L_0}
     \begin{bmatrix}
       \frac{\epsilon_0}{1 + \epsilon_0}  & 0                                & 0  & -\frac{\epsilon_0}{1 + \epsilon_0} & 0                                & 0 \\
       0                                & \frac{\epsilon_0}{1 + \epsilon_0}  & 0  & 0                                & -\frac{\epsilon_0}{1 + \epsilon_0} & 0 \\
       0                                & 0                                & 1  & 0                                & 0                                & -1\\
       -\frac{\epsilon_0}{1 + \epsilon_0} & 0                                & 0  & \frac{\epsilon_0}{1 + \epsilon_0}  & 0                                & 0 \\
       0                                & -\frac{\epsilon_0}{1 + \epsilon_0} & 0  & 0                                & \frac{\epsilon_0}{1 + \epsilon_0}  & 0 \\
       0                                & 0                                & -1 & 0                                & 0                                & 1 \\
     \end{bmatrix}
     \begin{bmatrix}
       u_{x,1}\\
       u_{y,1}\\
       u_{z,1}\\
       u_{x,2}\\
       u_{y,2}\\
       u_{z,2}\\
     \end{bmatrix}
       +
           EA\epsilon_0
     \begin{bmatrix}
       0\\
       0\\
       -1\\
       0\\
       0\\
       1\\
     \end{bmatrix}
       \end{aligned}

上述关系是在单元坐标系中表示的。在组装过程中，刚度矩阵和力向量会转换到全局坐标系。在上述方程中代入 :math:`\epsilon_0 = 0` 可以得到桁架单元的公式。上述线性模型仅在 :math:`L_d - L_0 > 0` 时有效，即 :math:`(L_e - L_0 + u_{z,2} - u_{z,1}) > 0`。如果在某个时间步不满足这个条件，实现应该中止。如果缆索的质量密度为正 :math:`\rho`，则单元的质量矩阵为：

.. math::

   \begin{aligned}
   \boldsymbol{M}_e = \rho L_e
   \left[
   \begin{array}{*{12}c}
   13/35 & 0       & 0   & &         & & 9/70  & 0       & 0   & &         & \\
   0     & 13/35   & 0   & & \boldsymbol{0}_3 & & 0     & 9/70    & 0   & & \boldsymbol{0}_3 & \\
   0     & 0       & 1/3 & &         & & 0     & 0       & 1/6 & &         & \\
         &         &     & &         & &       &         &     & &         & \\
         & \boldsymbol{0}_3 &     & & \boldsymbol{0}_3 & &       & \boldsymbol{0}_3 &     & & \boldsymbol{0}_3 & \\
         &         &     & &         & &       &         &     & &         & \\
   9/70  & 0       & 0   & &         & & 13/35 & 0       & 0   & &         & \\
   0     & 9/70    & 0   & & \boldsymbol{0}_3 & & 0     & 13/35   & 0   & & \boldsymbol{0}_3 & \\
   0     & 0       & 1/6 & &         & & 0     & 0       & 1/3 & &         & \\
         &         &     & &         & &       &         &     & &         & \\
         & \boldsymbol{0}_3 &     & & \boldsymbol{0}_3 & &       & \boldsymbol{0}_3 &     & & \boldsymbol{0}_3 & \\
         &         &     & &         & &       &         &     & &         & \\
   \end{array}
   \right]
   \label{eq:MassMatrixPreTension}
   \end{aligned}

其中 :math:`L_e` 是单元的未变形长度（不是 :math:`L_0`）。

.. _SD_Control_Cable:

可控预紧缆索
^^^^^^^^^^^^^^^^^^^^^^^^^^^

控制器在每个时间步改变缆索的静止长度，从而有效地改变缆索的预紧特性。在给定时间，缆索的静止长度为 :math:`L_r(t) = L_e + \Delta L`（而不是 :math:`L_0`），预紧力为 :math:`T(t)`（而不是 :math:`T_0`）。预紧力由下式给出：

.. math:: :label: tensionUnsteady

   \begin{aligned}
       T(t) = EA \frac{-\Delta L(t)}{L_r(t)} = EA \frac{-\Delta L(t)}{L_e + \Delta L(t)}
   \end{aligned}

在 :math:`t=0` 时，没有控制器作用，预紧力和长度为：

.. math:: :label: tensionZero

   \begin{aligned}
           T(0) = T_0 = EA \frac{-\Delta L_0}{L_e + \Delta L_0}
           ,\quad
           \Delta L(0) = \Delta L_0 = \frac{-L_e T_0}{EA + T_0}
   \end{aligned}


量 :math:`\Delta L` 是静止长度的变化，由下式给出：

.. math:: :label: DeltaLTot

   \begin{aligned}
       \Delta L(t) = \Delta L_0 + \Delta L_c(t)
   \end{aligned}

其中 :math:`\Delta L_c` 是控制器规定的长度变化，:math:`\Delta L_0` 是归因于初始预紧的长度变化。这样选择使得控制器输入名义上为零。SubDyn中不允许缆索伸长超过单元长度 :math:`L_e`，因此 :math:`\Delta L` 限制为负值（:math:`L_r = L_e + \Delta L \leq L_e`）。

将:eq:`DeltaLTot`代入:eq:`tensionUnsteady`可以得到给定时间的张力：

.. math::

   \begin{aligned}
       T(t) = -EA \frac{\Delta L_0 + \Delta L_c}{L_e + \Delta L_0 + \Delta L_c}
   \end{aligned}


下面我们提供实现细节和引入的近似。缆索单元的"运动方程"写为：

.. math::

   \begin{aligned}
       \boldsymbol{M}_e \boldsymbol{\ddot{u}}_e = \boldsymbol{f}_e
   \end{aligned}

如果预紧力是常数（等于:math:`T_0`），并且忽略附加外部载荷，则单元力为：

.. math::  :label: CstCableA

   \begin{aligned}
   \boldsymbol{f}_e = \boldsymbol{f}_e (t, T_0) = -\boldsymbol{K}_c(T_0) \boldsymbol{u}_e + \boldsymbol{f}_c(T_0) + \boldsymbol{f}_g
        \end{aligned}

其中 :math:`\boldsymbol{f}_c(T_0)` 和 :math:`\boldsymbol{K}_c(T_0)` 由:eq:`StiffnessMatrixCable`给出。如果预紧力随时间变化（:math:`T=T(t)`），则力为：

.. math::  :label: VaryingCableA

   \begin{aligned}
      \boldsymbol{f}_e (t) = -\boldsymbol{K}_c(T) \boldsymbol{u}_e + \boldsymbol{f}_c(T) + \boldsymbol{f}_g
   \end{aligned}

其中:eq:`VaryingCableA` 使用 :math:`\epsilon = \frac{T}{EA}` 和 :math:`L = \frac{L_e}{1 + \epsilon}` 计算。我们试图将:eq:`VaryingCableA` 表示为常数预紧缆索方程（即:eq:`CstCableA`，其中 :math:`T(0)=T_0`）的修正项。我们将 :math:`\pm\boldsymbol{f}_e(t, T_0)` 添加到方程中，得到：

.. math::

   \begin{aligned}
      \boldsymbol{f}_e (t) &= \left [ -\underbrace{\boldsymbol{K}_c(T_0) \boldsymbol{u}_e}_{\text{在CB中}} + \underbrace{\boldsymbol{f}_c(T_0) + \boldsymbol{f}_g}_{\text{在} F_G 中} \right ] + \boldsymbol{f}_{c,\text{control}}(T)
   \end{aligned}

其中 :math:`\boldsymbol{f}_{c,\text{control}}(T)` 是考虑 :math:`T` 随时间变化的修正项：

.. math::

   \begin{aligned}
   \boldsymbol{f}_{c,\text{control}}(T) &= \left( \boldsymbol{K}_c(T_0) - \boldsymbol{K}_c(T) \right) \boldsymbol{u}_e + \boldsymbol{f}_c(T) - \boldsymbol{f}_c(T_0)
   \end{aligned}

这个方程通过单元的方向余弦矩阵转换到全局系统。涉及 :math:`\boldsymbol{u}` 的部分引入了非线性，目前被忽略。因此，给定单元的附加控制力为：

.. math::

   \begin{aligned}
   \boldsymbol{f}_{c,\text{control}}(T) &\approx \boldsymbol{f}_c(T) - \boldsymbol{f}_c(T_0) = (T - T_0)
     \begin{bmatrix}
       0\\
       0\\
       -1\\
       0\\
       0\\
       1\\
     \end{bmatrix}
   \end{aligned}









旋转节点引入的约束
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如:numref:`SD_FEM_Process`所述，约束的处理通过直接消除技术实现。该技术通过计算变换矩阵 :math:`\boldsymbol{T}` 来实现，该矩阵给出了考虑约束的降阶自由度集合与全自由度集合之间的关系。当没有约束时，这个矩阵是单位矩阵。本节描述如何获得旋转节点的 :math:`\boldsymbol{T}` 矩阵。


**公式**
这里考虑两个节点 :math:`k` 和 :math:`l` 之间的节点。在考虑节点引入的约束之前，存在12个自由度：:math:`(\boldsymbol{u}_k, \boldsymbol{\theta}_k, \boldsymbol{u}_l, \boldsymbol{\theta}_l)`。应用约束后，新的自由度集合记为 :math:`(\boldsymbol{\tilde{u}}_{kl}, \boldsymbol{\tilde{\theta}}_{kl})`。下表显示了每种节点类型保留的自由度。不同的 :math:`\theta` 变量的含义将在后面的段落中说明。

.. table:: 不同节点类型的节点自由度(DOF)

   ============== =================================== ===================================== =================================== =====================================================================================
   **节点类型**      :math:`\boldsymbol{n}_\text{c}`     :math:`\boldsymbol{n}_\text{DOF}`     :math:`\boldsymbol{\tilde{u}}_{kl}` :math:`\boldsymbol{\tilde{\theta}}_{kl}`
   ============== =================================== ===================================== =================================== =====================================================================================
   悬臂             :math:`6`                           :math:`12 \to 6`                      :math:`u_x, u_y, u_z`                 :math:`\theta_x, \theta_y, \theta_k`
   销               :math:`5`                           :math:`12 \to 7`                      :math:`u_x, u_y, u_z`                 :math:`\theta_1, \theta_2, \theta_3, \theta_4`
   万向节           :math:`4`                           :math:`12 \to 8`                      :math:`u_x, u_y, u_z`                 :math:`\theta_1, \theta_2, \theta_3, \theta_4, \theta_5`
   球               :math:`3`                           :math:`12 \to 9`                      :math:`u_x, u_y, u_z`                 :math:`\theta_{x,k}, \theta_{y,k}, \theta_{z,k}, \theta_{x,l}, \theta_{y,l}, \theta_{z,l}`
   ============== =================================== ===================================== =================================== =====================================================================================

对于所有考虑的节点，两个节点的平动自由度是相等的，这可以形式化地表示为：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{u}_{k} \\
       \boldsymbol{u}_{l}
       \end{bmatrix}
       =
       \begin{bmatrix}
       \boldsymbol{I}_3 \\
       \boldsymbol{I}_3 \\
       \end{bmatrix}
       \boldsymbol{\tilde{u}}_{kl}
   \end{aligned}

由于这个关系对于所有节点都是相同的，因此自由度之间的关系在组装步骤中处理。
因此，每个节点的约束将以以下形式表示：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \boldsymbol{\theta}_{l}
       \end{bmatrix}
       = \boldsymbol{T}_{kl}
       \boldsymbol{\tilde{\theta}}_{kl}
       \label{eq:RotationalDOFJoint}
   \end{aligned}

**悬臂节点** 对于两个单元之间的悬臂节点，降阶方式为：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \boldsymbol{\theta}_{l}
       \end{bmatrix}
       =
       \boldsymbol{T}_{kl}
       \boldsymbol{\tilde{\theta}}_{kl}
       ,\qquad
           \text{其中}
       \quad
       \boldsymbol{\tilde{\theta}}_{kl}
       =
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \end{bmatrix}
       ,\qquad
       \boldsymbol{T}_{kl} =
       \begin{bmatrix}
       \boldsymbol{I}_3 \\
       \boldsymbol{I}_3 \\
       \end{bmatrix}
   \end{aligned}

这个关系在组装过程中直接处理，并且很容易扩展到:n个单元的情况。

**球/球形节点** 对于两个单元之间的球形节点，降阶方式如下：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \boldsymbol{\theta}_{l}
       \end{bmatrix}
       =
       \boldsymbol{T}_{kl}
       \boldsymbol{\tilde{\theta}}_{kl}
       ,\qquad
           \text{其中}
       \quad
       \boldsymbol{\tilde{\theta}}_{kl}
       =
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \boldsymbol{\theta}_{l} \\
       \end{bmatrix}
       ,\qquad
       \boldsymbol{T}_{kl} =
       \begin{bmatrix}
       \boldsymbol{I}_3 & \boldsymbol{0} \\
       \boldsymbol{0}   & \boldsymbol{I}_3 \\
       \end{bmatrix}
   \end{aligned}

对于由球节点（约束:c）连接的:n个单元:math:`[e_1,\cdots, e_n]`，关系扩展如下：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{e_1} \\
       \cdots\\
       \boldsymbol{\theta}_{e_n}
       \end{bmatrix}
       =
       \boldsymbol{T}^c
       \boldsymbol{\tilde{\theta}}^c
       ,\qquad
           \text{其中}
       \quad
       \boldsymbol{\tilde{\theta}}^c
       =
       \begin{bmatrix}
       \boldsymbol{\theta}_{e_1} \\
       \cdots\\
       \boldsymbol{\theta}_{e_n} \\
       \end{bmatrix}
       ,\qquad
       \boldsymbol{T}^c =
       \begin{bmatrix}
       \boldsymbol{I}_3 &        & \boldsymbol{0} \\
              & \ddots &                \\
       \boldsymbol{0}   &        & \boldsymbol{I}_3 \\
       \end{bmatrix}
       \label{eq:BallJointMulti}
   \end{aligned}

**销/转动副** 销节点的特征是存在一个方向，绕该方向没有力矩传递。指示该方向的单位向量记为 :math:`\boldsymbol{\hat{p}}`。然后定义两个正交向量 :math:`\boldsymbol{p}_1` 和 :math:`\boldsymbol{p}_2`，与 :math:`\hat{p}` 构成正交基，方向任意（参见:numref:`fig:FEJointPin`）。

.. figure:: figs/FEJointPin.png
   :alt: 销节点
   :name: fig:FEJointPin
   :width: 40.0%

   销节点约束推导使用的符号


然后定义变量 :math:`\tilde{\theta}_1` 到 :math:`\tilde{\theta}_4`：

.. math::

   \begin{aligned}
   \tilde{\theta}_1 &=
       \boldsymbol{p}_1^t \cdot \boldsymbol{\theta}_k
      =
       \boldsymbol{p}_1^t \cdot \boldsymbol{\theta}_l \\
   \tilde{\theta}_2 &=
       \boldsymbol{p}_2^t \cdot \boldsymbol{\theta}_k
      =
       \boldsymbol{p}_2^t \cdot \boldsymbol{\theta}_l \\
   \tilde{\theta}_3 &=
             \boldsymbol{\hat{p}}^t \cdot \boldsymbol{\theta}_k\\
   \tilde{\theta}_4 &=
             \boldsymbol{\hat{p}}^t \cdot \boldsymbol{\theta}_l
   \end{aligned}

这可以写成矩阵形式：

.. math::

   \begin{aligned}
   \begin{bmatrix}
   \tilde{\theta}_1 \\
   \tilde{\theta}_2 \\
   \tilde{\theta}_3 \\
   \tilde{\theta}_4 \\
   \end{bmatrix}
   =
   \boldsymbol{A}
   \begin{bmatrix}
   \boldsymbol{\theta}_k \\
   \boldsymbol{\theta}_l \\
   \end{bmatrix}
   =
   \begin{bmatrix}
   \boldsymbol{p}_1^t/2 & \boldsymbol{p}_1^t/2 \\
   \boldsymbol{p}_2^t/2 & \boldsymbol{p}_2^t/2 \\
   \boldsymbol{\hat{p}}^t & \boldsymbol{0}   \\
   \boldsymbol{0} & \boldsymbol{\hat{p}}^t   \\
   \end{bmatrix}
   \begin{bmatrix}
   \boldsymbol{\theta}_k \\
   \boldsymbol{\theta}_l \\
   \end{bmatrix}
   \end{aligned}

使用伪逆来反转关系，伪逆定义为 :math:`\boldsymbol{A}^{-1^\ast} = \boldsymbol{A}^t (\boldsymbol{A} \boldsymbol{A}^t)^{-1}`。使用伪逆，这个方程可以重写为：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{k} \\
       \boldsymbol{\theta}_{l}
       \end{bmatrix}
       =
       \boldsymbol{T}_{kl}
       \boldsymbol{\tilde{\theta}}_{kl}
       ,\qquad
           \text{其中}
       \quad
       =
       \boldsymbol{\tilde{\theta}}_{kl}
       \begin{bmatrix}
           \tilde{\theta}_1 \\
           \tilde{\theta}_2 \\
           \tilde{\theta}_3 \\
           \tilde{\theta}_4 \\
       \end{bmatrix}
       ,\qquad
       \boldsymbol{T}_{kl} =
       \begin{bmatrix}
       \boldsymbol{p}_1^t/2 & \boldsymbol{p}_1^t/2 \\
       \boldsymbol{p}_2^t/2 & \boldsymbol{p}_2^t/2 \\
       \boldsymbol{\hat{p}}^t & \boldsymbol{0}   \\
       \boldsymbol{0} & \boldsymbol{\hat{p}}^t   \\
       \end{bmatrix}^{-1^\ast}
   \end{aligned}

如果有:n个单元:math:`[e_1,\cdots, e_n]` 通过销节点（约束:c）连接，关系扩展如下：

.. math::

   \begin{aligned}
       \begin{bmatrix}
       \boldsymbol{\theta}_{e_1} \\
       \cdots\\
       \boldsymbol{\theta}_{e_n}
       \end{bmatrix}
       =
       \boldsymbol{T}^c
       \boldsymbol{\tilde{\theta}}^c
       ,\qquad
           \text{其中}
       \quad
       \boldsymbol{\tilde{\theta}}^c
       =
       \begin{bmatrix}
           \tilde{\theta}_1 \\
           \tilde{\theta}_2 \\
           \tilde{\theta}_{e_1} \\
           \cdots \\
           \tilde{\theta}_{e_n} \\
       \end{bmatrix}
       ,\qquad
       \boldsymbol{T}^c =
       \begin{bmatrix}
       \boldsymbol{p}_1^t/n & \cdots & \boldsymbol{p}_1^t/n \\
       \boldsymbol{p}_2^t/n & \cdots & \boldsymbol{p}_2^t/n \\
       \boldsymbol{\hat{p}}^t &  & \boldsymbol{0}   \\
                & \ddots &    \\
       \boldsymbol{0}  &  & \boldsymbol{\hat{p}}^t   \\
       \end{bmatrix}^{-1^\ast}
       \label{eq:PinJointMulti}
   \end{aligned}

**万向节** 万向节传递绕两个不对齐轴的力矩。这种节点只连接两个单元，标记为:math:`j`和:math:`k`，轴取为每个单元的:math:`z`轴。轴向量以全局坐标系表示，记为:math:`\boldsymbol{\hat{z}}_j` 和 :math:`\boldsymbol{\hat{z}}_k`。类似的符号用于:math:`x`和:math:`y`轴。两个轴之间共享旋转对应的自由度记为:math:`\tilde{\theta}_1`。每个单元有两个额外的自由旋转自由度，记为:math:`\tilde{\theta}_x` 和 :math:`\tilde{\theta}_y`。原始自由度和降阶自由度之间的约束关系是通过将每个单元的旋转自由度投影到不同的轴上获得的。使用伪逆来反转关系，伪逆定义为:math:`\boldsymbol{A}^{-1^\ast} = \boldsymbol{A}^t (\boldsymbol{A} \boldsymbol{A}^t)^{-1}`。然后约束定义为：

.. math::

   \begin{aligned}
       \boldsymbol{\tilde{\theta}}_c
           =
       \begin{bmatrix}
           \tilde{\theta}_1 \\
           \tilde{\theta}_{x,j} \\
           \tilde{\theta}_{y,j} \\
           \tilde{\theta}_{x,k} \\
           \tilde{\theta}_{y,k} \\
       \end{bmatrix}
           ,\quad
       \boldsymbol{T}_c =
       \begin{bmatrix}
       \boldsymbol{\hat{z}}_j/2 & \boldsymbol{\hat{z}}_k/2 \\
       \boldsymbol{\hat{x}}_j & 0             \\
       \boldsymbol{\hat{y}}_j & 0             \\
       0             & \boldsymbol{\hat{x}}_k \\
       0             & \boldsymbol{\hat{y}}_k \\
       \end{bmatrix}^{-1^\ast}
   \end{aligned}

.. math::

   \begin{aligned}
       \tilde{\theta}_c
           =
       \begin{Bmatrix}
           \tilde{\theta}_1 \\
           \tilde{\theta}_{x,e_1} \\
           \tilde{\theta}_{y,e_1} \\
           \vdots\\
           \tilde{\theta}_{x,e_n} \\
           \tilde{\theta}_{y,e_n} \\
       \end{Bmatrix}
           ,\quad
       T_c =
       \begin{bmatrix}
       \hat{z}_{e_1}^t/2 & \cdots & \hat{z}_{e_n}^t/n \\
       \hat{x}_{e_1}^t &        & 0                     \\
       \hat{y}_{e_1}^t &        & 0                     \\
       0                     & \ddots & 0                     \\
       0                     &        & \hat{x}_{e_n}^t   \\
       0                     & \cdots & \hat{y}_{e_n}^t   \\
       \end{bmatrix}^{-1^\ast}
   \end{aligned}






.. _SD_RigidLinks:


刚性连杆
~~~~~~~~~~~

刚性连杆和刚性单元在多个自由度之间施加关系，因此可以视为线性多点约束。刚性构件可用于将不同的单元连接在一起，或对两个弹性体之间的大刚度连杆建模（参见Cook:cite:`cook`）。刚性连杆的质量属性可以在输入文件中提供，在这种情况下，梁单元的质量矩阵用于该刚性连杆。

考虑节点:math:`j`和:math:`k`之间的刚性连杆，称为单元:math:`j-k`。给定节点的六个自由度（三个平动和三个转动）在全局系统中记为:math:`\boldsymbol{x} = [u_x, u_y, u_z, \theta_x, \theta_y, \theta_z]^t`。节点:math:`j`和:math:`k`刚性连接的事实可以形式化表示为：

.. math:: :label: RigidLinkElem

   \begin{aligned}
      \boldsymbol{x}_k = \boldsymbol{A}_{jk} \boldsymbol{x}_j
     ,\qquad
      \boldsymbol{A}_{jk} =
      \begin{bmatrix}
       1 & 0 & 0 & 0                     & \phantom{-} (z_k - z_j) & -(y_k - y_j)            \\
       0 & 1 & 0 & -(z_k - z_j)            & 0                     & \phantom{-} (x_k - x_j) \\
       0 & 0 & 1 & \phantom{-} (y_k - y_j) & -(x_k - x_j)            & 0                     \\
       0 & 0 & 0 & 1                     & 0                     & 0                     \\
       0 & 0 & 0 & 0                     & 1                     & 0                     \\
       0 & 0 & 0 & 0                     & 0                     & 1                     \\
      \end{bmatrix}
   ,\qquad
      \begin{bmatrix}
      \boldsymbol{x}_j\\
      \boldsymbol{x}_k'\\
      \end{bmatrix}
      =
      \boldsymbol{T}
      \boldsymbol{x}_j
      =
      \begin{bmatrix}
       \boldsymbol{I}_6\\
       \boldsymbol{A}_{jk}'\\
      \end{bmatrix}
      \boldsymbol{x}_j
          \end{aligned}

其中节点坐标:math:`(x, y, z)`以全局系统表示。矩阵:math:`\boldsymbol{T}`表示压缩坐标和原始坐标之间的关系。

在一般情况下，多个节点可以通过刚性连杆耦合在一起。这里假设:n个节点组成的组件，每个节点的6个自由度记为:math:`\boldsymbol{x}_1, \cdots, \boldsymbol{x}_n`。进一步假设选择第一个节点作为主导节点。对于每个节点:math:`j \in \{2, \cdots, n\}`，根据:eq:`RigidLinkElem`形成矩阵:math:`\boldsymbol{A}_{1j}`。这些矩阵使用每个节点对的全局坐标构建。对于给定的刚性组件（或约束:c），节点自由度和降阶主导自由度之间的关系为：

.. math::

   \begin{aligned}
      \boldsymbol{x}^c = \boldsymbol{T}^c \boldsymbol{\tilde{x}}^c
          \quad
          \text{其中}
          \quad
      \boldsymbol{x}^c =
           \begin{bmatrix}
           \boldsymbol{x}_1\\
           \boldsymbol{x}_2\\
           \cdots\\
           \boldsymbol{x}_n\\
           \end{bmatrix}
      ,\quad
       \boldsymbol{T}^c =
          \begin{bmatrix}
           \boldsymbol{I}_6\\
           \boldsymbol{A}_{12}\\
           \cdots\\
           \boldsymbol{A}_{1n}\\
           \end{bmatrix}
      ,\quad
       \boldsymbol{\tilde{x}}^c = \boldsymbol{x}_1
      \label{eq:RigidLinkGlobMulti}
   \end{aligned}

SubDyn会检测刚性连杆组件，并为该组件选择主导节点。如果其中一个节点是界面节点，则选择它作为主导节点。

以下限制适用：从动节点不能是边界节点。

约束在全系统组装完成后应用。



.. _SD_SpringElement:


弹簧单元
~~~~~~~~~~~~~~~

不要将弹簧构件与陆基系统中定义为边界条件的弹簧混淆。弹簧单元通过对称的6×6刚度矩阵（:math:`k_{ij} = k_{ji}`）连接两个节点。

.. math::

   \begin{aligned}
      K =
      \begin{bmatrix}
      k_{11} & k_{12} & k_{13} & k_{14} & k_{15} & k_{16} \\
      k_{21} & k_{22} & k_{23} & k_{24} & k_{25} & k_{26} \\
      k_{31} & k_{32} & k_{33} & k_{34} & k_{35} & k_{36} \\
      k_{41} & k_{42} & k_{43} & k_{44} & k_{45} & k_{46} \\
      k_{51} & k_{52} & k_{53} & k_{54} & k_{55} & k_{56} \\
      k_{61} & k_{62} & k_{63} & k_{64} & k_{65} & k_{66} \\
      \end{bmatrix}
   \end{aligned}

弹簧单元没有相关质量。但是，如果需要，可以在节点处定义集中质量。

由于每个节点有6个自由度（3个平动和3个转动），数学上弹簧单元的维度为12×12。

.. math::

   \begin{aligned}
      K_e =
      \begin{bmatrix}
      k_{6×6} & -k_{6×6} \\
      -k_{6×6} & k_{6×6}  \\
      \end{bmatrix}
   \end{aligned}

弹簧单元必须定义在两个重合的节点之间，并且必须通过方向余弦提供方向。这允许在全局系统刚度矩阵中组装弹簧单元。

弹簧单元可以连接到梁、运动副（例如转动副、万向节和球形副）、界面节点和刚性连杆。但是，它不能连接到缆索。

.. _GenericCBReduction:

Craig-Bampton降阶（理论）
--------------------------------

全系统
~~~~~~~~~~~


SubDyn的有限元运动方程写为：

.. math:: :label: main

         [M] \{ \ddot{U} \} + [C] \{ \dot{U} \} + [K] \{ U \} = \{ F \}


其中:math:`{[M]}`和:math:`{[K]}`是子结构梁框架的全局质量和刚度矩阵，由单元质量和刚度矩阵组装而成。此外，:math:`{[M]}`和:math:`{[K]}`包含任何指定的:math:`{[M_{SSI}]}`和:math:`{[K_{SSI}]}`的贡献，这些矩阵直接添加到部分约束节点自由度行和列索引的对应元素中。

:math:`{{U}}`和:math:`{{F}}`是组装系统所有自由度的位移和外力。阻尼矩阵:math:`{[C]}`不是由单元贡献组装的，因为这些通常是未知的，但可以通过不同方式指定，如:numref:`SD_DampingSpecifications`所述。对时间的导数用点表示，因此:math:`{\dot{U}}`和:math:`{\ddot{U}}`分别是:math:`{{U}}`的一阶和二阶时间导数。

对于典型的梁框架子结构，与方程:eq:`main`相关的自由度数量很容易增长到数千个。这个因素，加上风机动力学时域仿真的需要，可能会严重降低气动弹性代码（如FAST）的计算效率（注意，ElastoDyn中典型的风机系统模型约有20个自由度）。因此，使用C-B方法将子结构有限元模型重新表征为降阶自由度模型，保留结构的基本低频响应模态。通过C-B方法，子结构的自由度可以减少到约10个（用户定义，另见:numref:`CBguide`节）。这种系统降阶方法最初由:cite:`hurty1964`提出，后来由:cite:`craig1968`扩展。


CB降阶系统
~~~~~~~~~~~~~~~~~
本节介绍通用的Craig-Bampton技术。在后续章节中介绍在SubDyn中的具体应用。
在C-B降阶中，结构节点分为两组：(1) 边界节点（以下用下标:math:`{R}`标识），包括完全约束在结构底部的节点和界面节点；(2) 内部节点（或剩余节点，以下用下标:math:`{L}`标识）。注意，在包含SSI功能的这个版本的SubDyn中，结构底部部分约束或"自由"节点的自由度包含在:math:`{L}`子集中。

系统降阶的推导如下。方程:eq:`main`的系统运动方程可以分块为：

.. math:: :label: main2

        \begin{bmatrix}
        	M_{RR} & M_{RL} \\
             M_{LR} & M_{LL}
        \end{bmatrix}
        \begin{bmatrix}
        	\ddot{U}_R \\
            \ddot{U}_L
        \end{bmatrix} +
        \begin{bmatrix}
	          C_{RR} & C_{RL} \\
	          C_{LR} & C_{LL} \\
        \end{bmatrix}
         \begin{bmatrix}
        	\dot{U}_R \\
            \dot{U}_L
        \end{bmatrix} +
        \begin{bmatrix}
            K_{RR} & K_{RL} \\
            K_{LR} & K_{LL} \\
        \end{bmatrix}
        \begin{bmatrix}
        	U_R \\
            U_L \\
        \end{bmatrix} =
        \begin{bmatrix}
            F_R \\
            F_L \\
         \end{bmatrix}


其中下标*R*表示边界自由度（有*R*个自由度），下标*L*表示内部自由度（有*L*个自由度）。在方程:eq:`main2`中，施加的力包括外力（例如水动力和通过TP传递到子结构的力）、重力和预紧力，这些被认为是集中在每个节点的静态力。

C-B方法的基本假设是，内部节点位移的贡献可以通过内部广义自由度（:math:`q_L`）的子集:math:`q_m`（:math:`{q_m \leq L}`）来近似。物理自由度和广义自由度之间的关系可以写成：

.. math:: :label: CB1

        \begin{bmatrix}
        	U_R \\
            U_L
        \end{bmatrix} =
      \begin{bmatrix}
        	I & 0 \\
           \Phi_R & \Phi_L
        \end{bmatrix}
        \begin{bmatrix}
        	U_R \\
            q_L
        \end{bmatrix}

其中*I*是单位矩阵；:math:`{\Phi_R}`是:math:`{L \times R}`的Guyan模态矩阵，表示边界（界面节点自由度，因为约束节点自由度根据定义被锁定）做静态刚体运动时内部节点的物理位移。考虑齐次静态版本的:eq:`main2`，可以对第二行进行处理得到：

.. math:: :label: CB2

	[K_{LR}] {U_R} + [K_{LL}]{U_L} = {0}

重新排列并考虑可得：

.. math:: :label: PhiR

	\Phi_R = -K_{LL}^{-1} K_{LR}

为了简化，这里去掉了括号。如果结构是无约束的，矩阵:math:`{\Phi_R}`对应刚体模态，确保内部节点跟随界面自由度施加的刚体位移。这已经使用单梁单元的刚度矩阵进行了解析验证。
:math:`{\Phi_L}`（:math:`{L \times L}`矩阵）表示内部特征模态，即系统在边界（界面和底部节点）约束下的固有模态，可以通过求解特征值问题获得：

.. math:: :label: PhiL1

	K_{LL} \Phi_L = \omega^2 M_{LL} \Phi_L

方程:eq:`PhiL1`的特征值问题得到广义模态自由度:math:`q_m`的降阶基，选择为前*m*个按特征频率升序排列的特征向量。:math:`\Phi_L`是质量归一化的，因此：

.. math:: :label: PhiL2

	\Phi_L^T M_{LL} \Phi_L = I

通过将广义自由度的数量减少到*m*（:math:`{\leq L}`），:math:`{\Phi_m}`是:math:`{L \times m}`矩阵，表示截断的:math:`{\Phi_L}`集合（保留总共内部模态中的*m*个，因此有*m*列），:math:`{\Omega_m}`是对角:math:`{m \times m}`矩阵，包含相应的特征频率（即:math:`\Phi_m^T K_{LL} \Phi_m = \Omega_m^2`）。在SubDyn中，用户决定保留多少模态，包括可能的零模态或所有模态。保留零模态对应于Guyan（静态）降阶；保留所有模态对应于保留完整的有限元模型。

因此，C-B变换由坐标变换矩阵:math:`T_{\Phi_m}`表示：

.. math:: :label: CB3

        \begin{bmatrix}
        	U_R \\
            U_L \\
        \end{bmatrix} =
        T_{\Phi_m}
        \begin{bmatrix}
        	U_R \\
            q_m \\
        \end{bmatrix}
        ,\qquad
        T_{\Phi_m} =
        \begin{bmatrix}
              I     & 0 \\
             \Phi_R & \Phi_m
        \end{bmatrix}

通过方程:eq:`CB3`，内部自由度因此从物理自由度转换为模态自由度。将方程:eq:`main2`两边左边乘以:math:`T_{\Phi_m}^T`，右边乘以:math:`T_{\Phi_m}`，并利用方程:eq:`PhiL2`，方程:eq:`main2`可以重写为：

.. math:: :label: main3

        \begin{bmatrix}
        	M_{BB} & M_{Bm} \\
            M_{mB} & I
        \end{bmatrix}
        \begin{bmatrix}
        	\ddot{U}_R \\
            \ddot{q}_m
        \end{bmatrix} +
        \begin{bmatrix}
        	C_{BB} & C_{Bm} \\
            C_{mB} & C_{mm}
        \end{bmatrix}
         \begin{bmatrix}
        	\dot{U}_R \\
            \dot{q}_m
        \end{bmatrix} +
        \begin{bmatrix}
             K_{BB} & 0 \\
             0      & K_{mm}
        \end{bmatrix}
        \begin{bmatrix}
        	U_R \\
            q_m
        \end{bmatrix} =
        \begin{bmatrix}
            F_B \\
            F_m
        \end{bmatrix}


其中：

.. math:: :label: partitions
   :nowrap:

    \begin{align}
    M_{BB} &= M_{RR} + M_{RL} \Phi_R + \Phi_R^T M_{LR} + \Phi_R^T M_{LL} \Phi_R \\
    C_{BB} &= C_{RR} + C_{RL} \Phi_R + \Phi_R^T C_{LR} + \Phi_R^T C_{LL} \Phi_R \nonumber \\
    K_{BB} &= K_{RR} + K_{RL} \Phi_R \nonumber \\
    M_{mB} &= \Phi_m^T M_{LR} + \Phi_m^T M_{LL} \Phi_R \nonumber \\
    C_{mB} &= \Phi_m^T C_{LR} + \Phi_m^T C_{LL} \Phi_R \nonumber \\
    K_{mm} &= \Phi_m^T K_{LL} \Phi_m = \Omega_m^2 \nonumber \\
    C_{mm} &= \Phi_m^T C_{LL} \Phi_m \nonumber \\
    F_B &= F_R + \Phi_R^T F_L \nonumber \\
    F_m &= \Phi_m^T F_L \nonumber
    \end{align}

且 :math:`M_{Bm} = M_{mB}^T`，:math:`C_{Bm} = C_{mB}^T`。



SubDyn中的有限元公式
-----------------------------------------------


.. _TP2Interface:

边界节点：固定自由度和与TP的刚性连接
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本节介绍边界节点的处理：消除固定自由度，并通过与TP参考点的刚性连接来压缩界面自由度。

边界节点分为界面节点:math:`{\bar{U}_R}`和底部固定节点：

.. math:: :label: UR

	U_R = \begin{bmatrix}
		\bar{U}_R \\
		0
	      \end{bmatrix}


这里和下面的上划线表示应用固定底部边界条件后的矩阵/向量。

假设界面节点之间以及与TP参考点刚性连接，因此使用TP刚体自由度（TP参考点处的一个节点，6个自由度）代替界面自由度会更方便。界面自由度:math:`{\bar{U}_R}`和TP自由度之间的关系如下：

.. math:: :label: UTP

	\bar{U}_R = T_I U_{TP}

其中:math:`T_I`是:math:`{(6 N_{IN}) \times 6}`矩阵，:math:`N_{IN}`是界面节点的数量，:math:`{U_{TP}}`是刚性过渡段的6个自由度。矩阵:math:`T_I`可以写成如下形式：

.. math:: :label: TI

   T_I = \begin{bmatrix}
	1 & 0 & 0 & 0           & \Delta Z_1 & - \Delta Y_1 \\
	0 & 1 & 0 & -\Delta Z_1 & 0 & - \Delta X_1 \\
	0 & 0 & 1 & \Delta Y_1 &  - \Delta X_1 & 0 \\
	0 & 0 & 0 & 1 & 0 & 0 \\
	0 & 0 & 0 & 0 & 1 & 0  \\
	0 & 0 & 0 & 0 & 0 & 1   \\
	\vdots & \vdots & \vdots & \vdots &  \vdots & \vdots \\
	1 & 0 & 0 & 0           & \Delta Z_i & - \Delta Y_i \\
	0 & 1 & 0 & -\Delta Z_i & 0 & - \Delta X_i \\
	0 & 0 & 1 & \Delta Y_i &  - \Delta X_i & 0 \\
	0 & 0 & 0 & 1 & 0 & 0 \\
	0 & 0 & 0 & 0 & 1 & 0  \\
	0 & 0 & 0 & 0 & 0 & 1   \\
	\vdots & \vdots & \vdots & \vdots &  \vdots & \vdots \\
	\end{bmatrix}, \left( i = 1, 2, \cdots, N_{IN} \right)

其中：

.. math:: :label: DXYZ
   :nowrap:

    \begin{align}
        \Delta X_i &= X_{IN,i} - X_{TP} \nonumber \\
        \Delta Y_i &= Y_{IN,i} - Y_{TP}  \\
        \Delta Z_i &= Z_{IN,i} - Z_{TP} \nonumber
    \end{align}


其中:math:`{ \left( X_{IN,i}, Y_{IN,i}, Z_{IN,i} \right) }`是第:i个界面节点的坐标，:math:`{ \left( X_{TP}, Y_{TP}, Z_{TP} \right) }`是TP参考点在全局坐标系中的坐标。

应用边界约束（去除与海床约束节点自由度对应的行和列）后，用TP自由度表示的系统运动方程:eq:`main3`变为：

.. math:: :label: main4

        \begin{bmatrix}
        	\tilde{M}_{BB} & \tilde{M}_{Bm} \\
            \tilde{M}_{mB} & I
        \end{bmatrix}
        \begin{bmatrix}
        	\ddot{U}_{TP} \\
            \ddot{q}_m
        \end{bmatrix} +
        \begin{bmatrix}
	          \tilde{C}_{BB} & \tilde{C}_{Bm} \\
	          \tilde{C}_{mB} &   C_{mm}
        \end{bmatrix}
         \begin{bmatrix}
        	\dot{U}_{TP} \\
            \dot{q}_m
        \end{bmatrix} +
        \begin{bmatrix}
             \tilde{K}_{BB} & 0 \\
            0      &           K_{mm}
        \end{bmatrix}
        \begin{bmatrix}
        	U_{TP} \\
            q_m
        \end{bmatrix} =
        \begin{bmatrix}
             \tilde{F}_{TP} \\
             F_m
         \end{bmatrix}


其中：

.. math:: :label: tilde_partitions0
   :nowrap:

    \begin{align}
        \tilde{M}_{BB} &= T_I^T \bar{M}_{BB} T_I, \quad
        \tilde{C}_{BB}  = T_I^T \bar{C}_{BB} T_I, \quad
        \tilde{K}_{BB}  = T_I^T \bar{K}_{BB} T_I \\
        \tilde{M}_{Bm} &= T_I^T \bar{M}_{Bm}, \quad
        \tilde{C}_{Bm}  = T_I^T \bar{C}_{Bm} \nonumber \\
        \tilde{F}_{TP} &= T_I^T F_B \nonumber
    \end{align}

..
        \tilde{F}_{TP} &= F_{TP} + T_I^T \left[ \bar{F}_{HDR} + \bar{F}_{Rg} + \bar{\Phi}_{R}^T \left( F_{L,e} + F_{L,g} \right) \right] \nonumber \\
        \tilde{C}_{mm} &= C_{mm}, \quad
        \tilde{K}_{mm} = K_{mm} = \Omega_m^2 \nonumber \\
        \tilde{F}_{m}  &= \Phi_m^T \left( F_{L,e} + F_{L,g} \right)  \nonumber

且 :math:`\tilde{M}_{mB} = \tilde{M}_{Bm}^T`，:math:`\tilde{C}_{mB} = \tilde{C}_{Bm}^T`。

方程:eq:`main4`表示C-B降阶后子结构的运动方程。子结构的总自由度从（6×总节点数）减少到（6 + m）。

在初始化阶段，SubDyn计算：参数矩阵:math:`{\tilde{M}_{BB}, \tilde{M}_{mB}, \tilde{M}_{Bm}, \tilde{K}_{BB}, \Phi_m, \Phi_R, T_I}`；常数载荷数组；以及内部频率矩阵:math:`\Omega_m`。然后可以使用下一节讨论的状态空间公式获得每个时间步的子结构响应。


浮式或固定式基础结构
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SubDyn中根据结构是"固定式基础"还是"浮式"使用不同的公式。

如果没有反力节点，结构被认为是"浮式"的。

在其他情况下，结构被认为是"固定式基础"的。


.. _SD_Loads:

载荷
~~~~~

本节详细介绍作用在边界(*R*)和内部(*L*)节点以及过渡段(*TP*)节点的载荷。

SubDyn考虑的外部载荷，如重力载荷或预紧载荷，以下标:math:`g`表示。作用在子结构上的来自附加模块的外部载荷，例如水动力、系泊或土壤载荷，以下标:math:`e`表示。ElastoDyn传递给SubDyn的耦合载荷以下标:math:`cpl`表示。在模块化实现中，SubDyn不从ElastoDyn接收这些耦合载荷，而是接收过渡段的位移，并输出相应的载荷。这与状态空间公式相关，但就本节而言，可以认为耦合载荷来自ElastoDyn。

边界节点(*R*)处的外部载荷由SubDyn重力和缆索载荷(:math:`g`)、ElastoDyn耦合载荷(:math:`cpl`)以及其他模块的外部载荷(:math:`e`)组成：

.. math:: :label: FR

	F_R = F_{R,e} + F_{R,g} + F_{R,\text{cpl}}

内部节点处的外部载荷类似地分解为：

.. math:: :label: FL

	F_L = F_{L,e} + F_{L,g}

过渡段节点(*TP*)处的载荷通过变换矩阵:math:`T_I`与界面边界节点(:math:`\bar{R}`)相关，该矩阵假设:math:`\bar{R}`和*TP*节点刚性连接：

.. math:: :label: FTP1

	F_{TP} = T_I^T \bar{F}_R

特别是，ElastoDyn和SubDyn之间交换的耦合力为：

.. math:: :label: FTP1cpl

	F_{TP,cpl} = T_I^T \bar{F}_{R,\textit{cpl}}


然后，方程:eq:`tilde_partitions0`中给出的Guyan TP力:math:`\tilde{F}_{TP}`和CB力:math:`F_m`分解为：

.. math:: :label: FTPtilde

       \tilde{F}_{TP} &= F_{TP,cpl} + T_I^T \left[ \bar{F}_{R,e} + \bar{F}_{R,g} + \bar{\Phi}_R^T \left( F_{L,e} + F_{L,g} \right) \right]

       F_m &= \Phi_m^t \left(F_{L,e}  +  F_{L,g}\right)




.. _SD_Rotated_Loads:
.. _SD_Extra_Moment:

基础公式的修正（"Guyan载荷修正"）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

基本有限元实现需要修正，以考虑载荷是在SubDyn的偏转位置施加的，以及浮式情况下的刚体运动。在以前版本的SubDyn中，通过将参数**GuyanLoadCorrection**设置为True来激活这些修正。这个输入参数已从SubDyn主输入文件中删除，并且在当前和未来版本的SubDyn中始终使用载荷修正。



**浮式情况的坐标系旋转**

在浮式情况下，有限元公式需要旋转到体坐标系。当**GuyanLoadCorrection**设置为True时会执行此操作。CB模式和静态模式在随Guyan模式刚体旋转的坐标系中求解。有关这种特殊情况的更多细节，请参见:numref:`SD_summary`节。


**外部载荷的附加力臂**

施加在子结构上的外部载荷是在结构偏转位置计算的。另一方面，有限元公式期望载荷相对于结构的未偏转位置提供，或者如果存在刚体运动，则相对于参考未偏转位置提供（参见图:numref:`sd_fig_extramoment`）。偏转节点处的节点力可以直接施加到参考节点位置，但这种映射会在参考节点位置引入力矩。

输入文件中的参数**GuyanLoadCorrection**用于考虑由于有限元载荷预期在参考位置而非偏转位置表达而产生的这种额外节点力矩。

当参数**GuyanLoadCorrection**设置为True时，节点力的映射如下进行。首先，定义结构的参考未偏转位置，根据结构是"固定"在海床还是不固定，有两种可能的配置。这两种配置如图:numref:`sd_fig_extramoment`所示。

.. _sd_fig_extramoment:

.. figure:: figs/extramoment.png
   :width: 90%

   由于结构偏转位置与有限元表示使用的参考位置之间的距离而产生的附加力矩示意图。为简单起见，假设载荷作用在Guyan位置而非真实偏转位置。

其次，假设外部载荷施加在"Guyan"偏转结构上，而不是完全偏转的结构上。为了避免输入载荷和Craig-Bampton状态之间的非线性依赖，忽略了Craig-Bampton位移。基于这个假设，将Guyan位置的外部载荷映射到参考位置。

对于所有外力，包括重力，都包含附加力矩。对于给定节点:math:`i \in [R, L]`，以及节点力:math:`f_i = f_{i,g} + f_{i,e}`，计算以下附加力矩：

.. math::

   \Delta m_i = \Delta u_i \times \left[ f_{i,g} + f_{i,e} \right]

其中向量:math:`\Delta u_i = \{ \Delta u_{ix}, \Delta u_{iy}, \Delta u_{iz} \}`，根据参考位置（固定或自由）以及节点是内部(*L*)还是边界(*R*)节点而有不同的定义：

.. math:: :label: eqextramom
   :nowrap:

    \begin{align}
          \text{（固定式基础）} \qquad
          \Delta u_{ij} = [\bar{\Phi}_R T_I]_{ij} U_{TP} \quad \text{对于} i \in L
          \
          &\text{，并且}
          \quad
          \Delta u_{ij} = [T_I]_{ij} U_{TP} \quad \text{对于} i \in \bar{R}
          \\
          \text{（自由/浮式）} \qquad
          \Delta u_{ij} = [\bar{\Phi}_R T_I]_{ij} U_{TP} - U_{TP} \quad \text{对于} i \in L
          \
          &\text{，并且}
          \quad
          \Delta u_{ij} = [T_I]_{ij} U_{TP} - U_{TP} \quad \text{对于} i \in \bar{R}
    \end{align}


其中:math:`j \in [x, y, z]`，:math:`[\bar{\Phi}_R T_I]_{ij}`中的下标:math:`ij`表示对应于节点i和平动自由度j的行。固定的边界自由度没有位移，因此没有额外力矩贡献。自由的边界自由度在实现中属于内部自由度:L的一部分。每个节点的重力和缆索力（在初始化时计算并存储在常数向量:math:`F_G`中）用于获得:math:`f_{i,g}`。注意，力矩的:math:`g`贡献:math:`\Delta m_i`不是常数，需要在每个时间步计算。

为了避免增加更多符号，当**GuyanLoadCorrection=True**时，本文中使用的所有载荷向量都隐含包含附加力矩。这适用于例如:math:`F_{R,e}, F_{L,e}, F_{R,g}, F_{L,g}`，其中隐含以下替换：

.. math::

    F_{R,e} =
    \begin{Bmatrix}
       \vdots\\
       f_{ix,e}\\
       f_{iy,e}\\
       f_{iz,e}\\
       m_{ix,e}\\
       m_{iy,e}\\
       m_{iz,e}\\
       \vdots\\
     \end{Bmatrix}
     \quad
    \longrightarrow
     \quad
    F_{R,e} =
    \begin{Bmatrix}
       \vdots\\
       f_{ix,e}\\
       f_{iy,e}\\
       f_{iz,e}\\
       m_{ix,e} + \Delta m_{ix,e}\\
       m_{iy,e} + \Delta m_{iy,e}\\
       m_{iz,e} + \Delta m_{iz,e}\\
       \vdots\\
     \end{Bmatrix}
     \
     \text{（GuyanLoadCorrection=True）}



载荷向量对:math:`U_{TP}`的依赖给状态空间表示带来了一些复杂性，例如:eq:`ABFx`中的:math:`B`和:math:`F_X`矩阵需要修改以考虑对:math:`U_{TP}`的依赖。即使:math:`F_{L,e}`和:math:`F_{L,g}`包含对:math:`U_{TP}`的依赖，方程仍然有效，但矩阵:math:`B`不应用于线性化（为简单起见，优先使用数值微分）。类似的考虑适用于方程:eq:`bigY2`。


方程:eq:`bigY1`中给出的耦合载荷:math:`F_{TP,cpl}`对应于TP参考位置的反力。在"自由边界条件"（浮式）情况下，不需要修正这个输出载荷，因为参考位置在偏转位置。对于"固定边界条件"情况，参考位置与偏转位置不对应，因此反作用力矩需要转移到偏转位置，如下所示：

.. math::

    F_{TP,cpl} =
   \begin{Bmatrix}
   f_{TP,cpl} \\
   m_{TP,cpl} \\
   \end{Bmatrix}
     \quad
    \longrightarrow
     \quad
    F_{TP,cpl}  =
   \begin{Bmatrix}
   f_{TP,cpl} \\
   m_{TP,cpl} - u_{TP} \times f_{TP,cpl} \\
   \end{Bmatrix}
     \
     \text{（GuyanLoadCorrection=True且固定边界条件）}

然后修改输出方程:math:`y_1 = -F_{TP,cpl}`以包含这个额外贡献。


.. _SD_DampingSpecifications:

阻尼规范
~~~~~~~~~~~~~~~~~~~~~~


有三种方式可以指定与SubDyn中界面节点运动相关的阻尼：无阻尼、瑞利阻尼或用户定义的6×6矩阵。

注意：与节点相关的阻尼尚未记录，并且会改变以下开发。

当**GuyanDampMod=0**时，SubDyn假设Guyan模态无阻尼，CB模态有模态阻尼，无交叉耦合：

.. math:: :label: dampingassumptions

            C_{BB} =  \tilde{C}_{BB} &= 0

        C_{Bm} = C_{mB} = \tilde{C}_{Bm} = \tilde{C}_{mB} &= 0

            C_{mm} = \tilde{C}_{mm} &= 2\zeta \Omega_m

换句话说，唯一保留的阻尼矩阵项是与内部自由度阻尼相关的项。这个假设对与风机系统界面处的阻尼有影响，如:ref:`TowerTurbineCpling`节所讨论。对角:math:`m \times m`矩阵:math:`\zeta`包含与每个保留内部模态对应的模态阻尼比。在SubDyn中，用户为保留模态提供阻尼比（以临界阻尼系数的百分比表示）。

当**GuyanDampMod=1**时，SubDyn假设Guyan模态为瑞利阻尼，CB模态为模态阻尼，无交叉耦合：


.. math:: :label: dampingRayleigh

        \tilde{C}_{BB} = \alpha \tilde{M}_{BB} + \beta \tilde{K}_{BB}

        \tilde{C}_{Bm} = \tilde{C}_{mB} &= 0

        \tilde{C}_{mm} &= 2\zeta \Omega_m

其中:math:`\alpha`和:math:`\beta`是质量和刚度比例瑞利阻尼系数。阻尼直接应用于波浪号矩阵，即与TP节点的6个自由度相关的矩阵。

当**GuyanDampMod=2**时，与前一种情况类似，只是用户指定:math:`\tilde{C}_{BB}`的6×6项。



.. _sim:
.. _SD_SIM:

静态改进法
~~~~~~~~~~~~~~~~~~~~~~~~~
为了考虑静态重力（构件自重）和浮力的影响，需要在C-B降阶中包含所有结构轴向模态。对于实际问题，这通常意味着需要保留数百个模态。因此，提出了一种替代方法来减少这个限制并加速SubDyn。这种方法称为SIM，它在每个时间步计算两个静态解：一个基于全系统刚度矩阵，另一个基于降阶刚度矩阵。然后动态解按前面章节所述进行，每个时间步将时变动态解叠加到两个静态解的差值上，这相当于准静态地考虑了动态解中未直接包含的那些模态的贡献。

SIM公式为内部节点的位移提供了修正。未修正的位移现在记为:math:`{\hat{U}_{L}}`，修正后的位移记为:math:`U_L`。SIM修正包括一个附加项:math:`U_{L,\text{SIM}}`，它是所有内部自由度的静态挠度(:math:`U_{L0}`)减去与CB模态相关的静态挠度(:math:`U_{L0m}`)，如:eq:`SIM`所示：

.. math:: :label: SIM

   U_L = \hat{U}_L + U_{L,\text{SIM}} = \hat{U}_L + \underbrace{U_{L0} - U_{L0m}}_{U_{L,\text{SIM}}} = \underbrace{\Phi_R U_R + \Phi_m q_m}_{\hat{U}_L} + \underbrace{\Phi_L q_{L0}}_{U_{L0}} - \underbrace{\Phi_m q_{m0}}_{U_{L0m}}


.. 其中:math:`U_{L0}`和:math:`U_{L0m}`的表达式将在下一段推导。将在接下来的段落中推导。方程:eq:`SIM`可以重写为：

..
            \begin{bmatrix}
                U_R \\
                    U_L
            \end{bmatrix} =
          \begin{bmatrix}
                I & 0 & 0 & 0 \\
               \Phi_R & \Phi_m & \Phi_L & -\Phi_m
            \end{bmatrix}
            \begin{bmatrix}
                U_R \\
                    q_m \\
                    q_{L0} \\
                    q_{m0}
            \end{bmatrix}
   其中：
        U_{L0} = \Phi_L q_{L0}, \qquad U_{L0m} = \Phi_m q_{m0}

其中:math:`{q_{m0}}`和:math:`{q_{L0}}`是假设以静态方式作用的m和L模态系数。这些系数是在边界节点固定的C-B假设下计算的。静态位移向量可以计算如下：


.. math:: :label: SIM3

	K_{LL} U_{L0} = F_{L,e} + F_{L,g}

将方程两边左乘:math:`\Phi_L^T`，方程:eq:`SIM3`可以重写为：:math:`\Phi_L^T K_{LL} \Phi_L q_{L0} = \Phi_L^T (F_{L,e} + F_{L,g}) = \tilde{F}_L`，或者回想一下:math:`\Phi_L^T K_{LL} \Phi_L = \Omega_L^2`，可以得到：:math:`\Omega_L^2 q_{L0} = \tilde{F}_L`，或者等价地用:math:`U_{L0}`表示：

.. math:: :label: UL02

	U_{L0} = \Phi_L \left[ \Omega_L^2 \right]^{-1} \tilde{F}_L

类似地：

.. math:: :label: UL0m2

   K_{LL} U_{L0m} = F_{L,e} + F_{L,g} \quad \rightarrow \quad U_{L0m} = \Phi_m \left[ \Omega_m^2 \right]^{-1} \tilde{F}_m

其中:math:`\tilde{F}_m = \Phi_m^T (F_{L,e} + F_{L,g})`。
注意：:math:`{\dot{U}_{L0} = \dot{q}_{L0} = \dot{U}_{L0m} = \dot{q}_{m0} = 0}`，并且:math:`{\ddot{U}_{L0} = \ddot{q}_{L0} = \ddot{U}_{L0m} = \ddot{q}_{m0} = 0}`。

在浮式情况下，当"GuyanLoadCorrection"为True时，载荷:math:`F_L`会旋转到体坐标系（更多细节参见:numref:`SD_ExtraMoment`，最终使用的方程参见:numref:`SD_summary`）。





.. _SSformulation:

状态空间公式
------------------------------

为了将SubDyn集成到FAST模块化框架中，设计了子结构结构动力学问题的状态空间公式。状态空间公式根据输入、输出、状态和参数开发。这里突出的符号与Jonkman（2013）中使用的一致。输入（由*u*标识）是提供给SubDyn的一组值，与状态一起，用于计算未来状态和系统输出。输出（*y*）是SubDyn计算并返回的一组值，它们通过输出方程（带有函数*Y*）依赖于状态、输入和/或参数。状态是SubDyn的一组内部值，受输入影响，用于计算未来状态值和输出。在SubDyn中，仅考虑连续状态。连续状态（*x*）是在时间上可微的状态，由连续时间微分方程（带有函数*X*）表征。参数（*p*）是系统的一组内部值，独立于状态和输入。此外，参数可以在初始化时完全定义，并表征系统的状态方程和输出方程。

在SubDyn中，输入定义为：

.. math:: :label: inputs

	u = \begin{bmatrix}
	u_1 \\
	u_2 \\
	u_3 \\
	u_4 \\
	u_5 \\
	\end{bmatrix} = \begin{bmatrix}
	      U_{TP} \\
	      \dot{U}_{TP} \\
	      \ddot{U}_{TP} \\
	      F_{L,e} \\
	      F_{R,e} \\
	      \end{bmatrix}


其中:math:`F_L`是HydroDyn作用在子结构每个内部节点上的水动力，:math:`F_{HDR}`是边界节点上的类似力；:math:`{U_{TP}, \dot{U}_{TP}, \text{以及} \ddot{U}_{TP}}`分别是TP挠度（6个自由度）、速度和加速度。对于独立模式下的SubDyn（与FAST解耦），假设:math:`F_{L,e}`和:math:`F_{R,e}`为零。

以一阶形式，状态定义为：

.. math:: :label: states

	x = \begin{bmatrix}
	x_1 \\
	x_2 \\
	\end{bmatrix}
	= \begin{bmatrix}
	q_m \\
	\dot{q}_m \\
	\end{bmatrix}


从系统运动方程，对应于方程:eq:`main4`的状态方程可以写成标准线性系统状态方程：

.. math:: :label: state_eq

	\dot{x} = X = A x + B u + F_X

这些状态矩阵是通过从方程:eq:`main4`的第二块行中分离模态加速度:math:`\ddot{q}_m`得到的：

.. math:: :label: ddotqm
   :nowrap:

    \begin{align}
    \ddot{q}_m = \underbrace{\Phi_m^T (F_{L,e} + F_{L,g})}_{F_m}
        - \tilde{M}_{mB} \ddot{U}_{TP}
        - \tilde{C}_{mB} \dot{U}_{TP}
        - \tilde{C}_{mm} \dot{q}_m
        - \tilde{K}_{mm} q_m
    \end{align}

从而得到以下等式：

.. math:: :label: ABFx

    A = \begin{bmatrix}
        0 & I \\
        -\tilde{K}_{mm} & -\tilde{C}_{mm}
        \end{bmatrix}
    ,\quad
    B = \begin{bmatrix}
        0 & 0  & 0 & 0 & 0 \\
        0 & -\tilde{C}_{mB} & -\tilde{M}_{mB} & \Phi_m^T & 0
    \end{bmatrix}
    ,\qquad
    F_X = \begin{bmatrix}
        0 \\
        \Phi_m^T F_{L,g}
    \end{bmatrix}


在SubDyn中，向ElastoDyn模块的输出是过渡段处的耦合（反作用）力:math:`F_{TP,cpl}`：

.. math:: :label: smally1

	y_1 = Y_1 = -F_{TP,cpl}

通过检查方程:eq:`main4`和:eq:`FTPtilde`，力从第一块行中提取：


.. math:: :label: FTP2
   :nowrap:

    \begin{align}
	F_{TP,cpl} =& \tilde{M}_{BB} \ddot{U}_{TP} + \tilde{M}_{Bm} \ddot{q}_m
            \\
           &+ \tilde{C}_{BB} \dot{U}_{TP} + \tilde{C}_{Bm} \dot{q}_m
            + \tilde{K}_{BB} U_{TP}
            - T_I^T \left( \bar{F}_{R,e} + \bar{F}_{R,g} + \bar{\Phi}_R (F_{L,e} + F_{L,g}) \right)
              \nonumber
    \end{align}


将:math:`\ddot{q}_m`的表达式代入:math:`F_{TP}`得到：

.. math:: :label: FTP3
   :nowrap:

    \begin{align}
     F_{TP,cpl} =&
      \left[ - \tilde{M}_{Bm} \tilde{K}_{mm} \right] q_m
     + \left[ \tilde{C}_{Bm} - \tilde{M}_{Bm} \tilde{C}_{mm} \right] \dot{q}_m
     \\
     &+ \left[ \tilde{K}_{BB} \right] U_{TP}
     + \left[ \tilde{C}_{BB} - \tilde{M}_{Bm} \tilde{C}_{mB} \right] \dot{U}_{TP}
     + \left[ \tilde{M}_{BB} - \tilde{M}_{Bm} \tilde{M}_{mB} \right] \ddot{U}_{TP}
    \nonumber \\
     &+ \left[ \tilde{M}_{Bm} \Phi_m^T - T_I^T \bar{\Phi}_R^T \right] (F_{L,e} + F_{L,g})
     + \left[ -T_I^T \right] (\bar{F}_{R,e} + \bar{F}_{R,g})
    \nonumber
    \end{align}


现在可以确定:math:`y_1`的输出方程为：

.. math:: :label: bigY1

	 -Y_1 = F_{TP,cpl} = C_1 x + D_1 \bar{u} + F_{Y1}

其中：


.. math:: :label: C1D1FY1u
   :nowrap:

    \begin{align}
        C_1 &= \begin{bmatrix}
        -\tilde{M}_{Bm} \tilde{K}_{mm} & \tilde{C}_{Bm} - \tilde{M}_{Bm} \tilde{C}_{mm}
        \end{bmatrix}
            \nonumber \\
        D_1 &= \begin{bmatrix}
            \tilde{K}_{BB} & \tilde{C}_{BB} - \tilde{M}_{Bm} \tilde{C}_{mB} & \tilde{M}_{BB} - \tilde{M}_{Bm} \tilde{M}_{mB} & \tilde{M}_{Bm} \Phi_m^T - T_I^T \bar{\Phi}_R^T & -T_I^T
            \end{bmatrix}
        \nonumber \\
        F_{Y1} &= \begin{bmatrix} \tilde{M}_{Bm} \Phi_m^T F_{L,g} - T_I^T \left( \bar{F}_{R,g} + \bar{\Phi}_R^T F_{L,g} \right) \end{bmatrix}
        \\
        \bar{u} &= \begin{bmatrix}
        U_{TP} \\
        \dot{U}_{TP} \\
        \ddot{U}_{TP} \\
        F_{L,e} \\
        \bar{F}_{R,e}
        \end{bmatrix}
        \nonumber
    \end{align}


注意，输入向量上使用上划线表示力仅适用于界面节点。



向HydroDyn和其他模块的输出是子结构的挠度、速度和加速度。引入两个网格来存储这些运动，记为:math:`y_2`和:math:`y_3`。对于浮式情况，这两个网格有不同的位移，其中:math:`y_2`仅包含Guyan运动，而:math:`y_3`包含完整的弹性运动。下面的方程中没有区分这一点。本节假设:math:`y_2`中包含完整的弹性运动。更多细节参见:numref:`SD_summary`节。输出运动为：

.. math:: :label: y2

	y_2 = Y_2 = \begin{bmatrix}
        	\bar{U}_R \\
                    U_L \\
            \dot{\bar{U}}_R \\
            \dot{U}_L \\
            \ddot{\bar{U}}_R \\
            \ddot{U}_L \\
	\end{bmatrix}



从CB坐标变换（方程:eq:`CB3`）以及边界节点和TP节点之间的关系（方程:eq:`UTP`），运动由下式给出：

.. math:: :label: y2motions

        \bar{U}_R  &= T_I U_{TP}
        ,\qquad
        \bar{U}_L  = \bar{\Phi}_R \bar{U}_R + \Phi_m q_m + \boldsymbol{U_{L,\text{SIM}}}

        \dot{\bar{U}}_R  &= T_I \dot{U}_{TP}
        ,\qquad
        \dot{\bar{U}}_L  = \bar{\Phi}_R \dot{\bar{U}}_R + \Phi_m \dot{q}_m

        \ddot{\bar{U}}_R  &= T_I \ddot{U}_{TP}
        ,\qquad
        \ddot{\bar{U}}_L  = \bar{\Phi}_R \ddot{\bar{U}}_R + \Phi_m \ddot{q}_m

:math:`y2motions`的表达式包含可选的SIM贡献（参见:numref:`SD_SIM`）。使用方程:eq:`ddotqm`中的:math:`\ddot{q}_m`表达式，内部加速度为：


.. math:: :label: y2internalacc

        \ddot{\bar{U}}_L  = \bar{\Phi}_R T_I \ddot{U}_{TP} + \Phi_m \left[ \Phi_m^T (F_{L,e} + F_{L,g})
                - \tilde{M}_{mB} \ddot{U}_{TP}
                - \tilde{C}_{mB} \dot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m \right]

在浮式情况下，引入了一些微妙的变化：1) 运动的Guyan部分被解析刚体运动取代；2) 为了避免与HydroDyn的耦合问题，弹性位移被设置为零（细节参见:numref:`SD_summary`节）。由于第2点，引入了第三个网格:math:`y_3`，它始终包含完整的弹性运动（完整的弹性位移、速度和加速度，包括浮式情况下的解析刚体运动）。例如，第三个网格被MoorDyn使用。

然后可以将:math:`y_2`的输出方程写为：

.. math:: :label: bigY2

  Y_2 = C_2 x + D_2 u + F_{Y2}


其中：

.. math:: :label: C2D2FY2

    C_2 = \begin{bmatrix}
           0 & 0 \\
           \Phi_m & 0 \\
           0 & 0 \\
           0 & \Phi_m \\
           0 & 0 \\
       -\Phi_m \tilde{K}_{mm} & -\Phi_m \tilde{C}_{mm} \\
          \end{bmatrix}

    D_2 = \begin{bmatrix}
           T_I & 0 & 0 & 0 & 0 \\
           \bar{\Phi}_R T_I & 0 & 0 & 0 & 0 \\
           0 & T_I  & 0 & 0 & 0 \\
           0 & \bar{\Phi}_R T_I & 0 & 0 & 0 \\
           0 & 0 & T_I  & 0 & 0  \\
           0 & -\Phi_m \tilde{C}_{mB} & \bar{\Phi}_R T_I - \Phi_m \tilde{M}_{mB} &  \Phi_m \Phi_m^T & 0
          \end{bmatrix}

    F_{Y2} = \begin{bmatrix}
           0 \\
           \boldsymbol{U_{L,\text{SIM}}} \\
           0 \\
           0 \\
           0 \\
           0 \\
           \Phi_m \Phi_m^T F_{L,g}
          \end{bmatrix}





输出和时间积分
--------------------------------


.. _SD_MemberForce:

节点载荷计算
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们首先介绍单元载荷的计算方式，然后详细说明如何获得节点载荷。

**单元载荷**：

SubDyn使用单元的全局运动计算单元坐标系中的12维单元载荷：


.. math:: :label: el_loads

	\text{单元惯性载荷：} ~ F_{I,12}^e &= [D_{c,12}]^T [m] \ddot{U}_{e,12}

	\text{单元刚度载荷：} ~ F_{S,12}^e &= [D_{c,12}]^T [k] \left[ \hat{U}_e + U_{L,\text{SIM}} \right]_{12}

其中*[k]*和*[m]*是以全局坐标系表示的单元刚度和质量矩阵，:math:`D_{c,12}`是给定单元的12×12方向余弦矩阵，下标12表示考虑单元的12个自由度，:math:`U_e`和:math:`\ddot{U}_e`是单元节点挠度和加速度，可以从方程:eq:`y2`获得，并可能包含静态位移贡献:math:`U_{L,\text{SIM}}`。目前没有很好的方法来量化每个单元的阻尼力，因此不计算单元阻尼力。

**节点载荷**

对于给定的单元节点，载荷是6维向量，第一个节点取第1到6个元素，第二个节点取第7到12个元素。按照惯例，第一个节点的6维向量乘以-1，第二个节点乘以+1：

.. math:: :label: nd_loads

         F_{6}^{n_1}  = - F_{12}^e(1:6)
         ,\quad
         F_{6}^{n_2}  = + F_{12}^e(7:12)

上述适用于惯性和刚度载荷。


**用户请求的构件节点载荷**

用户可以输出一组构件的节点载荷（参见:numref:`SD_Member_Output`）。

对于用户请求的构件节点输出，载荷要么是：1) 构件端部节点的相应6维向量，要么是：2) 构件中间节点周围两个单元的6维向量的平均值。进行平均时，两个周围单元的12维向量使用请求输出所在构件的方向余弦来表示。


**"AllOuts"节点载荷**

对于"AllOuts"节点输出，载荷不进行平均，6维向量（带有适当符号）直接写入文件。

**反力节点载荷**
（参见:numref:`SD_Reaction`）



.. _SD_Reaction:

反力计算
~~~~~~~~~~~~~~~~~~~~~~~

结构底部的反力是基础节点处的节点载荷。



此外，用户可以请求集中在子结构（塔筒中心线）和泥线处的整体反力:math:`\overrightarrow{R}`（六个力和力矩），即在全局参考系中的参考点(0, 0, -**WtrDpth**)处，其中**WtrDpth**表示水深。

为了获得这个整体反力，将:math:`N_\text{React}`个约束节点处的力和力矩以全局坐标系表示，并收集到向量:math:`F_{\text{React}}`中，这是一个(6×:math:`N_{\text{React}}`)数组。对于给定的反力节点，6维载荷向量通过将连接到该节点的所有单元的节点载荷贡献（以全局坐标系表示，此处不考虑符号）相加，并减去施加在该节点上的外部载荷(:math:`F_{HDR}`)得到。然后将所有节点的载荷:math:`F_{\text{React}}`刚性传递到(0, 0, -**WtrDpth**)以获得整体的6维数组:math:`\overrightarrow{R}`：

.. math:: :label: reaction

	\overrightarrow{R} = \begin{bmatrix}
					F_X \\
					\vdots \\
					M_Z \\
				     \end{bmatrix} = T_{\text{React}} F_{\text{React}}

其中:math:`T_{\text{React}}`是一个(6×:math:`6 N_{\text{React}}`)矩阵，如下所示：

.. math:: :label: Treact

	T_{\text{React}} = \begin{bmatrix}
		1  & 0 & 0 & 0 & 0 & 0 & \cdots & 1 & 0 & 0 & 0 & 0 & 0 \\
		0  & 1 & 0 & 0 & 0 & 0 & \cdots & 0 & 1 & 0 & 0 & 0 & 0 \\
		0  & 0 & 1 & 0 & 0 & 0 & \cdots & 0 & 0 & 1 & 0 & 0 & 0 \\
		0  & -\Delta Z_1 & \Delta Y_1   & 1 & 0 & 0 & \cdots & 0                     & -\Delta Z_{N_{\text{react}}} & \Delta Y_{N_{\text{react}}}   & 1 & 0 & 0 \\
		\Delta Z_1 & 0   & -\Delta X_1  & 0 & 1 & 0 & \cdots & \Delta Z_{N_{\text{react}}} & 0                  & -\Delta X_{N_{\text{react}}} & 0 & 1 & 0 \\
		\Delta Y_1 & \Delta X_1     & 0 & 0 & 0 & 1 & \cdots & \Delta Y_{N_{\text{react}}} & \Delta X_{N_{\text{react}}}  & 0                     & 0 & 0 & 1 \\
		\end{bmatrix}

其中:math:`{X_i, ~Y_i}`和:math:`Z_i`（:math:`{i = 1 .. N_{\text{React}}}`）是边界节点相对于参考点的坐标。



.. _TimeIntegration:

时间积分
~~~~~~~~~~~~~~~~~

在时间:math:`{t=0}`时，初始状态被指定为初始条件（在SubDyn中假设全为零），初始输入被提供给SubDyn。在随后的每个时间步中，输入和状态是已知值，输入:math:`u(t)`来自ElastoDyn和HydroDyn，状态:math:`x(t)`由前一个时间步积分得到。所有参数矩阵都在SubDyn初始化模块中计算。已知:math:`u(t)`和:math:`x(t)`，可以使用状态方程:math:`{\dot{x}(t) = X(u,x,t)}`（参见方程:eq:`state_eq`）计算:math:`{\dot{x}(t)}`，并求解方程:eq:`bigY1`和:eq:`bigY2`计算输出:math:`y_1(t)`和:math:`y_2(t)`。还可以使用方程:eq:`el_loads`计算单元力。下一个时间步的状态:math:`{x(t + \Delta t)}`通过积分获得：

.. math:: :label: integration

	\left[ u(t), \dot{x}(t), x(t) \right] \xrightarrow[]{\text{积分}} x(t + \Delta t)


对于松耦合，SubDyn使用自己的积分器，而对于紧耦合，所有模块的状态将由耦合代码中的积分器同时积分。SubDyn内置的用于松耦合的时间积分器选项有：

- 四阶显式龙格-库塔法

- 四阶显式亚当斯-巴什福思预测法

- 四阶显式亚当斯-巴什福思-穆尔顿预测-校正法

- 隐式二阶亚当斯-穆尔顿法。

更多信息请参考任何数值方法参考文献，例如:cite:`chapra2010`。






.. _SD_summary:

实现的公式总结
--------------------------------------

本节总结了SubDyn中当前实现的方程，区分了浮式和固定式基础情况。我们引入了算子:math:`R_{g2b}`（全局到体旋转）和:math:`R_{b2g}`（体到全局旋转），它们作用于右侧的数组。这些算子旋转数组中存在的各个3维向量。当应用于载荷向量（例如:math:`F_L`）时，在通过:math:`\boldsymbol{T}`矩阵将载荷传递到降阶系统之前，旋转实际上应用于全系统的载荷。


状态方程
~~~~~~~~~~~~~~

**固定式基础情况**

.. math::
   :nowrap:

    \begin{align}
    \ddot{q}_m = \Phi_m^T F_L
                - \tilde{M}_{mB} \ddot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m
    \end{align}

注意：如果用户请求**GuyanLoadCorrection**，:math:`F_L`包含"附加力矩"。

**未使用"GuyanLoadCorrection"的浮式情况**

.. math::
   :nowrap:

    \begin{align}
    \ddot{q}_m = \Phi_m^T F_L
                - \tilde{M}_{mB} \ddot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m
    \end{align}

注意：:math:`F_L`*不*包含"附加力矩"。


**使用"GuyanLoadCorrection"的浮式情况**

.. math::
   :nowrap:

    \begin{align}
    \ddot{q}_m = \Phi_m^T R_{g2b} F_L
                - \tilde{M}_{mB} R_{g2b} \ddot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m
    \end{align}

注意：:math:`F_L`*不*包含"附加力矩"。外部和重力载荷以及TP的加速度会旋转到体坐标系。


输出：界面反力
~~~~~~~~~~~~~~~~~~~~~~~~~~

**固定式基础情况**

.. math::
   :nowrap:

    \begin{align}
     -Y_1 = F_{TP,cpl} =
       \begin{Bmatrix}
       f_{TP,cpl} \\
       m_{TP,cpl} \\
       \end{Bmatrix}
     &=
      \left[ - \tilde{M}_{Bm} \tilde{K}_{mm} \right] q_m
     + \left[ -\tilde{M}_{Bm} \tilde{C}_{mm} \right] \dot{q}_m
     \\
     &+ \left[ \tilde{K}_{BB} \right] U_{TP}
     + \left[ \tilde{C}_{BB} \right] \dot{U}_{TP}
     + \left[ \tilde{M}_{BB} - \tilde{M}_{Bm} \tilde{M}_{mB} \right] \ddot{U}_{TP}
    \nonumber \\
     &+ \left[ \tilde{M}_{Bm} \Phi_m^T \right] F_L + \left[ - T_I^T \bar{\Phi}_R^T \right] F_{L}
     + \left[ - T_I^T \right] \bar{F}_R
    \nonumber
    \end{align}

注意：:math:`F_L`和:math:`\bar{F}_R`如果用户请求则包含"附加力矩"。
如果是这种情况，以下附加项会添加到:math:`Y_1`的力矩部分：
:math:`m_{Y_1,\text{extra}} = u_{TP} \times f_{TP,cpl}`。



**未使用"GuyanLoadCorrection"的浮式情况**

.. math::
   :nowrap:

    \begin{align}
     -Y_1 = F_{TP,cpl} =&
      \left[ - \tilde{M}_{Bm} \tilde{K}_{mm} \right] q_m
     + \left[ -\tilde{M}_{Bm} \tilde{C}_{mm} \right] \dot{q}_m
     \\
     &+ \left[ \tilde{K}_{BB} \right] U_{TP}
     + \left[ \tilde{C}_{BB} \right] \dot{U}_{TP}
     + \left[ \tilde{M}_{BB} - \tilde{M}_{Bm} \tilde{M}_{mB} \right] \ddot{U}_{TP}
    \nonumber \\
     &+ \left[ \tilde{M}_{Bm} \Phi_m^T \right] F_L + \left[ - T_I^T \bar{\Phi}_R^T \right] F_{L}
     + \left[ - T_I^T \right] \bar{F}_R
    \nonumber
    \end{align}

注意：:math:`F_L`和:math:`\bar{F}_R`*不*包含"附加力矩"。


**使用"GuyanLoadCorrection"的浮式情况**

.. math::
   :nowrap:

    \begin{align}
     -Y_1 = F_{TP,cpl} =&
       R_{b2g} \left[ - \tilde{M}_{Bm} \tilde{K}_{mm} \right] q_m
     + R_{b2g} \left[ -\tilde{M}_{Bm} \tilde{C}_{mm} \right] \dot{q}_m
     \\
     &+ \left[ \tilde{K}_{BB} \right] U_{TP}
     + \left[ \tilde{C}_{BB} \right] \dot{U}_{TP}
     + \left[ \tilde{M}_{BB} - \tilde{M}_{Bm} \tilde{M}_{mB} \right] \ddot{U}_{TP}
    \nonumber \\
     &+ R_{b2g} \left[ \tilde{M}_{Bm} \Phi_m^T \right] R_{g2b} F_L + \left[ - T_I^T \bar{\Phi}_R^T \right] F_{L,\text{extra}}
     + \left[ - T_I^T \right] \bar{F}_{R,\text{extra}}
    \nonumber
    \end{align}


注意：1) :math:`F_{L,\text{extra}}`和:math:`F_{R,\text{extra}}`在Guyan贡献中包含"附加力矩"；2) 对于Craig-Bampton贡献，载荷使用算子:math:`R_{g2b}`（全局到体）旋转到体坐标系；3) 旋转:math:`R_{b2g} \tilde{M}_{Bm} \tilde{M}_{mB} R_{g2b}`不执行，因为它会引入稳定性问题。

输出：节点运动
~~~~~~~~~~~~~~~~~~~~~

**固定式基础情况**

.. math:: :label: nodalMotionFixed

        \bar{U}_R  &= T_I U_{TP}
        ,\qquad
        \bar{U}_L  = \bar{\Phi}_R \bar{U}_R + \Phi_m q_m + U_{L,\text{SIM}}

        \dot{\bar{U}}_R  &= T_I \dot{U}_{TP}
        ,\qquad
        \dot{\bar{U}}_L  = \bar{\Phi}_R \dot{\bar{U}}_R + \Phi_m \dot{q}_m

        \ddot{\bar{U}}_R  &= T_I \ddot{U}_{TP}
        ,\qquad
        \ddot{\bar{U}}_L  = \bar{\Phi}_R \ddot{\bar{U}}_R + \Phi_m \left[ \Phi_m^T F_L
                - \tilde{M}_{mB} \ddot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m \right]


注意：如果用户请求**GuyanLoadCorrection**，:math:`F_L`包含"附加力矩"。
网格:math:`y_2`和:math:`y_3`是相同的（使用:math:`Phi_R`计算Guyan位移，包含弹性位移以及弹性速度/加速度）。



**浮式情况**

.. math:: :label: nodalMotionFloating

        \bar{U}_R  = U_{R,\text{rigid}}
        ,\qquad
        \bar{U}_L  = U_{L,\text{rigid}} + 0 \cdot R_{b2g} \left( \Phi_m q_m +  U_{L,\text{SIM}} \right)

        \dot{\bar{U}}_R  = \dot{U}_{R,\text{rigid}}
        ,\qquad
        \dot{\bar{U}}_L  = \dot{U}_{L,\text{rigid}} + R_{b2g} \Phi_m \dot{q}_m

        \ddot{\bar{U}}_R  = \ddot{U}_{R,\text{rigid}}
        ,\qquad
        \ddot{\bar{U}}_L  = \ddot{U}_{L,\text{rigid}} + R_{b2g} \Phi_m \left[ \Phi_m^T R_{g2b} F_L
                - \tilde{M}_{mB} R_{g2b} \ddot{U}_{TP}
                - \tilde{C}_{mm} \dot{q}_m
                - \tilde{K}_{mm} q_m \right]

其中：1) :math:`F_L`不包含附加力矩；2) 当GuyanLoadCorrection为True时使用算子:math:`R_{g2b}`和:math:`R_{b2g}`；3) 出于稳定性目的，:math:`y_2`（供HydroDyn使用）中的弹性位移设置为0（假设这些位移很小），但:math:`y_3`（供MoorDyn使用）中的弹性位移不设置为零；4) Guyan运动（:math:`U_{L,\text{rigid}}`）使用解析刚体运动计算。
对于给定节点:math:`P`，它在未变形配置中距离界面的位置为:math:`r_{IP,0}`，由于刚体运动引起的位置（从界面点算起）、位移、平移速度和加速度为：


.. math::
        r_{IP} &= R_{b2g} r_{IP,0}
        ,\quad
        u_P = r_{IP} - r_{IP,0} + u_{TP}
          ,\quad

       \dot{u}_P &= \dot{u}_{TP} + \omega_{TP} \times r_{IP}
       ,\quad
       \ddot{u}_P = \ddot{u}_{TP} + \dot{\omega}_{TP} \times r_{IP} + \omega_{TP} \times (\omega_{TP} \times r_{IP})

其中:math:`\omega_{TP}`是过渡段的角速度。由于刚体旋转，每个节点的小角度旋转、角速度和角加速度与界面值:math:`\theta_{TP}`、:math:`\omega_{TP}`和:math:`\dot{\omega}_{TP}`相同，因此：

.. math::
      U_{P,\text{rigid}} = \{ u_P ; \theta_{TP} \}^T
      ,\quad
      \dot{U}_{P,\text{rigid}} = \{ \dot{u}_P ; \omega_{TP} \}^T
      ,\quad
      \ddot{U}_{P,\text{rigid}} = \{ \ddot{u}_P ; \dot{\omega}_{TP} \}^T

其中:math:`P`是属于R或L节点集的点。


文件输出：
~~~~~~~~~~~~~~~~

**运动**：写入文件的节点运动是全局坐标系下的。
对于固定式基础情况，它们是:math:`\bar{U}_L = \bar{\Phi}_R \bar{U}_R + \Phi_m q_m + U_{L,\text{SIM}}`（参见方程:eq:`nodalMotionFixed`）。
对于浮式情况，它们是:math:`\bar{U}_L = U_{L,\text{rigid}} + R_{b2g} (\Phi_m q_m + U_{L,\text{SIM}})`。
注意，浮式情况的输出包含弹性运动（类似于返回给MoorDyn的内容），而这些运动不会返回给HydroDyn的耦合代码（参见方程:eq:`nodalMotionFloating`中的"0"）。


**载荷**：写入文件的节点载荷是单元坐标系下的。固定式基础和浮式情况的处理过程相同。
</think_never_used_51bce0c785ca2f68081bfa7d91973934>
<seed:tool_call><function name="finish_task">
<parameter name="finish_reason" string="true">已完成 SubDyn 理论文档的全部翻译，包括所有章节、公式和图表说明，严格遵循术语统一和格式保留要求。