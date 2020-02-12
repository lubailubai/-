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
    3:     Bt ← Bt ∪ (w)
    4: for yi ∈ V : yi = yt do
    5:     pi ← PERSONALIZEDPAGERANK(yi, yt, G)
    6: yh(h=1,2,...,m-1) ← SORT(V \{yt}) according to scores pi
    7: for h:1 to m-1 do
    8:     for all word w in µ(yh) do
    9:         if w /∈ Bt then
    10:        Bt ← Bt ∪ (w)
    11: return sequence Bt
    
>  Personalized PageRank Algorithm(个性化PageRank)

    P[t+1] = (1-σ)P[0] + σMP[t]
    
    * 首先将WordNet的语义网络表示为行随机转移矩阵M，其中m是WordNet中的synset个数。如果synset i和synset j之间存在语义关系，单元格Mij被设为i的次数的倒数，否则设置为零。
    * σ为阻尼系数，通常设为0.85
    * P[0]为初始化向量，与yt对应维度设为1.0
    * The weight pi in line 5 is the value of the ith dimension of the PPR vector P computed for the synset yt.
学习语义表示：

    argmin v∗[si] α d(v∗[si], v[si]) + δij d(v∗[si], v[bij] )
    
    * where v[si] and v[bij] are the respective word representations (∈ V) of the lemma of si and the jth biasing word in the list of biasing words for si
    
义元信息参与的词向量表达
=====
概念及背景知识：

* 1.Hownet有三个主要的概念，即sememe（义元），sense（词义）和word（词语）。
    
        eg：word（苹果）包含了两个sense，sense1是水果，sense2是电脑
    
    对于每一个sense来说是由sememe来描述其属性;
    在hownet的描述体系中，sememe是最小的描述单元，相似的sense会包含相同的sememe，这些sememe就相当于是整个世界所有事物最基本的单元。
    而且sememe的数量相对词语的数量来说是非常有限的，在hownet中大概就是几百个。
* 2.SkipGram：
    
        +{i=K,...,n-K} logPr(w[i-k],...,w[i+k]|w[i]) 
  其中K代表了滑动窗口的大小，它定义了target词语所对应的上下文的范围，也就是说它的整体目标是利用当前词语target，来最大化的预测其上下文的信息，skip-gram对于上下文的信息做了简化处理，即从K大小的窗口内随机抽取一个词语来作为其上下文的替代，故上面的最优化式子简化如下： 
  
        Pr(w[i-k],...,w[i+k]|w[i]) = *{w[c] b.t. C(w[i]} Pr(w[c]|w[i]) 
        = *{w[c] b.t. C(w[i]} (exp(w[c]*w[i])/+{w[j] b.t. Dictionaryexp(w[i]*w[j])})

模型：
* 1.Simple Sememe Aggregation Model (SSA) 

        在传统的skip-gram模型基础之上，对target word部分进行改造，使用target word 由对应的所有sense包含的所有的sememe embedding相加求均值得到。
    
        和传统的skip-gram相比，这种方式里sememe向量会被一些词语共享，模型可以利用sememe之间的共享信息来表征词语之间的某些语义关联关系。在这种模式之下，拥有相同sememe的词语最终会有比较相似的向量表征形式。 
    
* 2.Sememe Attention over Context Model (SAC) 

        这部分应用了attention机制，而且是在context内容上进行应用的。
        
        因为对于一个词语的表征，在不同的语境中其对应的sense集合中的元素之间的重要性也是不同的，这里很好的应用了attentin机制来对于重要性进行标识。
    
        一个词语的sense向量是由其对应的sememe向量直接相加求均值得到。然后三个不同的sense向量s1，s2，s3通过attention机制相加得到最终context的word 级别的embedding形式，具体三种权值的大小的选取由这三种sense向量和target 向量的相似程度得到。
    
        将注意力机制引入到模型中，从而可以辅助训练过程中的消歧。
    
         也就是说如果一个上下文词的某个义原跟中心词的意思更加相近，那么他就获得更多的关注，从而在生成上下文词的向量表示时，这个义原的权重就会越大，而最终生成的上下文词向量也就具有更好的表达能力。所以这个模型也称之为上下文模型（context model）。
    
* 3.Sememe Attention over Target Model (SAT) 

    SAT和SAC模型正好是互相对称的，但是需要注意的是，这里不是从上下文随机挑选了一个词，而是应用了一定窗口内的词，这样的context可以更好的去生成target word 的向量表征形式。
    
    这里通过利用上下文信息对中心词的义原信息的关注程度，从而选择出符合情境的义原信息，为中心词表示的生成提供必要的辅助，也就提升了中心词的向量表示能力，因此这个模型也被称之为目标模型（target model）。
    
        
        
    
    
    
    
    
    
    
