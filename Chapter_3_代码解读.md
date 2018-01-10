## 客户机-服务器
  OpenGL的客户端是存储在CPU存储器中的，并且**在应用程序中执行**，或者在主系统内存中的驱动程序中执行。驱动程序将渲染命令与数据结合起来。
  
  在一台典型的桌面计算机上，服务器会跨越一些系统总线，实际上，它就是**图形加速卡的硬件和内存**。
  
  客户机和服务器在功能上也是**异步**的，也就是说它们有各自独立的软件块和硬件块，或者软硬件都有。为获得最佳性能，我们希望两方面都尽可能不停工作。客户机不断地将**数据块**和**命令块**组合在一起并送入**缓冲区**，然后这些缓冲区会发送给服务器执行。服务器将执行这些缓冲区内的内容，与此同时客户端又做好发送一下一个用于渲染的数据和信息的准备。
  
  如果服务器停止工作等待客户机，或者客户机停止工作等待服务器做好接收的准备，这种情况叫做**管线停滞**。管线停滞是注重性能程序员的噩梦，我们真的不希望CPU或者GPU无所事事等待工作。
  
## 着色器
  顶点着色器、片段着色器，使用GLSL编写，这些着色器必须从源码中编译链接在一起，最终准备就绪的着色器程序在随后第一阶段构成顶点着色器、第二阶段构成片段着色器。
  
  **顶点着色器**处理从客户端输入的数据，应用变换，或者进行其他类型的数学运算来计算光照效果、位移、颜色值。
  
  每个片段都通过**片段着色器**而进行了填充，片段着色器会输出我们将在屏幕上看到的最终颜色值。
  
  我们必须为这些着色器提供数据:
  
  * **属性**: 一个对每个顶点都要做改变的元素，属性总是以四维向量的形式进行内部存储。除了顶点的空间位置之外，还有一些其他可能要逐个修改的属性，包括**纹理坐标、颜色值和用于光照计算的表面法线**，不过，在顶点程序中，属性可以代表我们想要任何意义。属性会从**本地客户机内存中**复制存储在**图形硬件中**的一个缓冲区上，这些属性只供顶点着色器使用。
  
  * **Uniform值**: 我们通常设置哇Uniform变量就发出渲染一个图元批次的命令，Uniform变量实际上可以无限制次数使用。Uniform变量最常见的应用在顶点渲染中设置**变换矩阵**。顶点着色器和片段着色器都可以有Uniform变量。
  
  * **纹理**: 从顶点着色器到片段着色器都可以对纹理直进行采样和筛选，典型情况下，片段着色器对一个纹理进行采样，并在一个三角形的表面应用图形数据。但是，纹理数据的作用不仅仅是**图形表现**，很多**图形文件格式都是以无符号字节形式对颜色分量进行存储**。
  
  * **输出**:输出数据是作为一个阶段着色器的输出定义的，而在后续阶段的着色器则是作为输入定义的。输出类型数据可以简单的从一个阶段传递到下一个阶段，也可以以不同的方式插入。
  
## 创建坐标系
  这些投影，或者说坐标系类型，实际上只是一种**特定的4x4变换矩阵**，如果我们不采用这些矩阵，则会**默认获取一个坐标范围在-1.0到+1.0之间的正投影**。
  
  **Math3D**库是GLTools包含的函数的一部分，它为我们构建了不同种类的矩阵。在本章，我们使用**GLFrustum**类来作为投影矩阵的容器。
  
  * **正投影**: 我们通常在2d绘图中使用正投影，并在我们的几何图形中将z坐标设为0.0。**视景体**将包含所有的几何图形，如何指定了视景体之外的几何图形，那么它将被裁减掉。在正投影中，所有在这个空间范围内的所有东西都会被显示在屏幕上，而不存在**照相机**或**视点坐标系**的概念，我们通过调用下面函数完成上述工作：
  ```c++
  GLFrustum::SetOrthographic(GLfloat xMin, GLfloat xMax, 
                             GLfloat yMin, GLfloat yMax, 
                             GLfloat zMin, GLfloat zMax);
  ```

  * **透视投影**: 透视投影会进行**透视除法**对距离观察者很远的对象进行缩短和收缩。在投影到屏幕之后，视景体背面和视景体正面的宽度测量标准不同。这样逻辑尺寸相同的对象绘制在视景体的前面比绘制在视景体的背面显得更大。我们通过下面函数创建**平头截体(frustum)**:
  ```c++
  /**
   *@brief create frustum
   *@param float fFov, 垂直方向上的视场角度
   *@param float fAspect, 窗口的宽度和高度的纵横比
   *@param float fNear, 近裁截面的距离
   *@param float fFar, 远裁截面的距离
   */
  GLFrustum::SetPerspective(float fFov, float fAspect, float fNear, float fFar);
  ```

## 使用存储着色器
  在OpenGL核心框架中，并没有提供任何内建渲染管线。在提交一个几何图形进行渲染之前，必须制定一个着色器。
  
  这些着色器由GLTools的C++类GLShaderManager进行管理，它们能够满足进行通常渲染基本要求。在使用前必须进行初始化:
  ```c++
  shaderManager.InitializeStockShaders();
  ```
  
  * 属性: OpenGL支持多达16种可以为每个顶点设置不同的类型参数，这些参数编号从0到15，并且可以与**顶点着色器**中的任何指定变量相关联。**存储着色器**为每个变量都使用一致的内部变量命名规则和相同的属性槽  
  
  标识符|描述
  -----|----
  GLT\_ATTRIBUTE\_VERTEX|3分量(x, y, z)顶点位置
  GLT\_ATTRIBUTE\_COLOR|4分量(r, g, b, a)颜色值
  GLT\_ATTRIBUTE\_NORMAL|3分量(x, y, z)表面法线
  GLT\_ATTRIBUTE\_TEXTURE0|第一对2分量(s, t)纹理坐标
  GLT\_ATTRIBUTE\_TEXTURE1|第二对2分量(s, t)纹理坐标
  
  * Uniform值
    要对几何图形进行渲染，我们需要为对象递交**属性矩阵**，但是首先要绑定到我们想要使用的**着色器程序**，并提供程序的**Uniform值**。通过函数:
    ```c++
    GLShaderManager::UseStockShader(GLenum shader, ......);
    ```
    
    * 单位(Identity)着色器: 简单地使用默认的笛卡尔坐标系(-1.0 ~ 1.0)，所有片段都应用同一颜色，几何图形为实心且未渲染。这种着色器只使用**GLT_ATTRIBUTE_VERTEX**，vColor参数包含了颜色:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_IDENTITY, GLfloat vColor[4]);
    ```
    
    * 平面(Flat)着色器: 将统一着色器进行扩展，允许为几何图形变**换指定一个4x4的变换矩阵**，典型情况，这是一种**左乘模型视图矩阵和投影矩阵**，经常被称为“**模型视图投影矩阵**”。这种着色器只使用一个属性**GLT_ATTRIBUTE_VERTEX**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_FLAT, Glfloat mvp[16], Glfloat vColor);
    ```
    
    * 上色(Shaded)着色器: 提供一个变换矩阵，颜色值将被**平滑的插入到顶点之间**，使用两个属性**GLT_ATTRIBUTE_VERTEX**，**GLT_ATTRIBUTE_COLOR**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_SHADED, GLfloat mvp[16]);
    ```
    
    * 默认光源着色器: 这种着色器创造一种错觉，类似于由位于观察者位置的单漫射光。从本质上说，这种着色器使对象产生阴影和光照的效果。这里需要**模型视图矩阵**、**投影矩阵**和作为基本色的**颜色值**等Uniform值，所需要的属性**GLT_ATTRIBUTE_VERTEX**, **GLT_ATTRIBUTE_NORMAL**。大多数光照着色器需要**正规矩阵**作为Uniform值，着色器从模型视图矩阵中推到出了正规矩阵，但是效率不高:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_DEFAULT_LIGHT, GLfloat mvMatrix[16],
                                                              GLfloat pMatrix[16], 
                                                              GLfloat vColor[4]);
    ```
    
    * 点光源着色器: 光源位置是固定的，接受4个Uniform值**模型视图矩阵**, **投影矩阵**, **视点坐标系中的光源位置**, **对象的基本漫反射颜色**，所需要的属性有**GLT_ATTRIBUTE_VERTEX**， **GLT_ATTRIBUTE_NORMAL**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_POINT_LIGHT_DIFF, GLfloat mvMatrix[16], 
                                                                 GLfloat pMatrix[16], 
                                                                 GLfloat vLightPos[3],
                                                                 GLfloat vColor[4]);
    ```
    
    * 纹理替换矩阵着色器: 着色器通过给定的**模型视图矩阵**，使用绑定到**nTextUnit指定的纹理单元**的纹理对几何图形进行变换。**片段颜色**是直接从纹理样本中获取，所需要的属性有**GLT_ATTRIBUTE_VERTEX**，**GLT_ATTRIBUTE_NORMAL**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_TEXTURE_REPLACE, GLfloat mvpMaxtrix[16],
                                                                GLint nTextureUnit);
    ```
    
    * 纹理调整着色器: 着色器将一个**基本色**乘以一个取自**纹理单元nTextUnit的纹理**。所需要的属性**GLT_ATTRIBUTE_VERTEX**，**GLT_ATTRIBUTE_TEXTURE0**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_TEXTURE_MODULATE, GLfloat mvpMatrix[16],
                                                                 GLfloat vColor[4],
                                                                 GLint nTextureUnit);
    ```
    
    * 纹理光源着色器: 这种着色器将一个纹理通过**漫反射照明计算进行调整**，光线在视觉空间中的位置是给定的。接受的Uniform值**模型视图矩阵**，**投影矩阵**，**视觉空间中的光源位置**，**几何图形的基本色**，**将要使用的纹理单元**。需要的属性**GLT_ATTRIBUTE_VERTEX**，**GLT_ATTRIBUTE_NORMAL**，**GLT_ATTRIBUTE_TEXTURE0**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_TEXTURE_POINT_LIGHT_DIFF, GLfloat mvMatrix[16],
                                                                         GLfloat pMatrix[16],
                                                                         GLfloat vLightPos[3],
                                                                         GLfloat vBaseColor[4],
                                                                         GLint nTextureUnit);
    ```

## 将点连接起来
  我们关心的不是物理屏幕的坐标像素，而是视景体中的**位置坐标**。将这些点、线、三角形从创建的**3D空间**投影到计算机**屏幕上的2D图形**则是**着色器程序**和**光栅化硬件**所要完成的工作。

#### 点和线
OpenGL几何图元

图元|描述
---|----
GL\_POINTS|每个顶点在屏幕上都是一个单独的点
GL\_LINES|每对对点对应一个线段
GL\_LINE\_STRIP|一个从第一个顶点依次经过每个后续顶点而绘制的线条
GL\_LINE\_LOOP|和GL\_LINE\_STRIP一样，但是最后一个顶点和第一个顶点也连接起来
GL\_TRIANGLES|每三个顶点定义一个新的三角形
GL\_TRIANGLES\_STRIP|共用一个条带上的顶点的一组三角形

  * **点**: 指定一个允许范围之外的点大小并不会被认为是一个错误，相反，这种情况下将根据哪个值离指定值最近来使用所允许的最大值或者最小值。
  ```c++
  /**
   *使能程序点大小模式
   */
  glEnable(GL_PROGRAM_POINT_SIZE);
    
  /**
   *@brief 设置默认点的大小
   */
  void glPointSize(GLfloat size);
  
  /**
   *@brief 获取点的大小范围，以及它们之间的最小间隔
   */
  GLfloat sizes[2];
  GLfloat step;
  glGetfloatv(GL_POINT_SIZE_RANGE, sizes);
  glGetfloatv(GL_POINT_SIZE_GRANULARITY, &step);
  
  /**
   *@brief 着色器内建变量
   */
  gl_PointSize = 5.0;
  ```
  
  * **线**:
  ```c++
  /**
   *@brief 设置线段宽度
   */
  void glLineWidth(GLfloat width);
  ```
  
  * **线代**:
  
  * **线环**:

#### 三角形

  为了绘制实体表面，我们需要的不仅仅是点和线，还需要**多边形**，多边形是一个封闭的图形，它可以用**颜色**或者**纹理数据**进行填充，也可能不进行填充，在OpenGL中，**它是所有实体对象构建的基础**。
   
  可能存在的最简单的多边形就是**三角形**，光栅化硬件最欢迎三角形，而三角形已经是OpenGL中支持的**唯一一种多边形了**。

  **环绕**： 这种顺序和方向结合起来指定顶点的方式称为环绕。
  
  在默认情况下，OpenGL认为具有**逆时针方向环绕的多边形是正面**，我们们常常希望为一个多边形的正面和背面分别设置不同的物理特征。
  
  ```c++
  /**
   *@brief 改变OpenGL默认行为，设置默认正面的环绕方向
   */
  glFront(GL_CW);
  glFront(GL_CCW);
  ```
  
  * 三角形带: GL\_TRIANGLES\_STRIP, **按照第一个三角形的环绕顺序进行后面三角形的绘制**，需要绘制大量三角形时，采用这种方式可以节省大量程序代码和数据空间，而且可以提高运算性能和节省宽带
  * 三角形扇: GL\_TRIANGLE\_FAN
  
#### 批次容器  
  GLTools库中包含一个简单的容器类，叫做GBatch，它可以作为7种图元简单批次的容器使用，而且它还知道在使用GLShaderManager支持的任意存储着色器时如何对图元进行渲染
  ```c++
  void GLBatch::Begin(GLenum primitive, GLint nVerts, GLuint nTextureUnits = 0);
  
  // 复制
  void GLBatch::CopyVertexData3f(GLfloat *vVerts);
  
  // 复制表面法线、颜色、纹理坐标
  void GLBatch::CopyNormalDataf(GLfloat *vNorms);
  void GLBatch::CopyColorData4f(GLfloat *vColors);
  void GLBatch::CopyTexCoordData2f(GLfloat *vTexCoords, GLuint uiTextureLayer);
  
  // End表明完成数据的复制
  void GLBatch::End(void);
  ```
  
#### 不希望出现的几何图形  
  在默认情况下，我们所渲染的每一个点、线、三角形都会在屏幕上进行光栅化，并且按照在组合图元批次时指定的顺序进行排列，这在某些情况下会产生问题。
  * 油画法(painters algorithm)，对这些三角形进行排序
  * 背面剔除
  ```c++
  glEnable(GL_CULL_FACE);
  glDisable(GL_CULL_FACE);

  // 指明剔除正面还是背面
  // mode: GL_FRONT/GL_BACK/GL_FRONT_AND_BACK
  glCullFace(GLenum mode);
  ```

#### 深度测试
  深度测试时另外一种高效消除隐藏表面的技术。它的概念很简单：在绘制一个像素时，将一个值(称为z值)分配给它，这个值表示它到观察者的距离。我们在使用GLUT设置OpenGL窗口时，应该请求一个深度缓冲区。
  ```c++
  // 设置深度缓冲区
  glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA | GLUT_DEPTH);

  // 如果没有深度缓冲区，那么启动深度测试命令将被忽略
  glEnable(GL_DEPTH_TEST);
  ```
  就算背面剔除能够消除对象背面的三角形，那么如果时重叠的独立对象又该怎么办？采用油画法，会导致在同一片区域重复进行绘制，而每一次绘制都会产生性能开销。如果开销过大则导致光栅化过程变慢，我们将这种方式成为"填充受限"。但是将油画法颠倒过来，实际上会加速填充性能，这样可以利用深度测试消除那些已存在的像素。

#### 多边形模式
  默认情况下，多边形是作为实行图形绘制的，但我们可以通过将多边形指定为显示轮廓或只有点(只显示顶点)来改变这种行为。
  ```c++
  // face: GL_FRONT/GL_BACK/GL_FRONT_AND_BACK
  // mode: GL_FILL/GL_LINE/GL_POINT
  void glPolygonMode(GLenum face, GLenum mode);
  ```

#### 多边形偏移
  虽然深度测试能够实现真实视觉并提高性能，但有时也会带来一些小麻烦，我们可能需要稍微蒙骗它一下，这种情况发生在有意将两个图形绘制到同一位置。例如贴花(decaling)，这种情况成为z-fighting(z冲突)。

  可以通过第二次绘制时在z方向稍微做一点偏移来解决问题，但我们必须小心确保只能沿着z轴向镜头方向移动。

  ```c++
  // Depth_Offset = (DZ * factor) + (r * units)
  // DZ: 深度值相对于多边形屏幕区域的变化量
  // r: 使深度缓冲区值产生变化的最小值
  void glPolygonOffset(GLfloat factor, GLfloat units);
  ```

  除了使用glPolygonOffset设置偏移之外，还必须启用多边形单独偏移来填充几何图形(GL\_POLYGON\_OFFSET\_FILL)、线(GL\_POLYGON\_OFFSET\_LINE)、点(GL\_POLYGON\_OFFSET\_POINT)。
  
#### 剪裁
  另外一种提高渲染性能的方法时只刷新屏幕上变化的部分，我们可能还需要将OpenGL渲染限制在窗口中一个较小的矩形中。默认情况下，剪裁框与窗口同样大小，并且不会进行剪裁测试。
  ```c++
  // 开启剪裁测试
  glEnable(GL_SCISSOR_TEST);

  // 指定剪裁窗口
  glScissor(GLint x, GLint y, GLsizei width, GLsizei height);
  ```
  
## 混合
  我们已经了解，通常情况下OpenGL渲染时把颜色值放在颜色缓冲区。每个片段的深度值也是放在深度缓冲区中。当深度测试被关闭，新的颜色值简单地覆盖颜色缓冲区中已经存在的其他值。当深度测试打开时，新的颜色片段只有当它们比原来的值更接近临近的裁减平面才会替换原来的颜色片段。

  如果打开了混合功能，那么下层的颜色值就不会清除
  ```c++
  glEnable(GL_BLEND);
  ```
  
#### 组合颜色
  已经存储在颜色缓冲区中的颜色值叫做目标颜色值，这个颜色值包含了RGB以及一个可选的alpha值。

  作为当前渲染命令的结果进入颜色缓冲区的颜色值称为源颜色。

  当混合功能启用时，源颜色和目标颜色的组合方式是由混合方程式控制的：
  ```c++
  Cf = (Cs * S) + (Cd * D)
  Cf: 最终计算产生的颜色
  Cs: 源颜色
  Cd: 目标颜色
  S/D: 源或目标混合因子

  glBlendFunc(GLenum S, GLenum D);
  ```

  S/D都是枚举值：

  函数|RGB混合因子|Alpha混合因子
  ----|-----------|-------------
  GL\_ZERO|(0, 0, 0)|0
  GL\_ONE|(1, 1, 1)|1
  GL\_SRC\_COLOR|(Rs, Gs, Bs)|As
  GL\_ONE\_MINUS\_SRC\_COLOR|(1, 1, 1) - (Rs, Gs, Bs)|1 - As
  GL\_DST\_COLOR|(Rd, Gd, Bd)|Ad
  GL\_ONE\_MINUS\_DST\_COLOR|(1, 1, 1) - (Rd, Gd, Bd)|1 - Ad
  GL\_SRC\_ALPHA|(As, As, As)|As
  GL\_ONE\_MINUS\_SRC\_ALPHA|(1, 1, 1) - (As, As, As)|1 - As
  GL\_DST\_ALPHA|(Ad, Ad, Ad)|Ad
  GL\_ONE\_MINUS\_DST\_ALPHA|(1, 1, 1) - (Ad, Ad, Ad)|1 - Ad
  GL\_CONSTANT\_COLOR|(Rc, Gc, Bc)|Ac
  GL\_ONE\_MINUS\_CONSTANT\_COLOR|(1, 1, 1) - (Rc, Gc, Bc)|1 - Ac
  GL\_CONSTANT\_ALPHA|(Ac, Ac, Ac)|Ac
  GL\_ONE\_MINUS\_CONSTANT\_ALPHA|(1, 1, 1) - (Ac, Ac, Ac)|1 - Ac
  GL\_SRC\_ALPHA\_SATURATE|(f, f, f)[f = min(As, 1- Ad)]|1

  ```c++
  // 这个函数进场用于实现在其他一些不透明的物体前面绘制一个透明物体的效果
  glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC)
  ```
 
#### 改变混合方程式
  可用的混合方程模式：

  模式|函数
  ----|----
  GL\_FUNC\_ADD|Cf = (Cs \* S) + (Cd \* D)
  GL\_FUNC\_SUBTRACT|Cf = (Cs \* S) - (Cd \* D)
  GL\_FUNC\_REVERSE\_SUBTRACT|Cf = (Cd \* D) - (Cs \* S)
  GL\_MIN|Cf = min(Cs, Cd)
  GL\_MAX|Cf = max(Cs, Cd)

  ```c++
  // 设置混合方程模式
  void glBlendEquation(GLenum mode);

  // 为RGB和alpha分别指定混合函数
  void glBlendFuncSeparate(GLenum srcRGB, GLenum dstRGB, GLenum srcAlpha, GLenum dstAlpha);
  ```

  方程式中常量的设定，例如GL\_CONSTANT\_COLOR：
  ```c++
  void glBlendColor(GLclampf red, GLclampf green, GLclampf blue, GLclampf alpha);
  ```

#### 抗锯齿
  OpenGL混合功能的另一个用途就是抗锯齿。

  为了消除图元之间的锯齿状边缘，OpenGL使用混合功能来混合片段的颜色。

  开启抗锯齿功能：
  ```c++
  glEnable(GL_BLEND);
  glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
  glBlendEquation(GL_FUNC_ADD);

  // 选择对点、线、多边形尽心抗锯齿处理
  glEnable(GL_POINT_SMOOTH);
  glEnable(GL_LINE_SMOOTH);
  glEnable(GL_POLYGON_SMOOTH);
  ```

  对实心物体尽心抗锯齿处理并不常用，在很大程度上已经被一种更好的对3D几何图形平滑边缘的成为多重采用的方法代替。如果不采用多重采用，我们在使用重叠的抗锯齿直线时仍然可能遇到这种重叠几何图形问题。例如，对于线框模型，通常可以通过禁用深度测试避免直线交叉部分的深度人工痕迹。

#### 多重采样
  抗锯齿处理的最大优点之一就是能够使多边形的边缘更为平滑，使渲染效果显得更为自然和逼真。点和直线的平滑处理得到广泛支持，但是多边形的平滑处理并没有在所有平台上都得到实现，这是因为抗锯齿处理是基于混合操作的，这就需要从前到后对所有的图元进行排序，这是非常麻烦的。

  OpenGL新增了一个特性，成为多重采样(multisampling)。它在颜色、深度、模板值的帧缓冲区额外添加一个缓冲区。所有的图元在每个像素上都进行了多次采样，其结果就存储在这个缓冲区中。每当这个像素进行更新时，这些采样值进行解析，以产生一个单独的值。

  这种处理会带来额外的内存和处理器开销，有可能对性能造成影响，因此，有些OpenGL实现可能并不支持多渲染环境中的多重采样。

  ```c++
  // 获取支持多重采样帧缓冲区的渲染环境
  glInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH | GLUT_MULTISAMPLE);

  // 打开或关闭
  glEnable(GL_MULTISAMPLE);
  glDisable(GL_MULTISAMPLE);
  ```

  当多重采样被启用时，点、直线、多边形的平滑特性都将被忽略，这意味着在使用多重采样时，就不能同时使用点和直线的平滑处理。
  ```c++
  glDisable(GL_MULTISAMPLE);
  glEnable(GL_POINT_SMOOTH);


  // Draw some smooth points
  // ...

  glDiable(GL_POINT_SMOOTH);
  glEnable(GL_MULTISAMPLE);
  ```

  对于性能敏感的程序员常常会不辞幸苦地对所有绘图命令进行排序，这样需要相同状态的几何图形就可以一起绘制。这种状态排序是在游戏中常用的提高速度的方法之一。

  多重采样缓冲区在默认情况下使用片段的RGB值，并不包含alpha成分，修改此行为：
  ```c++
  // 使用alpha值
  GL_SAMPLE_ALPHA_TO_COVERAGE

  // 将alpha值设为1并使用它
  GL_SAMPLE_ALPHA_TO_ON

  // 使用glSampleCoverage所设置的值
  GL_SAMPLE_COVERAGE
  void glSampleCoverage(GLclampf value, GLboolean invert);
  ```
