
.. _running-subdyn:

运行 SubDyn
===============

本节讨论如何在个人计算机上获取和运行 SubDyn。包括软件的独立版本和与 FAST 耦合的版本。

下载 SubDyn 软件
--------------------------------

SubDyn 软件有两种形式可供选择：独立版本和与 FAST 模拟器耦合的版本。虽然用户不一定需要两种形式，但如果要从头开始构建子结构模型，可能需要熟悉并运行独立模型。独立版本也有助于模型故障排除，并且可能对有兴趣进行海上风电机组气动-水动-伺服-弹性耦合仿真的用户有所帮助。

用户可以参考 OpenFAST 安装文档来下载和编译 SubDyn。


运行 SubDyn
---------------

运行独立 SubDyn 程序
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

独立 SubDyn 程序 *SubDyn_win32.exe* 模拟用户输入模型的子结构动态响应，不需要与 FAST 耦合。与耦合版本不同，独立软件除了主 SubDyn 输入文件外，还需要使用驱动文件。该驱动文件指定了通常由 FAST 提供给 SubDyn 的输入，包括过渡段参考点的运动。使用独立 SubDyn 时，可以获得 SubDyn 汇总文件和结果输出文件（有关 SubDyn 输出文件的更多信息，请参见第 4 节）。

从 DOS 命令提示符运行独立 SubDyn 软件，例如输入：

.. code-block:: bash

    >SubDyn_win32.exe MyDriverFile.dvr

其中，*MyDriverFile.dvr* 是 SubDyn 驱动文件的名称，如 :numref:`sd_main-input-file` 中所述。SubDyn 主输入文件在第 :numref:`sd_driver-input-file` 节中描述。


运行与 FAST 耦合的 SubDyn
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

从 DOS 命令提示符运行耦合的 FAST 软件，例如输入：

.. code-block:: bash

    >FAST_Win32.exe Test21.fst

其中，*Test21.fst* 是 FAST 主输入文件的名称。该输入文件有一个功能开关，用于启用或禁用 FAST 中的 SubDyn 功能，以及对 SubDyn 输入文件的相应引用。有关更多信息，请参阅 FAST 随附的文档。
