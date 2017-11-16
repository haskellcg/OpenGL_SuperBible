## drawtriangle代码解读
程序运行结果![运行结果](https://github.com/haskellcg/Blog_Pictures/blob/master/Chapter_2_%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB_1.PNG)

### drawtriangle_test函数
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
