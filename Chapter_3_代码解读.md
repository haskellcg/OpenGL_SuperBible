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
  GLT_ATTRIBUTE_VERTEX|3分量(x, y, z)顶点位置
  GLT_ATTRIBUTE_COLOR|4分量(r, g, b, a)颜色值
  GLT_ATTRIBUTE_NORMAL|3分量(x, y, z)表面法线
  GLT_ATTRIBUTE_TEXTURE0|第一对2分量(s, t)纹理坐标
  GLT_ATTRIBUTE_TEXTURE1|第二对2分量(s, t)纹理坐标
  
  * Uniform值
    要对几何图形进行渲染，我们需要为对象递交**属性矩阵**，但是首先要绑定到我们想要使用的**着色器程序**，并提供程序的**Uniform值**。通过函数:
    ```c++
    GLShaderManager::UseStockShader(GLenum shader, ......);
    ```
    
    * 单位(Identity)着色器: 简单地使用默认的笛卡尔坐标系(-1.0 ~ 1.0)，所有片段都应用同一颜色，几何图形为实心且未渲染。这种着色器只使用**GLT_ATTRIBUTE_VERTEX**，vColor参数包含了颜色:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_IDENTITY, GLfloat vColor[4]);
    ```
    
    * 平面(Flat)着色器: 将统一着色器进行扩展，允许为几何图形变**换指定一个4x4的变换矩阵**，典型情况，这是一种**左乘模型视图矩阵和投影矩阵**，经常被称为“**模型试图投影矩阵**”。这种着色器只使用一个属性**GLT_ATTRIBUTE_VERTEX**:
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
    
    * 电光源着色器: 光源位置是固定的，接受4个Uniform值**模型视图矩阵**, **投影矩阵**, **视点坐标系中的光源位置**, **对象的基本漫反射颜色**，所需要的属性有**GLT_ATTRIBUTE_VERTEX**， **GLT_ATTRIBUTE_NORMAL**:
    ```c++
    GLShaderManager::UseStockShader(GLT_SHADER_POINT_LIGHT_DIFF, GLfloat mvMatrix[16], 
                                                                 GLfloat pMatrix[16], 
                                                                 GLfloat vLightPos[3],
                                                                 GLfloat vColor[4]);
    ```
    
    * 纹理替换矩阵: 着色器通过给定的**模型视图矩阵**，使用绑定到**nTextUnit指定的纹理单元**的纹理对几何图形进行变换。**片段颜色**是直接从纹理样本中获取，所需要的属性有**GLT_ATTRIBUTE_VERTEX**，**GLT_ATTRIBUTE_NORMAL**:
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
GL_POINTS|每个顶点在屏幕上都是一个单独的点
GL_LINES|每对对点对应一个线段
GL_LINE_STRIP|一个从第一个顶点依次经过每个后续顶点而绘制的线条
GL_LINE_LOOP|和GL_LINE_STRIP一样，但是最后一个顶点和第一个顶点也连接起来
GL_TRIANGLES|每三个顶点定义一个新的三角形
GL_TRIANGLES_STRIP|共用一个条带上的顶点的一组三角形

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
  
  * 三角形带: GL_TRIANGLES_STRIP, **按照第一个三角形的环绕顺序进行后面三角形的绘制**，需要绘制大量三角形时，采用这种方式可以节省大量程序代码和数据空间，而且可以提高运算性能和节省宽带
  * 三角形扇: GL_TRIANGLE_FAN
  
#### 批次容器  
  ```c++
  ```
