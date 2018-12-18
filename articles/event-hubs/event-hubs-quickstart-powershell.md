---
title: Azure 快速入門 - 使用 PowerShell 建立事件中樞 | Microsoft Docs
description: 本快速入門說明如何使用 Azure PowerShell 建立事件中樞，然後使用 .NET Standard SDK 傳送和接收事件。
services: event-hubs
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: 305776db1d3e0bacc266e514e0a59fe6b3fbd4b4
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388522"
---
# <a name="quickstart-create-an-event-hub-using-azure-powershell"></a>快速入門：使用 Azure PowerShell 建立事件中樞

Azure 事件中樞是可高度調整的資料串流平台和擷取服務，每秒可接收和處理數百萬個事件。 本快速入門將示範如何使用 Azure PowerShell 建立事件中樞，然後使用 .NET Standard SDK 在事件中樞中進行傳送和接收。

若要完成本快速入門，您需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前[建立免費帳戶][]。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您具有下列項目︰

- [Visual Studio 2017 Update 3 (版本 15.3, 26730.01)](http://www.visualstudio.com/vs) 或更新版本。
- [.NET Standard SDK](https://www.microsoft.com/net/download/windows)，2.0 版或更新版本。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您在本機使用 PowerShell，您必須執行最新版的 PowerShell，才能完成本快速入門。 如果您需要安裝或升級，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.7.0)。

## <a name="provision-resources"></a>佈建資源

### <a name="create-a-resource-group"></a>建立資源群組

資源群組是 Azure 資源的邏輯集合，而您需要資源群組來建立事件中樞。 

下列範例會在美國東部區域中建立一個資源群組。 以您要使用的資源群組名稱取代 `myResourceGroup`：

```azurepowershell-interactive
New-AzureRmResourceGroup –Name myResourceGroup –Location eastus
```

### <a name="create-an-event-hubs-namespace"></a>建立事件中樞命名空間

建立您的資源群組之後，在該資源群組內建立事件中樞命名空間。 事件中樞命名空間提供唯一的完整網域名稱，您可以在其中建立事件中樞。 以命名空間的唯一名稱取代 `namespace_name`：

```azurepowershell-interactive
New-AzureRmEventHubNamespace -ResourceGroupName myResourceGroup -NamespaceName namespace_name -Location eastus
```

### <a name="create-an-event-hub"></a>建立事件中樞

現在，您已有事件中樞命名空間，接著在該命名空間內建立事件中樞：

```azurepowershell-interactive
New-AzureRmEventHub -ResourceGroupName myResourceGroup -NamespaceName namespace_name -EventHubName eventhub_name
```

### <a name="create-a-storage-account-for-event-processor-host"></a>建立事件處理器主機的儲存體帳戶

事件處理器主機可透過管理檢查點以及平行接收，來簡化來自事件中樞的事件接收作業。 若要使用檢查點，事件處理器主機需要儲存體帳戶。 若要建立儲存體帳戶並取得其金鑰，請執行下列命令：

```azurepowershell-interactive
# Create a standard general purpose storage account 
New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name storage_account_name -Location eastus -SkuName Standard_LRS 
e
# Retrieve the storage account key for accessing it
Get-AzureRmStorageAccountKey -ResourceGroupName myResourceGroup -Name storage_account_name
```

### <a name="get-the-connection-string"></a>取得連接字串

必須有連接字串才能連線到事件中樞及處理事件。 若要取得您的連接字串，請執行：

```azurepowershell-interactive
Get-AzureRmEventHubKey -ResourceGroupName myResourceGroup -NamespaceName namespace_name -Name RootManageSharedAccessKey
```

## <a name="stream-into-event-hubs"></a>串流至事件中樞

您現在可以開始串流至事件中樞。 您可以從[事件中樞存放庫](https://github.com/Azure/azure-event-hubs)下載範例，或複製 Git

### <a name="ingest-events"></a>內嵌事件

若要開始串流事件，請從 GitHub 下載 [SampleSender](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender)，或透過發出下列命令來複製[事件中樞的 GitHub 存放庫](https://github.com/Azure/azure-event-hubs)：

```bash
git clone https://github.com/Azure/azure-event-hubs.git
```

瀏覽至 \azure-event-hubs\samples\DotNet\Microsoft.Azure.EventHubs\SampleSender 資料夾，然後將 SampleSender.sln 檔案載入到 Visual Studio。

接下來，將 [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) Nuget 套件新增至專案。

在 Program.cs 檔案中，用您事件中樞名稱和連接字串取代下列預留位置：

```C#
private const string EhConnectionString = "Event Hubs connection string";
private const string EhEntityPath = "Event Hub name";

```

現在，建置並執行範例。 您可以看到事件正在內嵌至事件中樞：

![][3]

### <a name="receive-and-process-events"></a>接收並處理事件

現在，下載事件處理器主機接收者範例來接收您剛傳送的訊息。 從 GitHub 下載 [SampleEphReceiver](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver)，或透過發出下列命令來複製[事件中樞的 GitHub 存放庫](https://github.com/Azure/azure-event-hubs)：

```bash
git clone https://github.com/Azure/azure-event-hubs.git
```

瀏覽至 \azure-event-hubs\samples\DotNet\Microsoft.Azure.EventHubs\SampleEphReceiver 資料夾，然後將 SampleEphReceiver.sln 解決方案檔案載入到 Visual Studio。

接下來，將 [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) 和 [Microsoft.Azure.EventHubs.Processor](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) NuGet 套件新增至專案。

在 Program.cs 檔案中，以常數的對應值來取代下列常數：

```C#
private const string EventHubConnectionString = "Event Hubs connection string";
private const string EventHubName = "Event Hub name";
private const string StorageContainerName = "Storage account container name";
private const string StorageAccountName = "Storage account name";
private const string StorageAccountKey = "Storage account key";
```

現在，建置並執行範例。 您可以看見範例應用程式正在接收事件：

![][4]

在 Azure 入口網站中，您可以檢視指定事件中樞命名空間的事件處理速率，如下所示：

![][5]

## <a name="clean-up-resources"></a>清除資源

當您完成本快速入門時，您可以刪除資源群組和命名空間，以及儲存體帳戶和其中的事件中樞。 以您建立的資源群組名稱取代 `myResourceGroup`。 

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本文中，您已建立從事件中樞傳送和接收事件所需的事件中樞命名空間和其他資源。 若要深入了解，請繼續下列教學課程：

> [!div class="nextstepaction"]
> [在事件中樞資料流上將資料異常視覺化](event-hubs-tutorial-visualize-anomalies.md)

[建立免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[Install and Configure Azure PowerShell]: https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[New-AzureRmResourceGroup]: https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[3]: ./media/event-hubs-quickstart-powershell/sender1.png
[4]: ./media/event-hubs-quickstart-powershell/receiver1.png
[5]: ./media/event-hubs-quickstart-powershell/metrics.png
