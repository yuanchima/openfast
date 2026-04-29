.. _AD_theory:

AeroDyn 理论
==============

本理论手册仍在编写中，更多细节请参考 AeroDyn 14 手册 :cite:`ad-AeroDyn:manual`。自 AeroDyn 14 以来发生了许多变化（例如 BEM 公式、BEM 方程中使用的坐标系、动态失速、动态 BEM），但这些变化尚未在此处记录。


定常 BEM
~~~~~~~~~~

定常叶素动量（BEM）方程作为约束方程求解，公式遵循 Ning 的描述 :cite:`ad-Ning:2014`。


.. _AD_DBEMT:

动态 BEM 理论（DBEMT）
~~~~~~~~~~~~~~~~~~~~~~~~~~


AeroDyn 中实现了两个等效版本的 Oye 动态入流模型。
第一个使用离散时间，可与恒定 τ₁ 模型（``DBEMT_Mod=1``）或可变 τ₁ 模型（``DBEMT_Mod=2``）一起使用，但不能用于线性化。
第二个版本使用连续时间状态空间公式（``DBEMT_Mod=1``），它假设 τ₁ 恒定，可用于线性化。
对于相同的 :math:`\tau_1` 值，离散时间和连续时间公式返回完全相同的结果。



Oye 的动态入流模型由两个一阶微分方程组成（参见 :cite:`ad-Branlard:book`）：

.. math::
   \begin{aligned}
       \boldsymbol{W}_\text{int}+\tau_1    \boldsymbol{\dot{W}}_\text{int} &= \boldsymbol{W}_\text{qs} + k \tau_1 \boldsymbol{\dot{W}}_\text{qs} \\
       \boldsymbol{W}+\tau_2 \boldsymbol{\dot{W}} &= \boldsymbol{W}_\text{int}
   \end{aligned}

其中
:math:`\boldsymbol{W}` 是转子处的动态诱导速度矢量（在给定叶片位置和径向位置处），
:math:`\boldsymbol{W}_\text{qs}` 是准稳态诱导速度，
:math:`\boldsymbol{W}_\text{int}` 是耦合准稳态和实际诱导速度的中间值（如果准稳态诱导速度不连续，该值可能不连续）。
:math:`(\dot{\ })` 表示时间导数。
耦合常数 :math:`k` 的值在 0 到 1 之间，通常选择 :math:`k=0.6`。
Oye 的动态入流模型依赖于两个时间常数 :math:`\tau_1` 和 :math:`\tau_2`：

.. math::
        \tau_1=\frac{1.1}{1-1.3 \min(\overline{a},0.5)} \frac{R}{\overline{U}_0}
        , \qquad
        \tau_2 =\left[ 0.39-0.26\left(\frac{r}{R}\right)^2\right] \tau_1

其中 :math:`R` 是转子半径，:math:`\overline{U}_0` 是转子上方的平均风速，:math:`\overline{a}` 是转子上方的平均轴向诱导因子，:math:`r` 是沿叶片的径向位置。
对于 ``DBEMT_Mod=1`` 或 ``DBEMT_Mod=3``，用户需要提供 :math:`\tau_1` 的值。



动态入流模型的连续时间状态空间公式（``DBEMT_Mod=3``）在 :cite:`ad-Branlard:2022` 中推导得出。

.. math::
   \begin{align}
      \begin{bmatrix}
      \boldsymbol{\dot{W}}_\text{red}\\
      \boldsymbol{\dot{W}}\\
      \end{bmatrix}
      =
      \begin{bmatrix}
      -\frac{1}{\tau_1}\boldsymbol{I}_2 & \boldsymbol{0} \\
       \frac{1}{\tau_2}\boldsymbol{I}_2 &
      -\frac{1}{\tau_2}\boldsymbol{I}_2 \\
      \end{bmatrix}
      \begin{bmatrix}
      \boldsymbol{W}_\text{red}\\
      \boldsymbol{W}\\
      \end{bmatrix}
      +
      \begin{bmatrix}
       \frac{1-k}{\tau_1} \\
       \frac{k}{\tau_2}\\
      \end{bmatrix}
     \boldsymbol{W}_\text{qs}
   \end{align}

其中
:math:`\boldsymbol{I}_2` 是 2×2 单位矩阵，
:math:`\boldsymbol{W}_\text{red}` 是简化诱导速度，是准稳态诱导速度的连续、缩放和滞后版本，定义为：

.. math::
    \boldsymbol{W}_\text{int} = \boldsymbol{W}_\text{red} + k \boldsymbol{W}_\text{qs}


该模型的离散时间版本记录在未发表的 DBEMT 手册中。当前的离散时间公式很复杂，未来可以通过使用 :math:`\boldsymbol{W}_\text{red}` 来简化。




.. _AD_twr_shadow:

塔筒阴影模型
~~~~~~~~~~~~~~~~~~~

Powles 塔筒阴影模型（**TwrShadow=1**）公式如下：

.. math::
   u_{TwrShadow} = - \frac{C_d}{  \sqrt{\overline{r}}  }
               \cos\left( \frac{\pi/2 \overline{y}}{\sqrt{\overline{r}}}\right)^2

其中 :math:`\overline{r} = \sqrt{ \overline{x}^2 + \overline{y}^2 }`。


Eames 塔筒阴影模型（**TwrShadow=2**）公式如下：

.. math::
   u_{TwrShadow} = -\frac{C_d}{ TI \: \overline{x} \, \sqrt{2 \pi }  }
               \exp{\left(  -\frac{1}{2}  \left(\frac{ \overline{y}}{ TI \: \overline{x} } \right)^2 \right) }

其中 :math:`TI` 是塔筒节点处的湍流强度。


.. _AD_buoyancy:

浮力
~~~~~~~~

当固体物体浸没在流体中时，其表面会受到流体静压力产生的净力，即浮力。这种力在密度较低的流体（如空气）中通常可以忽略，但在密度较高的流体（如水）中可能会很大。为了捕捉这种力对 MHK 涡轮机的影响，需要计算涡轮机叶片、塔筒、轮毂和机舱的浮力载荷。所有组件都忽略海洋生物附着的影响。:numref:`AD_buoy_coords` 至 :numref:`AD_buoy_hubnacelle` 详细介绍了坐标系以及叶片、塔筒、轮毂和机舱的浮力计算方法。

.. _AD_buoy_coords:

坐标系
------------------
作用在单元上的浮力取决于其瞬时朝向和深度。朝向由航向角和倾角定义，每个时间步都会为每个单元计算这些角度。总水深由用户定义，相对于静水位（或者当使用 AeroDyn 驱动程序独立运行 AeroDyn 时，相对于平均海平面）。每个单元的瞬时深度基于其在每个时间步的全局坐标位置。

.. _AD_buoy_bladestower:

叶片和塔筒
----------------
为了实现高效的解析解，叶片和塔筒被建模为锥形圆柱。锥形圆柱的横截面积设置为与叶片或塔筒的横截面积相等。通过将叶片或塔筒分解为给定长度的单元，并在每个单元的湿表面积上积分流体静压力来估算载荷。对于叶片，载荷施加在用户指定的浮心位置。对于塔筒，载荷施加在中心线上。适用时，通过计算单元暴露的轴向端面上的流体压力来考虑端部效应。假设塔筒要么嵌入海床，要么连接到其他支撑结构构件，因此不需要考虑塔筒底部的端部效应。对于带有支撑结构的 MHK 涡轮机（即除了简单嵌入海床的塔筒之外的任何结构），目前建议在 HydroDyn 中对包括塔筒在内的整个支撑结构进行建模。未来版本将支持忽略 AeroDyn 中建模的塔筒与 HydroDyn 中建模的平台之间界面处的流体载荷。


叶片和塔筒的浮力计算按照以下步骤完成：

1.  计算与单元几何相关的不随时间变化的参数
2.  检查没有单元穿过自由表面或低于海床
3.  计算每个单元的瞬时朝向和深度
4.  在每个单元的湿表面积上积分流体静压力，并表示为作用在浮心处的力
5.  对于叶片，计算叶根和叶尖轴向端面上的浮力；将叶尖力添加到相邻单元并存储叶根力
6.  对于塔筒，计算并存储塔筒顶端轴向端面上的浮力
7.  将浮力载荷从浮心转移到气动中心
8.  按照 OpenFAST 期望的格式表示浮力载荷
9.  将浮力载荷添加到气动载荷中

虽然叶片和塔筒的浮力载荷不是基于体积的，但这些组件的体积会写入 AeroDyn 摘要文件以供参考。叶片和塔筒的体积通过求和每个单元的体积来计算，单元假设为锥形圆柱。单个单元的体积 :math:`(V_{elem})` 公式如下：

.. math::
   V_{elem} = \frac{\pi}{3} (r_i^2 + r_i r_{i+1} + r_{i+1}^2) dl

其中 :math:`r_i` 是节点 :math:`i` 处的单元半径，:math:`r_{i+1}` 是节点 :math:`i+1` 处的单元半径，:math:`dl` 是单元长度。


.. _AD_buoy_hubnacelle:

轮毂和机舱
---------------
轮毂和机舱被视为独立组件。浮力由轮毂或机舱的体积决定，并施加在用户指定的浮心位置。由于接头位置不暴露于流体压力，因此需要对轮毂与叶片之间以及机舱与塔筒之间的接头进行修正。轮毂与机舱之间的接头不需要修正。

轮毂和机舱的浮力计算按照以下步骤完成：

1.  检查组件没有穿过自由表面或低于海床
2.  计算组件的瞬时深度
3.  根据组件的体积计算浮力
4.  将浮力载荷从浮心转移到气动中心
5.  对于轮毂，修正载荷以考虑与每个叶片的接头
6.  对于机舱，修正载荷以考虑与塔筒的接头

.. _AD_addedmass_inertia:

附加质量和流体惯性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

附加质量载荷由物体和流体的加速度引起。这些力在密度较低的流体（如空气）中通常可以忽略，但在密度较高的流体（如水）中可能会很大。为了捕捉这些力对 MHK 涡轮机的影响，需要计算涡轮机叶片和塔筒的附加质量和流体惯性载荷。通过根据 Morison 方程中的相应项计算附加质量和流体惯性力，在每个叶片或塔筒节点处估算单位长度载荷。所得载荷与先前计算的单位长度水动力和/或浮力载荷相加。叶片的载荷施加在气动中心。塔筒的载荷施加在中心线上。忽略海洋生物附着和端部效应，且构件不允许穿过自由表面（即构件始终完全浸没）。不考虑压载。节点不需要均匀间隔，忽略轴向载荷。假设塔筒是轴对称的（两个横向方向使用相同的系数），但叶片不是（弦向法向和切向使用不同的系数，以及俯仰附加质量系数）。

.. _AD_addedmass_inertia_Morison:

Morison 方程
------------------
附加质量和流体惯性载荷根据 Morison 方程中的相应项计算。附加质量力公式如下：

.. math::
   F_{a} = \rho C_a V (\dot{u} - \dot{v})

其中 :math:`\rho` 是流体密度，:math:`C_a` 是附加质量系数，:math:`V` 是单元体积，:math:`\dot{u}` 是流体加速度，:math:`\dot{v}` 是物体加速度。

流体惯性力公式如下：

.. math::
   F_{i} = \rho C_p V \dot{u}

其中 :math:`C_p` 是动压系数。

流体密度、附加质量和动压系数由用户指定。通过将相关系数设置为零，可以关闭附加质量和流体惯性载荷。有关计算附加质量系数的更多信息，请参见 :numref:`AD_user_guide`（"使用计算流体动力学确定浮动水动力涡轮机叶片的附加质量系数"）。物体和流体加速度在内部计算并传递给 AeroDyn。物体加速度可从结构求解器（或驱动程序）获得，流体加速度根据入流速度时间序列计算。AeroDyn 中将附加质量和流体惯性载荷计算为单位长度载荷。因此，:math:`V` 取为相关节点处的横截面积。对于叶片，法向和切向项的参考横截面积为弦长×厚度（:math:`ct`）。这可以表示为 :math:`(c^2)(t/c)`，其中 :math:`t/c`（即 ``t_c``）在 AeroDyn 叶片输入文件中指定，且不能小于 0。对于塔筒，参考横截面积为 :math:`\pi r^2`，其中 :math:`r` 计算为（0.5 × ``TwrDiam``）。``BlCpn``、``BlCpt``、``BlCan`` 和 ``BlCat`` 系数的归一化应为 :math:`\rho ct`；``BlCam`` 系数的归一化应为 :math:`(1/12)\rho ct(c^2+t^2)`；``TwrCp`` 和 ``TwrCa`` 系数的归一化应为 :math:`\rho\pi(0.5 \times \text{``TwrDiam``})^2`。

叶片附加质量和流体惯性
----------------------------------
在叶片坐标系中，计算弦向法向、弦向切向和俯仰方向的附加质量和流体惯性载荷。用户在 AeroDyn 叶片输入文件中定义以下系数：

- ``BlCpn`` 指定叶片弦向法向动压系数；要忽略叶片上的弦向法向流体惯性载荷，将 ``BlCpn`` 设置为 0

- ``BlCpt`` 指定叶片弦向切向动压系数；要忽略叶片上的弦向切向流体惯性载荷，将 ``BlCpt`` 设置为 0

- ``BlCan`` 指定叶片弦向法向附加质量系数，不能小于 0；要忽略叶片上的弦向法向附加质量载荷，将 ``BlCan`` 设置为 0

- ``BlCat`` 指定叶片弦向切向附加质量系数，不能小于 0；要忽略叶片上的弦向切向附加质量载荷，将 ``BlCat`` 设置为 0

- ``BlCam`` 指定叶片俯仰附加质量系数，不能小于 0；要忽略叶片上的俯仰附加质量载荷，将 ``BlCam`` 设置为 0

塔筒附加质量和流体惯性
----------------------------------
在塔筒坐标系中计算横向的附加质量和流体惯性载荷。用户在 AeroDyn 主输入文件中定义以下系数：

- ``TwrCp`` 指定塔筒横向动压系数；要忽略塔筒上的流体惯性载荷，将 ``TwrCp`` 设置为 0

- ``TwrCa`` 指定塔筒横向附加质量系数，不能小于 0；要忽略塔筒上的附加质量载荷，将 ``TwrCa`` 设置为 0
