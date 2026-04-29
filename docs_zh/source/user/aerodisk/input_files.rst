
.. _adsk_input-files:

输入和输出文件
==============

单位
-----

AeroDisk 使用国际单位制（kg, m, s, N）。

.. _adsk_input-file:

输入文件
---------

AeroDisk 输入文件定义了致动盘计算所需的常规输入。用户可以修改以下输入以获得所需的行为。

仿真控制
~~~~~~~~

**echo** [开关]

   将输入文件内容写入 <RootName>.ADsk.ech 文件。这对于诊断输入文件报告的错误很有用。

**DT** [秒]

   AeroDisk 使用的积分时间步长，或设置为 _"default"_ 使用耦合代码时间步长。

环境条件
~~~~~~~~

**AirDens** [kg/m^3]

   空气密度，或设置为 _"default"_ 使用耦合代码提供的空气密度。

致动盘特性
~~~~~~~~~~

**RotorRad** [m]

   转子半径，或设置为 _"default"_ 使用耦合代码传递的值。

接下来是致动盘力和力矩的查找表。该表中的数据非常灵活，允许基于单个变量（例如 **TSR**）或最多四个变量进行非常简单的查找。表的最后六列必须包含六个力和力矩系数，这些系数对应于第一组列中给出的一组条件。

**InColNames** [-]

   输入文件中变量列对应的列名列表，用逗号分隔。选项见下文。

**InColDims** [-]

   每个命名变量列的唯一条目数量列表，用逗号分隔。表中的行数必须等于所有给定数字的乘积。必须与 **InColNames** 中给出的条目数量相同。

对于表中的输入变量列，必须至少提供一列，最多可以包含下面列出的五个中的四个（**TSR** 和 **RtSpd** 是互斥的）。

**TSR** [-]

   尖速比，不能与 _RtSpd_ 一起使用。

**RtSpd** [rpm]

   转子转速，不能与 _TSR_ 一起使用。

**VRel** [m/s]

   垂直于转子的相对风速。

**Pitch** [deg]

   集体叶片变桨角。

**Skew** [deg]

   入流偏斜角。如果未提供，偏斜的影响将建模为 :math:`(cos(\chi))^2`。

表的其余六列必须包含力和力矩系数。参见下面的示例表。

示例输入文件
~~~~~~~~~~~~

注意，下面给出的表仅用于说明格式，不代表任何特定的涡轮机。

.. code::

   --- AERO DISK INPUT FILE -------
   Sample actuator disk input file
   --- SIMULATION CONTROL ---------
    FALSE        echo            - Echo input data to "<RootName>.ADsk.ech" (flag)
     "default"   DT              - Integration time step (s)
   --- ENVIRONMENTAL CONDITIONS ---
         1.225   AirDens         - Air density (kg/m^3) (or "default")
   --- ACTUATOR DISK PROPERTIES ---
        63.0     RotorRad        - Rotor radius (m) (or "default")
   "RtSpd,VRel"  InColNames      - Input column headers (string) {may include a combination of "TSR, RtSpd, VRel, Pitch, Skew"} (up to 4 columns) [choose TSR or RtSpd,VRel; if Skew is absent, Skew is modeled as (COS(Skew))^2]
   9,2           InColDims       - Number of unique values in each column (-) (must have same number of columns as InColName) [each >=2]
   RtSpd     VRel      C_Fx   C_Fy   C_Fz   C_Mx   C_My   C_Mz
   (rpm)     (m/s)     (-)    (-)    (-)    (-)    (-)    (-)
     3.0      9.0     0.2347  0.0    0.0   0.0306  0.0    0.0
     4.0      9.0     0.2349  0.0    0.0   0.0314  0.0    0.0
     5.0      9.0     0.2350  0.0    0.0   0.0322  0.0    0.0
     6.0      9.0     0.2351  0.0    0.0   0.0330  0.0    0.0
     7.0      9.0     0.2352  0.0    0.0   0.0338  0.0    0.0
     8.0      9.0     0.2352  0.0    0.0   0.0346  0.0    0.0
     9.0      9.0     0.2351  0.0    0.0   0.0353  0.0    0.0
    10.0      9.0     0.2350  0.0    0.0   0.0361  0.0    0.0
    11.0      9.0     0.2349  0.0    0.0   0.0368  0.0    0.0
     3.0     12.0     0.7837  0.0    0.0   0.0663  0.0    0.0
     4.0     12.0     0.7733  0.0    0.0   0.0663  0.0    0.0
     5.0     12.0     0.7628  0.0    0.0   0.0663  0.0    0.0
     6.0     12.0     0.7520  0.0    0.0   0.0662  0.0    0.0
     7.0     12.0     0.7409  0.0    0.0   0.0660  0.0    0.0
     8.0     12.0     0.7297  0.0    0.0   0.0658  0.0    0.0
     9.0     12.0     0.7182  0.0    0.0   0.0656  0.0    0.0
    10.0     12.0     0.7066  0.0    0.0   0.0653  0.0    0.0
    11.0     12.0     0.6947  0.0    0.0   0.0649  0.0    0.0
   --- OUTPUTS --------------------
                 OutList         - The next line(s) contains a list of output parameters.  See OutListParameters.xlsx for a listing of available output channels, (-)
   END of input file (the word "END" must appear in the first 3 columns of this last OutList line)
   --------------------------------


.. _adsk_outputs:

输出
-----

写入的输出有：
 -  "ADSpeed":    致动盘转速 (rpm)
 -  "ADTSR":      致动盘尖速比 (-)
 -  "ADPitch":    致动盘集体叶片变桨角 (deg)
 -  "ADVWindx, ADVWindy, ADVWindz":    局部坐标系中致动盘平均风速 (m/s)
 -  "ADSTVx, ADSTVy, ADSTVz":          局部坐标系中致动盘结构平移速度 (m/s)
 -  "ADVRel":     致动盘平均相对风速 (m/s)
 -  "ADSkew":     致动盘入流偏斜角 (deg)
 -  "ADCp, ADCt, ADCq":   致动盘功率、推力和力矩系数 (-)
 -  "ADFx, ADFy, ADFz":   局部坐标系中致动盘气动力载荷 (N)
 -  "ADMx, ADMy, ADMz":   局部坐标系中致动盘气动力矩载荷 (N-m)
 -  "ADPower":   致动盘功率 (W)
