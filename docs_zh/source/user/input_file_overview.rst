.. _input_file_overview:

输入文件格式
==========

OpenFAST 使用两种主要的输入文件格式：*值列* 格式（读取每行的第一个值）和 *键+值* 格式（读取值和关键字对）。两种格式都是基于行号的，特定的输入需要出现在特定的行上，但也有一些例外。

.. _sec_value_column:

值列输入文件
------------

在基于 *值列* 的输入文件中，只会读取第一列内容。这是 OpenFAST 及其前身使用的历史格式（关键字通常在源代码和文档中被引用，但 OpenFAST 不会处理关键字或描述）。读取到第一个值之后的所有内容都会被代码忽略。这允许用户在修改内容时保留旧值。例如，如下输入行：

::

 2       20   TMax            - Total run time (s)

会被读取为 `2`，而 `20` 及其之后的所有内容都会被忽略。

这种格式及其相关的解析方法在向用户通知解析错误方面存在一定局限，同时也限制了将整个输入文件作为文本字符串从其他代码（如 Python 驱动代码）传递的能力。

.. _sec_format_key_value:

键+值输入文件
-------------

在 *键+值* 输入文件中，会读取前两列内容。这两列中必须有一列包含 **精确** 的关键字，另一列包含对应的值。例如，如下输入行：

::

         20   TMax            - Total run time (s)

等价于：

::

   TMax         20            - Total run time (s)

这种输入文件格式的另一个特性是允许用户在任意位置添加任意数量的注释行。任何以 `!`、`#` 或 `%` 开头的行都会被视为注释行并被忽略。例如：

::

  ! This is a comment line that will be skipped
          %  and this is also a comment line that will be skipped
  # as is this comment line
         20   TMax            - Total run time (s)
  ! the first two columns in the above line will be read as the value + key

这种格式输入文件的解析器还会跟踪哪些行是注释，哪些行包含值和键对。如果未找到某个键名，解析器会返回错误，并提供它正在读取的行号信息。

使用键+值格式的模块
~~~~~~~~~~~~~~~~~~~

以下模块使用 *键+值* 格式的输入文件（所有其他模块使用 *值列* 格式）：

============== ==========================================================
 模块         输入文件
============== ==========================================================
AeroDyn         AeroDyn 主输入文件
AeroDyn         翼型文件
HydroDyn        HD 主输入文件
InflowWind      IfW 主输入文件
InflowWind      均匀风输入文件
InflowWind      Bladed 风摘要文件
ServoDyn        ServoDyn 主输入文件
ServoDyn        结构控制子模块输入文件
ServoDyn        结构控制子模块预定力输入文件
SubDyn          SubDyn SSI 矩阵输入文件
============== ==========================================================

请注意，如果值写在键之前，键+值格式和值列输入文件可以是相同的。

变更原因
~~~~~~~~

变更输入文件解析方式的主要原因是为了允许将完整的输入文件通过内存从封装代码传递到 OpenFAST 或其模块中。例如，当将 AeroDyn 模块集成到 Python 代码中时，可以直接在内存中传递输入文件，而无需先写入磁盘。这有助于减少优化循环中的 IO 开销，在优化循环中，模块可能会被连续调用多次，且输入文件的变化非常小。*注意：这仍是一项正在进行的工作，因此并非所有模块都可以通过这种方式链接*。

为了实现这一功能，我们使用了由 Marshall Buhl 为 FAST8 中的 AeroDyn 15 编写的解析翼型表的文件解析器。该解析器支持更健壮的 *键+值* 输入格式。

.. _sec_troubleshoot_input_file:

输入文件故障排除
-----------------

当排查输入文件错误时，请尝试以下步骤：

1. 如果错误消息包含行号和变量名，则正在解析的文件格式为 *键+值* 格式。检查关键字的拼写是否与输入文件中的完全一致。请参阅 :numref:`sec_format_key_value`。
2. 如果错误消息仅包含变量名但没有行号，则为 *值列* 输入文件格式。请参阅 :numref:`sec_value_column`。
3. 打开输入文件中的 `echo` 选项，检查生成的 `.ech` 文件，查看文件解析停止在哪一行。当错误消息中没有给出行号时，这可能有助于定位输入文件解析失败的位置。
4. 将有问题的输入文件与 OpenFAST 分发的回归测试套件中的同类型输入文件进行比较。有关回归测试的详细信息，请参阅 :numref:`testing` 部分，或查看 `r-test <https://github.com/openfast/r-test>`__ 仓库。

..
   Input file type by module
   -------------------------
   ============== ====================== =====================
    Module         Input file             Type
   ============== ====================== =====================
   OpenFAST        Main .fst input file   Value column
   OpenFAST        Matlab mode shape      Value column
   OpenFAST        Mode shape             Value column
   OpenFAST        Checkpoint file        Binary
   ============== ====================== =====================
