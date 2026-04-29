.. _ad_appendix:

附录
========

.. _ad_input_files:

AeroDyn输入文件
-------------------

在本附录中，我们描述AeroDyn输入文件的结构并提供示例。

1) 基准AeroDyn驱动程序输入文件：
:download:`(驱动程序输入文件示例) <examples/ad_driver_example.dvr>`：
驱动程序输入文件仅适用于独立版本的AeroDyn，包含通常由OpenFAST生成的输入，以及控制非耦合模型气动仿真所需的参数。

AeroDyn驱动程序时间序列输入文件
:download:`(驱动程序时间序列输入文件示例) <examples/ad_driver_timeseries_example.csv>`：
AeroDyn驱动程序中案例的时间序列输入文件允许参数随时间变化。此功能对于在OpenFAST外调试气动响应非常有用。

2) 多转子AeroDyn驱动程序输入文件
:download:`(驱动程序输入文件示例) <examples/ad_driver_multiple.dvr>`


3) AeroDyn主输入文件
:download:`(主输入文件示例) <examples/ad_primary_example.dat>`

AeroDyn主输入文件定义建模选项、环境条件（除自由流外）、翼型、塔筒节点离散化和属性、塔筒、轮毂和机舱属性，以及输出文件规范。

该文件分为几个功能部分。每个部分对应气动模型的一个方面。

输入文件以两行标题信息开头，这些信息供用户使用，软件不会使用。

4) 翼型数据输入文件

:download:`(极曲线数据) <examples/ad_polar_example.dat>`

:download:`(翼型坐标) <examples/ad_airfoil_example.dat>`

翼型数据输入文件本身（每个翼型一个）包含升力系数、阻力系数和俯仰力矩系数随攻角变化的表格，以及UA模型参数。在这些文件中，任何第一个非空白字符为感叹号（!）的行都会被忽略（用于插入注释行）。非注释行应按顺序出现在文件中，但为了阅读清晰，可以根据需要插入注释行。

5) 叶片数据输入文件
:download:`(叶片数据输入文件示例) <examples/ad_blade_example.dat>`

叶片数据输入文件包含叶片的节点离散化、几何、扭转、弦长、翼型标识符和浮力属性。每个叶片使用单独的文件，这允许对气动不平衡进行建模。

.. _ad_output_channels:

AeroDyn输出通道列表
-------------------------------


AeroDyn有常规输出（参见 :numref:`AD-Outputs`）和节点输出（参见 :numref:`AD-Nodal-Outputs`）。

输出使用的坐标系（标记为i、h、p、l、a）在 :numref:`ad_coordsys` 中描述。


所有可能输出参数的完整、最新列表在Excel文件 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>` 中给出，常规输出和节点输出分别在`AeroDyn`和`AeroDyn_Nodes`标签中。Excel文件中的名称按含义分组，但可以根据需要在AeroDyn输入文件的OUTPUTS部分中任意排序。



**常规输出**
下面给出一些常规输出的示例（完整列表请参见 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>`）：


   - `RtAeroCp`：气动功率系数。


   - :math:`B \alpha N \beta` 指叶片 :math:`\alpha` 的输出节点 :math:`\beta`，其中 :math:`\alpha` 是[1,3]范围内的数字，:math:`\beta` 是[1,9]范围内的数字，对应 :math:`\textit{BlOutNd}` 列表中的第 :math:`\beta` 项。

   - :math:`\textit{TwN}\beta` 指塔筒的输出节点 :math:`\beta`，范围为[1,9]，对应 :math:`\textit{TwOutNd}` 列表中的第 :math:`\beta` 项。


**节点输出**

下面描述节点输出的一个示例（完整列表请参见 :download:`OutListParameters.xlsx <../../../OtherSupporting/OutListParameters.xlsx>`）。

要请求惯性坐标系(:math:`i`)中所有叶片节点处的未受扰流动速度的x分量(`VUnd`)，只需将 :math:`VUndxi` 放入AeroDyn节点输出列表中。这将生成形式为`AB`:math:`\alpha N\beta` `Vundxi`的输出通道，对应叶片 :math:`\alpha` 的节点 :math:`\beta`，其中 :math:`\alpha` 是[1,3]范围内的数字，:math:`\beta` 是[1,999]范围内的数字，对应AeroDyn叶片节点的索引。
