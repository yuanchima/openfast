
.. _ep_input-files:

输入文件
========

本节描述 ExtPtfm 使用的不同输入文件。ExtPtfm 使用国际单位制（kg, m, s, N）。
不得在输入文件中添加或删除任何行。

.. _ep_main-input-file:

ExtPftm 模块输入文件
---------------------

在 OpenFAST 2.3 之前，ExtPtfm 模块没有"模块"输入文件，直接提供 Guyan ASCII 输入文件。现在不再支持这种方式，必须使用模块输入文件。
`OpenFAST 与 ExtPtfm 集成的示例可在此处获取 <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/5MW_OC4Jckt_ExtPtfm/>`_。
`ExtPtfm 模块输入文件的示例可在此处获取 <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/5MW_OC4Jckt_ExtPtfm/ExtPtfm.dat>`_。
格式与其他 *OpenFAST* 模块类似。
输入参数如下：

- ``DT``：数值积分的时间步长（秒）。用户可以在此处指定时间步长，或使用"default"以依赖粘合代码的时间步长。

- ``IntMethod``：时间积分的数值方法。可用的方法有 Runge-Kutta、Adams–Bashforth 和 Adams–Bashforth-Moutlon。

  ``RBMod``：处理浮式结构刚体运动的方法（开关）。
  0：不对刚体运动进行特殊处理（固定式基础结构）；
  1：转换到刚体参考系；
  2：转换到刚体参考系并添加虚拟力和精确自重。

- ``Red_Filename``：包含 Guyan/Craig-Bampton 输入的文件路径。

- ``ActiveDOFList``：大小为 ``NActiveDOFList`` 的列表，包含活动的 CB 模态索引。如果 ``NActiveDOFList<=0``，则不读取此列表。指定后，所有系统矩阵将重构为 :math:`\boldsymbol{M}_\text{new}=\boldsymbol{M}(I,I)`，其中 :math:`I` 是索引列表，可以是未排序和不连续的。设置 ``NActiveDOFList=0`` 等效于 Guyan 降阶。设置 :math:`\texttt{NActiveCBDOF}=-1` 将使用输入文件中提供的所有 CB 自由度。

- ``InitPosList``：大小为 ``NInitPosList`` 的列表，包含 CB 模态的初始位置。如果 ``NInitPosList<=0``，则不读取此列表，此时所有 CB 自由度位置初始化为 0。

- ``InitVelList``：大小为 ``NInitVelList`` 的列表，包含 CB 模态的初始速度。如果 ``NInitVelList<=0``，则不读取此列表，此时所有 CB 自由度速度初始化为 0。

  ``Connections``：包含结构上连接点的标志。如果为 true，则必须通过 ``Conn_FileName`` 提供指定连接的文件。目前，此功能仅用于与其中一个系泊模块耦合。

  ``UserForcing``：用户定义模态激励的标志。如果为 true，则必须通过 ``Force_FileName`` 提供包含要施加的力时间序列的文件。

  ``ConnForcing``：在连接点处施加用户定义外力的标志。如果为 true，则必须通过 ``FConn_FileName`` 提供包含要在连接点处施加的力时间序列的文件。此选项要求 ``Connections`` 为 true。不支持施加力矩。

- ``SumPrint``：将摘要数据打印到 <RootName>.sum 文件中。

- ``OutFile``、``TabDelim``、``OutFmt``、``TStart``：输出标志，目前未使用。

- ``OutList``：指定用户请求的输出列表。这些输出在 :ref:`epOutputChannels` 中描述。

.. _epOutputChannels:

输出通道
--------

输出通过 *OpenFAST* 导出的 '.out' 或 '.outb' 文件写入磁盘。
界面处的载荷和位移时间序列可以在 ElastoDyn 中选择（例如 ``PtfmPitch``）。
*ExtPtfm* 中还实现了额外的"写入输出"，如下列表所示。理论部分（:ref:`ep-theory`）中使用的符号也在表中给出。

.. table:: *ExtPtfm* 模块的输出通道

   ================ ======================================================================== ==================================== =========
   **通道名称**     **描述**                                                                 **符号**                             **单位**
   ================ ======================================================================== ==================================== =========
   ``IntrfFx``      平台界面力 - 沿 x 方向                                                   :math:`f_{C}[1]`                     (N)
   ``IntrfFy``      平台界面力 - 沿 y 方向                                                   :math:`f_{C}[2]`                     (N)
   ``IntrfFz``      平台界面力 - 沿 z 方向                                                   :math:`f_{C}[3]`                     (N)
   ``IntrfMx``      平台界面力矩 - 沿 x 方向                                                 :math:`f_{C}[4]`                     (Nm)
   ``IntrfMy``      平台界面力矩 - 沿 y 方向                                                 :math:`f_{C}[5]`                     (Nm)
   ``IntrfMz``      平台界面力矩 - 沿 z 方向                                                 :math:`f_{C}[6]`                     (Nm)
   ``ExtrnFx``      界面点处的降阶输入力 - 沿 x 方向                                         :math:`f_{r1}[1]`                    (N)
   ``ExtrnFy``      界面点处的降阶输入力 - 沿 y 方向                                         :math:`f_{r1}[2]`                    (N)
   ``ExtrnFz``      界面点处的降阶输入力 - 沿 z 方向                                         :math:`f_{r1}[3]`                    (N)
   ``ExtrnMx``      界面点处的降阶输入力矩 - 沿 x 方向                                       :math:`f_{r1}[4]`                    (Nm)
   ``ExtrnMy``      界面点处的降阶输入力矩 - 沿 y 方向                                       :math:`f_{r1}[5]`                    (Nm)
   ``ExtrnMz``      界面点处的降阶输入力矩 - 沿 z 方向                                       :math:`f_{r1}[6]`                    (Nm)
   ``CBDXXX``       CB 自由度 XXX 的位移（例如 ``CBD001``）                                  :math:`\boldsymbol{x}_2[XXX]`        (-)
   ``CBVXXX``       CB 自由度 XXX 的速度（例如 ``CBV001``）                                  :math:`\boldsymbol{\dot{x}}_2[XXX]`  (-)
   ``CBAXXX``       CB 自由度 XXX 的加速度（例如 ``CBA001``）                                :math:`\boldsymbol{\ddot{x}}_2[XXX]` (-)
   ``CBFXXX``       CB 自由度 XXX 中的降阶输入模态力（例如 ``CBF001``）                       :math:`\boldsymbol{f}_{r2}[XXX]`     (-)
   ================ ======================================================================== ==================================== =========

.. _epSuperelementInputFile:

Guyan/Craig-Bampton 超单元输入文件（通过 ``Red_Filename`` 提供）
----------------------------------------------------------------

此超单元输入文件用于提供系统矩阵。

示例
^^^^

超单元文件的示例可在此处获取 <https://github.com/OpenFAST/r-test/blob/main/glue-codes/openfast/5MW_OC4Jckt_ExtPtfm/ExtPtfm_SE.dat>`_。
下面给出一个"虚拟"示例，其中 ``n``、``M(i,j)``、``K(i,j)``、``C(i,j)`` 等表示数值。

.. code::

   !Comment
   !Comment
   !Dimension: n

   !Mass Matrix (Units (kg,m))
     M(1,1) ... M(1,n)
           [...]
     M(n,1) ... M(n,n)

   !Stiffness Matrix (Units (N,m))
     K(1,1) ... K(1,n)
           [...]
     K(n,1) ... K(n,n)

   !Damping Matrix (Units (N,m,kg))
     C(1,1) ... C(1,n)
           [...]
     C(n,1) ... C(n,n)

   !Weight constant (Units (N,m))
     W_0(1) W_0(2) ... W_0(n)

   !Weight stiffness (Units (N,m))
     K_W(1,1) ... K_W(1,n)
             [...]
     K_W(n,1) ... K_W(n,n)

规范
^^^^

文件遵循以下规范：

- ASCII 文件

- 文件可以以任意数量的注释行开头，注释行以感叹号 '\ ``!``\ ' 开头

- 接下来必须提供以下（不区分大小写）关键字：

  - '\ ``!dimension``: ' 后跟整数 ``n``\ :math:`=6+n_{CB}`

- 其余行包含以下特殊（不区分大小写）关键字：

  - '\ ``!mass matrix``\ '：后跟一些文本。接下来的 :math:`n` 行每行包含 :math:`n` 个浮点值。这些值对应于 :math:`\boldsymbol{M}_r`。请注意，当使用 ``RBMod`` > 0 建模浮式结构时，前 6 个模态（界面模态）也用作刚体模态。因此，:math:`\boldsymbol{M}_r` 的前 6×6 个条目必须是自洽的刚体质量矩阵。如果不是这种情况，ExtPtfm 将发出警告。在内部，ExtPtfm 使用此信息来确定质量、刚体转动惯量和重心位置。

  - '\ ``!stiffness matrix``\ '：与质量矩阵类似，值对应于 :math:`\boldsymbol{K}_r`。对于浮式结构，不应有与刚体运动相关的结构刚度；因此，:math:`\boldsymbol{K}_r` 的前 6 行和 6 列应全为零。

  - '\ ``!weight constant``\ '：后跟一些文本。下一行应包含 :math:`n` 个浮点值。这些值对应于恒定自重 :math:`\boldsymbol{W}_0`。对于浮式结构，:math:`\boldsymbol{W}_0` 的第 1、2 和 6 个条目必须为零。自重引起的恒定横滚和俯仰力矩必须与从质量矩阵导出的重心位置一致。

  - '\ ``!weight stiffness``\ '：与质量矩阵类似，值对应于 :math:`\boldsymbol{K}_W`。对于浮式结构，采用的约定要求 :math:`\boldsymbol{W}_0` - :math:`\boldsymbol{K}_W` * (模态位移) 返回跟随刚体/界面运动的参考系中的自重。这意味着 :math:`\boldsymbol{K}_W` 的条目 (1,5) 必须等于 :math:`\boldsymbol{W}_0(3)`。条目 (2,4) 必须等于 :math:`-\boldsymbol{W}_0(3)`。条目 (4,4) 和 (5,5) 必须等于 :math:`\boldsymbol{W}_0(3) * z_{CG}`。条目 (6,4) 必须等于 :math:`-\boldsymbol{W}_0(3) * x_{CG}`，条目 (6,5) 必须等于 :math:`-\boldsymbol{W}_0(3) * y_{CG}`，其中 :math:`(x_{CG},y_{CG},z_{CG})` 是刚体质心相对于 ElastoDyn 中定义的平台参考点的位置，即 (*PtfmRefxt*, *PtfmRefyt*, *PtfmRefzt*)。:math:`\boldsymbol{K}_W` 的第 1、2 和 6 列中的所有条目应为零。同样，如果输入矩阵不遵循此结构，将发出警告。

.. _epConnectInputFile:

连接输入文件（通过 ``Conn_Filename`` 提供）
-------------------------------------------

连接输入文件用于提供结构上连接点的数量和位置。目前，当启用任何可用的系泊模块时，连接点仅用于与系泊导缆孔耦合。此文件还需要提供每个 Guyan 模态和 Craig-Bampton 模态下每个连接点的结构运动/挠度。

示例
^^^^

下面给出一个"虚拟"示例，其中 ``m``、``X1``、``Y1``、``Z1`` 等表示数值。

.. code::

   !Comment
   !Comment
   !nConn: m

   !Connections (m)
     X1  Y1  Z1
        [...]
     Xm  Ym  Zm

   !Displacement (m)
     [3m x n matrix]

规范
^^^^

文件遵循以下规范：

- ASCII 文件

- 文件可以以任意数量的注释行开头，注释行以感叹号 '\ ``!``\ ' 开头

- 接下来必须按给定顺序提供以下（不区分大小写）关键字：

  - '\ ``!nConn``: ' 后跟整数 ``m``，给出结构上的连接点数量。

  - '\ ``!Connections``\ '：后跟一些文本。接下来的 :math:`m` 行每行包含 3 个浮点值，给出 :math:`m` 个连接点的 :math:`x`、:math:`y` 和 :math:`z` 坐标。请注意，坐标参考于 HydroDyn 和 MoorDyn 输入文件中使用的同一全局原点，而不是相对于 ElastoDyn 中定义的平台参考点。

  - '\ ``!Displacement``\ '：后跟一些文本。接下来的 :math:`3m` 行每行包含 :math:`n` 个浮点值。此矩阵的列对应于超单元输入文件中定义的 :math:`n` 个模态，前 3 行给出第一个连接点的 :math:`x`、:math:`y` 和 :math:`z` 位移，接下来的 3 行给出第二个连接点的位移，依此类推，对应于每个模态的单位运动。当使用 ``RBMod`` > 0 建模浮体时，前 6 个模态是 Guyan 界面模态，也用作刚体模态。因此，此矩阵的前 6 列应简单描述每个连接点跟随 ElastoDyn 中定义的平台参考点（即 (*PtfmRefxt*, *PtfmRefyt*, *PtfmRefzt*)）的线性化刚体运动。

用户模态和连接力文件（通过 ``Force_Filename`` 和 ``FConn_Filename`` 提供）
---------------------------------------------------------------------------

这两个文件允许用户向系统模态和连接点施加外部激励。这两个文件的格式相似。唯一的区别在于列数。

示例
^^^^

下面给出一个"虚拟"示例，其中 ``k``、``t1``、``F1`` 等表示数值。

.. code::

   !Comment
   !Comment
   !nSteps: k

   !Forcing (N/-)
     t(1)  F_1(1)  F_2(1) ...
           [...]
     t(k)  F_1(k)  F_2(k) ...

规范
^^^^

文件遵循以下规范：

- ASCII 文件

- 文件可以以任意数量的注释行开头，注释行以感叹号 '\ ``!``\ ' 开头

- 接下来必须按给定顺序提供以下（不区分大小写）关键字：

  - '\ ``!NSteps``: ' 后跟整数 ``k``，给出力时间序列中的时间步数量。

  - '\ ``!Forcing``\ '：后跟一些文本。接下来的 :math:`k` 行提供用户定义的模态或连接力时间序列。列数取决于这是模态激励还是连接激励。在两种情况下，第一列都是时间。对于模态激励，时间列后面应有 :math:`n` 列，每列对应于超单元输入文件中定义的一个模态。对于连接激励，时间列后面应有 :math:`3m` 列，给出第一个连接点的 :math:`x`、:math:`y` 和 :math:`z` 力分量，然后是第二个连接点的，依此类推。请注意，第一列中的时间不必与仿真时间步匹配，也不必是均匀间隔的。ExtPtfm 将根据需要在提供的时间之间进行线性插值。
