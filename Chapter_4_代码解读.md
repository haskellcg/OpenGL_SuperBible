# 4.基础变换：初识向量/矩阵
## 4.1 本章时令人生畏的数学课吗

## 4.2 3D图形数学速成课
### 4.2.1 向量
  一个向量首先是空间中从原点指向这个点的方向。

  单位向量/标准化

  math3d库有两个数据类型：
  ```c++
  // M3DVector3f可以表示一个三维向量(X, Y, Z)
  typedef float M3DVector3f[3];

  // M3DVector4f可以表示一个四维向量(X, Y, Z, W)
  typedef float M3DVector4f[4];
  ```

  典型情况下，W坐标设为1.0，X、Y、Z值通过W来进行缩放。
  ```c++
  M3DVector3f vVertor;

  M3DVector4f vVertex = {0.0f, 0.0f, 1.0f, 1.0f};

  M3DVector3f vVerts = {-0.5f, 0.0f, 0.0f,
                        0.5f, 0.0f, 0.0f,
                        0.0f, 0.5f, 0.0f};
  ```

  点乘(dot product)：运算得到一个标量，它表示两个向量之间的夹角。在漫射光计算中，表面法向量和指向光源之间大量进行着这种运算。
  ```c++
  // 计算结果表示这两个单位向量之间的夹角的余弦值
  float m3dDotProduct3(const M3DVector3f u, const M3DVector3f v);

  // 计算结果更近一步，返回这个夹角的弧度值
  float m3dGetAngleBetweenVectors3(const M3DVector3f u, const M3DVector3f v);
  ```

  叉乘(cross product)：运算得到另一个向量，这个向量与原来两个向量定义的平面垂直：
  ```c++
  // 叉乘时向量的顺序时非常重要的
  void m3dCrossProduct3(M3DVector3f result, const M3DVector3f u, const M3DVector3f v);
  ```

### 4.2.2 矩阵
  矩阵(matrix)：大大简化了求解变量之间有复杂关系的方程或方程组的过程，例如坐标变换、旋转。
  ```c++
  // 3 x 3
  typedef float M3DMatrix33f[9];

  // 4 x 4
  typedef float M3DMatrix44f[16];
  ```

  OpenGL约定中拒绝了这个传统并使用了一个一维数组。这样做的原因是，OpenGL使用一种叫做Column-Major(以列为主的)矩阵排序的矩阵约定。

## 4.3 理解变换
  将3D数据被压扁成2D数据的处理过程叫做投影(projection)。

  投影、旋转对象、移动对象、伸展、收缩、扭曲

  变换|应用
  ----|----
  视图|指定观察者或照相机的位置
  模型|在场景中移动物体
  模型视图|描述视图和模型变换的二元性
  投影|改变视景体的大小或重新设置它的形状
  视口|这是一种伪变换，只是对窗口上的最终输出进行缩放

### 4.3.1 视觉坐标
