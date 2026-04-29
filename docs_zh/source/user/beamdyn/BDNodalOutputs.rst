
节点输出
~~~~~~~~

除了上述 :numref:`BD-Outputs` 中的命名输出外，BeamDyn 还允许输出全套叶片节点运动和载荷（目前塔筒节点不可用）。请参考 Excel 文件 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>` 中的 BeamDyn_Nodes 标签页，获取可能输出参数的完整列表。

本节位于上述常规输出部分的 ``END`` 语句之后，包括一个分隔符描述行，后跟以下选项。

**BldNd_BlOutNd** 指定要输出的节点。目前该参数未使用。

**OutList** 部分控制 BeamDyn 生成的节点输出量。在本节中，用户指定要输出的通道族名称。每个通道的输出名称由 BeamDyn 在内部通过组合叶片编号、节点编号和通道族名称生成。例如，如果用户指定 **TDxr** 作为通道族名称，输出通道将按照 **B**\ :math:`\mathbf{\beta}`\ **N###TDxr** 的约定命名，其中 :math:`\mathbf{\beta}` 是叶片编号，**###** 是三位节点编号。


示例节点输出部分
^^^^^^^^^^^^^^^^

此示例包含常规输出部分的 ``END`` 语句。

.. container::
   :name: File:BDNodalOutputs

   .. literalinclude:: examples/NodalOutputs.txt
      :linenos:
