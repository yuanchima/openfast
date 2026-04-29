
.. _hd-primary-input_example:

附录A：OC4半潜式平台输入文件
===========================================
以下是OC4半潜式结构的HydroDyn主输入文件：

::

      ------- HydroDyn Input File ----------------------------------------------------
      historic NREL 5.0 MW offshore baseline floating platform HydroDyn input properties for the OC4 Semi-submersible.
      False            Echo           - 回显输入文件数据（标志）
      ---------------------- FLOATING PLATFORM --------------------------------------- [WaveMod=6时未使用]
                   1   PotMod         - 势流模型 {0: 无势流, 1: 基于WAMIT输出的频域到时域转换, 2: 流体冲量理论（FIT）}（开关）
                   1   ExctnMod       - 波浪激励模型 {0: 不计算波浪激励, 1: 离散傅里叶变换, 2: 状态空间}（开关）[仅在PotMod=1时使用；状态空间需要*.ssexctn输入文件；如果PtfmYMod=1，则需要ExctnMod=0或1]
                   0   ExctnDisp      - 波浪激励计算方法 {0: 使用未位移位置, 1: 使用位移位置, 2: 使用低通滤波后的位移位置} [仅在PotMod=1、ExctnMod>0且SeaState的WaveMod>0时使用]（开关）
                  10   ExctnCutOff    - 低通滤波位移位置的截止（拐角）频率（Hz）[>0.0] [仅在PotMod=1、ExctnMod>0且ExctnDisp=2时使用]
                   0   PtfmYMod       - 大平台偏航偏移模型 {0: 基于PtfmRefY的静态参考偏航偏移, 1: 基于对PRP偏航运动进行低通滤波的动态参考偏航偏移，截止频率为PtfmYCutOff}（开关）
                   0   PtfmRefY       - 平台参考偏航偏移，如果PtfmYMod=0则为恒定值，如果PtfmYMod=1则为初始值（度）
                0.01   PtfmYCutOff    - PtfmYMod=1时PRP偏航运动低通滤波的截止频率[>0.0；PtfmYMod=0时未使用]（Hz）
                  36   NExctnHdg      - [-180, 180)度范围内均匀分布的平台偏航/航向角数量，将为这些角度计算波浪激励[>=2；PtfmYMod=0时未使用]（-）
                   1   RdtnMod        - 波浪辐射记忆效应模型 {0: 不计算记忆效应, 1: 卷积, 2: 状态空间}（开关）[仅在PotMod=1时使用；状态空间需要*.ss输入文件]
                  60   RdtnTMax       - 波浪辐射核计算的分析时间（秒）[仅在PotMod=1且RdtnMod>0时使用；决定余弦变换中的RdtnDOmega=Pi/RdtnTMax；请确保对于给定平台，这个时间足够长，使得辐射冲量响应函数衰减到接近零]
              0.0125   RdtnDT         - 波浪辐射核计算的时间步长（秒）[仅在PotMod=1且ExctnMod>0或RdtnMod>0时使用；建议DT<=RdtnDT<=0.1；决定余弦变换中的RdtnOmegaMax=Pi/RdtnDT]
                   1   NBody          - 要使用的WAMIT体数量（-）[>=1；仅在PotMod=1时使用。如果NBodyMod=1，则WAMIT数据包含大小为6*NBody x 1的向量和大小为6*NBody x 6*NBody的矩阵；如果NBodyMod>1，则有NBody组WAMIT数据，每组包含大小为6 x 1的向量和大小为6 x 6的矩阵]
                   1   NBodyMod       - 体耦合模型 {1: 包含每个体之间的耦合项，HydroDyn中的NBody等于WAMIT中的NBODY, 2: 忽略每个体之间的耦合项，NBODY=1且WAMIT中XBODY=0, 3: 忽略每个体之间的耦合项，NBODY=1且WAMIT中XBODY≠0}（开关）[仅在PotMod=1时使用]
        "marin_semi"   PotFile        - 势流模型数据的根名称；WAMIT输出文件包含线性无量纲静水恢复矩阵（.hst）、频率相关的水动力附加质量矩阵和阻尼矩阵（.1）、以及单位波幅下与频率和方向相关的波浪激励力向量（.3）（带引号的字符串）[如果NBodyMod>1，则有1到NBody个文件；请确保这些WAMIT文件中固有的频率覆盖了给定平台物理上重要的频率范围；它们必须包含零频率和无穷频率极限]
                   1   WAMITULEN      - 用于将WAMIT输出重新无量纲化的特征体长度尺度（米）[如果NBodyMod>1，则有1到NBody个值][仅在PotMod=1时使用]
                   0   PtfmRefxt      - 体参考点相对于(0,0,0)的xt偏移（米）[1到NBody个值][仅在PotMod=1时使用]
                   0   PtfmRefyt      - 体参考点相对于(0,0,0)的yt偏移（米）[1到NBody个值][仅在PotMod=1时使用]
                   0   PtfmRefzt      - 体参考点相对于(0,0,0)的zt偏移（米）[1到NBody个值][仅在PotMod=1时使用。如果NBodyMod=2，则PtfmRefzt=0.0]
                   0   PtfmRefztRot   - 体参考框架相对于xt/yt绕zt的旋转角度（度）[1到NBody个值][仅在PotMod=1时使用]
               13917   PtfmVol0       - 体在未位移位置时的排开水体积（m^3）[1到NBody个值][仅在PotMod=1时使用；请使用与WAMIT在.out文件中输出的相同值]
                   0   PtfmCOBxt      - 浮心（COB）相对于(0,0)的xt偏移（米）[1到NBody个值][仅在PotMod=1时使用]
                   0   PtfmCOByt      - 浮心（COB）相对于(0,0)的yt偏移（米）[1到NBody个值][仅在PotMod=1时使用]
      ---------------------- 2ND-ORDER FLOATING PLATFORM FORCES ---------------------- [WaveMod=0或6，或PotMod=0或2时未使用]
                   0   MnDrift        - 计算的平均漂移二阶力 {0: 无；[7, 8, 9, 10, 11, 或 12]: 要使用的WAMIT文件} [MnDrift、NewmanApp和DiffQTF中只能有一个非零。如果NBody>1，则MnDrift≠8]
                   0   NewmanApp      - 使用Newman近似计算的平均和慢漂移二阶力 {0: 无；[7, 8, 9, 10, 11, 或 12]: 要使用的WAMIT文件} [MnDrift、NewmanApp和DiffQTF中只能有一个非零。如果NBody>1，则NewmanApp≠8。仅在WaveDirMod=0时使用]
                   0   DiffQTF        - 使用完整QTF计算的完全差频二阶力 {0: 无；[10, 11, 或 12]: 要使用的WAMIT文件} [MnDrift、NewmanApp和DiffQTF中只能有一个非零。如果PtfmYMod=1，则需要DiffQTF=0]
                   0   SumQTF         - 使用完整QTF计算的完全和频二阶力 {0: 无；[10, 11, 或 12]: 要使用的WAMIT文件} [如果PtfmYMod=1，则需要SumQTF=0]
      ---------------------- PLATFORM ADDITIONAL STIFFNESS AND DAMPING  -------------- [PotMod=0或2时未使用]
                   0   AddF0    - 附加预载荷（N, N·m）[如果NBodyMod=1，则为一个大小为6*NBody x 1的向量；如果NBodyMod>1，则为NBody个大小为6 x 1的向量]
                   0
                   0
                   0
                   0
                   0
                   0             0             0             0             0             0   AddCLin  - 附加线性刚度（N/m, N/rad, N·m/m, N·m/rad）[如果NBodyMod=1，则为一个大小为6*NBody x 6*NBody的矩阵；如果NBodyMod>1，则为NBody个大小为6 x 6的矩阵]
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0   AddBLin  - 附加线性阻尼（N/(m/s), N/(rad/s), N·m/(m/s), N·m/(rad/s)）[如果NBodyMod=1，则为一个大小为6*NBody x 6*NBody的矩阵；如果NBodyMod>1，则为NBody个大小为6 x 6的矩阵]
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0   AddBQuad - 附加二次阻尼（N/(m/s)^2, N/(rad/s)^2, N·m/(m/s)^2, N·m/(rad/s)^2）[如果NBodyMod=1，则为一个大小为6*NBody x 6*NBody的矩阵；如果NBodyMod>1，则为NBody个大小为6 x 6的矩阵]
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
                   0             0             0             0             0             0
      ---------------------- STRIP THEORY OPTIONS --------------------------------------
                   0   WaveDisp       - 波浪运动学计算方法 {0: 使用未位移位置, 1: 使用位移位置}（开关）[如果PtfmYMod=1，则需要WaveDisp=1]
                   0   AMMod          - 分布附加质量力的计算方法（0: 仅且始终在未位移位置静水位以下的节点上计算，2: 计算到瞬时自由表面）[当WaveMod=0或6，或SeaState中的WaveStMod=0时覆盖为0]
      ---------------------- AXIAL COEFFICIENTS --------------------------------------
                   2   NAxCoef        - 轴向系数数量（-）
      AxCoefID  AxCd     AxCa     AxCp    AxFDMod   AxVnCOff  AxFDLoFSc
         (-)    (-)      (-)      (-)      (-)       (-)        (-)
          1     0.00     0.00     1.00      0        0.00       1.00
          2     9.60     0.00     1.00      0        0.00       1.00
      ---------------------- MEMBER JOINTS -------------------------------------------
                  44   NJoints        - 节点数量（-）[必须恰好为0或至少为2]
      JointID   Jointxi     Jointyi     Jointzi  JointAxID   JointOvrlp   [JointOvrlp= 0: 节点处不做处理, 1: 通过计算超级构件消除重叠]
         (-)     (m)         (m)         (m)        (-)       (switch)
          1     0.00000     0.00000   -20.00000      1            0
          2     0.00000     0.00000    10.00000      1            0
          3    14.43376    25.00000   -14.00000      1            0
          4    14.43376    25.00000    12.00000      1            0
          5   -28.86751     0.00000   -14.00000      1            0
          6   -28.86751     0.00000    12.00000      1            0
          7    14.43376   -25.00000   -14.00000      1            0
          8    14.43376   -25.00000    12.00000      1            0
          9    14.43375    25.00000   -20.00000      2            0
         10   -28.86750     0.00000   -20.00000      2            0
         11    14.43375   -25.00000   -20.00000      2            0
         12     9.23760    22.00000    10.00000      1            0
         13   -23.67130     3.00000    10.00000      1            0
         14   -23.67130    -3.00000    10.00000      1            0
         15     9.23760   -22.00000    10.00000      1            0
         16    14.43375   -19.00000    10.00000      1            0
         17    14.43375    19.00000    10.00000      1            0
         18     4.04145    19.00000   -17.00000      1            0
         19   -18.47520     6.00000   -17.00000      1            0
         20   -18.47520    -6.00000   -17.00000      1            0
         21     4.04145   -19.00000   -17.00000      1            0
         22    14.43375   -13.00000   -17.00000      1            0
         23    14.43375    13.00000   -17.00000      1            0
         24     1.62500     2.81500    10.00000      1            0
         25    11.43376    19.80385    10.00000      1            0
         26    -3.25000     0.00000    10.00000      1            0
         27   -22.87000     0.00000    10.00000      1            0
         28     1.62500    -2.81500    10.00000      1            0
         29    11.43376   -19.80385    10.00000      1            0
         30     1.62500     2.81500   -17.00000      1            0
         31     8.43376    14.60770   -17.00000      1            0
         32    -3.25000     0.00000   -17.00000      1            0
         33   -16.87000     0.00000   -17.00000      1            0
         34     1.62500    -2.81500   -17.00000      1            0
         35     8.43376   -14.60770   -17.00000      1            0
         36     1.62500     2.81500   -16.20000      1            0
         37    11.43376    19.80385     9.13000      1            0
         38    -3.25000     0.00000   -16.20000      1            0
         39   -22.87000     0.00000     9.13000      1            0
         40     1.62500    -2.81500   -16.20000      1            0
         41    11.43376   -19.80385     9.13000      1            0
         42    14.43376    25.00000   -19.94000      1            0
         43   -28.86751     0.00000   -19.94000      1            0
         44    14.43376   -25.00000   -19.94000      1            0
      ---------------- CYLINDRICAL MEMBER CROSS-SECTION PROPERTIES -------------------
                   4   NPropSetsCyl    - 圆柱形构件属性集数量（-）
      PropSetID    PropD         PropThck
         (-)        (m)            (m)
          1        6.50000        0.03000          ! 主立柱
          2       12.00000        0.06000          ! 上立柱
          3       24.00000        0.06000          ! 基立柱
          4        1.60000        0.01750          ! 浮筒
      ---------------- RECTANGULAR MEMBER CROSS-SECTION PROPERTIES -------------------
                   0   NPropSetsRec    - 矩形构件属性集数量（-）
      MPropSetID   PropA      PropB    PropThck
         (-)        (m)        (m)       (m)
      -------- SIMPLE CYLINDRICAL-MEMBER HYDRODYNAMIC COEFFICIENTS (model 1) ---------
      SimplCd    SimplCdMG    SimplCa    SimplCaMG    SimplCp    SimplCpMG   SimplAxCd  SimplAxCdMG   SimplAxCa  SimplAxCaMG  SimplAxCp   SimplAxCpMG    SimplCb    SimplCbMG
         (-)         (-)         (-)         (-)         (-)         (-)         (-)         (-)         (-)         (-)         (-)         (-)            (-)         (-)
         0.00        0.00        0.00        0.00        1.00        1.00        0.00        0.00        0.00        0.00        1.00        1.00           1.00        1.00
      -------- SIMPLE RECTANGULAR-MEMBER HYDRODYNAMIC COEFFICIENTS (model 1) ---------
      SimplCdA    SimplCdAMG    SimplCdB    SimplCdBMG    SimplCaA    SimplCaAMG    SimplCaB    SimplCaBMG    SimplCp    SimplCpMG   SimplAxCd  SimplAxCdMG   SimplAxCa  SimplAxCaMG  SimplAxCp   SimplAxCpMG  SimplCb  SimplCbMG
        (-)         (-)           (-)         (-)           (-)         (-)           (-)         (-)           (-)         (-)         (-)         (-)          (-)         (-)         (-)         (-)          (-)       (-)
        0.0         0.0           0.0         0.0           0.0         0.0           0.0         0.0           0.0         0.0         0.0         0.0          0.0         0.0         0.0         0.0          1.0       1.0
      ------ DEPTH-BASED CYLINDRICAL-MEMBER HYDRODYNAMIC COEFFICIENTS (model 2) -------
                   0   NCoefDpthCyl    - 基于深度的圆柱形构件系数数量（-）
      Dpth      DpthCd   DpthCdMG   DpthCa   DpthCaMG       DpthCp   DpthCpMG   DpthAxCd   DpthAxCdMG   DpthAxCa   DpthAxCaMG   DpthAxCp   DpthAxCpMG   DpthCb   DpthCbMG
      (m)       (-)      (-)        (-)      (-)            (-)      (-)        (-)        (-)          (-)        (-)          (-)        (-)           (-)      (-)
      ------ DEPTH-BASED RECTANGULAR-MEMBER HYDRODYNAMIC COEFFICIENTS (model 2) -------
                   0   NCoefDpthRec    - 基于深度的矩形构件系数数量（-）
      Dpth    DpthCdA   DpthCdAMG    DpthCdB   DpthCdBMG   DpthCaA   DpthCaAMG   DpthCaB   DpthCaBMG     DpthCp   DpthCpMG   DpthAxCd   DpthAxCdMG   DpthAxCa   DpthAxCaMG   DpthAxCp   DpthAxCpMG   DpthCb   DpthCbMG
      (m)       (-)       (-)          (-)       (-)         (-)       (-)        (-)        (-)          (-)        (-)       (-)        (-)          (-)        (-)          (-)        (-)          (-)      (-)
      ------ MEMBER-BASED CYLINDRICAL-MEMBER HYDRODYNAMIC COEFFICIENTS (model 3) ------
                  25   NCoefMembersCyl    - 基于构件的圆柱形构件系数数量（-）
      MemberID    MemberCd1     MemberCd2    MemberCdMG1   MemberCdMG2    MemberCa1     MemberCa2    MemberCaMG1   MemberCaMG2    MemberCp1     MemberCp2    MemberCpMG1   MemberCpMG2   MemberAxCd1   MemberAxCd2  MemberAxCdMG1 MemberAxCdMG2  MemberAxCa1   MemberAxCa2  MemberAxCaMG1 MemberAxCaMG2  MemberAxCp1  MemberAxCp2   MemberAxCpMG1   MemberAxCpMG2    MemberCb1     MemberCb2    MemberCbMG1   MemberCbMG2
         (-)         (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)           (-)              (-)           (-)           (-)           (-)
          1          0.56          0.56          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 主立柱
          2          0.61          0.61          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 上立柱 1
          3          0.61          0.61          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 上立柱 2
          4          0.61          0.61          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 上立柱 3
          5          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱 1
          6          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱 2
          7          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱 3
         23          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱端盖 1
         24          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱端盖 2
         25          0.68          0.68          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 基立柱端盖 3
          8          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，上部 1
          9          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，上部 2
         10          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，上部 3
         11          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，下部 1
         12          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，下部 2
         13          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 三角浮筒，下部 3
         14          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，上部 1
         15          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，上部 2
         16          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，上部 3
         17          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，下部 1
         18          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，下部 2
         19          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! Y型浮筒，下部 3
         20          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 交叉支撑 1
         21          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 交叉支撑 2
         22          0.63          0.63          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00          0.00             1.00          1.00          1.00          1.00          ! 交叉支撑 3
      ------ MEMBER-BASED RECTANGULAR-MEMBER HYDRODYNAMIC COEFFICIENTS (model 3) ------
                  0   NCoefMembersRec - 基于构件的矩形构件系数数量（-）
      MemberID    MemberCdA1     MemberCdA2    MemberCdAMG1   MemberCdAMG2    MemberCdB1     MemberCdB2    MemberCdBMG1   MemberCdBMG2    MemberCaA1     MemberCaA2    MemberCaAMG1   MemberCaAMG2    MemberCaB1     MemberCaB2    MemberCaBMG1   MemberCaBMG2    MemberCp1     MemberCp2    MemberCpMG1   MemberCpMG2   MemberAxCd1   MemberAxCd2  MemberAxCdMG1 MemberAxCdMG2  MemberAxCa1   MemberAxCa2  MemberAxCaMG1 MemberAxCaMG2  MemberAxCp1  MemberAxCp2   MemberAxCpMG1   MemberAxCpMG2   MemberCb1     MemberCb2    MemberCbMG1   MemberCbMG2
         (-)         (-)            (-)           (-)            (-)             (-)            (-)           (-)            (-)             (-)            (-)           (-)            (-)             (-)            (-)           (-)            (-)             (-)           (-)          (-)           (-)           (-)           (-)          (-)           (-)            (-)           (-)          (-)           (-)              (-)           (-)          (-)           (-)            (-)           (-)          (-)           (-)
      ---------------------- MEMBERS -------------------------------------------------
                  25   NMembers       - 构件数量（-）
      MemberID  MJointID1  MJointID2  MPropSetID1  MPropSetID2  MSecGeom    MSpinOrient   MDivSize   MCoefMod  MHstLMod  PropPot   [MCoefMod=1: 使用简单系数表, 2: 使用基于深度的系数表, 3: 使用基于构件的系数表] [如果构件采用势流理论建模，则PropPot≠0]
        (-)        (-)        (-)         (-)          (-)      (switch)       (deg)        (m)      (switch)  (switch)  (flag)
         1          1          2           1            1           1            0         1.0000        3         1      TRUE           ! 主立柱
         2          3          4           2            2           1            0         1.0000        3         1      TRUE           ! 上立柱 1
         3          5          6           2            2           1            0         1.0000        3         1      TRUE           ! 上立柱 2
         4          7          8           2            2           1            0         1.0000        3         1      TRUE           ! 上立柱 3
         5         42          3           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱 1
         6         43          5           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱 2
         7         44          7           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱 3
        23          9         42           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱端盖 1
        24         10         43           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱端盖 2
        25         11         44           3            3           1            0         1.0000        3         1      TRUE           ! 基立柱端盖 3
         8         12         13           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，上部 1
         9         14         15           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，上部 2
        10         16         17           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，上部 3
        11         18         19           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，下部 1
        12         20         21           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，下部 2
        13         22         23           4            4           1            0         1.0000        3         1      TRUE           ! 三角浮筒，下部 3
        14         24         25           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，上部 1
        15         26         27           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，上部 2
        16         28         29           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，上部 3
        17         30         31           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，下部 1
        18         32         33           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，下部 2
        19         34         35           4            4           1            0         1.0000        3         1      TRUE           ! Y型浮筒，下部 3
        20         36         37           4            4           1            0         1.0000        3         1      TRUE           ! 交叉支撑 1
        21         38         39           4            4           1            0         1.0000        3         1      TRUE           ! 交叉支撑 2
        22         40         41           4            4           1            0         1.0000        3         1      TRUE           ! 交叉支撑 3
      ---------------------- FILLED MEMBERS ------------------------------------------
                   2   NFillGroups     - 填充构件组数量（-）[如果FillDens=DEFAULT，则FillDens=WtrDens；FillFSLoc与MSL2SWL相关]
      FillNumM FillMList FillFSLoc     FillDens
      (-)      (-)       (m)           (kg/m^3)
       3   2   3   4    -6.17           1025
       3   5   6   7   -14.89           1025
      ---------------------- MARINE GROWTH -------------------------------------------
                   0   NMGDepths      - 指定的海洋生物生长深度数量（-）
      MGDpth     MGThck       MGDens
      (m)        (m)         (kg/m^3)
      ---------------------- MEMBER OUTPUT LIST --------------------------------------
                   0   NMOutputs      - 构件输出数量（-）[必须小于10]
      MemberID   NOutLoc    NodeLocs [NOutLoc < 10; 节点位置是距构件起点的归一化距离，必须>=0且<=1] [NMOutputs=0时未使用]
        (-)        (-)        (-)
      ---------------------- JOINT OUTPUT LIST ---------------------------------------
                   0   NJOutputs      - 节点输出数量 [必须小于10]
                       JOutLst        - 要输出的JointID列表（-）[NJOutputs=0时未使用]
      ---------------------- OUTPUT --------------------------------------------------
      True             HDSum          - 输出汇总文件（标志）
      False            OutAll         - 输出所有用户指定的构件和节点载荷（仅在每个构件端部，不包括内部位置）（标志）
                   2   OutSwtch       - 输出请求的通道到：[1:Hydrodyn.out, 2:GlueCode.out, 3:两个文件]
      "E16.8e2"        OutFmt         - 数值结果的输出格式（带引号的字符串）[不检查有效性]
      "A11"            OutSFmt        - 头字符串的输出格式（带引号的字符串）[不检查有效性]
      ---------------------- OUTPUT CHANNELS -----------------------------------------
      HydroFxi
      HydroFyi
      HydroFzi
      HydroMxi
      HydroMyi
      HydroMzi
      END of output channels and end of file.（"END"必须出现在该行的前3列）

附录B：OC4半潜式平台驱动输入文件
===========================================
以下是OC4半潜式结构的HydroDyn驱动输入文件：

::

      ------- HydroDyn Driver Input File --------------------------------------------
      HydroDyn Driver file for OC4 Semi-submersible.
            FALSE   Echo                - 回显输入文件数据（标志）
      ---------------------- ENVIRONMENTAL CONDITIONS -------------------------------
          9.80665   Gravity             - 重力加速度（m/s^2）
             1025   WtrDens             - 水密度（kg/m^3）
              200   WtrDpth             - 水深（m）
                0   MSL2SWL             - 静水位与平均海平面之间的偏移（m）[向上为正]
      ---------------------- HYDRODYN -----------------------------------------------
      "./OC4Semi.dat"    HDInputFile       - HydroDyn主输入文件名（带引号的字符串）
      "./SeaState.dat"   SeaStateInputFile - SeaState主输入文件名（带引号的字符串）
      "./OC4Semi"        OutRootName       - 所有HydroDyn生成文件的前缀名称（带引号的字符串）
            FALSE        Linearize         - 启用线性化的标志
             4801        NSteps            - 仿真中的时间步数量（-）[总时长60秒]
           0.0125        TimeInterval      - 仿真的时间步长（秒）
      ---------------------- PRP INPUTS (Platform Reference Point) ------------------
                0   PRPInputsMod      - PRP（平台参考点）输入模型 {0: 每个时间步的所有输入都为零, 1: 稳态输入, 2: 从文件读取输入（InputsFile）}（开关）
                0   PtfmRefzt         - 地面到平台参考点的垂直距离（m）
      "not_used"    PRPInputsFile     - PRP输入文件名，InputsMod=2时使用（带引号的字符串）
      ---------------------- PRP STEADY STATE INPUTS  -------------------------------
                0,          0,          0,          0,          0,          0    uPRPInSteady         - PRP稳态位移（3个）和旋转（3个），在平台参考点处（m, m, m, rad, rad, rad）
                0,          0,          0,          0,          0,          0    uDotPRPInSteady      - PRP稳态平移速度（3个）和旋转速度（3个），在平台参考点处（m/s, rad/s）
                0,          0,          0,          0,          0,          0    uDotDotPRPInSteady   - PRP稳态平移加速度（3个）和旋转加速度（3个），在平台参考点处（m/s^2, rad/s^2）

.. _hd-output-channels:

附录C：输出通道列表
===================================
这是HydroDyn模块所有可能的输出参数列表。名称按含义分组，但可以按照您的需要在HydroDyn输入文件的"OUTPUT CHANNELS"部分中任意排序。MαNβ指输出构件α的输出节点β，其中α是[1,9]范围内的数字，对应"MEMBER OUTPUT LIST"表中的第α行，β是[1,9]范围内的数字，对应该表项中**NodeLocs**列表的第β个位置。Jα指输出节点α，其中α是[1,9]范围内的数字，对应"JOINT OUTPUT LIST"表中的第α行。Bα指体α，其中α是[1,9]范围内的数字。如果α > NBody，则输出无效；如果NBody > 9，则只能输出前9个体。Waveα指可以输出波浪高程的点α，其中α是[1,9]范围内的数字。如果α > NWaveElev，则输出无效。所有输出都在全局惯性框架坐标下。

================================================================ ========================================================================================================== ========================================================================================
通道名称                                                         单位                                                                                                      描述
================================================================ ========================================================================================================== ========================================================================================
**波浪和水流运动学**
MαNβVxi, MαNβVyi, MαNβVzi                                        (m/s), (m/s), (m/s)                                                                                        MαNβ处的总（一阶加二阶）流体粒子速度
MαNβAxi, MαNβAyi, MαNβAzi                                        (m/s^2), (m/s^2), (m/s^2)                                                                                  MαNβ处的总（一阶加二阶）流体粒子加速度
MαNβDynP                                                         (Pa)                                                                                                       MαNβ处的总（一阶加二阶）流体粒子动压力
JαVxi, JαVyi, JαVzi                                              (m/s), (m/s), (m/s)                                                                                        Jα处的总（一阶加二阶）流体粒子速度
JαAxi, JαAyi, JαAzi                                              (m/s^2), (m/s^2), (m/s^2)                                                                                  Jα处的总（一阶加二阶）流体粒子加速度
JαDynP                                                           (Pa)                                                                                                       Jα处的总（一阶加二阶）流体粒子动压力
**总载荷和附加载荷**
BαAddFxi, BαAddFyi, BαAddFzi, BαAddMxi, BαAddMyi, BαAddMzi       (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处由附加预载荷、刚度和阻尼产生的载荷
HydroFxi, HydroFyi, HydroFzi, HydroMxi, HydroMyi, HydroMzi       (N), (N), (N), (N·m), (N·m), (N·m)                                                                         (0,0,0)处势流和切片理论的总集成水动力载荷
**势流解产生的载荷**
BαWvsFxi, BαWvsFyi, BαWvsFzi, BαWvsMxi, BαWvsMyi, BαWvsMzi       (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处衍射产生的总（一阶加二阶）波浪激励载荷
BαWvsF1xi, BαWvsF1yi, BαWvsF1zi, BαWvsM1xi, BαWvsM1yi, BαWvsM1zi (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处衍射产生的一阶波浪激励载荷
BαWvsF2xi, BαWvsF2yi, BαWvsF2zi, BαWvsM2xi, BαWvsM2yi, BαWvsM2zi (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处衍射产生的二阶波浪激励载荷
BαHdSFxi, BαHdSFyi, BαHdSFzi, BαHdSMxi, BαHdSMyi, BαHdSMzi       (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处的静水载荷
BαRdtFxi, BαRdtFyi, BαRdtFzi, BαRdtMxi, BαRdtMyi, BαRdtMzi       (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Bα处的波浪辐射载荷
**结构运动**
PRPSurge, PRPSway, PRPHeave, PRPRoll, PRPPitch, PRPYaw           (m), (m), (m), (rad), (rad), (rad)                                                                         平台参考点（PRP）处的位移和旋转
PRPTVxi, PRPTVyi, PRPTVzi, PRPRVxi, PRPRVyi, PRPRVzi             (m/s), (m/s), (m/s), (rad/s), (rad/s), (rad/s)                                                             PRP的平移速度和旋转速度
PRPTAxi, PRPTAyi, PRPTAzi, PRPRAxi, PRPRAyi, PRPRAzi             (m/s^2), (m/s^2), (m/s^2), (rad/s^2), (rad/s^2), (rad/s^2)                                                 PRP的平移加速度和旋转加速度
BαSurge, BαSway, BαHeave, BαRoll, BαPitch BαYaw                  (m), (m), (m), (rad), (rad), (rad)                                                                         Bα处的位移和旋转
BαTVxi, BαTVyi, BαTVzi, BαRVxi, BαRVyi, BαRVzi                   (m/s), (m/s), (m/s), (rad/s), (rad/s), (rad/s)                                                             Bα处的平移速度和旋转速度
BαTAxi, BαTAyi, BαTAzi, BαRAxi, BαRAyi, BαRAzi                   (m/s^2), (m/s^2), (m/s^2), (rad/s^2), (rad/s^2), (rad/s^2)                                                 Bα处的平移加速度和旋转加速度
MαNβSTVxi, MαNβSTVyi, MαNβSTVzi                                  (m/s), (m/s), (m/s)                                                                                        MαNβ处的结构平移速度
MαNβSTAxi, MαNβSTAyi, MαNβSTAzi                                  (m/s^2), (m/s^2), (m/s^2)                                                                                  MαNβ处的结构平移加速度
JαSTVxi, JαSTVyi, JαSTVzi                                        (m/s), (m/s), (m/s)                                                                                        Jα处的结构平移速度
JαSTAxi, JαSTAyi, JαSTAzi                                        (m/s^2), (m/s^2), (m/s^2)                                                                                  Jα处的结构平移加速度
**构件上的分布载荷（单位长度）**
MαNβFDxi, MαNβFDyi, MαNβFDzi                                     (N/m), (N/m), (N/m)                                                                                        MαNβ处的粘性阻力
MαNβFIxi, MαNβFIyi, MαNβFIzi                                     (N/m), (N/m), (N/m)                                                                                        MαNβ处的流体惯性力
MαNβFBxi, MαNβFByi, MαNβFBzi, MαNβMBxi, MαNβMByi, MαNβMBzi       (N/m), (N/m), (N/m), (N·m/m), (N·m/m), (N·m/m)                                                             MαNβ处的浮力载荷
MαNβFBFxi, MαNβFBFyi, MαNβFBFzi, MαNβMBFxi, MαNβMBFyi, MαNβMBFzi (N/m), (N/m), (N/m), (N·m/m), (N·m/m), (N·m/m)                                                             MαNβ处由注水/压载产生的负浮力载荷
MαNβFMGxi, MαNβFMGyi, MαNβFMGzi, MαNβMMGxi, MαNβMMGyi, MαNβMMGzi (N/m), (N/m), (N/m), (N·m/m), (N·m/m), (N·m/m)                                                             MαNβ处由海洋生物生长重量产生的载荷
MαNβFAMxi, MαNβFAMyi, MαNβFAMzi                                  (N/m), (N/m), (N/m)                                                                                        MαNβ处的水动力附加质量力
MαNβFAGxi, MαNβFAGyi, MαNβFAGzi, MαNβMAGxi, MαNβMAGyi, MαNβMAGzi (N/m), (N/m), (N/m), (N·m/m), (N·m/m), (N·m/m)                                                             MαNβ处的海洋生物生长质量惯性载荷
MαNβFAFxi, MαNβFAFyi, MαNβFAFzi, MαNβMAFxi, MαNβMAFyi, MαNβMAFzi (N/m), (N/m), (N/m), (N·m/m), (N·m/m), (N·m/m)                                                             MαNβ处的注水/压载质量惯性载荷
**节点处的集中载荷**
JαFDxi, JαFDyi, JαFDzi                                           (N), (N), (N)                                                                                              Jα处的粘性阻力
JαFIxi, JαFIyi, JαFIzi                                           (N), (N), (N)                                                                                              Jα处的流体惯性力
JαFBxi, JαFByi, JαFBzi, JαMBxi, JαMByi, JαMBzi                   (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Jα处的浮力载荷
JαFBFxi, JαFBFyi, JαFBFzi, JαMBFxi, JαMBFyi, JαMBFzi             (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Jα处由注水/压载产生的负浮力载荷
JαFMGxi, JαFMGyi, JαFMGzi                                        (N), (N), (N)                                                                                              Jα处由海洋生物生长重量产生的力
JαFAMxi, JαFAMyi, JαFAMzi                                        (N), (N), (N)                                                                                              Jα处的水动力附加质量力
JαFAGxi, JαFAGyi, JαFAGzi, JαMAGxi, JαMAGyi, JαMAGzi             (N), (N), (N), (N·m), (N·m), (N·m)                                                                         Jα处的海洋生物生长质量惯性载荷
================================================================ ========================================================================================================== ========================================================================================
