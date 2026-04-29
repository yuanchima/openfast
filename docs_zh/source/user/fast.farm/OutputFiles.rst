
.. _FF:Output:

输出文件
========

FAST.Farm 生成五种类型的输出文件：回显文件、汇总文件、可视化输出文件、时间序列结果文件以及与 OpenFAST 相关的文件。以下章节详细介绍这些文件的用途和内容。

回显文件
---------

如果 FAST.Farm 主输入文件中的 **Echo** = TRUE，文件内容将被回显到命名规则为 <*RootName.ech*> 的文件中，其中 <*RootName*> 的定义见 :numref:`FF:AmbWindVTK`。回显文件有助于调试主输入文件。如果 FAST.Farm 在解析主输入文件时遇到错误，回显文件的内容将被截断。错误通常对应于最后成功回显行的下一行。

.. _FF:Output:Sum:

汇总文件
------------

如果 FAST.Farm 主输入文件中的 **SumPrint** = TRUE，FAST.Farm 将生成命名规则为 <*RootName.sum*> 的汇总文件。该文件汇总了风电场模型的关键信息，包括风力机位置和 OpenFAST 主输入文件；尾流动力学有限差分网格和参数；各模型组件的时间步长；以及所选输出的名称、单位和顺序。

.. _FF:Output:Vis:

可视化输出文件
--------------------------

如果 FAST.Farm 主输入文件中的 **WrDisWind** = TRUE，FAST.Farm 将生成完整的 3D 低分辨率和高分辨率扰动风数据输出文件，即整个风电场的环境风和尾流相互作用数据，用于可视化。这些输出文件的 VTK 数据格式和空间分辨率（网格点数、原点和间距）与 FAST.Farm 仿真使用的对应低分辨率和高分辨率环境风数据相匹配。VTK 文件将写入到 FAST.Farm 主文件存储目录下名为 *vtk_ff* 的目录中。这些输出文件的命名规则分别为：
低分辨率扰动风数据文件：*<RootName>.Low.Dis.<n*\ :sub:`low`\ *>.vtk*
高分辨率扰动风数据文件：*<RootName>.HighT<n*\ :sub:`t`\ *>.Dis.<n*\ :sub:`high`\ *>.vtk*
其中 *<n*\ :sub:`t`\ *>*、*<n*\ :sub:`low`\ *>* 和 *<n*\ :sub:`high`* 的定义见 :numref:`FF:AmbWindVTK`，但包含前导零。

- 同样地，如果 FAST.Farm 主输入文件中的 **NOutDisWindXY**、**NOutDisWindYZ** 或 **NOutDisWindXZ** 设置为大于零，FAST.Farm 将生成低分辨率扰动风数据（包括尾流）输出文件，这些文件是完整低分辨率域的 2D 切片。2D 切片分别平行于全局惯性坐标系的 *X-Y*、*Y-Z* 和/或 *X-Z* 平面。VTK 文件将写入到 FAST.Farm 主文件存储目录下名为 *vtk_ff* 的目录中。这些输出文件的命名规则分别为：
  *X-Y* 切片：*<RootName>.Low.DisXY<n*\ :sub:`Out`\ *>.<n*\ :sub:`low`\ *>.vtk*，
  *Y-Z* 切片：*<RootName>.Low.DisYZ<n*\ :sub:`Out`\ *>.<n*\ :sub:`low`\ *>.vtk*，
  *X-Z* 切片：*<RootName>.Low.DisXZ<n*\ :sub:`Out`\ *>.<n*\ :sub:`low`\ *>.vtk*，
  其中 *<n*\ :sub:`Out`\ *>* 的定义见 :numref:`FF:AmbWindVTK`，但包含前导零。

所有扰动风数据文件的时间步长（帧率的倒数）由 FAST.Farm 主输入文件中的输入参数 **WrDisDT** 设置。请注意，完整的高分辨率扰动风数据输出文件不会以 :math:`1/`\ **DT_High** 的帧率输出，而是每隔 **WrDisDT** 秒输出一次。

每个可视化输出文件采用与高保真实前仿真环境风数据文件相同的 VTK 格式。有关文件格式的详细信息，请参见 :numref:`FF:AmbWindIfW`。

可视化环境风和尾流相互作用有助于解释结果和调试问题。但是，当 **WrDisWind** = TRUE 和/或 **NOutDisWindXY**、**NOutDisWindYZ** 和/或 **NOutDisWindXZ** 设置为大于零时，FAST.Farm 每个输出选项都会生成大量文件。这种文件生成会减慢 FAST.Farm 的运行速度并占用大量磁盘空间，尤其是在生成完整的低分辨率和高分辨率扰动风数据文件时。因此，在运行大量 FAST.Farm 仿真时，建议禁用可视化功能。


.. _FF:Output:Planes:

尾流动力学平面文件
-------------------------

在 FAST.Farm 主输入文件中将 **OutAllPlanes** 选项设置为 true 将导致输出尾流动力学模块的尾流平面。该选项需要大量磁盘写入操作，会显著减慢仿真速度。尾流平面以 VTK 格式写入，存储在仿真目录根目录下的 `vtk_ff_planes` 文件夹中。
每个平面、每个时间步和每台风力机都会生成一个 VTK 文件。写入的平面数量会随着每个 **DT_Low** 时间步增加，直到达到 **Num_Planes** 的总数。
平面的坐标采用蜿蜒参考系（而非全局坐标）并使用笛卡尔坐标系。
始终写入以下字段：
尾流亏损速度的 x、y、z 分量（在蜿蜒参考系中）、
环境涡黏性、剪切涡黏性和总涡黏性。
当 **Mod_Wake=1**（极坐标尾流）时，字段会从极坐标转换为笛卡尔坐标。
当 **Mod_Wake≠1** 时，VTK 文件中会提供额外的速度梯度。


.. _FF:Output:Time:

时间序列结果文件
------------------------

FAST.Farm 时间序列结果写入到命名规则为 <*RootName.out*> 的 ASCII 文本文件中。结果采用表格格式，每列是一个数据通道，第一列始终是仿真时间；每行对应一个仿真输出时间步。数据通道在 FAST.Farm 主输入文件的 *OUTPUT* 部分的 **OutList** 节中指定。FAST.Farm 生成文件的列格式由 FAST.Farm 主输入文件的 **OutFmt** 参数指定。

OpenFAST 输出文件
---------------------

除了 FAST.Farm 生成的输出文件外，每台风力机的 OpenFAST 模型也可能生成输出文件。OpenFAST 可能生成的各种输出文件（包括驱动程序/耦合代码层面和模块层面，在 OpenFAST 输入文件中指定）在 OpenFAST 文档中有描述，包括汇总（*.sum*）文件、时间序列结果（ASCII *.out* 或二进制 *.outb*）文件、可视化（*.vtk*）文件等。FAST.Farm 仿真会生成这些相同的文件，但路径/根名称会更改为 *<RootName of WT_FASTInFile>.T<n*\ :sub:`t`\ *>*。
