---
title: Xamarin.Essentials： 指南针
description: 本文档介绍 Xamarin.Essentials，可让你监视设备的地方，磁北标题中的 Compass 类。
ms.assetid: BF85B0C3-C686-43D9-811A-07DCAF8CDD86
author: jamesmontemagno
ms.author: jamont
ms.date: 05/04/2018
ms.openlocfilehash: c3fe98c384a87bdc08ce94e7537d1a6343767561
ms.sourcegitcommit: 51c274f37369d8965b68ff587e1c2d9865f85da7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39353878"
---
# <a name="xamarinessentials-compass"></a>Xamarin.Essentials： 指南针

![预发行版 NuGet](~/media/shared/pre-release.png)

**指南针**类可让你监视设备的地方，磁北标题。

## <a name="using-compass"></a>使用指南针

在类中添加对 Xamarin.Essentials 的引用：

```csharp
using Xamarin.Essentials;
```

指南针功能适用于通过调用`Start`和`Stop`方法来侦听对指南针更改。 任何更改发送回通过`ReadingChanged`事件。 下面是一个示例：

```csharp
public class CompassTest
{
    // Set speed delay for monitoring changes.
    SensorSpeed speed = SensorSpeed.UI;

    public CompassTest()
    {
        // Register for reading changes, be sure to unsubscribe when finished
        Compass.ReadingChanged += Compass_ReadingChanged;
    }

    void Compass_ReadingChanged(object sender, CompassChangedEventArgs e)
    {
        var data = e.Reading;
        Console.WriteLine($"Reading: {data.HeadingMagneticNorth} degrees");
        // Process Heading Magnetic North
    }

    public void ToggleCompass()
    {
        try
        {
            if (Compass.IsMonitoring)
              Compass.Stop();
            else
              Compass.Start(speed);
        }
        catch (FeatureNotSupportedException fnsEx)
        {
            // Feature not supported on device
        }
        catch (Exception ex)
        {
            // Some other exception has occurred
        }
    }
}
```

[!include[](~/essentials/includes/sensor-speed.md)]

## <a name="platform-implementation-specifics"></a>平台实现的细节

# <a name="androidtabandroid"></a>[Android](#tab/android)

Android 不提供一个 API 用于检索的指南针标题。 我们在此使用加速感应器和磁力仪来计算磁北方标题，google 建议这样做。

在极少数情况下，您可能看到不一致的结果传感器需要校准，因为其中涉及图 8 动态移动你的设备。 这将打开 Google 地图，点击上获得您所在位置的点，然后选择这样做的最佳方式**校准指南针**。

请注意，从您的应用程序在同一时间运行的多个传感器可能调整传感器速度。

## <a name="low-pass-filter"></a>低通滤波器

由于如何 Android 指南针更新值，并计算可能需要消除这些值。 一个_低传递筛选器_可以应用的角度的正弦和余弦值求平均值，并可以通过设置开启`ApplyLowPassFilter`属性`Compass`类：

```csharp
Compass.ApplyLowPassFilter = true;
```

这只应用在 Android 平台上。 可以读取的详细信息[此处](https://github.com/xamarin/Essentials/pull/354#issuecomment-405316860)。

--------------

## <a name="api"></a>API

- [指南针的源代码](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/Compass)
- [指南针 API 文档](xref:Xamarin.Essentials.Compass)
