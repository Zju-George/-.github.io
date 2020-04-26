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
def GetKMat3(rows=5, colums=5):
    '''the actual size is (n * dimensions, n * dimensions) dimensions = 2'''
    n = rows * colums * 5
    name = "Arrow"
    K = np.mat(np.zeros((n * 3, n * 3), float))
    def assignAt(i, j):
        ''' ith object and jth object'''
        Mat3 = GaussianKernelMat3(name + str(i), name + str(j))
        for x in range(0,3):
            for y in range(0,3):
                K[i*3+x, j*3+y]=Mat3[x,y]
        # print(K)
    for i in range(0, n):
        for j in range(0, n):
            assignAt(i, j)
    ''' assert K is symmetrix matrix '''
    assert((K.T==K).all())
    return K
```

获得特征值从大到小排列的 index:

```python
def GetArgSortList(K=0):
    if(type(K)==type(0)):
        K=GetKMat3()
    '''eigh is intended for symmetric matrices'''
    evals, evecs=np.linalg.eigh(K)
    # print(evals)
    big2small_indices = np.argsort(evals).tolist()
    big2small_indices.reverse()
    return big2small_indices
```

积分方程的特征函数 $\Phi_{i}(x)$ 就可以算出来：

```python
def Phi(i, t = "Arrow0", K=0):
    if(type(K)==type(0)):
        global KMatrix
        K = KMatrix
    global KX
    def assignAt(j):
        Mat3 = GaussianKernelMat3("Arrow" + str(j), t)
        for x in range(0, 3):
            for y in range(0, 3):
                KX[x, j * 3 + y] = Mat3[x, y]
    for j in range(0, 25 * 5):
        assignAt(j)

    evals, evecs = np.linalg.eigh(K)
    big2small_indices = GetArgSortList(K)

    u_i = evecs[big2small_indices[i]].T
    eval_i = evals[big2small_indices[i]]

    ''' res should be vector3 '''
    res = np.matmul(KX, u_i)
    res*= (eval_i ** 0.5)
    res = res.flatten()
    res = res.getA()
    '''res is type of numpy.ndarray, and then we normolize it by Phi(0, "Arrow0")'''
    res = res[0]
    global average
    res[0] = res[0]/average
    res[1] = res[1]/average
    res[2] = res[2]/average
    return res
```

我是用三维的箭头模型表示空间中的形变向量，首先生成 5 * 5 * 5 个箭头：

```python
def SpawnArrows(number=5):
    # spawun 5 * 5 arrows
    arrow=bpy.data.objects['Arrow']
    arrow.location = [0, 0, 10]
    for z in range(0, number):
        for y in range(0,number):
            for x in range(0,number):
                newarrow = arrow.copy()
                newarrow.name='Arrow' + str(z * number * number + y * number + x)
                newarrow.location=[x, y, z]
                bpy.data.collections[0].objects.link(newarrow)
                # bpy.context.scene.objects.link(newarrow)
    return
```

然后预览就是通过 `def Point_at(obj, direction)`:

```python
def Point_at(obj, direction):
    if(type(direction)!=Vector):
       direction = Vector(direction)
    if(type(obj)==str):
        obj = bpy.data.collections[0].objects[obj]
    # print('Norm of ' + str(direction) +  ' is :' + str(Norm(direction)))
    obj.scale.x =  Norm(direction) * 0.4
    obj.scale.y =  Norm(direction) * 0.2
    obj.scale.z =  Norm(direction) * 0.3
    # point the obj 'Y' and use its 'Z' as up
    rot_quat = direction.to_track_quat('Y', 'Z')
    
    # assume we're using euler rotation
    obj.rotation_euler = rot_quat.to_euler()
    return
```

如下图所示，用箭头可视化不同 $\sigma$ （高斯核函数的参数）下形变函数的分布：

<center><img src="/assets/arrow.png" alt="公式" width="100%" height="100%" align="center" /></center>

这里每运行一次程序得到的都是每个位置的随机的向量，可以看到随着 $\sigma$ 变大，形变是越来越光滑的，这个是符合预期的。