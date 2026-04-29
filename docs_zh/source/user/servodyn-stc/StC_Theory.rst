.. _StC-Theory:

==========================================================
OpenFAST 调谐质量阻尼器模块理论手册
==========================================================

:作者: William La Cava & Matthew A. Lackner
   马萨诸塞大学阿默斯特分校机械与工业工程系
   美国马萨诸塞州阿默斯特，01003
   ``wlacava@umass.edu``, ``lackner@ecs.umass.edu``

本文档由 NREL 的 Jason M. Jonkman 编辑，
增加了 OpenFAST 中独立垂直方向 TMD 的功能。
``jason.jonkman@nrel.gov``


本手册介绍 OpenFAST 中仿真用于结构控制的调谐质量阻尼器（TMD）
的更新功能。阻尼器可添加到叶片、机舱、塔筒或子结构上。
关于这些系统的应用研究，请参阅
:cite:`stc-lackner_passive_2011,stc-lackner_structural_2011,stc-namik_active_2013,stc-stewart_effect_2011,stc-stewart_impact_2014,stc-stewart_optimization_2013`。
TMD 是三个独立的、单自由度的线性质量弹簧阻尼单元，
分别作用于每个部件局部坐标系 :math:`x`、:math:`y` 和 :math:`z` 方向。
结构控制（StC）模块的其他功能，包括全向 TMD 和 TLCD，
本文未予记载。本文首先介绍理论背景，然后描述代码修改。

理论背景
======================

定义
-----------

.. container::
   :name: tab:defs

   .. table:: 定义

      +-----------------+--------------------+
      | 变量            | 描述               |
      +=================+====================+
      | |O_eq|          | |O_desc|           |
      +-----------------+--------------------+
      | |P_eq|          | |P_desc|           |
      +-----------------+--------------------+
      | |TMD_eq|        | |TMD_desc|         |
      +-----------------+--------------------+
      | |G_eq|          | |G_desc|           |
      +-----------------+--------------------+
      | |N_eq|          | |N_desc|           |
      +-----------------+--------------------+
      | |TMD_OG_eq|     | |TMD_OG_desc|      |
      +-----------------+--------------------+
      | |TMD_PN_eq|     | |TMD_PN_desc|      |
      +-----------------+--------------------+
      | |TMD_X_eq|      | |TMD_X_desc|       |
      +-----------------+--------------------+
      | |TMD_Y_eq|      | |TMD_Y_desc|       |
      +-----------------+--------------------+
      | |TMD_Z_eq|      | |TMD_Z_desc|       |
      +-----------------+--------------------+
      | |P_OG_eq|       | |P_OG_desc|        |
      +-----------------+--------------------+
      | |R_OG_eq|       | |R_OG_desc|        |
      +-----------------+--------------------+
      | |R_GN_eq|       | |R_GN_desc|        |
      +-----------------+--------------------+
      | |Omega_NON_eq|  | |Omega_NON_desc|   |
      +-----------------+--------------------+
      | |OmegaD_NON_eq| | |OmegaD_NON_desc|  |
      +-----------------+--------------------+
      | |a_GOG_eq|      | |a_GOG_desc|       |
      +-----------------+--------------------+
      | |a_GON_eq|      | |a_GON_desc|       |
      +-----------------+--------------------+

.. |O_eq|            replace:: :math:`O`
.. |O_desc|          replace:: 全局惯性参考系的原点
.. |P_eq|            replace:: :math:`P`
.. |P_desc|          replace:: 固定于部件（叶片、机舱、塔筒、子结构）的非惯性参考系原点，TMD 在此处静止
.. |TMD_eq|          replace:: :math:`TMD`
.. |TMD_desc|        replace:: TMD 的位置点
.. |G_eq|            replace:: :math:`G`
.. |G_desc|          replace:: 全局参考系的轴方向
.. |N_eq|            replace:: :math:`N`
.. |N_desc|          replace:: 局部部件参考系的轴方向，单位向量为 :math:`\hat{\imath}, \hat{\jmath}, \hat{k}`
.. |TMD_OG_eq|       replace:: :math:`\vec{r}_{_{_{TMD/O_G}}} = \left[ \begin{array}{c} x \\ y\\ z \end{array} \right]_{_{TMD/O_G}}`
.. |TMD_OG_desc|     replace:: TMD 相对于 :math:`O`、方向为 :math:`G` 的位置
.. |TMD_PN_eq|       replace:: :math:`\vec{r}_{_{_{TMD/P_N}}} = \left[ \begin{array}{c} x \\ y\\ z \end{array} \right]_{_{TMD/P_N}}`
.. |TMD_PN_desc|     replace:: TMD 相对于 :math:`P_N` 的位置
.. |TMD_X_eq|        replace:: :math:`\vec{r}_{_{_{TMD_X}}}`
.. |TMD_X_desc|      replace:: :math:`TMD_X` 的位置向量
.. |TMD_Y_eq|        replace:: :math:`\vec{r}_{_{_{TMD_Y}}}`
.. |TMD_Y_desc|      replace:: :math:`TMD_Y` 的位置向量
.. |TMD_Z_eq|        replace:: :math:`\vec{r}_{_{_{TMD_Z}}}`
.. |TMD_Z_desc|      replace:: :math:`TMD_Z` 的位置向量
.. |P_OG_eq|         replace:: :math:`\vec{r}_{_{P/O_G}} =\left[ \begin{array}{c} x \\ y\\ z \end{array} \right]_{_{P/O_G}}`
.. |P_OG_desc|       replace:: 部件相对于 :math:`O_G` 的位置向量
.. |R_OG_eq|         replace:: :math:`R_{_{N/G}}`
.. |R_OG_desc|       replace:: 将方向 :math:`G` 变换为 :math:`N` 的 3×3 旋转矩阵
.. |R_GN_eq|         replace:: :math:`R_{_{G/N}} = R_{_{N/G}}^T`
.. |R_GN_desc|       replace:: 从 :math:`N` 到 :math:`G` 的变换
.. |Omega_NON_eq|    replace:: :math:`\vec{\omega}_{_{N/O_N}} = \dot{\left[ \begin{array}{c} \theta \\ \phi \\ \psi \end{array} \right]}_{_{N/O_N}}`
.. |Omega_NON_desc|  replace:: 部件在方向 :math:`N` 上的角速度；对 :math:`G` 同样定义
.. |OmegaD_NON_eq|   replace:: :math:`\dot{\vec{\omega}}_{_{N/O_N}} = \vec{\alpha}_{_{N/O_N}}`
.. |OmegaD_NON_desc| replace:: 部件的角加速度
.. |a_GOG_eq|        replace:: :math:`\vec{a}_{G/O_G} = \left[ \begin{array}{c}0 \\ 0\\ -g \end{array} \right]_{/O_G}`
.. |a_GOG_desc|      replace:: 全局坐标下的重力加速度
.. |a_GON_eq|        replace:: :math:`\vec{a}_{G/O_N} = R_{_{N/G}} \vec{a}_{G/O_G} = \left[ \begin{array}{c}a_{_{G_X}} \\ a_{_{G_Y}}\\ a_{_{G_Z}} \end{array} \right]_{/O_N}`
.. |a_GON_desc|      replace:: 相对于 :math:`O_N` 的重力


运动方程
-------------------

TMD 在两个参考系 :math:`O` 和 :math:`P` 中的位置向量关系为

.. math:: \vec{r}_{_{TMD/O_G}} =  \vec{r}_{_{P/O_G}} +  \vec{r}_{_{TMD/P_G}}

以方向 :math:`N` 表示，

.. math:: \vec{r}_{_{TMD/O_N}} =  \vec{r}_{_{P/O_N}} +  \vec{r}_{_{TMD/P_N}}

.. math:: \Rightarrow \vec{r}_{_{TMD/P_N}} =  \vec{r}_{_{TMD/O_N}} -  \vec{r}_{_{P/O_N}}

求导得，[1]_

.. math::
   \dot{\vec{r}}_{_{TMD/P_N}}= \dot{\vec{r}}_{_{TMD/O_N}}
      - \dot{\vec{r}}_{_{P/O_N}}
      - \vec{\omega}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}}

再次求导可得 TMD 相对于 :math:`P`（机舱位置）、方向为 :math:`N` 的加速度：

.. math::
   \begin{array}{cc}
      \ddot{\vec{r}}_{_{TMD/P_N}} =
               &  \ddot{\vec{r}}_{_{TMD/O_N}}
                  - \ddot{\vec{r}}_{_{P/O_N}} - \vec{\omega}_{_{N/O_N}}
                  \times (\vec{\omega}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}}) \\[1.1em]
               &- \vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}}
                  - 2 \vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD/P_N}}
   \end{array}
   :label: accel

右侧包含以下各项：

.. container::
   :name: tab:

   .. table:: 右侧各项

      +--------------------+-----------------------+
      | |Rddot_TMD_ON_eq|  | |Rddot_TMD_ON_desc|   |
      +--------------------+-----------------------+
      | |Rddot_P_ON_eq|    | |Rddot_P_ON_desc|     |
      +--------------------+-----------------------+
      | |Omega_N_ON_eq|    | |Omega_N_ON_desc|     |
      +--------------------+-----------------------+
      | |CentripAcc_eq|    | |CentripAcc_desc|     |
      +--------------------+-----------------------+
      | |TangentAcc_eq|    | |TangentAcc_desc|     |
      +--------------------+-----------------------+
      | |Coriolus_eq|      | |Coriolus_desc|       |
      +--------------------+-----------------------+

.. |Rddot_TMD_ON_eq|   replace:: :math:`\ddot{\vec{r}}_{_{TMD/O_N}}`
.. |Rddot_TMD_ON_desc| replace:: TMD 在*惯性*系 :math:`O_N` 中的加速度
.. |Rddot_P_ON_eq|   replace:: :math:`\ddot{\vec{r}}_{_{P/O_N}} = R_{_{N/G}} \ddot{\vec{r}}_{_{P/O_G}}`
.. |Rddot_P_ON_desc| replace:: 机舱原点 :math:`P` 相对于 :math:`O_N` 的加速度
.. |Omega_N_ON_eq|   replace:: :math:`\vec{\omega}_{_{N/O_N}} = R_{_{N/G}} \vec{\omega}_{_{N/O_G}}`
.. |Omega_N_ON_desc| replace:: 机舱相对于 :math:`O_N` 的角速度
.. |CentripAcc_eq|   replace:: :math:`\vec{\omega}_{_{N/O_N}} \times (\vec{\omega}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}})`
.. |CentripAcc_desc| replace:: 向心加速度
.. |TangentAcc_eq|   replace:: :math:`\vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}}`
.. |TangentAcc_desc| replace:: 切向加速度
.. |Coriolus_eq|   replace:: :math:`2\vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD/P_N}}`
.. |Coriolus_desc| replace:: 科里奥利加速度


惯性系中的加速度 :math:`\ddot{\vec{r}}_{_{TMD/O_N}}` 可用力平衡替代

.. math::
   \begin{aligned}
      \ddot{\vec{r}}_{_{TMD/O_N}} = \left[
         \begin{array}{c} \ddot{x} \\
            \ddot{y} \\
            \ddot{z}
         \end{array}
      \right]_{_{TMD/O_N}} = \frac{1}{m} \left[
         \begin{array}{c}
            \sum{F_X} \\
            \sum{F_Y} \\
            \sum{F_Z}
         \end{array}
      \right]_{_{TMD/O_N}} = \frac{1}{m} \vec{F}_{_{TMD/O_N}}
    \end{aligned}

将力平衡代入方程 :eq:`accel` 可得 TMD 的一般运动方程：

.. math::
   \begin{array}{cc}
      \ddot{\vec{r}}_{_{TMD/P_N}} = & \frac{1}{m} \vec{F}_{_{TMD/O_N}}
         - \ddot{\vec{r}}_{_{P/O_N}}
         - \vec{\omega}_{_{N/O_N}} \times (\vec{\omega}_{_{N/O_N}}
               \times \vec{r}_{_{TMD/P_N}}) \\[1.1em]
      & - \vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD/P_N}}
         - 2 \vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD/P_N}}
   \end{array}
   :label: EOM

现在分别求解 :math:`TMD_X`、:math:`TMD_Y` 和 :math:`TMD_Z` 的运动方程。

TMD_X：
~~~~~~~

外力 :math:`\vec{F}_{_{TMD_X/O_N}}` 由下式给出

.. math::
   \vec{F}_{_{TMD_X/O_N}} = \left[
      \begin{array}{c}
         - c_x \dot{x}_{_{TMD_X/P_N}}
         - k_x x_{_{TMD_X/P_N}}
         + m_x a_{_{G_X/O_N}}
         + F_{ext_x}
         + F_{StopFrc_{X}} \\
         F_{Y_{_{TMD_X/O_N}}}
         + m_x a_{_{G_Y/O_N}}  \\
         F_{Z_{_{TMD_X/O_N}}}
         + m_x a_{_{G_Z/O_N}}
      \end{array}
   \right]

:math:`TMD_X` 在 :math:`y` 和 :math:`z` 方向上固定于参考系 :math:`N`，因此

.. math::
   {r}_{_{TMD_X/P_N}} = \left[
      \begin{array}{c}
         x_{_{TMD_X/P_N}} \\
         0 \\
         0
      \end{array}
   \right]

方程 :eq:`EOM` 的其他分量为：

.. math::
   \vec{\omega}_{_{N/O_N}} \times (\vec{\omega}_{_{N/O_N}} \times \vec{r}_{_{TMD_X/P_N}})
         = x_{_{TMD_X/P_N}} \left[
      \begin{array}{c}
         - (\dot{\phi}_{_{N/O_N}}^2 + \dot{\psi}_{_{N/O_N}}^2) \\
         \dot{\theta}_{_{N/O_N}}\dot{\phi}_{_{N/O_N}} \\
         \dot{\theta}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}}
       \end{array}
   \right]

.. math::
   2\vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD_X/P_N}}
         = \dot{x}_{_{TMD_X/P_N}} \left[
      \begin{array}{c} 0 \\
         2\dot{\psi}_{_{N/O_N}} \\
         -2\dot{\phi}_{_{N/O_N}}
      \end{array}
   \right]

.. math:: \vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD_X/P_N}} = x_{_{TMD_X/P_N}} \left[ \begin{array}{c} 0 \\ \ddot{\psi}_{_{N/O_N}} \\ -\ddot{\phi}_{_{N/O_N}}\end{array} \right]

因此 :math:`\ddot{x}_{_{TMD_X/P_N}}` 由以下方程控制

.. math::
   \begin{aligned}
      \ddot{x}_{_{TMD_X/P_N}} =& (\dot{\phi}_{_{N/O_N}}^2
         + \dot{\psi}_{_{N/O_N}}^2-\frac{k_x}{m_x}) x_{_{TMD_X/P_N}}
         - (\frac{c_x}{m_x}) \dot{x}_{_{TMD_X/P_N}}
         -\ddot{x}_{_{P/O_N}}+a_{_{G_X/O_N}} \\
      &+ \frac{1}{m_x} ( F_{ext_X} + F_{StopFrc_{X}})
   \end{aligned}
   :label: EOM_Xx

力 :math:`F_{Y_{_{TMD_X/O_N}}}` 和 :math:`F_{Z_{_{TMD_X/O_N}}}` 的求解注意到
:math:`\ddot{y}_{_{TMD_X/P_N}} = \ddot{z}_{_{TMD_X/P_N}} = 0`：

.. math::
   F_{Y_{_{TMD_X/O_N}}} = m_x \left( - a_{_{G_Y/O_N}} +\ddot{y}_{_{P/O_N}}
      + (\ddot{\psi}_{_{N/O_N}}
      + \dot{\theta}_{_{N/O_N}}\dot{\phi}_{_{N/O_N}} ) x_{_{TMD_X/P_N}}
      + 2\dot{\psi}_{_{N/O_N}} \dot{x}_{_{TMD_X/P_N}} \right)
   :label: EOM_Xy

.. math::
   F_{Z_{_{TMD_X/O_N}}} = m_x \left( - a_{_{G_Z/O_N}} +\ddot{z}_{_{P/O_N}}
      - (\ddot{\phi}_{_{N/O_N}}
      - \dot{\theta}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}} ) x_{_{TMD_X/P_N}}
      - 2\dot{\phi}_{_{N/O_N}} \dot{x}_{_{TMD_X/P_N}} \right)
   :label: EOM_Xz

TMD_Y：
~~~~~~

:math:`TMD_Y` 上的外力 :math:`\vec{F}_{_{TMD_Y/P_N}}` 由下式给出

.. math::
   \vec{F}_{_{TMD_Y/P_N}} =  \left[
      \begin{array}{c}
         F_{X_{_{TMD_Y/O_N}}} + m_y a_{_{G_X/O_N}}\\
         - c_y \dot{y}_{_{TMD_Y/P_N}} - k_y y_{_{TMD_Y/P_N}}
         + m_y a_{_{G_Y/O_N}} + F_{ext_y} + F_{StopFrc_{Y}} \\
         F_{Z_{_{TMD_Y/O_N}}}+ m_y a_{_{G_Z/O_N}}
      \end{array}
   \right]

:math:`TMD_Y` 在 :math:`x` 和 :math:`z` 方向上固定于参考系 :math:`N`，因此

.. math::
   {r}_{_{TMDYX/P_N}} = \left[
      \begin{array}{c}
         0 \\
         y_{_{TMD_Y/P_N}} \\
         0
      \end{array}
   \right]

方程 :eq:`EOM` 的其他分量为：

.. math::
   \vec{\omega}_{_{N/O_N}} \times (\vec{\omega}_{_{N/O_N}}
         \times \vec{r}_{_{TMD_Y/P_N}})
      = y_{_{TMD_Y/P_N}}
      \left[
         \begin{array}{c}
            \dot{\theta}_{_{N/O_N}}\dot{\phi}_{_{N/O_N}} \\
            -(\dot{\theta}_{_{N/O_N}}^2 + \dot{\psi}_{_{N/O_N}}^2)  \\
            \dot{\phi}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}}
         \end{array}
      \right]

.. math::
   2\vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD_Y/P_N}}
      = \dot{y}_{_{TMD_Y/P_N}} \left[
         \begin{array}{c}
            - 2 \dot{\psi}_{_{N/O_N}} \\
            0 \\
            2 \dot{\theta}_{_{N/O_N}}
         \end{array}
      \right]

.. math::
   \vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD_Y/P_N}}
      = y_{_{TMD_Y/P_N}} \left[
         \begin{array}{c}
            - \ddot{\psi}_{_{N/O_N}} \\
            0 \\
            \ddot{\theta}_{_{N/O_N}}
         \end{array}
      \right]

因此 :math:`\ddot{y}_{_{TMD_Y/P_N}}` 由以下方程控制

.. math::
   \begin{aligned}
      \ddot{y}_{_{TMD_Y/P_N}}
         = & (\dot{\theta}_{_{N/O_N}}^2
            + \dot{\psi}_{_{N/O_N}}^2-\frac{k_y}{m_y}) y_{_{TMD_Y/P_N}}
            - (\frac{c_y}{m_y}) \dot{y}_{_{TMD_Y/P_N}}
            -\ddot{y}_{_{P/O_N}} + a_{_{G_Y/O_N}}\\
         &+ \frac{1}{m_y} (F_{ext_Y} + F_{StopFrc_{Y}})
   \end{aligned}
   :label: EOM_Yy

力 :math:`F_{X_{_{TMD_Y/O_N}}}` 和 :math:`F_{Z_{_{TMD_Y/O_N}}}` 的求解注意到
:math:`\ddot{x}_{_{TMD_Y/P_N}} = \ddot{z}_{_{TMD_Y/P_N}} = 0`：

.. math::
   F_{X_{_{TMD_Y/O_N}}} = m_y \left( - a_{_{G_X/O_N}} + \ddot{x}_{_{P/O_N}}
      - (\ddot{\psi}_{_{N/O_N}}
      - \dot{\theta}_{_{N/O_N}}\dot{\phi}_{_{N/O_N}}) y_{_{TMD_Y/P_N}}
      - 2\dot{\psi}_{_{N/O_N}} \dot{y}_{_{TMD_Y/P_N}} \right)
   :label: EOM_Yx

.. math::
   F_{Z_{_{TMD_Y/O_N}}} = m_y \left( - a_{_{G_Z/O_N}} + \ddot{z}_{_{P/O_N}}
      + (\ddot{\theta}_{_{N/O_N}}
      + \dot{\phi}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}}) y_{_{TMD_Y/P_N}}
      + 2\dot{\theta}_{_{N/O_N}} \dot{y}_{_{TMD_Y/P_N}} \right)
   :label: EOM_Yz


TMD_Z：
~~~~~~~

外力 :math:`\vec{F}_{_{TMD_Z/O_N}}` 由下式给出

.. math::
   \vec{F}_{_{TMD_Z/O_N}} = \left[
      \begin{array}{c}
         F_{X_{_{TMD_Z/O_N}}} + m_z a_{_{G_X/O_N}} \\
         F_{Y_{_{TMD_Z/O_N}}} + m_z a_{_{G_Y/O_N}} \\
         - c_z \dot{z}_{_{TMD_Z/P_N}} - k_z z_{_{TMD_Z/P_N}}
         + m_z a_{_{G_Z/O_N}} + F_{ext_z} + F_{StopFrc_{Z}} + F_{Z_{PreLoad}}
      \end{array}
   \right]

其中 :math:`F_{Z_{PreLoad}}` 是弹簧预载，用于在重力作用于 :math:`TMD_Z` 质量块时
偏移其中性位置。
:math:`TMD_Z` 在 :math:`x` 和 :math:`y` 方向上固定于参考系 :math:`N`，因此

.. math::
   {r}_{_{TMD_Z/P_N}} = \left[
      \begin{array}{c}
         0 \\
         0 \\
         z_{_{TMD_Z/P_N}}
      \end{array}
   \right]

方程 :eq:`EOM` 的其他分量为：

.. math::
   \vec{\omega}_{_{N/O_N}} \times (\vec{\omega}_{_{N/O_N}} \times \vec{r}_{_{TMD_Z/P_N}})
      = z_{_{TMD_Z/P_N}} \left[
         \begin{array}{c}
            \dot{\theta}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}} \\
            \dot{\phi}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}} \\
            -(\dot{\theta}_{_{N/O_N}}^2 + \dot{\phi}_{_{N/O_N}}^2)
         \end{array}
      \right]

.. math::
   2\vec{\omega}_{_{N/O_N}} \times \dot{\vec{r}}_{_{TMD_Z/P_N}}
      = \dot{z}_{_{TMD_Z/P_N}} \left[
         \begin{array}{c}
            2\dot{\phi}_{_{N/O_N}} \\
            -2\dot{\theta}_{_{N/O_N}} \\
            0
         \end{array}
      \right]

.. math::
   \vec{\alpha}_{_{N/O_N}} \times \vec{r}_{_{TMD_Z/P_N}}
      = z_{_{TMD_Z/P_N}} \left[
         \begin{array}{c}
            \ddot{\phi}_{_{N/O_N}} \\
            -\ddot{\theta}_{_{N/O_N}} \\
            0
         \end{array}
      \right]

因此 :math:`\ddot{z}_{_{TMD_Z/P_N}}` 由以下方程控制

.. math::
   \begin{aligned}
      \ddot{z}_{_{TMD_Z/P_N}}
         = & (\dot{\theta}_{_{N/O_N}}^2
            + \dot{\phi}_{_{N/O_N}}^2-\frac{k_z}{m_z}) z_{_{TMD_Z/P_N}}
            - (\frac{c_z}{m_z}) \dot{z}_{_{TMD_Z/P_N}}
            -\ddot{z}_{_{P/O_N}} + a_{_{G_Z/O_N}}\\
         &+ \frac{1}{m_z} (F_{ext_Z} + F_{StopFrc_{Z}} + F_{Z_{PreLoad}})
   \end{aligned}
   :label: EOM_Zz



力 :math:`F_{X_{_{TMD_Z/O_N}}}` 和 :math:`F_{Y_{_{TMD_Z/O_N}}}` 的求解注意到
:math:`\ddot{x}_{_{TMD_Z/P_N}} = \ddot{y}_{_{TMD_Z/P_N}} = 0`：

.. math::
   F_{X_{_{TMD_Z/O_N}}} = m_z \left( - a_{_{G_X/O_N}} + \ddot{x}_{_{P/O_N}}
      + (\ddot{\phi}_{_{N/O_N}}
      + \dot{\theta}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}}) z_{_{TMD_Z/P_N}}
      + 2\dot{\phi}_{_{N/O_N}} \dot{z}_{_{TMD_Z/P_N}} \right)
   :label: EOM_Zx

.. math::
   F_{Y_{_{TMD_Z/O_N}}} = m_z \left( - a_{_{G_Y/O_N}} + \ddot{y}_{_{P/O_N}}
      - (\ddot{\theta}_{_{N/O_N}}
      - \dot{\phi}_{_{N/O_N}}\dot{\psi}_{_{N/O_N}}) z_{_{TMD_Z/P_N}}
      - 2\dot{\theta}_{_{N/O_N}} \dot{z}_{_{TMD_Z/P_N}} \right)
   :label: EOM_Zy


状态方程
---------------

输入：
~~~~~~~

输入为部件线加速度和角位置、角速度和角加速度：

.. math::
   \vec{u} = \left[
      \begin{array}{c}
         \ddot{\vec{r}}_{_{P/O_G}} \\
         \vec{R}_{_{N/G}} \\
         \vec{\omega}_{_{N/O_G}} \\
         \vec{\alpha}_{_{N/O_G}}
      \end{array}
   \right]
   \Rightarrow \left[
      \begin{array}{c}
         \ddot{\vec{r}}_{_{P/O_N}} \\
         \vec{\omega}_{_{N/O_N}} \\
         \vec{\alpha}_{_{N/O_N}}
      \end{array}
    \right]
    = \left[
      \begin{array}{c}
         \vec{R}_{_{N/G}} \ddot{\vec{r}}_{_{P/O_G}} \\
         \vec{R}_{_{N/G}} \vec{\omega}_{_{N/O_G}} \\
         \vec{R}_{_{N/G}} \vec{\alpha}_{_{N/O_G}
      }\end{array}
   \right]

状态：
~~~~~~~

状态为 TMD 在其各自部件参考系 DOF 方向上的位置和速度：

.. math::
   \vec{R}_{_{TMD/P_N}} = \left[
      \begin{array}{c}
         x \\
         \dot{x} \\
         y \\
         \dot{y} \\
         z \\
         \dot{z}
      \end{array}
   \right]_{_{TMD/P_N}}
   = \left[
      \begin{array}{c}
         {x}_{_{TMD_X/P_N}} \\
         \dot{x}_{_{TMD_X/P_N}} \\
         {y}_{_{TMD_Y/P_N}} \\
         \dot{y}_{_{TMD_Y/P_N}} \\
         {z}_{_{TMD_Z/P_N}} \\
         \dot{z}_{_{TMD_Z/P_N}}
      \end{array}
   \right]

运动方程可以重写为如下形式的非线性一阶方程组

.. math::
   \dot{\vec{R}}_{_{TMD}} = A \vec{R}_{_{TMD}} + B

\ 其中

.. math::
   A(\vec{u}) = \left[
      \begin{array}{cccccc}
      0& 1 &0&0&0&0 \\
      (\dot{\phi}_{_{N/O_N}}^2 + \dot{\psi}_{_{N/O_N}}^2-\frac{k_x}{m_x}) & - (\frac{c_x}{m_x}) &0&0&0&0 \\
      0&0&0& 1 &0&0 \\
      0&0& (\dot{\theta}_{_{N/O_N}}^2 + \dot{\psi}_{_{N/O_N}}^2-\frac{k_y}{m_y}) & - (\frac{c_y}{m_y}) &0&0 \\
      0&0&0&0&0& 1 \\
      0&0&0&0& (\dot{\theta}_{_{N/O_N}}^2 + \dot{\phi}_{_{N/O_N}}^2-\frac{k_z}{m_z}) & - (\frac{c_z}{m_z}) \\
   \end{array} \right]

且

.. math::
   B(\vec{u}) = \left[
      \begin{array}{l}
         0 \\
         -\ddot{x}_{_{P/O_N}}+a_{_{G_X/O_N}} + \frac{1}{m_x} ( F_{ext_X} + F_{StopFrc_{X}}) \\
         0 \\
         -\ddot{y}_{_{P/O_N}}+a_{_{G_Y/O_N}} + \frac{1}{m_y} (F_{ext_Y}+ F_{StopFrc_{Y}}) \\
         0 \\
         -\ddot{z}_{_{P/O_N}}+a_{_{G_Z/O_N}} + \frac{1}{m_z} (F_{ext_Z}+ F_{StopFrc_{Z}} + F_{Z_{PreLoad}})
      \end{array}
   \right]
   :label: Bu

输入与状态变量耦合，导致 A 和 B 为 :math:`f(\vec{u})`。

输出
-------

输出向量 :math:`\vec{Y}` 为

.. math::
   \vec{Y} = \left[
      \begin{array}{c}
         \vec{F}_{_{P_G}} \\
         \vec{M}_{_{P_G}}
      \end{array}
   \right]

输出包括对应于方程 :eq:`EOM_Xy`、:eq:`EOM_Xz`、:eq:`EOM_Yx`、:eq:`EOM_Yz`、
:eq:`EOM_Zx` 和 :eq:`EOM_Zy` 中
:math:`F_{Y_{_{TMD_X/O_N}}}`、:math:`F_{Z_{_{TMD_X/O_N}}}`、:math:`F_{X_{_{TMD_Y/O_N}}}`、
:math:`F_{Z_{_{TMD_Y/O_N}}}`、:math:`F_{X_{_{TMD_Z/O_N}}}` 和 :math:`F_{Y_{_{TMD_Z/O_N}}}`
的反力。作用在部件上的合力 :math:`\vec{F}_{_{P_G}}` 和合力矩 :math:`\vec{M}_{_{P_G}}` 为

.. math::
   \begin{aligned}
      \vec{F}_{_{P_G}} = R^T_{_{N/G}} & \left[
         \begin{array}{l}
            k_x {x}_{_{TMD_X/P_N}} + c_x \dot{x}_{_{TMD_X/P_N}} - F_{StopFrc_{X}} - F_{ext_x} - F_{X_{_{TMD_Y/O_N}}} - F_{X_{_{TMD_Z/O_N}}} \\
            k_y {y}_{_{TMD_Y/P_N}} + c_y \dot{y}_{_{TMD_Y/P_N}} - F_{StopFrc_{Y}} - F_{ext_y} - F_{Y_{_{TMD_X/O_N}}} - F_{Y_{_{TMD_Z/O_N}}} \\
            k_z {z}_{_{TMD_Z/P_N}} + c_z \dot{z}_{_{TMD_Z/P_N}} - F_{StopFrc_{Z}} - F_{ext_z} - F_{Z_{_{TMD_X/O_N}}} - F_{Z_{_{TMD_Y/O_N}}} - F_{Z_{PreLoad}}
         \end{array}
      \right]
   \end{aligned}
   :label: OutputForces

且

.. math::
   \vec{M}_{_{P_G}} = R^T_{_{N/G}} \left[
      \begin{array}{c}
         M_{_X} \\
         M_{_Y} \\
         M_{_Z}
      \end{array}
   \right]_{_{N/N}} = R^T_{_{N/G}} \left[
      \begin{array}{c}
         -(F_{Z_{_{TMD_Y/O_N}}}) y_{_{TMD/P_N}} + (F_{Y_{_{TMD_Z/O_N}}} ) z_{_{TMD/P_N}} \\
          (F_{Z_{_{TMD_X/O_N}}}) x_{_{TMD/P_N}} - (F_{X_{_{TMD_Z/O_N}}} ) z_{_{TMD/P_N}} \\
         -(F_{Y_{_{TMD_X/O_N}}}) x_{_{TMD/P_N}} + ( F_{X_{_{TMD_Y/O_N}}}) y_{_{TMD/P_N}}
      \end{array}
   \right]

止动力
~~~~~~~~~~~

当 TMD_X、TMD_Y 或 TMD_Z 的运动超出质量块的最大轨道长度时，
额外的力 :math:`F_{StopFrc_{X}}`、:math:`F_{StopFrc_{Y}}` 和 :math:`F_{StopFrc_{Z}}`
被添加到输出力中。否则它们为零。
轨道长度在 TMD 方向的正端和负端有限制（X_PSP 和 X_NSP、Y_PSP 和 Y_NSP、
以及 Z_PSP 和 Z_NSP）。若定义通用的最大和最小位移分别为 :math:`x_{max}` 和
:math:`x_{min}`，则止动力具有以下形式

.. math::
   F_{StopFrc} = -\left\{
      \begin{array}{lr}
         \begin{aligned}
            k_S \Delta x  & \quad : ( x > x_{max} \wedge \dot{x}<=0) \vee ( x < x_{min} \wedge \dot{x}>=0)\\
            k_S \Delta x + c_S \dot{x} & \quad : ( x > x_{max} \wedge \dot{x}>0) \vee ( x < x_{min} \wedge \dot{x}<0)\\
            0 & \quad : \text{otherwise}
         \end{aligned}
      \end{array}
   \right.

其中 :math:`\Delta x` 是质量块超出止动位置的距离，:math:`k_S` 和 :math:`c_S` 是大刚度和阻尼常数。


.. _SrvD-StCz-PreLoad:

预载力
~~~~~~~~~~~~~~~

额外的力 :math:`F_{Z_{PreLoad}}` 被添加到输出力中，作为一种在重力作用时
偏移 TMD_Z 静止位置的方法。这对于子结构安装的 StC 特别有用，
当使用非常大的质量和软弹簧常数时。此项出现在 :math:`\vec{F}_{_{TMD_Z/O_N}}` 项中，
以及由 :eq:`Bu` 给出的运动方程和 :eq:`OutputForces` 中的合力中。


代码修改
==================

结构控制（StC）功能是链接到 ServoDyn 中的一个子模块。除了
ServoDyn.f90 和 ServoDyn.txt 中的引用外，包含 StC 模块的新文件列于下方。

新增文件
---------

-  StrucCtrl.f90：结构控制模块

-  StrucCtrl.txt：注册表文件，包括表 `1 <#tbl2>`__ 和 `2 <#tbl1>`__ 中的
   文件、输入、状态、参数和输出

-  StrucCtrl_Types.f90：自动生成

变量
---------

.. container::
   :name: tbl2

   .. table:: StC 注册表中字段定义摘要。注意状态向量 :math:`\vec{tmd_x}` 对应 :math:`\vec{R}_{_{TMD/P_N}}`，且输出 :math:`\vec{F}_{_{P_G}}` 和 :math:`\vec{M}_{_{P_G}}` 包含在 MeshType 对象（y.Mesh）中。:math:`X_{DSP}`、:math:`Y_{DSP}` 和 :math:`Z_{DSP}` 是 TMD 的初始位移。

      +----------------------+------------------------------------------------------------------------------+
      + 数据类型             + 变量名                                                                       +
      +======================+==============================================================================+
      | **InitInput**        |                                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | InputFile                                                                    |
      +----------------------+------------------------------------------------------------------------------+
      |                      | Gravity                                                                      |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\vec{r}_{_{N/O_G}}`                                                   |
      +----------------------+------------------------------------------------------------------------------+
      | **Input u**          |                                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\ddot{\vec{r}}_{_{P/O_G}}`                                            |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\vec{R}_{_{N/O_G}}`                                                   |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\vec{\omega}_{_{N/O_G}}`                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\vec{\alpha}_{_{N/O_G}}`                                              |
      +----------------------+------------------------------------------------------------------------------+
      | **Parameter p**      |                                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`m_x`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`c_x`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`k_x`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`m_y`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`c_y`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`k_y`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`m_z`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`c_z`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`k_z`                                                                  |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`K_S = \left[ k_{SX}\hspace{1em}k_{SY}\hspace{1em}k_{SZ}\right]`       |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`C_S = \left[c_{SX}\hspace{1em}c_{SY}\hspace{1em}c_{SZ}\right]`        |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`P_{SP}=\left[X_{PSP}\hspace{1em}Y_{PSP}\hspace{1em}Z_{PSP}\right]`    |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`P_{SP}=\left[X_{NSP}\hspace{1em}Y_{NSP}\hspace{1em}Z_{NSP}\right]`    |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`F{ext}`                                                               |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`Gravity`                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | TMDX_DOF                                                                     |
      +----------------------+------------------------------------------------------------------------------+
      |                      | TMDY_DOF                                                                     |
      +----------------------+------------------------------------------------------------------------------+
      |                      | TMDZ_DOF                                                                     |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`X_{DSP}`                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`Y_{DSP}`                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`Z_{DSP}`                                                              |
      +----------------------+------------------------------------------------------------------------------+
      | **State x**          |                                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | :math:`\vec{tmd_x}`                                                          |
      +----------------------+------------------------------------------------------------------------------+
      | **Output y**         |                                                                              |
      +----------------------+------------------------------------------------------------------------------+
      |                      | Mesh                                                                         |
      +----------------------+------------------------------------------------------------------------------+


输入、参数、状态和输出定义汇总于
表 `1 <#tbl2>`__。来自文件的输入列于
表 `2 <#tbl1>`__。

.. container::
   :name: tbl1

   .. table:: 从 TMDInputFile 读取的数据。

      +------------+------------+------------------------------------------------------+
      | 字段名     | 字段类型   | 描述                                                 |
      +============+============+======================================================+
      | TMD_CMODE  | int        | 控制模式（1: 被动, 2: 主动）                         |
      +------------+------------+------------------------------------------------------+
      | TMD_X_DOF  | logical    | DOF 开启或关闭                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_DOF  | logical    | DOF 开启或关闭                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_DOF  | logical    | DOF 开启或关闭                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_X_DSP  | real       | TMD_X 初始位移                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_DSP  | real       | TMD_Y 初始位移                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_DSP  | real       | TMD_Z 初始位移                                       |
      +------------+------------+------------------------------------------------------+
      | TMD_X_M    | real       | TMD 质量                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_X_K    | real       | TMD 刚度                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_X_C    | real       | TMD 阻尼                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_M    | real       | TMD 质量                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_K    | real       | TMD 刚度                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_C    | real       | TMD 阻尼                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_M    | real       | TMD 质量                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_K    | real       | TMD 刚度                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_C    | real       | TMD 阻尼                                             |
      +------------+------------+------------------------------------------------------+
      | TMD_X_PSP  | real       | 正向止动位置（X 质量块最大位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_X_NSP  | real       | 负向止动位置（X 质量块最小位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_X_K_SX | real       | 止动弹簧刚度                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_X_C_SX | real       | 止动弹簧阻尼                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_PSP  | real       | 正向止动位置（Y 质量块最大位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_NSP  | real       | 负向止动位置（Y 质量块最小位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_K_S  | real       | 止动弹簧刚度                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_Y_C_S  | real       | 止动弹簧阻尼                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_PSP  | real       | 正向止动位置（Z 质量块最大位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_NSP  | real       | 负向止动位置（Z 质量块最小位移）                     |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_K_S  | real       | 止动弹簧刚度                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_Z_C_S  | real       | 止动弹簧阻尼                                         |
      +------------+------------+------------------------------------------------------+
      | TMD_P_X    | real       | 机舱坐标系中 P 的 x 原点                             |
      +------------+------------+------------------------------------------------------+
      | TMD_P_Y    | real       | 机舱坐标系中 P 的 y 原点                             |
      +------------+------------+------------------------------------------------------+
      | TMD_P_Z    | real       | 机舱坐标系中 P 的 z 原点                             |
      +------------+------------+------------------------------------------------------+

致谢
================

作者感谢 Jason Jonkman 博士审阅本手册。

.. [1]
   注意 :math:`( R a ) \times ( Rb ) = R( a \times b )`。
