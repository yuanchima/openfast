
.. _AA-introduction:

介绍
-----

风能在电力结构中的渗透率不断提高，这得益于装机容量的持续增长，而这些装机容量目前大多位于陆上。然而，陆上安装越来越受到当地法规的限制，其中一个常见的限制因素是最大允许噪声水平。为了进一步增加陆上安装的数量，开发准确的建模工具来估算风力机产生的噪声非常重要。这有助于更准确地评估噪声排放，并有可能设计出更安静的风力机。

风力机产生的噪声主要有两个来源：

-  转子叶片与湍流大气边界层相互作用产生的气动噪声

-  机舱组件产生的机械噪声，主要来自齿轮箱、发电机和偏航机构。

本工作针对第一类噪声产生，旨在提供一组开源模型来估算任意风力机转子产生的气动噪声。这些模型用 Fortran 实现，并完全耦合到气动伺服弹性风力机仿真器 OpenFAST 中。代码可在 OpenFAST 的 GitHub 仓库中获取。[1]_ 代码基于 NAFNoise 的实现以及 :cite:`aa-MoriartyMigliore:2003` 和 :cite:`aa-Moriarty:2005` 中介绍的文档。OpenFAST 作为模块化框架实现，气动噪声模型作为 AeroDyn 的子模块实现 (:cite:`aa-MoriartyHansen:2005`)。

这组模型在 :numref:`AA-noise-models` 中描述，并在 :numref:`AA-model-verification` 中应用于国际能源署（IEA）陆上参考风力机的噪声估算。在 :numref:`AA-model-verification` 中，我们还展示了与慕尼黑工业大学实现的噪声模型运行结果的对比。本文档最后给出结论、未来工作展望以及附录，其中介绍了 OpenFAST 的输入文件。


.. [1]
   https://github.com/OpenFAST/openfast
