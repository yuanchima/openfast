.. _types_files:

类型文件与 OpenFAST Registry
=====================================
作为一个现代软件项目，OpenFAST 有一个复杂的自定义数据
类型系统。在 Fortran 中，这些被称为"派生数据类型"（derived data types）。每个模块
包含一组独特的派生类型，可以扩展但必须
符合 OpenFAST 框架。模块类型通常
由包含的程序 *OpenFAST Registry* 自动生成。OpenFAST
Registry 是用 C 编写的，改编自
`WRF <http://www.mmm.ucar.edu/wrf/WG2/software_2.0/registry_schaffer.pdf>`__ 中使用的类似工具。
更多信息请访问 `OpenFAST Registry README <https://github.com/OpenFAST/openfast/tree/main/modules/openfast-registry>`__。

OpenFAST Registry 需要一个输入文件来描述给定模块
所需的类型。通常，所有模块使用类似的命名约定：
**<Module>_Registry.txt**，生成的 Fortran 代码将在一个
名为 **<Module>_Types.f90** 的文件中。例如，AeroDyn OpenFAST Registry 输入
文件位于 ``openfast/modules/aerodyn/src/AeroDyn_Registry.txt``，
生成的自动生成 Fortran 源代码位于
``openfast/modules/aerodyn/src/AeroDyn_Types.f90``。

由于类型模块是自动生成的，任何对数据类型的
直接更改都应在 OpenFAST Registry 输入文件中表达，以便
更改不会被后续覆盖。

编译 OpenFAST Registry
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
OpenFAST Registry 通过 CMake 包含在
OpenFAST 构建系统中。但是，默认情况下
**不** 编译 OpenFAST Registry 可执行文件，而是在编译 OpenFAST 时使用
*git* 中包含的类型模块。要将
OpenFAST Registry 包含在构建过程中并编译 Registry 程序，
请使用 ``GENERATE_TYPES`` 标志配置 CMake：

.. code-block:: bash

    cmake .. -DGENERATE_TYPES=ON

启用 ``GENERATE_TYPES`` 后，CMake 将配置 `openfast-registry`
目标作为所有其他目标的依赖项进行编译。OpenFAST Registry
可执行文件将位于
``openfast/build/modules/openfast-registry/openfast-registry``。

.. _regenerating_types:

重新生成类型模块
~~~~~~~~~~~~~~~~~~~~~~~~~~~
启用 ``GENERATE_TYPES`` 标志后，将向配置为
可使用 OpenFAST Registry 的模块添加一个附加步骤。该
附加步骤将执行 OpenFAST Registry 并重新生成类型
模块，覆盖现有模块。对类型模块的任何更改将在
*git* 中体现出来。对于 Registry 输入文件
未更改的模块，生成的类型模块不会改变。但是，对于已修改的 Registry
输入文件，输出的类型模块将被
重新编译。

添加新的类型模块
~~~~~~~~~~~~~~~~~~~~~~~~~
添加新的类型模块的过程与 :ref:`regenerating_types`
密切相关。此处需要额外的步骤来配置 CMake 以
对新的输入文件执行 Registry 并将生成的类型模块
包含在编译步骤中。

首先，必须创建一个新的 OpenFAST Registry 输入文件。然后，必须
配置它以在相应模块的
``CMakeLists.txt`` 中通过 Registry 处理：

.. code-block:: cmake

    # 这是允许 Registry 执行的控制语句
    if (GENERATE_TYPES)

        # 这是执行 Registry 的 CMake 包装函数
        # 语法：generate_f90_types(<Registry 输入文件> <输出文件位置>)
        generate_f90_types(src/AeroDyn_Registry.txt ${CMAKE_CURRENT_LIST_DIR}/src/AeroDyn_Types.f90)
        generate_f90_types(src/New_Registry.txt ${CMAKE_CURRENT_LIST_DIR}/src/New_Types.f90)
    endif()

最后，生成的类型模块必须添加到
相应模块的源文件中：

.. code-block:: cmake

    # AeroDyn lib
    set(AD_LIBS_SOURCES
        src/AeroDyn.f90
        src/AeroDyn_IO.f90
        src/AirfoilInfo.f90
        src/BEMT.f90
        src/DBEMT.f90
        src/BEMTUncoupled.f90
        src/UnsteadyAero.f90
        src/fmin_fcn.f90
        src/mod_root1dim.f90
        src/AeroDyn_Types.f90
        src/AirfoilInfo_Types.f90
        src/BEMT_Types.f90
        src/DBEMT_Types.f90
        src/UnsteadyAero_Types.f90

        # 在此处添加新的类型模块
        src/New_Types.f90
    )

CMake 正确配置后，构建过程中将显示一条消息，
指示 OpenFAST Registry 正在执行：

.. code-block:: bash

    [ 64%] Generating ../../../modules/aerodyn/src/New_Types.f90

    ----- FAST Registry --------------
    ----------------------------------------------------------
    input file: /Users/rmudafor/Development/openfast/modules/aerodyn/src/New_Registry.txt
    # 后续会有更多构建过程输出

最后，应显示生成的类型模块正在
编译：

.. code-block:: bash

    Scanning dependencies of target aerodynlib
    [ 70%] Building Fortran object modules/aerodyn/CMakeFiles/aerodynlib.dir/src/New_Types.f90.o
