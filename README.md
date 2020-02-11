# 行业词语义表达
基于知识的词语义表达方法

去融合语义表示（De-Conflated Semantic Representations）
=====
准备工作：

    ①一组预训练的词表达（如word embedding）（3.1）
    ②语义关系可视为G = (V, E)的词汇资源，V中的每个顶点对应于一个概念，E中的每条边表示顶点之间的词语义关系。
    （WordNet顶点是同义词集，WordNet中的映射功能是将每个同义词集映射到它包含的所有同义词上）
方法概述：

    ①方法要点在于基于给定词义的偏义词列表的计算。
    ②首先分析WordNet的语义网络，并提取最能够表示各独立同义词集语义的词列表（2.2）
确定语义偏义词：
> 具体算法：
    
    Require: Graph G = (V, E) of vertices V = {yi} (i=1,2,...,m;of m synsets) and edges E (semantic relationships between synsets)
    Require: Function µ(yi) that returns for a given synset yi the words it contains
    Require: Target synset yt ∈ V for which a sense biasing word sequence is required
    Ensure: The sequence Bt of sense biasing words for synset yt
    
    1: Bt ← ()
    2: for all word w in µ(yt) do
    3:  Bt ← Bt ∪ (w)
    4: for yi ∈ V : yi = yt do
    5:  pi ← PERSONALIZEDPAGERANK(yi, yt, G)
    6: yh(h=1,2,...,m-1) ← SORT(V \{yt}) according to scores pi
    7: for h:1 to m-1 do
    8:  for all word w in µ(y∗h) do
    9:      if w /∈ Bt then
    10:     Bt ← Bt ∪ (w)
    11: return sequence Bt
    
    
    
    
