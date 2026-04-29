.. _sed-theory:

理论
=============

本模块中，转子用一个刚性盘表示。


本模块具有两个状态 :math:`\psi` 和 :math:`\dot{\psi}`，分别对应方位角和转子转速。
（注意：将方位角作为状态引入是可选的，但为了方便与 AeroDyn 耦合。
由于两个方程实际上是解耦的，这种引入不应对所需的时间步长产生任何影响。）
状态空间方程为：

.. math::  :label: sed_stateEq

   \begin{aligned}
       \dot{\psi}  & = \dot{\psi} \\
       \ddot{\psi} & = \frac{1}{J_\text{DT}}\left( Q_g - Q_a + Q_b\right)
   \end{aligned}

其中 :math:`J_{DT}` 是传动链的总惯量（叶片+轮毂+发电机），
:math:`Q_g`、:math:`Q_a` 和 :math:`Q_b` 分别是发电机转矩、气动转矩和制动转矩，
均表示为低速轴（LSS）侧的量。
传动链总惯量按下式获得：

.. math::  :label: sed_JDT

   J_\text{DT} = J_r + n_g^2 J_{g,HSS}

其中 :math:`J_r` 是转子的惯量（叶片+轮毂+"轴"），
:math:`n_g` 是齿轮箱的传动比，
:math:`J_{g,HSS}` 是发电机在高速轴（HSS）上的惯量。
注意，OpenFAST 将轴的惯量视为包含在"轮毂"（即转子）中。
发电机和制动转矩在 LSS 上的值由 HSS 上的值按以下方式获得：

.. math::  :label: QgLSS

   Q_g = n_g Q_{g,HSS}
   ,\quad
   Q_b = n_g Q_{b,HSS}

..
   其中 :math:`\eta_{DT}` 是传动链的效率。
   Q_g = \frac{n_g}{\eta_{DT}} Q_{g,HSS}

方程 :eq:`sed_stateEq` 相关的初始条件为：

.. math::  :label: sed_stateInit

   \begin{aligned}
       \psi       & = \psi_0 \\
       \dot{\psi} & = \Omega_0
   \end{aligned}

其中 :math:`\psi_0` 是以 rad 为单位的初始方位角，:math:`\Omega_0` 是以 rad/s 为单位的初始转子转速。



如果发电机自由度关闭，则状态按下式简单确定：


.. math::  :label: sed_stateEqGenDOF


   \begin{aligned}
       \psi  & = \psi_0 +  \int_{0}^t \dot{\psi} dt = \psi_0  +  \Omega_0 n \Delta t \\
       \dot{\psi} & = \Omega_0
   \end{aligned}

其中 :math:`n` 是时间步索引，:math:`\Delta t` 是模块的时间步长。
