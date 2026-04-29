
.. _AA-noise-models:

气动噪声模型
--------------

风力机转子的气动噪声源于沿叶片产生的压力振荡并在大气中传播。历史上，这类噪声的仿真采用了不同保真度水平的模型。在低保真度层面，模型将气动噪声与转子推力和扭矩相关联 (:cite:`aa-Lowson:1970,aa-Viterna:1981`)。在高保真度层面，三维不可压缩计算流体动力学模型与 Ffowcs Williams-Hawkings 模型耦合，将沿转子叶片表面产生的压力振荡传播到远场 (:cite:`aa-Klein:2018`)。后一类模型通常仅适用于估算低频噪声，因为要捕获通常定义在 20 赫兹（Hz）到 20 千赫兹（kHz）之间的可听范围内的噪声，需要非常精细的时空离散化，计算成本极高。

对于可听范围，公共领域有多种模型可用，:cite:`aa-Sucameli:2018` 提供了最新的文献综述。这些模型的输入与现代气动伺服弹性求解器（如 OpenFAST）的输入输出相匹配，因此经常被耦合在一起。此外，这些声学模型的计算成本与现代气动伺服弹性求解器的成本相近，这也促进了耦合。

根据 :cite:`aa-Brooks:1989` 定义的分类，模型针对不同的噪声产生机制，以及湍流来流噪声机制。后者代表一种宽带噪声源，当任意形状的物体由于入射湍流的存在而经历非定常升力时产生。对于翼型，这种现象可以解释为前缘噪声。湍流来流噪声是过去几十年中多项研究的主题，因此已经发表了多个模型 (:cite:`aa-Sucameli:2018`)。BPM 模型包括流动中翼型的五种噪声产生机制：

1. 湍流边界层-后缘（TBL-TE）
2. 分离失速
3. 层流边界层-涡脱落
4. 叶尖涡
5. 后缘钝度-涡脱落

对于这五种机制，最初为 NACA 0012 翼型定义了半经验模型。BPM 模型仍然是风力机噪声预测的常用模型，后续的研究通过去除最初采用的一些假设对模型进行了改进。最近的研究尤其关注 TBL-TE 机制，这通常是现代风力机的主要噪声源。因此，BPM 模型中定义的每个噪声源现在都有多种变体。

以下小节描述了每种机制的细节以及在 OpenFAST 模型中实现的模型。

.. _aa-turbinflow:

湍流来流
~~~~~~~~

任意形状的物体浸没在湍流中时，会产生表面压力脉动。多年来，已经开发了多种湍流来流噪声模型的公式 (:cite:`aa-Sucameli:2018`)。在 OpenFAST 的这个模型中，采用了 :cite:`aa-MoriartyGuidatiMigliore:2004` 中定义的公式。该公式基于 Amiet 的模型 (:cite:`aa-Amiet:1975,aa-Paterson:1976`)，并在 :numref:`aa-amiet` 中介绍。此外，用户可以激活 :cite:`aa-MoriartyHansen:2005` 定义的修正，该修正建立在 Amiet 模型的基础上，考虑了沿叶片展向采用的翼型厚度。第二个模型名为简化 Guidati，在 :numref:`aa-guidati` 中介绍。

.. _aa-amiet:

Amiet 模型
^^^^^^^^^^^

该公式基于 :cite:`aa-Amiet:1975` 和 :cite:`aa-Paterson:1976` 的工作，将叶片表示为平板，忽略翼型的形状。

模型首先计算给定频率 :math:`f` 对应的波数 :math:`k_{1}`：

.. math::
   k_{1} = \frac{2\text{πf}}{U_{1}}
   :label:  aa-eq:1

其中 :math:`U_{1}` 是翼型的入射来流速度。从 :math:`k_{1}` 计算波数 :math:`{\overline{k}}_{1}` 和 :math:`{\widehat{k}}_{1}`：

.. math::
   {\overline{k}}_{1} = \frac{k_{1}c_{i}}{2}
   :label:  aa-eq:2

.. math::
   {\widehat{k}}_{1} = \frac{k_{1}}{k_{e}}
   :label:  aa-eq:3

其中 :math:`c_{i}` 是当地弦长，:math:`k_{e}` 是含能涡的波数范围，定义为：

.. math::
   k_{e} = \frac{3}{4L_{t}}.
   :label:  aa-eq:4

:math:`L_{t}` 是湍流长度尺度，多年来已经提出了多种不同的公式。作为默认实现，:math:`L_{t}` 按照 :cite:`aa-Zhu:2005` 中提出的公式定义：

.. math::
   L_{t} = 25z^{0.35}z_{0}^{- 0.063}
   :label:  aa-eq:5

其中 :math:`z` 是给定时刻 :math:`t` 下截面 :math:`i` 前缘的地面以上高度，而 :math:`z_{0}` 是表面粗糙度。注意，适当设置 :math:`L_{t}` 是一个挑战，该模型的高级用户可能希望根据实验数据验证这个公式。

声压级（:math:`\text{SPL}`）的值以给定频率 :math:`f` 下的 1/3 倍频程表示，产生于给定叶片站位 :math:`i`，计算方式为：

.. math::
   \text{SPL}_{\text{TI}} = 10\log_{10}{\left( \rho^{2}c^{4}\frac{L_{t}d}{{2r}_{e}^{2}}M^{5}I_{1}^{2}
      \frac{{\widehat{k}}_{1}^{3}}{\left( 1 + {\widehat{k}}_{1}^{2} \right)^{\frac{7}{3}}}
      \overline{D} \right) +}78.4
   :label:  aa-eq:6

其中 :math:`\rho` 是空气密度，:math:`c` 是声速，:math:`d` 是叶片单元展长，:math:`r_{e}` 是前缘与观察者之间的有效距离，:math:`M` 是马赫数，:math:`I_{1}` 是翼型来流的湍流强度，:math:`\overline{D}` 是指向性项。:math:`\overline{D}` 在某个称为"截止"的频率以下（:math:`{\overline{D}}_{l}`）和以上（:math:`{\overline{D}}_{h}`）是不同的，截止频率定义为：

.. math::
   f_{\text{co}} = \frac{10U_{1}}{\pi c_{i}}.
   :label:  aa-eq:7

:math:`{\overline{D}}_{h}` 和 :math:`{\overline{D}}_{l}` 的公式在 :numref:`aa-directivity` 中介绍。

当前实现提供了两种估算 :math:`I_{1}` 的方法。第一种是通过用户定义的 :math:`I_{1}`。第二种选择是让代码从湍流风网格重构 :math:`I_{1}`，其中代码计算每个时刻每个叶片截面 :math:`i` 的翼型相对位置，并根据转子转速重构湍流强度的来流分量 :math:`I_{1}`。

该模型还实现了两种修正。第一种包括攻角 :math:`\alpha` 的修正，在 :cite:`aa-Amiet:1975` 和 Amiet 和 Peterson (1976) 的原始公式中忽略了这个效应。该修正的公式为：

.. math::
   \text{SPL}_{\text{TI}} = \text{SPL}_{\text{TI}} + 10\log_{10}{\left( 1 + 9a^{2} \right).}
   :label:  aa-eq:8

第二种修正称为低频修正（:math:`\text{LFC}`），公式为：

.. math::
   S^{2} = \left( \frac{2\pi{\overline{k}}_{1}}{\beta^{2}}
      + \left( 1 + 2.4\frac{{\overline{k}}_{1}}{\beta^{2}} \right)^{- 1} \right)^{- 1}
   :label:  aa-eq:9
.. math::
   LFC = 10S^{2}M{\overline{k}}_{1}^{2}\beta^{- 2}
   :label:  aa-eq:10
.. math::
   \text{SPL}_{\text{TI}} = \text{SPL}_{\text{TI}} + 10\log_{10}\left( \frac{\text{LFC}}{1 + LFC} \right).
   :label:  aa-eq:11

在 :eq:`aa-eq:9` 和 :eq:`aa-eq:10` 中，:math:`S^{2}` 表示西尔斯函数的平方，:math:`\beta^{2}` 是 Prandtl-Glauert 修正因子，定义为：

.. math::
   \beta^{2} = 1 - M^{2}.
   :label:  aa-eq:12

需要强调的是，存在许多湍流来流噪声模型的替代公式 (:cite:`aa-Sucameli:2018`)，主要区别包括 :math:`L_{t}` 和 :math:`k_{1}` 的不同定义。

.. _aa-guidati:

简化 Guidati
^^^^^^^^^^^^^

这里实现的 Amiet 模型通常会高估声谱。Guidatai (:cite:`aa-Guidati:1997`) 通过添加一个考虑翼型剖面形状和弯度的项，导出了对声压级的修正，但该方法对于风力机仿真来说计算成本过高。Moriarty 等人 (:cite:`aa-MoriartyGuidatiMigliore:2005`) 基于六个风力机翼型的几何特征提出了一个简化模型。该修正的有效性仅限于马赫数约 0.1 ≈ 0.2，且斯特劳哈尔数 :math:`\text{St}` 低于 75 的范围。:math:`\text{St}` 基于翼型弦长和平均来流速度定义：

.. math::
   St = \frac{fc_{i}}{U_{1}}.
   :label:  aa-eq:13

噪声谱的修正公式在 :cite:`aa-MoriartyGuidatiMigliore:2005` 的公式 4 中给出：

.. math::
   t = t_{1\%} + t_{10\%}
   :label:  aa-eq:14
.. math::
   {\mathrm{\Delta}SPL}_{\text{TI}} = -\left( 1.123t + 5.317t^{2} \right)\left( 2\pi St + 5 \right)
   :label:  aa-eq:15

其中 :math:`t_{x\%}` 是沿弦向 :math:`x` 位置处剖面的相对厚度（即 0% 为前缘，100% 为后缘）。

需要强调的是，曾在风洞中对二维翼型进行过验证实验 (:cite:`aa-MoriartyGuidatiMigliore:2004`)，结果显示简化 Guidati 模型与实验结果的匹配度相当差。因此，提出了在整个频率谱上对 SPL 水平进行 +10 分贝（dB）的修正。这个修正仍然在实现中，但需要在风力机层面进行验证，以评估湍流来流模型的准确性。还需要注意的是，代码目前不会检查马赫数和斯特劳哈尔数是否在该模型的有效范围内。

.. _aa-turb-TE:

湍流边界层-后缘
~~~~~~~~~~~~~~~~

浸没在流动中的翼型会形成边界层，在高雷诺数下边界层是湍流的。当湍流经过后缘时，会产生噪声。这个噪声源在 :cite:`aa-Brooks:1989` 中被命名为 TBL-TE，是现代风力机转子相关的气动噪声源。代码中实现了两种 TBL-TE 噪声公式：(1) BPM 模型的原始公式，在 :numref:`aa-amiet` 中描述；(2) 荷兰研究机构 TNO 开发的更近期的模型，在 :numref:`aa-guidati` 中描述。两个模型都将翼型边界层的特征作为输入。这些必须由用户提供，在 :numref:`aa-sec-BLinputs` 中讨论。

.. _aa-turb-TE-bpm:

BPM
^^^

BPM 模型中 TBL-TE 噪声的 :math:`\text{SPL}` 由三个贡献组成：

.. math::
   \text{SPL}_{TBL - TE} = 10\log_{10}\left( 10^{\frac{\text{SPL}_{p}}{10}}
      + 10^{\frac{\text{SPL}_{s}}{10}} + 10^{\frac{\text{SPL}_{\alpha}}{10}} \right)
   :label:  aa-eq:16

其中下标 :sub:`p`、:sub:`s` 和 :sub:`α` 分别指压力面、吸力面和攻角的贡献。描述这三个贡献的方程在 :cite:`aa-Brooks:1989` 的第 5.1.2 节中有详细描述，这里进行总结。

对于吸力面和压力面的贡献，方程为：

.. math::
   \text{SPL}_{p} = 10\log_{10}\left( \frac{\delta_{p}^{*}M^{5}d{\overline{D}}_{h}}{r_{e}^{2}} \right)
      + A\left( \frac{\text{St}_{p}}{\text{St}_{1}}\right) + \left( K_{1} - 3 \right) + {\mathrm{\Delta}K}_{1}
   :label:  aa-eq:17
.. math::
   \text{SPL}_{s} = 10\log_{10}\left( \frac{\delta_{s}^{*}M^{5}d{\overline{D}}_{h}}{r_{e}^{2}} \right)
      + A\left( \frac{\text{St}_{s}}{\text{St}_{1}} \right) + \left( K_{1} - 3 \right).
   :label:  aa-eq:18

方程中的术语也在本文档开头的命名法中描述，其中 :math:`\delta^{*}` 是翼型任一侧的边界层位移厚度，:math:`St` 是基于 :math:`\delta^{*}` 的斯特劳哈尔数，:math:`A`、:math:`A'`、:math:`B`、:math:`{\Delta K}_{1}`、:math:`K_{1}` 和 :math:`K_{2}` 是基于 :math:`\text{St}` 的经验函数。

对于攻角贡献，在失速角上下有所区别，在原始 BPM 模型中失速角设置为 12.5 度，而在这里假设为叶片站位 i 处翼型的实际失速攻角。在失速以下，:math:`\text{SPL}_{\alpha}` 等于：

.. math::
   \text{SPL}_{\alpha} = 10\log_{10}\left( \frac{\delta_{s}^{*}M^{5}d{\overline{D}}_{h}}{r_{e}^{2}} \right)
      + B\left( \frac{\text{St}_{s}}{\text{St}_{2}} \right) + K_{2}.
   :label:  aa-eq:19

在攻角高于失速点时，沿剖面的流动完全分离，噪声从整个弦长辐射。此时 :math:`\text{SPL}_{p}` 和 :math:`\text{SPL}_{s}` 设置为 -∞，而 :math:`\text{SPL}_{\alpha}` 变为：

.. math::
   \text{SPL}_{\alpha} = 10\log_{10}\left( \frac{\delta_{s}^{*}M^{5}d{\overline{D}}_{l}}{r_{e}^{2}} \right)
      + A'\left( \frac{\text{St}_{s}}{\text{St}_{2}} \right) + K_{2.}
   :label:  aa-eq:20

值得注意的是，在失速以上，方程 18 和 19 中采用了低频指向性 :math:`{\overline{D}}_{l}`（见 :numref:`aa-directivity`）。

.. _aa-turb-TE-tno:

TNO 模型
^^^^^^^^^

TNO 模型是一个更新的模型，用于仿真叶片后缘脱落涡产生的噪声，由 Parchen 提出 (:cite:`aa-Parchen:1998`)。这里采用的实现是 Moriarty 等人 (2005) 中描述的版本。TNO 模型使用非定常表面压力的波数 :math:`\overline{k}` 谱来估算远场噪声。谱 :math:`P` 假设为：

.. math::
   P\left( k_{1},k_{3},\omega \right) = 4\rho_{0}^{2}\frac{k_{1}^{2}}{k_{1}^{2}
   + k_{3}^{2}}\int_{0}^{10\frac{\omega}{Mc}}{L_{2}\overline{u_{2}^{2}}
   \left( \frac{\partial U_{1}}{\partial x_{2}} \right)^{2}
   \phi_{22}\left( k_{1},k_{3},\omega \right)} \\
   \phi_{m}\left( \omega - U_{c}\left( x_{2} \right)k_{1} \right)
   e^{\left( - 2\left| \overline{k} \right|x_{2} \right)}dx_{2}.
   :label:  aa-eq:21

在这个方程中，索引 1、2 和 3 分别指平行于翼型弦长、垂直于翼型弦长和沿展向的方向；:math:`\phi_{22}` 是垂直速度脉动谱；:math:`\phi_{m}` 是运动轴谱；:math:`U_{c}` 是涡沿后缘的对流速度。最后，:math:`L_{2}` 是垂直于弦长的垂直相关长度，表示在后缘对流的涡的垂直延伸。在本工作中，:math:`L_{2}` 假设等于混合长度 :math:`L_{m}` (Moriarty 等人 2005)。这个决定有一定的任意性，应该通过专门的研究更好地评估 TNO 模型中采用的正确积分长度。

从 :math:`P` 计算远场谱 :math:`S\left( \omega \right)`：

.. math::
   S\left( \omega \right) = \frac{d{\overline{D}}_{h}}{4\pi r_{e}^{2}}\int_{0}^{\delta}
   {\frac{\omega}{ck_{1}}P\left( k_{1},0,\omega \right)}\text{dk}_{1}.
   :label:  aa-eq:22

TNO 模型的实现与 :cite:`aa-MoriartyGuidatiMigliore:2005` 中描述的完全相同。模型的输入由用户提供的边界层特征生成（见 :numref:`aa-sec-BLinputs`）。

.. _aa-laminar-vortex:

层流边界层-涡脱落
~~~~~~~~~~~~~~~~~~

BPM 模型中包含的另一个翼型自噪声源是后缘脱落的涡与层流边界层中的不稳定波之间的反馈回路产生的噪声。这种噪声通常分布在窄频带内，当翼型的边界层保持层流时发生。这可能发生在小型风力机的内侧区域，那里的雷诺数可能小于 100 万，但在运行雷诺数大一个数量级的现代转子中几乎不会发生。以 1/3 倍频程表示的噪声谱估算公式为：

.. math::
   \text{SPL}_{LBL - VS} = 10\log_{10}{
      \left( \frac{\delta_{p}M^{5}d{\overline{D}}_{h}}{r_{e}^{2}} \right)
      + G_{1}\left( \frac{St'}{{St'}_{\text{peak}}} \right) \\
      + G_{2}\left\lbrack \frac{\text{Re}_{c}}{\left( \text{Re}_{c} \right)_{0}} \right\rbrack
      + G_{3}\left( \alpha_{*} \right)}
   :label:  aa-eq:23

其中 :math:`G` 表示经验函数，:math:`{St'}_{\text{peak}}` 是 :math:`\text{Re}_{c}` 的峰值斯特劳哈尔数函数，:math:`\text{Re}_{c}` 是弦长 :math:`c_{i}` 处的雷诺数。下标 :sub:`0` 指参考雷诺数，是攻角的函数（Brooks 等人 1989）。

.. _aa-tip-vortex:

叶尖涡
~~~~~~

叶尖产生的涡是 BPM 模型的另一个噪声源。虽然在现代风力机中很少相关，但提供了包含这个噪声源的可能性。声压级估算为：

.. math::
   \text{SPL}_{\text{Tip}} = 10\log_{10}{\left(
      \frac{M^{2}M_{\max}^{2}l^{2}{\overline{D}}_{h}}{r_{e}^{2}} \right)
      - 30.5\left( \log_{10}{St^{''}} + 0.3 \right)^{2} + 126}
   :label:  aa-eq:24

其中 :math:`M_{\max}\  = \ M_{\max}\left( \alpha_{\text{tip}} \right)` 是最大马赫数，在叶尖附近的分离流区域测量，假设取决于叶尖处的攻角 :math:`\alpha_{\text{tip}}`；:math:`l` 是分离区的展向范围；:math:`St'''` 是基于 :math:`l` 的斯特劳哈尔数。对于圆形叶尖形状，:math:`l` 估算为：

.. math::
   l = c_{i}0.008\alpha_{\text{tip}}
   :label:  aa-eq:25

其中 :math:`\alpha_{\text{tip}}` 是叶尖区域相对于来流的攻角。对于方形叶尖，BPM 模型基于 :math:`{\alpha'}_{\text{tip}}` 估算 :math:`l`，其定义为：

.. math::
   \left. \ {\alpha^{'}}_{\text{tip}} = \left\lbrack \left(
      \frac{\frac{\partial L'}{\partial y}}{\left(
         \frac{\partial L'}{\partial y} \right)_{\text{ref}}}
      \right)_{y\rightarrow tip}
         \right\rbrack \right.\ \alpha_{\text{tip}}
   :label:  aa-eq:26

其中 :math:`L'` 是沿叶片位置 :math:`y` 处的单位展长升力。当 :math:`{\alpha'}_{\text{tip}}` 在 0 到 2 度之间时，:math:`l` 变为：

.. math::
   l = c_{i}\left( 0.0230 + 0.0169{\alpha^{'}}_{\text{tip}} \right),
   :label:  aa-eq:27

而当 :math:`{\alpha'}_{\text{tip}}` 大于 2 度时，:math:`l` 为：

.. math::
   l = c_{i}\left( 0.0378 + 0.0095{\alpha^{'}}_{\text{tip}} \right).
   :label:  aa-eq:28

然而，必须注意的是，不幸的是，:math:`\alpha_{\text{tip}}` 不是标准气动弹性模型的可靠输出，无法准确确定 :math:`\alpha_{\text{tip}}` 削弱了叶尖涡噪声公式的有效性。

.. _aa-TE-vortex:

后缘钝度-涡脱落
~~~~~~~~~~~~~~~~

最后，风力机叶片通常具有有限高度的后缘，这会由于涡脱落产生噪声。这个噪声源的频率和幅度取决于后缘的几何形状，通常具有音调特性。在叶片外侧采用平背和截断翼型可能会增强这个噪声源。当激活这个噪声源时，用户需要提供沿叶片展向的后缘钝厚度 :math:`h` 分布，以及翼型吸力面和压力面之间的立体角 :math:`\Psi`（见 :numref:`aa-turb-TE`）。:math:`h` 和 :math:`\Psi` 是方程的输入：

.. math::
   \text{SPL}_{TEB - VS} = 10\log_{10}{
      \left( \frac{\delta_{p}^{*}M^{5}d{\overline{D}}_{h}}{r_{e}^{2}} \right)
      + G_{4}\left( \frac{h}{\delta_{\text{avg}}^{*}},\Psi \right) \\
      + G_{5}\left( \frac{h}{\delta_{\text{avg}}^{*}},\Psi,
         \frac{St''}{{St''}_{\text{peak}}} \right)}.
   :label:  aa-eq:29

在方程中，:math:`\delta_{\text{avg}}^{*}` 是翼型两侧的平均位移厚度。注意，这个噪声源对 :math:`h` 和 :math:`\Psi` 非常敏感，因此应该准确估算。

.. _aa-directivity:

指向性
~~~~~~~

一个或多个观察者的位置由用户指定，如 :numref:`aa-sec-ObsPos` 中所述。本实现采用了 BPM 模型的指向性 (:cite:`aa-Brooks:1989`)。指向性项 :math:`\overline{D}` 根据观察者相对于发射源的相对位置修正 :math:`\text{SPL}`。位置由展向指向性角 :math:`\Phi_{e}` 和弦向指向性角 :math:`\Theta_{e}` 描述，这两个角度在 :numref:`aa-fig:directivity` 中示意性表示，定义为：

.. math::
   \Phi_{e} = \text{atan}\left( \frac{z_{e}}{y_{e}} \right)
   :label:  aa-eq:30
.. math::
   \Theta_{e} = \text{atan}\left( \frac{y_{e} \bullet \cos\left( \Phi_{e} \right)
      + z_{e} \bullet \sin\left( \Phi_{e} \right)}{x_{e}} \right)
   :label:  aa-eq:31


.. figure:: media/NoiseN002.jpeg
   :alt:    指向性函数中使用的角度
   :name:   aa-fig:directivity
   :width:  100.0%

   指向性函数中使用的角度 (:cite:`aa-Brooks:1989,aa-MoriartyMigliore:2003`)

参考轴位于每个叶片节点处，:math:`x_{e}` 与弦长对齐，:math:`y_{e}` 与指向叶尖的展向对齐，:math:`z_{e}` 朝向翼型吸力面对齐。注意，在 OpenFAST 中使用局部翼型定向参考系，并应用了旋转。

给定角度 :math:`\Theta_{e}` 和 :math:`\Phi_{e}`，在高频下，后缘的 :math:`\overline{D}` 表达式为：

.. math::
   {\overline{D}}_{h-TE}\left( \Theta_{e},\Phi_{e} \right) = \frac{
      2\sin^{2}\left( \frac{\Theta_{e}}{2} \right)\sin^{2}\Phi_{e}}
      {\left( 1 + M\cos\Theta_{e} \right)
         \left( 1 + \left( M - M_{c} \right)
         \cos\Theta_{e} \right)^{2}}
   :label:  aa-eq:32

其中 :math:`M_{c}` 表示后缘处的马赫数，这里为了简化假设等于自由流 M 的 80%。

对于前缘，因此对于湍流来流噪声模型，在高频下，:math:`\overline{D}` 为：

.. math::
   {\overline{D}}_{h-LE}\left( \Theta_{e},\Phi_{e} \right) = \frac{
      2\cos^{2}\left( \frac{\Theta_{e}}{2} \right)\sin^{2}\Phi_{e}}
      {\left( 1 + M\cos\Theta_{e} \right)^{3}}
   :label:  aa-eq:33

注意，这个方程没有在 NLR 技术报告 NREL/TP-5000-75731 中报告！

在低频下，前缘和后缘的方程相同：

.. math::
   {\overline{D}}_{l}\left( \Theta_{e},\Phi_{e} \right) =
      \frac{\sin^{2}\left. \ \Theta_{e} \right.\ \sin^{2}\Phi_{e}}
      {\left( 1 + M\cos\Theta_{e} \right)^{4}}.
   :label:  aa-eq:34

每个模型区分低频和高频的不同阈值。对于 TI 噪声模型，低频和高频之间的转换基于 :math:`{\overline{k}}_{1}` 定义。对于 TBL-TE 噪声，模型的区别在于失速上下的转换，分别使用 :math:`\ {\overline{D}}_{h}` 和 :math:`{\overline{D}}_{l}`。

.. _aa-A-weighting:

A 计权
~~~~~~~

代码提供了通过 A 计权对气动输出进行加权的可能性，A 计权是一个实验系数，旨在考虑人耳对不同频率的灵敏度。A 权重 :math:`A_{w}` 计算为：

.. math::
   A_{w} = \frac{10\log\left( 1.562339\frac{f^{4}}
         {\left( f^{2} + {107.65265}^{2} \right)
            \left( f^{2} + {737.86223}^{2} \right)}
         \right)}{\log 10}\qquad\qquad\\
      + \frac{10\log\left( 2.422881e16\frac{f^{4}}
         {\left( f^{2} + {20.598997}^{2} \right)^{2}
            \left( f^{2} + {12194.22}^{2} \right)^{2}} \right)}
         {\log 10}
   :label:  aa-eq:35

A 计权是频率的函数，被添加到声压级的值中：

.. math::
   SPL_{A_{w}} = SPL + A_{w}
   :label:  aa-eq:36
