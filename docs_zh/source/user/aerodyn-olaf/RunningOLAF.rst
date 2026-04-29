使用OLAF
=================


.. _Running-OLAF:

运行OLAF
~~~~~~~~~~~~

由于OLAF是OpenFAST的一个模块，下载、编译和运行OLAF的过程与OpenFAST相同。相关说明可在:ref:`installation`文档中找到。

.. note::
   为了提高FVW模块的速度，用户可能希望使用`OpenMP`进行编译。为此，在CMake中添加`-DOPENMP=ON`选项。


.. _Guidelines-OLAF:

参数设置指南
~~~~~~~~~~

OLAF的大多数选项可以保留默认值。计算结果将取决于时间离散化、尾迹长度和正则化参数。我们在本节中提供这些参数的设置指南，以及一个简单的Python代码来计算这些参数。请定期查看本节，因为我们可能会随着时间的推移进一步完善指南。

我们在本节末尾提供了一个Python脚本，用于程序化地设置主要参数。


**时间步长**
我们建议将OLAF的时间步长（**DTfvw**）设置为对应转子旋转:math:`\Delta \psi = 6`度：

.. math::

    \Delta t_\text{FVW}
    = \frac{\Delta \psi_\text{rad}}{\Omega_\text{rad/s}}
    = \frac{\Delta \psi_\text{deg}}{6 \times \Omega_\text{RPM}}

如果结构求解器需要更小的时间步长，胶合代码的时间步长可以设置为与**DTfvw**不同的值，只要**DTfvw**是胶合代码时间步长的整数倍即可。



**尾迹长度和面板数量**

尾迹中的每个涡元素都会对转子升力线上的诱导速度产生贡献。尾迹越长，贡献越多。但需要权衡，因为尾迹长度增加会导致涡元素数量增加，从而提高计算时间。


如果尾迹由涡量分布恒定的涡柱组成，可以证明，尾迹长度为4D、5D和6D时，分别可以获得99.2%、99.5%和99.7%的诱导速度。因此，我们建议总尾迹长度至少为4D。作为近似，尾迹对流的距离是平均风速:math:`U_0`和尾迹内诱导速度的函数。诱导速度随下游距离变化，从转子处的:math:`-aU_0`（其中:math:`a`是转子处的平均轴向诱导因子）变化到尾迹达到平衡后的:math:`-2aU_0`（根据动量理论）。作为近似，可以假设对流速度为:math:`U_c = U_0(1-k_a a)`，其中:math:`k_a\approx1.2`。请注意，这里没有考虑粘性扩散，因此对于没有湍流的仿真，尾迹对流速度预计不会恢复到来流速度，应该使用"较大"的:math:`k_a`值（例如:math:`k_a\approx1.5`）。对于有湍流的仿真，尾迹摆动会扩散尾迹，应该使用较小的:math:`k_a`值（例如:math:`k_a\approx1.0`）。轴向诱导因子是运行工况和转子设计的函数。对于下面的估计，我们将使用:math:`a\approx1/3`。因此，尾迹达到目标下游距离:math:`d_\text{target}`所需的近似时间为：


.. math::

    T_\text{target} = \frac{d_\text{target}}{U_0(1-k_a a)}

该时间对应的时间步长数量（即尾迹面板总数）为：

.. math:: :label: ntargetDist

    n_\text{target,distance}
       =  \frac{T_\text{target}}{\Delta t_\text{FVW}}
       =  \frac{d_\text{target}}{U_0(1-k_a a) \Delta t_\text{FVW}}
       \approx \frac{d_\text{target}}{0.5 U_0 \Delta t_\text{FVW}}
       \approx \frac{12 D}{U_0 \Delta t_\text{FVW}}
       \qquad \text{(整数)}

其中第一个近似使用了:math:`k_a a\approx 0.5`，第二个近似假设目标距离为6D。也可以基于总转数:math:`n_\text{rot}`来定义近尾迹面板数量，得到：

.. math:: :label: ntargetRot

    n_\text{target, rotations}
       = \frac{n_\text{rot} T_\text{rot}}{dt_\text{FVW}}
       = \frac{n_\text{rot} 2 \pi }{\Omega dt_\text{FVW}}
       = \frac{n_\text{rot} 60 }{\Omega_\text{RPM} dt_\text{FVW}}
       \qquad \text{(整数)}


OLAF的尾迹由两个区域组成，定义为"近尾迹"（NW）和"远尾迹"（FW），其中远尾迹由卷起的叶尖涡和根涡组成。远尾迹的精度较低，因此穿过远尾迹的速度剖面可能不准确。远尾迹的作用是减少计算时间。为了提高精度，建议使用较长的近尾迹和较短的远尾迹。远尾迹也可以完全移除。

近尾迹和远尾迹进一步分为两个区域，一个区域中涡丝可以自由对流，另一个区域中涡丝以冻结的平均诱导速度对流。拥有"冻结"尾迹区域的优点是，它可以减轻尾迹截断的影响，尾迹截断是一种错误的边界条件（涡线不能在流体中终止）。如果尾迹在仍然"自由"的情况下被截断，那么涡量会在该区域卷起。另一个优点是，在没有扩散的情况下，尾迹在下游往往会变得过度扭曲，达到涡丝表示有效性的极限。因此，拥有冻结的远尾迹区域是有用的。尾迹面板总数等于自由近尾迹面板、冻结近尾迹面板、自由远尾迹面板和冻结远尾迹面板的数量之和：

.. math::

    n_\text{target} = n_\text{NW} + n_\text{FW} = n_\text{NW,Free} + n_\text{NW,Frozen} + n_\text{FW,Free} + n_\text{FW,Frozen}

OLAF输入文件定义了
:math:`n_\text{NW}`      (**nNWPanels**),
:math:`n_\text{NW,Free}` (**nNWPanelsFree**),
:math:`n_\text{FW}`      (**nFWPanels**), 和
:math:`n_\text{FW,Free}` (**nFWPanelsFree**).

如果使用了"冻结"近尾迹区域（:math:`n_\text{NW,Free}<n_\text{NW}`），那么"自由"远尾迹区域的长度需要为零（:math:`n_\text{FW,Free}=0`）。


我们目前建议：

- 总尾迹长度至少为4D（参见公式:eq:`ntargetDist`）
- 总尾迹长度至少对应10个转子转数（参见公式:eq:`ntargetRot`）
- （根据运行工况，上述两个条件中较大的一个将起主导作用，使用两者中较大的尾迹长度）
- 近尾迹范围至少对应8转
- 自由近尾迹范围至少对应1D
- 零远尾迹范围，或对应2转的冻结远尾迹范围

本节末尾提供的Python脚本实现了这些指南。

一般注意事项：

- 如果获得的功率系数高于贝茨极限，很可能是你的面板数量没有按照这些指南正确设置（面板数量太少）。
- 如果使用远尾迹，不要将其"自由"部分设置为超过长度的一半（即nFWPanelsFree <= nFWPanels/2）
- 近尾迹精度最高。如果计算时间不是主要问题，优先使用长近尾迹，短远尾迹或没有远尾迹。
- 目前，建议始终有一个冻结的近尾迹或冻结的远尾迹，以避免尾迹截断引入的误差。
- 远尾迹内的尾迹速度剖面可能不准确。
- 定期（例如每100个时间步）写入尾迹（**WrVTK=1或2**）以便目视检查。



**正则化参数**

涡方法的一个关键参数是正则化参数，也称为核心半径。我们目前建议将正则化参数设置为展向离散化（:math:`\Delta r`）的分数，即：**RegDeterMethod=3**，**WakeRegFactor=0.6**，**WingRegFactor=0.6**。当正则化因子设置为展向离散化的函数时，我们预计因子在0.25到3之间。


我们还建议正则化参数随下游距离增加：
**WakeRegMethod=3**。正则化参数随下游距离增加的因子对于现代多兆瓦风机可以设置为**CoreSpreadEddyVisc=1000**。当绘制尾迹时（**WrVTK**），如果稳态仿真的尾迹看起来"过于扭曲"，增加**CoreSpreadEddyVisc**参数来"平滑"尾迹。




**Python脚本**

以下Python脚本根据这些指南计算参数。（在这里检查更新：`olaf.py <https://github.com/ebranlard/welib/blob/dev/welib/fast/olaf.py>`_）


.. code::
   :number-lines:

   def OLAFParams(omega_rpm, U0, R, a=0.3, aScale=1.2,
                deltaPsiDeg=6, nPerRot=None,
                targetFreeWakeLengthD=1,
                targetWakeLengthD=4.,
                nNWrot=8, nFWrot=0, nFWrotFree=0,
                verbose=True, dt_glue_code=None):
      """
      根据以下输入计算OLAF的推荐时间步长和尾迹长度：

      输入：
       - omega_rpm: 旋转速度 [RPM]
       - U0: 平均风速 [m/s]
       - R: 转子半径 [m]

      时间步长选项：
         - 二者选其一：
            - deltaPsiDeg : 目标方位角离散化 [度]
                 或者
            - nPerRot     : 每转的时间步长数量。
                        deltaPsiDeg  -  nPerRot
                             5            72
                             6            60
                             7            51.5
                             8            45
        - dt_glue_code: 胶合代码时间步长。如果提供，OLAF的时间步长将近似为胶合代码时间步长的整数倍。

      尾迹长度选项：
        - a: 转子处的平均轴向诱导因子 [-]
        - aScale: 用于估计诱导的比例因子，使得尾迹对流速度为：Uc=U0(1-aScale*a)
        - targetWakeLengthD: 目标尾迹长度，以直径为单位 [D]
        - nNWrot     : 最小近尾迹转数
        - nFWrot     : 最小远尾迹转数
        - nFWrotFree : 最小远尾迹转数（自由面板）

      """
      def myprint(*args, **kwargs):
          if verbose:
              print(*args, **kwargs)

      # 旋转周期
      omega = omega_rpm*2*np.pi/60
      T = 2*np.pi/omega
      # 对流速度
      Uc = U0 * (1-aScale*a)

      # 期望的时间步长
      if nPerRot is not None:
          dt_wanted    = np.around(T/nPerRot,5)
          deltaPsiDeg  = np.around(omega*dt_wanted*180/np.pi ,2)
      else:
          dt_wanted    = np.around(deltaPsiDeg/(6*omega_rpm),5)
          nPerRot = int(2*np.pi /(deltaPsiDeg*np.pi/180))

      # 根据胶合代码时间步长调整期望的时间步长
      if dt_glue_code is not None:
          dt_rounded = round(dt_wanted/dt_glue_code)*dt_glue_code
          deltaPsiDeg2 = np.around(omega*dt_rounded *180/np.pi ,2)
          myprint('>>> 为了满足胶合代码时间步长要求：')
          myprint('    将dt   从 {} 四舍五入到 {}'.format(dt_wanted, dt_rounded    ))
          myprint('    将dpsi 从 {} 更改为 {}'.format(deltaPsiDeg, deltaPsiDeg2))
          dt_fvw   = dt_rounded
          deltaPsiDeg = deltaPsiDeg2
          nPerRot = int(2*np.pi /(deltaPsiDeg*np.pi/180))
      else:
          dt_fvw = dt_wanted

      # 有用的函数
      n2L = lambda n: (n * dt_fvw * Uc)/(2*R)  # 将面板数量转换为距离
      n2R = lambda n:  n * dt_fvw / T          # 将面板数量转换为转数

      # 总尾迹（AW）面板 - 根据平均风速计算尾迹长度
      targetWakeLength = targetWakeLengthD * 2 * R
      nAWPanels_FromU0 = int(targetWakeLength / (Uc*dt_fvw))
      # 自由近尾迹面板（基于距离）
      targetFreeWakeLength = targetFreeWakeLengthD * 2 * R
      nNWPanelsFree_FromU0 = int(targetFreeWakeLength / (Uc*dt_fvw))
      # 远尾迹（FW）面板，始终基于转数
      nFWPanels     = int(nFWrot*nPerRot)
      nFWPanelsFree = int(nFWrotFree*nPerRot)
      # 根据旋转速度和转数计算尾迹长度
      nAWPanels_FromRot = int(nNWrot*nPerRot) # 总面板数 NW+FW

      # 下面我们在转数标准或下游距离标准之间选择
      # 这可以调整/改进
      myprint('根据风速和距离计算的面板数（NW自由）：{:15d}'.format(nNWPanelsFree_FromU0))
      myprint('根据风速和距离计算的面板数（NW+FW）：{:15d}'.format(nAWPanels_FromU0))
      myprint('根据转数计算的面板数（NW+FW）：{:15d}'.format(nAWPanels_FromRot))
      myprint('两者平均的面板数（NW+FW）：{:15d}'.format(int((nAWPanels_FromRot+nAWPanels_FromU0)/2)))
      if nAWPanels_FromRot>nAWPanels_FromU0:
          # 基于转数的标准胜出：
          myprint('[信息] 使用转数设置面板数量')
          nAWPanels = nAWPanels_FromRot # 总面板数 NW+FW
      else:
          myprint('[信息] 使用风速和距离设置面板数量')
          # 尾迹距离标准胜出，我们保持来自转数的nFW但增加nNW
          nAWPanels = nAWPanels_FromU0  # 总面板数 NW+FW
      nNWPanels = nAWPanels - nFWPanels # nNW = 总数 - 远尾迹

      # 处理"自由"近尾迹
      nNWPanelsFree = nNWPanelsFree_FromU0
      if nNWPanelsFree>nNWPanels:
          nNWPanelsFree=nNWPanels
          myprint('[信息] 将自由NW面板数限制为最大值')
      if nNWPanelsFree<nNWPanels and nFWPanelsFree>0:
          nFWPanelsFree=0
          myprint('[信息] 因为使用了冻结近尾迹，将自由FW面板数设置为零')

      # 过渡时间（两倍于发展完整尾迹范围所需的时间）
      # 这是预计尾迹收敛之前的最小推荐时间（可能相当长）
      tMin = 2 * dt_fvw*nAWPanels
      if verbose:
          myprint('')
          myprint('{:15.2f} 过渡时间   ({:5.1f} 转)'.format(tMin, tMin/T))
          myprint('{:15d} nAWPanels        ({:5.1f} 转, {:5.1f}D)'.format(nAWPanels, n2R(nAWPanels), n2L(nAWPanels)))
          myprint('')
          myprint('OLAF输入文件：')
          myprint('----------------------- 常规选项 ---------------------')
          myprint('{:15.6f} DTFVW        (delta psi = {:5.1f}度)'.format(dt_fvw, deltaPsiDeg))
          myprint('--------------- 尾迹范围和离散化 --------------')
          myprint('{:15d} nNWPanels     ({:5.1f} 转, {:5.1f}D)'.format(nNWPanels    , n2R(nNWPanels    ), n2L(nNWPanels    )))
          myprint('{:15d} nNWPanelsFree ({:5.1f} 转, {:5.1f}D)'.format(nNWPanelsFree, n2R(nNWPanelsFree), n2L(nNWPanelsFree)))
          myprint('{:15d} nFWPanels     ({:5.1f} 转, {:5.1f}D)'.format(nFWPanels    , n2R(nFWPanels    ), n2L(nFWPanels    )))
          myprint('{:15d} nFWPanelsFree ({:5.1f} 转, {:5.1f}D)'.format(nFWPanelsFree, n2R(nFWPanelsFree), n2L(nFWPanelsFree)))

      return dt_fvw, tMin, nNWPanels, nNWPanelsFree, nFWPanels, nFWPanelsFree
