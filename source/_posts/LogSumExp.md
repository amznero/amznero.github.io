---
title: LogSumExp求导
date: 2024-10-08 11:07:07
tags:
---
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css" integrity="sha384-GvrOXuhMATgEsSwCs4smul74iXGOixntILdUW9XmUC6+HX0sLNAK3q71HotJqlAn" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js" integrity="sha384-cpW21h6RZv/phavutF+AuVYrr+dA8xD9zs6FwLpaCct6O9ctzYFfFr4dgmgccOTx" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js" integrity="sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05" crossorigin="anonymous"></script>
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
          // customised options
          // • auto-render specific keys, e.g.:
          delimiters: [
              {left: '$$', right: '$$', display: true},
              {left: '$', right: '$', display: false},
              {left: '\\(', right: '\\)', display: false},
              {left: '\\[', right: '\\]', display: true}
          ],
          // • rendering keys, e.g.:
          throwOnError : false
        });
    });
</script>


最近学了下陈天奇大佬的DeepLearningSystem课程，HW2里面有一块是对LogSumExp（简称LSE）算子求导数。
LSE应用非常广泛（例如多分类里的Softmax可以利用LSE来解决上溢问题 ）。
所以这篇文章对LSE做了一个求导(但写的有点繁琐
顺便练练LaTeX :D
下面是一些符号的说明:

<span>
$$input: z \in \mathbb{R}^n \\ argmax \left(z \right) = j, \max{z}=z_j\\ \hat{z_{i}} = z_{i} - \max{z}=z_i-z_j\\ LogSumExp(z_i) = \log(\sum_{k=1}^{n}\exp(z_{i}-\max{z}))+\max{z}=\log(\sum_{k=1}^{n}\exp(\hat{z_i}))+z_j \\ LSE=LogSumExp$$
</span>

专门强调下，其中最大元素对应的编号为$j$，即$\max{z}=z_j$

**下面分两种情况对LSE求导：**

1. 当$i\neq j$时：

<span>
$$\begin{align} 		\frac{\partial{LSE}}{\partial{z_{i}}} &= \frac{\partial{LSE}}{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}} \cdot \frac{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{z_{i}}} + \frac{\partial{LSE}}{\partial{\max{z}}} \cdot \frac{\partial{\max{z}}}{\partial{z_{i}}} \\ 		&= 1 \cdot \frac{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{\sum_{k=1}^{n}{\exp(\hat{z_{k}})}}} \cdot \frac{\partial{\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{\hat{z_{i}}}} + 1 \cdot 0 \\ 		&= \frac{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{\sum_{k=1}^{n}{\exp(\hat{z_{k}})}}} \cdot \sum_{k=1}^{n}\left(\frac{\partial{\exp(\hat{z_{k}})}}{\partial{\hat{z_{i}}}}\right) \\ &= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \sum_{k=1}^{n}\left(\frac{\partial{\exp(\hat{z_{k}})}}{\partial{\hat{z_{k}}}} \cdot \frac{\partial{\hat{z_{k}}}}{\partial{z_{i}}} \right) \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \sum_{k=1}^{n}\left(\exp(\hat{z_k}) \cdot \frac{\partial{({z_{k}-\max{z}})}}{\partial{z_{i}}} \right) \\ 	&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \sum_{k=1}^{n}\left(\exp(\hat{z_{k}}) \cdot \mathbb{I}\left(k=i\right) \right) \\ 	&= \frac{1}{\sum\exp(\hat{z_{k}})} \cdot \exp(\hat{z_{i}}) \\ 	&= \frac{\exp(\hat{z_{i}})}{\sum_{k=1}^{n} {\exp(\hat{z_{k}})}} 	\nonumber 	   \end{align}$$
</span>

2. 当$i=j$时，即$z_i=z_j=\max{z} $：

<span>
$$\begin{align} 		\frac{\partial{LSE}}{\partial{z_{i}}} &= \frac{\partial{LSE}}{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}} \cdot \frac{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{z_{i}}} + \frac{\partial{LSE}}{\partial{\max{z}}} \cdot \frac{\partial{\max{z}}}{\partial{z_{i}}} \\ 		&= 1 \cdot \frac{\partial{\log\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{\sum_{k=1}^{n}{\exp(\hat{z_{k}})}}} \cdot \frac{\partial{\sum_{k=1}^{n}\exp(\hat{z_{k}})}}{\partial{\hat{z_{i}}}} + 1 \cdot 1 \\ 		&= \frac{1}{\partial{\sum_{k=1}^{n}{\exp(\hat{z_{k}})}}} \cdot \sum_{k=1}^{n}\left(\frac{\partial{\exp(\hat{z_{k}})}}{\partial{\hat{z_{i}}}}\right) + 1 \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \sum_{k=1}^{n}\left(\exp(\hat{z_{k}}) \cdot \frac{\partial{(z_k-\max{z})}}{\partial{z_{i}}} \right) + 1 \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \left( \exp(\hat{z_k})\cdot \frac{\partial{(z_i-\max{z})}}{\partial{z_i}} + \sum_{k=1,k \neq i}^{n}\left(\exp(\hat{z_{k}}) \cdot \frac{\partial{(z_k-\max{z})}}{\partial{z_{i}}} \right)\right) + 1 \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \left( \exp(\hat{z_k})\cdot \frac{\partial{(z_i-z_i)}}{\partial{z_i}} + \sum_{k=1,k \neq i}^{n}\left(\exp(\hat{z_{k}}) \cdot \frac{\partial{(z_k-z_i)}}{\partial{z_{i}}} \right)\right) + 1 \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \left( \exp(\hat{z_k})\cdot 0 + \sum_{k=1,k \neq i}^{n}\left(\exp\left(\hat{z_{k}}\right) \cdot -1 \right)\right) + 1 \\ 		&= \frac{1}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \cdot \left( -\sum_{k=1,k \neq i}^{n}\exp\left(\hat{z_{k}} \right)\right) + 1 \\ 		&= \frac{-\sum_{k=1,k \neq i}^{n}\exp\left(\hat{z_{k}} \right)}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} + \frac{\sum_{k=1}^{n}\exp(\hat{z_{k}})}{\sum_{k=1}^{n}\exp(\hat{z_{k}})}\\ 		&= \frac{\exp(\hat{z_{i}})}{\sum_{k=1}^{n}\exp(\hat{z_{k}})} \end{align}$$
</span>
---

可知两种情况下对LSE求导结果都等于$\frac{\exp(\hat{z_{i}})}{\sum_{k=1}^{n}\exp(\hat{z_{k}})}$

Ref:
1. [Hurry Z：关于LogSumExp](https://zhuanlan.zhihu.com/p/153535799)