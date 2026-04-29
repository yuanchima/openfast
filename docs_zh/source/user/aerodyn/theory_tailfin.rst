.. _TF-aerotheory:

尾翼空气动力学理论
============================

符号说明
---------

**尾翼气动参考点**

尾翼气动参考点 :math:`\boldsymbol{x}_\text{ref}` 是计算尾翼气动载荷的点。结构求解器在每个时间步计算参考点的瞬时位置、速度、加速度。参考点相对于塔顶的初始位置是用户输入。典型选择是翼的前缘/顶点，或者接近零攻角下的压力中心的点。其他气动输入（例如气动力矩系数）需要与参考点的选择保持一致。

**尾翼坐标系**

惯性坐标系和尾翼坐标系如 :numref:`figTFcoord1` 所示。从惯性坐标系到尾翼坐标系的变换矩阵为 :math:`\boldsymbol{R}_\text{tf,i}`。

.. _figTFcoord1:
.. figure:: figs/TailFinCoord.png
   :width: 70%

   尾翼空气动力学使用的坐标系和速度矢量

参考方向（当结构未偏转时），变换矩阵为：

.. math::  :label: tfRrfiinit

   \boldsymbol{R}_\text{tf,i} = \operatorname{EulerConstruct}(\theta_\text{bank}, \theta_\text{tilt}, \theta_\text{skew})

对于垂直翼的常见应用，三个角度均为零。

.. :red:`TODO: 当前实现中的角度顺序可能不同，使用(3-2-1)而不是上面的(1-2-3)`


**速度**

定义以下速度矢量（全局坐标系下的3D矢量）（参见 :numref:`figTFcoord1`）：

- :math:`\boldsymbol{V}_\text{wind}`：参考点处的未受扰风速矢量
- :math:`\boldsymbol{V}_\text{dist}`：参考点处的受扰风速矢量（受扰风包含塔筒对流动的影响）。AeroDyn有内部方法从 :math:`\boldsymbol{V}_\text{wind}` 计算 :math:`\boldsymbol{V}_\text{dist}`。
- :math:`\boldsymbol{V}_\text{elast}`：参考点处的结构平移速度矢量
- :math:`\boldsymbol{V}_\text{ind}`：参考点处尾迹产生的诱导速度（目前假设为零）
- :math:`\boldsymbol{\omega}`：翼的结构旋转速度

..  :red:`目前我们使用"wind"，但未来可能使用"dist"。在下面的理论中，我们只需将所有"wind"替换为"dist"`。

所有速度（除了由AeroDyn内部计算的 :math:`\boldsymbol{V}_\text{ind}` 和 :math:`\boldsymbol{V}_\text{dist}` 之外）都作为输入提供给气动求解器。翼型感受到的相对风为：

.. math::  :label: tfVrel

   \boldsymbol{V}_\text{rel} =
        \boldsymbol{V}_\text{wind}
       -\boldsymbol{V}_\text{elast}
       +\boldsymbol{V}_\text{ind}



**攻角**

攻角定义在尾翼坐标系的 :math:`x_\text{tf}-y_\text{tf}` 平面中，如 :numref:`figTFcoord2` 所示。

.. _figTFcoord2:
.. figure:: figs/TailFinAirfoilCoord.png
   :width: 70%

   尾翼翼型坐标系和x-y平面中攻角的定义

我们将 :math:`V_{\text{rel},\perp}` 记为 :math:`\boldsymbol{V}_\text{rel}` 在该平面上的投影。攻角由该矢量的分量给出：

.. math::  :label: tfalpha

   \alpha = \arctan\frac{V_{\text{rel},y_\text{tf}}}{V_{\text{rel},x_\text{tf}}}

在本实现中，使用 `atan2` 函数计算攻角。


**载荷**

如果已知无量纲系数，它们可以按如下方式投影到 :math:`x_\text{tf}-y_\text{tf}` 平面：

.. math::  :label: tfCxCy

       C_{x_\text{tf}}(\alpha)  = -C_l(\alpha) \sin\alpha + C_d(\alpha)\cos\alpha
       ,\quad
       C_{y_\text{tf}}(\alpha)  =  C_l(\alpha) \cos\alpha + C_d(\alpha)\sin\alpha

因此载荷为：

.. math::  :label: tffxfymz

      f_{x_\text{tf}} = \frac{1}{2}\rho V_{\text{rel},\perp}^2 A  \,C_{x_\text{tf}}(\alpha)
                   ,\quad
      f_{y_\text{tf}} = \frac{1}{2}\rho V_{\text{rel},\perp}^2 A  \,C_{y_\text{tf}}(\alpha)
                   ,\quad
      m_{z_\text{tf}} = \frac{1}{2}\rho V_{\text{rel},\perp}^2 Ac \, C_m(\alpha)


一旦在尾翼坐标系中得到载荷，它们按如下方式转换到惯性坐标系：

.. math::  :label: tfforcesi

   \left.\boldsymbol{f}\right|_{i} = \boldsymbol{R}_\text{tf,i}^t  \left.\boldsymbol{f}\right|_\text{tf}
    = \boldsymbol{R}_\text{tf,i}^t
    \begin{bmatrix}
    f_{x_\text{tf}}\\
    f_{y_\text{tf}}\\
    0\\
    \end{bmatrix}
    ,\qquad
   \left.\boldsymbol{m}\right|_{i} = \boldsymbol{R}_\text{tf,i}^t  \left.\boldsymbol{m}\right|_\text{tf}
    = \boldsymbol{R}_\text{tf,i}^t
    \begin{bmatrix}
    0\\
    0\\
    m_{z_\text{tf}}\\
    \end{bmatrix}


**诱导速度**

参考点处尾迹产生的诱导速度会影响相对风，从而影响尾翼的攻角。实现了不同的模型来计算该诱导速度。
作为一阶近似，该速度可以设置为零（对应输入 `TFinIndMod=0`）：

.. math::  :label: TFVindZero

    \boldsymbol{V}_\text{ind}=0

也可以使用转子平均诱导速度作为估计值（`TFinIndMod=1`）。它计算为所有叶片和气动节点的平均诱导速度：

.. math::  :label: TFVindRtAvg

    \boldsymbol{V}_\text{ind}=\frac{1}{n_B n_r}\sum_{i_b=1..n_B} \sum_{i_r=1..n_r}  \boldsymbol{V}_{\text{ind},\text{blade}}[i_b, i_r]

其中 :math:`\boldsymbol{V}_{\text{ind},\text{blade}}[i_b, i_r]` 是叶片 :math:`i_b` 在径向节点 :math:`i_r` 处的诱导速度矢量。

.. :red:`注意：该平均值对应于AeroDyn中入流盘平均的处理方式。未来，我们可以使用按半径加权的方法，或者使用预计算的系数，如Envision所做的那样。`

更先进的模型可以在尾迹边界外时将诱导速度设置为零，或者包含类似塔筒阴影的尾迹模型。此类选项目前尚不可用。


基于极曲线的模型
-----------------

在基于极曲线的模型中，用户提供气动系数 :math:`C_l, C_d, C_m`，作为攻角函数的表格数据。气动力矩假设是在参考点处提供的。常见做法是将零攻角下的压力中心用于极曲线数据，因此用户可能希望选择这样的点作为翼的参考点。
表格数据作为AeroDyn输入文件中 `AFNames` 给出的翼型列表的一部分提供。用户只需在列表 `AFNames` 中指定索引 `TFinAFIndex`，以指示尾翼使用哪个极曲线。


非定常细长体模型
---------------------------

尾翼的非定常空气动力学基于非定常细长体理论建模。该理论被扩展以包含大偏航角的影响 :cite:`ad-hammam_NREL:2023`。为了简化实现，假设尾翼的臂长远大于弦长，且特征时间（弦长/风速）很小。

尾翼上的法向力可以描述为三个贡献的和（势升力、涡升力和阻力），由分离函数 :math:`x_i` 加权：

.. math::
    :label: tfusbforce

    N = \frac{\rho}{2} A_{tf} \bigg(  K_p x_1 V_{\text{rel},x} V_{\text{rel},y} +  \Big[x_2 K_v+(1- x_3)C_{Dc} \Big] V_{\text{rel},y}\big|V_{\text{rel},y}\big|\bigg)

其中 :math:`\rho` 是空气密度，:math:`A_{tf}` 是尾翼面积，:math:`K_p` 是势升力系数，:math:`K_v` 是涡升力系数，:math:`C_{Dc}` 是阻力系数。
注意OpenFAST的符号约定与 :cite:`ad-hammam_NREL:2023` 中使用的略有不同。这反映在公式 :eq:`tfusbforce` 中。

:math:`x_i` 是使用准稳态近似计算的分离函数：

.. math::  :label: TFUSBxiEquation

    x_i = (1+exp{[\sigma_i (|\gamma_{tf}|-\alpha^*_i)]})^{-1}

其中 :math:`\sigma_i` 是表征分离函数衰减的经验常数，:math:`\gamma_{tf}` 是尾翼相对于自由流风(:math:`V_{\text{wind}}`)的偏航角，:math:`\alpha^*_i` 是分离函数的特征角。
:math:`x_i` 的取值在0和1之间，用于激活或停用势升力、涡升力和阻力对尾翼法向力的贡献。

假设法向力作用在用户定义的尾翼参考点上，并相应计算法向力的力矩。
