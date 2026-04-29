
.. _hd-getting-started:

安装与入门指南
================================
HydroDyn 包含在 OpenFAST 软件仓库中，由两个主要组件组成：

* `hydrodyn_driver` 是 HydroDyn 的独立可执行程序
* `hydrodynlib` 是 OpenFAST 模块库；通常通过 HydroDyn 驱动程序或 OpenFAST 耦合框架调用

安装说明请参见 :ref:`installation`。在需要指定安装目标的章节中，请使用 `hydrodyn_driver`。

运行 HydroDyn 驱动程序
~~~~~~~~~~~~~~~~~~~~~~~~~~~

HydroDyn 驱动程序提供简单的命令行界面：

.. code-block:: bash

    hydrodyn_driver <input_file>

其中 `input_file` 是 :ref:`hd-driver-input` 中描述的文件。还需要其他输入文件，包括 :ref:`hd-primary-input`。HydroDyn 的时间序列输出及其他输出在 :ref:`hd-output` 中说明。

运行与 OpenFAST 耦合的 HydroDyn
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

要运行启用了 HydroDyn 模块的 OpenFAST 仿真，必须开启 `CompHydro` 开关，并在 OpenFAST 主输入文件中提供 :ref:`hd-primary-input` 的路径：

.. code-block::

    # In the "Feature switches" section
    1               CompHydro   - Compute hydrodynamic loads (switch) {0=None; 1=HydroDyn}

    # In the "Input files" section
    "HydroDyn.dat"  HydroFile   - Name of file containing hydrodynamic input parameters (quoted string)

HydroDyn 的时间序列输出及其他输出在 :ref:`hd-output` 中说明。
