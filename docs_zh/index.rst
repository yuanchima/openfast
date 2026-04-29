.. OpenFAST documentation master file, created by
   sphinx-quickstart on Wed Jan 25 13:52:07 2017.

OpenFAST 文档
======================

.. only:: html

   :Version: |release|
   :Date: |today|

OpenFAST 是一个多物理场、多保真度的工具，用于仿真风力机的耦合
动态响应。实际上，OpenFAST 是耦合计算模块的
框架（或"胶水代码"），这些模块包括
气动力学、海上结构物的水动力学、控制与电气
系统（伺服）动力学以及结构动力学，从而实现时域中的
非线性气动-水动-伺服-弹性耦合仿真。OpenFAST 能够
分析多种风力机配置，包括两叶片或
三叶片水平轴转子、变桨或失速调节、刚性或
翘板式轮毂、上风向或下风向转子，以及格构式或管状塔筒。
风力机可以建模为陆上或海上固定式基础或浮式
子结构。

OpenFAST 成立于 2017 年，是一个基于 FAST v8（参见 :ref:`fast_to_openfast`）的
开源软件包。其耦合框架
和底层模块主要使用 Fortran
（遵循 2003 标准）编写，模块也可以用 C 或
C++ 编写。它的创建目标是成为一个由研究实验室、学术界和工业界
共同开发和使用的社区模型。它
由 National Lab of the Rockies 的一支专门团队管理。
我们的目标是确保 OpenFAST 是一个经过充分测试、文档完善
且自我维持的软件。为此，我们持续
改进现有代码的文档和测试覆盖率，并
期望新功能将包含充分的测试和
文档。如果您想贡献，请参见
:ref:`dev_guide` 以及 GitHub 上带有
`Help Wanted <https://github.com/OpenFAST/openfast/issues?q=is%3Aopen+is%3Aissue+label%3A"Help+wanted">`_
标签的开放 Issue。

以下链接提供了关于 OpenFAST 作为一个软件包的更多信息：

- `OpenFAST Github 组织 <https://github.com/OpenFAST>`_

源代码、发布版和回归测试示例可在以下 Github 页面获取：
- `Github 仓库 <https://github.com/OpenFAST/OpenFAST>`_
- `OpenFAST 发布说明和 Windows 可执行文件 <https://github.com/OpenFAST/openfast/releases>`_
- `OpenFAST 输入文件 <https://github.com/OpenFAST/r-test/releases>`_

**文档目录**

.. toctree::
   :numbered:
   :maxdepth: 2

   source/this_doc.rst
   source/install/index.rst
   source/working.rst
   source/user/index.rst
   source/testing/index.rst
   source/dev/index.rst
   source/license.rst
   source/help.rst
   source/acknowledgements.rst
