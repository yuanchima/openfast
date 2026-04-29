
.. _running-beamdyn:

运行 BeamDyn
============

本节讨论如何在个人计算机上获取和执行 BeamDyn。同时考虑软件的独立版本和与 FAST 耦合的版本。

下载 BeamDyn 软件
------------------

有两种形式的 BeamDyn 软件可供选择：独立版本和与 FAST 模拟器耦合的版本。虽然用户不一定需要两种形式，但如果要从零开始构建叶片模型，他/她可能需要熟悉并运行独立模型。即使最终目标是在 FAST 中进行陆上/海上风力机的气动-水动-伺服-弹性仿真，独立驱动程序对于模型故障排除也很有帮助。

运行 BeamDyn
-------------

运行独立 BeamDyn 程序
~~~~~~~~~~~~~~~~~~~~~~

独立 BeamDyn 程序 ``BeamDyn_Driver.exe`` 用于仿真用户输入模型的静态和动态响应，无需耦合到 FAST。与耦合版本不同，独立软件除了主 BeamDyn 输入文件和叶片输入文件外，还需要使用驱动文件。该驱动文件指定了通常由 FAST 提供给 BeamDyn 的输入，包括叶片根的运动和外加载荷。使用独立 BeamDyn 时，可以获得 BeamDyn 摘要文件和结果输出文件（有关 BeamDyn 输出文件的更多信息，请参见第 :ref:`bd-output-files` 节）。

在 DOS 命令提示符下运行独立 BeamDyn 软件，例如输入：

.. code-block:: bash

    >BeamDyn_Driver.exe Dvr_5MW_Dynamic.inp

其中，``Dvr_5MW_Dynamic.inp`` 是 BeamDyn 驱动输入文件的名称，如第 :ref:`driver-input-file` 节所述。

运行耦合到 FAST 的 BeamDyn
~~~~~~~~~~~~~~~~~~~~~~~~~~~

在命令提示符下运行耦合的 FAST 软件，例如输入：

.. code-block:: bash

    >openfast.exe Test26.fst

其中 ``Test26.fst`` 是 FAST 主输入文件的名称。该输入文件有一个功能开关，用于启用或禁用 FAST 中的 BeamDyn 功能，以及对 BeamDyn 输入文件的相应引用。有关更多信息，请参见 FAST 随附的文档。
