# OpenFAST 文档中文翻译任务

本文件供 Claude Code 每次启动时自动读取，定义将 OpenFAST 英文 RST 文档翻译为中文的完整规则和工作流程。

---

## 任务概述

将 `docs/` 目录下的全部 RST 文档翻译为中文，输出至仓库根目录下的 `docs_zh/` 目录，完整镜像 `docs/` 的子目录结构，不改变任何文件名和子目录名。

**路径映射规则：**
```
docs/[任意路径]/[文件名].rst
  →
docs_zh/[任意路径]/[文件名].rst
```

示例：
- `docs/index.rst` → `docs_zh/index.rst`
- `docs/source/user/aerodyn/input.rst` → `docs_zh/source/user/aerodyn/input.rst`
- `docs/source/install/index.rst` → `docs_zh/source/install/index.rst`

---

## 翻译顺序与优先级

### 入口文件

整个翻译任务从 `docs/index.rst` 出发，解析其 `.. toctree::` 指令获得顶层章节顺序：

```
docs/index.rst  ←  翻译总入口
  ├── source/this_doc.rst
  ├── source/install/index.rst
  ├── source/working.rst
  ├── source/user/index.rst          ← 最高优先级，优先完整翻译
  ├── source/testing/index.rst
  ├── source/dev/index.rst
  ├── source/license.rst
  ├── source/help.rst
  └── source/acknowledgements.rst
```

### 优先级规则

**始终优先翻译 `docs/source/user/` 下的全部文件，完成后再按顶层 toctree 顺序翻译其余章节。**

具体执行顺序：

1. **第一阶段（最高优先）**：`docs/source/user/` 下全部 RST 文件
   - 从 `docs/source/user/index.rst` 出发，按其 toctree 顺序逐模块翻译
   - 每个模块目录内，按该模块 `index.rst` 的 toctree 顺序逐文件翻译

2. **第二阶段**：`docs/source/install/` 下全部 RST 文件

3. **第三阶段**：顶层单文件（`source/this_doc.rst`、`source/working.rst`、`source/license.rst`、`source/help.rst`、`source/acknowledgements.rst`）

4. **第四阶段**：`docs/source/testing/` 下全部 RST 文件

5. **第五阶段**：`docs/source/dev/` 下全部 RST 文件

6. **最后**：`docs/index.rst` 本身

### user/ 内部模块顺序

按 `docs/source/user/index.rst` 实际 toctree 解析结果为准，已知顺序大致为：

```
source/user/index.rst
  ├── openfast/          （Glue Code 耦合框架）
  ├── aerodyn/           （气动力模块）
  ├── aerodyn-olaf/      （OLAF 自由涡尾迹）
  ├── aeroacoustics/     （气动噪声）
  ├── aerodisk/          （AeroDisk 简化致动盘）
  ├── beamdyn/           （叶片结构动力学）
  ├── subdyn/            （子结构动力学）
  ├── extptfm/           （外部平台超单元）
  ├── elastodyn/         （整机弹性动力学）
  ├── hydrodyn/          （水动力模块）
  ├── seastate/          （海洋状态）
  ├── inflowwind/        （来流风场）
  ├── moordyn/           （系泊系统）
  ├── servodyn/          （控制系统）
  ├── sed/               （Simplified ElastoDyn）
  ├── structural_control/（结构控制）
  ├── turbsim/           （湍流风场生成器）
  ├── fast.farm/         （风电场多机仿真）
  ├── cppapi/            （C++ API）
  ├── wavetank/          （波浪水槽）
  └── 顶层散落文件：api_change.rst、fast_to_openfast.rst 等
```

> 以上顺序仅供参考，实际执行时以读取 `index.rst` 解析结果为准。

---

## 翻译规则

### 1. RST 格式：完整保留，不得修改

以下所有 RST 语法元素原样保留：

- **标题装饰行**：`=====`、`-----`、`~~~~~`、`^^^^^`、`"""""` 等，必须与上方文字等长
- **指令块**：`.. toctree::`、`.. note::`、`.. warning::`、`.. important::`、`.. tip::`、`.. math::`、`.. code-block::`、`.. figure::`、`.. image::`、`.. only::`、`.. rubric::` 等
- **指令选项**：`:maxdepth:`、`:hidden:`、`:numbered:`、`:caption:`、`:name:`、`:width:`、`:alt:` 等，选项值不翻译
- **内联角色**：
  - `:ref:\`label\`` — 标签名不翻译；有显示文字时（`:ref:\`显示文字 <label>\``）显示文字可翻译
  - `:numref:\`label\`` — 整体不翻译
  - `:download:\`显示文字 <路径>\`` — 显示文字可翻译，路径不翻译
  - `:math:\`公式\`` — 公式内容不翻译
  - `:class:`、`:func:`、`:meth:`、`:attr:`、`:mod:` 等代码角色 — 不翻译
- **标签定义行**：`.. _label-name:` 整行不翻译
- **超链接**：所有 URL 原样保留
- **表格分隔符**：`=====`、`-----`、`+-----+` 等
- **RST 注释行**：`.. ` 开头的注释（本任务自动添加的文件头注释除外）

### 2. 绝对不翻译的内容

- `.. code-block::` 块内的**全部**内容（命令行、代码、配置示例等）
- 行内代码 `` `code` ``（单反引号包裹）
- 输入文件示例中的参数关键字，例如：`CompAero`、`DT`、`TMax`、`Echo`、`WakeMod`、`NumBlades` 等
- **模块名称**（正文中出现时保留英文）：`OpenFAST`、`AeroDyn`、`BeamDyn`、`ElastoDyn`、`ServoDyn`、`HydroDyn`、`SeaState`、`InflowWind`、`MoorDyn`、`SubDyn`、`ExtPtfm`、`TurbSim`、`OLAF`、`FAST.Farm`、`AeroDisk`、`SED`、`NWTC Library`
- `.. math::` 块内及行内 `:math:\`...\`` 中的数学表达式
- 单位符号：m、s、kg、N、Pa、rad、rad/s、N·m、Hz、rpm、kW、MW 等
- 文件名、目录路径、程序名（如 `openfast.exe`、`TurbSim.exe`）
- 变量名、函数名、子程序名
- Fortran / C / C++ / Python / MATLAB 代码片段
- 人名；机构缩写（NREL、NWTC、DOE 等）保留，首次出现时括注中文全称

### 3. 术语统一对照表

| 英文原文 | 中文译文 |
|---------|---------|
| glue code | 耦合框架（胶水代码） |
| module | 模块 |
| coupling / coupled | 耦合 |
| blade | 叶片 |
| rotor | 转子 |
| nacelle | 机舱 |
| tower | 塔筒 |
| drivetrain | 传动链 |
| hub | 轮毂 |
| hub height | 轮毂高度 |
| airfoil | 翼型 |
| lift | 升力 |
| drag | 阻力 |
| pitching moment | 俯仰力矩 |
| aerodynamic loads | 气动载荷 |
| structural dynamics | 结构动力学 |
| aero-hydro-servo-elastic | 气动-水动-伺服-弹性耦合 |
| time-domain | 时域 |
| offshore | 海上 |
| onshore / land-based | 陆上 |
| fixed-bottom | 固定式基础 |
| floating | 浮式 |
| substructure | 子结构 |
| platform | 浮式平台 |
| wake | 尾流 |
| inflow wind | 来流风 |
| wind shear | 风切变 |
| turbulence | 湍流 |
| pitch | 变桨 |
| yaw | 偏航 |
| generator torque | 发电机转矩 |
| fatigue loads | 疲劳载荷 |
| ultimate loads | 极限载荷 |
| degree of freedom (DOF) | 自由度（DOF） |
| finite element | 有限元 |
| actuator line | 致动线 |
| BEM (Blade Element Momentum) | 叶素动量理论（BEM） |
| tip-loss correction | 叶尖损失修正 |
| hub-loss correction | 轮毂损失修正 |
| unsteady aerodynamics | 非定常气动力 |
| skewed wake | 偏斜尾流 |
| free vortex wake | 自由涡尾迹 |
| mooring | 系泊 |
| wave | 波浪 |
| current | 海流 |
| linearization | 线性化 |
| driver / standalone driver | 独立驱动程序 |
| time step | 时间步长 |
| simulation | 仿真 |
| input file | 输入文件 |
| output file | 输出文件 |
| summary file | 汇总文件 |
| regression test | 回归测试 |
| unit test | 单元测试 |
| source code | 源代码 |
| compile / build | 编译 / 构建 |
| executable | 可执行文件 |
| binary | 二进制文件 |

### 4. 每个翻译文件的文件头

不要在每个翻译后的 RST 文件**最顶部第一行**插入以下注释，如果已经插入请删除，保持和源文件的结构一致：

```rst
.. 本文档由 Claude Code 自动翻译，原文路径：docs/[相对路径]/[文件名].rst，翻译日期：[YYYY-MM-DD]
```

### 5. 排版细节

- 保持原文件所有**空行**不变（RST 用空行分隔段落和块，不能随意增删）
- 保持原文件所有**缩进**不变（指令选项、代码块等缩进错误会导致 Sphinx 渲染失败）
- 中文正文中，英文单词/术语/数字与中文之间各加一个半角空格
- 不修改任何文件名（保持与原文件完全相同，包括大小写）

---

## 执行流程

### 启动指令

用户说"**开始翻译**"或"**继续翻译**"时，执行以下操作：

1. 读取 `docs/source/user/index.rst`，解析 toctree（已解析过则跳过）
2. 扫描 `docs_zh/` 统计已完成与剩余文件数
3. 输出当前进度摘要：
   ```
   [进度] 已翻译：42 个文件 / 剩余：138 个文件
   [当前阶段] 第一阶段：source/user/ — aerodyn/ 模块（共 11 个文件，已完成 3 个）
   [下一个] docs/source/user/aerodyn/input.rst
   ```
4. 翻译下一个未完成文件，保存，继续推进

### 断点续传

每次翻译前先检查 `docs_zh/` 中对应路径是否已存在该文件，已存在则跳过，继续下一个。

### 进度播报

每完成一个模块目录输出一行：
```
[已完成] source/user/aerodyn/ — 11 个文件 ✓
[开始] source/user/aerodyn-olaf/
```

### 大文件处理

单个 RST 文件超过约 600 行时，分段翻译后合并输出，不在指令块、代码块、数学块内部断开。

---

## 各章节特殊说明

| 章节 | 特殊处理 |
|------|---------|
| `source/user/` 理论章节（`theory*.rst`）| 不确定的物理概念保留英文并括注中文，例如：generalized-α solver（广义-α 求解器）|
| `source/user/` 附录（`appendix*.rst`）| 表头翻译，参数关键字列不翻译，描述列翻译 |
| `.. only:: html` / `.. only:: latex` | 块内文字正常翻译，指令行保留 |
| `source/install/`（安装文档）| 命令行步骤中的命令不翻译，说明性文字翻译 |
| `source/dev/`（开发者文档）| 内联代码和 API 名称不翻译，仅翻译说明性散文 |
| `source/testing/`（测试文档）| 测试命令、脚本名不翻译，流程说明翻译 |