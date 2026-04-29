.. _working_with_OF:

使用 OpenFAST
=====================

本节为 OpenFAST 的一些典型使用场景提供支持。
假设用户已拥有 OpenFAST 可执行文件（安装方法参见 :ref:`installation`）。





快速入门 - 运行 OpenFAST
------------------------------

本快速入门将说明如何运行 OpenFAST。OpenFAST 通常从终端
（也称为命令提示符或命令行）运行。
运行 OpenFAST 的最简单方法是将 OpenFAST 可执行文件复制到工作目录中，
然后在该目录中打开终端。步骤如下：

 - 将 OpenFAST 可执行文件复制到将运行仿真的目录中
 - 打开终端
 - 导航到包含 OpenFAST 可执行文件的文件夹
 - 运行 OpenFAST 查看其版本
 - 使用给定的输入文件运行 OpenFAST

详细步骤如下。


打开终端
~~~~~~~~~~~~~~~

要了解如何在您的操作系统上打开终端，可以尝试以下搜索：

 - `在 Windows 上 <https://www.google.com/search?q=how+to+open+a+command+prompt+on+windows>`__
 - `在 Linux 上 <https://www.google.com/search?q=how+to+open+a+command+prompt+on+linux>`__
 - `在 Mac 上 <https://www.google.com/search?q=how+to+open+a+command+prompt+on+mac>`__



导航到仿真目录
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在终端中，可以使用命令 `cd` 在文件夹之间导航。
本例假设仿真目录为 `simulations/test`，因此要导航到此目录，需要输入：

.. code-block:: bash

    cd simulations
    cd test

或：

.. code-block:: bash

    cd simulations/test


要进入父目录，可以使用 `cd ..`。
如果目录路径包含空格，请用引号将路径括起来。
通常好的做法是避免在目录和文件名中使用空格。
路径也可以是绝对路径，例如 `cd C:/simulations/test`。



运行 OpenFAST 查看版本号
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当终端位于 OpenFAST 所在目录后，可以尝试运行 OpenFAST 并查看版本，如下所示：

.. code-block:: bash

    ./openfast /h

命令开头的 `./` 字符表示可执行文件位于当前目录中。
以上命令将显示 OpenFAST 的版本、编译选项，并显示关于调用 OpenFAST 可执行文件语法的帮助信息。


.. note::

   请始终阅读终端窗口中显示的 OpenFAST 输出，因为错误和警告将显示在其中。通常，如果终端中显示错误，可以参考 :ref:`troubleshooting` 中的指南进行故障排除。


.. warning::

    跟踪您使用的 OpenFAST 版本非常重要，因为输入文件格式可能因版本而异（参见 :ref:`api_change`）。


.. tip::

    为避免将可执行文件复制到工作目录中，可以将可执行文件放入某个文件夹并将该文件夹添加到系统路径中。如果选择此方法并重启终端，应该能够从任何文件夹运行 `openfast /h`，此时 `./` 不再需要。


运行您的第一个 OpenFAST 仿真
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


运行仿真的典型语法是：

.. code-block:: bash

    ./openfast InputFile.fst

其中 `InputFile.fst` 是 OpenFAST 的主输入文件。建议 OpenFAST 主输入文件使用 `.fst` 扩展名，其他输入文件使用 `.dat` 扩展名。
OpenFAST 输入文件的格式规范见
:ref:`input_file_overview`。

我们提供一个最小工作示例，帮助您开始第一次 OpenFAST 运行。
此示例使用 `历史上著名的 NREL 5-MW <https://www.nrel.gov/docs/fy09osti/38060.pdf>`__ 风力机，
这是一个虚构但具有代表性的多兆瓦风力机，额定功率 5 MW，额定转子转速
12.1 rpm，轮毂高度 90 m，转子直径 126 m。
此示例为"陆上"版本的风力机，仅包含结构（无气动力），塔筒顶部初始位移为 3m。文件位于以下
`github 目录 <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/>`__。
您需要下载以下文件并将其放在工作目录中：

- `Main.fst <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/Main.fst>`__  ：OpenFAST 主输入文件
- `ElastoDyn.dat <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/ElastoDyn.dat>`__  ：ElastoDyn 模块的输入文件
- `ElastoDyn_Blade.dat <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/ElastoDyn_Blade.dat>`__  ：定义叶片结构属性的输入文件，供 ElastoDyn 模块使用
- `ElastoDyn_Tower.dat <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/ElastoDyn_Tower.dat>`__  ：定义塔筒结构属性的输入文件，供 ElastoDyn 模块使用

将这 4 个文件放入工作目录（OpenFAST 可执行文件所在且终端所在的目录）后，即可按如下方式运行仿真：

.. code-block:: bash

    ./openfast Main.fst


仿真应成功运行，OpenFAST 将生成一个扩展名为 `.out` 或 `.outb` 的输出文件。
我们在 `github 目录 <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/MinimalExample/>`__ 中提供了简单的 Python 和 Matlab 脚本，用于显示此仿真的部分输出通道。有关如何可视化输出的更多信息，请参见 :ref:`visualizing_input_output_OF`。
通常，如果终端中显示错误，可以参考 :ref:`troubleshooting` 中的指南进行故障排除。




.. tip::
    在某些平台上（如 Windows），可以在文件资源管理器中将输入文件拖放到 OpenFAST 可执行文件上，即可运行仿真。但如果使用此方法发生错误，您将无法看到错误消息。


.. tip::
   您可以使用指向 OpenFAST 可执行文件和 OpenFAST 主输入文件的相对路径和绝对路径。OpenFAST 的输入文件中也包含引用其他输入文件的文件路径。这些文件路径要么相对于当前文件，要么可以为绝对路径。








仿真故障排除
----------------------------


.. _troubleshooting:

简单故障排除
~~~~~~~~~~~~~~~~~~~~~~

当仿真过程中捕获到错误时，OpenFAST 将中止并显示类似以下的信息：

.. code-block:: bash

    FAST encountered an error during module initialization.
    Simulation error level: FATAL ERROR

    Aborting OpenFAST.

此消息之前的行将揭示错误的性质，这些信息可用于排除仿真故障。



典型错误
**************

以下列出一些典型错误及其解决方案：

- *未找到输入文件 "FILE"*：如信息所示，输入文件未找到。Linux 和 Mac 平台区分大小写，并要求使用正斜杠。OpenFAST 接受相对或绝对路径。相对路径是相对于引用它们的文件所在的路径。

- *读取 VAR 时文件 "FILE" 中的输入无效*：此类错误通常发生在初始化期间，当读取 OpenFAST 的某个输入文件时。可能是输入文件中的变量类型错误（整数应为逻辑型、浮点数应为字符串等）。不过，此类错误通常表示输入文件格式与正在执行的 OpenFAST 版本不匹配。最可能的原因是文件已过时。OpenFAST 输入文件中经常添加新行。可以查看 :ref:`api_change` 了解 OpenFAST 各版本之间的行变化，或查看 `r-test <https://github.com/openfast/r-test>`__ 寻找最新发布版和开发版 OpenFAST 输入文件的工作示例。

- *解析来自 "FILE" 的数据时发生致命错误。在第 #II 行未找到变量 "VAR"*。此类错误与上述类似。请检查文件格式是否与使用的 OpenFAST 版本匹配。

类似的消息表示用户输入错误（所选选项不可用或不兼容）。
此类错误消息通常足够明确。可以查看输入文件中的注释获取指导，并参考用户指南了解各模块单独输入的更多详细信息：:ref:`user_guide`。

.. tip::
   90% 的情况下，错误是由于 OpenFAST 版本与输入文件不匹配造成的（见上面第二点）。


典型警告
****************

不同模块（通常是气动模块）偶尔会发出一些警告并报告到命令窗口中。

 - *SkewedWakeCorrection encountered a large value of chi*：表明风力机处于大偏航/倾斜状态。当风力机经历显著运动时可能发生。
 - *The BEM solution is being turned off due to low TSR.*：表明瞬时转子转速接近零，或相对风速很大（检查输出通道 `RtSpeed` 和 `RtVavgx`）。

警告有时可以忽略，但通常表示模型中存在问题。请参见下一节的高级故障排除。




高级故障排除
~~~~~~~~~~~~~~~~~~~~~~~~

在某些情况下，仿真可能在运行过程中中止（*FAST encountered an error at simulation time T*），或者可能运行完成但在几个时间步后（甚至一个时间步后）输出为空或 "NaN"。此类错误通常是由于模型不物理造成的。
此时，您可能会在命令窗口中看到以下类型的错误消息：

- *Small angle assumption violated* 或 *Angles in GetSmllRotAngs() are larger than 0.4 radians*：此类警告表明结构的某些部分正在经历大转动，而 OpenFAST 的某些模块仅在小角度假设下有效。
- *Denominator is zero in GetSmllRotAngs()*

通常，当仿真中止或具有不现实或 NaN 值时，很可能是模型中存在错误（结构过刚、过软、来流不正确、初始条件不正确、控制器行为异常、OLAF 正则化参数设置错误等）。

.. tip::
   故障排除的关键是简化模型。可以选择逐步简化模型，直到其运行并产生物理结果。或者反过来，将模型简化到极致，然后逐步重新引入复杂性。典型的简化包括：无气动力、刚性结构、稳恒来流、无控制器。



以下是排除模型故障可以采取的一些步骤，特别是尝试将问题隔离到特定模块和输入：


- 使用简单的环境条件简化模型：稳恒均匀来流、静水。

- 移除控制器：在 ElastoDyn 中将 `GenDOF` 设为 False，在主输入文件中将 `CompServo` 设为 0。转子将以恒定 RPM 旋转。

- 通过在 ElastoDyn 输入文件中关闭大多数自由度来简化模型。可以从关闭所有自由度开始，然后逐步添加更多自由度。这可能指示问题来自叶片、机舱、塔筒还是子结构。一些经常出现问题的自由度包括传动链扭转（`DrTrDOF`）和偏航自由度（`YawDOF`）。ElastoDyn 中的传动链刚度和阻尼值经常设置错误。偏航的常见问题是 `NacYaw`（在 ElastoDyn 中）和 `YawNeut`（在 ServoDyn 中）不一致，或者偏航弹簧和阻尼 `YawSpr` 和 `YawDamp` 不物理。对于海上仿真，如果 `YawDOF` 和 `PtfmYDOF` 开启，模型需要有真实的 `PtfmYIner`，否则这些自由度在 ElastoDyn 中将是不适定的。PtfmYiner 应包含未变形塔筒的转动惯量，如果未使用 SubDyn，还应包含平台/过渡段的扭转惯量（如有）。

- 简化物理模型：使用 ElastoDyn（`CompElast=1`）而非 BeamDyn，使用 BEM（`WakeMod=1`）而非 OLAF，在 SubDyn 中使用 0 个 Craig-Bampton 模态。

- 可视化时间序列输出（参见 :ref:`visualizing_input_output_OF`）。向模型添加相关的位移输出，例如：PtfmSurge、PtfmSway、PtfmHeave、PtfmRoll、PtfmPitch、PtfmYaw、NacYaw、TTDspFA、TTDspSS、RotSpeed、OoPDefl1、IPDefl1 和 RtSkew。风力机很可能由于模型中的某些错误而出现大位移。

- 调整初始条件。如上所述，当偏航自由度开启时，`NacYaw`（ElastoDyn）和 `YawNeut`（ServoDyn）需要匹配。如果结构处于在给定环境条件下不现实的初始位置，则可能会超调（例如高风速但变桨角过低）。一个常见错误是未将转子转速和叶片变桨角初始化为仿真初始风速下的期望（平均）值，这会导致许多风力机控制器出现问题。

- 可视化输入（参见 :ref:`visualizing_input_output_OF`）。检查叶片和塔筒的质量和刚度分布是否符合预期。

- 验证系统的质量和刚度。叶片质量和塔顶质量显示在 ElastoDyn 汇总文件中。子结构的等效 6x6 矩阵可在 SubDyn 汇总文件中找到。

- 如果已将问题隔离到特定模块，请检查该模块汇总文件中提供的信息。大多数模块在其输入文件末尾都有一个名为 `SumPrint` 或类似的标志，以便将汇总文件写入磁盘。

- 减小时间步长。仿真时间步长需要根据系统中建模的频率进行调整（通常时间步长需要约为最快频率的十分之一）。BeamDyn 和 SubDyn 等模块通常需要较细的时间步长。
  除了减小时间步长，通常等效的做法是引入 1 个修正步（`NumCrctn`）。使用修正时，Jacobian 需要定期更新，例如将 `DT_UJac` 设为 100 个时间步。对于浮式系统，建议使用 `DT_UJac = 1/(10*f_pitch)`，其中 `f_pitch` 是浮式风力机俯仰方向的固有频率。


- 在真空（`CompInflow=0`、`CompAero=0`）和静止（`RotSpeed=0`）条件下对结构执行线性化（参见 :ref:`linearization_analysis_OF`），检查频率和阻尼是否在预期范围内。否则调整结构输入。

- 生成 VTK 输出以可视化风力机和 OpenFAST 使用的各种网格。VTK 输出通过 `WrVTK=1` 或 `WrVTK=2` 激活。VTK 文件写入主目录下的 `vtk*` 文件夹中，可使用 Paraview 进行可视化（参见 :ref:`visualizing_input_output_OF`）。




.. _moduleTroubleshooting:

特定模块的故障排除
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OpenFAST 的所有模块都需要一定程度的专业知识来确保仿真具有物理意义。
各模块的指南可参见本文档各处，特别参见：


- AeroDyn：:ref:`ad_modeling`

- HydroDyn：:ref:`hd-modeling-considerations`

- OLAF：:ref:`Guidelines-OLAF`

- SubDyn：:ref:`sd_modeling-considerations`

- FAST.Farm：:ref:`FF:ModGuidance`







脚本编写及其他支持工具
---------------------------------

NLR 维护了多个与 OpenFAST 配合使用的脚本仓库。
这些脚本可用于读取 OpenFAST 的输入和输出、可视化、生成多个仿真输入以及后处理。以下各节将详细介绍其中一些应用程序。


NLR 工具箱
~~~~~~~~~~~~~

- `openfast_toolbox <https://github.com/OpenFAST/openfast_toolbox>`__：底层 Python 工具集，用于与 OpenFAST 配合工作并执行简单操作，具有精细化粒度。

- `matlab-toolbox <https://github.com/OpenFAST/matlab-toolbox>`__：底层 Matlab 工具集，用于与 OpenFAST 配合工作。

- `WEIS <https://github.com/WEIS>`__：高层 Python 脚本，全称 Wind Energy with Integrated Servo-control。可执行风力机的多保真度协同设计。WEIS 是一个框架，结合了多个 NLR 开发的工具，用于实现浮式海上风力机的设计优化。

用户可查阅各仓库的文档，并在各自的 GitHub 页面上讨论相关问题。欢迎并鼓励社区对 NLR 仓库的贡献。



其他 NLR 相关工具
~~~~~~~~~~~~~~~~~~~~~~~

NLR 维护的其他仓库如下：

- `WISDEM <https://github.com/NLRWindSystems/WISDEM>`__：用于评估风电场整体能源成本（COE）的模型，还包含文件 IO、（DLC）工况生成、极值处理、可视化等功能。
- `ROSCO_toolbox <https://github.com/natlabrockies/ROSCO_toolbox>`__：用于配合 OpenFAST 支持的 `ROSCO <https://github.com/natlabrockies/ROSCO>`__ 控制器工作的工具。



第三方工具
~~~~~~~~~~~~~~~~~

- `pyDatView <https://github.com/ebranlard/pyDatView>`_：用于绘制 OpenFAST 输入输出文件、CSV 文件以及其他风能软件（Hawc2、Flex、Bladed）文件的工具。可同时打开多个文件以比较不同仿真的结果。

- `WindEnergyToolbox <https://gitlab.windenergy.dtu.dk/toolbox/WindEnergyToolbox>`_：DTU 开发的库，提供对不同文件格式的支持。

- `FASTTool <https://github.com/TUDelft-DataDrivenControl/FASTTool>`_：TUDelft 开发的用于 FASTv8 的 MATLAB GUI 和 Simulink 集成。

- `Continuous Section Field (CSF) <https://github.com/giovanniboscu/continuous-section-field>`_：Giovanni Boscu 开发的 Python 小工具，用于计算塔筒的截面属性。该工具侧重于：

   - 使用直纹曲面对非等截面构件进行建模。
   - 沿塔筒高度计算截面属性 :math:`(A, I, Ip)`。
   - 提供扭转刚度 :math:`( J )` 的简化估算。





.. _models_OF:

开源的 OpenFAST 模型
---------------------------

开源的 OpenFAST 风力机模型可在此处找到：

- `r-test <https://github.com/OpenFAST/r-test>`__：OpenFAST 的回归测试，包含 OpenFAST 及其驱动程序（AeroDyn、SubDyn、HydroDyn 等）的模型。此仓库并非旨在作为模型"数据库"，但其优势在于输入文件始终与最新的 `格式规范 <https://openfast.readthedocs.io/en/master/source/user/api_change.html>`_ 保持同步。以前版本的 OpenFAST 输入文件可通过此仓库的 git 标签访问。
- `IEA Wind Task 37 仓库 <https://github.com/IEAWindTask37>`_：包含 IEA Wind 3.4-MW、10-MW、15-MW 以及即将推出的 22-MW 参考风力机的 OpenFAST 模型。
- `openfast-turbine-models <https://github.com/natlabrockies/openfast-turbine-models>`_：开源风力机模型（开发中且可能过时）。








.. _visualizing_input_output_OF:

可视化输入和输出文件
------------------------------------



要可视化 OpenFAST 的输入和输出文件，可使用以下图形界面工具：

- `pyDatView <https://github.com/ebranlard/pyDatView>`_：用于绘制 OpenFAST 输入输出文件、CSV 文件以及其他风能软件（Hawc2、Flex、Bladed）文件的工具。可同时打开多个文件以比较不同仿真的结果。

OpenFAST 写入的 VTK 可视化文件可使用以下工具打开：

- `paraview <https://www.paraview.org/>`_：用于打开 OpenFAST 生成的 VTK 文件（即速度场和风力机几何）的工具。


对于高级用例，用户可能希望编写脚本读取和绘制输入文件。
Python 和 Matlab 工具分别由 `openfast_toolbox <https://github.com/OpenFAST/openfast_toolbox>`_ 和 `matlab-toolbox <https://github.com/OpenFAST/matlab-toolbox>`_ 提供。
在 Matlab 工具箱中，脚本 `FAST2Matlab.m` 和 `Matlab2FAST.m` 用于读写输入文件，脚本 `ReadFASTbinary` 用于打开二进制（`.outb`）输出文件。
这些仓库的 README 文件指向示例和更多文档。





.. _running_multiple_OF:

运行参数化研究和设计载荷工况（DLC）
------------------------------------------------------

参数化研究可以通过 `matlab-toolbox <https://github.com/OpenFAST/matlab-toolbox>`__
和 Python
`openfast_toolbox <https://github.com/OpenFAST/openfast_toolbox>`__
中提供用于读写 OpenFAST 输入文件的脚本来运行。openfast_toolbox 提供专用的 Python 脚本和示例来自动化此过程（更多信息参见仓库的 README）。
`WEIS <https://github.com/NLRWindSystems/WEIS>`__ 中的 `AeroelasticSE` 模块可以为标准中规定的设计载荷工况生成输入文件。
更多信息请查阅 WEIS 仓库。






.. _linearization_analysis_OF:

执行线性化分析
---------------------------------




背景
~~~~~~~~~~

许多应用需要系统的线性模型：特征值分析、频域分析、用于观测器的线性状态空间模型等。OpenFAST 的大多数模型是非线性的，因此需要对底层系统进行线性化。
线性化围绕给定的工作点进行，工作点对应于系统状态和输入的一组值（通常是仿真的某个特定时刻）。
线性化的输出是一个线性状态空间模型（四个矩阵，关联状态、输入和输出），在工作点附近有效。

由于转子在旋转，平衡解（如果存在）很可能是周期性的。
需要在一个旋转周期内的不同工作点（即不同方位角位置）进行线性化。

另一个复杂之处在于 OpenFAST 的某些状态处于旋转参考系中（例如 ElastoDyn 的叶片状态）。要获得固定（非旋转）参考系下系统的线性状态空间模型，需要应用多叶片坐标变换（MBC）。对于纯周期系统，可将 MBC 应用于不同方位角位置的线性化输出，将其组合形成固定参考系下的线性化系统。
注意 MBC 仅适用于 3 个或以上叶片。
1 或 2 个叶片需要 Floquet 理论，但 NLR 目前没有使用 Floquet 理论的后处理器。


.. note::
   我们当前推荐的实践是避免周期性，将模型简化为恒定平衡（例如，移除倾斜和重力）。MBC 仍然需要，但不要求在不同方位角位置使用不同的线性化。

线性化的输出之一是状态矩阵（`A`），它将系统状态与其时间导数相关联。
`A` 的特征值分析提供全系统模态振型及其频率和阻尼。

.. note::
    与线性有限元软件不同，OpenFAST 没有全系统刚度和质量矩阵的概念（某些模块有局部矩阵，但仅与模块相关）。底层方程组是非线性的，系统频率将随工作条件（如风速、转速）而变化。


以下各节详细介绍使用 OpenFAST 获取线性模型的过程，并将重点放在其用于获取系统模态频率和阻尼的应用上。




单次仿真的线性化模型（手动）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本节介绍使用 OpenFAST 生成系统线性化模型的关键步骤。

执行简单线性化分析的步骤如下：

1. 编辑主 `.fst` 文件，设置 `Linearize=True`

2. 将输出格式 `OutFmt` 设为 "ES20.11E3"。输出文件将以这种高精度写入，这是精确特征值分析所必需的。

.. warning::
    由于线性化输出文件为 ASCII 格式，特征值分析的结果将对输出精度（`OutFmt`）敏感。因此如上所述以较大精度设置此参数非常重要。

3. 有两种主要方法确定进行线性化的时间：

   - 使用 `CalcSteady=False`，用户通过 `NLinTimes` 和 `LinTimes` 指定进行线性化的时间
     （用户有责任提供系统处于平衡或周期性稳态的时间，即足够长的时间）；
   - `CalcSteady=True`（推荐方法），OpenFAST 将在系统达到周期性稳态（基于给定的容差 `TrimTol`）时自动开始线性化，并在一个转子旋转周期内执行 `NLinTimes` 次线性化。当使用控制器时，选项 `CalcSteady` 还会调整控制器输入（变桨、偏航或发电机转矩，基于输入 `TrimCase`），以使其达到初始条件中指定的转速。`TrimGain` 和 `TrimTol` 可能需要调整。`Twr_Kdmp` 和 `Bld_Kdmp` 可用于在稳态计算期间向塔筒和叶片添加阻尼。这有助于加速稳态计算，并且如果塔筒和/或叶片在其他情况下不稳定，则可能是必需的。一旦找到稳态解，`Twr_Kdmp` 和 `Bld_Kdmp` 将不会影响线性化结果（即线性解不会有额外的塔筒和叶片阻尼）。



4. 选择线性化次数。对于静止工况，`NLinTimes=1`；对于旋转工况，如果平衡点是周期性的，建议使用 `NLinTimes=36`（对应每 10 度方位角旋转一次线性化），否则 `NLinTimes=1`。如果 `CalcSteady=False` 且用户设置 `NLinTimes=36`，用户需要设置 `LinTimes` 的值，使其对应转子在 36 个独特方位角位置（基于转子转速）。


5. 对于典型的线性化，用户可设置 `LinInputs=0`、`LinOutputs=0`、`LinOutJac=False`、`LinOutMod=False`、`Twr_Kdmp=0`、`Bld_Kdmp=0`（参见 OpenFAST 输入文件文档）。
   设置 `LinInputs = LinOutputs = 0` 可避免生成 B、C 和 D 矩阵（无输入和输出）。
   当 `LinInputs=1` 时，线性化系统中固有的标准线性化输入集可用。这包括例如集体叶片变桨。当 `LinOutputs = 1` 时，各模块 `OutList` 部分的输出包含在线性化系统中。例如，通过在 ElastoDyn 的 `OutList` 中包含 `GenSpeed`，可以将 `GenSpeed` 纳入。要对 OpenFAST 的所有输入和输出进行线性化，设置 `LinInputs=2`、`LinOutputs=2`，代价是输出文件较大。

6. 在此 `.fst` 文件上运行 OpenFAST。OpenFAST 在执行每次单独线性化时将显示一条消息，并将带 `.lin` 扩展名的单独文件写入磁盘。

7. 建议检查常规输出文件 `.out` 或 `.outb`。如果 `CalcSteady=False`，用户应查看风力机在执行线性化的时刻是否确实已达到稳态（或周期性稳态）。如果 `CalcSteady=True` 且使用了控制器，用户可以检查转速是否确实收敛到目标 RPM，并可能选择调整 `TrimGain` 和 `TrimTol` 以供后续运行。

然后使用提供的 Python 或 Matlab 工具对线性化文件 `*.lin` 进行后处理。

.. note::
    并非 OpenFAST 的所有模块和选项在进行线性化时都可用。OpenFAST 将中止并显示错误消息，指示哪些选项可用。请相应调整输入文件。



后处理
~~~~~~~~~~~~~~

要获取系统的特征频率，用户可以打开 `.lin` 文件，提取状态矩阵 `A` 并执行特征值分析。对于旋转转子，需要打开单次仿真生成的所有不同方位角位置的 lin 文件，并使用 MBC 变换进行转换。我们为此提供了脚本。

当仅使用一个线性化文件时（例如静止状态下），可以使用脚本 `postproLin_OneLinFile_NoRotation`。该脚本位于 `matlab-toolbox/Campbell/example` 或 `openfast_toolbox/openfast_toolbox/linearization/examples/`。

当需要后处理多个线性化文件时（特别是对应不同方位角位置的多个文件），可以使用脚本 `postproLin_MultiLinFile_Campbel`，位于上述相同文件夹中。
如果线性化是在不同风速和 RPM 下进行的（通过不同的 OpenFAST 调用），此脚本也可使用。显示这些不同风力机工作条件下的频率和阻尼称为 Campbell 图。



Campbell 图
~~~~~~~~~~~~~~~~~

不久的将来，我们将提供一个专用工具来简化 Campbell 图的生成过程。

在此之前，为避免手动编辑不同风力机工作条件的输入文件，我们提供脚本 `runCampbell`，位于 `matlab-toolbox/Campbell/example` 或 `openfast_toolbox/openfast_toolbox/linearization/examples/`。
该脚本依赖一个包含参考 "fst" 文件的模板文件夹。该文件夹被复制，为每个风力机工作条件（风速/RPM）创建文件，运行 OpenFAST，并对线性化文件进行后处理。


脚本 `runCampbell` 生成一组 CSV 文件或一个 Excel 文件。脚本尝试识别模态（例如第 1 阶塔筒前后模态、第 1 阶挥舞模态等），但通常需要手动过程才能完全识别模态。此过程可能困难且繁琐。建议先在真空条件下并使用少量工作点进行仿真，以熟悉系统。

手动识别过程包括更改 CSV 文件 `Campbell_ModesID.csv`（或 Excel 电子表格 `ModesID`，如果使用 Excel 输出）。为避免在重新运行 `runCampbell` 时此文件被重写，建议将此文件重命名为 `Campbell_ModesID_Manual.csv`。脚本 `runCampbell` 中绘制 Campbell 图的部分可以调整以使用 "Manual" 文件。
建议使用 CSV 格式，因为这是与 Python 和 MacOS 兼容的方法。

手动识别过程包括在模态表中分配索引，其中索引对应排序后的模态频率列表。

例如，在 Excel 中打开 CSV 文件，`ModeID` 文件可能如下所示：

.. code::

    Mode Number Table
    Wind Speed (mps)   2.0   5.0   8.0
    1st Tower FA        0     0     0
    1st Tower SS        1     0     0

在此示例中，假设线性化在 2、5 和 8 m/s 下运行。表中的 "0" 表示模态未被识别。您可以查看文件 `Campbell_Summary.txt`，了解每个模态和工作点的频率、阻尼和"模态含量"。更多详细信息，可以打开各工作点的单独 CSV 文件（如果使用 Excel 格式，这些位于不同的工作表中）。
您可能会发现，在 2 和 5 m/s 下，塔筒前后方向是第二个频率，塔筒侧向方向是模态列表中出现的第一个频率。在 8 m/s 下，您可能会发现相反的情况。此时，您将编辑文件如下：

.. code::

    Mode Number Table
    Wind Speed (mps)   2.0   5.0   8.0
    1st Tower FA        2     2     1
    1st Tower SS        1     1     2


主要问题是如何确定哪个模态是哪个。这个问题没有真正的解决方案，以下是一些有助于识别的要素：

 - 系统频率在 0 m/s 和 0 rpm 时通常容易确定。系统频率将从该参考点随 RPM/WS/pitch 的变化而逐步变化。叶片后退和前进模态通常会显示等于 +/- 旋转速度频率的"分裂"，随着转速增加。挥舞方向的集体模态频率往往会由于离心刚化而随转子转速增加。

 - 当存在气动力时，叶片挥舞模态通常高度阻尼（显著高于摆振模态）。

 - 从一个工作点到下一个工作点，阻尼不会发生剧烈变化。

 - 塔筒模态不受工作条件变化的强烈影响。

 - 需要查看"模态含量"，以了解每个模态的能量所在。文件 `Campbell_Summary.csv` 显示了模态含量的摘要。在某些情况下，没有明显的最大值（显示关键字 `NoMax`）。此时，识别模态可能很困难。类似的内容可在各工作点文件中找到。

 - 模态的可视化可帮助识别（参见下一节）。但此过程可能仍然漫长。

一旦识别表设置完成，保存文件并绘制 Campbell 图。此过程可能需要迭代，直到获得令人满意的图表。在此过程中应该无需关闭 Excel。

我们意识到这个过程很漫长，感谢您的耐心，我们将努力简化此过程。



模态振型可视化
~~~~~~~~~~~~~~~~~~~~~~~~

目前可以进行模态振型可视化。需要为每次仿真生成 viz 文件，并重新运行 OpenFAST 以生成 VTK 文件。Matlab 脚本 `runCampbell` 辅助此过程，但目前提供的支持和文档有限。

用户可查阅以下示例：
-  https://github.com/OpenFAST/r-test/tree/main/glue-codes/openfast/5MW_Land_ModeShapes

及其相关文档：
- https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/5MW_Land_ModeShapes/vtk-visualization.md


其他参考资料
~~~~~~~~~~~~~~~~~~~~~

一些线性化问题曾在论坛和 GitHub Issues 中讨论过：

- https://wind.nrel.gov/forum/wind/

- https://github.com/OpenFAST/openfast/issues/480

感谢您的耐心，我们将努力简化线性化和 Campbell 图生成过程。





