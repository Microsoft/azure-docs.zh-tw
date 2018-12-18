---
title: 使用 PowerShell 複製 Web 應用程式
description: 了解如何使用 PowerShell，將您的 Web Apps 複製到新的 Web Apps。
services: app-service\web
documentationcenter: ''
author: ahmedelnably
manager: stefsch
editor: ''
ms.assetid: f9a5cfa1-fbb0-41e6-95d1-75d457347a35
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/14/2016
ms.author: aelnably
ms.openlocfilehash: 30817a1a6a8079e7a896305ab0b59e48fad4d644
ms.sourcegitcommit: f1c1789f2f2502d683afaf5a2f46cc548c0dea50
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/18/2018
ms.locfileid: "27867465"
---
# <a name="azure-app-service-app-cloning-using-powershell"></a>使用 PowerShell 複製 Azure App Service App
隨著 Microsoft Azure PowerShell 1.1.0 版的發行，`New-AzureRMWebApp` 中新增了一個選項，可讓您將現有的 Web 應用程式複製到在不同區域或在相同區域中新建立的應用程式。 此選項可讓客戶輕鬆且快速地跨不同區域部署許多應用程式。

應用程式複製目前僅支援 Premium 層 App Service 方案。 新的功能使用與 Web Apps 備份功能相同的限制，請參閱 [在 Azure App Service 中備份 Web 應用程式](web-sites-backup.md)。

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="cloning-an-existing-app"></a>複製現有的應用程式
案例：您想要將位在美國中南部區域的現有 Web 應用程式內容，複製到位在美國中北部區域的新 Web 應用程式。 使用 Azure Resource Manager 版本的 PowerShell Cmdlet 並搭配 `-SourceWebApp` 選項建立新的 Web 應用程式，即可實現此目的。

若知道包含來源 Web 應用程式的資源群組名稱，您即可使用下列 PowerShell 命令來取得來源 Web 應用程式的資訊 (在此例中名為 `source-webapp`)：

```PowerShell
$srcapp = Get-AzureRmWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

若要建立新的 App Service 方案，您可以使用 `New-AzureRmAppServicePlan` 命令，如下列範例所示。

```PowerShell
New-AzureRmAppServicePlan -Location "South Central US" -ResourceGroupName DestinationAzureResourceGroup -Name NewAppServicePlan -Tier Premium
```

使用 `New-AzureRmWebApp` 命令，就能在美國中北部區域建立新的 Web 應用程式，並將它繫結到現有的進階層 App Service 方案。 此外，您可以使用與來源 Web 應用程式相同的資源群組，也可以定義新的資源群組，如下列命令所示：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp
```

若要複製現有的 Web 應用程式 (包括所有相關聯的部署位置)，您必須使用 `IncludeSourceWebAppSlots` 參數。 下列 PowerShell 命令示範如何搭配 `New-AzureRmWebApp` 命令使用該參數：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots
```

若要複製相同區域內的現有 Web 應用程式，您必須在相同區域中建立新的資源群組和新的 App Service 方案，然後使用下列 PowerShell 命令來複製 Web 應用程式

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan NewAppServicePlan -SourceWebApp $srcap
```

## <a name="cloning-an-existing-app-to-an-app-service-environment"></a>複製現有應用程式至 App Service 環境
案例：您想要將位在美國中南部區域的現有 Web 應用程式內容，複製到現有 App Service Environment (ASE) 的新 Web 應用程式中。

若知道包含來源 Web 應用程式的資源群組名稱，您即可使用下列 PowerShell 命令來取得來源 Web 應用程式的資訊 (在此例中名為 `source-webapp`)：

```PowerShell
$srcapp = Get-AzureRmWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

若知道 ASE 的名稱和 ASE 所屬的資源群組名稱，您即可在現有 ASE 中建立新的 Web 應用程式，如下列命令所示：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -ASEName DestinationASE -ASEResourceGroupName DestinationASEResourceGroupName -SourceWebApp $srcapp
```

由於舊版因素，因此需有 `Location` 參數，但在 ASE 中建立應用程式時，則會忽略此參數。 

## <a name="cloning-an-existing-app-slot"></a>複製現有的應用程式位置
案例：您想要將現有 Web 應用程式位置複製到新的 Web 應用程式或新的 Web 應用程式位置。 新的 Web 應用程式可以在與原始的 Web 應用程式位置位於相同區域或不同區域中。

若知道包含來源 Web 應用程式的資源群組名稱，您即可使用下列 PowerShell 命令來取得來源 Web 應用程式位置繫結至 `source-webapp` Web 應用程式的資訊 (在此例中名為 `source-webappslot`)：

```PowerShell
$srcappslot = Get-AzureRmWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-webapp -Slot source-webappslot
```

下列命令示範如何建立從來源 Web 應用程式至新 Web 應用程式的複製作業：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot
```

## <a name="configuring-traffic-manager-while-cloning-an-app"></a>在複製應用程式時設定流量管理員
建立多區域 Web 應用程式並將 Azure 流量管理員設定為將流量路由傳送到所有 Web 應用程式，是確保客戶的應用程式具有高可用性的重要案例。 在複製現有 Web 應用程式時，您可以選擇將這兩個 Web 應用程式連線到新的流量管理員設定檔或現有設定檔。 只支援 Azure Resource Manager 版本的流量管理員。

### <a name="creating-a-new-traffic-manager-profile-while-cloning-an-app"></a>在複製應用程式時建立新流量管理員設定檔
案例：您想要將 Web 應用程式複製到另一個區域，同時設定包含這兩個 Web 應用程式的 Azure Resource Manager 流量管理員設定檔。 下列命令示範如何建立從來源 Web 應用程式至新 Web 應用程式的複製作業，同時設定新的流量管理員設定檔：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile
```

### <a name="adding-new-cloned-web-app-to-an-existing-traffic-manager-profile"></a>加入新複製的 Web 應用程式至現有的流量管理員設定檔
案例：您已經擁有 Azure Resource Manager 流量管理員設定檔，如今想要將這兩個 Web 應用程式新增為端點。 若要這麼做，您必須先組合現有的流量管理員設定檔識別碼。 您需要訂用帳戶識別碼、資源群組名稱和現有流量管理員設定檔名稱。

```PowerShell
$TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"
```

在擁有流量管理員識別碼後，下列命令會示範如何建立從來源 Web 應用程式至新 Web 應用程式的複製作業，同時將這兩個 Web 應用程式新增至現有的流量管理員設定檔：

```PowerShell
$destapp = New-AzureRmWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID
```

## <a name="current-restrictions"></a>目前的限制
這項功能目前為預覽狀態，日後會逐漸加入新功能。 以下是現行應用程式複製版本的已知限制：

* 不會複製自動調整設定
* 不會複製備份排程設定
* 不會複製 VNET 設定
* 未自動在目的地 Web 應用程式上設定 App Insights 
* 不會複製簡單驗證設定
* 不會複製 Kudu 延伸模組
* 不會複製 TiP 規則
* 不會複製資料庫內容
* 如果複製到不同縮放單位，輸出 IP 位址將會變更

### <a name="references"></a>參考
* [Web 應用程式複製](app-service-web-app-cloning.md)
* [在 Azure App Service 中備份 Web 應用程式](web-sites-backup.md)
* [Azure Resource Manager 的 Azure 流量管理員支援預覽](../traffic-manager/traffic-manager-powershell-arm.md)
* [App Service 環境簡介](environment/intro.md)
* [搭配使用 Azure PowerShell 與 Azure 資源管理員](../azure-resource-manager/powershell-azure-resource-manager.md)

