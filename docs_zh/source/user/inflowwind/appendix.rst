.. _ifw_appendix:

附录
========

.. _ifw_input_files:

InflowWind 输入文件
----------------------

本附录介绍 InflowWind 输入文件的结构并提供示例。

1) InflowWind 驱动程序输入文件
:download:`（驱动程序输入文件示例）<examples/inflowwind_driver_example.inp>`：

驱动程序输入文件仅在独立运行 InflowWind 时需要。它包含 InflowWind 文件、插值参数和所需输出文件的相关输入。
InflowWind 驱动程序也可以不使用此文件，改用命令行参数运行。

2) InflowWind 主输入文件
:download:`（主输入文件示例）<examples/inflowwind_example.dat>`：

InflowWind 主输入文件定义生成或从其他文件读取的来流。InflowWind 文件包含每种风场文件格式对应的节。

3) 原生 Bladed 缩放文件
:download:`（主输入文件示例）<examples/inflowwind_bladedscaling_example.dat>`：

此文件包含确定如何缩放 Bladed 无量纲全场湍流文件的行。

4) 均匀风数据文件
:download:`（均匀风输入文件示例）<examples/inflowwind_uniform_example.dat>`：

此文件包含定义均匀（确定性）风数据文件的行。


.. _ifw_output_channels:

InflowWind 输出通道列表
----------------------------------

以下是 InflowWind 模块所有可能的输出参数列表。
请参阅 :download:`（OutListParameters.xlsx 文件）<../../../OtherSupporting/OutListParameters.xlsx>` 的 InflowWind 选项卡：

