# 创建窗口

原文     | [Creating a window](http://learnopengl.com/#!Getting-started/Creating-a-window)
      ---|---
作者     | JoeyDeVries
翻译     | gjy_1992
校对     | Geequlim

在我们画出出色的效果之前，首先要做的就是创建一个OpenGL上下文(Context)和一个用于显示的窗口。然而，这些操作在每个系统上都是不一样的，OpenGL有目的的抽象(Abstract)这些操作。这意味着我们不得不自己处理创建窗口，定义OpenGL上下文以及处理用户输入。

幸运的是，有一些库已经提供了我们所需的功能，其中一部分是特别针对OpenGL的。这些库节省了我们书写平台相关代码的时间，提供给我们一个窗口和上下文用来渲染。最流行的几个库有GLUT，SDL，SFML和GLFW。在教程里我们将使用**GLFW**。

## GLFW

GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物件所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入。

这一节和下一节的内容是建立GLFW环境，并保证它恰当地创建窗口和OpenGL上下文。本教程会一步步从获取，编译，链接GLFW库讲起。我们使用Microsoft Visual Studio 2012 IDE，如果你用的不是它(或者只是Visual Studio的旧版本)请不要担心，大多数IDE上的操作都是类似的。Visual Studio 2012(或其他版本)可以从微软网站上免费下载(选择Express版本或Community版本)。

## 构建GLFW

GLFW可以从它们网站的[下载页](http://www.glfw.org/download.html)上获取。GLFW已经有针对Visual Studio 2012/2013的预编译的二进制版本和相应的头文件，但是为了完整性我们将从编译源代码开始，所以需要下载**源代码包**。


!!! Attention

	当你下载二进制版本时，请下载32位的版本而不是64位的除非你清楚你在做什么。大部分读者报告64位版本会出现很多奇怪的问题。


一旦下载完了源码包，解压到某处。我们只关心里面的这些内容：

- 编译生成的库
- **include**文件夹

从源代码编译库可以保证生成的目标代码是针对你的操作系统和CPU的，而一个预编译的二进制代码并不保证总是适合。提供源代码的一个问题是不是每个人都用相同的IDE来编译，因而提供的工程文件可能和一些人的IDE不兼容。所以人们只能从.cpp和.h文件来自己建立工程，这是一项笨重的工作。因此诞生了一个叫做CMake的工具。

### CMake

CMake是一个工程文件生成工具，可以使用预定义好的CMake脚本，根据用户的选择生成不同IDE的工程文件。这允许我们从GLFW源码里创建一个Visual Studio 2012工程文件。首先，我们需要从[这里](http://www.cmake.org/cmake/resources/software.html)下载安装CMake。我们选择Win32安装程序。

一旦CMake安装成功，你可以选择从命令行或者GUI启动CMake，为了简易我们选择后者。CMake需要一个源代码目录和一个存放编译结果的目标文件目录。源代码目录我们选择GLFW的源代码的根目录，然后我们新建一个_build_文件夹来作为目标目录。

![](http://learnopengl.com/img/getting-started/cmake.png)

之后，点击**Configure(设置)**按钮，我们选择生成的目标平台为**Visual Studio 11**(因为Visual Studio 2012的内部版本号是11.0)。CMake会显示可选的编译选项，这里我们使用默认设置，再次点击**Configure(设置)**按钮，保存这些设置。保存之后，我们可以点击**Generate(生成)**按钮，生成的工程文件就会出现在你的*build*文件夹中。

### 编译

在**build**文件夹里可以找到**GLFW.sln**文件，用Visual Studio 2012打开。因为CMake已经配置好了项目所以我们直接点击**Build Solution(构建解决方案)**然后编译的结果**glfw3.lib**就会出现在**src/Debug**文件夹内。(注意我们现在使用的glfw的版本号为3.1)

生成库之后，我们需要让IDE知道库和头文件的位置。有两种方法：

1. 找到IDE或者编译器的**/lib**和**/include**文件夹，之后添加GLFW的**include**目录到**/include**里去，相似的将**glfw3.lib**添加到**/lib**里去。这不是推荐的方式，因为很难去追踪library/include文件夹，而且重新安装IDE/Compiler可能会导致这些文件丢失。
2. 推荐的方式是建立一个新的目录包含所有的第三方库文件和头文件，并且在你的IDE/Compiler中指定这些文件夹。我个人使用一个单独的文件夹包含**Libs**和**Include**文件夹，在这里存放OpenGL工程用到的所有第三方库和头文件。这样我的所有第三方库都在同一个路径(并且应该在你的多台电脑间共享)，然而要求是每次新建一个工程我们都需要告诉IDE/编译器在哪能找到这些文件

完成上面步骤后，我们就可以使用GLFW创建我们的第一个OpenGL工程了！

## 我们的第一个工程

现在，让我们打开Visual Studio，创建一个新的工程。如果提供了多个选项，选择Visual C++，然后选择**空工程(Empty Project)**，别忘了给你的工程起一个合适的名字。现在我们有了一个空的工程去创建我们的OpenGL程序。

## 链接(Linking)

为了使我们的程序使用GLFW，我们需要把GLFW库**链接(Link)**进工程。于是我们需要在链接器的设置里写上**glfw3.lib**。但是我们的工程还不知道在哪寻找这个文件，于是我们首先需要将我们放第三方库的目录添加进设置。

为了添加这些目录，我们首先进入Project Properties(工程属性)(在解决方案窗口里右键项目)，然后选择**VC++ Directories**选项卡(如下图)。在下面的两栏添加目录：

![](http://learnopengl.com/img/getting-started/vc_directories.png)

从这里你可以把自己的目录加进去从而工程知道从哪去寻找库文件和头文件。可以手动把目录加在后面，也可以点**<Edit..>**选项，下面的图是Include Directories的设置：

![](http://learnopengl.com/img/getting-started/include_directories.png)

这里可以添加任意多个目录，IDE会从这些目录里寻找头文件。所以只要你将GLFW的**Include**文件夹加进路径中，你就可以使用**<GLFW/..>**来引用头文件。库文件也是一样的。

现在VS可以找到我们链接GLFW需要的所有文件了。最后需要在**Linker(链接器)**选项卡里的**Input**选项卡里添加**glfw3.lib**这个文件：

![](http://learnopengl.com/img/getting-started/linker_input.png)

要链接一个库我们必须告诉链接器它的文件名。因为我们的库名字是**glfw3.lib**，我们把它加到**Additional Dependencies**域里面(手动或者使用**<Edit..>**选项)。这样GLFW就会被链接进我们的工程。除了GLFW，你也需要链接OpenGL的库，但是这个库可能因为系统的不同而有一些差别。

### Windows上的OpenGL库

如果你是Windows平台，**opengl32.lib**已经随着Microsoft SDK装进了Visual Studio的默认目录，所以Windows上我们只需将**opengl32.lib**添加进Additional Dependencies。

### Linux上的OpenGL库

在Linux下你需要链接**libGl.so**，所以要添加**-lGL**到你的链接器设置里。如果找不到这个库你可能需要安装Mesa，NVidia或AMD的开发包，这部分因平台而异就不仔细讲解了。

现在，如果你添加好了GLFW和OpenGL库，你可以用如下方式添加GLFW头文件：

```c++
	#include <GLFW\glfw3.h>
```

这个头文件包含了GLFW的设置。

## GLEW

到这里，我们仍然有一件事要做。因为OpenGL只是一个规范，具体的实现是由驱动开发商针对特定显卡实现的。由于显卡驱动版本众多，大多数函数都无法在编译时确定下来，需要在运行时获取。开发者需要运行时获取函数地址并保存下来供以后使用。取得地址的方法因平台而异，Windows下看起来类似这样：

```c++
// 定义函数类型
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);
// 找到正确的函数并赋值给函数指针
GL_GENBUFFERS glGenBuffers  = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");
// 现在函数可以被正常调用了
GLuint buffer;
glGenBuffers(1, &buffer);
```

你可以看到代码复杂而笨重，因为我们对于每个函数都必须这样。幸运的是，有一个针对此目的的库，GLEW，是目前最流行的做这件事的方式。

### 编译和链接GLEW

GLEW是OpenGL Extension Wrangler Library的缩写，它管理我们上面提到的一系列繁琐的任务。因为GLEW也是一个库，我们同样需要链接进工程。GLEW可以从[这里](http://glew.sourceforge.net/index.html)下载，你可以选择下载二进制版本或者下载源码编译。记住，优先选择32位的二进制版本。

我们使用GLEW的静态版本glew32s.lib(注意这里的's')，用如上的方式添加其库文件和头文件，最后在链接器的选项里加上glew32s.lib。注意GLFW3也是编译成了一个静态库。


!!! Important

	**静态(Static)**链接是指编译时就将库代码里的内容合并进二进制文件。优点就是你不需要再放额外的文件，只需要发布你最终的二进制代码文件。缺点就是你的程序会变得更大，另外当库有升级版本时，你必须重新进行编译。
	**动态(Dynamic)**链接是指一个库通过.dll或.so的方式存在，它的代码与你的二进制文件的代码是分离的。优点是使你的程序大小变小并且更容易升级，缺点是你发布时必须带上这些dll。


如果你希望静态链接GLEW，必须在包含GLEW头文件之前定义预编译宏`GLEW_STATIC`：

```c++
#define GLEW_STATIC
#include <GL/glew.h>
```

如果你希望动态链接，那么就不要定义这个宏。但是使用动态链接的话你需要拷贝一份dll文件到你的应用程序目录。

!!! Important

	对于Linux用户建议使用这个命令行`-lGLEW -lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi`。没有正确链接相应的库会产生*undefined reference*(未定义的引用)这个错误。

我们现在成功编译了GLFW和GLEW库，我们将进入[下一节](http://learnopengl-cn.readthedocs.org/zh/latest/01%20Getting%20started/03%20Hello%20Window/)去使用GLFW和GLEW来设置OpenGL上下文并创建窗口。记住确保你的头文件和库文件的目录设置正确，以及链接器里引用的库文件名正确。如果仍然遇到错误，请参考额外资源中的例子。

##额外的资源

- [Building applications](http://www.opengl-tutorial.org/miscellaneous/building-your-own-c-application/): 提供了很多编译链接相关的信息以及一大批错误的解决方法。
- [GLFW with Code::Blocks](http://wiki.codeblocks.org/index.php?title=Using_GLFW_with_Code::Blocks):使用Code::Blocks IDE编译GLFW。
- [Running CMake](http://www.cmake.org/runningcmake/): 简要的介绍如何在Windows和Linux上使用CMake。
- [Writing a build system under Linux](http://learnopengl.com/demo/autotools_tutorial.txt): Wouter Verholst写的一个自动工具的教程，关于如何在Linux上建立编译环境，尤其是针对这些教程。
- [Polytonic/Glitter](https://github.com/Polytonic/Glitter): 一个简单的样板项目，它已经提前配置了所有相关的库；如果你想要很方便地搞到一个LearnOpenGL教程的范例工程，这是一个很好的东西。