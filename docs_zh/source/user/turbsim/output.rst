.. _TurbSim_output:

输出文件
============


HAWC 全场文件
---------------------------------------------------

当 TurbSim 被请求写入 HAWC 格式的输出文件（``WrHAWCFF=TRUE``）时，它将生成四个文件。
``<RootName>-u.bin``、``<RootName>-v.bin`` 和 ``<RootName>-w.bin``
是包含 3 个风速分量全场湍流数据的二进制文件。
``<RootName>.HAWC`` 是一个文本汇总文件，指示二进制文件中的点数以及如何缩放。
该文件中的数据以可复制到 HAWC2 输入文件的格式写入。


注意事项：

1. 汇总文件中的 ``factor_scaling`` 值表示 TurbSim 用于缩放 HAWC 文件中数据的值的倒数。
   ``factor_scaling`` 理论上可在 HAWC2 中用于获取 TurbSim 生成的原始数据。
   TurbSim 缩放数据使得 HAWC2 获得标准差比值为 1.0（u/u）、0.8（v/u）和 0.5（w/u），
   作为 HAWC2 缩放湍流文件方式问题的一种变通方案。请注意，这些比值可能不适用于非 IEC 湍流模型。

2. HAWC 格式文件始终是周期性的，因此所有分析时间步都会被写入。

3. u 分量风速文件已去除平均*轮毂高度*风速，因此它们将包含定义的任何风切变。

4. 对于 HAWC2 仿真，建议 TurbSim 在无风切变（``PLExp=0``）和
   无平均流角（``VFlowAng=0``, ``HFlowAng=0``）的条件下运行。
   这些值可改为在 HAWC2 输入文件或 OpenFAST 的 InflowWind 输入文件中添加。
