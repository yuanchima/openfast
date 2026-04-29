
C++ API用户指南
===================

.. only:: html

   本文档提供了C++ API和粘合代码的快速参考指南。它旨在供普通用户与其他OpenFAST手册结合使用。本手册将随着新版本的发布以及根据需要提供软件改进或修改的进一步信息而更新。

C++ API提供了一个高级API，通过C++粘合代码运行OpenFAST。C++ API的主要目的是帮助将OpenFAST与通常用C++编写的外部程序（如CFD求解器）对接。通过在CMake中开启:cmakeval:`BUILD_OPENFAST_CPP_API`标志来启用C++ API的安装。

提供了一个示例粘合代码`FAST_Prog.cpp <https://github.com/OpenFAST/openfast/blob/dev/glue-codes/openfast-cpp/src/FAST_Prog.cpp>`_，作为C++ API使用方法的演示。该粘合代码允许在多个处理器上并行使用OpenFAST模拟多台风力机。粘合代码接受一个名为``cDriver.i``的输入文件（:download:`下载 <files/cDriver.i>`）。

.. literalinclude:: files/cDriver.i
   :language: yaml

命令行调用
-----------------------

.. code-block:: bash

   mpiexec -np <N> openfastcpp

常用输入文件选项
-------------------------

.. confval:: n_turbines_glob

   模拟中的总风力机数量。输入文件必须包含与`nTurbinesGlob`一致的风力机特定部分（`Turbine0`、`Turbine1`、...、`Turbine(n-1)`）。

.. confval:: debug

   如果设置为true，则启用调试输出

.. confval:: dry_run

   如果dryRun设置为true，模拟将不会运行。但是，模拟将读取输入文件，为处理器分配风力机，并准备运行各个风力机实例。这个标志有助于在运行前测试模拟的设置。

.. confval:: sim_start

   指示模拟是从头开始还是重启的标志。``sim_start``有三种可能的值：

   * ``init`` - 当从`t=0s`开始模拟时使用此选项。
   * ``trueRestart`` - 虽然OpenFAST允许重启风力机模拟，但像Bladed风格控制器这样的外部组件可能不支持。当知道模拟的所有组件都支持重启时使用此选项。
   * ``restartDriverInitFAST`` - 当选择``restartDriverInitFAST``选项时，各个风力机模型从`t=0s`开始，并使用存储在hdf5文件中的执行器节点处的入流数据运行到指定的重启时间。C++ API在每个OpenFAST时间步将执行器节点处的入流数据存储在hdf5文件中，然后在使用此重启选项时读取回来。当粘合代码是CFD求解器时，这个重启选项特别有用。

.. confval:: coupling_mode

   耦合模式的选择。``coupling_mode``有两种可能的值：``strong``或``classic``。``strong``耦合模式在每个驱动时间步使用2次外部迭代，而``classic``耦合模式调用`step()`函数使用松耦合模式。

.. confval:: t_start

   模拟的开始时间

.. confval:: t_end

   模拟的结束时间。t_end <= t_max

.. confval:: t_max

   模拟的最大时间

.. confval:: dt_fast

   FAST的时间步长。所有风力机应该使用相同的时间步长。

.. confval:: n_substeps

   驱动程序每个时间步内OpenFAST的子时间步数量。

.. confval:: n_checkpoint

   每隔这么多时间步写入一次重启文件

.. confval:: set_exp_law_wind

   布尔值，True/False。当为true时，使用幂律风廓线设置Aerodyn节点处的速度，指数为0.2，参考风速为90米高度处的10 m/s。这个选项有助于在运行大规模CFD模拟之前，在独立模式下测试执行器线模拟的设置。

风力机特定输入选项
------------------------------

.. confval:: turbine_base_pos

   执行器线模拟中风力机基座的位置

.. confval:: num_force_pts_blade

   执行器线模拟中每个叶片上的执行器点数量

.. confval:: num_force_pts_tower

   执行器线模拟中塔筒上的执行器点数量。

.. confval:: restart_filename

   重启模拟时此风力机的检查点文件

.. confval:: FAST_input_filename

   此风力机的FAST输入文件

.. confval:: turb_id

   每台风力机的唯一ID
