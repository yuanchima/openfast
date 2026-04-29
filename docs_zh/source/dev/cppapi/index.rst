.. _cppapi:

OpenFAST C++ 应用程序编程接口
==============================================

OpenFAST 提供 C++ 应用程序编程接口（API），用于从 C++ 外部程序驱动风力机仿真。C++ API 主要开发用于将 OpenFAST 与计算流体力学（CFD）求解器集成以进行流固耦合（FSI）应用。目前支持使用致动线方法进行 FSI 仿真，计划在不久的将来支持叶片解析 FSI 仿真。C++ API 也可用于创建外部驱动程序或耦合框架代码，以并行运行多个风力机的 OpenFAST 仿真。

C++ API 在 :class:`~fast::OpenFAST` 类中定义和实现。任何想在 C++ 中为 OpenFAST 编写耦合框架代码的用户都应实例化 OpenFAST 类的对象，并使用它来驱动风力机仿真。提供了一个示例耦合框架代码 `FAST_Prog.cpp <https://github.com/OpenFAST/openfast/blob/dev/glue-codes/openfast-cpp/src/FAST_Prog.cpp>`_ 作为 C++ API 使用的演示。该耦合框架代码允许在单个或多个处理器上串行或并行地使用 OpenFAST 仿真多台风力机。使用消息传递接口（MPI）并行运行不同风力机实例。FAST_Prog.cpp 的简化版本如下所示。高亮行指示了 OpenFAST 类的使用。

.. literalinclude:: files/FAST_Prog.cpp
   :emphasize-lines: 1,27,28,32,36,38,40,45,49
   :language: C++

OpenFAST 类的所有输入通过 :class:`fast::fastInputs` 的对象期望传入。


FIXME: 需要 **doxygenclass** 来渲染 :class:`fast::fastInputs` 类结构

..
 .. doxygenclass:: fast::fastInputs
   :members:
   :private-members:
   :protected-members:
   :undoc-members:

:class:`~fast::fastInputs` 类的对象期望包含一个类型为 :class:`~fast::turbineDataType` 的结构体向量，其大小为仿真中
风力机的数量。

FIXME: 需要 **doxygenstruct** 来渲染 :class:`fast::turbineDataType` 类结构

..
 .. doxygenstruct:: fast::turbineDataType
   :members:
   :private-members:


使用 C++ API 进行致动线仿真
--------------------------------------------

C++ API 主要开发用于将 OpenFAST 与计算流体力学（CFD）求解器集成以进行流固耦合（FSI）应用。当今风能应用的主力 FSI 算法是致动线算法 :cite:`cpp-churchfield2012`。致动线算法将风力机对流场的影响表示为沿气动表面一系列点力（**致动点**）。OpenFAST 中使用的叶素动量理论被修改，以将 OpenFAST 与 CFD 求解器接口以进行致动线仿真。CFD 求解器成为 OpenFAST 的来流模块，提供风力机附近的速度信息。在 OpenFAST 中关闭诱导因子的计算，AeroDyn 仅使用查找表和可选的动态失速模型，根据从 CFD 求解器接收的来流场信息计算风力机上的载荷。应通过在 AeroDyn 输入文件中选择 :samp:`WakeMod=0` 在 OpenFAST 中关闭诱导模型。OpenFAST 将沿叶片和塔筒的线力集中为一系列用于致动线算法的点力。:numref:`actuatorline-viz` 说明了 OpenFAST 与 CFD 求解器之间用于致动线应用的信息传递。

.. _actuatorline-viz:

.. figure:: files/actuatorLine_illustrationViz.png
   :align: center
   :width: 100%

   用于致动线应用的 CFD 求解器与 OpenFAST 之间通过 C++ API 传递速度、载荷和挠度的示意图。

CFD 求解器预期作为耦合到 OpenFAST 的致动线 FSI 仿真的*驱动程序*。C++ API 允许*子步进*（substepping），即驱动程序时间步长是 OpenFAST 时间步长的整数倍（:math:`\Delta_t^{CFD} = n \Delta_t^{OpenFAST}`）。OpenFAST C++ API 的当前实现允许流体（CFD）和结构（OpenFAST）求解器之间的串行交错 FSI 方案。:numref:`actuatorline-css` 展示了用于致动线应用的松耦合串行交错 FSI 方案的推荐实现，以将仿真从时间步 `n` 推进到 `n+1`。可以通过"外层"迭代重复 :numref:`actuatorline-css` 中的耦合算法来构建强耦合 FSI 方案。

.. _actuatorline-css:

.. figure:: files/css_actuatorline.png
   :align: center
   :width: 100%

   一种可通过 C++ API 为致动线应用构建的传统串行交错 FSI 方案。


OpenFAST 为各种模块使用不同的空间网格 :cite:`cpp-fastv8ModFramework`。我们定义致动点位于风力机结构模型（ElastoDyn/BeamDyn）中定义的网格上。用户通过每台风力机的输入参数 :samp:`numForcePtsBlade` 和 :samp:`numForcePtsTower` 定义每个叶片和塔筒上所需的致动点数量。所有叶片上的致动点数量必须相同。C++ API 使用 OpenFAST 通过对结构模型中的节点进行线性插值来创建所请求数量的致动点。OpenFAST 中的网格映射算法 :cite:`cpp-fastv8AlgorithmsExamples` 用于将结构模型的挠度和 AeroDyn 的载荷传递到致动点。为了区分*致动点*和 Aerodyn 点，OpenFAST C++ 使用术语 :samp:`forceNodes` 表示致动点，:samp:`velNodes`（速度节点）表示 Aerodyn 点。以下代码片段说明了如何使用 C++ API 为致动线应用实现带"外层"迭代的强耦合 FSI 方案。此示例代码片段设置在 :samp:`velNodes` 处的速度，并访问 :samp:`forceNodes` 处的坐标和集中力。

.. code-block:: c++

   std::vector<double> currentCoords(3);
   std::vector<double> sampleVel(3);

   for (int iOuter=0; iOuter < nOuterIterations; iOuter++) {

      FAST.predict_states(); //Predict the location and force at the actuator points at time step 'n+1'.

      for(iTurb=0; iTurb < nTurbines; iTurb++) {
         for(int i=0; i < FAST.get_numVelPts(iTurb); i++) {
            // Get actuator node co-ordinates at time step 'n+1'
            FAST.getForceNodeCoordinates(currentCoords, i, iTurb, fast::np1);
            //Move the actuator point to this co-ordinate if necessary
            // Get force at actuator node at time step 'n+1'
            FAST.getForce(actForce, i, iTurb, fast::np1);
            //Do something with this force
         }
      }

      // Predict CFD solver to next time step here

      for(iTurb=0; iTurb < nTurbines; iTurb++) {
         for(int i=0; i < FAST.get_numVelPts(iTurb); i++) {
            // Get velocity node co-ordinates at time step 'n+1'
            FAST.getVelNodeCoordinates(currentCoords, i, iTurb, fast::np1);
            //Sample velocity from CFD solver at currentCoords into sampleVel here
            // Set velocity at the velocity nodes at time step 'n+1'
            FAST.setVelocity(sampleVel, i, iTurb, fast::np1);
         }
      }

      FAST.update_states_driver_time_step(); // Predict the state of OpenFAST at the next time step

   }

   // Move OpenFAST to next CFD time step
   FAST.advance_to_next_driver_time_step();

.. toctree::
   :maxdepth: 1

   api.rst


实现
--------------

C++ API 使用 C-Fortran 接口在内部调用与 Fortran 驱动程序相同的函数来推进 OpenFAST 的时间。FAST_Library.f90 包含所有可以从 C++ API 调用的函数。C++ API 与 Fortran 模块之间的一些对应函数显示在下表中。

.. table::

   +------------------------------------+---------------------------------+-------------------------------+
   | C++ API - OpenFAST.cpp             | Fortran - FAST_Library.f90      | FAST_Subs.f90                 |
   +====================================+=================================+===============================+
   | init()                             | FAST_AL_CFD_Init                | FAST_InitializeAll_T          |
   +------------------------------------+---------------------------------+-------------------------------+
   | solution0()                        | FAST_CFD_Solution0              | FAST_Solution0_T              |
   +------------------------------------+---------------------------------+-------------------------------+
   | prework()                          | FAST_CFD_Prework                | FAST_Prework_T                |
   +------------------------------------+---------------------------------+-------------------------------+
   |                                    | FAST_CFD_Store_SS               | FAST_Store_SS                 |
   +------------------------------------+---------------------------------+-------------------------------+
   | update_states_driver_time_step()   | FAST_CFD_UpdateStates           | FAST_UpdateStates_T           |
   +------------------------------------+---------------------------------+-------------------------------+
   |                                    | FAST_CFD_Reset_SS               | FAST_Reset_SS                 |
   +------------------------------------+---------------------------------+-------------------------------+
   | advance_to_next_driver_time_step() | FAST_CFD_AdvanceToNextTimeStep  | FAST_AdvanceToNextTimeStep_T  |
   +------------------------------------+---------------------------------+-------------------------------+

`FAST_Subs.f90` 中的 `FAST_Solution_T` 子程序被拆分为三个不同的子程序 `FAST_Prework_T`、`FAST_UpdateStates_T` 和 `FAST_AdvanceToNextTimeStep_T`，以允许外部驱动程序进行多次*外层*迭代。引入了额外的子程序 `FAST_Store_SS` 和 `FAST_Reset_SS`，以便在使用外部驱动程序的*子步进*时将 OpenFAST 回退超过 1 个时间步。当从外部驱动程序使用 C++ API 时，Fortran 子程序的典型访问顺序如下所示。

.. code-block:: fortran

   call FAST_AL_CFD_Init

   call FAST_CFD_Solution0

   do i=1, nTimesteps

      if (nSubsteps .gt. 1)
            call FAST_CFD_Store_SS
      else
            call FAST_CFD_Prework
      end if

      do iOuter=1, nOuterIterations

         if (nSubsteps .gt. 1)

            if (iOuter .ne. 1) then
               ! Reset OpenFAST back when not the first pass
               call FAST_CFD_Reset_SS

            end if

            do j=1, nSubsteps

               ! Set external inputs into modules here for the substep
               call FAST_CFD_Prework
               call FAST_CFD_UpdateStates
               call FAST_CFD_AdvanceToNextTimeStep

            end do !Substeps

         else

            call FAST_CFD_UpdateStates

         end if

      end do !Outer iterations

      if (nSubsteps .gt. 1) then

         ! Nothing to do here

      else

         call FAST_CFD_AdvanceToNextTimeStep

      end if

   end do



OpenFAST 中的 :class:`ExternalInflow` 模块执行载荷和挠度到致动点的映射。C++ API 支持使用 BeamDyn 和 ElastoDyn 来建模叶片。当使用 BeamDyn 建模叶片时，C++ API 要求每个叶片仅使用 1 个有限单元，并结合梯形积分用于致动线仿真。

映射过程测试
--------------------------

映射过程实现的测试如下。使用 C++ API 运行 OpenFAST 以仿真 Jonkman 5-MW（以前称为 NREL 5-MW）风力机一个时间步，在所有速度节点施加 :math:`8 m/s` 的规定速度，无诱导（:samp:`WakeMod=0`）。致动点数量从 10 变化到 100，而速度节点数量固定为 17。:numref:`actuator-force-nodes-mapping-test-thrust` 和 :numref:`actuator-force-nodes-mapping-test-torque` 显示，当致动点数量在 :math:`10-100` 之间变化时，推力和转矩分别变化小于 :math:`1.1 \times 10^{-6}\%` 和 :math:`2 \times 10^{-6}\%`。


.. _actuator-force-nodes-mapping-test-thrust:

.. figure:: files/thrustXActuatorForcePoints.png
   :align: center
   :width: 100%

   在相同数量速度节点下，使用不同数量的致动点在 `OpenFAST` 中推力的变化。

.. _actuator-force-nodes-mapping-test-torque:

.. figure:: files/torqueXActuatorForcePoints.png
   :align: center
   :width: 100%

   在相同数量速度节点下，使用不同数量的致动点在 `OpenFAST` 中转矩的变化。



参考文献
----------

.. bibliography:: bibliography.bib
   :labelprefix: cpp-
