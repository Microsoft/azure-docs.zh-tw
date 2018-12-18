---
title: 教學課程 - 升級 Azure Service Fabric Mesh 應用程式 | Microsoft Docs
description: 了解如何使用 Visual Studio 升級 Service Fabric 應用程式
services: service-fabric-mesh
documentationcenter: .net
author: tylerMSFT
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: azure-cli
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/18/2018
ms.author: twhitney
ms.custom: mvc, devcenter
ms.openlocfilehash: a734eb7b4efecf14a4d47a660c00d561a5fa6677
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971802"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>教學課程：了解如何使用 Visual Studio 升級 Service Fabric 應用程式

本教學課程是系列中的第四部分，說明如何直接從 Visual Studio 升級 Azure Service Fabric Mesh 應用程式。 升級將包含程式碼更新和組態更新。 您將看到在 Visual Studio 中升級和發佈的步驟是相同的。

在本教學課程中，您將了解如何：
> [!div class="checklist"]
> * 使用 Visual Studio 升級 Service Fabric Mesh 服務

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [在 Visual Studio 中建立 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [偵錯在本機開發叢集中執行的 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [部署 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * 升級 Service Fabric Mesh 應用程式
> * [清除 Service Fabric Mesh 資源](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您尚未部署待辦事項應用程式，請依照[發佈 Service Fabric Mesh Web 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)中的指示進行操作。

# <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>使用 Visual Studio 升級 Service Fabric Mesh 服務

本文說明如何獨立升級應用程式內的微服務。  在此範例中，我們將修改 `WebFrontEnd` 服務以顯示工作類別。 然後，升級已部署的服務。

## <a name="modify-the-config"></a>修改組態

升級可能是因為程式碼變更、組態變更，或兩者同時影響。  若要引進組態變更，請開啟 `WebFrontEnd` 專案的 `service.yaml` 檔案 (位於**服務資源**節點下方)。

在 `resources:` 區段中，將 `cpu:` 從 0.5 變更為 1.0，預計將會大量使用 Web 前端。 目前看起來應該如下所示：

```xml
              ...
              resources:
                requests:
                  cpu: 1.0
                  memoryInGB: 1
              ...
```

## <a name="modify-the-model"></a>修改模型

若要引進程式碼變更，請將 `Category` 屬性新增至 `ToDoItem.cs` 檔案中的 `ToDoItem` 類別。

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

然後更新 `Load()` 方法，在相同檔案中，將類別設定為預設的字串：

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>修改服務

`WebFrontEnd` 專案是一個 ASP.NET Core 應用程式，具有會在待辦事項清單項目中顯示的網頁。 在 `WebFrontEnd` 專案中，開啟 `Index.cshtml` 並新增下列兩行，以顯示工作的類別：

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

建立並執行應用程式，以驗證您是否在列出工作的網頁中看到新類別資料行。

## <a name="upgrade-the-app-from-visual-studio"></a>從 Visual Studio 升級應用程式

無論您是進行程式碼升級或組態升級 (在此案例中，我們兩者皆進行)，以升級 Azure 上的 Service Fabric Mesh 應用程式，請在 Visual Studio 中的 [todolistapp] 按一下滑鼠右鍵，然後選取 [發佈...]

接著，您將會看見 [發佈 Service Fabric 應用程式] 對話方塊。

![Visual Studio 的 Service Fabric Mesh 發佈對話方塊](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

選取您的 Azure 帳戶和訂用帳戶。 請確認**位置**已設為最初將待辦事項應用程式發佈到 Azure 時，所使用的位置。 本文使用**美國東部**。

請確認**資源群組**已設為最初將待辦事項應用程式發佈至 Azure 時，所使用的資源群組。

請確認 **Azure Container Registry** 已設為最初將待辦事項應用程式發佈至 Azure 時，所建立的 Azure Container Registry 名稱。

在 [發佈] 對話方塊中，按下 [發佈] 按鈕，以升級 Azure 上的待辦事項應用程式。

您可以在 Visual Studio 的 [輸出] 視窗中，選取 [Service Fabric 工具] 窗格，藉以監視升級的進度。 一旦升級完成之後，**Service Fabric 工具**輸出將會以 URL 形式來顯示應用程式的 IP 位址和連接埠。

```json
Packaging Application...
Building Images...
Web1 -> C:\Code\ServiceFabricMeshApp\ToDoService\bin\Any CPU\Release\netcoreapp2.0\ToDoService.dll
Uploading the images to Azure Container Registy...
Deploying application to remote endpoint...
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

開啟網頁瀏覽器，並瀏覽至 URL 以檢視在 Azure 中執行的網站。 現在，您應該會看到包含類別資料行的網頁。

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解：
> [!div class="checklist"]
> * 如何使用 Visual Studio 升級 Service Fabric Mesh 應用程式

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [清除 Service Fabric Mesh 資源](service-fabric-mesh-tutorial-cleanup-resources.md)