---
title: Xamarin.Essentials： 文件系统帮助程序
description: Xamarin.Essentials 中的文件系统类包含一系列的帮助程序以查找应用程序的缓存和数据目录并打开应用包内的文件。
ms.assetid: B3EC2DE0-EFC0-410C-AF71-7410AE84CF84
author: jamesmontemagno
ms.author: jamont
ms.date: 05/04/2018
ms.openlocfilehash: 13293ec05261cbdc1e70fd278002d1af18654851
ms.sourcegitcommit: 632955f8cdb80712abd8dcc30e046cb9c435b922
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/11/2018
ms.locfileid: "38815613"
---
# <a name="xamarinessentials-file-system-helpers"></a>Xamarin.Essentials： 文件系统帮助程序

![预发行版 NuGet](~/media/shared/pre-release.png)

**FileSystem**类包含一系列的帮助程序可以找到应用程序的缓存和数据目录并打开应用包内的文件。

## <a name="using-file-system-helpers"></a>使用文件系统帮助程序

在类中添加对 Xamarin.Essentials 的引用：

```csharp
using Xamarin.Essentials;
```

若要获取应用程序的目录来存储**缓存数据**。 缓存数据可以用于任何需要保留时间超过临时数据，但不是应正确操作所需的数据的数据。

```csharp
var cacheDir = FileSystem.CacheDirectory;
```

若要获取应用程序的顶级目录不是用户数据文件的任何文件。 与同步框架的操作系统备份这些文件。 请参阅下面的平台实现细节。

```csharp
var mainDir = FileSystem.AppDataDirectory;
```

若要打开捆绑到应用程序包文件：

```csharp
 using (var stream = await FileSystem.OpenAppPackageFileAsync(templateFileName))
 {
    using (var reader = new StreamReader(stream))
    {
        var fileContents = await reader.ReadToEndAsync();
    }
 }
```

## <a name="platform-implementation-specifics"></a>平台实现的细节

# <a name="androidtabandroid"></a>[Android](#tab/android)

- **CacheDirectory** – 返回[CacheDir](https://developer.android.com/reference/android/content/Context.html#getCacheDir)的当前上下文。
- **AppDataDirectory** – 返回[FilesDir](https://developer.android.com/reference/android/content/Context.html#getFilesDir)的当前上下文和是否使用备份[自动备份](https://developer.android.com/guide/topics/data/autobackup.html)启动上 API 23 和更高版本。

添加到任何文件**资产**文件夹中的 Android 项目，然后将标记为生成操作**AndroidAsset**与其一起使用`OpenAppPackageFileAsync`。

# <a name="iostabios"></a>[iOS](#tab/ios)

- **CacheDirectory** – 返回[库/缓存](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html)目录。
- **AppDataDirectory** – 返回[库](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html)由 iTunes 和 iCloud 备份的目录。

添加到任何文件**资源**文件夹中的 iOS 项目，然后将标记为生成操作**BundledResource**与其一起使用`OpenAppPackageFileAsync`。

# <a name="uwptabuwp"></a>[UWP](#tab/uwp)

- **CacheDirectory** – 返回[LocalCacheFolder](https://docs.microsoft.com/en-us/uwp/api/windows.storage.applicationdata.localcachefolder#Windows_Storage_ApplicationData_LocalCacheFolder)目录...
- **AppDataDirectory** – 返回[LocalFolder](https://docs.microsoft.com/en-us/uwp/api/windows.storage.applicationdata.localfolder#Windows_Storage_ApplicationData_LocalFolder)备份到云的目录。

将任何文件添加到 UWP 项目的根节点并将标记为生成操作**内容**与其一起使用`OpenAppPackageFileAsync`。

--------------

## <a name="api"></a>API

- [文件系统帮助程序源代码](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/FileSystem)
- [文件系统 API 文档](xref:Xamarin.Essentials.FileSystem)
