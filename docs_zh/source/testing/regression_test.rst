.. _regression_test:

回归测试
================
回归测试执行一系列旨在全面
描述 OpenFAST 及其模块功能的测试用例。跳转到以下
各节之一获取运行回归
测试的说明：

- :ref:`python_driver`
- :ref:`ctest_driver`
- :ref:`regression_test_example`
- :ref:`regression_test_windows`

每个本地计算的结果会与一组静态基线
结果进行比较。基线结果使用以下 OS、编译器和硬件
组合计算得到：

================== ============== ============================
 操作系统           编译器         硬件
================== ============== ============================
 Ubuntu 22.04       GNU 12.3.0     GitHub actions
================== ============== ============================

注意，如果在本地运行回归测试，由于编译器和硬件组合的不同，可能会存在微小的数值
差异，这可能导致少数
测试失败。如果在运行测试时使用 ``bokeh`` 软件包，您
将看到一个生成的 *html* 文件，其中包含显示任何
失败测试差异的图表（这些差异应该很小）。

编译器版本、具体数学库以及用于生成基线解决方案的硬件更多信息
记录在
`r-test 仓库文档 <https://github.com/openFAST/r-test>`__ 中。目前，
回归测试仅支持双精度编译。

回归测试系统可以通过 CMake（通过其内置的
测试驱动程序 CTest）或手动通过
自定义 Python 驱动程序来执行。两种系统在
测试方面提供类似的功能，但 CTest 集成提供了多线程、
自动化和通过 CDash 进行测试报告的功能。两种执行模式都需要一些
配置，详见以下各节。

在两种执行模式中，都会在编译目录中创建一个名为
``reg_tests`` 的目录，其中所有测试用例的输入文件被复制
（但不覆盖），
所有本地生成的输出都存储在此处。最终，CTest 和
手动执行程序都会调用 ``reg_tests`` 和 ``reg_tests/lib`` 中的
一系列 Python 脚本和库。其中一个脚本是 ``lib/pass_fail.py``，
它读取输出文件并对每个上报的通道计算范数。如果
最大范数大于给定的容差，则该特定测试
被报告为失败。失败标准如下所述。

.. code-block:: python

    difference = abs(testData - baselineData)
    for i in nChannels:
        if channelRange < 1:
            norm[i] = MaxNorm( difference[:,i] )
        else:
            norm[i] = MaxNorm( difference[:,i] ) / channelRange

    if max(norm) < tolerance:
        pass = True
    else:
        pass = False


测试环境
-------------------
我们建议使用 ``conda`` 创建本地环境来安装
*OpenFAST* 测试所需的软件包。您可以使用以下过程作为
在基于 Linux/MacOS 的系统上设置必要环境的粗略指南。

1. 为 *OpenFAST* 测试创建一个新的 ``conda`` 环境并安装 Python：

   - ``conda install python``

2. 从 ``<openfast>/build`` 目录，使用以下命令设置环境：

   - ``pip install ../requirements.txt``

      安装测试的基本依赖项。

   - ``pip install -e ../glue-codes/python/.``

      从 *OpenFAST* 仓库安装 ``pyOpenFAST`` 软件包

   - ``pip install -e ../openfast_io/.``
      从 *OpenFAST* 仓库安装 ``openfast_io`` 软件包


依赖项
------------
回归测试需要以下软件包（Python 模块另请参见
根目录中的 ``requirements.txt`` 文件）：

- CMake 和 CTest
- Python >=3.7
- numpy
- vtk
- bokeh>=2.4,!=3.0.0,!=3.0.1,!=3.0.2,!=3.0.3（可选）

除上述软件包外，还需要来自 *OpenFAST* 仓库的两个软件包。
我们建议如上节所述在 ``conda`` 环境中安装它们，
以免干扰系统 Python 安装。使用 ``pip install -e`` 命令将
使用本地目录安装，而不是将它们放在系统 Python 目录中。

- ``pyOpenFAST`` 是一个用于将 Python 接口连接到 *OpenFAST* 模块 c-bindings 库的软件包。
  在某些模块级别的测试中使用。
- ``openfast_io`` 是一个用于读写 *OpenFAST* 输入文件的软件包。
  在某些测试中使用。


.. _python_driver:

使用 Python 驱动程序执行
----------------------------
回归测试可以使用 ``openfast/reg_tests/manualRegressionTest.py`` 中的内置驱动程序手动执行。
此程序读取 ``openfast/reg_tests/r-test/glue-codes/openfast/CaseList.md`` 中的
用例列表文件。可以通过在该行开头添加 ``#`` 来
移除或忽略用例。驱动程序
程序包含多个可选标志，可以通过
执行帮助选项获取：

::

    >>>$ python manualRegressionTest.py -h
    usage: manualRegressionTest.py [-h] [-p [Plotting-Flag]] [-n [No-Execution]]
                                [-v [Verbose-Flag]] [-case [Case-Name]] [-module [Module-Name]]
                                Executable-Name Relative-Tolerance Absolute-Tolerance

    Executes OpenFAST or driver and a regression test for a single test case.

    positional arguments:
    Executable-Name       path to the executable
    Relative-Tolerance    Relative tolerance to allow the solution to deviate; expressed as order of magnitudes less than baseline.
    Absolute-Tolerance    Absolute tolerance to allow small values to pass; expressed as order of magnitudes less than baseline.

    optional arguments:
    -h, --help            show this help message and exit
    -p [Plotting-Flag], -plot [Plotting-Flag]
                            bool to include plots in failed cases
    -n [No-Execution], -no-exec [No-Execution]
                            bool to prevent execution of the test cases
    -v [Verbose-Flag], -verbose [Verbose-Flag]
                            bool to include verbose system output
    -case [Case-Name]     single case name to execute
    -module [Module-Name], -mod [Module-Name]
                            name of module to execute

.. note::

    对于 Jonkman 5-MW（以前称为 NREL 5-MW）风力机测试用例，必须
    编译外部 ServoDyn 控制器并将其放在适当的目录中，否则所有 Jonkman 5-MW（以前称为 NREL 5-MW）
    用例将在启动前失败。更多信息请参见
    `r-test 仓库文档 <https://github.com/openfast/r-test#note---servodyn-external-controllers-for-5mw_baseline-cases>`__，
    但请注意，以下三个 DISCON 控制器必须存在：

    .. code-block:: bash

        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON.dll
        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON_ITIBarge.dll
        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON_OC3Hywind.dll

.. _ctest_driver:

使用 CTest 执行
--------------------
CTest 包含在 CMake 中，主要是一组预配置的目标
和命令。要使用 CTest 驱动程序进行回归测试，请按
:ref:`installation` 中所述执行 CMake，但需要附加以下标志：
``-DBUILD_TESTING=ON``。

回归测试特定的 CMake 变量有：

::

    BUILD_TESTING
    CTEST_OPENFAST_EXECUTABLE
    CTEST_[MODULE]_EXECUTABLE 其中 [MODULE] 是模块名称
    CTEST_PLOT_ERRORS
    CTEST_REGRESSION_TOL

完整回归测试套件所需的一些额外资源
包含在 CMake 项目中。具体而言，外部 ServoDyn 控制器
必须为给定系统编译并放置在特定位置。因此，
请务必使用 ``install`` 目标执行编译命令：

.. code-block:: bash

    # 配置 CMake 并启用测试，接受所有其他测试相关
    # CMake 变量的默认值
    cmake .. -DBUILD_TESTING=ON

    # 编译并安装
    make install

.. note::

    提醒：对于 5MW 风力机测试用例，必须编译外部 ServoDyn 控制器
    并将其放在适当的目录中，否则所有 5MW
    用例将在启动前失败。更多信息请参见
    `r-test 仓库文档 <https://github.com/openfast/r-test#note---servodyn-external-controllers-for-5mw_baseline-cases>`__，
    但请注意，以下三个 DISCON 控制器必须存在：

    .. code-block:: bash

        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON.dll
        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON_ITIBarge.dll
        openfast/build/reg_tests/glue-codes/openfast/5MW_Baseline/ServoData/DISCON_OC3Hywind.dll

在 CMake 配置和编译之后，可以通过从
``build`` 目录运行 ``make test`` 或 ``ctest`` 命令来执行自动化回归测试。
如果要编译整个 OpenFAST 软件包，CMake 将
配置 CTest 在 ``openfast/build/glue-codes/openfast/openfast`` 找到新的二进制文件。
但是，如果意图是
仅编译测试套件，则应在 CMake 配置中的
``CTEST_OPENFAST_EXECUTABLE`` 标志下指定 OpenFAST 二进制文件。对于回归测试中包含的
每个模块，也有相应的
``CTEST_[MODULE]_NAME`` 标志。

当由 CTest 驱动时，回归测试可以通过从编译目录运行
各种形式的 ``ctest`` 命令来执行。基本命令
如下：

.. code-block:: bash

    # 运行完整的回归测试
    ctest

    # 禁用测试的实际执行；
    # 这有助于制定特定的 ctest 命令
    ctest -N

    # 运行完整的回归测试并输出详细信息
    ctest -V

    # 按名称运行测试，其中 TestName 是正则表达式（regex）
    ctest -R [TestName]

    # 运行所有测试，N 个测试并行执行
    ctest -j [N]

每个回归测试用例包含一系列标签，关联所有
使用的模块。标签可以在
``reg_tests/CTestList.cmake`` 中的测试实例化中查看，或通过以下命令查看：

.. code-block:: bash

    # 打印所有可用的测试标签
    ctest --print-labels

与特定标签对应的测试用例可以通过以下
命令执行：

.. code-block:: bash

    # 筛选与特定标签对应的测试用例
    ctest -L [Label]

标志可以组合使用，形成有用的变体，例如：

.. code-block:: bash

    # 运行所有使用 SubDyn 的用例并输出详细信息
    ctest -V -L subdyn

    # 在 16 个并发进程中运行所有使用 SubDyn 的用例
    ctest -j 16 -L subdyn

    # 运行名称为 "5MW_DLL_Potential_WTurb" 的用例并输出详细信息
    ctest -V -R 5MW_DLL_Potential_WTurb

    # 列出所有带 "beamdyn" 标签的测试
    ctest -N -L beamdyn

    # 列出匹配正则表达式 "bd" 的所有测试中包含的标签
    ctest -N -R bd --print-labels


自动化回归测试仅将新文件写入编译目录。
具体而言，所有本地生成的解都位于 ``openfast/build/reg_tests`` 中相应的
耦合框架或模块内。包含在
``openfast/reg_tests/r-test`` 中的基线解是严格只读的，不会
被自动化过程修改。

.. _regression_test_example:

回归测试示例
------------------------
以下示例说明了在基于 Unix 的系统上运行回归测试的
方法。但是，类似的步骤也可以在 Windows 上
使用 CMake 和 CTest 执行。在 Windows 上运行回归测试的
另一种方法见 :ref:`reg_test_windows`。

编译 OpenFAST 并使用 CTest 执行
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
以下示例假设用户完全从头开始。
以下命令下载源代码，使用 CMake 配置 OpenFAST 项目，
编译所有可执行文件，并执行完整的回归测试
套件。

.. code-block:: bash

    # 从 GitHub 下载源代码
    #    注意：默认分支为 'main'
    git clone --recursive https://github.com/openfast/openfast.git
    cd openfast

    # 如有必要，切换到另一个目标分支并更新 r-test
    git checkout dev
    git submodule update

    # 创建编译和安装目录，并进入编译目录
    mkdir build install && cd build

    # 配置 CMake 用于测试
    # - BUILD_TESTING - 开启
    # - CTEST_OPENFAST_EXECUTABLE - 接受默认值
    # - CTEST_[MODULE]_EXECUTABLE - 接受默认值
    cmake .. -DBUILD_TESTING=ON

    # 编译并安装
    make install

    # 使用 4 个并发进程执行完整测试套件
    ctest -j4

使用 CMake 配置并给定可执行文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
此示例假设用户拥有一个功能齐全的 OpenFAST 可执行文件
以及任何必要的库，但未下载源代码
仓库。这可能是当可执行文件在组织内
分发或从
`OpenFAST Release <https://github.com/openfast/openfast/releases>`__ 下载时的情况。
此处不进行任何编译，但测试套件将通过
CMake 配置以供 CTest 命令使用。

.. code-block:: bash

    # 从 GitHub 下载源代码
    #    注意：默认分支为 'main'
    git clone --recursive https://github.com/openfast/openfast.git
    cd openfast

    # 如有必要，切换到另一个目标分支并更新 r-test
    git checkout dev
    git submodule update

    # 创建编译目录并进入
    mkdir build && cd build

    # 使用 openfast/reg_tests/CMakeLists.txt 配置 CMake 用于测试
    # - BUILD_TESTING - 开启
    # - CTEST_OPENFAST_EXECUTABLE - 提供路径
    # - CTEST_[MODULE]_EXECUTABLE - 提供路径
    cmake ../reg_tests \
        -DBUILD_TESTING=ON \
        -DCTEST_OPENFAST_EXECUTABLE=/home/user/Desktop/openfast_executable \
        -DCTEST_BEAMDYN_EXECUTABLE=/home/user/Desktop/beamdyn_driver

    # 安装所需文件
    make install

    # 使用 4 个并发进程执行完整测试套件
    ctest -j4

.. _example_python_driver:

Python 驱动程序并给定可执行文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
此示例假设用户拥有一个功能齐全的 OpenFAST 可执行文件
以及任何必要的库，但未下载源代码
仓库。这可能是当可执行文件在组织内
分发或从
`OpenFAST Release <https://github.com/openfast/openfast/releases>`__ 下载时的情况。
不进行任何编译，但测试套件将使用内置的
Python 驱动程序执行。

.. code-block:: bash

    # 从 GitHub 下载源代码
    #    注意：默认分支为 'main'
    git clone --recursive https://github.com/openfast/openfast.git
    cd openfast

    # 如有必要，切换到另一个目标分支并更新 r-test
    git checkout dev
    git submodule update

    # 执行 Python 驱动程序
    cd reg_tests
    python manualRegressionTest.py -h
    # usage: manualRegressionTest.py [-h] [-p [Plotting-Flag]] [-n [No-Execution]]
    #                                [-v [Verbose-Flag]] [-case [Case-Name]] [-module [Module-Name]]
    #                                Executable-Name Relative-Tolerance Absolute-Tolerance
    #
    # Executes OpenFAST or driver and a regression test for a single test case.
    #
    # positional arguments:
    # Executable-Name       path to the executable
    # Relative-Tolerance    Relative tolerance to allow the solution to deviate; expressed as order of magnitudes less than baseline.
    # Absolute-Tolerance    Absolute tolerance to allow small values to pass; expressed as order of magnitudes less than baseline.
    #
    # optional arguments:
    #   -h, --help            show this help message and exit
    #   -p [Plotting-Flag], -plot [Plotting-Flag]
    #                         bool to include plots in failed cases
    #   -n [No-Execution], -no-exec [No-Execution]
    #                         bool to prevent execution of the test cases
    #   -v [Verbose-Flag], -verbose [Verbose-Flag]
    #                         bool to include verbose system output
    #   -case [Case-Name]     single case name to execute
    #   -module [Module-Name], -mod [Module-Name]
    #                         name of module to execute

    python manualRegressionTest.py ..\build\bin\openfast_x64_Double.exe 2.0 1.9

.. _reg_test_windows:

Windows 上运行的详细示例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
:ref:`example_python_driver` 示例可用于在 Windows 计算机上运行
回归测试。但是，更详细的分步
说明见 :ref:`regression_test_windows`。

.. toctree::
   :maxdepth: 1
   :hidden:

   regression_test_windows.rst

.. _new_regression_test_case:

添加测试用例
-----------------
在所有执行模式中，回归测试最终由
位于 ``openfast/reg_tests`` 目录中的一系列 Python 脚本驱动，
命名方案为 ``execute<Module>RegressionTest.py``。
添加新回归测试用例的第一步是验证
是否存在目标模块的脚本。如果不存在，应在
`OpenFAST Issues <https://github.com/openfast/openfast/issues>`_ 中提出 Issue
以与 NLR 团队协调创建此脚本。

下一步是将测试用例添加到
`r-test` 子模块中的适当位置。r-test 中的目录结构与
OpenFAST 中的目录结构镜像，因此模块级测试应放在
各自的模块目录中，而耦合框架测试放在
``r-test/glue-codes/openfast`` 中。注意现有测试的文件命名方案，
并相应调整新测试用例文件。具体而言，
Python 脚本可能期望主输入文件和输出文件名遵循特定的
约定。另外，需注意新测试用例输入文件中
的任何相对路径必须在 r-test 目录结构中有效。

一旦测试目录存在，测试用例必须向
适当的驱动程序注册。对于 OpenFAST 耦合框架测试，这既要在
CMake 中进行，也要在一个独立的测试用例列表中进行。对于 CMake，编辑文件
``openfast/reg_tests/CTestList.cmake``。额外的测试应
在该文件底部与模块或驱动程序对应的
部分中添加。对于 Python 驱动程序，新测试用例必须
添加到 ``openfast/reg_tests/r-test/glue-codes/openfast/CaseList.md`` 中。
此时，可以通过以下方式验证向 CTest 的注册：

.. code-block:: bash

    # 进入编译目录
    cd openfast/build

    # 运行 CMake 以接受对测试列表的新更改
    cmake .. -DBUILD_TESTING=ON  # 如果 BUILD_TESTING 标志之前已启用，此可省略

    # 列出已注册的测试，但不运行它们
    ctest -N

对于模块回归测试，唯一的执行选项是使用
CMake 驱动程序，因此请按上述说明编辑 ``CTestList.cmake``。

最后，r-test 子模块中的新测试用例必须添加到
r-test 仓库中。为此，请在 `r-test Issues <https://github.com/openfast/r-test/issues>`_ 中提出新的 Issue
请求 NLR 团队支持提交您的测试。
