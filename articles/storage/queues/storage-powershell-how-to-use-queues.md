---
title: 在使用 PowerShell 的 Azure 佇列儲存體上執行作業 | Microsoft Docs
description: 如何在使用 PowerShell 的 Azure 佇列儲存體上執行作業
services: storage
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/14/2017
ms.author: rogarana
ms.component: queues
ms.openlocfilehash: 2f347ef0bd53a8d148a56d8431e004e9e88894d8
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128566"
---
# <a name="perform-azure-queue-storage-operations-with-azure-powershell"></a>使用 Azure PowerShell 執行 Azure 佇列儲存體作業

Azure 佇列儲存體是一項儲存大量訊息的服務，全球任何地方都可透過 HTTP 或 HTTPS 叫來存取這些訊息。 如需詳細資訊，請參閱 [Azure 佇列簡介](storage-queues-introduction.md)。 本做法文章涵蓋一般的佇列儲存體作業。 您會了解如何：

> [!div class="checklist"]
> * 建立佇列
> * 擷取佇列
> * 新增訊息
> * 讀取訊息
> * 刪除訊息 
> * 刪除佇列

本做法需要 Azure PowerShell 模組 3.6 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

沒有任何適用於佇列資料層的 PowerShell Cmdlet。 若要執行資料層作業 (例如新增訊息、讀取訊息，以及刪除訊息)，您必須使用 PowerShell 中公開的 .NET 儲存體用戶端程式庫。 您可以建立訊息物件，然後使用 AddMessage 這類命令對該訊息執行作業。 本文示範如何執行這項作業。

## <a name="sign-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Connect-AzureRmAccount
```

## <a name="retrieve-list-of-locations"></a>擷取位置清單

如果您不知道要使用哪一個位置，您可以列出可用的位置。 當清單顯示之後，尋找您想要使用的位置。 這個練習將使用 **eastus**。 將其儲存於變數 **location** 中供未來使用。

```powershell
Get-AzureRmLocation | select Location 
$location = "eastus"
```

## <a name="create-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立資源群組。 

Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 在變數中儲存資源群組名稱供日後使用。 在此範例中，會在 *eastus* 區域中建立名為 *howtoqueuesrg* 的資源群組。

```powershell
$resourceGroup = "howtoqueuesrg"
New-AzureRmResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>建立儲存體帳戶

搭配使用本地備援儲存體 (LRS) 與 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/New-AzureRmStorageAccount) 來建立標準的一般用途儲存體帳戶。 取得儲存體帳戶內容，定義要使用的儲存體帳戶。 使用儲存體帳戶時，會參考內容而非重複提供認證。

```powershell
$storageAccountName = "howtoqueuestorage"
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS

$ctx = $storageAccount.Context
```

## <a name="create-a-queue"></a>建立佇列

下列範例會先使用儲存體帳戶內容建立 Azure 儲存體的連線，其中包含儲存體帳戶名稱及其存取金鑰 。 接著，它會呼叫 [New-AzureStorageQueue (英文)](/powershell/module/azure.storage/new-azurestoragequeue) Cmdlet 以建立名為 'queuename' 的佇列。

```powershell
$queueName = "howtoqueue"
$queue = New-AzureStorageQueue –Name $queueName -Context $ctx
```

如需 Azure 佇列服務命名慣例的資訊，請參閱 [為佇列和中繼資料命名](http://msdn.microsoft.com/library/azure/dd179349.aspx)。

## <a name="retrieve-a-queue"></a>擷取佇列

您可以查詢與擷取儲存體帳戶中的特定佇列或所有佇列清單。 下列範例示範如何擷取儲存體帳戶中的所有佇列和特定的佇列；這兩個命令使用 [Get-AzureStorageQueue](/powershell/module/azure.storage/get-azurestoragequeue) cmdlet。

```powershell
# Retrieve a specific queue
$queue = Get-AzureStorageQueue –Name $queueName –Context $ctx
# Show the properties of the queue
$queue

# Retrieve all queues and show their names
Get-AzureStorageQueue -Context $ctx | select Name
```

## <a name="add-a-message-to-a-queue"></a>將訊息新增至佇列

影響佇列中實際訊息的作業會使用 PowerShell 中公開的 .NET 儲存體用戶端程式庫。 若要將訊息新增至佇列，請建立 [Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage](http://msdn.microsoft.com/library/azure/jj732474.aspx) 類別，這個訊息物件的新執行個體。 接著，呼叫 [AddMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.addmessage.aspx) 方法。 您可以從字串 (採用 UTF-8 格式) 或位元組陣列建立 CloudQueueMessage。

下列範例示範如何將訊息新增至佇列。

```powershell
# Create a new message using a constructor of the CloudQueueMessage class
$queueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage `
  -ArgumentList "This is message 1"
# Add a new message to the queue
$queue.CloudQueue.AddMessage($QueueMessage)

# Add two more messages to the queue 
$queueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage `
  -ArgumentList "This is message 2"
$queue.CloudQueue.AddMessage($QueueMessage)
$queueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage `
  -ArgumentList "This is message 3"
$queue.CloudQueue.AddMessage($QueueMessage)
```

如果您使用 [Azure 儲存體總管](http://storageexplorer.com)，則可以連線到您的 Azure 帳戶，以及檢視儲存體帳戶中的佇列，並向下鑽研至某個佇列以檢視佇列上的訊息。 

## <a name="read-a-message-from-the-queue-then-delete-it"></a>讀取佇列的訊息，然後刪除它

會嘗試以最佳的先進先出順序讀取訊息。 不保證會採用此作法。 當您從佇列讀取訊息時，查看佇列的其他處理序會看不到它。 這可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。  

此**不可見逾時**定義在重新開放處理前，訊息維持不可見的時間。 預設值為 30 秒。 

您的程式碼可使用兩個步驟，從佇列讀取訊息。 當您呼叫 [Microsoft.WindowsAzure.Storage.Queue.CloudQueue.GetMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.getmessage.aspx) 方法，會取得佇列中的下一個訊息。 從 **GetMessage** 傳回的訊息，對於從此佇列讀取訊息的任何其他程式碼而言將會是不可見的。 若要完成從佇列移除訊息的動作，可呼叫 [Microsoft.WindowsAzure.Storage.Queue.CloudQueue.DeleteMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.deletemessage.aspx) 方法。 

在下列範例中，您可以閱讀三個佇列訊息，然後等待 10 秒 (不可見逾時)。 接著重新讀取三個訊息，然後呼叫 **DeleteMessage** 來刪除讀取後的訊息。 如果嘗試在刪除訊息後讀取佇列，會將 $queueMessage 傳回成 NULL。

```powershell
# Set the amount of time you want to entry to be invisible after read from the queue
# If it is not deleted by the end of this time, it will show up in the queue again
$invisibleTimeout = [System.TimeSpan]::FromSeconds(10)

# Read the message from the queue, then show the contents of the message. Read the other two messages, too.
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queueMessage 
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queueMessage 
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queueMessage 

# After 10 seconds, these messages reappear on the queue. 
# Read them again, but delete each one after reading it.
# Delete the message.
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queue.CloudQueue.DeleteMessage($queueMessage)
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queue.CloudQueue.DeleteMessage($queueMessage)
$queueMessage = $queue.CloudQueue.GetMessage($invisibleTimeout)
$queue.CloudQueue.DeleteMessage($queueMessage)
```

## <a name="delete-a-queue"></a>刪除佇列
若要刪除佇列及其內含的所有訊息，請呼叫 Remove-AzureStorageQueue Cmdlet。 下列範例示範如何使用 Remove-AzureStorageQueue Cmdlet 來刪除本練習所使用的特定佇列。

```powershell
# Delete the queue 
Remove-AzureStorageQueue –Name $queueName –Context $ctx
```

## <a name="clean-up-resources"></a>清除資源

若要移除您在這個練習中建立的所有資產，請移除此資源群組。 這會同時刪除群組內含的所有資源。 在本例中，它會移除建立的儲存體帳戶和資源群組本身。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>後續步驟

在本做法文章中，您學會使用 PowerShell 進行基本佇列儲存體管理，包括如何：

> [!div class="checklist"]
> * 建立佇列
> * 擷取佇列
> * 新增訊息
> * 讀取下一個訊息
> * 刪除訊息 
> * 刪除佇列

### <a name="microsoft-azure-powershell-storage-cmdlets"></a>Microsoft Azure PowerShell 儲存體 Cmdlet
* [儲存體 PowerShell Cmdlet](/powershell/module/azurerm.storage#storage)

### <a name="microsoft-azure-storage-explorer"></a>Microsoft Azure 儲存體總管
* [Microsoft Azure 儲存體總管](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。
