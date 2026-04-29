.. _regression_test_windows:

Windows 下使用 Visual Studio 的回归测试
==========================================

1) 克隆 openfast 仓库并初始化测试数据库

    a) 打开 git 命令终端窗口（如 git bash）

    b) 将工作目录切换到希望本地仓库所在位置的上层目录（仓库将在此位置放入名为 openfast 的文件夹中）

    c. 输入：``git clone https://github.com/openfast/openfast.git``（这将在您的计算机上创建 openfast 仓库的本地副本）。
    您应该看到类似以下内容：

    ::

          Cloning into 'openfast'...
          remote: Counting objects: 23801, done.
          remote: Compressing objects: 100% (80/80), done.
          remote: Total 23801 (delta 73), reused 102 (delta 50), pack-reused 23670
          Receiving objects: 100% (23801/23801), 92.10 MiB  18.99 MiB/s, done.
          Resolving deltas: 100% (13328/13328), done.
          Checking connectivity... done.


    d) 输入：``cd openfast``（将工作目录切换到 openfast 文件夹）

    e) 输入：``git checkout dev``（将本地仓库切换到 openfast 仓库的正确分支）

    f) 输入：``git submodule update --init --recursive``（将测试数据库下载到您的计算机）
       您应该看到类似以下内容：

    ::

          Submodule 'reg_tests/r-test' (https://github.com/openfast/r-test.git) registered for path 'reg_tests/r-test'
          Cloning into 'reg_tests/r-test'...
          remote: Counting objects: 3608, done.
          remote: Compressing objects: 100% (121/121), done.
          remote: Total 3608 (delta 22), reused 161 (delta 21), pack-reused 3442
          Receiving objects: 100% (3608/3608), 154.52 MiB  26.29 MiB/s, done.
          Resolving deltas: 100% (2578/2578), done.
          Checking connectivity... done.
          Submodule path 'reg_tests/r-test': checked out 'b808f1f3c1331fe5d03c5aaa4167532c2492d378'


2) 编译回归测试 DISCON DLL

    a) 打开位于 ``openfast\vs-build\Discon`` 文件夹中的 Visual Studio 解决方案（``Discon.sln``）

    b) 分别为解决方案配置和解决方案平台选择 Release 和 x64

    c) 从菜单栏选择 ``Build->Build Solution``

    d) 您现在应该能在 ``openfast\reg_tests\r-test\glue-codes\fast\5MW_Baseline\ServoData`` 文件夹中看到文件 ``Discon.dll``、``Discon_ITIBarge.dll`` 和 ``Discon_OC3Hywind.dll``。

3) 使用 Visual Studio 编译 OpenFAST

    a) 打开位于 ``openfast\vs-build\FAST`` 文件夹中的 Visual Studio 解决方案（``FAST.sln``）

    b) 分别为解决方案配置和解决方案平台选择 Release_Double 和 x64

    c) 从菜单栏选择 ``Build->Build Solution``

       i)  如果这是您第一次尝试编译 OpenFAST，您将遇到编译错误！！！[继续执行步骤 (ii) 和 (iii)，否则如果 FAST 成功编译，继续步骤 (3d)]

       ii) 使用菜单栏 ``Build->Cancel`` 取消编译
            [VS 对 FASTlib 中项目文件的编译顺序/依赖关系存在混淆，但取消并重启 VS 后，它从部分编译中以某种方式获得了足够的信息来正确处理]

       iii) 关闭 Visual Studio，然后重复步骤 (a) 至 (c)

    d) 您现在应该能在 ``openfast\build\bin`` 文件夹中看到文件 ``openfast_x64_Double.exe``。


4) 准备回归测试

    a) 在 ``openfast\build`` 文件夹中创建一个名为 ``reg_tests`` 的子目录。

    b) 将 ``openfast\reg_tests\r-test`` 的内容复制到 ``openfast\build\reg_tests``。


5) 执行 OpenFAST 回归测试

    a) 打开配置了 Python 的命令提示符 [ 如 Anaconda3 ]

    b) 将工作目录切换到 ``openfast\reg_tests``

    c) 输入：``python manualRegressionTest.py ..\build\bin\openfast_x64_Double.exe 2.0 1.9``
         您应该看到：``executing AWT_YFix_WSt``

    d) 测试将逐一执行，直到最终看到类似以下内容：

    ::

      executing AWT_YFix_WSt                           PASS
      executing AWT_WSt_StartUp_HighSpShutDown         PASS
      executing AWT_YFree_WSt                          PASS
      executing AWT_YFree_WTurb                        PASS
      executing AWT_WSt_StartUpShutDown                PASS
      executing AOC_WSt                                PASS
      executing AOC_YFree_WTurb                        PASS
      executing AOC_YFix_WSt                           PASS
      executing UAE_Dnwind_YRamp_WSt                   PASS
      executing UAE_Upwind_Rigid_WRamp_PwrCurve        PASS
      executing WP_VSP_WTurb_PitchFail                 PASS
      executing WP_VSP_ECD                             PASS
      executing WP_VSP_WTurb                           PASS
      executing WP_Stationary_Linear                   PASS
      executing SWRT_YFree_VS_EDG01                    PASS
      executing SWRT_YFree_VS_EDC01                    PASS
      executing SWRT_YFree_VS_WTurb                    PASS
      executing 5MW_Land_DLL_WTurb                     PASS
      executing 5MW_OC3Mnpl_DLL_WTurb_WavesIrr         PASS
      executing 5MW_OC3Trpd_DLL_WSt_WavesReg           PASS
      executing 5MW_OC4Jckt_DLL_WTurb_WavesIrr_MGrowth PASS
      executing 5MW_ITIBarge_DLL_WTurb_WavesIrr        PASS
      executing 5MW_TLP_DLL_WTurb_WavesIrr_WavesMulti  PASS
      executing 5MW_OC3Spar_DLL_WTurb_WavesIrr         PASS
      executing 5MW_OC4Semi_WSt_WavesWN                PASS
      executing 5MW_Land_BD_DLL_WTurb                  PASS

    e) 如果单个测试成功，您将看到 ``PASS``，否则将在该测试名称后看到 ``FAIL``
