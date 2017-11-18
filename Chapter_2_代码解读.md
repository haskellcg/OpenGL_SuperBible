## drawtriangle代码解读
程序运行结果![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_1.PNG)

#### drawtriangle_test函数
```c++
int drawtriangle_test(int argc, char *argv[])
{
	gltSetWorkingDirectory(argv[0]);

	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA | GLUT_DEPTH | GLUT_STENCIL);
	glutInitWindowSize(800, 600);
	glutCreateWindow("Triangle");
	glutReshapeFunc(ChangeSize);
	glutDisplayFunc(RenderScene);

	GLenum err = glewInit();
	if (GLEW_OK != err){
		fprintf(stderr, "GLEW Error: %s\n", glewGetErrorString(err));
		return 1;
	}

	SetupRC();

	glutMainLoop();

	return 0;
}
```
  * gltSetWorkingDirectory: GLTools函数， 用来设置当前工作目录， 实际上在Windows是不需要的，因为工作目录默认就是与程序的可执行程序相同的目录，但是在Mac OS X中，这个程序将目前工作目录改为应用程序捆绑包中的/Resource文件夹
  * glutInit: GLUT函数，这个函数只是传输命令行参数并初始化GLUT库
  * glutInitDisplayMode: GLUT函数， 告诉GLUT库，在创建窗口时要使用那种类型的显示模式， GLUT_DOUBLE表示双缓冲窗口(表示指绘制命令实际上上是在离屏缓冲区执行的，然后迅速转换为窗口视图)；GLUT_RGBA表示RGBA颜色模式；GLUT_DEPTH表示一个深度缓冲区分配为显示的一部分，可以进行深度测试；GLUT_STENCIL表示一个可用的模版缓冲区
  * glutInitWindowSize，glutCreateWindow: 创建窗口，设置窗口大小，标题为"Triangle"
  * glutReshapeFunc，glutDisplayFunc: GLUT内部运行一个本地消息循环，拦截适当的消息，然后调用我们为不同时间注册的回调函数。与使用真正的系统特定框架相比有一定的局限性，但是大大简化了组织并运行一个程序的过程。这里我们必须为窗口改变大小而设置一个回调函数，注册一个函数以包含OpenGL渲染代码
  * glewInit: GLEW函数，初始化GLEW库，重新调用GLEW库初始化OpenGL驱动程序中所有丢失的入口点，以确保OpenGL API对我们来说完全可用
  * SetupRC: 这个函数对GLUT没有什么影响，但是在实际开始渲染之前，我们在这里进行任何OpenGL初始化都非常方便。这里的RC代表渲染环境(Rendering Context)，这是一个运行中的OpenGL状态机的句柄。在任何OpenGL函数起作用之前必须创建渲染环境，而GLUT在我们第一次创建窗口时就完成了这项工作
  * glutMainLoop: 开始主消息循环，该函数被调用之后，在主窗口被关闭之前都不会返回，并且一个应用程序中只需要调用一次，这个函数负责处理所有操作系统特定的消息、按键动作

#### ChangeSize函数
```c++
void ChangeSize(int w, int h)
{
	glViewport(0, 0, w, h);
}
```
  由于在不同环境下窗口的大小变化检测和处理方式也不同，GLUT库专门为此提供了glutRashapeFunc函数，该函数注册了一个回调，供GLUT库在窗口纬度改变时调用。
  
```c++
void glViewport(GLint x, GLint y, GLsizei width, GLsizei height);
```
  glViewport修改从目标坐标到屏幕坐标系的映射，要理解视口分辨率。
  
  在开始将几何图形光栅化(实际绘制)到屏幕上时，OpenGL负责笛卡尔坐标系和窗口像素，我们要牢记，就是改变视口并不会改变基础坐标系，由于我们采用默认的-1到+1的映射，为三角形改变窗口大小会产生一些有趣的结果。
  
#### SetupRC函数
```c++
void SetupRC()
{
	glClearColor(0.0f, 0.0f, 1.0f, 1.0f);

	shaderManager.InitializeStockShaders();

	GLfloat vVerts[] = {
		-0.5f, 0.0f, 0.0f,
		0.5f, 0.0f, 0.0f,
		0.0f, 0.5f, 0.0f
	};
	triangleBatch.Begin(GL_TRIANGLES, 3);
	triangleBatch.CopyVertexData3f(vVerts);
	triangleBatch.End();
}
```
  设置渲染环境，首先设置背景颜色，使用函数:
```c++
void glClearColor(GLclampf red, GLclampf green, GLclampf blue, GLclampf alpha);
```
  glCLearColor最后一个参数alpha，它用来进行混合，产生一些特殊的效果，例如透明，当然不仅仅依靠alpha就可以了。
  
  没有着色器，在OpenGL核心框架中就无法进行任何渲染。我们先使用简单的存储着色器，它们可以用着色器管理器进行管理:  
```c++
GLShaderManager shaderManager;
...
shaderManager.InitializeStockShaders();
```
  
  在OpenGL中，三角形是一种“图元”类型，是一种基本的3D绘图元素。我们通过将这些顶点放进一个单个浮点数数组来指定它们。本书中会讲解关于提交一个批次的顶点用于渲染的内容。一个简单的GLTool封装类会将三角形顶点进行封装：
```c++
GLBatch triangleBatch;
...
triangleBatch.Begin(GL_TRIANGLES, 3);
triangleBatch.CopyVertexData3f(vVerts);
triangleBatch.End();
```

#### RenderScene函数
```c++
void RenderScene(void)
{
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);

	GLfloat vRed[] = { 1.0f, 0.0f, 0.0f, 1.0f };
	shaderManager.UseStockShader(GLT_SHADER_IDENTITY, vRed);
	triangleBatch.Draw();

	glutSwapBuffers();
}
```
  前面我们将清除颜色设置为蓝色，现在我们需要执行一个函数真正进行清除：glClear, glClear函数清除一个或一组特定的缓冲区，缓冲区是一块存储图像信息的存储空间，红色、绿色、蓝色、alpha分量通常一起作为颜色缓冲区或者像素缓冲区引用。
  
  OpenGL中不只一种缓冲区(颜色缓冲区、深度缓冲区、模版缓冲区)。
  
  我们设置一组浮点数来表示红色，并将它传递给存储着色器，即GLT_SHADER_IDENTITY着色器，该着色器只是使用指定的颜色以默认笛卡尔坐标系在屏幕上渲染几何图形。GLBatch的Draw方法指示将几何图形提交给着色器。
  
  在设置OpenGL窗口时，我们指定要一个双缓冲区渲染环境，这就意味着将在后台进行渲染，然后在结束时提交给前台，这种形式能够防止观察者看到可能伴随动画帧与动画帧之间闪烁的渲染过程。缓冲区减缓将以平台特定的方式进行，GLUT有一个单独的函数调用可以完成:
```c++
glutSwapBuffers();
```

## drawsquaremove代码解读
  **把所有代码整合到一块，难免代码会重名，所以在后面的编写过程中尽量添加static关键字**
  
  程序运行结果(按'->'):
  ![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_2.PNG)
  ![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_3.PNG)
  
#### 修改部分
  ```c++
  squareBatch.Begin(GL_TRIANGLE_FAN, 4);
  squareBatch.CopyVertexData3f(vVerts);
  squareBatch.End();
  ```
  改程序画矩形，使用了GL_TRIANGLE_FAN，这样多个三角形连起来就形成结果中的矩形
      
#### 添加部分
  ```c++
  glutSpecialFunc(SpecialKeys);
  ```
  使用GLUT注册一个回调函数，该函数能够在按特殊键时被调用
  
  ```c++
  static void SpecialKeys(int key, int x, int y)
  {
	GLfloat stepSize = 0.1f;

	GLfloat blockX = vVerts[0];
	GLfloat blockY = vVerts[7];

	if (GLUT_KEY_UP == key){
		blockY += stepSize;
	} else if (GLUT_KEY_DOWN == key){
		blockY -= stepSize;
	} else if (GLUT_KEY_LEFT == key){
		blockX -= stepSize;
	} else if (GLUT_KEY_RIGHT == key){
		blockX += stepSize;
	}

	if (blockX < -1.0f){
		blockX = -1.0f;
	}
	if (blockX > (1.0f - blockSize * 2)){
		blockX = 1.0f - blockSize * 2;
	}
	if (blockY > 1.0f){
		blockY = 1.0f;
	}
	if (blockY < (-1.0f + blockSize * 2)){
		blockY = -1.0f + blockSize * 2;
	}

	vVerts[0] = blockX;
	vVerts[1] = blockY - blockSize * 2;
	vVerts[3] = blockX + blockSize * 2;
	vVerts[4] = blockY - blockSize * 2;
	vVerts[6] = blockX + blockSize * 2;
	vVerts[7] = blockY;
	vVerts[9] = blockX;
	vVerts[10] = blockY;

	squareBatch.CopyVertexData3f(vVerts);

	glutPostRedisplay();
  }
  ```
  该函数实现了按键之后的响应，根据按键计算移动之后的位置，并**检查边界做碰撞检测**，之后根据移动后的点计算其他点的位置，进行重绘
  
  ```c++
  glutPostRedisplay();
  ```
  该函数告诉GLUT需要更新窗口内容，默认情况下，在窗口创建、改变大小或者重绘时，GLUT通过调用RenderScene函数更新窗口，只要窗口发生最小化、恢复、最小化、覆盖、重新显示等变化，就会发生更新。我们可以人工调用glutPostRedisplay来告诉GLUT发生了某些变化，应该对场景进行渲染了。

## drawsquarebounce代码解读
  利用窗口定时重绘，OpenGL会调用RenderScene函数，来更新位置形成动画效果
  
  程序运行结果:  
  ![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_4.PNG)
  ![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_5.PNG)
  
#### 修改部分
  背景色，以及前景色:
  ```c++
  //默认就是黑色
  glClearColor(0.0f, 0.0f, 0.4f, 1.0f);
  
  GLfloat vGreen[] = {1.0f, 1.0f, 0.0f, 1.0f};
  shaderManager.UseStockShader(GLT_SHADER_IDENTITY, vGreen);
  squareBatch.Draw();
  ```
  
  在RenderScene中添加位置更新以及刷新屏幕代码:
  ```c++
  static void RenderScene(void)
  {
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);

	GLfloat vGreen[] = {1.0f, 1.0f, 0.0f, 1.0f};
	shaderManager.UseStockShader(GLT_SHADER_IDENTITY, vGreen);
	squareBatch.Draw();

	glutSwapBuffers();

	BounceFunction();
	glutPostRedisplay();
  }
  ```

#### 添加部分
  更新square位置更新的函数
  ```c++
  static void BounceFunction(void)
  {
	//设置不同的值，移动轨迹会慢慢改变
	static GLfloat xDir = 0.25f;
	static GLfloat yDir = 0.28f;

	GLfloat stepSize = 0.04f;
	
	GLfloat blockX = vVerts[0];
	GLfloat blockY = vVerts[7];

	blockX += stepSize * xDir;
	blockY += stepSize * yDir;

	if (blockX < -1.0f){
		blockX = -1.0f;
		xDir *= -1.0f;
	}
	if (blockX > (1.0f - blockSize * 2)){
		blockX = 1.0f - blockSize * 2;
		xDir *= -1.0f;
	}
	if (blockY > 1.0f){
		blockY = 1.0f;
		yDir *= -1.0f;
	}
	if (blockY < (-1.0f + blockSize * 2)){
		blockY = -1.0f + blockSize * 2;
		yDir *= -1.0f;
	}

	vVerts[0] = blockX;
	vVerts[1] = blockY - blockSize * 2;
	vVerts[3] = blockX + blockSize * 2;
	vVerts[4] = blockY - blockSize * 2;
	vVerts[6] = blockX + blockSize * 2;
	vVerts[7] = blockY;
	vVerts[9] = blockX;
	vVerts[10] = blockY;

	squareBatch.CopyVertexData3f(vVerts);
  }
  ```
  xDir/yDir是X、Y方向的移动速度矢量，遇到边界反向可以形成碰撞反弹的效果
