.. _exbladed:

扩展 Bladed 接口
=========================

Bladed 风格 DLL 控制器接口已扩展，允许在预留范围内以通道组的形式
使用大量新通道。如下 :numref:`fig:BlEx` 所示。

.. figure:: BladedExInterface.png
   :alt: Bladed DLL 接口扩展的通道方案。
   :name: fig:BlEx
   :width: 100.0%

   Bladed DLL 接口扩展的通道方案。


ServoDyn 汇总文件包含所有正在使用的 DLL 接口通道的摘要，
以及已预留的通道块。

.. container::
   :name: SrvDSum

   .. literalinclude:: SrvD--Ex.sum
      :language: none
