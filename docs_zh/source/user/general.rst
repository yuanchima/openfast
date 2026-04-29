

.. _general-reference-docs:

通用说明
~~~~~~~
.. toctree::
   :maxdepth: 1

   fast_to_openfast.rst
   api_change.rst
   input_file_overview.rst

以下列出了研讨会材料、旧版文档和其他资源。

- `OpenFAST 概述（NAWEA WindTech 2023） <https://forums.nrel.gov/t/modeling-workshops/523/27>`_
- `OpenFAST 概述（NAWEA WindTech 2022） <https://drive.google.com/file/d/1bD5a6rRg6cCKht9Ar8AFJQ8YrI4-wsFe/view>`_
- `OpenFAST 实用指南（NAWEA WindTech 2022） <https://drive.google.com/file/d/1FHovo6btDStPBh1Kv2swA09hIQRcZGZf/view>`_
- `OpenFAST 概述（NAWEA WindTech 2019） <https://drive.google.com/file/d/1wagMTOV_CLxSKzS2EEPFp2CExUo3JLpQ/view>`_
- `研讨会演示文稿 <https://drive.google.com/drive/folders/1BDDfcnIyvmZCwf7eFo0ISI7aF_FMAOvt>`_
- :download:`旧版 FAST v6 用户指南 <../../OtherSupporting/Old_FAST6_UsersGuide.pdf>`
- :download:`FAST v8 自述文件 <../../OtherSupporting/FAST8_README.pdf>`
- `海上浮式风机子结构柔性与构件级载荷功能在 OpenFAST 中的实现 <https://www.nrel.gov/docs/fy20osti/76822.pdf>`_
- `风机仿真 FAST 模块化框架：全系统线性化 <https://www.nrel.gov/docs/fy17osti/67015.pdf>`_
- `OpenFAST 中海上浮式风机的全系统线性化 <https://www.nrel.gov/docs/fy19osti/71865.pdf>`_
- :download:`FAST 与 Labview 结合使用 <../../OtherSupporting/UsingFAST4Labview.pdf>`
- :download:`OutListParameters.xlsx <../../OtherSupporting/OutListParameters.xlsx>` - 包含各模块的完整输出参数列表。



模块化框架
************************

这里提供了 OpenFAST 模块化框架的特定信息，包括相关的出版物、演示文稿和过往研究。

- `FAST 风机 CAE 工具的新型模块化框架 <https://www.nrel.gov/docs/fy13osti/57228.pdf>`_
- :download:`模块实现计划示例 <../../OtherSupporting/ModulePlan_GasmiPaperExamples.doc>`
- :download:`模块与网格映射线性化实现计划 <../../OtherSupporting/LinearizationOfMeshMapping_Rev18_Rev2.doc>`
- :download:`方向余弦矩阵（DCM）插值方法 <../../OtherSupporting/DCM_Interpolation/DCM_Interpolation.pdf>` - 总结了使用对数映射和矩阵指数进行方向余弦矩阵（DCM）插值的数学原理。
- :download:`设定点线性化开发计划 <../../OtherSupporting/DevelopmentPlan-SetPoint-Linearization.pdf>`
- :download:`OpenFAST 紧耦合求解器 <../../OtherSupporting/TightCoupling_Rev4.doc>`

.. - :download:`OpenFAST Steady State Solution <../../OtherSupporting/OpenFASTSteadyStateSolution_Rev7.doc>`


耦合框架与网格映射
**************************

关于耦合框架结构、模块变量 API、求解器和线性化的最新文档，请参见 :ref:`glue-code`。

- `FAST 模块化风机 CAE 工具：非匹配时空网格 <https://www.nrel.gov/docs/fy14osti/60742.pdf>`_
- `FAST 模块化风机仿真框架：新算法与数值示例 <https://dx.doi.org/10.2514/6.2015-1461>`_
- :download:`预测-校正方法 <../../OtherSupporting/ProposedPCApproach_Rev4.docx>`
