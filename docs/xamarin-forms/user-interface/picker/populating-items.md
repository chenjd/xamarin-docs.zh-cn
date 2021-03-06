---
title: 将数据添加到选取器的项集合
description: 选取器视图是一个用于选择文本项中的数据列表控件。 本文介绍如何通过将其添加到项目集合中填充数据选取器以及如何响应用户的项选择。
ms.prod: xamarin
ms.assetid: 3C840F64-A430-457D-A4B2-3D7AF46F9DBE
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 04/11/2017
ms.openlocfilehash: 8d911108d7d72586a37a3281803eab9c0841f16c
ms.sourcegitcommit: 6e955f6851794d58334d41f7a550d93a47e834d2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2018
ms.locfileid: "38997102"
---
# <a name="adding-data-to-a-pickers-items-collection"></a>将数据添加到选取器的项集合

_选取器视图是一个用于选择文本项中的数据列表控件。本文介绍如何通过将其添加到项目集合中填充数据选取器以及如何响应用户的项选择。_

## <a name="populating-a-picker-with-data"></a>填充数据选取器

在 Xamarin.Forms 2.3.4，填充的过程之前[ `Picker` ](xref:Xamarin.Forms.Picker)的数据是添加到只读模式显示的数据[ `Items` ](xref:Xamarin.Forms.Picker.Items)集合，其类型`IList<string>`. 集合中的每个项的类型必须为`string`。 项可以在 XAML 中通过初始化中添加`Items`具有一系列属性`x:String`项：

```xaml
<Picker Title="Select a monkey">
  <Picker.Items>
    <x:String>Baboon</x:String>
    <x:String>Capuchin Monkey</x:String>
    <x:String>Blue Monkey</x:String>
    <x:String>Squirrel Monkey</x:String>
    <x:String>Golden Lion Tamarin</x:String>
    <x:String>Howler Monkey</x:String>
    <x:String>Japanese Macaque</x:String>
  </Picker.Items>
</Picker>
```

等效的 C# 代码如下所示：

```csharp
var picker = new Picker { Title = "Select a monkey" };
picker.Items.Add("Baboon");
picker.Items.Add("Capuchin Monkey");
picker.Items.Add("Blue Monkey");
picker.Items.Add("Squirrel Monkey");
picker.Items.Add("Golden Lion Tamarin");
picker.Items.Add("Howler Monkey");
picker.Items.Add("Japanese Macaque");
```

除了添加使用数据`Items.Add`方法中，数据还可以插入到集合使用`Items.Insert`方法。

## <a name="responding-to-item-selection"></a>响应的项选择

一个[ `Picker` ](xref:Xamarin.Forms.Picker)支持一次的一项选择。 当用户选择某个项， [ `SelectedIndexChanged` ](xref:Xamarin.Forms.Picker.SelectedIndexChanged)事件触发时，和[ `SelectedIndex` ](xref:Xamarin.Forms.Picker.SelectedIndex)属性更新为一个整数，表示列表中的选定项的索引。 `SelectedIndex`属性是一个从零开始的数字，指示用户选择的项。 如果未不选择任何项，这是这种情况时`Picker`首次创建和初始化，`SelectedIndex`将为-1。

> [!NOTE]
> 项中的选择行为[ `Picker` ](xref:Xamarin.Forms.Picker)可以上具有平台特定的 iOS 自定义。 有关详细信息，请参阅[控制选取器项选择](~/xamarin-forms/platform/platform-specifics/consuming/ios.md#picker_update_mode)。

下面的代码示例演示`OnPickerSelectedIndexChanged`事件处理程序方法，该方法时执行[ `SelectedIndexChanged` ](xref:Xamarin.Forms.Picker.SelectedIndexChanged)触发事件：

```csharp
void OnPickerSelectedIndexChanged(object sender, EventArgs e)
{
  var picker = (Picker)sender;
  int selectedIndex = picker.SelectedIndex;

  if (selectedIndex != -1)
  {
    monkeyNameLabel.Text = picker.Items[selectedIndex];
  }
}
```

此方法获取[ `SelectedIndex` ](xref:Xamarin.Forms.Picker.SelectedIndex)属性值，并使用该值来检索中的选定的项[ `Items` ](xref:Xamarin.Forms.Picker.Items)集合。 因为每个项目中`Items`集合是`string`，可以通过显示它们[ `Label` ](xref:Xamarin.Forms.Label)而无需强制转换。

> [!NOTE]
> 一个[ `Picker` ](xref:Xamarin.Forms.Picker)可初始化以通过设置显示特定项[ `SelectedIndex` ](xref:Xamarin.Forms.Picker.SelectedIndex)属性。 但是，`SelectedIndex`属性必须设置初始化后[ `Items` ](xref:Xamarin.Forms.Picker.Items)集合。

## <a name="summary"></a>总结

[ `Picker` ](xref:Xamarin.Forms.Picker)视图是一个用于选择文本项中的数据列表控件。 本文介绍了如何填充`Picker`通过将其添加到数据[ `Items` ](xref:Xamarin.Forms.Picker.Items)集合，以及如何响应用户的项选择。 这是使用的过程`Picker`Xamarin.Forms 2.3.4 之前。


## <a name="related-links"></a>相关链接

- [选取器演示 （示例）](https://developer.xamarin.com/samples/xamarin-forms/UserInterface/PickerDemo/)
- [选取器](xref:Xamarin.Forms.Picker)
