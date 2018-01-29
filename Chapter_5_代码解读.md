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
  GL\_PACK\_SKIP
