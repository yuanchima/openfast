
.. _AD_UA:

非定常空气动力学
=====================

非定常空气动力学（UA）模型用于解释流动滞后现象，包括非定常附着流、后缘流动分离、动态失速和流动再附着。*动态失速* 指的是可能引起或延迟失速行为的快速气动变化 :cite:`ad-Branlard:book`。风速的快速变化（例如，当叶片经过塔筒阴影时）会导致沿翼型的气流突然脱离然后再附着。叶片表面的这种效应无法用定常空气动力学预测，但可能影响风机运行，不仅在叶片遇到塔筒阴影时，还包括在偏斜流和湍流风条件下运行时。动态失速效应发生的时间尺度约为叶片处相对风穿过叶片弦长的时间，近似为 :math:`c/\Omega r`。对于大型风力发电机，这个时间在叶片根部约为0.5秒，在叶尖约为0.001秒。动态失速会导致随着风速增加出现高瞬态力，但失速被延迟。


.. _ua_theory:

理论
------

下面介绍AeroDyn中实现的不同动态失速模型。


.. _ua_notations:

符号和定义
~~~~~~~~~~~~~~~~~~~~~~~~~

有关翼型输入文件中所有输入的全面描述，请参见 :numref:`airfoil_data_input_file`（包括下面重复的一些内容）。

翼型截面坐标系和主要变量在 :numref:`fig:UAAirfoilSystem` 中给出，并在下面进一步描述：

.. figure:: figs/UAAirfoilSystem.svg
   :width: 70%
   :name: fig:UAAirfoilSystem

   非定常空气动力学模块中使用的翼型截面坐标系定义

- **气动中心（AC）**：翼型截面上假设气动力和力矩作用的点。对于常规翼型，通常靠近1/4弦长点；对于圆形截面，位于中心。

- **“3/4”弦长点**：在原始公式中，该点指的是弦轴上位于前缘后方3/4弦长处的点。这里将这个概念推广到位于气动中心和后缘之间的中点，以适应与1/4弦长点差异较大的气动中心位置。本文档中保留 :math:`3/4` 的符号。

- :math:`\omega`：翼型截面的旋转速度（俯仰/扭转速率），绕z轴正方向。

- :math:`\boldsymbol{v}_{ac}`：气动中心处的速度矢量 :math:`\boldsymbol{v}_{ac}=[v_{x,ac}, v_{y,ac}]`（坐标假设在翼型截面坐标系中表示）。

- :math:`\boldsymbol{v}_{34}`：3/4弦长点处的速度矢量 :math:`\boldsymbol{v}_{34}=[v_{x,34}, v_{y,34}]`（坐标假设在翼型截面坐标系中表示）。该速度由1/4弦长点处的速度和截面的旋转速度得到：
  :math:`\boldsymbol{v}_{34}=\boldsymbol{v}_{ac}+\omega d_{34} \hat{\boldsymbol{x}}_s`
  其中 :math:`d_{34}` 是气动中心和3/4弦长点之间的距离。

- :math:`U_{ac}`：气动中心处的速度模长。
  :math:`U_{ac}=\lVert\boldsymbol{v}_{ac}\rVert=\sqrt{v_{x,ac}^2 + v_{y,ac}^2}`

- :math:`\alpha_{ac}`：气动中心处的攻角
  :math:`\alpha_{ac}=\operatorname{atan2}(v_{x,ac},v_{y,ac})`

- :math:`\alpha_{34}`：3/4弦长点处的攻角
  :math:`\alpha_{34}=\operatorname{atan2}(v_{x,34},v_{y,34})`

- :math:`\boldsymbol{x}`：连续公式使用的状态矢量。

- :math:`c`：翼型弦长。

- :math:`C_l^{st}, C_d^{st}, C_m^{st}`：静态翼型系数。

- :math:`\alpha_0`：零升力攻角，:math:`C_l^{st}(\alpha_0)=0`。

- :math:`\alpha_1`：接近正失速的攻角。
- :math:`\alpha_2`：接近负失速的攻角。

- :math:`C_{l,\alpha}`：定常升力曲线在 :math:`\alpha_0` 附近的斜率。

- :math:`f^{st}_s(\alpha)`：定常分离函数，由升力曲线 :math:`C_l^{st}(\alpha)` 确定（见下文，例如 :cite:`ad-Hansen:2004`）。

- :math:`A_1`, :math:`A_2`, :math:`b_1`, :math:`b_2`：四个常数，是尾涡传播的特征（瓦格纳常数）。

**时间常数：**

- :math:`T_u(t) = \frac{c}{2U_{ac}(t)} \in [0.001, 50]`：流过多半个翼型截面的时间。该值被限定在范围内以避免非物理值。
- :math:`T_{f,0}`：与前缘分离相关的无量纲时间常数。默认值为3。
- :math:`T_{p,0}`：边界层、前缘压力梯度的无量纲时间常数。默认值为1.7。

**分离函数：**

定常分离函数 :math:`f_s^{st}` 定义为势流基尔霍夫流中平板上的分离点 :cite:`ad-Hansen:2004`：

.. math::

   \begin{aligned}
   \text{接近$\alpha_0$时},
   f_s^{st}(\alpha) &= \operatorname{min}\left\{\left[2 \sqrt{ \frac{C_l^{st}(\alpha)}{C_{l,\alpha}(\alpha-\alpha_0) } } -1 \right]^2 , 1 \right\}
   ,\quad
   \text{远离$\alpha_0$时},
   f_s^{st}(\alpha)=0
   \end{aligned}

当 :math:`\alpha=\alpha_0` 时，:math:`f_s^{st}(\alpha_0)=1`。远离 :math:`\alpha_0` 时，函数逐渐下降到0。一旦函数在 :math:`\alpha_0` 两侧都达到0，则 :math:`f_s^{st}` 保持为常数0。

**注意，对于UAMod=5，使用不同的分离函数。**
我们为 :math:`C_n` 函数定义一个偏移量 ``cn_offset``，其中 :math:`C_{n,offset}=\frac{C_n\left(\alpha^{Lower}\right)+C_n\left(\alpha^{Upper}\right)}{2}`。然后，分离函数是一个介于0和1之间的值，由以下公式给出：

.. math::

   f_s^{st}(\alpha) = \left[ 2 \max\left\{\frac{1}{4} , \sqrt{\frac{C_n^{st}(\alpha) - C_{n,offset}}{C_n^{fullyAttached}(\alpha)-C_{n,offset}}}\right\} -1 \right]^2

其中完全附着的 :math:`C_n` 曲线定义为：在 :math:`alpha^{Lower}` 和 :math:`alpha^{Upper}` 之间为 :math:`C_n`，在该范围之外为线性函数：

.. math::

   C_n^{fullyAttached}(\alpha) =  \begin{cases} C_n\left(\alpha^{Upper}\right) + C_n^{slope}\left(\alpha^{Upper}\right) \cdot \left(\alpha-\alpha^{Upper}\right)          & \alpha>\alpha^{Upper} \\
                                                        C_n(\alpha)                                                                                                       & \alpha^{Lower}<=\alpha<=\alpha^{Upper} \\
                                                        C_n\left(\alpha^{Lower}\right) + C_n^{slope}\left(\alpha^{Lower}\right) \cdot  \left(\alpha-\alpha^{Lower}\right) & \alpha<\alpha^{Lower} \end{cases}

注意，为了避免在 :math:`\pm180` 度边界处的数值问题，当分离函数在 :math:`alpha^{Upper}` 以上和 :math:`alpha^{Lower}` 以下为0时，该函数会改变斜率。这使得完全附着的线性部分是周期性的，避免了大攻角数值的数值问题。

**无粘和完全分离升力系数：**

无粘升力系数为 :math:`C_{l,\text{inv}}= C_{l,\alpha} (\alpha-\alpha_0)`。
完全分离升力系数可以用不同方式建模（:cite:`ad-Branlard:book`）。在大多数工程模型中，完全分离升力系数在 :math:`\alpha_0` 附近的斜率为 :math:`C_{l,\alpha}/2`。在非定常空气动力学子模块中，完全分离升力系数由定常分离函数导出：

.. math::

   \begin{aligned}
      C_{l,\text{fs}}(\alpha) = \frac{C_l^{st}(\alpha) - C_{l,\alpha}(\alpha-\alpha_0)f_s^{st}(\alpha)}{1-f_s^{st}(\alpha)}
     \text{当$f_s^{st}\neq 1$时}
      , \qquad
      C_{l,\text{fs}}(\alpha) =\frac{C_l^{st}(\alpha)}{2}
    \text{当$f_s^{st}=1$时}\end{aligned}


Beddoes-Leishman类型模型 (UAMod=2,3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Beddoes-Leishman模型用于解释附着流和后缘失速 :cite:`ad-LeishmanBeddoes:1989`。

非定常空气动力学模块中实现了两个变体。这两个（可压缩）模型目前在以下参考文献中描述：:cite:`ad-AeroDyn:manualUnsteady`。模型使用 :math:`C_n` 和 :math:`C_c` 作为主要物理量。模型使用离散状态，不能用于线性化。


Beddoes-Leishman 4状态模型 (UAMod=4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OpenFAST中实现的4状态（不可压缩）动态失速模型在 :cite:`ad-Branlard:2022` 中描述（该模型与Hansen-Gaunaa-Madsen（HGM）的原始公式略有不同 :cite:`ad-Hansen:2004`）。
使用 ``UAMod=4`` 启用该模型。模型使用 :math:`C_l` 作为主要物理量。
模型支持线性化。

注意：在AeroDyn-UA中实现刚性积分器之前，该模型可能需要更小的时间步长。

**状态方程：**
模型的状态方程为：

.. math::

   \begin{aligned}
       \dot{x}_1 &= - T_u^{-1}  b_1\, x_1  +  T_u^{-1} b_1 A_1  \alpha_{34}\nonumber \\
       \dot{x}_2 &= - T_u^{-1}  b_2\, x_2  +  T_u^{-1} b_2 A_2  \alpha_{34}\nonumber \\
       \dot{x}_3 &= - T_p^{-1} x_3  +  T_p^{-1} C_l^p                \nonumber \\
       \dot{x}_4 &= - T_f^{-1} x_4  +  T_f^{-1} f_s^{st}(\alpha_F)      ,\qquad x_4 \in[0,1]
       \nonumber
   \end{aligned}

其中：

.. math::

   \begin{aligned}
    \alpha_E(t) & =\alpha_{34}(t)(1-A_1-A_2)+ x_1(t) + x_2(t)                                      \nonumber \\
    C_{L}^p(t)  & =C_{l,\alpha} \, \left(\alpha_E(t)-\alpha_0\right) + \pi T_u(t) \omega(t) \nonumber \\
    \alpha_F(t) & =\frac{x_3(t)}{C_{l,\alpha}}+\alpha_0                                     \nonumber
    \end{aligned}


**输出方程：**
非定常翼型系数 :math:`C_{l,\text{dyn}}`, :math:`C_{d,\text{dyn}}`, :math:`C_{m,\text{dyn}}` 从状态中获得如下：

.. math::

   \begin{aligned}
       C_{l,\text{dyn}}(t) &= C_{l,\text{circ}} + \pi T_u \omega   \\
      C_{d,\text{dyn}}(t)  &=  C_d(\alpha_E) + \left[(\alpha_{ac}-\alpha_E) +T_u \omega \right]C_{l,\text{circ}} + \left[ C_d(\alpha_E)-C_d(\alpha_0)\right ] \Delta C_{d,f}'' \\
       C_{m,\text{dyn}}(t) &=  C_m(\alpha_E) - \frac{\pi}{2} T_u \omega\\
   \end{aligned}

其中：

.. math::
   \begin{aligned}
       \Delta C_{d,f}'' &= \frac{\sqrt{f_s^{st}(\alpha_E)}-\sqrt{x_4}}{2} - \frac{f_s^{st}(\alpha_E)-x_4}{4}
   ,\qquad
       x_4\ge 0  \\
    C_{l,\text{circ}}&= x_4 (\alpha_E-\alpha_0) C_{l,\alpha} +  (1-x_4) C_{l,{\text{fs}}}(\alpha_E)
   \end{aligned}


Beddoes-Leishman 5状态模型 (UAMod=5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
5状态（不可压缩）动态失速模型与Beddoes-Leishman 4状态模型（UAMod=4）类似，但增加了第5个状态来表示涡生成。
使用 ``UAMod=5`` 启用该模型。模型使用 :math:`C_n` 和 :math:`C_c` 作为主要物理量。
模型支持线性化。


.. _ua_oye:

Oye模型 (UAMod=6)
~~~~~~~~~~~~~~~~~~~

Oye动态失速模型是一个单状态（连续）模型，在 :cite:`ad-Oye:1991` 中提出，并在例如 :cite:`ad-Branlard:book` 中描述。
该模型试图捕捉后缘失速。
模型支持线性化。

**状态方程：**
Oye动态失速模型使用一个状态 :math:`\boldsymbol{x}=[f_s]`，其中 :math:`f_s` 是非定常分离函数。
状态方程是一阶微分方程：

.. math::

   \begin{aligned}
     \frac{df_s(t)}{dt} =- \frac{1}{T_f} f_s(t)  + \frac{1}{T_f} f_s^{st}(\alpha_{34}(t))
    \end{aligned}

其中 :math:`T_f=T_{f,0} T_u` 是流动分离的时间常数，:math:`f_s^{st}` 是在 :numref:`ua_notations` 中描述的定常分离函数。
:math:`T_{f,0}` 的值通常选择为6左右（与默认值不同）。
很容易看出，当系统处于定常状态时（即 :math:`\frac{df_s(t)}{dt}=0` 时），:math:`f_s` 达到 :math:`f_s^{st}` 的值。

**输出方程：**
非定常升力系数计算为无粘升力系数 :math:`C_{l, \text{inv}}` 和完全分离升力系数 :math:`C_{l,\text{fs}}` 的线性组合。这两个升力系数都由定常升力系数确定，定常升力系数通常以表格数据形式提供，记为 :math:`C_l^{st}(\alpha)`，其中上标 :math:`st` 代表“定常”。
非定常升力系数建模为：

.. math::

   \begin{aligned}
       C_{l,\text{dyn}}(\alpha_{34} ,t) = f_s(t)\; C_{l,\text{inv}}(\alpha_{34}) + (1-f_s(t))\; C_{l,\text{fs}}(\alpha_{34})
       \end{aligned}

其中 :math:`\alpha_{34}` 是3/4弦长点处的瞬时攻角。
:math:`f_s` 可以看作是两种流动状态之间的松弛因子。


Boeing-Vertol模型 (UAMod=7)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Boeing-Vertol模型在以下论文中提到 :cite:`ad-Murray:2011`。该参考文献中省略了模型的细节，因此这里的文档参考了涡旋代码CACTUS中的实现，该实现在AeroDyn中被几乎完全复制。该模型不支持线性化。

:cite:`ad-Murray:2011` 中提出的模型是一个仅输出模型，其中动态攻角使用准定常攻角和攻角变化率确定：

.. math::

   \alpha_{dyn} = \alpha_{34} - k_1 \gamma \sqrt{\left| \dot{\alpha} T_u\right|}

其中 :math:`k_1` 和 :math:`\gamma` 是模型的常数。实际上，升力和阻力系数的实现不同，正失速和负失速的实现也不同。模型需要一个离散状态来计算攻角的变化率，还需要两个离散状态来跟踪模型是否被激活。

**翼型常数：**

攻角正负变化率对应的常数 :math:`k_1` 设置为：

.. math::

   k_{1,p}= 1 ,\quad k_{1,n} = 1/2

过渡区域的范围计算为：

.. math::

   \Delta \alpha_\text{max} = \frac{0.9 \operatorname{min}\left(|\alpha_1-\alpha_0|, |\alpha_2-\alpha_0|\right)}{\operatorname{max}(k_{1,p},k_{1,n})}

其中 :math:`\alpha_1` 和 :math:`\alpha_2` 分别是正失速和负失速时的攻角（取自翼型输入文件中的值）。
系数0.9是一个余量，用于防止失速期间有效攻角达到 :math:`\alpha_0`。

**中间变量：**

升力和阻力的变量 :math:`\gamma` 计算为翼型厚度弦长比 :math:`t_c` 和马赫数 :math:`M_a` 的函数（当前实现中假设为0）：

.. math::

   \begin{aligned}
     \gamma_L &= (1.4-6\delta)\left[1-\frac{\text{Ma}-(0.4+5\delta)}{0.9+2.5\delta-(0.4+5\delta)}\right] &&\\
     \gamma_D &= (1-2.5\delta) ,&&\text{当$\text{Ma} < 0.2$时}  \\
     \gamma_D &= (1-2.5\delta)\left[1-\frac{\text{Ma}-0.2}{(0.7+2.5\delta-0.2)}\right] ,&& \text{其他情况}
   \end{aligned}

其中 :math:`\delta = 0.06-t_c`。

**离散状态（和中间变量）的更新：**

攻角的变化率计算为：

.. math::

   \dot{\alpha} = \frac{\alpha_{34}(t+\Delta t) - \alpha_{34}(t)}{\Delta t}

引入了一个额外的状态来存储 :math:`\dot{\alpha}` 的值，以避免其突然跳跃。超过 :math:`\pi \Delta t` 一定比例的变化率会被替换为前一个时间步的值。CACTUS实现中没有这个特性。

升力和阻力的动态攻角偏移（滞后）计算为：

.. math::

   \begin{aligned}
       \Delta \alpha_L &= k_1 \operatorname{min} \left(\gamma_L \sqrt{\dot{|\alpha}T_u|} , \Delta \alpha_\text{max}\right)\\
       \Delta \alpha_D &= k_1 \operatorname{min}\left(\gamma_D \sqrt{\dot{|\alpha}T_u|}, \Delta \alpha_\text{max} \right)
   \end{aligned}

如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0)<0`，则 :math:`k_1` 的值取 :math:`k_{1,n}`，否则取 :math:`k_{1,p}`。
升力和阻力的滞后攻角为：

.. math::

   \begin{aligned}
       \alpha_{\text{Lag},L} &= \alpha_{34} - \Delta \alpha_L\operatorname{sign}(\dot{\alpha}) \\
       \alpha_{\text{Lag},D} &= \alpha_{34} - \Delta \alpha_D\operatorname{sign}(\dot{\alpha})
   \end{aligned}

到正失速和负失速的距离计算如下。
如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0)<0` 且动态失速处于激活状态：

.. math::

           \Delta_n = \alpha_2  - \alpha_{\text{Lag},D} , \quad \Delta_p = \alpha_{\text{Lag},D} - \alpha_1

如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0)<0` 且动态失速未激活：

.. math::

           \Delta_n = 0 , \quad \Delta_p = 0

如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0)\ge0`：

.. math::

       \Delta_n = \alpha_2 - \alpha_{34}, \qquad
       \Delta_p = \alpha_{34} - \alpha_1

升力系数的有效攻角取滞后攻角：

.. math::

    \begin{aligned}
       \alpha_{e,L}  &= \alpha_{\text{Lag},L}
   \end{aligned}

阻力系数的有效攻角由滞后攻角和到失速的差值得到：

.. math::

    \begin{aligned}
       \alpha_{e,D}  &= \alpha_{\text{Lag},D},                                                &&\text{当$\Delta_n>T$ 或 $\Delta_p > T$时} \\
       \alpha_{e,D}  &= \alpha_{34}+(\alpha_{\text{Lag},D}-\alpha_{34}) \frac{\Delta_n}{T}  , &&\text{当$\Delta_n>0$ 且 $\Delta_n < T$时} \\
       \alpha_{e,D}  &= \alpha_{34}+(\alpha_{\text{Lag},D}-\alpha_{34}) \frac{\Delta_p}{T}  , &&\text{当$\Delta_p>0$ 且 $\Delta_p < T$时} \\
       \alpha_{e,D}  &= \alpha_{34} ,                                                         &&\text{其他情况}
   \end{aligned}

其中 :math:`T=2\Delta\alpha_\text{max}` 是“过渡”区域的范围。

如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0) \ge 0` 且攻角高于 :math:`\alpha_1` 或低于 :math:`\alpha_2`，则升力动态失速状态被激活。
如果 :math:`\dot{\alpha}(\alpha_{34}-\alpha_0) < 0` 且有效攻角低于 :math:`\alpha_1` 且高于 :math:`\alpha_2`，则状态关闭。

如果以下任一条件为真，则阻力动态失速状态被激活：

.. math::
    \begin{aligned}
        &\Delta_n > T \ \text{或 } \Delta_p > T \\
        &\Delta_n > 0 \ \text{且 } \Delta_n < T \\
        &\Delta_p > 0 \ \text{且 } \Delta_p < T
    \end{aligned}

否则状态关闭。

**输出计算：**
动态升力和阻力系数的计算如下：

.. math::
    \begin{aligned}
        C_{l,\text{dyn}}&=\frac{C_l^{st}(\alpha_{e,L})}{\alpha_{e,L}-\alpha_0} ,\quad  \text{当$C_l$的动态失速激活时}\  \\
        C_{l,\text{dyn}}&=C_l^{st}(\alpha_{34}) ,\quad\quad  \text{其他情况} \\
        C_{d,\text{dyn}}&=C_d^{st}(\alpha_{e,D})
    \end{aligned}

需要重新计算中间变量以获得 :math:`\alpha_{e,L}` 和 :math:`\alpha_{e,D}`。
力矩系数基于气动中心和中弦（"50"）处的值计算：

.. math::
       C_{m,\text{dyn}} = C_m^{st}(\alpha_{ac}) + \cos\alpha_{50}  \left[C_l^{st}(\alpha_{34}) - C_l^{st}(\alpha_{50})\right]/4

其中 :math:`\alpha_{50}` 的计算方式与 :math:`\alpha_{34}` 相同（使用气动中心处的速度和翼型的旋转速率），但使用气动中心到中弦的距离（参见 :numref:`ua_notations`）。


.. _UA_inputs:

输入
------

有关AeroDyn主文件中所需输入（例如 ``UAMod``）的描述，请参见 :numref:`ad_ua_inputs`。

有关翼型输入文件中所有输入的更全面描述，请参见 :numref:`airfoil_data_input_file`。
它们的默认值在 :numref:`UA_AFI_defaults` 中描述。

有关非定常空气动力学输入特有的符号和定义列表，请参见 :numref:`ua_notations`。

翼型数据示例（包含一些非定常空气动力学参数）可在此处获取：
:download:`(此处) <examples/ad_polar_example.dat>`。

非定常空气动力学驱动程序输入在 :numref:`ua_driver` 中记录。


.. _UA_AFI_defaults:

计算默认翼型系数
----------------------------------------

``cd0`` 的默认值是攻角在 :math:`\pm20` 度之间 :math:`C_d` 曲线的最小值。
:math:`\alpha_{c_{d0}}` 定义为出现 ``cd0`` 时的攻角。

计算完 ``cd0`` 后，:math:`C_n` 曲线计算为：

.. math::
       C_{n}(\alpha) = C_l(\alpha) \cos\alpha + \left(C_d(\alpha) - c_{d0}\right) \sin\alpha

:math:`C_n` 曲线的斜率计算如下：

.. math::
       C_{n}^{Slope}\left(\frac{\alpha_{i+1} + \alpha_i}{2}\right) = \frac{C_n(\alpha_{i+1}) - C_n(\alpha_i)}{\alpha_{i+1} - \alpha_i}

:math:`C_{n,smooth}^{Slope}` 是 :math:`C_{n}^{Slope}` 的平滑版本，使用窗口为2度的三权核计算。

.. math::
       C_{l}^{Slope}\left(\frac{\alpha_{i+1} + \alpha_i}{2}\right) = \frac{C_l(\alpha_{i+1}) - C_l(\alpha_i)}{\alpha_{i+1} - \alpha_i}

使用 :math:`C_{n,smooth}^{Slope}`，计算 ``alphaUpper`` 和 ``alphaLower``：

``alphaUpper`` 是 :math:`\alpha_{c_{d0}}` 和20度之间的最小攻角值，在该点 :math:`C_{n,smooth}^{Slope}` 曲线开始下降到其最大斜率的90%。

.. math::
       C_{n,smooth}^{Slope}\left(\alpha^{Upper}\right) < 0.9 \max_{\alpha \in \left[\alpha_{c_{d0}}, \alpha^{Upper}\right]}  C_{n,smooth}^{Slope}\left( \alpha \right)

``alphaLower`` 是-20度和 :math:`\alpha_{c_{d0}}` 之间的最大攻角值，在该点 :math:`C_{n,smooth}^{Slope}` 曲线开始下降到其最大斜率的90%。

.. math::
       C_{n,smooth}^{Slope}\left(\alpha^{Lower}\right) < 0.9 \max_{\alpha \in \left[\alpha^{Lower}, \alpha_{c_{d0}}\right]}  C_{n,smooth}^{Slope}\left( \alpha \right)

``Cn1`` 是 :math:`\alpha >= \alpha^{Upper}` 且分离函数 :math:`f_{st}(\alpha) = 0.7` 的最小 :math:`\alpha` 处的 :math:`C_n(\alpha)` 值。

``Cn2`` 是 :math:`\alpha <= \alpha^{Lower}` 且分离函数 :math:`f_{st}(\alpha) = 0.7` 的最大 :math:`\alpha` 处的 :math:`C_n(\alpha)` 值。

``Cn_offset`` 是 ``alphaUpper`` 和 ``alphaLower`` 处 :math:`C_n` 曲线的平均值：

.. math::
       C_{n}^{offset} = \frac{C_n\left(\alpha^{Lower}\right) + C_n\left(\alpha^{Upper}\right)}{2}

``C_nalpha`` 定义为攻角在 :math:`\pm20` 度之间平滑后的 :math:`C_n` 曲线 :math:`C_{n,smooth}^{Slope}` 的最大斜率。

``C_lalpha`` 定义为攻角在 :math:`\pm20` 度之间（未平滑）的 :math:`C_l` 曲线 :math:`C_{l}^{Slope}` 的最大斜率。

默认 ``alpha0`` 计算为斜率等于 ``C_lalpha`` 的直线的零点，该直线在 :math:`\alpha = \frac{\alpha^{Upper} + \alpha^{Lower}}{2}` 处穿过 :math:`C_l` 曲线：

.. math::
       \alpha_0 = \frac{\alpha^{Upper} + \alpha^{Lower}}{2} - \frac{C_l\left(\frac{\alpha^{Upper} + \alpha^{Lower}}{2}\right) }{C_{l,\alpha}}

``Cm0`` 是 ``alpha0`` 处 :math:`C_m` 曲线的值：:math:`C_{m,0} = C_m\left(\alpha_0\right)`。如果未包含 :math:`C_m` 极坐标值，则 :math:`C_{m,0} =0`。

``alpha1`` 是 ``alphaUpper`` 以上分离函数 :math:`f_s^{st}` 为0.7时的攻角。

``alpha2`` 是 ``alphaLower`` 以下分离函数 :math:`f_s^{st}` 为0.7时的攻角。

``Cn1`` 是 ``alpha1`` 处 :math:`C_n` 曲线的值。

``Cn2`` 是 ``alpha2`` 处 :math:`C_n` 曲线的值。


输出
-------

可以输出动态失速模型的变量，但需要设置预处理器变量 ``UA_OUTS`` 并重新编译程序（OpenFAST、AeroDyn驱动程序或非定常气动驱动程序）。
输出写入扩展名为 ``*.UA.out`` 的输出文件中。
要使用 ``cmake`` 激活这些输出，请使用 ``-DCMAKE_Fortran_FLAGS="-DUA_OUTS=ON"`` 进行编译。

使用驱动程序时，不需要使用此预处理器变量。


.. _ua_aeroelasttheory:

二维截面的气动弹性仿真
--------------------------------------

可以使用驱动程序对孤立的二维截面进行气动弹性仿真，以便在简化的环境中使用非定常空气动力学模型。
参见 :numref:`ua_driver`。
气动弹性仿真的理论和描述可在 :cite:`ad-UAElast:torquepaper` 中找到。


.. _ua_driver:

驱动程序
------

有一个驱动程序可用于运行单个翼型的仿真。

可以进行不同类型的仿真：

 - 使用攻角的正弦变化，
 - 用户定义的攻角、相对风速和俯仰速率的时间序列，
 - 具有3个自由度的气动弹性仿真，用于截面在其二维平面内的弹性运动（flap、edge和扭转），可以指定风速的时间序列，或指定截面的运动。

气动弹性仿真的理论和描述可在 :cite:`ad-UAElast:torquepaper` 中找到。


编译
~~~~~~~~~~~

使用 ``cmake`` 时，使用 ``make unsteadyaero_driver`` 编译驱动程序，生成的可执行文件位于 ``aerodyn`` 文件夹中。


驱动程序输入
~~~~~~~~~~~~~

非定常空气动力学驱动程序的输入文件示例可在 `r-test仓库 <https://github.com/OpenFAST/r-test/blob/main/modules/unsteadyaero/ua_redfreq/UA2.dvr>`__ 中找到。

下面描述不同的输入。

**环境条件**

``FldDens``：工作流体密度（kg/m^3）

``KinVisc``：工作流体运动粘度（m^2/s）

``SpdSound``：工作流体中的声速（m/s）

**非定常空气动力学选项**

``UAMod``：非定常气动模型开关（切换）{2=B-L Gonzalez，3=B-L Minnema/Pierce，4=B-L HGM 4状态，5=B-L 5状态，6=Oye，7=Boeing-Vertol} [仅在AFAeroMod=2时使用]

``FLookup``：标志，指示是否计算f'的查找表（TRUE），或是否使用最佳拟合指数方程（FALSE）；如果为FALSE，必须在翼型输入文件中提供S1-S4（标志）[仅在AFAeroMod=2时使用]

**翼型特性**

``AirFoil``：翼型表（第1列：攻角（AoA），第2列：升力系数，第3列：阻力系数）。

``Chord``：弦长（m）

``Vec_AQ``：在翼型坐标系中，从参考点"A"到气动中心（约1/4弦长）"Q"的矢量，单位为弦长。如果"A"在中弦处，值可能为(0, -0.25) (-)

``Vec_AT``：在翼型坐标系中，从参考点"A"到3/4弦长点"T"的矢量，单位为弦长。如果"A"在中弦处，值可能为(0, 0.25) (-)

``UseCm``：使用翼型表中的Cm（力矩系数）数据 {true/false}

**仿真控制**

``SimMod``：仿真模型 {1=减缩频率模型，2=指定气动时间序列，3=弹性截面}

**减缩频率仿真**（``SimMod=1``）

``InflowVel``：来流速度（m/s）

``NCycles``：整个仿真期间攻角振荡的次数 (-)

``StepsPerCycle``：每个周期的时间步数 (-)

``Frequency``：翼型振荡的频率（Hz）

``Amplitude``：攻角振荡的幅值（度）

``Mean``：周期平均值（度）

``Phase``：初始相位（步数）。

**指定气动仿真输入**（``SimMod=2``）

``TMax_PA``：总运行时间（s）

``DT_PA``：推荐的模块时间步长（s）

``AeroTSFile``：分隔输入文件（例如csv）中的时间序列数据，包含1个标题行，4列：时间（s）、攻角（度）、来流速度（m/s）、俯仰速率（rad/s）

**气动弹性仿真**（``SimMod=3``）

气动弹性仿真的理论可在 :numref:`ua_aeroelasttheory` 中找到。

``TMax``：总运行时间（s）

``DT``：时间步长（s）。

``ActiveDOF``：激活的自由度列表（true或false）

``InitPos``：弹性自由度的初始位置列表（m、m和rad）

``InitVel``：弹性自由度的初始速度列表（m/s、m/s和rad/s）

``GFScalingL1``：广义力缩放因子，用于将截面载荷转换为广义载荷（3x3）。每行三个值。

``MassMatrixL1``：质量矩阵（3x3）。每行三个值。

``DampMatrixL1``：阻尼矩阵（3x3）。每行三个值。

``StifMatrixL1``：刚度矩阵（3x3）。每行三个值。

``Twist``：扭转自由度为零时截面的固定扭转（度）

``InflowMod``：来流速度模型。{1：恒定速度，2：时间序列}

``Inflow``：x和y方向的来流速度 [仅在InflowMod=1时使用]

``InflowTSFile``：来流速度输入文件。分隔文件（例如csv），包含一个标题行，三列：时间（s）、Ux（m/s）、Uy（m/s）。[仅在InflowMod=2时使用]

``MotionMod``：自由度运动模型 {1：动态，2：指定}

``MotionTSFile``：指定运动的输入文件。分隔文件（例如csv），包含一个标题行，10列：时间（s）、x（m）、y（m）、th（rad）、速度和加速度。[仅在InflowMod=2时使用]

**输出控制**

``SumPrint``：写入非定常空气动力学摘要文件（标志）

``WrAFITables``：回写内部使用的空气动力学系数（标志）

**CSV输入文件示例**

非定常气动驱动程序现在使用CSV文件作为输入时间序列。
时间列不需要是恒定时间步长，但需要是单调递增的。

指定气动输入（``SimMod=2``）：

.. code::

    Time_[s] , Alpha_[deg] , VRel_[m/s] , omega_[rad/s]
    0.0      , 0           , 10         , 0
    0.01     , 0           , 10         , 0

来流文件输入（``SimMod=3``，``InflowMod=2``）：

.. code::

    Time_[s]  , Ux_[m/s], Uy_[m/s]
    0.0       , 1       , 10
    1.0       , 2       , 10
    5.0       , 2       ,  8
    10.0      , 1       , 12

运动文件输入（``SimMod=3``，``MotionMod=2``）（注意在这个示例中没有提供速度和加速度，但最好提供）：

.. code::

    Time_[s] , x_[m] , y_[m] , th_[rad] , xd_[m/s] , yd_[m/s] , thd_[rad/s] , xdd_[m/s^2] , ydd_[m/s^2] , thdd_[rad/s^2]
    0.0        , 1     , 1     , 1        , 0        , 0        , 0           , 0           , 0           , 0
    1.0        , 2     , 2     , 2        , 0        , 0        , 0           , 0           , 0           , 0
    5.0        , 2     , 2     , 2        , 0        , 0        , 0           , 0           , 0           , 0
    10.0       , 1     , 1     , 1        , 0        , 0        , 0           , 0           , 0           , 0
