.. _OLAF-Theory:

OLAF理论
===========

本节详细介绍OLAF方法，概述其计算方法，然后简要解释其与OpenFAST的集成。


.. _sec:vorticityformulation:

引言 - 涡量公式
------------------------------------

无粘均匀流动在无非保守力情况下的涡量方程由公式:eq:`eq:vorticityconservationincompr`给出：

.. math::
   \begin{aligned}
   \frac{d\vec{\omega}}{dt} = \frac{\partial\vec{\omega}}{\partial{t}} + \underbrace{(\vec{u} \cdot \nabla)}_{\text{对流}}\vec{\omega} = \underbrace{(\vec{\omega}\cdot\nabla)\vec{u}}_{\text{应变}} +\underbrace{\nu\Delta\vec{\omega}}_{\text{扩散}}
   \end{aligned}
   :label: eq:vorticityconservationincompr


这里，:math:`\vec{\omega}`是涡量，:math:`\vec{u}`是速度，:math:`\nu`是粘度。在自由涡尾迹方法中，涡量方程用于描述尾迹涡量的演化。为了简化求解，引入了不同的近似，例如将涡量投影到离散数量的涡元素（这里是涡丝）上，并分别处理对流和扩散步骤，称为粘性分裂。该方法存在几个难点；特别是离散化需要对涡量场（或速度场）进行正则化，以确保平滑近似。

叶片对流体施加的力也可以用涡量公式表达。这些涡量附着在叶片上，具有与升力相关的环量。这里使用升力线公式来模拟附着涡量。

以下各节描述了已实现的自由涡代码的不同模型。

.. _sec:discretization:

离散化 - 投影
---------------------------

数值方法使用有限数量的状态来模拟连续的涡量分布。为此，涡量分布被投影到基函数上，这些基函数称为涡元素。这里使用涡丝作为表示涡量场的元素。涡丝由两个点界定，因此具有由这两个点形成的方向。一个涡管沿单位向量:math:`\vec{e}_x`取向，横截面积为:math:`dS`，长度为:math:`l`。它可以近似为沿相同方向取向的长度为:math:`l`的涡丝。涡管和涡丝的总涡量相同，关系为：

.. math::
   \begin{aligned}
       \vec{\omega} \, dS = \vec{\Gamma}
   \end{aligned}
   :label: OmegaGamma

其中:math:`\vec{\Gamma}`是涡丝的环量强度。如果涡管复杂且占据较大体积，则投影到涡丝上比较困难，投影到涡粒子上更合适。假设尾迹被限制在薄涡量层中，该层定义了已知方向的速度跳跃，则可以将尾迹涡量片近似为涡丝网。这是涡丝尾迹方法的基础。涡丝是涡量场的奇异表示，因为它们占据线而不是体积。为了更好地表示涡量场，涡丝被"膨胀"，这个过程称为正则化（见:numref:`sec:Regularization`）。涡量场的正则化也正则化了速度场，避免了否则会出现的奇异性。


.. _sec:circ:

升力线表示
---------------------------

代码依赖于升力线公式。升力线方法有效地将叶片每个截面的载荷集中到叶片的平均线上，而不直接考虑每个截面的几何形状。在基于涡量的升力线方法版本中，叶片由变化环量的线表示。该线跟随叶片的运动，称为"附着"环量。附着环量不遵循与尾迹自由涡量相同的动力学方程。相反，其强度通过库塔-儒可夫斯基定理与翼型升力相关。附着环量的展向变化导致涡量被释放到尾迹中。这称为"拖曳涡量"。附着环量的时间变化也会被释放到尾迹中，称为"脱落涡量"。以下段落描述了附着涡量的表示。

升力线面板和释放的尾迹面板
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

升力线和尾迹表示如:numref:`fig:VortexLatticeMethod`所示。叶片升力线被离散为有限数量的面板，每个面板形成一个四边涡环。展向离散化遵循AeroDyn叶片输入文件的离散化。展向面板数量:math:`n_\text{LL}`比AeroDyn节点总数**NumBlNds**少1。面板的边与升力线和叶片后缘重合。目前，升力线定义为距离前缘（LE）1/4弦长的位置。关于网格划分的更多细节见:numref:`sec:Panelling`。在给定时间步，每个升力线面板的环量根据:numref:`sec:CirculationMethods`中开发的三种方法之一确定。在时间步结束时，每个升力线面板的环量被释放到尾迹中，形成自由涡量面板。为了满足库塔条件，第一近尾迹面板的环量与附着环量相等（见:numref:`fig:VortexLatticeMethod` b）。尾迹面板模拟叶片边界层延续形成的薄剪切层。该剪切层可以使用涡偶极子的连续分布来模拟。假设每个面板上的偶极子强度恒定，这反过来等效于恒定环量的涡环。

.. figure:: Schematics/VortexLatticeMethod.png
   :alt: 离散为涡环面板的尾迹和升力线涡量。
   :name: fig:VortexLatticeMethod
   :width: 100.0%

   离散为涡环面板的尾迹和升力线涡量。
   （a）概述。（b）截面视图，定义前缘、后缘和升力线。（c）面板的环量以及面板之间涡段的对应环量。（d）升力线面板的几何量。

当前实现存储面板角点的位置和环量。在涡环公式中，两个面板之间的边界对应于强度等于两个面板之间环量差的涡段。基于面板强度定义段强度的约定如:numref:`fig:VortexLatticeMethod` c所示。由于附着面板的环量与第一排近尾迹面板的环量相等，位于后缘的涡段没有环量。

.. _sec:Panelling:

网格划分
~~~~~~~~~

叶片网格划分的定义见:numref:`fig:VortexLatticeMethod` d，遵循van Garrel的符号(:cite:`olaf-Garrel03_1`)。前缘和后缘（TE）位置直接从AeroDyn网格获得。在两个展向位置，LE和TE定义了角点：:math:`\vec{x}_1`、:math:`\vec{x}_2`、:math:`\vec{x}_3`和:math:`\vec{x}_4`。当前实现假设气动中心、升力线和1/4弦长位置都重合。对于给定面板，升力线由点:math:`\vec{x}_9= 3/4\,\vec{x}_1 + 1/4\, \vec{x}_2`和:math:`\vec{x}_{10}=3/4\,\vec{x}_4 + 1/4\, \vec{x}_3`界定。四个面板边的中点记为:math:`\vec{x}_5`、:math:`\vec{x}_6`、:math:`\vec{x}_7`和:math:`\vec{x}_8`。升力线向量(:math:`\vec{dl}`)以及面板的切向(:math:`\vec{T}`)和法向(:math:`\vec{N}`)向量定义为：

.. math::
   \begin{aligned}
       \vec{dl} = \vec{x}_{10}-\vec{x}_9
      ,\qquad
      \vec{T}  = \frac{\vec{x}_6-\vec{x}_8}{|\vec{x}_6-\vec{x}_8|}
      ,\qquad
      \vec{N}  = \frac{\vec{T}\times\vec{dl}}{|\vec{T}\times\vec{dl}|}
   \end{aligned}
   :label: eq:GeometricDefinitions

面板的面积为:math:`dA = |(\vec{x}_6-\vec{x}_8)\times(\vec{x}_{7}-\vec{x}_5)|`。对于**CircSolvMethod=[1]**，控制点位于升力线上的:math:`\vec{x}_9+\eta_j \vec{dl}`位置。因子:math:`\eta_j`基于van Garrel的全余弦近似确定。这基于当前面板的展向宽度:math:`w_j`以及相邻面板的:math:`w_{j-1}`和:math:`w_{j+1}`：

.. math::
   \begin{aligned}
      \eta_1 &= \frac{w_1}{w_1+w_2},\\
      \eta_j &= \frac{1}{4}\left[\frac{w_{j-1}}{w_{j-1}+w_j} + \frac{w_j}{w_j+w_{j+1}} +1 \right]
      ,\ j=2..n-1,\\
      \eta_{n} &= \frac{w_{n-1}}{w_{n-1}+w_{n}}
   \end{aligned}

对于等间距，这种离散化将控制点放置在升力线的中间(:math:`\eta=0.5`)。这种离散化可以得到余弦间距椭圆翼的理论环量结果，因为它将控制点放置在更靠近翼端较强拖曳段的位置（参见例如:cite:`olaf-Kerwin:lecturenotes`）。

.. _sec:CirculationMethods:

环量求解方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~

已实现三种方法来确定附着环量强度。使用输入参数**CircSolvMethod**选择它们，以下各节介绍这些方法。

基于Cl的迭代方法
^^^^^^^^^^^^^^^^^^^^^^^^^

基于Cl的迭代方法在非线性迭代求解器中确定环量，该求解器使用升力线上每个控制点的极曲线数据。该算法确保使用攻角和极曲线数据获得的升力与使用库塔-儒可夫斯基定理获得的升力匹配。目前，这是计算叶片展向环量的首选方法。使用**CircSolvMethod=[1]**选择该方法。该方法在van Garrel的工作中描述(:cite:`olaf-Garrel03_1`)。该算法采用迭代方法实现，步骤如下：

1. 使用前一时间步的环量分布作为猜测环量:math:`\Gamma_\text{prev}`。

2. 计算每个控制点:math:`j`的速度，为风速、结构速度以及域内所有涡量在控制点位置评估的诱导速度之和。

   .. math::
      \begin{aligned}
          \vec{v}_j = \vec{V}_0 - \vec{V}_\text{elast} + \vec{v}_{\omega,\text{free}} + \vec{v}_{\Gamma_{ll}}
      \end{aligned}

   :math:`\vec{v}_{\omega,\text{free}}`是所有自由涡丝诱导的速度，如公式:eq:`eq:eq510`所述。:math:`\vec{v}_{\Gamma_{ll}}`的贡献来自升力线面板和第一排近尾迹面板，其环量设置为:math:`\Gamma_\text{prev}`。

3. 所有升力线面板:math:`j`的环量计算如下：

   .. math::
      \begin{aligned}
         \Gamma_{ll,j} =\frac{1}{2} C_{l,j}(\alpha_j) \frac{\left[ (\vec{v}_j \cdot \vec{N})^2 + (\vec{v}_j \cdot \vec{T})^2\right]\,dA}{
         \sqrt{\left[(\vec{v}_j\times \vec{dl})\cdot\vec{N}\right]^2 + \left[(\vec{v}_j\times \vec{dl})\cdot\vec{T}\right]^2}
         }
      ,\quad\text{其中}
      \quad
      \alpha_j = \operatorname{atan}\left(\frac{\vec{v}_j\cdot\vec{N}}{\vec{v}_j \cdot \vec{T}} \right)
      \end{aligned}

   函数:math:`C_{l,j}`是从叶片截面:math:`j`的极曲线数据获得的升力系数，:math:`\alpha_j`是控制点的攻角。

4. 使用松弛因子:math:`k_\text{relax}`（**CircSolvRelaxation**）设置新环量：

   .. math::
      \begin{aligned}
        \Gamma_\text{new}= \Gamma_\text{prev} + k_\text{relax} \Delta \Gamma
            ,\qquad
         \Delta \Gamma = \Gamma_{ll} - \Gamma_\text{prev}
      \end{aligned}

5. 使用准则:math:`k_\text{crit}`（**CircSolvConvCrit**）检查收敛性：

   .. math::
      \begin{aligned}
             \frac{ \operatorname{max}(|\Delta \Gamma|}{\operatorname{mean}(|\Gamma_\text{new}|)} < k_\text{crit}
       \end{aligned}

   如果未达到收敛，使用:math:`\Gamma_\text{new}`作为猜测环量:math:`\Gamma_\text{prev}`重复步骤2-5。

无穿流法
^^^^^^^^^^^^^^^^^^^^^^

无穿流环量求解方法（有时称为Weissinger-L基方法）可能会在未来实现(:cite:`olaf-Weissinger47_1,olaf-Bagai94_1,olaf-Gupta06_1,olaf-Ribera07_1`)。在该方法中，通过满足1/4弦点处的无穿流条件来求解环量。它将使用**CircSolvMethod=[2]**选择，但目前尚未实现。

预设环量
^^^^^^^^^^^^^^^^^^^^^^

最后一种可用方法是预设恒定环量。用户指定的展向环量分布被施加到叶片上。使用**CircSolvMethod=[3]**选择该方法。


.. _sec:vortconv:

自由涡量对流
-------------------------

涡丝的运动控制方程由拉格朗日标记点的对流方程给出：

.. math::
   \frac{d\vec{r}}{dt}=\vec{V}(\vec{r},t)
   :label: VortFilCart

其中:math:`\vec{r}`是拉格朗日标记点的位置。拉格朗日标记点是涡丝的端点。涡丝的拉格朗日对流拉伸了涡丝，因此自动考虑了涡量方程中的应变。

目前，已实现四阶龙格-库塔（**IntMethod=[1]**）或一阶前向欧拉（**IntMethod=[5]**）方法来数值求解公式:eq:`VortFilCart`的左侧，得到涡丝位置。对于一阶欧拉方法，对流简化为公式:eq:`eq:Euler`：

.. math::
   \vec{r} = \vec{r} + \vec{V} \Delta t
   :label: eq:Euler


.. _sec:vortconvPolar:

极坐标系下的自由涡量对流
----------------------------------------------

涡丝的运动控制方程为：

.. math::
   \frac{d\vec{r}(\psi,\zeta)}{dt}=\vec{V}[\vec{r}(\psi,\zeta),t]
   :label: VortFil

使用链式法则，公式:eq:`VortFil`重写为：

.. math::
   \frac{\partial\vec{r}(\psi,\zeta)}{\partial\psi}+\frac{\partial\vec{r}(\psi,\zeta)}{\partial\zeta}=\frac{\vec{V}[\vec{r}(\psi,\zeta),t]}{\Omega}
   :label: VortFil_expanded

其中:math:`d\psi/dt=\Omega`，:math:`d\psi=d\zeta` (:cite:`olaf-Leishman02_1`)。这里，:math:`\vec{r}(\psi,\zeta)`是拉格朗日标记点的位置向量，:math:`\vec{V}[\vec{r}(\psi,\zeta)]`是速度。


.. _sec:vortconvfrozen:

冻结涡量对流
---------------------------

为了提高计算效率，用户可以定义"冻结"近尾迹和远尾迹区域。在这些区域中，拉格朗日标记点使用公共诱导速度对流，该速度与标记点的径向位置无关，可能是尾迹年龄的函数。冻结区域中拉格朗日标记点的对流方程为：

.. math::
   \frac{d\vec{r}_\zeta}{dt}=\vec{V}_0(\vec{r}_\zeta,t) + \vec{V}_\text{avg}(t)*k(\zeta)

其中:math:`\vec{V}_\text{avg}(t)`是基于一部分"自由"标记点的对流速度计算的平均诱导速度。:math:`k(\zeta)`是基于尾迹年龄:math:`\zeta`的衰减因子，介于1和0之间。恒定衰减因子1将导致整个冻结尾迹的对流速度均匀。这是远尾迹使用的方式。对于近尾迹，典型值使得衰减因子在冻结尾迹开始处为1，在冻结尾迹结束处为0.5。事实上，当前验证表明，从:math:`k(0)=0.75`开始更好，否则在自由标记点子集上计算的平均对流速度会太低，冻结尾迹会更密集，导致在转子处的诱导更高。显然，平均速度及其衰减的选择是可能会在未来版本中改变的调优参数。这些参数目前不直接暴露在输入文件中。

一般来说，使用唯一的诱导速度对流整个"冻结"尾迹会引入一些误差，但大大减少了计算时间。拥有"冻结"远尾迹区域的优点是，它减轻了尾迹截断的影响，尾迹截断是一种错误的边界条件（涡线不能在流体中终止）。如果尾迹在仍然"自由"的情况下被截断，那么涡量会在该区域内自行卷起。另一个优点是，在没有扩散的情况下，尾迹在下游往往会变得过度扭曲，达到涡丝表示有效性的极限。




诱导速度和速度场
-----------------------------------

公式:eq:`VortFilCart`右侧的速度项是涡位置的非线性函数，代表来流和诱导速度的组合(:cite:`olaf-Hansen08_1`)。每个直涡丝在点:math:`\vec{x}`处引起的诱导速度使用毕奥-萨伐尔定律计算，该定律考虑了拉格朗日标记点的位置和涡元素的强度(:cite:`olaf-Leishman02_1`)：

.. math::
   d\vec{v}(\vec{x})=\frac{\Gamma}{4\pi}\frac{d\vec{l}\times\vec{r}}{r^3}
   :label: BiotSavart

这里，:math:`\Gamma`是涡丝的环量强度，:math:`\vec{dl}`是沿涡丝的微元长度，:math:`\vec{r}`是涡丝上一点到控制点:math:`\vec{x}`的向量，:math:`r=|\vec{r}|`是向量的模。沿由点:math:`\vec{x}_1`和:math:`\vec{x}_2`界定的涡丝长度积分毕奥-萨伐尔定律得到：

.. math::
   \begin{aligned}
     \vec{v}(\vec{x})
     =  F_\nu \frac{\Gamma}{4\pi} \frac{(r_1+r_2)}{r_1r_2(r_1r_2+\vec{r}_1\cdot\vec{r}_2)  }\vec{r}_1\times\vec{r}_2
   \end{aligned}
   :label: eq:BiotSavartSegment

其中:math:`\vec{r}_1= \vec{x}-\vec{x}_1`，:math:`\vec{r}_2= \vec{x}-\vec{x}_2`。因子:math:`F_\nu`是正则化参数，在:numref:`sec:RegularizationFunction`中讨论。:math:`r_0`是涡丝长度，其中:math:`\vec{r}_0= \vec{x}_2-\vec{x}_1`。到涡丝的正交距离为：

.. math::
   \begin{aligned}
      \rho = \frac{|\vec{r}_1\times\vec{r}_2|}{r_0}
   \end{aligned}

域内任意点的速度通过叠加所有涡丝诱导的速度，并叠加主流:math:`\vec{V}_0`（这里假设无散度）得到：

.. math::
   \begin{aligned}
    \vec{V}(\vec{x}) = \vec{V}_0(\vec{x}) + \vec{v}_\omega(\vec{x}), \quad\text{其中}\quad \vec{v}_\omega(\vec{x}) = \sum_{k} \vec{v}_k(\vec{x})
   \end{aligned}
   :label: eq:eq510

其中求和覆盖所有涡丝，每个涡丝的强度为:math:`\Gamma_k`。每个涡丝的强度由附着环量的展向和时间变化决定，如:numref:`sec:circ`所述。在基于树的方法中，通过将远离控制点的元素集中在一起来减少对所有涡元素的求和。


.. _sec:Regularization:

正则化
--------------

正则化和粘性扩散
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

公式:eq:`BiotSavart`中出现的奇异性极大地影响了涡方法的数值精度。通过正则化毕奥-萨伐尔定律的"1/r"核，可以得到收敛于纳维-斯托克斯方程的数值方法。与"真实"的连续涡量场相比，正则化用于提高离散涡量场的正则性。这种正则化通常通过与平滑函数卷积获得。在这种情况下，涡量场和速度场的正则化是相同的。一些工程模型也通过直接在毕奥-萨伐尔速度核的分母中引入额外项来执行正则化。公式:eq:`eq:BiotSavartSegment`中引入的因子:math:`F_\nu`就是为了考虑这种正则化。

在涡方法的收敛性证明中，正则化和粘性扩散是两个不同的方面。在涡丝方法中，通常将正则化概念与粘性扩散概念混淆。实际上，对于物理涡丝，粘性效应会阻止奇异性的发生，并随时间扩散涡强度。涡周围速度降为零的圆形区域称为涡核。涡段长度的增加会导致涡核半径减小，反之亦然。另一方面，扩散会不断地径向扩散涡。

由于上述类比，涡丝方法的实践者通常将正则化称为"粘性核心"模型，将正则化参数称为"核心半径"。此外，粘性扩散通常通过在空间和时间上修改正则化参数来引入，而不是求解涡量方程的扩散项。本文档在需要澄清时明确区分这两个概念，但在上下文明确时使用宽松的术语。

正则化参数的确定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

正则化参数既是所模拟物理（叶片边界层和尾迹）的函数，也是离散化选择的函数。影响因素包括弦长、边界层高度以及每个涡丝近似的体积。目前，这个选择留给用户（**RegDeterMethod=[0]**）。旋转叶片的经验结果可以在Gupta的工作中找到(:cite:`olaf-Gupta06_1`)。作为指导，正则化参数可以选择为叶片平均展向离散化的两倍。当用户选择**RegDeterMethod=[1]**时，将实现这个指导原则。未来将考虑对该选项的进一步完善。

.. _sec:RegularizationFunction:

已实现的正则化函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

已经开发了几种正则化函数(:cite:`olaf-Rankine58_1,olaf-Scully75_1,olaf-Vatistas91_1`)。目前有五个选项：1) 无修正，2) Rankine方法，3) Lamb-Oseen方法，4) Vatistas方法，或5) 分母偏移方法。如果不使用修正方法（**RegFunction=[0]**），则:math:`F_\nu=1`。其余方法在以下各节中详细介绍。这里，:math:`r_c`是正则化参数（**WakeRegParam**），:math:`\rho`是到涡丝的距离。两个变量都以米为单位。

Rankine
^^^^^^^

Rankine方法(:cite:`olaf-Rankine58_1`)是最简单的正则化模型。使用这种方法，Rankine涡具有有限核心，在涡中心附近是刚体旋转，在远离中心处是势涡。如果使用该方法（**RegFunction=[1]**），粘性核心修正由公式:eq:`rankine`给出。

.. math::
       F_\nu= \begin{cases} \rho^2/r_c^2 & 0 < \rho < 1 \\
       1 & \rho > 1 \end{cases}
   :label: rankine

这里，:math:`r_c`是涡丝的粘性核心半径，详见:numref:`sec:corerad`。

Lamb-Oseen
^^^^^^^^^^

如果使用Lamb-Oseen方法（**RegFunction=[2]**），粘性核心修正由公式:eq:`lamboseen`给出。

.. math::
   F_\nu= \bigg[1-\text{exp}(-\frac{\rho^2}{r_c^2})\bigg]
   :label: lamboseen

Vatistas
^^^^^^^^

如果使用Vatistas方法（**RegFunction=[3]**），粘性核心修正由公式:eq:`vatistas`给出。

.. math::
   F_\nu
   = \frac{\rho^2}{(\rho^{2n}+r_c^{2n})^{1/n}}
   = \frac{(\rho/r_c)^2}{(1 + (\rho/r_c)^{2n})^{1/n}}
   :label: vatistas

这里，:math:`\rho`是从涡段到任意点的距离(:cite:`olaf-Abedi16_1`)。旋翼飞行器应用的研究表明:math:`n=2`是合适的值，本工作中使用该值(:cite:`olaf-Bagai93_1`)。

分母偏移/截断
^^^^^^^^^^^^^^^^^^^^^^^^^^

如果使用分母偏移方法（**RegFunction=[4]**），粘性核心修正由公式:eq:`denom`给出：

.. math::
   \begin{aligned}
     \vec{v}(\vec{x})
     =   \frac{\Gamma}{4\pi} \frac{(r_1+r_2)}{r_1r_2(r_1r_2+\vec{r}_1\cdot\vec{r}_2) + r_c^2  r_0^2} \vec{r}_1\times\vec{r}_2
   \end{aligned}
   :label: denom

这里，通过在公式:eq:`eq:BiotSavartSegment`的分母中引入一个与涡丝长度:math:`r_0`成比例的加性因子来消除奇异性。在这种情况下，:math:`F_\nu=1`。该方法可以在van Garrel的工作中找到(:cite:`olaf-Garrel03_1`)。

.. _sec:corerad:

正则化参数的时间演化–核心扩展方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

正则化参数有四种可用的随时间演化的方法：1) 恒定值，2) 拉伸，3) 尾迹年龄，或4) 拉伸和尾迹年龄。后三种方法融合了粘性扩散和正则化的概念。本节中使用的符号:math:`r_{c0}`对应输入文件参数值**WakeRegParam**。

恒定
^^^^^^^

如果选择恒定值（**WakeRegMethod=[1]**），则在整个仿真过程中所有拉格朗日标记点的:math:`r_c`值保持不变，取参数**WakeRegParam**给出的值（以米为单位）。

.. math::
   r_c(\zeta) = r_{c0}
   :label: cst

这里，:math:`\zeta`是涡尾迹年龄，从其释放时间开始测量。

拉伸
^^^^^^^

如果选择拉伸方法（**WakeRegMethod=[2]**），粘性核心半径由公式:eq:`stretch`建模。

.. math::
   r_c(\zeta,\epsilon) = r_{c0} (1+\epsilon)^{-1}
   :label: stretch

.. math::
   \epsilon = \frac{\Delta l}{l}

这里，:math:`\epsilon`是涡丝应变，:math:`l`是涡丝长度，:math:`\Delta l`是两个时间步之间的长度变化。公式:eq:`stretch`中的积分表示应变效应。

该选项尚未实现。

尾迹年龄 / 核心扩展
^^^^^^^^^^^^^^^^^^^^^^^^^

如果选择尾迹年龄方法（**WakeRegMethod=[3]**），粘性核心半径由公式:eq:`age`建模。

.. math::
   r_c(\zeta) = \sqrt{r_{c0}^2+4\alpha\delta\nu \zeta}
   :label: age

其中:math:`\alpha=1.25643`，:math:`\nu`是运动粘度，:math:`\delta`是粘性扩散参数（通常介于:math:`1`和:math:`1,000`之间）。参数:math:`\delta`在输入文件中以**CoreSpreadEddyVisc**提供。这里，项:math:`4\alpha\delta\nu \zeta`考虑了尾迹向下游传播时的粘性效应。背景湍流越高，涡量随时间的扩散越多，:math:`\delta`的值应该越高。这种方法部分考虑了涡量的粘性扩散，而忽略了尾迹涡量自身之间或尾迹涡量与背景流之间的相互作用。它通常被称为核心扩展方法。设置**DiffusionMethod=[1]**与使用尾迹年龄方法（**WakeRegMethod=[3]**）相同。

核心半径的时间演化实现为：

.. math::

    \frac{d r_c}{dt} = \frac{2\alpha\delta\nu}{r_c(t)}

在叶片上:math:`\frac{d r_c}{dt}=0`。


拉伸和尾迹年龄
^^^^^^^^^^^^^^^^^^^^^^^

如果选择拉伸和尾迹年龄方法（**WakeRegMethod=[4]**），粘性核心半径由公式:eq:`stretchandage`建模。

.. math::
   r_c(\zeta,\epsilon) = \sqrt{r_{c0}^2 + 4\alpha\delta\nu \zeta \big(1+\epsilon\big)^{-1} }
   :label: stretchandage

该选项尚未实现。

.. _sec:diffusion:

扩散
---------

使用粘性分裂假设来分别求解涡量的对流和扩散。扩散项:math:`\nu \Delta \vec{\omega}`表示分子扩散。该项允许涡量线的粘性连接。此外，湍流会基于湍流涡粘度以类似方式扩散涡量。

参数**DiffusionMethod**用于在粘性扩散方法之间切换。目前，仅实现了核心扩展方法。该方法在:numref:`sec:corerad`中描述，因为它等价于正则化参数随尾迹年龄增加。
