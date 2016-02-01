# 面剔除（Face culling）

原文     | [Face culling](http://learnopengl.com/#!Advanced-OpenGL/Face-culling)
      ---|---
作者     | JoeyDeVries
翻译     | [Django](http://bullteacher.com/)
校对     | [Geequlim](http://geequlim.com)

尝试在头脑中想象一下有一个3D立方体，你从任何一个方向去看它，最多可以同时看到多少个面。如果你的想象力不是过于丰富，你最终最多能数出来的面是3个。你可以从一个立方体的任意位置和方向上去看它，但是你永远不能看到多于3个面。所以我们为何还要去绘制那三个不会显示出来的3个面呢。如果我们可以以某种方式丢弃它们，我们会提高片段着色器超过50%的性能！

!!! Important

        我们所说的是超过50%而不是50%，因为从一个角度只有2个或1个面能够被看到。这种情况下我们就能够提高50%以上性能了。


这的确是个好主意，但是有个问题需要解决：我们如何知道某个面在观察者的视野中不会出现呢？如果我们去想象任何封闭的几何平面，它们都有两面，一面面向用户，另一面背对用户。假如我们只渲染面向观察者的面会怎样？

这正是**面剔除**(Face culling)所要做的。OpenGL允许检查所有正面朝向（Front facing）观察者的面，并渲染它们，而丢弃所有背面朝向（Back facing）的面，这样就节约了我们很多片段着色器的命令（它们很昂贵！）。我们必须告诉OpenGL我们使用的哪个面是正面，哪个面是反面。OpenGL使用一种聪明的手段解决这个问题——分析顶点数据的连接顺序（Winding order）。


## 顶点连接顺序（Winding order）

当我们定义一系列的三角顶点时，我们会把它们定义为一个特定的连接顺序，它们可能是顺时针的或逆时针的。每个三角形由3个顶点组成，我们从三角形的中间去看，从而把这三个顶点指定一个连接顺序。

![](http://learnopengl.com/img/advanced/faceculling_windingorder.png)

正如你所看到的那样，我们先定义了顶点1，接着我们定义顶点2或3，这个不同的选择决定了这个三角形的连接顺序。下面的代码展示出这点：

```c++
GLfloat vertices[] = {
    //顺时针
    vertices[0], // vertex 1
    vertices[1], // vertex 2
    vertices[2], // vertex 3
    // 逆时针
    vertices[0], // vertex 1
    vertices[2], // vertex 3
    vertices[1] // vertex 2
};
```

每三个顶点都形成了一个包含着连接顺序的基本三角形。OpenGL使用这个信息在渲染你的基本图形的时候决定这个三角形是三角形的正面还是三角形的背面。默认情况下，**逆时针**的顶点连接顺序被定义为三角形的**正面**。

当定义你的顶点顺序时，你如果定义能够看到的一个三角形，那它一定是正面朝向的，所以你定义的三角形应该是逆时针的，就像你直接面向这个三角形。把所有的顶点指定成这样是件炫酷的事，实际的顶点连接顺序是在**光栅化**阶段（Rasterization stage）计算的，所以当顶点着色器已经运行后。顶点就能够在观察者的观察点被看到。

我们指定了它们以后，观察者面对的所有的三角形的顶点的连接顺序都是正确的，但是现在渲染的立方体另一面的三角形的顶点的连接顺序被反转。最终，我们所面对的三角形被视为正面朝向的三角形，后部的三角形被视为背面朝向的三角形。下图展示了这个效果：

![](http://learnopengl.com/img/advanced/faceculling_frontback.png)

在顶点数据中，我们定义的是两个逆时针顺序的三角形。然而，从观察者的方面看，后面的三角形是顺时针的，如果我们仍以1、2、3的顺序以观察者当面的视野看的话。即使我们以逆时针顺序定义后面的三角形，它现在还是变为顺时针。它正是我们打算剔除（丢弃）的不可见的面！



## 面剔除

在教程的开头，我们说过OpenGL可以丢弃背面朝向的三角形。现在我们知道了如何设置顶点的连接顺序，我们可以开始使用OpenGL默认关闭的面剔除选项了。

记住我们上一节所使用的立方体的定点数据不是以逆时针顺序定义的。所以我更新了顶点数据，好去反应为一个逆时针链接顺序，你可以[从这里复制它](http://learnopengl.com/code_viewer.php?code=advanced/faceculling_vertexdata)。把所有三角的顶点都定义为逆时针是一个很好的习惯。

开启OpenGL的`GL_CULL_FACE`选项就能开启面剔除功能：

```c++
glEnable(GL_CULL_FACE);
```

从这儿以后，所有的不是正面朝向的面都会被丢弃（尝试飞入立方体看看，里面什么面都看不见了）。目前，在渲染片段上我们节约了超过50%的性能，但记住这只对像立方体这样的封闭形状有效。当我们绘制上个教程中那个草的时候，我们必须关闭面剔除，这是因为它的前、后面都必须是可见的。

OpenGL允许我们改变剔除面的类型。要是我们剔除正面而不是背面会怎样？我们可以调用`glCullFace`来做这件事：

```c++
glCullFace(GL_BACK);
```

`glCullFace`函数有三个可用的选项：

* GL_BACK：只剔除背面。
* GL_FRONT：只剔除正面。
* GL_FRONT_AND_BACK：剔除背面和正面。

`glCullFace`的初始值是`GL_BACK`。另外，我们还可以告诉OpenGL使用顺时针而不是逆时针来表示正面，这通过glFrontFace来设置：

```c++
glFrontFace(GL_CCW);
```

默认值是`GL_CCW`，它代表逆时针，`GL_CW`代表顺时针顺序。

我们可以做个小实验，告诉OpenGL现在顺时针代表正面：

```c++
glEnable(GL_CULL_FACE);
glCullFace(GL_BACK);
glFrontFace(GL_CW);
```

最后的结果只有背面被渲染了：

![](http://learnopengl.com/img/advanced/faceculling_reverse.png)

要注意，你可以使用默认逆时针顺序剔除正面，来创建相同的效果：

```c
glEnable(GL_CULL_FACE);
glCullFace(GL_FRONT);
```

正如你所看到的那样，面剔除是OpenGL提高效率的一个强大工具，它使应用节省运算。你必须跟踪下来哪个物体可以使用面剔除，哪些不能。

## 练习

你可以自己重新定义一个顺时针的顶点顺序，然后用顺时针作为正面把它渲染出来吗：[解决方案](http://learnopengl.com/code_viewer.php?code=advanced/faceculling-exercise1)。
