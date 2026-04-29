.. _user_guide:

用户文档
========

.. note::
    我们正在将旧版 FAST v8 文档迁移到本在线文档中。旧版 FAST v8 文档可在 https://www.nrel.gov/wind/nwtc.html 查看。

本部分包含 OpenFAST 模块耦合环境及其底层模块的文档，内容涵盖模型使用方法、基础理论，部分模块还包含验证说明。


.. toctree::
   :maxdepth: 1

   General considerations <general.rst>
   Glue Code <glue-code/index.rst>
   AeroDyn <aerodyn/index.rst>
   OLAF <aerodyn-olaf/index.rst>
   Aeroacoustics <aerodyn-aeroacoustics/index.rst>
   AeroDisk <aerodisk/index.rst>
   BeamDyn <beamdyn/index.rst>
   SubDyn <subdyn/index.rst>
   ExtPtfm <extptfm//index.rst>
   ElastoDyn <elastodyn/index.rst>
   HydroDyn <hydrodyn/index.rst>
   SeaState <seastate/index.rst>
   InflowWind <inflowwind/index.rst>
   MoorDyn <moordyn/index.rst>
   ServoDyn <servodyn/index.rst>
   Simplified ElastoDyn <simplified_elastodyn/index.rst>
   Structural Control <servodyn-stc/StC_index.rst>
   TurbSim <turbsim/index.rst>
   FAST.Farm <fast.farm/index.rst>
   C++ API <cppapi/index.rst>
   WaveTank <other/index.rst>


其他模块文档
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

以下模块目前没有正式文档，或者是由 NLR 和 OpenFAST 核心团队之外的组织贡献的。随着文档的补充，这些资源将被移动到合适的位置。如果有更新版本的外部资源可用，请提交 `GitHub Issue <https://github.com/openfast/openfast/issues>`_ 并提供新文档的相关信息。

- MAP++

  - `MAP++ 官方文档 <https://map-plus-plus.readthedocs.io/en/latest/index.html>`_
  - :download:`多段准静态缆索模型实现 <../../OtherSupporting/MAP/cable_model_development.pdf>`

- FEAMooring

  - :download:`理论手册 <../../OtherSupporting/FEAMooring/FEAM_Theory_Manual.pdf>`
  - :download:`用户指南 <../../OtherSupporting/FEAMooring/FEAM_Users_Guide.pdf>`

- OrcaFlex 接口:

  - :download:`用户指南 <../../OtherSupporting/OrcaFlex/User_Guide_OrcaFlexInterface.pdf>`

- IceFloe

  - :download:`冰载荷项目最终技术报告 <../../OtherSupporting/IceFloe/Ice_Load_Final_Report.pdf>`

- IceDyn

  - :download:`草案：FAST 冰模块手册 <../../OtherSupporting/IceDyn/IceDyn_Manual.pdf>`



NWTC 子程序库
~~~~~~~~~~~~~~~~~~~~~~~

- :download:`NWTC 库 - 子程序与函数简要概述 <../../OtherSupporting/NWTC_Library_Description.pdf>`
