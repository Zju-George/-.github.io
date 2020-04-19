---
layout: post
title: "Adaptive Subdivision"
---
本文档介绍在面向三维打印的曲面雕刻中，自适应细分算法扮演的角色。
<br>

# <center>细分的原因</center>

**雕刻操作**依赖于曲面参数化，将二维图像映射到三维曲面上。对曲面上每个点，**采样**(双线性插值采样)得到对应的像素值，根据像素值决定是否进行几何操作。eg.，利用二值图像，黑色部分试图保留，白色部分试图镂空，那么曲面上的点采样值小于 0.5 的将会被保留，而大于 0.5 的将会被删去，这样就实现了镂空操作。如下图人脸镂空：

<center><img src="/assets/face_hollow2.jpg" alt="人脸镂空" width="80%" height="80%" align="center" /></center>

而**细分**是为了雕刻操作前增加模型几何信息，使得雕刻操作后雕刻边缘的锯齿状变小的一个重要操作。


---

# <center>自适应细分的原因</center>

在这里，细分的最基本规则被定义为让一个三角形一分为四。如下图：

<center><img src="/assets/subdivide.png" alt="细分规则" width="80%" height="80%" align="center" /></center>

所谓**全局细分**就是无差别的对每一个三角形，一分为四。然而全局细分会带来的一个显著问题是：有些三角形的三个顶点采样值都大于 0.5 或者都小于 0.5，也即这个三角形三个顶点都要保留或都要删去，
更直白的说，这个三角形所在位置不在雕刻边缘，所以不需要继续细分；因此全局细分增加了很多~~无用~~没有意义的几何信息。这会使得程序速度变慢，也增加模型的保存大小。

**自适应细分**，也可以叫**局部细分**，目标就是只细分那些在雕刻边缘附近的三角形。

- ### 雕刻边缘判定的转化
判断三角形三个顶点采样值有大于 0.5 也有小于 0.5 的 $\Longleftrightarrow$ 对图像用Canny算子提取边缘，判断参数三角形（2D）包含边缘像素。

- ### 判断三角形是否包含像素
采取面积法判断平面上一点是否落在三角形内。将一点连接三角形各顶点得到
三个三角形，若三个三角形的面积之和等于原三角形，判定该点落在三角形内

---

# 算法优化
考虑细分迭代一次，如果三角形个数是 $n$，边缘像素的个数是 $m$，用纯遍历的方法，算法复杂度为$\mathcal{O}(mn)$。然而这个是可以被优化的。

- ### 边缘像素的优化
  在判断一个三角形是否包含边缘像素时，无须遍历所有边缘像素，而只需要遍历一个矩形框AABB内的像素。所谓AABB，就是包含这个三角形的最小的矩形。如下图红色矩形框就是绿色三角形的AABB:
  <center><img src="/assets/AABB.png" alt="AABB" width="60%" height="60%"/></center>
  对应的 ***AABB*** 代码段(python)：
  ```python
  def AABB(face):
      # clamp is not considered! 
      uv_lay = bm.loops.layers.uv.active
      x0 = face.loops[0][uv_lay].uv.x
      y0 = face.loops[0][uv_lay].uv.y
      x1 = face.loops[1][uv_lay].uv.x
      y1 = face.loops[1][uv_lay].uv.y
      x2 = face.loops[2][uv_lay].uv.x
      y2 = face.loops[2][uv_lay].uv.y
      xmin = min(x0,x1,x2)*shape[1]
      xmax = max(x0,x1,x2)*shape[1]
      ymin = min(y0,y1,y2)*shape[0]
      ymax = max(y0,y1,y2)*shape[0]
      return floor(xmin),floor(ymin),ceil(xmax),ceil(ymax)
  ```
  对应的 ***是否在三角形内*** 代码段(python)：

  ```python
  def ispixIn(pix,face):
      if(area_pix(pix,face)>(area_tri(face)+0.000000001)):
          return False
      else:
          return True
  ```
  注意在 Blender 中要访问顶点的 uv 需要通过 loop，loop 和 vertex 的不同是 loop 是 **per face** 的，具体可以 Google 一下定义与用法。
- ### 三角形个数的优化
  因细分后的四个三角形均落在原三角形内，故落在“子”三角形内的边缘点必然也落在“父”三角形内。}如果记录每个参数三角形所包含的边缘点，则每次细分后判断“子”三角形是否包含边缘点，只需要遍历“父”三角形包含的边缘点，判断包含关系，进而更新“子”三角形所包含的边缘点。这个可以用一个字典来维护。

  具体的算法流程图：
  <left><img src="/assets/algo.png" alt="算法" width="60%" height="60%" align="center" /></left>

  对应的完整代码：
  ```python
  # Get the init dic_f2p
  obj = bpy.context.edit_object
  me = obj.data
  bm = bmesh.from_edit_mesh(me)
  bm.faces.ensure_lookup_table()
  dic_f2p={}
  initfaces = []
  for f in bm.faces:
      if(f.select==True):
          initfaces.append(f)
          
  for i in range(len(initfaces)):
      face = initfaces[i]
      temp_pix = []

      xmin,ymin,xmax,ymax = AABB(face)
      for m in range(shape[0]-ymax,shape[0]-ymin+1):
          for n in range(xmin,xmax+1):
              if(img[m][n]==255):
                  pix = [(n+0.5)/shape[1],(shape[0]-0.5-m)/shape[0]]
                  if(ispixIn(pix,face)):
                      temp_pix.append(pix)
      if(len(temp_pix)>0): # face.select == True
          dic_f2p[face.index]=temp_pix     

  for turn in range(3):
      print("--------------------------------")
      print("turn : "+str(turn))
      bm.select_flush(1)
      selected_set=selected_face(bm)
      dic_set=set(dic_f2p.keys())
      diff = selected_set-dic_set
      if(len(diff)>0):
          print("len diff: "+str(len(diff)))
      else:
          print("len diff: 0")
      for f_index in diff:
          face = bm.faces[f_index]
          temp=[]
          xmin,ymin,xmax,ymax = AABB(face)
          for m in range(shape[0]-ymax,shape[0]-ymin+1):
              for n in range(xmin,xmax+1):
                  if(img[m][n]==255):
                      pix = [(n+0.5)/shape[1],(shape[0]-0.5-m)/shape[0]]
                      if(ispixIn(pix,face)):
                          temp.append(pix)
          dic_f2p[f_index]=temp 
      dic_f2p_sorted=sorted(dic_f2p, reverse=True)
      edge_list =[]
      for f in dic_f2p.keys():
          edge_list.extend(bm.faces[f].edges[:])       
      edge_list = list(set(edge_list))
      print("dic_f2p items num: " + str(len(dic_f2p)))
      bpy.ops.ed.undo_push(message="Subdivide")
      ret = bmesh.ops.subdivide_edges(bm, edges=edge_list, cuts=1,use_grid_fill=True)
      # adding triangulate
      bmesh.ops.triangulate(bm, faces=bm.faces)
      bm.faces.ensure_lookup_table()
      counter=-1
      dic_f2p_next={}
      facenum=0
      for face in ret['geom_inner']:
          if isinstance(face, bmesh.types.BMFace):
              facenum+=1
      if(facenum//3>len(dic_f2p)):
          print('wrong!')
          print(facenum//3)

          bmesh.update_edit_mesh(me, True)
          bpy.ops.ed.undo_push(message="Subdivide")
      for f in ret['geom_inner']:
          if isinstance(f, bmesh.types.BMFace):
              counter+=1
              index=counter//3
              try:
                  f_mother=bm.faces[dic_f2p_sorted[index]]
              except:
                  import os
                  print(index)
                  os.system("pause")
              pixels = dic_f2p[dic_f2p_sorted[index]]
              temp_pix = []
              if(counter%3==0):
                  # print("pixels of index-- "+str(index)+" : " +str(len(pixels)))
                  temp = []
                  for j in range(len(pixels)):
                      pix = pixels[j]
                      if(ispixIn(pix,f_mother)):
                          temp.append(pix)                 
                  if(len(temp)>0):
                      # print("f_mother pix num: " + str(len(temp)))
                      f_mother.select = True
                      dic_f2p_next[f_mother.index] = temp
                  else:
                      f_mother.select = False
                      pass                
              for j in range(len(pixels)):
                  pix = pixels[j]
                  if(ispixIn(pix,f)):
                      temp_pix.append(pix)           
              if(len(temp_pix)>0): # face.select == True
                  # print("f pix num: "+ str(len(temp_pix)))
                  dic_f2p_next[f.index]=temp_pix
                  f.select = True
              pass        
      dic_f2p=dic_f2p_next
      del dic_f2p_next
  ```
---

## 工程文件

可以下载这个工程文件 [subdivide_test.blend]({{ 'https://github.com/Zju-George/blenderStatisticalModeling/blob/master/subdivide_test.blend?raw=true' | absolute_url }}). (Blender 版本>=2.80)

里面会包含完整代码 `subdivide.py`，但要注意把第 `7` 行的测试二值图像换成自己的。点 `Run Script` 即可。:thumbsup:

## 全局细分与自适应细分对比(待更新)

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |



