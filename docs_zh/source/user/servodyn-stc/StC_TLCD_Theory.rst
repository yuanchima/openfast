.. _TLCD_Theory:

TLCD：运动方程推导
=======================================

..  _TLCDfig:

.. figure:: Schematics/TLCD_Diagram.png
   :alt: TLCD 示意图
   :width: 100%
   :align: center

   TLCD 设计示意图。

定义：
------------

.. container::
   :name: tab:TLCDdefs

   .. table:: TLCD 定义

      +-----------------+-------------------+
      | 变量            | 描述              |
      +=================+===================+
      |  |O_eq|         |  |O_desc|         |
      +-----------------+-------------------+
      |  |P_eq|         |  |P_desc|         |
      +-----------------+-------------------+
      |  |W_R_eq|       |  |W_R_desc|       |
      +-----------------+-------------------+
      |  |W_L_eq|       |  |W_L_desc|       |
      +-----------------+-------------------+
      |  |G_eq|         |  |G_desc|         |
      +-----------------+-------------------+
      |  |N_eq|         |  |N_desc|         |
      +-----------------+-------------------+
      |  |w_eq|         |  |w_desc|         |
      +-----------------+-------------------+
      |  |g_eq|         |  |g_desc|         |
      +-----------------+-------------------+

.. |O_eq|            replace:: :math:`O`
.. |O_desc|          replace:: 全局惯性参考系原点，位于静止风力机底部中心
.. |P_eq|            replace:: :math:`P`
.. |P_desc|          replace:: 局部参考系原点（例如固定于机舱），位于水平液柱中心
.. |W_R_eq|          replace:: :math:`W_R`
.. |W_R_desc|        replace:: 连接在右侧液柱顶部中心的点（运动）
.. |W_L_eq|          replace:: :math:`W_L`
.. |W_L_desc|        replace:: 连接在左侧液柱顶部中心的点（运动）
.. |G_eq|            replace:: :math:`i`
.. |G_desc|          replace:: 惯性参考系的轴方向（全局）
.. |N_eq|            replace:: :math:`l`
.. |N_desc|          replace:: 局部参考系的轴方向
.. |w_eq|            replace:: :math:`w`
.. |w_desc|          replace:: 液体水柱的位置，如 :numref:`TLCDfig` 中定义
.. |g_eq|            replace:: :math:`g`
.. |g_desc|          replace:: 惯性参考系（全局）中的重力向量


右侧竖直液柱
----------------------------

从右侧竖直柱开始，定义以下向量表达式：

.. container::
   :name: tab:TLCD_r_vectors

      +--------------------+-----------------------+
      | 变量               | 描述                  |
      +====================+=======================+
      |  |iVecR_O2P_eq|    |  |iVecR_O2P_desc|     |
      +--------------------+-----------------------+
      |  |lVecR_P2Wr_eq|   |  |lVecR_P2Wr_desc|    |
      +--------------------+-----------------------+
      |  |iVecW_l_eq|      |  |iVecW_l_desc|       |
      +--------------------+-----------------------+
      |  |iVecR_O2Wr_eq|   |  |iVecR_O2Wr_desc|    |
      +--------------------+-----------------------+

.. |iVecR_O2P_eq|       replace:: :math:`\vec{r}_{i}^{O \rightarrow P} = \left[ \begin{array}{c} x \\ y \\ z \end{array} \right]_{i} ^{O \rightarrow P}`
.. |iVecR_O2P_desc|     replace:: 从点 :math:`O` 到点 :math:`P` 在惯性坐标系中的位置向量
.. |lVecR_P2Wr_eq|      replace:: :math:`\vec{r}_{l}^{P \rightarrow W_R} = \left[ \begin{array}{c} x \\ y \\ z \end{array} \right]_{l} ^{P \rightarrow W_R}`
.. |lVecR_P2Wr_desc|    replace:: 从点 :math:`P` 到点 :math:`W_R` 在局部坐标系中的位置向量
.. |iVecW_l_eq|         replace:: :math:`\vec{\omega}_{i}^{l} = \left[ \begin{array}{c} \theta \\ \phi \\ \psi \end{array} \right]_{i} ^{l}`
.. |iVecW_l_desc|       replace:: 参考系 :math:`l` 相对于惯性参考系 :math:`i` 的角速度
.. |iVecR_O2Wr_eq|      replace:: :math:`\vec{r}_{i}^{O \rightarrow W_R} = \vec{r}_{i}^{O \rightarrow P} + \vec{r}_{l}^{P \rightarrow W_R} = \left[ \begin{array}{c} x \\ y \\ z \end{array} \right]_{i} ^{O \rightarrow W_R}`
.. |iVecR_O2Wr_desc|    replace:: 从点 :math:`P` 到点 :math:`W_R` 在局部坐标系中的位置向量

对 :math:`\vec{r}_{i}^{O \rightarrow W_R}` 的最后一个表达式求导，得到点 :math:`W_R`
在全局参考系中的速度：

.. math::

   \dot{\vec{r}}_{i}^{W_R}
         =  \dot{\vec{r}}_{i}^{P}
         +  \dot{\vec{r}}_{l}^{W_R}
         +  \vec{\omega}_{i}^{l} \times \vec{r}_{l}^{P \rightarrow W_R}.


再重复一次得到其加速度：

.. math::

   \ddot{\vec{r}}_{i}^{W_R}
         =  \dot{\vec{r}}_{i}^{P}
         +  \ddot{\vec{r}}_{l}^{W_R}
         +  2 \vec{\omega}_{i}^{l} \times \dot{\vec{r}}_{l}^{W_R}
         +  \vec{\alpha}_{i}^{l} \times \vec{r}_{i}^{P \rightarrow W_R}
         +  \vec{\omega}_{i}^{l} \times \left( \vec{\omega}_{i}^{l} \times \vec{r}_{l}^{P \rightarrow W_R}\right)


按照牛顿第二定律，该表达式左侧可用力平衡替代：

.. math::

   \ddot{\vec{r}}_{i}^{W_R}
         = \frac{1}{m_R}
            \left[\begin{array}{c}
               \sum{F_x}\\
               \sum{F_y}\\
               \sum{F_z}
            \end{array}\right] ^ {W_R}
         = \frac{1}{m_R}
            \left[\begin{array}{c}
               F_x^{W_R/S} + m_R g_{x} \\
               F_y^{W_R/S} + m_R g_{y} \\
               m_R g_{z}
            \end{array}\right] ^ {W_R}

其中 :math:`g` 是惯性系中的重力向量。
描述右侧柱在局部参考系（:math:`i`）中位置的向量可写为：

.. math::

   \vec{r}_{l}^{P \rightarrow W_R}
         = \left[ \begin{array}{c}
            B/2   \\
            0     \\
            \frac{L-B}{2} + w
         \end{array}\right]_{l} ^{P \rightarrow W_R}.

竖直柱中液体的运动被限制在参考系 N 的 z 方向，因此右侧液柱加速度的表达式变为：

.. math::

   \frac{1}{m_R}
         \left[\begin{array}{c}
            F_x^{W_R/S} + m_R g_{x} \\
            F_y^{W_R/S} + m_R g_{y} \\
            m_R g_{z}
         \end{array}\right] ^{W_R}
   &  =  \quad\left[\begin{array}{c}
            \ddot{x} \\
            \ddot{y} \\
            \ddot{z}
         \end{array}\right]_{i}^{P}
      +  \left[\begin{array}{c}
            0     \\
            0     \\
            \ddot{\omega}
         \end{array}\right]_{l}^{W_R}  \\
   &  \quad
      + 2\left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            0        \\
            0        \\
            \dot{\omega}
         \end{array}\right]_{l}^{W_R}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \ddot{\theta}  \\
            \ddot{\phi}    \\
            \ddot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            B/2   \\
            0     \\
            \frac{L-B}{2} + w
         \end{array}\right]_{l}^{P \rightarrow W_R}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times\left(
         \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
         \times
         \left[\begin{array}{c}
            B/2   \\
            0     \\
            \frac{L-B}{2} + w
         \end{array}\right]_{l}^{P \rightarrow W_R}
      \right)



计算所有叉积，得到 x、y 和 z 三个方向上的三个不同表达式：

.. math::
   x: &  \quad
         \frac{1}{m_R} \left( F_x^{W_R/S} + m_R g_{x} \right)
      &=&
         \ddot{x}_{i}^{P}
      + 2\dot{\phi}\dot{w}
      +  \ddot{\phi} \left(\frac{L-B}{2} + w \right)
      -  \dot{\phi}^2 \frac{B}{2}
      -  \dot{\psi}^2 \frac{B}{2}
      +  \dot{\psi}\dot{\theta}  \left(\frac{L-B}{2} + w \right)   \\
   y: &  \quad
         \frac{1}{m_R}           \left( F_y^{W_R/S} + m_R g_{y} \right)
      &=&
         \ddot{y}_{i}^{P}
      - 2\dot{\theta}\dot{w}
      +  \ddot{\psi} \frac{B}{2}
      -  \ddot{\theta}           \left(\frac{L-B}{2} + w \right)
      +  \dot{\psi}\dot{\phi}    \left(\frac{L-B}{2} + w \right)
      +  \dot{\theta}\dot{\phi}\frac{B}{2} \\
   z: &  \quad
         g_z
      &=&
         \ddot{z}_{i}^{P}
      +  \ddot{w}
      -  \ddot{\phi} \frac{B}{2}
      +  \dot{\theta}\dot{\psi} \frac{B}{2}
      -  \dot{\theta}^2          \left(\frac{L-B}{2} + w \right)
      -  \dot{\phi}^2            \left(\frac{L-B}{2} + w \right)   \\




左侧竖直液柱
---------------------------

按照与上述相同的方法，可以确定描述左侧竖直液柱运动的方程。

类似地，左侧液柱的加速度可用力平衡替代：

.. math::

   \ddot{\vec{r}}_{i}^{W_L}
         = \frac{1}{m_L}
            \left[\begin{array}{c}
               \sum{F_x}\\
               \sum{F_y}\\
               \sum{F_z}
            \end{array}\right] ^ {W_L}
         = \frac{1}{m_L}
            \left[\begin{array}{c}
               F_x^{W_L/S} + m_L g_{x} \\
               F_y^{W_L/S} + m_L g_{y} \\
               m_L g_{z}
            \end{array}\right] ^ {W_L}

其中 :math:`g` 是惯性系中的重力向量。
描述左侧柱在局部参考系（:math:`i`）中位置的向量可写为：

.. math::

   \vec{r}_{l}^{P \rightarrow W_L}
         = \left[ \begin{array}{c}
           -B/2   \\
            0     \\
            \frac{L-B}{2} - w
         \end{array}\right]_{l} ^{P \rightarrow W_L}.

左侧液柱加速度的最终方程为：

.. math::

   \frac{1}{m_L}
         \left[\begin{array}{c}
            F_x^{W_L/S} + m_L g_{x} \\
            F_y^{W_L/S} + m_L g_{y} \\
            m_L g_{z}
         \end{array}\right] ^{W_L}
   &  =  \quad\left[\begin{array}{c}
            \ddot{x} \\
            \ddot{y} \\
            \ddot{z}
         \end{array}\right]_{i}^{P}
      +  \left[\begin{array}{c}
            0     \\
            0     \\
            - \ddot{w}
         \end{array}\right]_{l}^{W_L}  \\
   &  \quad
      + 2\left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            0        \\
            0        \\
            - \dot{w}
         \end{array}\right]_{l}^{W_L}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \ddot{\theta}  \\
            \ddot{\phi}    \\
            \ddot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            -B/2  \\
            0     \\
            \frac{L-B}{2} - w
         \end{array}\right]_{l}^{P \rightarrow W_L}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times\left(
         \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
         \times
         \left[\begin{array}{c}
            -B/2  \\
            0     \\
            \frac{L-B}{2} - w
         \end{array}\right]_{l}^{P \rightarrow W_L}
      \right)



x、y、z 方程变为：

.. math::
   x: &  \quad
         \frac{1}{m_L} \left( F_x^{W_L/S} + m_L g_{x} \right)
      &=&
         \ddot{x}_{i}^{P}
      - 2\dot{\phi}\dot{w}
      +  \ddot{\phi} \left(\frac{L-B}{2} - w \right)
      +  \dot{\phi}^2 \frac{B}{2}
      +  \dot{\psi}^2 \frac{B}{2}
      +  \dot{\psi}\dot{\theta}  \left(\frac{L-B}{2} - w \right)   \\
   y: &  \quad
         \frac{1}{m_L}           \left( F_y^{W_L/S} + m_L g_{y} \right)
      &=&
         \ddot{y}_{i}^{P}
      + 2\dot{\theta}\dot{w}
      -  \ddot{\psi} \frac{B}{2}
      -  \ddot{\theta}           \left(\frac{L-B}{2} - w \right)
      +  \dot{\psi}\dot{\phi}    \left(\frac{L-B}{2} - w \right)
      -  \dot{\theta}\dot{\phi}\frac{B}{2} \\
   z: &  \quad
         g_z
      &=&
         \ddot{z}_{i}^{P}
      -  \ddot{w}
      +  \ddot{\phi} \frac{B}{2}
      -  \dot{\theta}\dot{\psi} \frac{B}{2}
      -  \dot{\theta}^2          \left(\frac{L-B}{2} - w \right)
      -  \dot{\phi}^2            \left(\frac{L-B}{2} - w \right)   \\



水平液柱
------------------------

由于水平柱（:math:`H`）中液体的运动被限制在局部参考系 :math:`l` 的 x 方向，位置向量可表示为：

.. math::

   \vec{r}_{l}^{P \rightarrow W_H}
         = \left[ \begin{array}{c}
            w     \\
            0     \\
            0
         \end{array}\right]_{l} ^{P \rightarrow W_H}.


此外，水平液柱上的力平衡为...

.. math::

   \ddot{\vec{r}}_{i}^{W_H}
         = \frac{1}{m_H}
            \left[\begin{array}{c}
               \sum{F_x}\\
               \sum{F_y}\\
               \sum{F_z}
            \end{array}\right] ^ {W_H}
         = \frac{1}{m_H}
            \left[\begin{array}{c}
               m_H g_{x} - \frac{1}{2} \rho A \xi \left|\dot{w}\right| \dot{w} \\
               F_y^{W_H/S} + m_H g_{y} \\
               F_z^{W_H/S} + m_H g_{z}
            \end{array}\right] ^ {W_H}


其中 :math:`\rho` 项表示液体通过受限孔口时施加给液体的阻尼力，:math:`g` 是惯性系中的重力向量。

水通过水平柱的加速度最终表达式变为：

.. math::

   \frac{1}{m_H}
         \left[\begin{array}{c}
            m_H g_{x} - \frac{1}{2} \rho A \xi \left|\dot{w}\right| \dot{w} \\
            F_y^{W_H/S} + m_H g_{y} \\
            F_z^{W_H/S} + m_H g_{z}
         \end{array}\right] ^ {W_H}
   &  =  \quad\left[\begin{array}{c}
            \ddot{x} \\
            \ddot{y} \\
            \ddot{z}
         \end{array}\right]_{i}^{P}
      +  \left[\begin{array}{c}
            \ddot{w} \\
            0        \\
            0
         \end{array}\right]_{l}^{W_H}  \\
   &  \quad
      + 2\left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            \dot{w}  \\
            0        \\
            0
         \end{array}\right]_{l}^{W_H}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \ddot{\theta}  \\
            \ddot{\phi}    \\
            \ddot{\psi}
         \end{array}\right]_{i}^{l}
      \times
         \left[\begin{array}{c}
            w     \\
            0     \\
            0
         \end{array}\right]_{l}^{P \rightarrow W_H}  \\
   &  \quad
      +  \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
      \times\left(
         \left[\begin{array}{c}
            \dot{\theta}   \\
            \dot{\phi}     \\
            \dot{\psi}
         \end{array}\right]_{i}^{l}
         \times
         \left[\begin{array}{c}
            w     \\
            0     \\
            0
         \end{array}\right]_{l}^{P \rightarrow W_H}
      \right)


x、y、z 方程因此变为：

.. math::

   x: &  \quad
         g_{x} - \frac{1}{m_H} \left( \frac{1}{2} \rho A \xi \left|\dot{w}\right| \dot{w} \right)
      &=&
         \ddot{x}_{i}^{P} + \ddot{w} - \dot{\phi}^2 w - \dot{\psi}^2 w \\
   y: &  \quad
         \frac{1}{m_H} \left( F_y^{W_H/S} + m_H g_{y} \right)
      &=&
         \ddot{y}_{i}^{P} + 2 \dot{\psi} \dot{w} + \ddot{\psi} w + \dot{\theta} \dot{\phi} w  \\
   z: &  \quad
         \frac{1}{m_H} \left( F_z^{W_H/S} + m_H g_{z} \right)
      &=&
         \ddot{z}_{i}^{P} - \dot{\phi}\dot{w} -\ddot{\phi} w + \dot{\theta} \dot{\psi} w  \\


回顾水平柱中液体的位移 :math:`w` 为零，因为水平柱中液体的质心始终保持在点 :math:`l`，
即使液体加速通过管道。因此，从这些方程中移除 :math:`w` 项得到以下表达式：

.. math::

   x: &  \quad
         g_{x} - \frac{1}{m_H} \left( \frac{1}{2} \rho A \xi \left|\dot{w}\right| \dot{w} \right)
      &=&
         \ddot{x}_{i}^{P} + \ddot{w}   \\
   y: &  \quad
         \frac{1}{m_H} \left( F_y^{W_H/S} + m_H g_{y} \right)
      &=&
         \ddot{x}_{i}^{P} + 2 \dot{\psi} \dot{w}   \\
   z: &  \quad
         \frac{1}{m_H} \left( F_z^{W_H/S} + m_H g_{z} \right)
      &=&
         \ddot{z}_{i}^{P} - \dot{\phi}\dot{w}   \\


现在三个液柱的加速度已分别确定，我们可以提取惯性力并推导出
描述柱中液体加速度的单一方程。

惯性力写为：

.. math::

   F_{x}^{W_{R}/S}
      &  = m_{R} \left(
            \ddot{x}_{i}^{P}
         + 2 \dot{\phi} \dot{w}
         +  \ddot{\phi}          \left( \frac{L-B}{2} + w \right)
         -  \dot{\phi}^2 \frac{B}{2}
         -  \dot{\psi}^2 \frac{B}{2}
         +  \dot{\psi}\dot{\phi} \left( \frac{L-B}{2} + w \right)
         -  g_{x} \right)\\
   F_{y}^{W_{R}/S}
      &  = m_{R} \left(
            \ddot{y}_{i}^{P}
         - 2 \dot{\theta} \dot{w}
         +  \ddot{\psi} \frac{B}{2}
         -  \ddot{\theta}        \left( \frac{L-B}{2} + w \right)
         +  \dot{\psi}\dot{\phi} \left( \frac{L-B}{2} + w \right)
         +  \dot{\theta}\dot{\phi} \frac{B}{2}
         -  g_{y} \right)\\
   F_{x}^{W_{L}/S}
      &  = m_{L} \left(
            \ddot{x}_{i}^{P}
         -  2\dot{\phi}\dot{w}
         +  \ddot{\phi}          \left( \frac{L-B}{2} - w \right)
         +  \dot{\phi}^2 \frac{B}{2}
         +  \dot{\psi}^2 \frac{B}{2}
         +  \dot{\psi}\dot{\phi} \left( \frac{L-B}{2} - w \right)
        -  g_{x} \right)\\
   F_{y}^{W_{L}/S}
      &  = m_{L} \left(
            \ddot{y}_{i}^{P}
         + 2 \dot{\theta} \dot{w}
         -  \ddot{\psi} \frac{B}{2}
         -  \ddot{\theta}        \left( \frac{L-B}{2} - w \right)
         +  \dot{\psi}\dot{\phi} \left( \frac{L-B}{2} - w \right)
         -  \dot{\theta}\dot{\phi} \frac{B}{2}
         -  g_{y} \right)\\
   F_{y}^{W_{H}/S}
      &  = m_{H} \left(
            \ddot{y}_{i}^{P}
         + 2\dot{\psi}\dot{w}
         -  g_{y} \right)\\
   F_{z}^{W_{H}/S}
      &  = m_{H} \left(
            \ddot{z}_{i}^{P}
         -  \dot{\phi}\dot{w}
         -  g_{z} \right)



来自右侧液柱（z 方向）的 :math:`\ddot{w}` 方程：

.. math::

   \ddot{w} =
         -  \ddot{z}_{i}^{P}
         +  \ddot{\phi} \frac{B}{2}
         -  \dot{\theta}{\psi} \frac{B}{2}
         +  \dot{\theta}^2 \left( \frac{L-B}{2} + w \right)
         +  \dot{\phi}^2   \left( \frac{L-B}{2} + w \right)
         +  g_{z}


来自左侧液柱（z 方向）的 :math:`\ddot{w}` 方程：

.. math::

   \ddot{w} =
            \ddot{z}_{i}^{P}
         +  \ddot{\phi} \frac{B}{2}
         -  \dot{\theta}{\psi} \frac{B}{2}
         -  \dot{\theta}^2 \left( \frac{L-B}{2} - w \right)
         -  \dot{\phi}^2   \left( \frac{L-B}{2} - w \right)
         -  g_{z}

来自水平液柱（x 方向）的 :math:`\ddot{w}` 方程：

.. math::

   \ddot{w} =
         -  \ddot{x}_{i}^{P}
         +  g_{x}
         - \frac{1}{m_H} \left( \frac{1}{2} \rho A \xi \left|\dot{w}\right| \dot{w} \right)



根据牛顿第二定律，总液体质量的加速度可描述为：

.. math::

   m_{T} \ddot{w} =
         m_{R}\left( \ddot{w} \right)
         m_{L}\left( \ddot{w} \right)
         m_{H}\left( \ddot{w} \right)


其中

.. math::

   m_{T} &= \rho A L \\
   m_{R} &= \rho A \left( \frac{L-B}{2} + w \right)   \\
   m_{R} &= \rho A \left( \frac{L-B}{2} - w \right)   \\
   m_{H} &= \rho A B

组合以上方程得到表达式：

.. math::

   \rho A L \ddot{w}
      &  = \rho A \left( \frac{L-B}{2} + w \right)
       \left[ -  \ddot{z}_{i}^{P}
            +  \ddot{\phi} \frac{B}{2}
            - \dot{\theta} \dot{\psi} \frac{B}{2}  \right.\\
      &  \qquad\qquad\qquad\qquad\qquad \left.
            +  \dot{\theta}^2 \left( \frac{L-B}{2} + w \right)
            +  \dot{\phi}^2   \left( \frac{L-B}{2} + w \right)
            +  g_{z} \right]\\
      & \quad + \rho A B \left(
            -  \ddot{x}_{i}^{P}
            +  g_{x}
            -  \frac{1}{m_{H}} \left(
               \frac{1}{2} \rho A \xi \left|\dot{w}\right|\dot{w}
               \right)
            \right)


最终，简化该表达式得到描述液体通过 TLCD 运动的最终方程：

.. math::

   \rho A L \ddot{w}
      &   =  - 2\rho A w \ddot{z}_{i}^{P}
            +  \rho A B \ddot{\phi} \left( \frac{L-B}{2} \right)
            -  \rho A B \dot{\theta}\dot{\psi} \left( \frac{L-B}{2} \right)\\
      & \qquad \qquad
            + 2\rho A w \dot{\theta}^2 \left( L - B \right)
            + 2\rho A w \dot{\phi}^2   \left( L - B \right)\\
      & \qquad \qquad
            + 2\rho A w g_{z}
            -  \rho A B \ddot{x}_{i}^{P}
            +  \rho A B g_{x}\\
      & \qquad \qquad
            -  \frac{1}{2} \rho A B \xi \left|\dot{w}\right|\dot{w}



正交 TLCD
---------------

按照与上述相同的方法，在侧向（相对于前后方向）方向上得到
前柱、后柱和水平正交柱的以下方程：

后侧竖直正交液柱
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. math::

   x: &  \quad
         \frac{1}{m_B} \left( F_x^{W_B/S} + m_B g_{x} \right)
      &=&
         \ddot{x}_{i}^{P}
      + 2\dot{\phi}\dot{w_o}
      +  \ddot{\phi} \left(\frac{L-B}{2} + w_o \right)
      +  \dot{\psi}^2            \frac{B}{2}
      -  \dot{\phi}\dot{\theta}  \frac{B}{2}
      +  \dot{\psi}\dot{\theta}  \left(\frac{L-B}{2} + w_o \right)   \\
   y: &  \quad
         \frac{1}{m_B}           \left( F_y^{W_B/S} + m_B g_{y} \right)
      &=&
         \ddot{y}_{i}^{P}
      - 2\dot{\theta}\dot{w_o}
      -  \ddot{\theta}           \left(\frac{L-B}{2} + w_o \right)
      +  \dot{\psi}\dot{\phi}    \left(\frac{L-B}{2} + w_o \right)
      +  \dot{\psi}^2   \frac{B}{2}
      +  \dot{\theta}^2 \frac{B}{2} \\
   z: &  \quad
         g_z
      &=&
         \ddot{z}_{i}^{P}
      +  \ddot{w_o}
      -  \ddot{\theta} \frac{B}{2}
      -  \dot{\theta}^2          \left(\frac{L-B}{2} + w_o \right)
      -  \dot{\phi}^2            \left(\frac{L-B}{2} + w_o \right)
      -  \dot{\phi}\dot{\psi} \frac{B}{2}



前侧竖直正交液柱
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. math::

   x: &  \quad
         \frac{1}{m_F} \left( F_x^{W_F/S} + m_F g_{x} \right)
      &=&
         \ddot{x}_{i}^{P}
      - 2\dot{\phi}\dot{w_o}
      +  \ddot{\phi} \left(\frac{L-B}{2} - w_o \right)
      -  \dot{\psi}^2            \frac{B}{2}
      +  \dot{\phi}\dot{\theta}  \frac{B}{2}
      +  \dot{\psi}\dot{\theta}  \left(\frac{L-B}{2} - w_o \right)   \\
   y: &  \quad
         \frac{1}{m_F}           \left( F_y^{W_F/S} + m_F g_{y} \right)
      &=&
         \ddot{y}_{i}^{P}
      + 2\dot{\theta}\dot{w_o}
      -  \ddot{\theta}           \left(\frac{L-B}{2} - w_o \right)
      +  \dot{\psi}\dot{\phi}    \left(\frac{L-B}{2} - w_o \right)
      -  \dot{\psi}^2   \frac{B}{2}
      -  \dot{\theta}^2 \frac{B}{2} \\
   z: &  \quad
         g_z
      &=&
         \ddot{z}_{i}^{P}
      -  \ddot{w_o}
      +  \ddot{\theta} \frac{B}{2}
      -  \dot{\theta}^2          \left(\frac{L-B}{2} - w_o \right)
      -  \dot{\phi}^2            \left(\frac{L-B}{2} - w_o \right)
      +  \dot{\phi}\dot{\psi} \frac{B}{2}



水平正交液柱
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. math::

   x: &  \quad
         \frac{1}{m_H} \left( F_x^{W_H/S} + m_H g_{x} \right)
      &=&
         \ddot{x}_{i}^{P} - 2 \dot{\psi}\dot{w_o}  \\
   y: &  \quad
         \frac{1}{m_H} \left( m_H g_{y} - \frac{1}{2} \rho A \xi \left|\dot{w_o}\right| \dot{w_o} \right)
      &=&
         \ddot{y}_{i}^{P} + \ddot{w_o} \\
   z: &  \quad
         \frac{1}{m_H} \left( F_z^{W_H/S} + m_H g_{z} \right)
      &=&
         \ddot{z}_{i}^{P} + 2 \dot{\theta}\dot{w_o}


从这些方程提取惯性力得到：

.. math::

   F_{x}^{W_{B}/S}
      &  = m_{B} \left(
            \ddot{x}_{i}^{P}
         + 2 \dot{\phi} \dot{w_o}
         +  \ddot{\phi}             \left( \frac{L-B}{2} + w_o \right)
         +  \ddot{\psi}             \frac{B}{2}
         -  \dot{\phi}\dot{\theta}  \frac{B}{2}
         +  \dot{\psi}\dot{\theta}  \left( \frac{L-B}{2} + w_o \right)
         -  g_{x} \right)\\
   F_{y}^{W_{B}/S}
      &  = m_{B} \left(
            \ddot{y}_{i}^{P}
         - 2\dot{\theta} \dot{w_o}
         -  \ddot{\theta}           \left( \frac{L-B}{2} + w_o \right)
         +  \dot{\psi}\dot{\phi}    \left( \frac{L-B}{2} + w_o \right)
         +  \dot{\psi}^2            \frac{B}{2}
         +  \dot{\theta}^2          \frac{B}{2}
         -  g_{y} \right)\\
   F_{x}^{W_{F}/S}
      &  = m_{F} \left(
            \ddot{x}_{i}^{P}
         -  2\dot{\phi}\dot{w_o}
         +  \ddot{\phi}             \left( \frac{L-B}{2} - w_o \right)
         -  \ddot{\psi}             \frac{B}{2}
         +  \dot{\phi}\dot{\theta}  \frac{B}{2}
         +  \dot{\psi}\dot{\theta}  \left( \frac{L-B}{2} - w_o \right)
        -  g_{x} \right)\\
   F_{y}^{W_{F}/S}
      &  = m_{F} \left(
            \ddot{y}_{i}^{P}
         + 2 \dot{\theta} \dot{w_o}
         -  \ddot{\theta}        \left( \frac{L-B}{2} - w_o \right)
         +  \dot{\psi}\dot{\phi} \left( \frac{L-B}{2} - w_o \right)
         -  \dot{\psi}^2   \frac{B}{2}
         -  \dot{\theta}^2 \frac{B}{2}
         -  g_{y} \right)\\
   F_{x}^{W_{H}/S}
      &  = m_{H} \left(
            \ddot{x}_{i}^{P}
         - 2\dot{\psi}\dot{w_o}
         -  g_{x} \right)\\
   F_{z}^{W_{H}/S}
      &  = m_{H} \left(
            \ddot{z}_{i}^{P}
         + 2\dot{\theta}\dot{w_o}
         -  g_{z} \right)
.. there might be a descrepency with the 2 in the last term.  Doesn't match the
   other case.


其余方程组合后得到最终方程：

.. math::

   \rho A L \ddot{w}
      &  = \rho A \left( \frac{L-B}{2} + w_o \right)
       \left[ -  \ddot{z}_{i}^{P}
            +  \ddot{\theta} \frac{B}{2}
            +  \dot{\theta}^2 \left( \frac{L-B}{2} + w_o \right) \right.\\
      &  \qquad\qquad\qquad\qquad\qquad \left.
            +  \dot{\phi}^2   \left( \frac{L-B}{2} + w_o \right)
            +  \dot{\phi}\dot{\psi} \frac{B}{2}
            +  g_{z} \right]\\
      &  + \rho A \left( \frac{L-B}{2} - w_o \right)
       \left[  \ddot{z}_{i}^{P}
            +  \ddot{\theta} \frac{B}{2}
            -  \dot{\theta}^2 \left( \frac{L-B}{2} - w_o \right) \right.\\
      &  \qquad\qquad\qquad\qquad\qquad \left.
            -  \dot{\phi}^2   \left( \frac{L-B}{2} - w_o \right)
            +  \dot{\phi}\dot{\psi} \frac{B}{2}
            -  g_{z} \right]\\
      & \quad + \rho A B \left(
            -  \ddot{y}_{i}^{P}
            +  g_{y} \right)
            -  \frac{1}{2} \rho A \xi \left|\dot{w_o}\right|\dot{w_o}

可简化为：

.. math::

   \rho A L \ddot{w_o}
      &   = - 2\rho A w_o \ddot{z}_{i}^{P}
            +  \rho A B \ddot{\theta} \left( \frac{L-B}{2} \right)
            -  \rho A B \dot{\phi}\dot{\psi} \left( \frac{L-B}{2} \right)\\
      & \qquad \qquad
            + 2\rho A w_o \dot{\theta}^2  \left( L - B \right)
            + 2\rho A w_o \dot{\phi}^2    \left( L - B \right) \\
      & \qquad \qquad
            + 2\rho A w_o g_{z}
            -  \rho A B \ddot{y}_{i}^{P}
            +  \rho A B g_{y}\\
      & \qquad \qquad
            -  \frac{1}{2} \rho A B \xi \left|\dot{w_o}\right|\dot{w_o}
