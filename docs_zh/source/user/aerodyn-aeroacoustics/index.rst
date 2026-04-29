
.. _AeroAcoustics:

OpenFAST 气动噪声模型
======================

.. only:: html

   本报告描述了 OpenFAST 中新发布模型的理论和应用，该模型用于仿真任意风力机转子产生的气动噪声。OpenFAST 是由美国国家可再生能源实验室（National Renewable Energy Laboratory，NREL）积极开发的完全开源、公开可用的风力机分析工具。气动噪声模型同样完全开源且公开可用，基于过去三十年的研究工作开发。包含以下基于频率的模型：湍流来流、湍流边界层-后缘、层流边界层-涡脱落、叶尖涡以及后缘钝度-涡脱落噪声机制。还包含一个简单的指向性模型。

   通过仿真国际能源署风能任务 37 陆上参考风力机的气动噪声排放，对噪声模型进行了验证。还展示了本文实现与德国慕尼黑工业大学风能研究所现有实现之间的代码对比。

   本文档来源于 NREL 技术报告 TP-5000-75731，作者为 P. Bortolotti 等人（`https://www.nrel.gov/docs/fy20osti/75731.pdf <https://www.nrel.gov/docs/fy20osti/75731.pdf>`_）



.. toctree::
   :maxdepth: 2

   acronyms.rst
   symbols.rst
   01-introduction.rst
   02-noise-models.rst
   03-model-verification.rst
   04-conclusions.rst
   App-usage.rst
   refs.rst
