.. _ad_coordsys:

坐标系
==================

AeroDyn 在内部计算和输出中使用不同的坐标系。输出通道通常带有与所使用坐标系对应的字母后缀：

* (i)：惯性系
* (h)：轮毂系
* (p)：极坐标系
* (l)：局部极坐标系
* 无后缀或 (w)：旧版输出系统
* (n-t)：旧版翼型系

不同的系统如下所述。

惯性系 (i)
-------------------

惯性系 :math:`(i)` 是 OpenFAST 使用的全局坐标系（参见 ElastoDyn 文档）。

轮毂系 (h)
--------------

轮毂系 :math:`(h)` 基于轴方位角位置 :math:`\psi` 沿 :math:`x_h` 轴旋转（参见 ElastoDyn 文档）。

极坐标系 (p)
----------------

极坐标系 :math:`(p_k)` 由轮毂坐标系 :math:`(h)` 构造，将 :math:`x_h` 轴旋转每个叶片的方位偏移量 :math:`\psi_{0,k}`（叶片沿方位均匀分布）。为简洁起见，我们将该系统称为 :math:`(p)` 系统。如果叶片数量为 :math:`n_B`，则叶片 :math:`k` 的方位偏移量为：

.. math::
   \begin{aligned}
      \psi_{0,k} = 2 \pi \frac{k-1}{n_B}
   \end{aligned}

对于叶片 1，:math:`\psi_{0,1}=0`，因此 :math:`y_{p,1}=y_h` 且 :math:`z_{p,1}=z_h`。

:math:`x_{p,k}` 轴沿轮毂 x 轴方向。

在没有锥角的情况下，:math:`z_{p,k}` 轴对应于叶片的 :math:`z_{b,k}` 轴。

锥坐标系 (c)
---------------

锥坐标系 :math:`(c)` 由极坐标系绕每个叶片 :math:`k` 的 :math:`y_{p,k}` 轴旋转得到。更多细节参见 ElastoDyn/FAST8 文档。AeroDyn 使用该系统来估计叶片变桨角（通过比较 :math:`(c)` 和 :math:`(b)` 系统）。

叶片坐标系 (b)
----------------

叶片坐标系 :math:`(b)` 由锥坐标系绕 :math:`z_c` 轴旋转（变桨）得到。它用于定义气动中心的位置以及沿叶片展向的局部扭转。参见 :numref:`blade_data_input_file` 和 :numref:`ad_blade_geom`。

翼型坐标系 (a)
------------------

**目前用户在输入文件中指定预弯方向，用于确定翼型的方向（即 :math:`z_a` 轴）。未来，该方向可能会从气动中心线自动计算。**

翼型截面坐标系 :math:`(a_{_{kj}})`，或简称为 :math:`(a)`，是应用叶素理论的坐标系，也是提供翼型形状和极曲线数据的坐标系。翼型坐标系 :math:`(a_{_{kj}})` 为每个叶片 :math:`k` 和每个叶片节点 :math:`j` 定义。:math:`y_a` 轴沿翼型弦长（弦向），指向后缘。:math:`x_a` 轴垂直于弦长，指向吸力面（对于非对称翼型）。参见 :numref:`ad_cs_airfoil`。

.. _ad_cs_airfoil:

.. figure:: figs/FASTAirfoilSystem.svg
   :width: 80%
   :align: center

   翼型 (a) 坐标系。

:math:`(a)` 系统是应用叶素理论（BET）的坐标系：攻角 :math:`\alpha`、升力 :math:`L` 和阻力 :math:`D` 都在 :math:`x_a-y_a` 平面内定义。升力线载荷在该系统中计算。该系统中的相对风是 3D 相对风在 :math:`(a)` 系统 2D 平面上的投影，记为 :math:`{}^{\perp_a}\boldsymbol{V}_\text{rel}` 或 :math:`\boldsymbol{V}_\text{rel,a}`。

在翼型坐标系中，我们有：

.. math::
   \begin{aligned}
      C_{x_a}  =  C_l(\alpha) \cos\alpha + C_d(\alpha)\sin\alpha % that's Cn
      ,\quad
      C_{y_a}  = -C_l(\alpha) \sin\alpha + C_d(\alpha)\cos\alpha % that's -Ct for the t of AeroDyn
      ,\quad
      C_{m_a} = C_m(\alpha)
     \end{aligned}

载荷（单位长度）为：

.. math::
   \begin{aligned}
     f_{x_a} = \frac{1}{2}\rho V_{\text{rel},a}^2 c C_{x_a}
     ,\quad
     f_{y_a} = \frac{1}{2}\rho V_{\text{rel},a}^2 c C_{y_a}
     ,\quad
     m_{z_a} = \frac{1}{2}\rho V_{\text{rel},a}^2 c^2 C_{m_a}
     \end{aligned}

旧版 (n-t) 系统
-------------------

在旧版 AeroDyn 代码和文档中，有时会使用 :math:`(n-t)` 系统。:math:`n` 轴对应于 :math:`x_a` 轴（垂直于弦长）。:math:`t` 轴对应于 :math:`-y_a` 轴（相反方向）。

局部极坐标系 (l)
----------------------

**目前局部极坐标系仅用于输出目的。未来版本将在 BEM 实现中使用它。**

局部极坐标系 :math:`(l_{_{kj}})`，或简称为 :math:`(l)`，与极坐标系类似，但绕 :math:`x_h` 轴旋转，使得 :math:`z_{l,kj}` 轴穿过叶片 :math:`k` 节点 :math:`j` 的变形后位置。

:math:`x_l` 沿轮毂 :math:`x_h` 轴方向，
:math:`z_l` 是垂直于轴平面内的径向坐标。
该坐标系在 :numref:`ad_cs_localpolar` 中说明，左侧为仅有预弯的情况，右侧为仅有后掠的情况。

.. _ad_cs_localpolar:

.. figure:: figs/FASTLocalPolarSystem.svg
   :width: 70%
   :align: center

   极坐标 (p) 和局部极坐标 (l) 坐标系。
   左：纯预弯。
   右：纯后掠。

局部极坐标系为每个叶片节点定义如下。变形后叶片节点 :math:`A_j` 相对于轮毂 :math:`H` 的位置为：

   .. math::

      \begin{aligned}
         \boldsymbol{r}_{HA_j} = \boldsymbol{r}_{A_j}-\boldsymbol{r}_H
      \end{aligned}

该向量投影到转子平面如下：

   .. math::

      \begin{aligned}
         \boldsymbol{r}_{HA_j}^\perp = \mathop{\mathrm{\boldsymbol{\mathrm{P}}}}_{\boldsymbol{\hat{x}}_h}(\boldsymbol{r}_{HA_j}) = \boldsymbol{r}_{HA_j} - (\boldsymbol{\hat{x}}_h \cdot {\boldsymbol{r}_{HA_j}}) \boldsymbol{\hat{x}}_h
      \end{aligned}

然后局部极坐标系的向量定义为：

   .. math::

      \begin{aligned}
         \boldsymbol{\hat{x}}_{l} = \boldsymbol{\hat{x}}_h
         ,\quad
         \boldsymbol{\hat{z}}_{l} = \frac{ \boldsymbol{r}_{HA_j}^\perp }{\lVert\boldsymbol{r}_{HA_j}\rVert}
         ,\quad
         \boldsymbol{\hat{y}}_{l} = \boldsymbol{\hat{z}}_h \times \boldsymbol{\hat{x}}_h
       \end{aligned}

不同叶片节点的局部极坐标系之间的差异在于绕 :math:`x_h` 轴的方位旋转（以及沿 :math:`x_h` 轴的原点平移，这通常无关紧要）。

旧版局部输出系统 (w)
------------------------------

**AeroDyn 中标记为“x”或“y”且没有其他字母定义坐标系的输出目前在旧版输出系统中提供。**（例如 :math:`F_x`、:math:`V_x` 或 :math:`V_{dis,y}`）。

我们将 OpenFAST 的旧版输出系统记为 :math:`(w)`。旧版输出系统之前已在图 :numref:`ad_blade_local_cs` 中记录。该图还显示了局部角度和力分量的方向。在该图中，:math:`x` 应理解为 :math:`x_w`，:math:`y` 应理解为 :math:`y_w`。如果没有预弯、预锥或后掠，该图基本有效。

.. _ad_blade_local_cs:

.. figure:: figs/aerodyn_blade_local_cs.png
   :width: 80%
   :align: center
   :alt: aerodyn_blade_local_cs.png

   AeroDyn 旧版局部输出坐标系（从叶根看向叶尖）—— l：升力，d：阻力，m：俯仰，x：法向（平面外），
   y：切向（平面内），n：法向（弦向），
   t：切向（弦向）

:math:`(w_{kj})` 为每个叶片 :math:`k` 和节点 :math:`j` 定义，简称为 :math:`(w)`。
:math:`(w)` 系统是翼型系统的变换，使得该系统相对于锥坐标系没有绕 :math:`x`（后掠）或 :math:`z`（变桨/扭转）轴的旋转。

   - 该系统的 :math:`y_w` 轴（平面内）垂直于变桨轴，忽略后掠和面内变形。

   -  :math:`x_w` 轴（平面外）垂直于变形后的叶片，包括预弯和面外变形。

   -  :math:`z_w` 轴（径向）与变形后的叶片相切，包括预弯和面外变形。

AeroDyn 中该系统的构造如下。首先，使用以下子步骤和矩阵定义锥坐标系 :math:`(c)`（位于叶根，带锥角，但未变桨）：

   -  :math:`\boldsymbol{R}_{bi}`：从惯性系到叶根（叶根变桨角度为 :math:`\theta_p`）。

   -  :math:`\boldsymbol{R}_{hi}`：从惯性系到轮毂。

   -  :math:`\boldsymbol{R}_{bh} = \boldsymbol{R}_{bi} \boldsymbol{R}_{hi}^t=\mathop{\mathrm{Euler}}(\theta_1, \theta_2, -\theta_p)`：
      从轮毂到叶片。:math:`\boldsymbol{R}_{bh}` 的第三个欧拉角是变桨角 :math:`\theta_p` 的相反数（风力机使用绕 :math:`z` 轴的变桨和扭转的负约定）。将该欧拉角设置为零，并根据前两个角度构造变换矩阵，得到：

   -  :math:`\boldsymbol{R}_{ch}=\mathop{\mathrm{Euler}}(\theta_1, \theta_2,0)`：
      从轮毂到锥坐标系。

   -  :math:`\boldsymbol{R}_{ci}=\boldsymbol{R}_{ch} \boldsymbol{R}_{hi}`：
      从惯性系到锥坐标系。

然后，为每个翼型截面定义 :math:`(w)` 系统：

   -  :math:`\boldsymbol{R}_{ai}`：从惯性系到叶片翼型截面（包括弹性运动）

   - 从锥坐标系到叶片翼型截面：

      .. math::

         \begin{aligned}
                     \boldsymbol{R}_{ac}=\boldsymbol{R}_{ai}\boldsymbol{R}_{ci}^t=\mathop{\mathrm{Euler}}({}^w\!\tau,{}^w\!\kappa,-{}^w\!\beta)
                       \label{eq:R_acBetaFull}
           \end{aligned}

      其中 :math:`{}^w\!\beta` 包含全扭转（气动、弹性和变桨），:math:`{}^w\!\tau` 是 toe 角（但未使用），:math:`{}^w\!\kappa` 是 cant 角（存储为 ``Curve``）。我们使用上标 :math:`w` 因为这些角度是作为 :math:`(w)` 系统的一部分定义的。

   -  :math:`\boldsymbol{R}_{wc}=\mathop{\mathrm{Euler}}(0,{}^w\!\kappa,0)`：
      从锥坐标系到 :math:`w` 系统。:math:`(w)` 系统仅保留绕 :math:`y_c` 的旋转（≈预弯），从而忽略绕 :math:`x`（后掠）和 :math:`z`（≈扭转+变桨）的旋转。

   -  :math:`\boldsymbol{R}_{wi}=\boldsymbol{R}_{wc}\boldsymbol{R}_{ci}`：
      从惯性系到 :math:`w` 系统

塔筒坐标系
------------

局部塔筒坐标系如 :numref:`ad_tower_geom` 所示。
