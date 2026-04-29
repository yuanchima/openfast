.. _StC:

结构控制（ServoDyn）
=========================

.. only:: html

   ServoDyn 中的结构控制（StC）模块用于仿真调谐质量阻尼器（TMD）
   和调谐液柱阻尼器（TLCD）。该模块还提供了一个选项，
   可在结构控制节点位置施加力时间序列载荷。

   StC 节点的位置在 ServoDyn 主输入文件中指定（参见
   :numref:`SrvD-Stc-inputs` 和 :numref:`StC-Locations`）。这些装置可以安装在
   机舱、塔筒、叶片或浮式平台上。每个位置可以放置多个 StC，
   每个 StC 都有自己的输入文件。StC 模块的输出通道
   由 ServoDyn 处理（详见 :numref:`SrvD-Outputs`）。



.. toctree::
   :maxdepth: 2

   StC_input.rst
   StC_Theory.rst
   StC_TLCD_Theory.rst
   zrefs.rst
