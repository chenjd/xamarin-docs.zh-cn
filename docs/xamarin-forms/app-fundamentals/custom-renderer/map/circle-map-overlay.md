---
title: 突出显示地图上的圆形区域
description: 本文介绍如何将循环覆盖添加到映射中，以突出显示地图的圆形区域。 IOS 和 Android 提供的 Api 向地图添加圆形覆盖，而 UWP 上在覆盖区上将呈现为多边形。
ms.prod: xamarin
ms.assetid: 6FF8BD15-074E-4E6A-9522-F9E2BE32EF12
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 11/29/2017
ms.openlocfilehash: 3064296d4c78a3342fb27afc971c37a029987e5e
ms.sourcegitcommit: 6e955f6851794d58334d41f7a550d93a47e834d2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2018
ms.locfileid: "38998552"
---
# <a name="highlighting-a-circular-area-on-a-map"></a>突出显示地图上的圆形区域

_本文介绍如何将循环覆盖添加到映射中，以突出显示地图的圆形区域。_

## <a name="overview"></a>概述

覆盖是在地图上的分层的图形。 覆盖层支持绘制图形的内容，它被放大随着与该映射。 以下屏幕截图显示将循环覆盖添加到映射的结果：

![](circle-map-overlay-images/screenshots.png)

当[ `Map` ](xref:Xamarin.Forms.Maps.Map) Xamarin.Forms 应用程序，在 iOS 中呈现控件`MapRenderer`类实例化时，这反过来实例化本机`MKMapView`控件。 在 Android 平台上`MapRenderer`类实例化本机`MapView`控件。 在通用 Windows 平台 (UWP)，`MapRenderer`类实例化本机`MapControl`。 渲染过程时可以执行利用通过创建自定义呈现器为实现特定于平台的映射自定义`Map`每个平台上。 执行此操作的过程如下所示：

1. [创建](#Creating_the_Custom_Map)Xamarin.Forms 自定义地图。
1. [使用](#Consuming_the_Custom_Map)Xamarin.Forms 中的自定义映射。
1. [自定义](#Customizing_the_Map)通过每个平台上创建代码图的自定义呈现器映射。

> [!NOTE]
> [`Xamarin.Forms.Maps`](xref:Xamarin.Forms.Maps) 必须初始化和使用之前配置。 有关详细信息，请参阅 [`Maps Control`](~/xamarin-forms/user-interface/map.md)

有关自定义使用自定义呈现器的映射的信息，请参阅[自定义图钉](~/xamarin-forms/app-fundamentals/custom-renderer/map/customized-pin.md)。

<a name="Creating_the_Custom_Map" />

### <a name="creating-the-custom-map"></a>创建自定义地图

创建`CustomCircle`类具有`Position`和`Radius`属性：

```csharp
public class CustomCircle
{
  public Position Position { get; set; }
  public double Radius { get; set; }
}
```

然后，创建一个子类[ `Map` ](xref:Xamarin.Forms.Maps.Map)类，将类型的属性添加`CustomCircle`:

```csharp
public class CustomMap : Map
{
  public CustomCircle Circle { get; set; }
}
```

<a name="Consuming_the_Custom_Map" />

### <a name="consuming-the-custom-map"></a>使用自定义地图

使用`CustomMap`控件通过声明它的实例中的 XAML 页实例：

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MapOverlay;assembly=MapOverlay"
             x:Class="MapOverlay.MapPage">
    <ContentPage.Content>
        <local:CustomMap x:Name="customMap" MapType="Street" WidthRequest="{x:Static local:App.ScreenWidth}" HeightRequest="{x:Static local:App.ScreenHeight}" />
    </ContentPage.Content>
</ContentPage>
```

或者，使用`CustomMap`控件通过声明它的实例中的 C# 页实例：

```csharp
public class MapPageCS : ContentPage
{
    public MapPageCS ()
    {
        var customMap = new CustomMap {
            MapType = MapType.Street,
            WidthRequest = App.ScreenWidth,
            HeightRequest = App.ScreenHeight
        };
        ...
        Content = customMap;
    }
}
```

初始化`CustomMap`控件：

```csharp
public partial class MapPage : ContentPage
{
  public MapPage ()
  {
    ...
    var pin = new Pin {
      Type = PinType.Place,
      Position = new Position (37.79752, -122.40183),
      Label = "Xamarin San Francisco Office",
      Address = "394 Pacific Ave, San Francisco CA"
    };

    var position = new Position (37.79752, -122.40183);
    customMap.Circle = new CustomCircle {
      Position = position,
      Radius = 1000
    };

    customMap.Pins.Add (pin);
    customMap.MoveToRegion (MapSpan.FromCenterAndRadius (position, Distance.FromMiles (1.0)));
  }
}
```

此初始化添加[ `Pin` ](xref:Xamarin.Forms.Maps.Pin)并`CustomCircle`实例到自定义映射，并将具有地图的视图[ `MoveToRegion` ](xref:Xamarin.Forms.Maps.Map.MoveToRegion*)方法，以更改的位置和缩放通过创建映射的级别[ `MapSpan` ](xref:Xamarin.Forms.Maps.MapSpan)从[ `Position` ](xref:Xamarin.Forms.Maps.Position)和一个[ `Distance` ](xref:Xamarin.Forms.Maps.Distance)。

<a name="Customizing_the_Map" />

### <a name="customizing-the-map"></a>自定义地图

自定义呈现器必须现在添加到要向地图添加循环覆盖每个应用程序项目中。

#### <a name="creating-the-custom-renderer-on-ios"></a>在 ios 设备上创建自定义呈现器

创建一个子类`MapRenderer`类并重写其`OnElementChanged`方法中添加循环覆盖：

```csharp
[assembly: ExportRenderer(typeof(CustomMap), typeof(CustomMapRenderer))]
namespace MapOverlay.iOS
{
    public class CustomMapRenderer : MapRenderer
    {
        MKCircleRenderer circleRenderer;

        protected override void OnElementChanged(ElementChangedEventArgs<View> e)
        {
            base.OnElementChanged(e);

            if (e.OldElement != null) {
                var nativeMap = Control as MKMapView;
                if (nativeMap != null) {
                    nativeMap.RemoveOverlays(nativeMap.Overlays);
                    nativeMap.OverlayRenderer = null;
                    circleRenderer = null;
                }
            }

            if (e.NewElement != null) {
                var formsMap = (CustomMap)e.NewElement;
                var nativeMap = Control as MKMapView;
                var circle = formsMap.Circle;

                nativeMap.OverlayRenderer = GetOverlayRenderer;

                var circleOverlay = MKCircle.Circle(new CoreLocation.CLLocationCoordinate2D(circle.Position.Latitude, circle.Position.Longitude), circle.Radius);
                nativeMap.AddOverlay(circleOverlay);
            }
        }
        ...
    }
}

```

此方法执行下面的配置，前提是自定义呈现器附加到新 Xamarin.Forms 元素：

- `MKMapView.OverlayRenderer`属性设置为对应的委托。
- 通过设置静态创建圆形`MKCircle`对象，以米为单位指定圆的中心和圆的半径。
- 该圆形被添加到映射通过调用`MKMapView.AddOverlay`方法。

然后，实现`GetOverlayRenderer`方法以自定义在覆盖区上的呈现：

```csharp
public class CustomMapRenderer : MapRenderer
{
  MKCircleRenderer circleRenderer;
  ...

  MKOverlayRenderer GetOverlayRenderer(MKMapView mapView, IMKOverlay overlayWrapper)
  {
      if (circleRenderer == null && !Equals(overlayWrapper, null)) {
          var overlay = Runtime.GetNSObject(overlayWrapper.Handle) as IMKOverlay;
          circleRenderer = new MKCircleRenderer(overlay as MKCircle) {
              FillColor = UIColor.Red,
              Alpha = 0.4f
          };
      }
      return circleRenderer;
  }
}
```

#### <a name="creating-the-custom-renderer-on-android"></a>在 Android 上创建自定义呈现器

创建一个子类`MapRenderer`类并重写其`OnElementChanged`和`OnMapReady`添加循环覆盖的方法：

```csharp
[assembly: ExportRenderer(typeof(CustomMap), typeof(CustomMapRenderer))]
namespace MapOverlay.Droid
{
    public class CustomMapRenderer : MapRenderer
    {
        CustomCircle circle;

        public CustomMapRenderer(Context context) : base(context)
        {
        }

        protected override void OnElementChanged(Xamarin.Forms.Platform.Android.ElementChangedEventArgs<Xamarin.Forms.Maps.Map> e)
        {
            base.OnElementChanged(e);

            if (e.OldElement != null)
            {
                // Unsubscribe
            }

            if (e.NewElement != null)
            {
                var formsMap = (CustomMap)e.NewElement;
                circle = formsMap.Circle;
                Control.GetMapAsync(this);
            }
        }

        protected override void OnMapReady(Android.Gms.Maps.GoogleMap map)
        {
            base.OnMapReady(map);

            var circleOptions = new CircleOptions();
            circleOptions.InvokeCenter(new LatLng(circle.Position.Latitude, circle.Position.Longitude));
            circleOptions.InvokeRadius(circle.Radius);
            circleOptions.InvokeFillColor(0X66FF0000);
            circleOptions.InvokeStrokeColor(0X66FF0000);
            circleOptions.InvokeStrokeWidth(0);

            NativeMap.AddCircle(circleOptions);
        }
    }
}
```

`OnElementChanged`方法调用`MapView.GetMapAsync`方法，获取基础`GoogleMap`的，提供自定义呈现器附加到新 Xamarin.Forms 元素绑定到视图。 一次`GoogleMap`实例不可用，`OnMapReady`将调用方法，其中通过实例化创建圆形`CircleOptions`对象，以米为单位指定圆的中心和圆的半径。 该圆形然后通过调用添加到映射`NativeMap.AddCircle`方法。

#### <a name="creating-the-custom-renderer-on-the-universal-windows-platform"></a>在通用 Windows 平台上创建自定义呈现器

创建一个子类`MapRenderer`类并重写其`OnElementChanged`方法中添加循环覆盖：

```csharp
[assembly: ExportRenderer(typeof(CustomMap), typeof(CustomMapRenderer))]
namespace MapOverlay.UWP
{
    public class CustomMapRenderer : MapRenderer
    {
        const int EarthRadiusInMeteres = 6371000;

        protected override void OnElementChanged(ElementChangedEventArgs<Map> e)
        {
            base.OnElementChanged(e);

            if (e.OldElement != null)
            {
                // Unsubscribe
            }
            if (e.NewElement != null)
            {
                var formsMap = (CustomMap)e.NewElement;
                var nativeMap = Control as MapControl;
                var circle = formsMap.Circle;

                var coordinates = new List<BasicGeoposition>();
                var positions = GenerateCircleCoordinates(circle.Position, circle.Radius);
                foreach (var position in positions)
                {
                    coordinates.Add(new BasicGeoposition { Latitude = position.Latitude, Longitude = position.Longitude });
                }

                var polygon = new MapPolygon();
                polygon.FillColor = Windows.UI.Color.FromArgb(128, 255, 0, 0);
                polygon.StrokeColor = Windows.UI.Color.FromArgb(128, 255, 0, 0);
                polygon.StrokeThickness = 5;
                polygon.Path = new Geopath(coordinates);
                nativeMap.MapElements.Add(polygon);
            }
        }
        // GenerateCircleCoordinates helper method (below)
    }
}
```

提供自定义呈现器附加到新 Xamarin.Forms 元素，此方法将执行以下操作：

- 从检索的圆圈位置和半径`CustomMap.Circle`属性，并传递给`GenerateCircleCoordinates`方法，后者生成纬度和经度坐标圆圈外围。 此帮助器方法的代码如下所示。
- 圆形外围坐标转换为`List`的`BasicGeoposition`坐标。
- 通过实例化创建圆形`MapPolygon`对象。 `MapPolygon`类用于在地图上显示多点形状，通过设置其`Path`属性设置为`Geopath`对象，其中包含的形状坐标。
- 将其添加到地图上呈现多边形`MapControl.MapElements`集合。


```
List<Position> GenerateCircleCoordinates(Position position, double radius)
{
    double latitude = position.Latitude.ToRadians();
    double longitude = position.Longitude.ToRadians();
    double distance = radius / EarthRadiusInMeteres;
    var positions = new List<Position>();

    for (int angle = 0; angle <=360; angle++)
    {
        double angleInRadians = ((double)angle).ToRadians();
        double latitudeInRadians = Math.Asin(Math.Sin(latitude) * Math.Cos(distance) + Math.Cos(latitude) * Math.Sin(distance) * Math.Cos(angleInRadians));
        double longitudeInRadians = longitude + Math.Atan2(Math.Sin(angleInRadians) * Math.Sin(distance) * Math.Cos(latitude), Math.Cos(distance) - Math.Sin(latitude) * Math.Sin(latitudeInRadians));

        var pos = new Position(latitudeInRadians.ToDegrees(), longitudeInRadians.ToDegrees());
        positions.Add(pos);
    }

    return positions;
}
```

## <a name="summary"></a>总结

本文介绍了如何将循环覆盖添加到映射，以突出显示地图的圆形区域。


## <a name="related-links"></a>相关链接

- [循环映射 Ovlerlay （示例）](https://developer.xamarin.com/samples/xamarin-forms/customrenderers/map/circle/)
- [自定义图钉](~/xamarin-forms/app-fundamentals/custom-renderer/map/customized-pin.md)
- [Xamarin.Forms.Maps](xref:Xamarin.Forms.Maps)
