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
  视觉坐标相对于观察者而言，无论可能进行何种变换，我们都可以将它们视为"绝对的"屏幕坐标。这样，视觉坐标就表示一个虚拟的固定坐标系，它通常作为参考坐标系使用。

  当我们利用OpenGL进行3D绘制时，就会使用笛卡尔坐标系。

### 4.3.2 视图变换
  视图变换是应用到场景中的第一种变换。他用来确定场景中的有利位置。在默认情况下，透视投影中的观察点位于原点(0, 0, 0)，并沿着z轴的负方向进行观察。

  当观察点位于原点时，就像透视投影中一样，绘制在z坐标为正的位置的对象则位于观察者背后。

  在正投影中，观察者被认为是在z轴正方向无穷远的位置，能够看到视景体中的任何东西。

  视图变换允许我们把观察点放在所希望的任何位置，并允许在任何方向观察场景。确定视图变换就像在场景中放置照相机并让它指向某个方向。

  从大局上考虑，视图变换移动了当前的工作坐标系，所有后续变换随后都会基于新调整的坐标系进行，所以在应用任何其他模型变换之前，必须首先应用视图变换。

### 4.3.3 模型变换
  模型变换用于操作模型以及其中特定的对象。这些变换将对象移动到需要的位置，然后再对它们进行旋转和缩放：
  * Translation: 平移
  * Rotation：旋转
  * Scaling: 缩放

  在变换过程中，模型自己的坐标系会随之变换，所以变换的顺序不同会造成不同的结果。

### 4.3.4 模型视图的二元性
  实际上，视图和模型变换按照它们内部效果和对场景的最终外观来说是一样的。

### 4.3.5 投影变换
  投影变换将在模型变换之后应用到顶点上。这种投影实际上定义了视景体并创建了剪裁平面。

  剪裁平面是3D空间中的平面方程式，OpenGL用它来确定几何图形对于观察者来说是否可见。更具体地来说，投影变换指定一个完成的场景时如何映射到屏幕上的最终图像。

  在正投影(或者说平行投影)中，所有多边形都是精确地按照指定的相对大小来在屏幕上绘制的。典型情况下，这种投影用于渲染二维图像，例如蓝图或者文本或屏幕菜单等二维图形。

  透视投影应用在渲染包括广阔的空间或者需要应用投射缩短的物体的场景时使用。在大多数情况下，透视投影都应用在3D图形中。

### 4.3.6 视口变换
  当所有这些讲述完成后，就得到一个场景的二维投影，它被映射到屏幕上某处的窗口上。这种到物理窗口坐标的映射时我们最后要做的变换，成为视口变换。

## 4.4 模型视图矩阵
  模型视图矩阵是一个4x4矩阵，它表示一个表换后的坐标系，我们可以用来放置对象和确认对象的方向。

  我们为图元提供的顶点将作为一个单位矩阵(也就是一个向量)的形式来使用，并乘以一个模型视图矩阵来获得一个相对于视觉坐标系的经过变换的新坐标系。

### 4.4.1 矩阵构造
  OpenGL将矩阵表示为由一个16个浮点数组成的单个数组：
  ```c++
  // Nice OpenGL friendly matrix
  GLfloat matrix[16];

  // Popular, but not as efficient for OpenGL
  GLfloat matrix[4][4];
  ```

  列优先排序，两种形式的矩阵互为转置矩阵。
  ```c++
  GLfloat matrix[16] = {
    Xx, Yx, Zx, Tx,
    Xy, Yy, Zy, Ty,
    Xz, Yz, Zz, Tz,
    0,  0,  0,  0
  };
  ```

  最奇妙的是，如果有一个包含一个不同坐标系的位置和方向的4x4矩阵，然后用一个表示原来坐标系的向量乘以这个矩阵，得到的结果是一个转换到新坐标系下的新向量。

  单位矩阵：本书中使用的的哥着色器就是单位着色器
  ```c++
  GLfloat m[] = {
    1.0f, 0.0f, 0.0f, 0.0f,
    0.0f, 1.0f, 0.0f, 0.0f,
    0.0f, 0.0f, 1.0f, 0.0f,
    0.0f, 0.0f, 0.0f, 1.0f
  };

  M3DMatrix44f m = {
    1.0f, 0.0f, 0.0f, 0.0f,
    0.0f, 1.0f, 0.0f, 0.0f,
    0.0f, 0.0f, 1.0f, 0.0f,
    0.0f, 0.0f, 0.0f, 1.0f
  };

  // math3d库函数，初始化一个空单位矩阵
  void m3dLoadIdentity44(M3DMatrix44f m);
  ```

  平移：
  ```c++
  void m3dTranslationMatrix44(M3DMatrix44f m, float x, float y, float z);
  ```

  旋转：将一个对象沿着3个坐标轴的一个或任意向量进行旋转，需要找到一个旋转矩阵
  ```c++
  // 生成旋转矩阵
  // 围绕(x, y, z)指定的向量进行旋转，旋转的角度沿逆时针方向按照弧度计算
  m3dRotationMatrix44(M3DMatrix44f m, float angle, float x, float y, float z);

  // for example
  M3DMatrix44f m;
  m3dRotationMatrix44(m, m3dDegToRad(45.0), 1.0f, 1.0f, 1.0f);
  ```

  缩放：
  ```c++
  M3DMatrix44f m;
  void m3dScaleMatrix44(M3DMatrix44f m, float xScale, float yScale, float zScale);
  ```

  综合变换：将两种变换加在一起很简单，只需要将两个矩阵相乘。不过在矩阵乘法中有一个小陷阱需要注意，就是运算的顺序是有影响的。
  ```c++
  void m3dMatrixMultiple44(M3DMatrix44f product,
                           const M3DMatrix44f a,
                           const M3DMatrix44f b);
  ```

### 4.4.2 运用模型视图矩阵
  在屏幕上先平移然后旋转正方形：
  ```c++
  void RenderScene(void)
  {
      // Clear the window with current clearing color
      glClear(GL_COLOR_BUFFER_BIT |
              GL_DEPTH_BUFFER_BIT |
              GL_STENCIL_BUFFER_BIT);

      GLfloat vRed[] = {1.0f, 0.0f, 0.0f, 1.0f};

      M3DMatrix44f mFinalTransform, mTranslationMatrix, mRotationMatrix;

      // Just Translate
      m3dTranslationMatrix44(mTranslationMatrix, xPos, yPos, 0.0f);

      // Rotate 5 degrees every tine we redraw
      static float yRot = 0.0f;
      yRot += 5.0f;
      m3dRotationMatrix44(mRotationMatrix, m3dDegToRad(yRot), 0.0f, 0.0f, 1.0f);

      m3dMatrixMultiple44(mFinalTransform, mTranslationMatrix, mRotationMatrix);

      shaderManager.UseStockShader(GLT_SHADER_FLAT, mFinalTransform, vRed);
      squareBatch.Draw();

      // Perform the buffer swap
      glutSwapBuffers();
  }
  ```

  然而，这个简单的坐标系并不是总能满足我们的需要，而且在更大的坐标空间中考虑我们的对象会更加方便，需要一种类型的矩阵变换，成为投影。

## 4.5 更多对象
  GLTriangleBatch类，这个类是专门作为三角形的容器，每个顶点都可以有一个表面法线，已进行光照计算和纹理坐标，它将三角形以更加高效的方式(索引定点数组)进行组织，而实际上将多边形存储在图形卡(使用顶点缓冲对象)上就够了。

### 4.5.1 使用三角形批次类
  ```c++
  GLTriangleBatch myCoolObject;

  // 最多使用顶点数目
  myCoolObject.BeginMesh(200);

  // 3顶点数组、3法线数组、3纹理坐标数组
  // 不用担心出现重复的顶点数组，GLTriangleBatch会搜索重复值并对我们的批次
  // 进行优化，所以，对于大的批次，这种方式添加速度会越来越低
  myCoolObject.End();

  // Draw
  myCoolObject.Draw();
  ```

### 4.5.2 创建一个球体
  ```c++
  // iStacks：表示从球底堆叠到顶部的三角形带的数量
  // iSlices：表示围绕球体排列的三角形对数
  // 典型情况下一个对称性较好的球体的片段数目是堆叠数目的2倍
  // 另外，这些球体都是围绕z轴的，这样+z就是球体的顶部，-z就是球体的底部
  void gltMakeSphere(GLTriangleBatch &sphereBatch,
                     GLfloat fRadius,
                     GLint iSlices,
                     GLint iStacks);
  ```

### 4.5.3 创建一个花托
  ```c++
  // numMajor/numMinor：沿着主半径和内部较小半径的细分单元的数量
  void gltMakeTorus(GLTriangleBatch &torusBatch,
                    GLfloat majorRadius,
                    GLfloat minorRadius,
                    GLint numMajor,
                    GLint numMinor);
  ```

### 4.5.4 创建一个圆柱或圆锥
  ```c++
  // fLength：应该是高度
  void gltMakeCylinder(GLTriangleBatch &cylinderBatch,
                       GLfloat baseRadius,
                       GLfloat topRadius,
                       GLfloat fLength,
                       GLint numSlices,
                       GLint numStacks);
  ```
  理解：基本几何图形需要的参数都会以GLfloat的参数给出，GLint指出的三角形带的数目只是为了提高三角形的密度。

### 4.5.5 创建一个圆盘
  ```c++
  void gltMakeDisk(GLTriangleBatch &diskBatch,
                   GLfloat innerRadius,
                   GLfloat outerRadius,
                   GLint nSlices,
                   GLint nStacks);
  ```

## 4.6 投影矩阵
  到目前为止，我们已经在屏幕或窗口上使用了范围-1到+1的默认坐标系。事实上，这个小小的坐标范围确实是硬件唯一能够接受的。

  那么使用不同坐标系的技巧就是，将我们想要的坐标系变换到这个单位立方体中。使用投影矩阵。

### 4.6.1 正投影
  ```c++
  GLFrustum::SetOrthographic(GLfloat xMin, GLfloat xMax,
                             GLfloat yMin, GLfloat yMax,
                             GLfloat zMin, GLfloat zMax);
  ```

### 4.6.2 透视投影
  ```c++
  GLFrustum::SetPerspective(float fFov,
                            float fAspect,
                            float fNear,
                            float fFar);
  ```

### 4.6.3 模型视图矩阵
  ModelviewProject

  使用CStopWatch类来基于经过的时间长短设置旋转速度，我们应该根据经过的时间设置动画率，而不是单纯的基于帧的方式：
  ```c++
  static CStopWatch rotTimer;
  float yRot = rotTimer.GetElapsdSeconds() * 60.0f;
  ```

  添加模型视图、投影矩阵：
  ```c++
  viewFrustum.SetPerspective(35.0f, float(w) / flloat(h), 1.0f, 1000.0f);

  m3dMatrixMultiple44(mModelview, mTranslate, mRotate);
  m3dMatrixMultiple44(mModelViewProjection,
                      viewFrustum.GetProjectionMatrix(),
                      mModelview);

  shaderManager.UseStockShader(GLT_SHADER_FLAT, mModelViewProjection, vBlack);
  torusBatch.Draw();
  ```

## 4.7 变换管线
  初始顶点 => [模型视图矩阵] => 变换的视觉坐标 => [投影矩阵] =>
  剪裁坐标 => [透视除法] => 规范化的设备坐标 => [顶点变换管线...] =>
  [视口变换] => 窗口坐标

### 4.7.1 使用矩阵堆栈
  在分层方式中，一个或多个对象会相对于另一个对象进行绘制，在这种方式中经常会应用到变换。这样就需要大量有用户代码进行构造和管理的矩阵在3D空间中建立复杂场景。

  习惯上，我们使用矩阵堆栈。GLMatrixStack类，这个类的构造函数允许指定堆栈的最大深度，默认的堆栈深度为64。这个矩阵堆栈在初始化时已经在堆栈中包含了单位矩阵。
  ```c++
  GLMatrixStack::GLMatrixStack(int iStackDepth = 64);

  void GLMatrixStack::LoadIdentity(void);

  void GLMatrixStack::LoadMatrix(const M3DMatrix44f m);

  // 用一个矩阵乘以矩阵堆栈的顶部矩阵，相乘得到的结果随后存储在堆栈顶部
  void GLMatrixStack::MultMatrix(const M3DMatrix44f m);

  // 获取矩阵堆栈顶部的值
  const M3DMatrix44f &GLMatrixStack::GetMatrix(void);
  void GLMatrixStack::GetMatrix(M3DMatrix44f mMatrix);


  // Pop/Push
  void GLMatrixStack::PushMatrix(void);
  void GLMatrixStack::PushMatrix(const M3DMatrix44f mMatrix);
  void GLMatrixStack::PushMatrix(GLFrame &frame);

  void GLMatrixStack::PopMatrix(void);


  // 放射变换，这些函数可以创建适当的矩阵，然后用这个矩阵乘以矩阵堆栈顶部的元素
  void GLMatrixStack::Rotate(GLfloat angle, GLfloat x, GLfloat y, GLfloat z);
  void GLMatrixStack::Translate(GLfloat x, GLfloat y, GLfloat z);
  void GLMatrixStack::Scale(GLfloat x, GLfloat y, GLfloat z);
  ```

### 4.7.2 管理管线
  为模型视图矩阵和投影矩阵都建立一个矩阵堆栈会有很多优势，我们还经常需要检索这两种矩阵并将它们相乘得到模型视图投影矩阵。另一种有用的矩阵就是正规矩阵，它用来进行光照计算，并且可以从模型视图矩阵推到出来。

  另一个实用类GLGeometryTransform为我们跟踪记录这两种矩阵堆栈，并快速检索模型视图投影矩阵的顶部或者正规矩阵堆栈的顶部。

  SphereWorld:
  ```c++
  GLMatrixStack modelViewMatrix;
  GLMatrixStack projectionMatrix;
  GLFrustum viewFrustum;
  GLGeometryTransform transformPipeline;

  viewFrustum.SetPerspective(35.0f, float(nWidth) / float(nHeight), 1.0f, 1000.0f);
  projectionMatrix.LoadMatrix(viewFrustum.GetProjectionMatrix());

  // 将transformPipeline的内部指针指向两个矩阵堆栈
  transformPipeline.SetMatrixStacks(modelViewMatrix, projectionMatrix);

  modelViewMatrix.PushMatrix();
  shaderManager.UseStockShader(GLT_SHADER_FLAT,
                               transformPipeline.GetModelViewProjectionMatrix(),
                               vFloorColor);
  floorBatch.Draw();

  modelViewMatrix.Translate(0.0f, 0.0f, -2.5f);
  modelViewMatrix.Rotate(yRot, 0.0f, 1.0f, 0.0f);
  shaderManager.UseStockShader(GLT_SHADER_FLAT,
                               transformPipeline.GetModelViewProjectionMatrix(),
                               vTorusColor);
  torusBatch.Draw();

  modelViewMatrix.PopMatrix();
  ```

### 4.7.3 加点调料
  添加蓝色球体，以花托为中心旋转
  ```c++
  ...modelViewMatrix.PopMatrix();

  modelViewMatrix.Rotate(yRot * -2.0f, 0.0f, 1.0f, 0.0f);
  modelViewMatrix.Translate(0.8f, 0.0f, 0.0f);
  shaderManager.UseStockShader(GLT_SHADER_FLAT,
                               transformPipeline.GetModelViewProjectionMatrix(),
                               vSphereColor);
  sphereBatch.Draw();
  modelViewMatrix.PopMatrix();
  ```

## 4.8 使用照相机和角色进行移动
