.. _testing:

测试 OpenFAST
================

OpenFAST 是一个复杂的软件，包含许多动态部件。为了
维持新旧代码的稳定性，源代码中直接包含了一个测试套件。
存在两个主要级别的测试：最高级别的回归测试和最低级别的
单元测试。回归测试将本地生成的结果与存储的
"基线"结果进行比较。这些测试可指示
全系统或子系统的响应是否发生了变化。单元测试专注于
单个子程序或代码块。这些测试不需要具有物理
真实性，而是侧重于数学和算法的运用。
包含这些测试的目的是快速捕获 bug 或结果的
意外变化。此外，测试可以帮助程序员
以可持续和可维护的方式设计其模块和子程序接口。

所有与回归测试对应的必要文件都包含在
``reg_tests`` 目录中。单元测试框架位于
``unit_tests`` 中，而实际的测试包含在被测试模块
对应的目录中。

OpenFAST GitHub 仓库使用 `GitHub Actions <https://github.com/openfast/openfast/actions>`_
自动为新的提交和 Pull Request 执行测试套件。
此云计算资源对所有
GitHub 用户可用，强烈推荐作为开发
工作流的一部分。在 OpenFAST 仓库中启用 GitHub Actions 后，只需
推送新提交即可触发测试。

.. toctree::
   :maxdepth: 1

   unit_test.rst
   regression_test.rst

获取和配置测试套件
----------------------------------------
测试套件的一部分通过
`git submodule` 链接到 OpenFAST 仓库。具体而言，包含以下仓库：

- `r-test <https://github.com/openfast/r-test>`__

.. tip::

    请务必使用 ``--recursive`` 标志克隆仓库，或在克隆后执行
    ``git submodule update --init --recursive``。

测试套件通过 CMake 配置，类似于一般的 OpenFAST
编译过程，附加一个 CMake 标志：

.. code-block:: bash

    # BUILD_TESTING     - 编译测试树（默认：OFF）
    cmake .. -DBUILD_TESTING:BOOL=ON

除此标志外，默认的 CMake 配置适用于大多数
系统。有关配置 CMake 目标的更多详细信息，请参见 :ref:`understanding_cmake`
部分。虽然单元测试必须使用 CMake 编译，因为它依赖
于源代码中包含的 `test_drive`，但回归
测试可以在没有 CMake 的情况下执行，如 :ref:`python_driver` 中所述。
