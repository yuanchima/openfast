
HydroDyn 用户指南与理论手册
=====================================

.. toctree::
   :maxdepth: 1

   getting_started.rst
   input_files.rst
   output_files.rst
   modeling_considerations.rst
   future_work.rst
   zrefs.rst
   appendix.rst

HydroDyn 是一款时域水动力学模块，已耦合到 OpenFAST 风电机组多物理场工程工具中，可实现海上风电机组的气动-水动-伺服-弹性耦合仿真。HydroDyn 适用于固定式基础和浮式海上子结构。当前版本的 HydroDyn 通过 FAST 模块化框架与 OpenFAST 集成。HydroDyn 也可以作为独立代码运行，在不与 OpenFAST 耦合的情况下计算水动力载荷。

除本文档外，还提供以下材料供参考，包括演示幻灯片、开发计划和出版物。请注意，其中部分内容可能已过时，仅适用于旧版本的 HydroDyn。

- :download:`浮式海上风电机组多向海况下的波浪载荷计算 <https://www.nlr.gov/docs/fy14osti/61161.pdf>`
- :download:`二阶水动力对浮式海上风电机组的影响 <https://www.nlr.gov/docs/fy14osti/60966.pdf>`
- :download:`FAST 中波浪辐射力的状态空间实现 <https://www.nlr.gov/docs/fy13osti/58099.pdf>`
- :download:`海上浮式风电机组动力学——模型开发与验证 <https://dx.doi.org/10.1002/we.347>`
- :download:`海上浮式风电机组动力学建模与载荷分析 <https://www.nlr.gov/docs/fy08osti/41958.pdf>`
- :download:`实施计划草案——HydroDyn 中支持 Morison 构件时变浮力载荷的修改 <../../../OtherSupporting/HydroDyn/HydroDyn_Plan_TCF_Morison.docx>`
- :download:`实施计划——HydroDyn 中支持多 WAMIT 物体的状态空间模块修改 <../../../OtherSupporting/HydroDyn/HydroDyn_Plan_TCF_NBodyStateSpace.docx>`
- :download:`实施计划（修订版）——HydroDyn 中支持多 WAMIT 物体的修改 <../../../OtherSupporting/HydroDyn/HydroDyn_Plan_TCF_NBody.docx>`
- :download:`实施计划——HydroDyn 中的二阶力 <../../../OtherSupporting/HydroDyn/HydroDyn_2ndOrderForces_Plan.pdf>`
- :download:`实施计划——HydroDyn 中的二阶波浪运动学 <../../../OtherSupporting/HydroDyn/WAVE2_document.pdf>`
- :download:`HydroDyn 添加波浪拉伸功能计划 <../../../OtherSupporting/HydroDyn/HydroDyn_WaveStretching_Plan.docx>`
- :download:`FAST 的破碎波建模方法 <../../../OtherSupporting/HydroDyn/Breaking_Wave_Modeling_Approach_for_FAST.docx>`


HydroDyn 提供多种计算结构水动力载荷的方法：

* 势流理论求解
* 切片理论求解
* 塔筒混合组合方法

HydroDyn 内部生成的波浪可以是规则（周期）波或不规则（随机）波，也可以是长峰波（单向）或短峰波（波浪能量分布在多个方向）。波面高程或完整的波浪运动学也可以在外部生成后导入 HydroDyn 使用。HydroDyn 内部采用一阶（线性 Airy）或一阶加二阶波浪理论 :cite:`SharmaDean:1981` 对有限水深进行波浪解析生成，可选择包含方向扩散，但波浪运动学仅在平坦海床与静水位（SWL）之间的区域计算，不包含波浪拉伸或更高阶波浪理论。二阶水动力实现包含差频（平均和慢漂）和和频项的时域计算。为了最小化计算成本，所有波浪频率分量的求和采用快速傅里叶变换（FFT）。

势流求解适用于尺寸相对于典型波长较大的子结构或子结构构件。势流求解可采用频域到时域变换或流体脉冲理论（FIT）。对于前者，势流水动力载荷包括线性静水恢复力、线性波浪辐射带来的附加质量和阻尼贡献（包含自由表面记忆效应），以及一阶和二阶衍射（Froude-Kriloff 和散射）带来的入射波激励。势流求解所需的水动力系数（一阶和二阶）与频率相关，必须通过独立的频域面元法程序（如 WAMIT）预先计算提供。辐射记忆效应可以通过直接时域卷积或线性状态空间方法计算，状态空间模型通过 `SS_Fitting <https://www.nlr.gov/wind/nwtc/ss-fitting.html>`_ 预处理程序导出。二阶项可以从完整的差频和和频二次传递函数（QTF）导出，或者差频项可以通过 Standing 等人 :cite:`Standing:1987` 对 Newman 近似的扩展来估计，仅需基于一阶系数。FIT 的使用尚未在本手册中说明。

切片理论求解更适用于直径相对于典型波长较小的子结构或子结构构件。切片理论水动力载荷可以应用于多个相互连接的构件（每个构件可能有倾斜和锥度），并直接从子结构未位移位置处的未扰动波浪和海流运动学导出。切片理论载荷包含 Morison 方程的相对形式，用于计算分布流体惯性、附加质量和粘性阻力分量。额外的分布载荷分量包括锥形构件的轴向载荷和静水浮力载荷。水动力载荷也可以作为集中载荷施加在构件端点（称为节点）。还可以考虑构件的进水或压载，以及海洋生物附着的影响。该求解所需的水动力系数由用户指定的动压力、附加质量和粘性阻力系数提供。

对于某些子结构和海况，势流理论的水动力载荷需要补充流动分离带来的载荷。为此，可以在势流理论求解中加入切片理论求解的粘性阻力分量。另一个可选方案是向系统提供全局阻尼矩阵（线性或二次）来表示这种效应。

当 HydroDyn 与 OpenFAST 耦合时，HydroDyn 在每个耦合时间步接收（刚性或柔性）子结构的位置、方向、速度和加速度，然后计算水动力载荷并返回给 OpenFAST。目前，OpenFAST 的 ElastoDyn 结构动力学模块假设浮式平台的子结构（浮式平台）是一个六自由度（DOF）刚体。对于固定式基础海上风电机组，OpenFAST 的 SubDyn 模块支持多构件子结构的结构柔性，与 HydroDyn 的耦合包含水弹性效应。

HydroDyn 主输入文件定义了子结构几何、水动力系数、入射波浪运动学和海流、势流求解选项、进水/压载和海洋生物附着，以及辅助参数。切片理论构件的几何由未位移子结构在全局参考系中的节点坐标定义，原点位于未变形塔筒中心线与平均海平面（MSL）的交点。一个构件连接两个节点；多个构件可以共用同一个节点。水动力载荷在节点处计算，节点是将构件细分为多个（**MDivSize** 输入参数）单元后的结果（节点位于每个单元的端点），由模块自动计算。构件属性包括外径、厚度，以及动压力、附加质量和粘性阻力系数。构件属性在节点处指定；如果两个节点之间的属性发生变化，将对内部节点进行线性插值。

有关如何下载或编译 HydroDyn 和 OpenFAST 软件可执行文件的详细信息，以及运行独立版 HydroDyn 和与 OpenFAST 耦合版本的说明，请参见 :ref:`hd-getting-started`。
