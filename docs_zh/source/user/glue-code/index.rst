.. _glue-code:

耦合代码
========

OpenFAST *耦合代码* 是初始化每个物理模块、管理模块之间数据流、协调时间步进循环，以及可选地对组装好的系统进行线性化的软件层。本部分从用户和模块开发者的角度记录耦合代码的相关内容。

.. toctree::
   :maxdepth: 2

   overview
   modvar
   modglue
   solver
   linearization
