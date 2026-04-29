.. _code_style:

代码风格
~~~~~~~~~~
OpenFAST 及其底层模块主要使用 Fortran 编写，遵循
2003 标准，但模块也可以用 C 或 C++ 编写。
:download:`NWTC 程序员手册 <../../OtherSupporting/NWTC_Programmers_Handbook.pdf>`
是所有与 FAST 框架及相关工作以及在 OpenFAST 中添加代码的问题的权威参考。

通常，代码应该写得直截了当、易于阅读。
语法糖或简洁性不应损害可读性。例外
情况是当性能要求代码可读性较差时。
此时，应使用注释块来描述代码中不明显的内容。
缩进通常为三个空格，不使用制表符。
