.. _fast_to_openfast:

FAST v8 与向 OpenFAST 的迁移
======================================

本页面介绍了用于仿真风机耦合动态响应的计算机辅助工程工具 FAST v8 向 OpenFAST 的迁移过程。OpenFAST 由美国国家可再生能源实验室（NREL）的研究人员于 2017 年创立，得到了美国能源部风能技术办公室（DOE-WETO）的支持。2025 年秋季，该实验室更名为落基山国家实验室（NLR）。

FAST v8
-------

FAST v8 是用于仿真风机耦合动态响应的计算机辅助工程工具。它整合了气动模型、海上结构水动力模型、控制与电气系统（伺服）动态模型以及结构（弹性）动态模型，可实现时域内的非线性气动-水动-伺服-弹性耦合仿真。FAST 支持分析多种风机构型，包括两叶片或三叶片水平轴转子、变桨或失速调节、刚性或跷跷板轮毂、上风或下风转子、桁架式或管状塔筒等。风机可建模为陆上、海上固定式基础或浮式基础形式。FAST 基于从基本定律推导而来的先进工程模型，同时进行了适当的简化和假设，并在适用情况下辅以计算方法和试验数据进行补充。

气动模型使用来流风数据，求解转子尾流效应和叶素气动载荷，包括动态失速。水动力模型仿真规则或不规则入射波浪和海流，求解海上子结构的静水载荷、辐射载荷、衍射载荷和粘性载荷。控制与电气系统模型仿真控制器逻辑、传感器，以及变桨、发电机转矩、机舱偏航等控制装置的执行器，还有电气传动系统的发电机和变流器组件。结构动力学模型施加控制与电气系统的反作用力、气动和水动载荷，添加重力载荷，并仿真转子、传动链和支撑结构的弹性特性。所有模型之间通过模块化接口和耦合器实现耦合。

向 OpenFAST 的迁移
----------------------

OpenFAST 的发布标志着一次转型，旨在更好地支持来自研究机构、工业界和学术界的开源开发者社区，共同开发基于 FAST 的风机和风电场气动-水动-伺服-弹性工程模型。OpenFAST 的目标是为 FAST 开发提供坚实的软件工程框架，包括文档完善的源代码、广泛的自动化回归测试与单元测试，以及可靠的多平台、多编译器构建系统。

相对于 FAST v8.16，OpenFAST 包含以下组织架构层面的变更：

* 在 https://github.com/openfast 建立了新的 GitHub 组织

* OpenFAST 耦合框架代码、模块、模块驱动程序和编译工具都包含在同一个代码仓库中：https://github.com/openfast/openfast

* FAST 程序已更名为 OpenFAST（从 OpenFAST v1.0.0 版本开始）

* OpenFAST 采用了新的版本编号规则（从 OpenFAST v1.0.0 版本开始），例如 OpenFAST-v1.0.0-123-gabcd1234-dirty，其中：

  * v1.0.0 是"主版本号-次版本号-补丁号"格式的版本编号，对应 NREL（现为 NLR）在 GitHub 上打的标签提交

  * 123-g 表示自最近一次标签之后的额外提交次数（"-g"代表"git"）

  * abcd1234 是当前提交哈希值的前 8 个字符

  * dirty 表示存在未提交的本地修改

* 由于所有模块都包含在同一个代码仓库中，各个模块独立的版本号已被取消，现在统一使用 OpenFAST 版本号（从 OpenFAST v1.0.0 版本开始），但旧文档中可能仍会引用旧的模块版本号

* OpenFAST 回归测试基准结果（原称为认证测试或 CertTest）存放在独立的代码仓库中：https://github.com/openfast/r-test（从 OpenFAST v1.0.0 版本开始）

* 引入了子程序级别的单元测试（从 OpenFAST v1.0.0 版本的 BeamDyn 模块开始）

* 建立了在线文档系统以取代原有的 FAST v8 文档：http://openfast.readthedocs.io/；在向 OpenFAST 迁移期间，大部分用户相关文档仍通过 NWTC 信息门户提供：https://nwtc.nrel.gov（现已废弃）

* 通过 CMake 实现了在 macOS、Linux 和 Cygwin（Windows）系统上的跨平台编译

* 提供了 Visual Studio 项目（VS-Build）用于在 Windows 上编译 OpenFAST（从 OpenFAST v1.0.0 版本开始），但开发团队正在努力在未来版本中通过 CMake 自动生成 Visual Studio 构建文件

* `GitHub Issues <https://github.com/openfast/openfast/issues>`__ 已成为开发者报告和跟踪 bug、请求功能增强，以及询问与源代码、编译、回归测试/单元测试相关问题的主要平台；关于 OpenFAST 理论和使用的一般用户问题仍应通过论坛 https://forums.nlr.gov/ 处理

* 新增了一个 API，提供了通过 C++ 驱动程序运行 OpenFAST 的高级接口，有助于将 OpenFAST 与外部程序对接，例如用 C++ 编写的 CFD 求解器（从 OpenFAST v1.0.0 版本开始）


OpenFAST 发布说明
--------------------------

本部分概述了每个标签版本中对 OpenFAST 做出的重要修改。

v0.1.0（2017 年 4 月）
```````````````````

从算法角度来看，OpenFAST v0.1.0 是与 FAST v8.16 关系最密切的版本。

* 组织架构变更：

  * 在 https://github.com/openfast 建立了新的 GitHub 组织

  * OpenFAST 耦合框架代码、模块、模块驱动程序和编译工具都包含在同一个代码仓库中：https://github.com/openfast/openfast

  * 通过 CMake 实现了在 macOS、Linux 和 Cygwin（Windows）系统上的跨平台编译

  * 建立了在线文档系统以取代原有的 FAST v8 文档：http://openfast.readthedocs.io/

  * `GitHub Issues <https://github.com/openfast/openfast/issues>`__ 已成为开发者报告和跟踪 bug、请求功能增强，以及询问与源代码、编译、回归测试/单元测试相关问题的主要平台；关于 OpenFAST 理论和使用的一般用户问题仍应通过论坛 https://forums.nlr.gov/ 处理

* AeroDyn v15 气动模块得到了重大更新。叶素/动量理论（BEMT）求解算法有以下改进：

  * BEMT 现在支持局部叶片坐标系 x 方向未扰动速度（Vx）小于零的情况

  * 当无法找到有效的入流角 (:math:`\phi`) 时，BEMT 不再中止运行；这种情况下，入流角将通过几何方法计算（不考虑诱导效应）

  * 入流角 (:math:`\phi`) 现在在首次调用时初始化，而不是默认使用 :math:`\phi` = 0，从而在仿真启动阶段获得更好的结果

  * 启用轮毂和/或叶尖损失时（HubLoss = True 和/或 TipLoss = True），在轮毂和/或叶尖位置，切向诱导因子（a'）设置为 0 而不是 -1（轴向诱导因子（a）在轮毂和/或叶尖位置仍设置为 1）

  * BEMT 求解效率得到提升

  * 此外，修复了 AeroDyn v15 中的多个 bug，包括：

    * 修复了同时启用轮毂和/或叶尖损失（HubLoss = True 和/或 TipLoss = True）以及 Pitt/Peters 偏斜尾流修正（SkewMod = 2）时，BEMT 不会修改轮毂和/或叶尖位置诱导因子的 bug

    * 修复了 AeroDyn 耦合到 OpenFAST 并启用冻结尾流（FrozenWake = True）时，线性化分析后时间序列受到影响的 bug

* BeamDyn 有限元叶片结构动力学模型的源代码进行了全面清理。修复了结构阻尼诱导刚度中非对角项的 bug（该 bug 会导致阻尼力随梁位移发生变化）。

* 引入了新的用户自定义平台载荷模块（ExtPtfm）。ExtPtfm 允许用户指定 6×6 的附加质量、阻尼和刚度矩阵，以及 6×1 的载荷向量，用于定义施加到 ElastoDyn 塔筒底部/平台的载荷，例如支持通过超单元表示（超单元由外部软件生成）对子结构或基础进行建模。ExtPtfm 还为用户提供了一个可定制的模块，用于实现更高级的平台施加载荷功能。可以通过在 FAST 主输入文件中将 CompSub 设置为 2（新选项），并将 SubFile 设置为包含平台矩阵和载荷时间历程的文件名来启用 ExtPtfm 模块，但将 CompSub 设置为 2 需要禁用水动力模块（将 CompHydro 设置为 0）。请注意，CompSub 选项 2 的引入是输入文件的一个小变更（也是 OpenFAST v0.1.0 中唯一的输入文件变更），但 MATLAB 转换脚本尚未更新。

* 在 ServoDyn 控制与电气系统模块中，修正了输出参数 YawMom 的单位和符号

* 在 InflowWind 来流风模块中，修复了使用 Bladed 格式的 TurbSim 生成塔筒风数据文件的功能

* 对 ElastoDyn 中的错误检查进行了小修复


v1.0.0（2017 年 9 月）
```````````````````````

* 组织架构变更：

  * FAST 程序已更名为 OpenFAST

  * OpenFAST 采用了新的版本编号规则（详情参见第 4.3.2 节）

  * OpenFAST 回归测试基准结果（原称为认证测试或 CertTest）存放在独立的代码仓库中：https://github.com/openfast/r-test

  * 引入了子程序级别的单元测试（从 BeamDyn 模块开始）

  * 在线文档（http://openfast.readthedocs.io/en/latest/index.html）得到了全面更新，新增了安装、测试、用户指南（AeroDyn、BeamDyn、从 FAST v8 迁移、发布说明）以及开发者指南等内容

  * 更新了在 macOS、Linux 和 Cygwin（Windows）系统上使用 CMake 编译 OpenFAST 的脚本，新增了单精度编译和使用 Spack 构建的功能

  * 提供了 Visual Studio 项目（VS-Build）用于在 Windows 上编译 OpenFAST

  * TurbSim 已被包含在 OpenFAST 代码仓库中

* AeroDyn 气动模块得到了更新：

  * 为海洋流体动能（MHK） turbine 添加了空化检查功能。包括在 AeroDyn 主输入文件中新增了输入参数 CavitCheck、Patm、Pvap 和 FluidDepth，在翼型数据文件中新增了 Cpmin 参数（当 CavitCheck = True 时需要），以及新增了最小压力系数、临界空化和叶片节点局部空化数的输出通道。请注意，本次输入文件变更是 OpenFAST v1.0.0 中唯一的输入文件变更，但 MATLAB 转换脚本尚未更新。

  * 修复了塔筒风载荷计算中使用塔筒位移代替塔筒速度的 bug

  * 用于计算塔筒对叶片局部来风影响的模型检测到塔筒撞击时，现在将其视为致命错误而非严重错误

  * 修复了非定常翼型气动模型中的小 bug

* BeamDyn 有限元叶片结构动力学模块有额外变更：

  * 源代码进一步清理，包括修改内部坐标系以匹配 IEC 标准（局部 z 轴沿变桨轴方向）

  * 梯形点现在由叶片站位而非关键点正确定义

  * 根据 GitHub issue #10（https://github.com/OpenFAST/openfast/issues/10）修正了叶尖旋转输出

  * 修复了 BeamDyn 驱动程序在涉及旋转叶片场景下的问题

  * BeamDyn 在单精度下不再产生数值"尖峰"，因此使用 BeamDyn 时不再需要以双精度编译 OpenFAST

* ElastoDyn 结构动力学模型有小幅更新：

  * 部分作为 BeamDyn 模块输入的模块级输出精度从单精度提升到双精度，以避免在单精度下运行 BeamDyn 时出现数值"尖峰"

  * 对错误检查进行了小修复

* ServoDyn 控制与电气系统模块有小幅更新：

  * 根据 GitHub issue #40（https://github.com/OpenFAST/openfast/issues/40），修复了 ServoDyn 发送给 Bladed 风格 DLL 控制器的发电机转矩和电功率数值的问题

  * 对错误检查进行了小修复

* OpenFAST 驱动/耦合框架代码得到了更新：

  * 在 OpenFAST 驱动程序的前几个时间步中添加了校正步骤，以解决 BeamDyn 的初始化问题（即使 NumCrctn = 0 时也有效）

  * 根据 GitHub issue #8（https://github.com/OpenFAST/openfast/issues/8），修复了载荷的 Line2 到 Point 映射中的 bug。此前，增强网格是通过不正确的投影形成的，因此在某些情况下会导致载荷传递异常。这可能会导致 ElastoDyn 与 AeroDyn 之间和/或 HydroDyn 与 SubDyn 之间的耦合出现问题

  * 添加了一个未正式文档化的功能，支持写入无压缩的二进制输出，以支持新的回归测试。可以通过在 FAST 主输入文件中将 OutFileFmt 设置为 0 来使用新格式。

* 新增了一个 API，提供了通过 C++ 驱动程序运行 OpenFAST 的高级接口。C++ API 的主要目的是帮助将 OpenFAST 与外部程序对接，例如通常用 C++ 编写的 CFD 求解器。

* TurbSim 来流风湍流预处理器得到了更新：

  * 修正了 API 频谱

  * 修复了多个小 bug。


OpenFAST 未来展望
-------------------------

我们的目标是持续改进 OpenFAST 文档，并提高自动化单元测试和回归测试的覆盖率。为了提高测试覆盖率并保持软件的健壮性，我们要求：

* 新模块需要由模块开发者提供足够的模块特定单元测试和回归测试，以及相应的 OpenFAST 回归测试；

* bug 修复需要包含相应的单元测试；

* 新功能/能力需要包含相应的单元测试和回归测试。我们正在为 BeamDyn 模块添加更完善的测试，作为新模块要求的示范。

单元测试将采用 pFUnit 框架（https://sourceforge.net/projects/pfunit）。

目前 OpenFAST 提供项目和解决方案文件，以支持用户使用 Visual Studio 进行开发和编译。不过，开发团队正在持续努力，在未来版本中通过 CMake 自动生成 Visual Studio 构建文件。

如有关于 OpenFAST 开发计划的问题，请联系 `Jason.Jonkman@nlr.gov <mailto:jason.jonkman@nlr.gov>`_ 或 `Andy.Platt@nlr.gov <mailto:andy.platt@nlr.gov>`_。
