.. _build_doc:

开发文档
========================
OpenFAST 文档托管在
`readthedocs <http://openfast.readthedocs.io/>`_ 上。每当有新的提交时，它通过
``readthedocs`` 构建系统自动从 ``main`` 和 ``dev``
分支生成。本文档使用
`reStructuredText <http://www.sphinx-doc.org/en/main/usage/restructuredtext/basics.html>`_
标记语言。

本地构建本文档
-----------------------------------
文档使用 `Sphinx <http://sphinx-doc.org>`__ 编译，这是一个
基于 Python 的工具。使用 ``pip`` 或其他 Python 包
管理器安装它以及 ``openfast/docs/requirements.txt`` 中列出的其他必需 Python 包。

以下附加包是可选的，不包含在 requirements
文件中：

- `Doxygen <http://www.stack.nl/~dimitri/doxygen/>`__
- `Doxylink <https://pythonhosted.org/sphinxcontrib-doxylink/>`__
- `Graphviz <http://www.graphviz.org>`__
- `LaTeX <https://www.latex-project.org>`__

Doxygen 和 Graphviz 可以直接从它们的网站安装，或使用
``brew``、``yum`` 或 ``apt`` 等包管理器安装。

本地构建文档的结果将是一组
HTML 文件及其附带的必需文件。主 HTML 文件
将位于 ``openfast/build/docs/html/index.html``。此文件可以
用任何浏览器打开，像浏览其他网站一样查看和导航本地生成的
文档。

纯 Python 构建
~~~~~~~~~~~~~~~~~
如果系统中没有 CMake 和 Make，可以直接使用 `sphinx`
生成文档。

.. note::

    此方法不会通过 Doxygen 生成 API 文档。

首先，将目录结构调整为标准的 OpenFAST 构建布局，
在 ``openfast/build`` 处创建一个目录。然后，进入
``openfast/build`` 并运行以下命令：

.. code-block:: bash

    # sphinx-build -b <builder-name> <source-directory> <output-directory>
    sphinx-build -b html ../docs ./docs/html

如果成功完成，将在
``build/docs/html/index.html`` 创建 HTML 文件，可用任何 Web 浏览器打开。

使用 CMake 和 Make 构建
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在 OpenFAST 目录中，创建一个 ``build`` 目录并进入。
然后，使用以下标志运行 CMake：``-DBUILD_DOCUMENTATION=ON``。CMake 将
配置构建系统以包含构建文档所需的文件。

接下来，运行命令编译文档：

.. code-block:: bash

    make docs

这将首先构建 Doxygen API 文档，然后构建 Sphinx
文档。如果成功完成，将在
``build/docs/html/index.html`` 创建 HTML 文件，可用任何 Web 浏览器打开。

配置和构建文档的完整过程如下：

.. code-block:: bash

    mkdir build
    cd build
    cmake .. -DBUILD_DOCUMENTATION=ON
    make docs

如果对 ``openfast/docs/source`` 中的源文件进行了任何修改，
只需再次执行 ``make`` 命令即可更新 HTML 文件。

下表列出了与文档相关的 make 目标。

======================= ================== ========================================
 目标                    命令               输出位置
======================= ================== ========================================
 完整文档                 make docs          openfast/build/docs/html/index.html
 完整文档                 make sphinx        openfast/build/docs/html/index.html
 Doxygen API 参考         make doxygen
 仅 HTML                  make sphinx-html   openfast/build/docs/html/index.html
 仅 PDF                   make sphinx-pdf    openfast/build/docs/latex/Openfast.pdf
======================= ================== ========================================

添加文档
--------------------

即将推出。想贡献吗？从这里开始！
