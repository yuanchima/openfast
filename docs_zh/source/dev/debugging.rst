.. _debugging:

调试 OpenFAST
==================

作为一个 Fortran 项目，OpenFAST 的调试可能具有挑战性，且过程
因系统和环境而异。请记住，某些 OpenFAST
算例的内存占用可能相当大，可能需要很长时间
才能到达代码中感兴趣的点。仔细选择测试用例
可以节省大量时间。

编写一个小型 Fortran 程序来验证所有
调试工具已正确设置，然后再深入 OpenFAST 可能会有所帮助。请务必
通过执行诸如访问未分配数组元素的操作来模拟 bug，
并验证您可以使用给定的一组工具捕获该 bug。

.. note::

    所有系统的一个要求是以 **debug** 模式编译 OpenFAST。

.. _debugging_windows:

在 Windows 上调试
--------------------
使用 Intel 工具的 Windows 开发者可以使用 OpenFAST 仓库中包含的
Visual Studio 解决方案进行调试。这是一个直接的过程，
有来自 Intel 的大量支持。

否则，在 Unix 风格环境中编译的 Windows 开发者应
转到 :ref:`debugging_linux`。

.. _debugging_linux:

在 Linux 和 macOS 上调试
----------------------------
首先，通过将 ``CMAKE_BUILD_TYPE`` 设置为
"Debug" 以调试模式编译 OpenFAST。这可以在命令行上通过以下方式完成：

.. code-block:: bash

    cmake .. -D CMAKE_BUILD_TYPE=Debug

或使用 ``ccmake`` 打开命令行 CMake GUI 来更改它。

GNU 调试器 ``gdb`` 适用于调试编译后的代码。它有一个
全面的命令行界面，使开发者能够添加
断点并检查变量。

通过 IDE 驱动调试器可以使代码检查更加
高效。一个已知运行良好的 IDE 是 `Visual Studio Code <https://code.visualstudio.com>`__
配合 `Native Debug <https://marketplace.visualstudio.com/items?itemName=webfreak.debug>`__
扩展。您可以设置一个 `启动配置 <https://code.visualstudio.com/docs/editor/debugging#_launch-configurations>`__
，以便通过 IDE 调试特定的 OpenFAST 算例。为此，
打开启动配置并添加类似以下内容的块：

.. code-block:: json

        {
            "name": "AOC_WSt",
            "type": "gdb",
            "request": "launch",
            "printCalls": false,
            "showDevDebugOutput": false,
            "valuesFormatting": "prettyPrinters",
            "gdbpath": "gdb",
            "target": "${workspaceRoot}/build/glue-codes/openfast/openfast",
            "cwd": "${workspaceRoot}/build/reg_tests/glue-codes/openfast/AOC_WSt/",
            "arguments": "${workspaceRoot}/build/reg_tests/glue-codes/openfast/AOC_WSt/AOC_WSt.fst",
        }

macOS 特定配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
macOS 上的 GDB 在系统允许其接管进程之前需要一些配置。
建议通过 Homebrew 安装 ``gdb``：

.. code-block:: bash

    brew info gdb
    brew install gdb

完成后，请务必按照注意事项完成安装。
对于 ``gdb 8.2.1``，如下所示：

.. code-block:: bash

    ==> Caveats
    gdb requires special privileges to access Mach ports.
    You will need to codesign the binary. For instructions, see:

    https://sourceware.org/gdb/wiki/BuildingOnDarwin

    On 10.12 (Sierra) or later with SIP, you need to run this:

    echo "set startup-with-shell off" >> ~/.gdbinit

对于 macOS 上的 Native Debug，您需要以某种方式"破解"扩展，
以允许在 Fortran 文件中设置断点，方法是在 ``.vscode/settings.json`` 中添加此行：

.. code-block:: json

    {
        "debug.allowBreakpointsEverywhere": true
    }
