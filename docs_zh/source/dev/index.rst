.. _dev_guide:

开发者文档
=======================

**作为 OpenFAST 的开发者，我们的目标是确保它是一个经过充分测试、文档完善且自我维持的软件。**
为此，我们持续努力改进文档和测试覆盖率，同时添加功能和改进。文档的这一部分概述了我们为外部开发者与 NLR OpenFAST 团队合作进行代码开发所建立的流程和程序。

如果您希望帮助进行 OpenFAST 的一般开发或参与某个特定特性，请首先按照适用于您机器的
:doc:`安装说明 <../install/index>` 安装 OpenFAST。接下来，
按照 :doc:`测试说明 <../testing/index>` 运行测试套件以验证安装是否有效。在 OpenFAST 编译期间，我们
鼓励阅读 :ref:`development_philosophy` 部分，以
了解个人和协作开发的一般工作流程。
最后，请务必查看 :doc:`GitHub 工作流程 <github_workflow>` 以
避免任何合并或代码冲突。

随着 NLR、行业合作伙伴和大学之间的开发并行进行，NLR 依靠 GitHub 协调各方工作：

- `GitHub Issues <https://github.com/openfast/openfast/issues>`_ 是提出使用或开发问题、报告 bug 和
  建议代码改进的地方
- `GitHub Pull Requests <https://github.com/openfast/openfast/pulls>`_
  是与 OpenFAST 团队互动、将新代码
  合并到主仓库的地方。

如有关于 OpenFAST 的其他问题，请联系
`Mike Sprague <mailto:michael.a.sprague@nrel.gov>`_。

.. tip::

    以下各节提供了有关工作流程和
    开发技巧的宝贵指导，使过程更高效和
    有效：

    - :ref:`github_workflow`
    - :ref:`code_style`
    - :ref:`debugging`

.. _development_philosophy:

开发理念与指南
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OpenFAST 旨在成为一个自我维持、由社区开发的软件。
虽然 NLR OpenFAST 团队作为仓库的守门人，我们
积极鼓励社区分享新想法和贡献代码。
此处概述了贡献代码的注意事项。

与 NLR 互动
-------------------

社区代码贡献的过程始于直接
与 NLR OpenFAST 团队互动，以定义工作范围并协调
开发工作。这一点尤其重要，因为许多团队
同时在 OpenFAST 上工作。通过尽早互动，所有开发者都可以
保持最新状态并最大限度地减少代码合并期间的冲突。
首选的沟通方式是 `GitHub Issues <https://github.com/openfast/openfast/issues>`_。
初始帖子应包含有关计划中
开发工作的所有相关信息、软件中将受影响的区域
以及任何模型验证材料。有关描述计划工作的更多信息，请参见
:ref:`development_plan`。

NLR OpenFAST 团队始终在进行需要大部分精力的内部项目，
但我们将尽一切努力
在合理的时间范围内与社区互动并支持开发工作。
在发布 Issue 后，NLR OpenFAST
团队可能会联系您安排会议讨论细节。

.. _development_plan:

开发计划 / 实施计划
--------------------------------------
NLR 的重大代码开发工作始于制定
详细的实施计划，有若干计划可供
下载参考：

- :download:`OpenFAST 气动力线性化开发计划 <../../OtherSupporting/AeroDyn/AeroLin_2019-12.pdf>`
- :download:`FAST.Farm 开发计划 <../../OtherSupporting/FAST.Farm_Plan_Rev25.doc>`.
- :download:`实施计划 - HydroDyn 中的二阶力 <../../OtherSupporting/HydroDyn/HydroDyn_2ndOrderForces_Plan.pdf>`
- :download:`实施计划 - HydroDyn 中的二阶波浪运动学 <../../OtherSupporting/HydroDyn/WAVE2_document.pdf>`

在 OpenFAST 模块化框架内的良好计划应
遵循 :download:`NWTC 程序员手册 <../../OtherSupporting/NWTC_Programmers_Handbook.pdf>`
所使用的定义和命名法。它应传达以下信息：

- 说明模块是用于松耦合、
  时间推进的紧耦合和/或线性化。
- 定义模块的输入（包括初始化）、
  输出（包括初始化）、状态（连续、
  离散和约束）及参数，包括单位。
- 列出模块的示例输入文件。
- 以框架所要求的形式解释模块的数学表述，包括
  Jacobian 矩阵（用于紧耦合和线性化）。
- 规定模块的输入如何从
  其他特定模块的输出中导出
- 识别任何潜在的数值问题以及如何在
  代码中避免它们。
- 使用伪代码（而非实际代码）列出模块的子程序，
  包括识别哪些数学公式
  由哪些子程序使用，并描述求解过程中使用的算法。

这些信息非常有用，因为在更改源代码之前更容易审查、迭代
和就计划达成一致。此外，
实施计划将极大地辅助编程工作，是
编写用户和开发者文档的有用起点。

良好提交的品质
------------------------------

开发工作应在整个
开发过程中包含充分的测试。在可能的情况下，新的子程序应包含单元级测试，
并应定期运行现有的回归测试以确保
全系统行为没有意外改变。对于新特性，
应添加额外的回归测试以覆盖新代码。
如果回归测试结果以预期的方式改变，基线
结果应在本地和 `openfast/r-test <https://github.com/openfast/r-test>`_
仓库中更新。`r-test README <https://github.com/openfast/r-test#updating-the-baselines>`_
描述了更新基线，本文档中的 :ref:`testing`
部分包含有关测试的更多详细信息。

新代码应从开发者和用户的
角度考虑稳健性。以下是代码开发期间需要考虑的一些问题：

- 其他开发者是否清楚如何使用您的子程序？
- 您的新代码是否表现出清晰且可预测的行为？
- 您的代码在不同质量的数据下表现如何？
- 您的代码如何影响仿真的性能？

此外，新代码应包含
用户和开发者文档。用户文档包括理论、建模指南
以及任何输入和输出的描述。用户文档应
作为 :ref:`build_doc` 中描述的在线文档的一部分包含在内。
开发者文档通常包含在源代码的注释中。
这应描述子程序 API（输入和输出）以及
任何不清楚的算法或代码行。问问自己，
如果您两年不再看这段代码，您需要知道什么才能完全理解它。

提交审查和 NLR 反馈
-----------------------------------

新代码可以通过
按照 :ref:`github_workflow` 中所述
发起 `Pull Request <https://github.com/openfast/openfast/pulls>`_ 提交给 NLR OpenFAST 团队审查。
我们将审查代码的
准确性、有效性、质量和稳健性。审查开源
代码贡献可能很困难，因此值得审查
自己的代码并考虑哪些信息能帮助您
确定它是否准备好合并。

审查过程始于确保自动化
测试在 `GitHub Actions <https://github.com/openfast/openfast/actions>`_ 中通过。
**请在请求审查之前确保所有自动化测试均通过。**
之后，该过程将涉及审查者和提交者之间的一些沟通，
可能包括要求提供有关背景或验证的更多信息，
以及在 Pull Request 中针对特定代码行
发表评论以获取额外的见解。

在提交者和审查者之间达成共识后，
Pull Request 将被合并到目标分支（通常为
`dev`），Pull Request 将被关闭。大功告成！
当 `dev` 分支合并到 `main` 时，此更改将包含在
OpenFAST 的后续发布版本中。

Bug 修复
---------

如果您在代码中发现 bug，重要的是在
`GitHub Issue <https://github.com/openfast/openfast/issues>`_
和最小化测试中充分描述它。在提交 bug 修复之前，
先提交揭示该 bug 的新测试。此测试应该失败。
然后，提交 bug 修复并展示测试通过。git 提交
历史应如下所示（从下到上推进）：

.. mermaid::

  gitGraph BT:
  options
  {
    "nodeSpacing": 60,
    "nodeFillColor": "white",
    "nodeStrokeWidth": 2,
    "nodeStrokeColor": "#747474",
    "lineStrokeWidth": 2,
    "branchOffset": 30,
    "lineColor": "grey",
    "leftMargin": 20,
    "branchColors": ["#007bff", "#ff2d54"],
    "nodeRadius": 5,
    "nodeLabel": {
      "width": 75,
      "height": 100,
      "x": -25,
      "y": 0
    }
  }
  end

  commit
  branch dev
  checkout dev
  commit "Merge pull request #123"
  commit "Merge pull request #124"
  branch bugfix
  checkout bugfix
  commit "Add unit test exposing out of bounds error"
  commit "Fix out of bounds error in array"
  checkout dev
  commit "Merge pull request #125"
  merge bugfix

更多信息见 :ref:`testing` 和 :ref:`github_workflow`。

附加指南
-------------------

以下各节提供了关于开发源代码、
与 NLR OpenFAST 团队和其他社区贡献者互动以及
一般调试和构建功能的扩展指南。

.. toctree::
    :maxdepth: 1

    github_workflow.rst
    code_style.rst
    build_doc.rst
    types_files.rst
    debugging.rst
    performance.rst
    versioning.rst




API 参考
~~~~~~~~~~~~~
源代码中的一些子程序和派生类型具有内嵌
文档，通过 Doxygen 编译。虽然这部分的
文档始终在开发中，现有的 API 参考可
在以下页面找到：

- `主页 <../../html/index.html>`_
- `类型索引 <../../html/classes.html>`_
- `源文件 <../../html/files.html>`_

C++ API 参考
~~~~~~~~~~~~~~~~~~~
提供 C++ API 文档。

.. toctree::
   :maxdepth: 1

   cppapi/index.rst



其他文档
~~~~~~~~~~~~~~~~~~~
存在一些额外的文档，可能对寻求更深入
理解求解器和数学原理的开发者有用。

- :download:`NWTC 程序员手册 <../../OtherSupporting/NWTC_Programmers_Handbook.pdf>`
   这是 FAST 8 编程指南的概述。虽然 OpenFAST 中的一些语法和细节已发生变化，
   但本指南的大部分内容仍然相关。
- :download:`OutListParameters.xlsx <../../OtherSupporting/OutListParameters.xlsx>`
   此 Excel 文件包含每个模块的完整输出列表。它用于生成
   每个模块的输出通道列表处理的 Fortran 代码（此代码通常在
   _IO.f90 文件中）。相关的 MATLAB 脚本见
   `matlab-toolbox <https://github.com/OpenFAST/matlab-toolbox>`__ 仓库中的 *Utilities/GetOutListParameters.m*。
