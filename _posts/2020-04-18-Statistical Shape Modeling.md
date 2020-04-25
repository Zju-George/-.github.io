---
layout: post
title: "Statistical Shape Modeling"
permalink: /ssm
---
本文档介绍论文 [Gaussian Process Morphable Models]({{ 'https://arxiv.org/abs/1603.07254' | absolute_url }}) 中最重要的将高斯过程离散化的代码实现。完整代码见 [`gpmm.py`](https://github.com/Zju-George/blenderStatisticalModeling/blob/master/gpmm.py)。

<br>

<center><img src="/assets/formula.png" alt="公式" width="80%" height="80%" align="center" /></center>

对于公式 5 ，特征值 $\lambda_i$ 从大到小排列，而随着下标 i 增大，$\lambda_i$ 的值下降的很快。则可以用有限项来近似，

<center><img src="/assets/fomula6.png" alt="公式" width="70%" height="70%" align="center" /></center>

关键是求积分算子 (7) 对应的特征值 $\lambda_i$ 和基函数 $\lbrace \Phi_i \rbrace_{i=1}^r$。

<br>

<center><img src="/assets/formula7.png" alt="公式" width="70%" height="70%" align="center" /></center>

这个就要利用 Nystrom method 来将积分算子离散化，具体公式见 (9) (10)。
首先代码中我们要定义高斯核函数:

```python
def GaussianKernelValue(point1, point2, sigma):
    '''
    access point1&2 by name if point is string
    '''
    if(type(point1)==str):
        point1=bpy.data.collections[0].objects[point1].location
    if(type(point2)==str):
        point2=bpy.data.collections[0].objects[point2].location
    distance=(point1[0]-point2[0])**2+(point1[1]-point2[1])**2+(
        point1[2]-point2[2])**2
    distance = distance ** 0.5
    tmp=-1*distance/(sigma**2)
    result = math.exp(tmp)
    return result
```

然后计算公式 (9) 的矩阵 K:

```python
```