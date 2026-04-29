
.. _ed_intro:

ElastoDyn 用户指南和理论手册
============================

本文档为 ElastoDyn 软件程序提供快速参考指南。它旨在供普通用户与其他 OpenFAST 手册结合使用。随着新版本的发布，本手册将根据需要更新，以提供有关软件改进或修改的更多信息。

有关此模块的更多文档可在 :ref:`general-reference-docs` 中列出的遗留文档中找到。具体来说，FAST v6 用户指南以及 FAST v8 README 中的小幅更新描述了此模块，并包含有关 OpenFAST 的一般信息。

.. note::

   我们正在将文档从 FAST 8 迁移到 OpenFAST。作为参考，这里提供了旧文档的各个部分。虽然大部分内容仍然直接适用于 OpenFAST，但部分内容可能已经过时。


以下文档是 ElastoDyn 运动方程的详细推导。这些文档尚未汇编成报告，因此它们主要包含方程，解释性文本很少。具有运动学和动力学背景的读者可能能够理解这些方程。按照以下顺序学习这些文档最容易理解：

1. :download:`FASTDOFs.xls <../../../OtherSupporting/ElastoDyn/FASTDOFs.xls>`：包含 FAST 运动方程中使用的自由度索引列表。
2. :download:`FASTCoordinateSystems.doc <../../../OtherSupporting/ElastoDyn/FASTCoordinateSystems.doc>`：记录了 FAST 中每个坐标系之间的变换矩阵。遗憾的是，本文档中没有绘制这些坐标系的图表。希望可以通过变换矩阵来可视化它们。
3. :download:`FASTKinematics.doc <../../../OtherSupporting/ElastoDyn/FASTKinematics.doc>`：记录了系统中每个"重要"点的线性位置、速度和加速度矢量，并记录了系统中每个"重要"参考系的角速度和角加速度矢量。还包括凯恩动力学所需的偏速度矢量的文档。
4. :download:`FASTKinetics.doc <../../../OtherSupporting/ElastoDyn/FASTKinetics.doc>`：记录了使用凯恩动力学推导运动方程的过程。
5. :download:`FASTLoads.doc <../../../OtherSupporting/ElastoDyn/FASTLoads.doc>`：记录了如何使用运动方程中的项计算输出载荷。
6. :download:`FASTMotions.doc <../../../OtherSupporting/ElastoDyn/FASTMotions.doc>`：记录了如何使用运动方程中的变量计算输出运动。
7. :download:`FASTLogicFlow.doc <../../../OtherSupporting/ElastoDyn/FASTLogicFlow.doc>`：包含 FAST 使用的子程序名称列表。这些名称按照它们在程序中被调用的顺序列出。

这些论文中记录的方程存在一些小错误，理解这些方程后可能会发现这些错误。已实现的代码没有这些错误。这些论文没有描述 Fortran 源代码和变量命名约定，但通过仔细研究可以进行源代码比较。

请注意，"非官方 FAST 理论手册"适用于 FAST v7 的结构方程以及 FAST v8 和 OpenFAST 的 ElastoDyn 模块。

.. toctree::

   coordsys.rst
   input.rst
   theory.rst
   zrefs.rst
