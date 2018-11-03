---
title: MonoGame 中绘制 3D 图形的顶点
description: MonoGame 支持使用数组的顶点来定义三维对象在每个点的基础上的呈现方式。 用户可以利用顶点数组来创建动态的几何图形、 实现特殊效果，提高通过消除其呈现效率。
ms.prod: xamarin
ms.assetid: 932AF5C2-884D-46E1-9455-4C359FD7C092
author: conceptdev
ms.author: crdun
ms.date: 03/28/2017
ms.openlocfilehash: df36c149e98e8c0cbb16de4c2cf52def5713ec13
ms.sourcegitcommit: e268fd44422d0bbc7c944a678e2cc633a0493122
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50121433"
---
# <a name="drawing-3d-graphics-with-vertices-in-monogame"></a>MonoGame 中绘制 3D 图形的顶点

_MonoGame 支持使用顶点数组来定义3D对象在每个点上的渲染方式。 用户可以利用顶点数组创建动态几何体，实现特殊效果，并通过剔除提高渲染效率。_

阅读过[渲染模型指南](~/graphics-games/monogame/3d/part1.md)的用户会对在 MonoGame 中渲染3D模型感到熟悉。 在处理文件（例如.fbx）中定义的数据时以及处理静态数据时，`Model`类是渲染3D图形的有效方法。 某些游戏需要在运行时动态定义或操纵3D几何体。 在这些情况下，我们可以使用顶点数组来定义和渲染几何体。 顶点是3D空间中的点的通用术语，它是用于定义几何体的有序列表的一部分。 通常，顶点以定义一系列三角形的方式排序。

为了帮助直观地显示如何使用顶点创建3D对象，让我们考虑以下球体：


![](part2-images/image1.png "若要帮助直观地显示如何使用顶点创建三维对象，请考虑此球体")

如上所示，球体明显由多个三角形组成。 我们可以查看球体的线框以查看顶点是如何连接以组成三角形的。

![](part2-images/image2.png "查看球体，若要查看顶点如何连接到窗体三角形的线框")

本演练将涵盖以下主题：

- 创建一个工程
- 创建顶点
- 添加绘制代码
- 渲染纹理
- 修改纹理坐标
- 使用模型渲染顶点

完成的项目将包含一个使用顶点数组绘制的棋盘图案地板：


![](part2-images/image3.png "完成的项目将包含其将通过将顶点数组绘制越过方格形终点的 floor")

## <a name="creating-a-project"></a>创建项目

首先，我们将下载一个项目作为我们的起点。 我们将使用[可在此处找到](https://developer.xamarin.com/samples/mobile/ModelRenderingMG/)的模型项目。
下载并解压缩后，打开并运行该项目。 我们会在屏幕上看到六个机器人模型：

![](part2-images/image4.png "屏幕上绘制的六个机器人模型")

到本项目结束时，我们将自己的自定义顶点渲染与机器人`Model`相结合，因此我们不会删除机器人渲染代码。 相反，我们现在只需清除`Game1.Draw`方法中绘制6个机器人的部分。 为此，打开**Game1.cs**文件并找到`Draw`方法。 修改它以使其包含以下代码：

```csharp
protected override void Draw(GameTime gameTime)
{
  GraphicsDevice.Clear(Color.CornflowerBlue);
  base.Draw(gameTime);
}
```

这会导致我们的游戏显示蓝色屏幕：

![](part2-images/image5.png "这将导致显示空蓝屏游戏")

## <a name="creating-the-vertices"></a>创建顶点

我们将创建一个顶点数组来定义几何体。 在本演练中，我们将创建一个3D平面（3D空间中的正方形，而不是飞机，英文中plane也有飞机的意思）。 虽然我们的平面有四个边和四个角，但它将由两个三角形组成，每个三角形需要三个顶点。 因此，我们将总共定义六个点。

到目前为止，我们一直在讨论一般意义上的顶点，但 MonoGame 提供了一些可用于顶点的标准结构：


- `Microsoft.Xna.Framework.Graphics.VertexPositionColor`
- `Microsoft.Xna.Framework.Graphics.VertexPositionColorTexture`
- `Microsoft.Xna.Framework.Graphics.VertexPositionNormalTexture`
- `Microsoft.Xna.Framework.Graphics.VertexPositionTexture`

每种类型的名称都表明了它所包含的组件。 例如，`VertexPositionColor`包含位置和颜色的值。 让我们来看看每个组件：

- Position - 所有顶点类型都包含`Position`组件。 `Position`值定义了顶点在3D空间（X，Y和Z）中的位置。
- Color - 顶点可以选择指定`Color`以执行自定义着色。
- Normal - Normal 定义物体表面朝向的方向。 如果渲染具有光照的对象，则法线是必需的，因为曲面所面向的方向会影响它接收多少光照。 法线通常被指定为*单位矢量* - 长度为1的3D矢量。
- Texture - 纹理是指纹理坐标 - 即纹理的哪个部分和给定的顶点对应。 如果使用纹理渲染3D对象，则必须使用纹理值。 纹理坐标是归一化坐标，这意味着值将落在 0 和 1 之间。我们将在本指南后面更详细地介绍纹理坐标。

我们的平面将作为基底，并且我们将在执行渲染时使用纹理，因此我们将使用`VertexPositionTexture`类型来定义顶点。

首先，我们将在 Game1 类中添加一个成员：

```csharp
VertexPositionTexture[] floorVerts; 
```

接下来，在`Game1.Initialize`中定义我们的顶点。 请注意，本文前面提到的模板不包含`Game1.Initialize`方法，因此我们需要将整个方法添加到`Game1`：

```csharp
protected override void Initialize ()
{
    floorVerts = new VertexPositionTexture[6];
    floorVerts [0].Position = new Vector3 (-20, -20, 0);
    floorVerts [1].Position = new Vector3 (-20,  20, 0);
    floorVerts [2].Position = new Vector3 ( 20, -20, 0);
    floorVerts [3].Position = floorVerts[1].Position;
    floorVerts [4].Position = new Vector3 ( 20,  20, 0);
    floorVerts [5].Position = floorVerts[2].Position;
    // We’ll be assigning texture values later
    base.Initialize ();
}
```

为了帮助直观地观察顶点，请参考下图：


![](part2-images/image6.png "若要帮助直观地显示顶点将如下所示，请考虑此关系图")

我们需要依靠我们的图来可视化顶点，直到我们完成渲染代码的实现。

## <a name="adding-drawing-code"></a>添加绘制代码

在我们已经定义了几何体的位置，我们可以编写渲染代码。

首先，我们需要定义一个`BasicEffect`实例，该实例将保存用于渲染的参数，例如位置和光照。 为此，将一个类型为`BasicEffect`的成员添加到`Game1`类的`floorVerts`字段下面：


```csharp
...
VertexPositionTexture[] floorVerts;
// new code:
BasicEffect effect;
```

接下来，修改`Initialize`方法以定义效果：

```csharp
protected override void Initialize ()
{
    floorVerts = new VertexPositionTexture[6];

    floorVerts [0].Position = new Vector3 (-20, -20, 0);
    floorVerts [1].Position = new Vector3 (-20,  20, 0);
    floorVerts [2].Position = new Vector3 ( 20, -20, 0);

    floorVerts [3].Position = floorVerts[1].Position;
    floorVerts [4].Position = new Vector3 ( 20,  20, 0);
    floorVerts [5].Position = floorVerts[2].Position;
    // new code:
    effect = new BasicEffect (graphics.GraphicsDevice);

    base.Initialize ();
}
```

现在我们可以添加代码来执行绘制：

```csharp
void DrawGround()
{
    // The assignment of effect.View and effect.Projection
    // are nearly identical to the code in the Model drawing code.
    var cameraPosition = new Vector3 (0, 40, 20);
    var cameraLookAtVector = Vector3.Zero;
    var cameraUpVector = Vector3.UnitZ;

    effect.View = Matrix.CreateLookAt (
        cameraPosition, cameraLookAtVector, cameraUpVector);

    float aspectRatio = 
        graphics.PreferredBackBufferWidth / (float)graphics.PreferredBackBufferHeight;
    float fieldOfView = Microsoft.Xna.Framework.MathHelper.PiOver4;
    float nearClipPlane = 1;
    float farClipPlane = 200;

    effect.Projection = Matrix.CreatePerspectiveFieldOfView(
        fieldOfView, aspectRatio, nearClipPlane, farClipPlane);


    foreach (var pass in effect.CurrentTechnique.Passes)
    {
        pass.Apply ();

        graphics.GraphicsDevice.DrawUserPrimitives (
            // We’ll be rendering two trinalges
            PrimitiveType.TriangleList,
            // The array of verts that we want to render
            floorVerts,
            // The offset, which is 0 since we want to start 
            // at the beginning of the floorVerts array
            0,
            // The number of triangles to draw
            2);
    }
}
```

我们需要在我们的`Game1.Draw`方法中调用`DrawGround`:


```csharp
protected override void Draw (GameTime gameTime)
{
    GraphicsDevice.Clear (Color.CornflowerBlue);

    DrawGround ();

    base.Draw (gameTime);
}
```

应用程序在执行时将显示以下内容：


![](part2-images/image7.png "应用程序执行时将显示此")

让我们来看看上面代码中的一些细节。


### <a name="view-and-projection-properties"></a> View和Projection属性

`View`和`Projection`属性控制我们查看场景的方式。 稍后当我们重新添加模型渲染代码时，我们将修改此代码。 具体来说，`View`控制摄像机的位置和方向，`Projection`控制*视野*（可用于缩放摄像机）。

### <a name="techniques-and-passes"></a>Techniques和Passes

一旦我们为效果指定了属性，我们就可以执行实际渲染。

我们不会在本演练中更改`CurrentTechnique`属性，但更高级的游戏可以以不同的方式执行绘制（例如如何应用颜色值）来实现一种效果。 每一种渲染方式都可以表示为可以在渲染之前选用的Technique。 此外，每种Technique可能需要多个Pass才能正确渲染。 如果渲染复杂的视觉效果（如发光表面或毛发），效果可能需要多个Pass。

要记住的重要一点是，`foreach`循环使相同的C＃代码能够渲染任何效果，而不管底层`BasicEffect`的复杂性如何。

### <a name="drawuserprimitives"></a>DrawUserPrimitives

`DrawUserPrimitives`是渲染顶点的方法。 第一个参数告诉方法我们如何组织顶点。 我们已经组织了顶点，以便每个三角形由三个有序顶点定义，因此我们使用`PrimitiveType.TriangleList`的值。

第二个参数是我们之前定义的顶点数组。

第三个参数指定要绘制的第一个索引值。 由于我们希望渲染整个顶点数组，因此我们将传递的值为 0。

最后，我们指定要渲染的三角形数量。 我们的顶点数组包含两个三角形，因此传递的值为 2。

## <a name="rendering-with-a-texture"></a>使用纹理渲染


此时，我们的应用程序渲染出了一个白色平面（在透视模式下）。 接下来，我们将为渲染平面时使用的项目添加纹理。

为了简单起见，我们将.png直接添加到项目中，而不是使用 MonoGame Pipeline 工具。 为此，请将[此.png 文件](https://github.com/xamarin/mobile-samples/blob/master/ModelRenderingMG/Resources/checkerboard.png?raw=true)下载到您的计算机上。 下载完成后，右键单击解决方案面板中的**Content**文件夹，然后选择**Add>Add Files...**。 如果是在Android上工作，则此文件夹将位于Android工程中的**Assets**文件夹下。 如果在iOS上，那么此文件夹将位于iOS工程的根目录中。 导航到保存**checkerboard.png**的位置，然后选择该文件。 选择将文件复制到这个目录中。

接下来，我们将添加代码来创建`Texture2D`实例。 首先，将`Texture2D`作为`Game1`的成员添加到`BasicEffect`实例下方：

```csharp
...
BasicEffect effect;
// new code:
Texture2D checkerboardTexture;
```

修改`Game1.LoadContent`，如下所示：


```csharp
protected override void LoadContent()
{
    // Notice that loading a model is very similar
    // to loading any other XNB (like a Texture2D).
    // The only difference is the generic type.
    model = Content.Load<Model> ("robot");

    // We aren't using the content pipeline, so we need
    // to access the stream directly:
    using (var stream = TitleContainer.OpenStream ("Content/checkerboard.png"))
    {
        checkerboardTexture = Texture2D.FromStream (this.GraphicsDevice, stream);
    }
}
```

接下来，修改`DrawGround`方法。 唯一必要的修改是将`effect.TextureEnabled`赋值为`true`并将`effect.Texture`设置为`checkerboardTexture`：

```csharp
void DrawGround()
{
    // The assignment of effect.View and effect.Projection
    // are nearly identical to the code in the Model drawing code.
    var cameraPosition = new Vector3 (0, 40, 20);
    var cameraLookAtVector = Vector3.Zero;
    var cameraUpVector = Vector3.UnitZ;

    effect.View = Matrix.CreateLookAt (
        cameraPosition, cameraLookAtVector, cameraUpVector);

    float aspectRatio = 
        graphics.PreferredBackBufferWidth / (float)graphics.PreferredBackBufferHeight;
    float fieldOfView = Microsoft.Xna.Framework.MathHelper.PiOver4;
    float nearClipPlane = 1;
    float farClipPlane = 200;

    effect.Projection = Matrix.CreatePerspectiveFieldOfView(
        fieldOfView, aspectRatio, nearClipPlane, farClipPlane);

    // new code:
    effect.TextureEnabled = true;
    effect.Texture = checkerboardTexture;

    foreach (var pass in effect.CurrentTechnique.Passes)
    {
        pass.Apply ();

        graphics.GraphicsDevice.DrawUserPrimitives (
                    PrimitiveType.TriangleList,
            floorVerts,
            0,
            2);
    }
}
```

最后，我们需要修改`Game1.Initialize`方法以在顶点上指定纹理坐标：



```csharp
protected override void Initialize ()
{
    floorVerts = new VertexPositionTexture[6];

    floorVerts [0].Position = new Vector3 (-20, -20, 0);
    floorVerts [1].Position = new Vector3 (-20,  20, 0);
    floorVerts [2].Position = new Vector3 ( 20, -20, 0);

    floorVerts [3].Position = floorVerts[1].Position;
    floorVerts [4].Position = new Vector3 ( 20,  20, 0);
    floorVerts [5].Position = floorVerts[2].Position;

    // New code:
    floorVerts [0].TextureCoordinate = new Vector2 (0, 0);
    floorVerts [1].TextureCoordinate = new Vector2 (0, 1);
    floorVerts [2].TextureCoordinate = new Vector2 (1, 0);

    floorVerts [3].TextureCoordinate = floorVerts[1].TextureCoordinate;
    floorVerts [4].TextureCoordinate = new Vector2 (1, 1);
    floorVerts [5].TextureCoordinate = floorVerts[2].TextureCoordinate;

    effect = new BasicEffect (graphics.GraphicsDevice);

    base.Initialize ();
} 
```

如果我们运行此代码，我们可以看到我们的平面现在会显示出一个棋盘图案：


![](part2-images/image8.png "在平面现在显示棋盘图案")

## <a name="modifying-texture-coordinates"></a>修改纹理坐标

MonoGame 使用归一化纹理坐标，即坐标在 0 到 1 之间，而不是 0 到纹理的宽度或高度之间。 下图有助于来可视化归一化的坐标：

![](part2-images/image9.png "此图可以帮助可视化规范化的坐标")

归一化纹理坐标允许纹理调整大小而无需重写代码或重新创建模型（例如.fbx文件）。 因为归一化坐标表示的是比率而不是特定像素，才使之成为可能。 例如，无论纹理大小如何，（1, 1）将始终表示右下角。

我们可以更改纹理坐标分配，以使用单个变量来表示重复次数：


```csharp
protected override void Initialize ()
{
    floorVerts = new VertexPositionTexture[6];

    floorVerts [0].Position = new Vector3 (-20, -20, 0);
    floorVerts [1].Position = new Vector3 (-20,  20, 0);
    floorVerts [2].Position = new Vector3 ( 20, -20, 0);

    floorVerts [3].Position = floorVerts[1].Position;
    floorVerts [4].Position = new Vector3 ( 20,  20, 0);
    floorVerts [5].Position = floorVerts[2].Position;

    int repetitions = 20;

    floorVerts [0].TextureCoordinate = new Vector2 (0, 0);
    floorVerts [1].TextureCoordinate = new Vector2 (0, repetitions);
    floorVerts [2].TextureCoordinate = new Vector2 (repetitions, 0);

    floorVerts [3].TextureCoordinate = floorVerts[1].TextureCoordinate;
    floorVerts [4].TextureCoordinate = new Vector2 (repetitions, repetitions);
    floorVerts [5].TextureCoordinate = floorVerts[2].TextureCoordinate;

    effect = new BasicEffect (graphics.GraphicsDevice);

    base.Initialize ();
}
```

这会导致纹理重复 20 次：


![](part2-images/image10.png "这会导致重复 20 次的纹理")


## <a name="rendering-vertices-with-models"></a>渲染模型的顶点

现在，我们的平面被正确地渲染了，我们可以重新添加模型来查看一个整体的情况。 首先，我们将绘制模型的代码重新添加到我们的`Game1.Draw`方法中（包含已修改好的位置）：


```csharp
protected override void Draw(GameTime gameTime)
{
    GraphicsDevice.Clear(Color.CornflowerBlue);

    DrawGround ();

    DrawModel (new Vector3 (-4, 0, 3));
    DrawModel (new Vector3 ( 0, 0, 3));
    DrawModel (new Vector3 ( 4, 0, 3));

    DrawModel (new Vector3 (-4, 4, 3));
    DrawModel (new Vector3 ( 0, 4, 3));
    DrawModel (new Vector3 ( 4, 4, 3));

    base.Draw(gameTime);
} 
```

我们还将在`Game1`类中创建一个`Vector3`实例来表示我们相机的位置。 我们将在`checkerboardTexture`声明下添加一个字段：


```csharp
...
Texture2D checkerboardTexture;
// new code:
Vector3 cameraPosition = new Vector3(0, 10, 10); 
```

接下来，从`DrawModel`方法中删除局部变量`cameraPosition`：


```csharp
void DrawModel(Vector3 modelPosition)
{
    foreach (var mesh in model.Meshes)
    {
        foreach (BasicEffect effect in mesh.Effects)
        {
            effect.EnableDefaultLighting ();
            effect.PreferPerPixelLighting = true;

            effect.World = Matrix.CreateTranslation (modelPosition);

            var cameraLookAtVector = Vector3.Zero;
            var cameraUpVector = Vector3.UnitZ;

            effect.View = Matrix.CreateLookAt (
                cameraPosition, cameraLookAtVector, cameraUpVector); 
            ...
```

同样从`DrawGround`方法中删除局部变量`cameraPosition`：


```csharp
void DrawGround()
{
    // The assignment of effect.View and effect.Projection
    // are nearly identical to the code in the Model drawing code.
    var cameraLookAtVector = Vector3.Zero;
    var cameraUpVector = Vector3.UnitZ;

    effect.View = Matrix.CreateLookAt (
        cameraPosition, cameraLookAtVector, cameraUpVector);
    ... 
```

现在，如果我们运行代码，我们可以同时看到模型和地面：


![](part2-images/image11.png "模型和一开始显示在同一时间")

如果我们修改摄像机位置（例如通过增加其 X 值，在这种情况下摄像机将向左移动），我们可以看到该值会同时影响地面和模型：

```csharp
Vector3 cameraPosition = new Vector3(15, 10, 10);
```

此代码产生以下结果：

![](part2-images/image3.png "此代码会导致此视图")

## <a name="summary"></a>总结

本演练演示了如何使用顶点数组执行自定义渲染。 在这种情况下，我们通过将基于顶点的渲染与纹理和`BasicEffect`相结合来创建棋盘地板，此处提供的代码可用作任何 3D 渲染的基础。 我们还展示了基于顶点的渲染可以与同一场景中的模型一同被渲染。

## <a name="related-links"></a>相关链接

- [棋盘文件 （示例）](https://github.com/xamarin/mobile-samples/blob/master/ModelRenderingMG/Resources/checkerboard.png?raw=true)
- [已完成的项目 （示例）](https://developer.xamarin.com/samples/mobile/ModelsAndVertsMG/)
