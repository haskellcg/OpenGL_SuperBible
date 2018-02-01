# 5. 基础纹理
  为了达到更加现实的效果，还有一种非常帮的捷径，这就是纹理贴图(texture mapping)。纹理只是一种能够应用到场景中的三角形上的图像数据，它通过经过过滤的纹理单元(texel，相当于基于纹理的像素)填充到实心区域。
## 5.1 原始图像数据
### 5.1.1 像素包装
  图像数据在内存中很少以紧密包装的形式存在，在许多硬件平台上，处于性能上的考虑，一幅图像的每一行都应该从一种特定的字节对齐地址开始。

  在默认情况下，OpenGL采用4个字节的对齐方式。很多程序员会简单地将图像宽度值乘以高度值，再乘以每个像素的字节数，这样就错误地判断了存储一个图像所需要的存储器数量。

  我们可以使用下列函数改变或者恢复像素的存储方式:
  ```c++
  void glPixelStorei(GLenum pname, GLint param);
  void glPixelStoref(GLenum pname, GLfloat param);
  
  // 改成紧密包装像素数据
  glPixelStorei(GL_UNPACK_ALIGHMENT, 1);
  ```

  glPixelStore参数:

  参数名|类型|初始值
  ------|----|------
  GL\_PACK\_SWAP\_BYTES|GLboolean|GL\_FALSE
  GL\_UNPACK\_SWAP\_BYTES|GLboolean|GL\_FALSE
  GL\_PACK\_LSB\_FIRST|GLboolean|GL\_FALSE
  GL\_UNPACK\_LSB\_FIRST|GLboolean|GL\_FALSE
  GL\_PACK\_ROW\_LENGTH|GLint|0
  GL\_UNPACK\_ROW\_LENGTH|GLint|0
  GL\_PACK\_SKIP\_ROWS|GLint|0
  GL\_UNPACK\_SKIP\_ROWS|GLint|0
  GL\_PACK\_SKIP\_PIXELS|GLint|0
  GL\_UNPACK\_SKIP\_PIXELS|GLint|0
  GL\_PACK\_ALIGNMENT|GLint|0
  GL\_UNPACK\_ALIGNMENT|GLint|0
  GL\_PACK\_IMGAE\_HEIGHT|Glint|0
  GL\_UNPACK\_IMAGE\_HEIGHT|GLint|0
  GL\_PACK\_SKIP\_IMAGES|Glint|0
  GL\_UNPACK\_SKIP\_IMAGES|GLint|0

### 5.1.2 像素图
  在当今全彩色的计算机系统中，更加有趣并且更加实用一些的是像素图(pixmap)。像素图在内存布局上与位图非常相似，但是每一个像素将需要一个以上的存储位表示。每个像素的附加位允许存储强度(intensity，有时被称为亮度，即luminance值)或者颜色分量值。

  在OpenGL核心版本中，我们无法直接将一个像素图绘制到颜色缓冲区中，但是可以使用下面的函数将颜色缓冲区的内容作为像素图直接读取:
  ```c++
  // x, y指定矩形左下角的窗口坐标
  // width, height指定窗口的大小
  // format指定pixels指向的数据元素的颜色布局
  // type解释参数pixels指向的数据
  void glReadPixels(GLint x,
                    GLint y,
                    GLSizei width,
                    GLSizei height,
                    GLenum format,
                    GLenum type,
                    const void *pixels);
  ```

  format枚举值:

  常量|描述
  ----|----
  GL\_RGB|按照红、绿、蓝顺序排列的颜色
  GL\_RGBA|按照红、绿、蓝、Alpha顺序排列的颜色
  GL\_BGR|按照蓝、绿、红顺序排列的颜色
  GL\_BGRA|按照蓝、绿、红、Alpha顺序排列的颜色
  GL\_RED|每个像素只包含一个红色分量
  GL\_GREEN|每个像素只包含一个绿色分量
  GL\_BLUE|每个像素只包含一个蓝色分量
  GL\_RG|每个像素一次包含一个红色和一个绿色分量
  GL\_RED\_INTEGER|每个像素包含一个整数形式的红色分量
  GL\_GREEN\_INTEGER|每个像素包含一个整数形式的绿色分量
  GL\_BLUE\_INTEGER|每个像素包含一个整数形式的蓝色分量
  GL\_RG\_INTEGER|每个像素一次包含一个整数形式的红色、绿色分量
  GL\_RGB\_INTEGER|每个像素一次包含整数形式的红色、绿色、蓝色分量
  GL\_RGBA\_INTEGER|每个像素一次包含整数形式的红色、绿色、蓝色、Alpha分量
  GL\_BGR\_INTEGER|每个像素一次包含整数形式的蓝色、绿色、红色分量
  GL\_BGRA\_INTEGER|每个像素一次包含整数形式的蓝色、绿色、红色、Alpha分量
  GL\_STENCIL\_INDEX|每个像素只包含一个模板值
  GL\_DEPTH\_COMPONENT|每个像素只包含一个深度值
  GL\_DEPTH\_STENCIL|每个像素包含一个深度值、模板值

  type枚举值:

  常量|描述
  ----|----
  GL\_UNSIGNED\_BYTE|每种颜色分量都是8位无符号整数
  GL\_BYTE|8位有符号整数
  GL\_UNSIGNED\_SHORT|16位无符号整数
  GL\_SHORT|16位有符号整数
  GL\_UNSIGNED\_INT|32位无符号整数
  GL\_INT|32位有符号整数
  GL\_FLOAT|单精度浮点数
  GL\_HALF\_FLOAT|半精度浮点数
  GL\_UNSIGNED\_BYTE\_3\_2\_2|包装的RGB值
  GL\_UNSIGNED\_BYTE\_2\_3\_3\_REV|包装的RGB值
  GL\_UNSIGNED\_SHORT\_5\_6\_5|包装的RGB值
  GL\_UNSIGNED\_SHORT\_5\_6\_5\_REV|包装的RGB值
  GL\_UNSIGNED\_SHORT\_4\_4\_4\_4|包装的RGBA值
  GL\_UNSIGNED\_SHORT\_4\_4\_4\_4\_REV|包装的RGBA值
  GL\_UNSIGNED\_SHORT\_5\_5\_5\_1|包装的RGBA值
  GL\_UNSIGNED\_SHORT\_1\_5\_5\_5\_REV|包装的RGBA值
  GL\_UNSIGNED\_INT\_8\_8\_8\_8|包装的RGBA值
  GL\_UNSIGNED\_INT\_8\_8\_8\_8\_REV|包装的RGBA值
  GL\_UNSIGNED\_INT\_10\_10\_10\_2|包装的RGBA值
  GL\_UNSIGNED\_INT\_2\_10\_10\_10\_REV|包装的RGBA值
  GL\_UNSIGNED\_INT\_24\_8|包装的RGBA值
  GL\_UNSIGNED\_INT\_10F\_11F\_11F\_REV|包装的RGBA值
  GL\_FLOAT\_32\_UNSIGNED\_INT\_24\_8\_REV|包装的RGBA值

  有必要指出，glReadPixels从图形硬件中复制数据，通常通过总线传输到系统内存。在这种情况下，应用程序将被阻塞，直到内存传输完成。

  此外，如果我们指定一个与图形硬件的本地排列不同的像素布局，那么在数据进行重定格式时将产生额外的性能开销。

### 5.1.3 包装的像素格式
  包装的像素格式将颜色数据压缩到了尽可能少的存储位中，每个颜色通道的位数显示在常量中
  ```c++
  UNSIGNED_BYTE_3_3_2
  +-+-+-+-+-+-+-+-+
  + 1st + 2nd + 3 +
  +-+-+-+-+-+-+-+-+

  UNSIGNED_BYTE_2_3_3_REV
  +-+-+-+-+-+-+-+-+
  + 3 + 2nd + 1st +
  +-+-+-+-+-+-+-+-+
  ```

  对于glReadPixels函数来说，读取操作在双缓冲区渲染环境下将在后台缓冲区进行，而在但缓冲区渲染环境下则在前台缓冲区进行，我们可以使用下面的函数改变这些像素操作的源:
  ```c++
  // GL_FRONT
  // GL_BACK
  // GL_LEFT
  // GL_RIGHT
  // GL_FRONT_LEFT
  // GL_FRONT_RIGHT
  // GL_BACK_LEFT
  // BL_BACK_RIGHT
  void glReadBuffer(GLenum mode);
  ```

### 5.1.4 保存像素
  GLTools库中的glWriteTGA函数从前台颜色缓冲区中读取颜色数据，并将这些数据存储到一个Target文件格式的图像文件中。
  ```c++
  // 捕获当前视口，并将它保存位一个targa文件
  // 确保在调用这个函数之前，在双缓冲区环境下调用SwapBuffers，而在單缓冲区环境下调用glFinish
  GLint gltWriteTGA(const char *szFileName)
  {
      FILE *pFile;                      // 文件指针
      TGAHEADER tgaHeader;              // TGA文件头
      unsigned long lImageSize;         // 图像大小，用字节表示
      GLbyte *pBits = NULL;             // 指向位的指针
      GLint iViewport[4];               // 以像素表示的视口
      GLenum lastBuffer;                // 存储当前的读取缓冲区设置
  }
  ```
