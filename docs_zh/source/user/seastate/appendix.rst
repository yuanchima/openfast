.. 本文档由Claude Code自动翻译，原文路径：docs/source/user/seastate/appendix.rst，翻译日期：2026-04-22

.. _ss-primary-input_example:

附录A：SeaState主输入文件示例
===============================================
以下是一个内部生成不规则（JONSWAP）波浪的SeaState主输入文件结构：
::

      ------- SeaState v1.00.* Input File --------------------------------------------
      Example SeaState primary input file
      False            Echo           - 回显输入文件数据（标志）
      ---------------------- 环境条件 --------------------------------
           "default"   WtrDens        - 水密度（kg/m^3）
           "default"   WtrDpth        - 相对于MSL的水深（米）
           "default"   MSL2SWL        - 静水位与平均海平面之间的偏移量（米）[向上为正；WaveMod = 6时未使用；如果PotMod=1或2，必须为零]
      ---------------------- 空间离散化 ---------------------------------------------------
                30.0   X_HalfWidth    – X方向域的半宽（米）[>0，注意：X[nX] = nX*dX，其中nX = {-NX+1,-NX+2,…,NX-1}，dX = X_HalfWidth/(NX-1)]
                30.0   Y_HalfWidth    – Y方向域的半宽（米）[>0，注意：Y[nY] = nY*dY，其中nY = {-NY+1,-NY+2,…,NY-1}，dY = Y_HalfWidth/(NY-1)]
                25.0   Z_Depth        – 相对于SWL的Z方向域深度（米）[0 < Z_Depth <= WtrDpth+MSL2SWL；"default"：Z_Depth = WtrDpth+MSL2SWL；Z[nZ] = ( COS( nZ*dthetaZ ) – 1 )*Z_Depth，其中nZ = {0,1,…NZ-1}，dthetaZ = pi/( 2*(NZ-1) )]
                  10   NX             – X方向半域的节点数（-）[>=2]
                  10   NY             – Y方向半域的节点数（-）[>=2]
                  10   NZ             – Z方向的节点数（-）[>=2]
      ---------------------- 波浪 ---------------------------------------------------
                   2   WaveMod        - 入射波浪运动学模型 {0: 无=静水, 1: 规则（周期性）, 1P#: 用户指定相位的规则波, 2: JONSWAP/Pierson-Moskowitz谱（不规则）, 3: 白噪声谱（不规则）, 4: 来自UserWaveSpctrm例程的用户自定义谱（不规则）, 5: 外部生成的波浪高程时间序列, 6: 外部生成的完整波浪运动学时间序列 [PotMod≠0时选项6无效], 7: 用户定义的波浪频率分量}（开关）
                   1   WaveStMod      - 将入射波浪运动学拉伸到瞬时自由表面的模型 {0: 无=不拉伸, 1: 垂直拉伸, 2: 外推拉伸, 3: Wheeler拉伸}（开关）[WaveMod=0或PotMod≠0时未使用]
                 600   WaveTMax       - 入射波浪计算的分析时间（秒）[WaveMod=0时未使用；决定IFFT中的WaveDOmega=2Pi/WaveTMax]
                 0.2   WaveDT         - 入射波浪计算的时间步长（秒）[WaveMod=0或7时未使用；建议0.1<=WaveDT<=1.0；决定IFFT中的WaveOmegaMax=Pi/WaveDT]
                 2.0   WaveHs         - 入射波浪的有效波高（米）[仅在WaveMod=1、2或3时使用]
                  10   WaveTp         - 入射波浪的谱峰周期（秒）[仅在WaveMod=1或2时使用]
           "DEFAULT"   WavePkShp      - 入射波浪谱的峰形参数（-）或DEFAULT（字符串）[仅在WaveMod=2时使用；对于Pierson-Moskowitz使用1.0]
            0.314159   WvLowCOff      - 波浪谱的低截止频率或下限，超过该值的波浪谱被置零（rad/s）[WaveMod=0、1或6时未使用]
            1.570796   WvHiCOff       - 波浪谱的高截止频率或上限，超过该值的波浪谱被置零（rad/s）[WaveMod=0、1或6时未使用]
                   0   WaveDir        - 入射波浪传播方向（度）[WaveMod=0或6时未使用]
                   0   WaveDirMod     - 方向扩展函数 {0: 无, 1: COS2S}（-）[仅在WaveMod=2、3或4时使用]
                   1   WaveDirSpread  - 波浪方向扩展系数（>0）（-）[仅在WaveMod=2、3或4且WaveDirMod=1时使用]
                   1   WaveNDir       - 波浪方向数量（-）[仅在WaveMod=2、3或4且WaveDirMod=1时使用；仅为奇数]
                   0   WaveDirRange   - 波浪方向范围（全范围：WaveDir +/- 1/2*WaveDirRange）（度）[仅在WaveMod=2、3或4且WaveDirMod=1时使用]
           123456789   WaveSeed(1)    - 入射波浪的第一个随机种子[-2147483648到2147483647]（-）[WaveMod=0、5或6时未使用]
              RANLUX   WaveSeed(2)    - 入射波浪的第二个随机种子[-2147483648到2147483647]，用于内置pRNG，或替代pRNG："RanLux"（-）[WaveMod=0、5或6时未使用]
               FALSE   WaveNDAmp      - 正态分布振幅标志（flag）[仅在WaveMod=2、3或4时使用]
            "unused"   WvKinFile      - 外部生成的波浪数据文件的根名称（带引号的字符串）
      ---------------------- 二阶波浪 ----------------------------------------- [WaveMod=0或6时未使用]
               FALSE   WvDiffQTF      - 完整差频二阶波浪运动学（标志）
               FALSE   WvSumQTF       - 完整和频二阶波浪运动学（标志）
                   0   WvLowCOffD     - 差频中使用的低频率截止（rad/s）[仅与差频方法一起使用]
            1.256637   WvHiCOffD      - 差频中使用的高频率截止（rad/s）[仅与差频方法一起使用]
            0.618319   WvLowCOffS     - 和频中使用的低频率截止（rad/s）[仅与和频方法一起使用]
            3.141593   WvHiCOffS      - 和频中使用的高频率截止（rad/s）[仅与和频方法一起使用]
      ---------------------- 约束波浪 ---------------------------------------
                   0   ConstWaveMod   - 约束波浪模型：0=无；1=具有指定波峰高程α的约束波；2=具有保证峰到谷波高HCrest的约束波（标志）
                   3   CrestHmax      - 波峰高度（ConstWaveMod=1时为2*α，ConstWaveMod=2时为HCrest），必须大于WaveHs（m）[ConstWaveMod=0时未使用]
                  60   CrestTime      - 波峰出现的时间（s）[ConstWaveMod=0时未使用]
                   0   CrestXi        - 波峰的X位置（m）[ConstWaveMod=0时未使用]
                   0   CrestYi        - 波峰的Y位置（m）[ConstWaveMod=0时未使用]
      ---------------------- 水流 ------------------------------------------------- [WaveMod=6时未使用]
                   0   CurrMod        - 水流剖面模型 {0: 无=无水流, 1: 标准, 2: 来自UserCurrent例程的用户自定义}（开关）
                   0   CurrSSV0       - 静水位处的次表层水流速度（m/s）[仅在CurrMod=1时使用]
           "DEFAULT"   CurrSSDir      - 次表层水流方向（度）或DEFAULT（字符串）[仅在CurrMod=1时使用]
                  20   CurrNSRef      - 近表层水流参考深度（米）[仅在CurrMod=1时使用]
                   0   CurrNSV0       - 静水位处的近表层水流速度（m/s）[仅在CurrMod=1时使用]
                   0   CurrNSDir      - 近表层水流方向（度）[仅在CurrMod=1时使用]
                   0   CurrDIV        - 深度无关水流速度（m/s）[仅在CurrMod=1时使用]
                   0   CurrDIDir      - 深度无关水流方向（度）[仅在CurrMod=1时使用]
      ---------------------- MacCamy-Fuchs衍射模型 -------------------------
                   0   MCFD           - MacCamy-Fuchs构件半径（如果半径<=0则忽略）
      ---------------------- 输出 --------------------------------------------------
      False            SeaStSum       - 输出汇总文件（标志）
                   3   OutSwtch       - 输出请求的通道到：[1=SeaState.out, 2=GlueCode.out, 3=两个文件]
      "E15.7e2"        OutFmt         - 数值结果的输出格式（字符串）
      "A15"            OutSFmt        - 标题字符串的输出格式（字符串）
                   2   NWaveElev      - 可计算入射波浪高程的点数（最多9个输出位置）
            0.0, 5.0   WaveElevxi     - 可输出入射波浪高程的点的xi坐标列表（米）[NWaveElev个点，用逗号或空格分隔；如果NWaveElev = 0则未使用]
            0.0, 0.0   WaveElevyi     - 可输出入射波浪高程的点的yi坐标列表（米）[NWaveElev个点，用逗号或空格分隔；如果NWaveElev = 0则未使用]
                   2   NWaveKin       - 可输出波浪运动学的点数（最多9个输出位置）
          0.0,   0.0   WaveKinxi - 可输出波浪运动学的点的xi坐标列表（米）[NWaveKin个点，用逗号或空格分隔；如果NWaveKin = 0则未使用]
          0.0,   5.0   WaveKinyi - 可输出波浪运动学的点的yi坐标列表（米）[NWaveKin个点，用逗号或空格分隔；如果NWaveKin = 0则未使用]
        -14.0, -17.0   WaveKinzi - 可输出波浪运动学的点的zi坐标列表（米）[NWaveKin个点，用逗号或空格分隔；如果NWaveKin = 0则未使用]
      ---------------------- 输出通道 -----------------------------------------
      "Wave1Elev, Wave1Elv1, Wave1Elv2"               - 波浪高程
      "Wave2Elev, Wave2Elv1, Wave2Elv2"
      "FVel1xi, FVel1yi, FVel1zi"  - 位置1处的流体速度
      "FAcc1xi, FAcc1yi, FAcc1zi"  - 位置1处的流体加速度
      "FDynP1"                     - 位置1处的流体动压力
      "FVel2xi, FVel2yi, FVel2zi"  - 位置2处的流体速度
      "FAcc2xi, FAcc2yi, FAcc2zi"  - 位置2处的流体加速度
      "FDynP2"
      END

附录B：SeaState驱动输入文件示例
==============================================
以下是SeaState驱动输入文件结构：
::

      Seastate driver file
      Compatible with SeaState v1.00
      FALSE            Echo               - 回显输入文件数据（标志）
      ---------------------- 环境条件 -------------------------------
      9.80665          Gravity            - 重力加速度（m/s^2）
      1025             WtrDens            - 水密度（kg/m^3）
      200              WtrDpth            - 水深（m）
      0                MSL2SWL            - 静水位与平均海平面之间的偏移量（m）[向上为正]
      ---------------------- SEASTATE -----------------------------------------------
      "./seastate_input.dat" SeaStateInputFile  - SeaState主输入文件名（带引号的字符串）
      "./seastate.SeaSt"     OutRootName        - 所有SeaState生成文件的前缀名（带引号的字符串）
          0                  WrWvKinMod         - 写入波浪运动学？[0: 不写入任何运动学到文件, 1: 仅将(0,0)波浪高程写入文件, 2: 将完整波浪运动学写入文件, 如果WaveMod=6则不写入文件]
       5001                  NSteps             - 仿真中的时间步数（-）
        0.1                  TimeInterval       - 仿真的时间步长（秒）
      ---------------------- 波浪多点高程输出 ----------------------
      False                  WaveElevSeriesFlag - 输出波浪高程场的T/F标志（用于动画）
      END of driver input file

.. _sea-output-channels:

附录C：输出通道列表
===================================
这是SeaState模块所有可能的输出通道列表。名称按含义分组，但可以按您认为合适的顺序在SeaState主输入文件的输出通道部分排列。α表示SeaState主输入文件的输出部分中指定的波浪高程或波浪运动学的输出位置，其中对于波浪高程输出，α是[1,NWaveElev]范围内的数字；对于波浪运动学输出，α是[1,NWaveKin]范围内的数字。设置α > NWaveElev或α > NWaveKin会产生无效输出。所有输出都在全局惯性坐标系中。

================================================================ ========================================================================================================== ==========================================================================================
通道名称                                                         单位                                                                                                      描述
================================================================ ========================================================================================================== ==========================================================================================
**波浪高程**
WaveαElev                                                        (m)                                                                                                        总（一阶加二阶）波浪高程（最多9个指定位置）
WaveαElv1                                                        (m)                                                                                                        一阶波浪高程（最多9个指定位置）
WaveαElv2                                                        (m)                                                                                                        二阶波浪高程（最多9个指定位置）
**波浪和水流运动学**
FVelαxi, FVelαyi, FVelαzi                                        (m/s), (m/s), (m/s)                                                                                        α处的总（一阶加二阶波浪和水流）流体速度
FAccαxi, FAccαyi, FAccαzi                                        (m/s²), (m/s²), (m/s²)                                                                                      α处的总（一阶加二阶波浪）流体加速度
FDynPα                                                           (Pa)                                                                                                       α处的总（一阶加二阶波浪）流体动压力
FAccMCFαxi, FAccMCFαyi, FAccMCFαzi                               (m/s²), (m/s²), (m/s²)                                                                                      用于HydroDyn中MacCamy-Fuchs构件的缩放一阶波浪流体加速度（在α处）
================================================================ ========================================================================================================== ==========================================================================================
