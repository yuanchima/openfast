.. _unit_test:

单元测试
==========
在像 OpenFAST 这样动态和协作的软件包中，对多层代码的信心最好通过强大的单元
测试系统来实现。通过稳健的测试实践，整个 OpenFAST 社区可以
理解代码块背后的意图，并更快、更有信心和更稳定地调试或扩展功能。

OpenFAST 模块中的单元测试通过 `test-drive <https://github.com/fortran-lang/test-drive>`__ 实现。
test-drive 在 CMake 变量 ``BUILD_TESTING`` 开启（默认关闭）且 CMake 变量
``BUILD_UNIT_TESTING`` 开启（当 ``BUILD_TEST`` 开启时默认开启）时，
通过 CMake 与 OpenFAST 一起编译。

BeamDyn 和 NWTC Library 模块包含一些示例单元测试，应
作为未来开发和测试的参考。

依赖项
------------
单元测试需要以下软件包：

- CMake
- test-drive - 包含在 OpenFAST 仓库的 unit_test/test-drive 中

编译
---------
编译单元测试通过 CMake 处理，类似于一般的 OpenFAST 编译。
在配置 CMake 并开启 ``BUILD_TESTING`` 后，将为单元测试
框架中包含的每个模块创建新的编译目标，
命名为 ``[module]_utest``。然后，``make`` 目标进行测试：

.. code-block:: bash

    cmake .. -DBUILD_TESTING=ON
    make beamdyn_utest

这将在 ``openfast/build/unit_tests/beamdyn_utest`` 创建单元测试可执行文件。

执行
---------
要执行模块的单元测试，只需运行单元测试二进制文件。例如：

.. code-block:: bash

    >>>$ ./openfast/build/unit_tests/beamdyn_utest
    All tests PASSED

每个测试运行时会给出通过或失败状态。测试失败时输出错误消息。
失败情况显示以下输出：

.. code-block:: bash

    >>>$ ./unit_tests/beamdyn_utest
    # Testing: Crv
    Starting test_BD_CheckRotMat ... (1/6)
        ... test_BD_CheckRotMat [PASSED]
    Starting test_BD_ComputeIniNodalCrv ... (2/6)
        ... test_BD_ComputeIniNodalCrv [PASSED]
    Starting test_BD_CrvCompose ... (3/6)
        ... test_BD_CrvCompose [PASSED]
    Starting test_BD_CrvExtractCrv ... (4/6)
        ... test_BD_CrvExtractCrv [PASSED]
    Starting test_BD_CrvMatrixH ... (5/6)
    [Fatal] Uncaught error
    Code: 1 Message: A(1,1) simple rotation with known parameters: Pi on xaxis:
    Note: The following floating-point exceptions are signalling: IEEE_INVALID_FLAG IEEE_DIVIDE_BY_ZERO
    ERROR STOP

    Error termination. Backtrace:
    #0  0xffff9f70d08b in ???
    #1  0xffff9f70ddb3 in ???
    #2  0xffff9f70f333 in ???

添加单元测试
-----------------
每个新的、*可测试的*代码块（子程序或函数）都应包含单元测试。
什么是可测试的由开发者自行判断，但
Pull Request 审查过程的要素之一将是评估测试覆盖率。

新的单元测试可以添加到各模块中 ``src`` 目录旁的
``tests`` 目录中。例如，一个模块目录可能
按以下结构组织：

::

  openfast/
    └── modules/
        └── sampledyn/
            ├── src/
            │   ├── SampleDyn.f90
            │   └── SampleDyn_Subs.f90
            └── tests/
                ├── sampledyn_utest.F90
                ├── test_SampleDyn_Feature1.F90
                ├── test_SampleDyn_Feature2.F90
                └── test_SampleDyn_Feature3.F90

每个单元测试文件必须包含一个模块，该模块导出一个函数，根据
``test-drive`` 文档填充单元测试列表。这些模块
包含一个子程序，该子程序接受一个 ``error`` 参数，该参数由 ``test-drive``
提供的 ``check`` 子程序填充。``sampledyn_utest.F90`` 收集相邻模块中所有的
单元测试列表并运行它们。这些程序通过
``unit_tests/CMakeLists.txt`` 文件编译，因此所有相关模块和程序都在
其中指定。

有关如何构建和编译单元测试驱动程序的示例，请参考 ``BeamDyn`` 或
``NWTC Library`` 单元测试的现有单元测试。另请参阅 ``test-drive`` 文档：
`test-drive <https://github.com/fortran-lang/test-drive>`__。

开发和测试 OpenFAST 时需要考虑的一些有用主题：

- `测试驱动开发 <https://en.wikipedia.org/wiki/Test-driven_development#Test-driven_development_cycle>`__
- `关注点分离 <https://en.wikipedia.org/wiki/Separation_of_concerns>`__
