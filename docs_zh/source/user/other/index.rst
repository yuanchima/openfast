.. _WaveTank:

WaveTank
========

WaveTank 耦合框架是一种实验性代码，用于将水槽中硬件在环的
MHK 模型与仿真无法在水槽中物理建模的 MHK 水轮机载荷的软件进行耦合。
*OpenFAST* 模块 *SeaState*、*AeroDyn*、*MoorDyn* 和 *InflowWind* 被静态链接到一个
具有 C 语言绑定接口的单一动态库（``cmake`` 目标 ``wavetanktesting_c_binding``）中。
该库可以从 *LabView* 或其他代码中调用。

库的输入包括时间以及位于每个时间步单一参考点处的运动（包括速度和
加速度）。计算得到的力和力矩返回给调用代码。

限制
~~~~~~~~~~~~
当前设置的 WaveTank 库有以下限制：

- 刚性结构，包括平台、塔筒和机舱
- 无偏航 DOF
- 刚性转子
- 整个仿真过程中转子 RPM 恒定
- 目前没有控制器接口选项
- 可视化限于 *AeroDyn* 和 *SeaState*
- 当前实现仅支持浮式 MHK 水轮机（``MHK = 2``）。其他模式存在但未完全实现。




输入文件
~~~~~~~~~~


.. toctree::
   :maxdepth: 2

   wavetank_input.rst
