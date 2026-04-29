
.. _OLAF-List-of-Output-Channels:

附录 C：OLAF 输出通道列表
==========================

这是 OLAF 模块所有可能输出参数的列表。名称按含义分组，但用户可以根据需要在 *AeroDyn* 主输入文件的 OUTPUTS 部分中任意排序。:math:`N\beta` 指输出节点 :math:`\beta`，其中 :math:`\beta` 是 [1,9] 范围内的数字，对应于 **OutNd** 列表中的第 :math:`\beta` 项。每个输出名称前缀为 :math:`B\alpha`，其中 :math:`\alpha` 是 [1,3] 范围内的数字，对应于叶片编号。


.. list-table:: 可用的 OLAF 输出通道
   :widths: 25 15 50
   :header-rows: 1
   :align: center
   :name: Tab:OLAFoutputs

   *  - 通道名称
      - 单位
      - 描述
   *  - :math:`B \alpha N \beta Gam`
      - :math:`m^2/s`
      - 沿叶片的环量


..
      ============================ ============= ===========================
      通道名称                      单位           描述
      ============================ ============= ===========================
      :math:`B \alpha N \beta Gam` :math:`m^2/s` 沿叶片的环量
      ============================ ============= ===========================
