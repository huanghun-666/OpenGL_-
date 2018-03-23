

# 图元
## 一、基础管线
OpenGL 中图元只不过是顶点的集合以预定义的方式结合在一起。
管线分为2个部分：上半部分是客户机端，下半部分是服务器端。
服务器和客户端是功能和运行上都是异步的，它们是各自独立的软件或硬件块。
![](https://raw.githubusercontent.com/huanghun-666/OpenGL_-/master/2.png)
* 客户端是存储在 CPU 存储器中的，并且在应用程序中执行，或者在主系统内存的驱动程序中执行。驱动程序会将渲染命令和数组组合起来，发送给服务器执行！（在一台典型的个人计算机上，服务器实际上就是图形加速卡上的硬件和内存）
* 服务器和客户机在功能上也是异步的。它们是各自独立的软件块或硬件块。我们是希望它们2个端都尽量在不停的工作。客户端不断的把数据块和命令块组合在一起输送到缓冲区，然后缓冲区就会发送到服务器执行。
* 如果服务器停止工作等待客户机，或者客户机停止工作来等待服务器做好接受更多的命令和准备，我们把这种情况称为管线停滞

## 二、着色器
渲染过程，必备的2个着色器“顶点着色器”和“片元着色器”。
上图的Vertex Shader(顶点着⾊色器器) 和 Fragment Shader(⽚片段着⾊色器器)。

* 着⾊器是使⽤GLSL编写的程序，看起来与C语⾔非常类似。着⾊器必须从源代码中编译和链	接在⼀起。最终准备就绪的着色器程序
* 顶点着⾊器-->处理从客户机输⼊的数据、应⽤变换、进⾏其他的类型的数学运算来计算光照效果、位移、颜⾊色值等等。（**为了渲染共有3个顶点的三角形，顶点着⾊器将执⾏3次，也就是为了每个顶点执行⼀次）在⽬前的硬件上有多个执行单元同时运行，就意味着所有的3个顶点可以同时进行处理！
* 图上（primitive Assembly）说明的是：3个顶点已经组合在一起，而三角形已经逐个片段的进行了光栅化。每个片段通过执行 **片元着色器** 进行填充。片元着色器会输出我们将屏幕上看到的最终颜色值。


## 三、3种向 OpenGL 着色器传递渲染数据的方法
* 属性
* Uniform
* 纹理

### 3.1 属性
属性：就是对一个顶点都要作改变的数据元素。实际上，顶点位置本身就是一个属性。属性就可以浮点类型、整形、布尔类型。

* 属性总是以四维向量的形势进行内部存储的，即使我们不会使用所有的4个分量。一个顶点位置可能存储 （x,y,z），将占有4个分量中的3个。
* 实际上如果是在平面情况下：只要在 xy 平面上就能绘制，那么z分量就自动设置为0。
* 属性还可以是：纹理坐标、颜色值、光照计算表面法线
* 在顶点程序 （shader渲染）可以代表你想要的任何意义。因为都是你设定的。
* 属性会从本地客户机内存中复制在图形硬件中的一个缓冲区上。这些属性只提供给顶点着色器使用，对于片元着色器没有太大意义。
* **声明** 这些属性对每个顶点都要作改变，但并不意味着它们的值不能重复。通常情况下，它们都是不一样的，但有可能整个数组都是同一值的情况。

### 3.2 Uniform 值
通过设置 Uniform 变量就紧接着发送一个图元批次命令。Uniform 变量实际上可以无限地使用，设置一个应用于整个表面的单个颜色值。还可以设置一个时间值。在每次渲染某种类型的顶点动画时修改它。
* **注意** 这里的 uniform 变量每个批次改变一次，而不是每个顶点改变一次。
* uniform 变量最常见的应用是在顶点渲染中设置变换矩阵
* 与属性相同点：可以是浮点型、整数、布尔值
* 与属性不同点：顶点着色器和片元着色器都可以使用 uniform 变量。uniform 变量还可以是标量类型、矢量类型、uniform 矩阵。
### 3.3 纹理
简单的说，纹理就是矩形的数据数组。例如颜色、亮度数据等，之所以复杂，是因为矩形纹理可以映射到非矩形的区域。
## 四、创建坐标系
```
GLFrustum frustum;
void SetOrthographic(GLfloat xMin, GLfloat xMax, GLfloat yMin, GLfloat yMax, GLfloat zMin, GLfloat zMax);
```

## 五、透视投影
![](https://raw.githubusercontent.com/huanghun-666/OpenGL_-/master/1.jpg)
透视投影会进行透视除法对距离观察者很远的对象进行缩放和收缩。在投影到屏幕之后，视景体背面与视景体正面的宽度测量标准不同。
上图所示： 平头截体（frustum）的几何体，它的观察方向是从金字塔的尖端到宽阔端。观察者的视点与金字塔的尖端拉开一定距离。

GLFrustum类通过setPerspective ⽅方法为我们构建⼀一个平截头体。
参数：fFov:垂直⽅方向上的视场⻆角度fAspect:窗⼝口的宽度与⾼高度的纵横⽐比fNear:近裁剪⾯面距离fFar:远裁剪⾯面距离纵横⽐比 = 宽(w)/⾼高(h)
```
 CLFrustum::SetPerspective(float fFov,float fAspect,float fNear,float fFar);
```

## 六、存储着色器
### 6.1 GLShaderManager 的初始化
```
  GLShaderManager shaderManager;
  shaderManager.InitializeStockShaders();
  shaderManager.UseStockShader(参数列表) // 使用      
```
### 6.2 GLShaderManager 属性

| 标识符 | 描述符 | 
| :---  | :---: | 
| `GLT_ATTRIBUTE_VERTEX` | 3分量(x,y,z)顶点位置 | 
| `GLT_ATTRIBUTE_COLOR`  | 4分量(r,g,b,a)颜色值 |
| `GLT_ATTRIBUTE_NORMAL` | 3分量(x,y,z）表面法线 |
| `GLT_ATTRIBUTE_TEXTUREO` | 第一对2分量(s,t)纹理坐标 |
| `GLT_ATTRIBUTE_TEXTURE1` | 第二对2分量(s,t)纹理坐标 |

存储着色器为每一个变量都使用一致的内部变量命名规则和相同的属性槽。

### 6.3 GLShaderManager 的 uniform 值
* 一般情况，要对几何图形进行渲染，我们需要给对象递交属性矩阵，首先要绑定我们要使用的着色程序上，并提供程序的 uniform 值。但是 GLShanderManager 类可以暂时为我们完成工作。
* __useStockShader__ 函数会选择一个存储着色器并提供这个着色器的 unifrom 值。
```
GLShaderManager::UserStockShader(GLeunm shader ...);
```

* **单位着色器** 只是简单地使用默认笛卡尔坐标系（坐标范围（-1.0，1.0））。所有的片段都应用同一种颜色，几何图形为实心和为渲染的。 需要这种存储着色器一个属性： `GLT_ATTRIBUTE_VERTEX` （顶点分量）vColor[4] 需要的颜色
	```
	GLShaderManager::UserStockShader(GLT_ATTRIBUTE_VERTEX,GLfloat vColor[4]);
	```
* **平面着色器** 它将统一着色器进行拓展。允许为几何图形变换指定一个 4*4 变换矩阵。经常被统称为“模型视图投影矩阵”

	参数1：平面着色器 
	参数2：允许变化的 4*4 矩阵
	
	```
	GLShaderManager::UserStockShader(GLT_ATTRIBUTE_VERTEX,GLfloat vColor[4]);
	```
* **上色着色器** 在几何图形中应用的变换矩阵。需要设置存储着色器的 `GLT_ATTRIBUTE_VERTEX` (顶点分量) 和 `GLT_ATTRIBUTE_COLOR` (颜色分量）2个属性。颜色值将被平滑地插入顶点之间（平滑着色）
	```
	GLShaderManager::UserStockShader(GLT_SHADER_SHADED,GLfloat mvp[16]);
	```
* **默认光源着色器** 这种着色器，是对象产生阴影的光照效果。需要设置存储着色器的`GLT_ATTRIBUTE_VERTEX(顶点分量)`和`GLT_ATTRIBUTE_NORMAL(表面法线)`
	参数1：默认光源着色器
	参数2：模型视图矩阵
	参数3：投影矩阵
	参数4：颜色值
	```
	GLShaderManager::UserStockShader(GLT_SHADER_DEFAULT_LIGHT,GLfloat 	mvMatrix[16],GLfloat pMatrix[16],GLfloat vColor[4]);
	```
	
* **点光源着色器** 点光源着色器和默认光源着色器很相似，区别在于，光源位置是特定的。同样需要设置存储着色器的`GLT_ATTRIBUTE_VERTEX(顶点分量)`和`GLT_ATTRIBUTE_NORMAL(表面法线)`
	参数1：点光源着色器
	参数2：模型视图矩阵
	参数3：投影矩阵
	参数4：视点坐标光源位置
	参数5：颜色值
	```
	GLShaderManager::UserStockShader(GLT_SHADER_DEFAULT_LIGHT_DIEF,GLfloat 	mvMatrix[16],GLfloat pMatrix[16],GLfloat vLightPos[3],GLfloat vColor[4]);
	```

* **纹理替换矩阵** 着色器通过给定的模型视图投影矩阵，使用绑定到`nTextureUnit`(纹理单元)指定纹理单元的纹理对几何图形进行变化。
	片段颜色：是直接从纹理样本中直接获取的。
	需要设置存储着色器的 `GLT_ATTRIBUTE_VERTEX(顶点分量)`和`GLT_ATTRIBUTE_NORMAL(表面法	线)`
	```
	GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_REPLACE,GLfloat 	mvMatrix[16],GLintnTextureUnit);
	```
* **纹理调整着色器** 将纹理调整着色器乘以一个取自纹理单元 `nTextureUnit`的纹理。需要设置存储着色器的 `GLT_ATTRIBUTE_VERTEX(顶点分量)`和`GLT_ATTRIBUTE_TEXTURE0(纹理坐标)`
  ```
  GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_MODULATE,GLfloat	mvMatrix[16],GLfloat vColor[4],GLint nTextureUnit);
  ```
	
* **纹理光源着色器** 将一个纹理通过漫反射照明计算机进行调整（相乘）。光线在视觉空间中的位置是给定的。需要设置存储着色器的 `GLT_ATTRIBUTE_VERTEX(顶点分量)`和`GLT_ATTRIBUTE_TEXTURE0(纹理坐标)`、`GLT_ATTRIBUTE_NORMAL(表面法线)`
	参数1：纹理光源着色器
	参数2：投影矩阵
	参数3：视觉空间中的光源位置
	参数4：几何图形的基本色
	参数5：将要使用的纹理单元
	
	```
	GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_MODULATE,GLfloat	mvMatrix[16],GLfloat vColor[4],GLint nTextureUnit);
	```
	

## 七、OpenGL 图元
点、线、线带、线环、三角形、三角形金字塔、三角形带、三角形扇
```
//修改点的大小glPointSize(4.0f);//设置点的大小范围，点和点之间的距离GLfloat sizes[2] = {2.0f,4.0f};GLfloat stepSize = 1.0f;//获取点的大小范围和步长glGetFloatv(GL_POINT_SIZE_RANGE,sizes);glGetFloatv(GL_POINT_GRAULRITY,&stepSize);//3.通过使用程序点大小模式设置点大小glEnable(GL_PROGAM_POINT_SIZE);//GLSL程序gl_PointSize = 5.0f;
```
## 八、GLBatch 容器（帮助类）
```
void GLBatch::Begin(GLenum primitive, GLuint nVerts, GLuint nTextureUnits = 0);
void GLBatch::CopyNormalDataf(M3DVector3f *vNorms);
void GLBatch::CopyColorData4f(M3DVector4f *vColors);
void GLBatch::CopyTexCoordData2f(M3DVector2f *vTexCoords, GLuint uiTextureLayer);
```
![](https://raw.githubusercontent.com/huanghun-666/OpenGL_-/master/3.png)



