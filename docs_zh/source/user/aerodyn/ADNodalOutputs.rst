节点输出
~~~~~~~~~~~~~

除了上面 :numref:`AD-Outputs` 中列出的命名输出外，AeroDyn还允许输出完整的叶片节点运动和载荷（目前塔筒节点不可用）。请参考Excel文件 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>` 中的AeroDyn_Nodes标签，获取所有可能输出参数的完整列表。

本节位于上述正常输出部分的`END`语句之后，包含一个分隔描述行，后跟以下选项。

**BldNd_BladesOut** 指定要输出的叶片数量。可能的值为0到AeroDyn建模的叶片数量。如果该值设置为1，则仅输出叶片1；如果值为2，则输出叶片1和2。

**BldNd_BlOutNd** 指定要输出的节点（在所有选择输出的叶片上）。有效的输入包括"ALL"（所有叶片节点）、"TIP"（仅最后一个叶片节点）、"ROOT"（仅第一个叶片节点），或者对应要输出节点的数字列表；有效数字为1到AeroDyn为每个叶片建模的叶片节点数量。

**OutList** 部分控制AeroDyn生成的节点输出量。在本节中，用户指定要输出的通道族名称。然后，AeroDyn在内部通过组合叶片编号、节点编号和通道族名称来创建每个通道的输出名称。例如，如果用户指定**AxInd**作为通道族名称，则输出通道将按照**B**\ :math:`\mathbf{\beta}`\ **N###AxInd** 的约定命名，其中 :math:`\mathbf{\beta}` 是叶片编号，**###** 是三位节点编号。


**节点输出部分示例**

本示例包含常规输出部分的``END``语句。

.. container::
   :name: File:ADNodalOutputs

   .. literalinclude:: examples/NodalOutputs.txt
      :linenos:
