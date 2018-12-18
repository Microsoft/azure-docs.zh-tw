---
title: 使用 Azure IoT 中樞設定訊息路由 (.NET) | Microsoft Docs
description: 使用 Azure IoT 中樞設定訊息路由
author: robinsh
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 09/11/2018
ms.author: robinsh
ms.custom: mvc
ms.openlocfilehash: 575c8a5bec4c7763c75154835830ba350f009e93
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946930"
---
# <a name="tutorial-configure-message-routing-with-iot-hub"></a>教學課程：使用 IoT 中樞設定訊息路由

[訊息路由](iot-hub-devguide-messages-d2c.md)可讓您將遙測資料從 IoT 裝置傳送至內建事件中樞相容端點或自訂端點 (例如 Blob 儲存體、服務匯流排佇列、服務匯流排主題和事件中樞)。 設定訊息路由時，您可以建立[路由查詢](iot-hub-devguide-routing-query-syntax.md)來自訂符合特定條件的路由。 設定好之後，IoT 中樞就會自動將內送資料路由至端點。 

在本教學課程中，您將了解如何透過 IoT 中樞設定和使用路由查詢。 您會將訊息從 IoT 裝置路由至多個服務的其中一個，包括 Blob 儲存體和服務匯流排佇列。 邏輯應用程式將會挑選傳送至服務匯流排佇列的訊息，並透過電子郵件傳送。 沒有特別設定路由的訊息會傳送至預設端點，並在 Power BI 視覺效果中加以檢視。

在本教學課程中，您會執行下列工作：

> [!div class="checklist"]
> * 使用 Azure CLI 或 PowerShell 設定基礎資源 -- IoT 中樞、儲存體帳戶、服務匯流排佇列和模擬裝置。
> * 在 IoT 中樞內設定儲存體帳戶和服務匯流排佇列的端點及路由。
> * 建立邏輯應用程式，使其在訊息新增至服務匯流排佇列時觸發並傳送電子郵件。
> * 下載並執行模擬 IoT 裝置的應用程式，針對不同路由選項將訊息傳送至中樞。
> * 為傳送至預設端點的資料建立 Power BI 視覺效果。
> * 檢視結果...
> * ...在服務匯流排佇列和電子郵件中。
> * ...在儲存體帳戶中。
> * ...在 Power BI 視覺效果中。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

- 安裝 [Visual Studio](https://www.visualstudio.com/)。 

- PowerBI 帳戶，用來分析預設端點的串流分析。 ([免費試用 Power BI](https://app.powerbi.com/signupredirect?pbi_source=web)。)

- 用來傳送通知電子郵件的 Office 365 帳戶。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-resources"></a>設定資源

在此教學課程中，您需要 IoT 中樞、儲存體帳戶和服務匯流排佇列。 這些資源可使用 Azure CLI 或 Azure PowerShell 來建立。 請將所有資源建立在相同的資源群組和位置。 如此一來，您就可以在結束時，透過刪除資源群組的單一步驟來移除所有項目。

下列各節會說明如何執行這些必要步驟。 請遵循 CLI 或 PowerShell 指示。

1. 建立[資源群組](../azure-resource-manager/resource-group-overview.md)。 

2. 在 S1 層建立 IoT 中樞。 將取用者群組新增至 IoT 中樞。 Azure 串流分析會在擷取資料時使用取用者群組。

3. 透過 Standard_LRS 複寫來建立標準 V1 儲存體帳戶。

4. 建立服務匯流排命名空間與佇列。 

5. 為傳送訊息至中樞的模擬裝置建立裝置身分識別。 儲存測試階段的金鑰。

### <a name="set-up-your-resources-using-azure-cli"></a>使用 Azure CLI 設定資源

複製此指令碼並貼到 Cloud Shell。 如果您已登入，它會一次執行一行指令碼。 

必須是全域唯一的變數會與 `$RANDOM` 串連。 執行指令碼並設定變數時，系統會產生隨機數字字串，並將其串連至固定字串的結尾，使其成為唯一。

```azurecli-interactive

# This is the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity. 
az extension add --name azure-cli-iot-ext

# Set the values for the resource names that don't have to be globally unique.
# The resources that have to have unique names are named in the script below
#   with a random number concatenated to the name so you can probably just
#   run this script, and it will work with no conflicts.
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults
iotDeviceName=Contoso-Test-Device 

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, so add a random number to the end.
iotHubName=ContosoTestHub$RANDOM
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# Add a consumer group to the IoT hub.
az iot hub consumer-group create --hub-name $iotHubName \
    --name $iotHubConsumerGroup

# The storage account name must be globally unique, so add a random number to the end.
storageAccountName=contosostorage$RANDOM
echo "Storage account name = " $storageAccountName

# Create the storage account to be used as a routing destination.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Get the primary storage account key. 
#    You need this to create the container.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"') 

# See the value of the storage account key.
echo "$storageAccountKey"

# Create the container in the storage account. 
az storage container create --name $containerName \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off 

# The Service Bus namespace must be globally unique, so add a random number to the end.
sbNameSpace=ContosoSBNamespace$RANDOM
echo "Service Bus namespace = " $sbNameSpace

# Create the Service Bus namespace.
az servicebus namespace create --resource-group $resourceGroup \
    --name $sbNameSpace \
    --location $location
    
# The Service Bus queue name must be globally unique, so add a random number to the end.
sbQueueName=ContosoSBQueue$RANDOM
echo "Service Bus queue name = " $sbQueueName

# Create the Service Bus queue to be used as a routing destination.
az servicebus queue create --name $sbQueueName \
    --namespace-name $sbNameSpace \
    --resource-group $resourceGroup

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName

```

### <a name="set-up-your-resources-using-azure-powershell"></a>使用 Azure PowerShell 設定資源

複製此指令碼並貼到 Cloud Shell。 如果您已登入，它會一次執行一行指令碼。

必須是全域唯一的變數會與 `$(Get-Random)` 串連。 執行指令碼並設定變數時，系統會產生隨機數字字串，並將其串連至固定字串的結尾，使其成為唯一。

```azurepowershell-interactive
# Log into Azure account.
Login-AzureRMAccount

# Set the values for the resource names that don't have to be globally unique.
# The resources that have to have unique names are named in the script below
#   with a random number concatenated to the name so you can probably just
#   run this script, and it will work with no conflicts.
$location = "West US"
$resourceGroup = "ContosoResources"
$iotHubConsumerGroup = "ContosoConsumers"
$containerName = "contosoresults"
$iotDeviceName = "Contoso-Test-Device"

# Create the resource group to be used 
#   for all resources for this tutorial.
New-AzureRmResourceGroup -Name $resourceGroup -Location $location

# The IoT hub name must be globally unique, so add a random number to the end.
$iotHubName = "ContosoTestHub$(Get-Random)"
Write-Host "IoT hub name is " $iotHubName

# Create the IoT hub.
New-AzureRmIotHub -ResourceGroupName $resourceGroup `
    -Name $iotHubName `
    -SkuName "S1" `
    -Location $location `
    -Units 1

# Add a consumer group to the IoT hub for the 'events' endpoint.
Add-AzureRmIotHubEventHubConsumerGroup -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EventHubConsumerGroupName $iotHubConsumerGroup `
  -EventHubEndpointName "events"

# The storage account name must be globally unique, so add a random number to the end.
$storageAccountName = "contosostorage$(Get-Random)"
Write-Host "storage account name is " $storageAccountName

# Create the storage account to be used as a routing destination.
# Save the context for the storage account 
#   to be used when creating a container.
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind Storage
$storageContext = $storageAccount.Context 

# Create the container in the storage account.
New-AzureStorageContainer -Name $containerName `
    -Context $storageContext

# The Service Bus namespace must be globally unique,
#   so add a random number to the end.
$serviceBusNamespace = "ContosoSBNamespace$(Get-Random)"
Write-Host "Service Bus namespace is " $serviceBusNamespace

# Create the Service Bus namespace.
New-AzureRmServiceBusNamespace -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name $serviceBusNamespace 

# The Service Bus queue name must be globally unique,
#  so add a random number to the end.
$serviceBusQueueName  = "ContosoSBQueue$(Get-Random)"
Write-Host "Service Bus queue name is " $serviceBusQueueName 

# Create the Service Bus queue to be used as a routing destination.
New-AzureRmServiceBusQueue -ResourceGroupName $resourceGroup `
    -Namespace $serviceBusNamespace `
    -Name $serviceBusQueueName 

```

接下來，建立裝置身分識別並儲存金鑰以供稍後使用。 模擬應用程式會使用此裝置身分識別來將訊息傳送到 IoT 中樞。 這項功能不能在 PowerShell 中使用，但是您可以在 [Azure 入口網站](https://portal.azure.com)中建立裝置。

1. 開啟 [Azure 入口網站](https://portal.azure.com)並登入您的 Azure 帳戶。

2. 按一下 [資源群組] 來選取資源群組。 本教學課程使用 **ContosoResources**。

3. 在資源清單中，按一下您的 IoT 中樞。 本教學課程使用 **ContosoTestHub**。 在 [中樞] 窗格中選取 [IoT 裝置]。

4. 按一下 [+ 新增]。 在 [新增裝置] 窗格中，填入裝置識別碼。 本教學課程使用 **Contoso-Test-Device**。 金鑰的部分保留空白，並勾選 [自動產生金鑰]。 請確定 [將裝置連線到 IoT 中樞] 已啟用。 按一下 [檔案] 。

   ![顯示新增裝置畫面的螢幕擷取畫面。](./media/tutorial-routing/add-device.png)

5. 現在金鑰已建立，請按一下裝置來查看產生的金鑰。 按一下主要金鑰上的 [複製] 圖示，並將它儲存在「記事本」之類的位置中，以便在本教學課程的測試階段中使用。

   ![顯示裝置詳細資訊 (包括金鑰) 的螢幕擷取畫面。](./media/tutorial-routing/device-details.png)

## <a name="set-up-message-routing"></a>設定訊息路由

您將根據由模擬裝置附加至訊息的屬性，將訊息路由至不同的資源。 沒有自訂路由的訊息會傳送至預設的端點 (訊息/事件)。 

|value |結果|
|------|------|
|level="storage" |寫入至 Azure 儲存體。|
|level="critical" |寫入至服務匯流排佇列。 邏輯應用程式會從佇列擷取訊息，並使用 Office 365 以電子郵件傳送訊息。|
|預設值 |使用 Power BI 顯示此資料。|

### <a name="routing-to-a-storage-account"></a>路由至儲存體帳戶 

現在來設定儲存體帳戶的路由。 移至 [訊息路由] 窗格，然後新增路由。 在新增路由時，定義新的路由端點。 設定好之後，其 **level** 屬性設定為 **storage** 的訊息會自動寫入儲存體帳戶。

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [資源群組]，然後選取您的資源群組。 本教學課程使用 **ContosoResources**。 

2. 按一下資源清單下方的 [IoT 中樞]。 本教學課程使用 **ContosoTestHub**。 

3. 按一下 [訊息路由]。 在 [訊息路由] 窗格中，按一下 [+新增]。 在 [新增路由] 窗格中，按一下 [端點] 欄位旁的 [+新增]，如下圖所示：

   ![顯示如何開始對路由新增端點的螢幕擷取畫面。](./media/tutorial-routing/message-routing-add-a-route-w-storage-ep.png)

4. 選取 [Blob 儲存體]。 您會看到 [新增儲存體端點] 窗格。 

   ![顯示新增端點的螢幕擷取畫面。](./media/tutorial-routing/message-routing-add-storage-ep.png)

5. 輸入端點的名稱。 本教學課程使用 **StorageContainer**。

6. 按一下 [挑選容器]。 這會帶您前往儲存體帳戶清單。 選取您在準備步驟中設定的帳戶。 本教學課程使用 **contosostorage**。 它會顯示該儲存體帳戶中的容器清單。 選取您在準備步驟中設定的容器。 本教學課程使用 **contosoresults**。 按一下 [選取] 。 您會返回 [新增端點] 窗格。 

7. 對其餘欄位使用預設值。 按一下 [建立] 來建立儲存體端點，並將它新增至路由。 您會返回 [新增路由] 窗格。

8.  現在，請完成其餘路由查詢資訊。 此查詢會指定準則，來規範如何將訊息傳送至您剛才新增為端點的儲存體容器。 填寫畫面上的欄位。 

   **名稱**：輸入路由查詢的名稱。 本教學課程使用 **StorageRoute**。

   **端點**：這會顯示您剛才設定的端點。 
   
   **資料來源**：從下拉式清單選取 [裝置遙測訊息]。

   **啟用路由**：務必啟用此選項。
   
   **路由查詢**：輸入 `level="storage"` 作為查詢字串。 

   ![顯示建立儲存體帳戶路由查詢的螢幕擷取畫面。](./media/tutorial-routing/message-routing-finish-route-storage-ep.png)  
   
   按一下 [檔案] 。 完成時會返回 [訊息路由] 窗格，您可以在其中看到儲存體的新路由查詢。 關閉 [路由] 窗格即會返回 [資源群組] 頁面。

### <a name="routing-to-a-service-bus-queue"></a>路由至服務匯流排佇列 

現在來設定服務匯流排佇列的路由。 移至 [訊息路由] 窗格，然後新增路由。 在新增路由時，定義新的路由端點。 設定好之後，其 **level** 屬性設定為 **critical** 的訊息會寫入至服務匯流排佇列中，然後觸發邏輯應用程式，並以電子郵件傳送資訊。 

1. 在 [資源群組] 頁面上，按一下 IoT 中樞，然後按一下 [訊息路由]。 

2. 在 [訊息路由] 窗格中，按一下 [+新增]。 

3. 在 [新增路由] 窗格中，按一下 [端點] 欄位旁的 [+新增]。 選取 [服務匯流排佇列]。 您會看到 [新增服務匯流排端點] 窗格。 

   ![新增服務匯流排端點的螢幕擷取畫面](./media/tutorial-routing/message-routing-add-sbqueue-ep.png)

4. 填寫欄位：

   **端點名稱**：輸入端點的名稱。 本教學課程使用 **CriticalQueue**。
   
   **服務匯流排命名空間**：按一下此欄位以顯示下拉式清單；選取您在準備步驟中設定的服務匯流排命名空間。 本教學課程使用 **ContosoSBNamespace**。

   **服務匯流排佇列**：按一下此欄位以顯示下拉式清單；從下拉式清單選取服務匯流排佇列。 本教學課程使用 **contososbqueue**。

5. 按一下 [建立] 以新增服務匯流排佇列端點。 您會返回 [新增路由] 窗格。 

6.  現在，請完成其餘路由查詢資訊。 此查詢會指定準則，來規範如何將訊息傳送至您剛才新增為端點的服務匯流排佇列。 填寫畫面上的欄位。 

   **名稱**：輸入路由查詢的名稱。 本教學課程使用 **SBQueueRoute**。 

   **端點**：這會顯示您剛才設定的端點。

   **資料來源**：從下拉式清單選取 [裝置遙測訊息]。

   **路由查詢**：輸入 `level="critical"` 作為查詢字串。 

   ![顯示建立服務匯流排佇列路由查詢的螢幕擷取畫面。](./media/tutorial-routing/message-routing-finish-route-sbq-ep.png)

7. 按一下 [檔案] 。 返回 [路由] 窗格中時，您會看到兩個新的路由顯示於此處。

   ![顯示剛才設定的路由螢幕擷取畫面。](./media/tutorial-routing/message-routing-show-both-routes.png)

8. 您可以按一下 [自訂端點] 索引標籤，以看到您所設定的自訂端點。

   ![顯示剛才設定的自訂端點螢幕擷取畫面。](./media/tutorial-routing/message-routing-show-custom-endpoints.png)

9. 關閉 [訊息路由] 窗格即會返回 [資源群組] 窗格。

## <a name="create-a-logic-app"></a>建立邏輯應用程式  

服務匯流排佇列會用來接收指定為 critical 的訊息。 設定邏輯應用程式來監視服務匯流排佇列，並在訊息新增至佇列時傳送電子郵件。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [+ 建立資源]。 在 [搜尋] 方塊中輸入**邏輯應用程式**，然後按一下 Enter。 從顯示的搜尋結果中選取邏輯應用程式，然後按一下 [建立] 繼續前往 [建立邏輯應用程式] 窗格。 填寫欄位。 

   **名稱**︰此欄位是邏輯應用程式的名稱。 本教學課程使用 **ContosoLogicApp**。 

   **訂用帳戶**：選取您的 Azure 訂用帳戶。

   **資源群組**：按一下 [使用現有的] 並選取您的資源群組。 本教學課程使用 **ContosoResources**。 

   **位置**：使用您的位置。 本教學課程使用**美國西部**。 

   **Log Analytics**：此切換應為關閉狀態。 

   ![顯示建立邏輯應用程式畫面的螢幕擷取畫面。](./media/tutorial-routing/create-logic-app.png)

   按一下頁面底部的 [新增] 。

1. 現在請移至邏輯應用程式。 取得邏輯應用程式最簡單的方式是按一下 [資源群組]，選取您的資源群組 (本教學課程使用 **ContosoResources**)，然後從資源清單中選取邏輯應用程式。 Logic Apps 設計工具頁面隨即顯示 (您可能需要捲動至右側才能看到完整頁面)。 在 Logic Apps 設計工具頁面上，向下捲動直到您看到標示 [空白邏輯應用程式 +] 的圖格，然後按一下它。 

1. 連接器清單隨即顯示。 選取 [服務匯流排]。 

   ![顯示連接器清單的螢幕擷取畫面。](./media/tutorial-routing/logic-app-connectors.png)

1. 觸發程序清單隨即顯示。 選取 [服務匯流排 - 在佇列中收到訊息時] \(自動完成\)。 

   ![顯示服務匯流排觸發程序清單的螢幕擷取畫面。](./media/tutorial-routing/logic-app-triggers.png)

1. 在下一個畫面上，填寫連線名稱。 本教學課程使用 **ContosoConnection**。 

   ![顯示設定服務匯流排佇列連線的螢幕擷取畫面。](./media/tutorial-routing/logic-app-define-connection.png)

   按一下服務匯流排命名空間。 本教學課程使用 **ContosoSBNamespace**。 當您選取命名空間時，入口網站會查詢服務匯流排命名空間來擷取金鑰。 選取 [RootManageSharedAccessKey]，然後按一下 [建立]。 
   
   ![顯示完成設定連線的螢幕擷取畫面。](./media/tutorial-routing/logic-app-finish-connection.png)

1. 在下一個畫面上，從下拉式清單中選取佇列的名稱 (本教學課程使用 **contososbqueue**)。 其他欄位可使用預設值。 

   ![顯示佇列選項的螢幕擷取畫面。](./media/tutorial-routing/logic-app-queue-options.png)

1. 現在要設定在佇列中收到訊息時會傳送電子郵件的動作。 在 Logic Apps 設計工具中，按一下 [+ 新增步驟] 來新增步驟，然後按一下 [新增動作]。 在 [選擇動作] 窗格中，尋找並按一下 [Office 365 Outlook]。 在觸發程序畫面上，選取 [Office 365 Outlook - 傳送電子郵件]。  

   ![顯示 Office365 選項的螢幕擷取畫面。](./media/tutorial-routing/logic-app-select-outlook.png)

1. 接下來，請登入 Office 365 帳戶來設定連線。 指定電子郵件收件者的電子郵件地址。 也請指定主旨，然後在本文中輸入要讓收件者看到的訊息。 為了進行測試，請填寫您自己的電子郵件地址作為收件者。

   按一下 [新增動態內容] 會顯示您可以包含的訊息內容。 選取 [內容] -- 訊息將會包含在電子郵件中。 

   ![顯示邏輯應用程式電子郵件選項的螢幕擷取畫面。](./media/tutorial-routing/logic-app-send-email.png)

1. 按一下 [檔案] 。 然後關閉 Logic Apps 設計工具。

## <a name="set-up-azure-stream-analytics"></a>設定 Azure 串流分析

若要在 Power BI 視覺效果中查看資料，請先設定串流分析作業來擷取資料。 請記住，只有 **level** 是 **normal** 的訊息會傳送至預設的端點，並由串流分析作業擷取該訊息以用於 Power BI 視覺效果。

### <a name="create-the-stream-analytics-job"></a>建立串流分析作業

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [建立資源] > [物聯網] > [串流分析作業]。

1. 輸入作業的以下資訊。

   **作業名稱**：作業名稱。 此名稱必須是全域唯一的。 本教學課程使用 **contosoJob**。

   **資源群組**︰使用 IoT 中樞所用的相同資源群組。 本教學課程使用 **ContosoResources**。 

   **位置**：使用安裝指令碼中使用的相同位置。 本教學課程使用**美國西部**。 

   ![顯示如何建立串流分析作業的螢幕擷取畫面。](./media/tutorial-routing/stream-analytics-create-job.png)

1. 按一下 [建立] 來建立作業。 若要回到作業，按一下 [資源群組]。 本教學課程使用 **ContosoResources**。 選取資源群組，然後按一下資源清單中的串流分析工作。 

### <a name="add-an-input-to-the-stream-analytics-job"></a>將輸入新增至串流分析作業

1. 在 [作業拓撲] 之下，按一下 [輸入]。

1. 在 [輸入] 窗格中，按一下 [新增資料流輸入] 並選取 IoT 中樞。 在顯示的畫面上，填寫下列欄位：

   **輸入別名**：本教學課程使用 **contosoinputs**。

   **訂用帳戶**︰選取您的訂用帳戶。

   **IoT 中樞**：選取 IoT 中樞。 本教學課程使用 **ContosoTestHub**。

   **端點**：選取 [傳訊]。 (如果您選取「作業監視」，則您會取得有關 IoT 中樞的遙測資料，而不是您要傳送的資料。) 

   **共用存取原則名稱**：選取 **iothubowner**。 入口網站會為您填入共用存取原則金鑰。

   **取用者群組**：選取您稍早建立的取用者群組。 本教學課程使用 **contosoconsumers**。
   
   對於其餘欄位，請採用預設值。 

   ![顯示如何為串流分析作業設定輸入的螢幕擷取畫面。](./media/tutorial-routing/stream-analytics-job-inputs.png)

1. 按一下 [檔案] 。

### <a name="add-an-output-to-the-stream-analytics-job"></a>將輸出新增至串流分析作業

1. 在 [作業拓撲] 之下，按一下 [輸出]。

1. 在 [輸出] 窗格中，按一下 [新增]，然後選取 [Power BI]。 在顯示的畫面上，填寫下列欄位：

   **輸出別名**︰輸出的唯一別名。 本教學課程使用 **contosooutputs**。 

   **資料集名稱**：Power BI 中要使用的資料集名稱。 本教學課程使用 **contosodataset**。 

   **表格名稱**：Power BI 中要使用的表格名稱。 本教學課程使用 **contosotable**。

   對於其餘欄位，請採用預設值。

1. 按一下 [授權]，然後登入您的 Power BI 帳戶。

   ![顯示如何為串流分析作業設定輸出的螢幕擷取畫面。](./media/tutorial-routing/stream-analytics-job-outputs.png)

1. 按一下 [檔案] 。

### <a name="configure-the-query-of-the-stream-analytics-job"></a>設定串流分析作業的查詢

1. 在 [作業拓撲] 之下，按一下 [查詢]。

1. 使用作業的輸入別名取代 `[YourInputAlias]`。 本教學課程使用 **contosoinputs**。

1. 使用作業的輸出別名取代 `[YourOutputAlias]`。 本教學課程使用 **contosooutputs**。

   ![顯示如何為串流分析作業設定查詢的螢幕擷取畫面。](./media/tutorial-routing/stream-analytics-job-query.png)

1. 按一下 [檔案] 。

1. 關閉 [查詢] 窗格。 您會回到資源群組中的資源檢視。 按一下串流分析作業。 本教學課程將它稱為 **contosoJob**。

### <a name="run-the-stream-analytics-job"></a>執行串流分析作業

在串流分析作業中，按一下 [啟動] > [立即] > [啟動]。 成功啟動作業後，作業狀態會從 [已停止] 變更為 [執行中]。

由於您需要資料來設定 Power BI 報表，因此您要先建立裝置並執行裝置模擬應用程式，然後再設定 Power BI。

## <a name="run-simulated-device-app"></a>執行模擬裝置應用程式

在前面的指令碼設定一節中，您已設定裝置來模擬使用 IoT 裝置。 在本節中，您會下載模擬裝置的 .NET 主控台應用程式，來將裝置到雲端的訊息傳送至 IoT 中樞。 此應用程式會針對每個不同的路由方式傳送訊息。 

下載 [IoT 裝置模擬](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip)的解決方案。 這會下載包含數個應用程式的存放庫；您需要的解決方案位於 iot-hub/Tutorials/Routing/SimulatedDevice/。

按兩下解決方案檔案 (SimulatedDevice.sln)，從 Visual Studio 中開啟程式碼，然後開啟 Program.cs。 以 IoT 中樞的主機名稱取代 `{iot hub hostname}`。 IoT 中樞主機名稱的格式是 **{iot-hub-name}.azure-devices.net**。 在此教學課程中，中樞主機名稱是 **ContosoTestHub.azure-devices.net**。 接下來，以您稍早設定模擬裝置時儲存的裝置金鑰取代 `{device key}`。 

   ```csharp
        static string myDeviceId = "contoso-test-device";
        static string iotHubUri = "ContosoTestHub.azure-devices.net";
        // This is the primary key for the device. This is in the portal. 
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key. 
        static string deviceKey = "{your device key here}";
   ```

## <a name="run-and-test"></a>執行和測試 

執行主控台應用程式。 請等待數分鐘。 在應用程式的主控台畫面上，您可以看到正在傳送的訊息。

應用程式每一秒就會將最新裝置到雲端的訊息傳送至 IoT 中樞。 訊息包含 JSON 序列化的物件與裝置識別碼、溫度、溼度和訊息層級 (預設位置為 `normal`)。 應用程式會隨機指派 `critical` 或 `storage` 層級，讓訊息可路由至儲存體帳戶或服務匯流排佇列 (進而觸發邏輯應用程式以傳送電子郵件)。 預設 (`normal`) 的讀取將會顯示在稍後設定的 BI 報表中。

如果所有項目皆已正確設定，此時您應該會看到下列結果：

1. 您會開始收到關於重大訊息的電子郵件。 

   ![顯示結果電子郵件的螢幕擷取畫面。](./media/tutorial-routing/results-in-email.png)

   這表示：

   * 服務匯流排佇列的路由可正確運作。
   * 從服務匯流排佇列擷取訊息的邏輯應用程式可正常運作。
   * Outlook 的邏輯應用程式連接器可正常運作。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [資源群組]，然後選取您的資源群組。 本教學課程使用 **ContosoResources**。 選取儲存體帳戶，按一下 [Blob]，然後選取 [容器]。 本教學課程使用 **contosoresults**。 您應該會看到資料夾，然後您可以向下鑽研這些目錄，直到您看到一個或多個檔案。 開啟這些檔案的其中一個，其中會包含路由至儲存體帳戶的項目。 

   ![顯示儲存體中結果檔案的螢幕擷取畫面。](./media/tutorial-routing/results-in-storage.png)

這表示：

   * 儲存體帳戶的路由可正確運作。

趁著應用程式仍在執行時，現在請設定 Power BI 視覺效果以查看來自預設路由的訊息。 

## <a name="set-up-the-power-bi-visualizations"></a>設定 Power BI 視覺效果

1. 登入您的 [Power BI](https://powerbi.microsoft.com/) 帳戶。

1. 請移至**工作區**，然後選取建立串流分析作業輸出時所設定的工作區。 本教學課程使用 **My Workspace**。 

1. 按一下 [資料集]。

   您應該會看到在建立串流分析作業輸出時所指定的資料集。 本教學課程使用 **contosodataset**。 (第一次執行可能需要 5-10 分鐘才會出現資料集)。

1. 在 [動作] 之下，按一下第一個圖示以建立報告。

   ![螢幕擷取畫面中顯示 Power BI 工作區與動作，以及醒目提示的報表圖示。](./media/tutorial-routing/power-bi-actions.png)

1. 建立折線圖以顯示一段時間的即時溫度。

   a. 在報表建立頁面上，按一下折線圖的圖示來新增折線圖。

   ![顯示視覺效果和欄位的螢幕擷取畫面。](./media/tutorial-routing/power-bi-visualizations-and-fields.png)

   b. 在 [欄位] 窗格上，展開您建立串流分析作業輸出時指定的資料表。 本教學課程使用 **contosotable**。

   c. 將 **EventEnqueuedUtcTime** 拖放至 [視覺效果] 窗格上的 [軸]。

   d. 將 **temperature** 拖放至 [值]。

   折線圖已建立。 x 軸會顯示 UTC 時區的日期和時間。 y 軸會顯示感應器的溫度。

1. 建立另一個折線圖以顯示一段時間的即時溼度。 若要設定第二個圖表，請遵循上述相同步驟，並將 **EventEnqueuedUtcTime** 放在 x 軸上，將 **humidity** 放在 y 軸上。

   ![顯示最終 Power BI 報表與兩個圖表的螢幕擷取畫面。](./media/tutorial-routing/power-bi-report.png)

1. 按一下 [儲存] 以儲存報告。

您應該會看到兩個圖表的資料。 這表示：

   * 預設端點的路由可正確運作。
   * Azure 串流分析作業工作可正確串流。
   * Power BI 視覺效果已正確設定。

您可以按一下 Power BI 視窗頂端的 [重新整理] 按鈕，以查看最新資料的圖表。 

## <a name="clean-up-resources"></a>清除資源 

如果您要移除所有已建立的資源，請刪除資源群組。 此動作會同時刪除群組內含的所有資源。 在此案例中，則會移除 IoT 中樞、服務匯流排命名空間和佇列、邏輯應用程式、儲存體帳戶和資源群組本身。 

### <a name="clean-up-resources-in-the-power-bi-visualization"></a>清除 Power BI 視覺效果中的資源

登入您的 [Power BI](https://powerbi.microsoft.com/) 帳戶。 移至工作區。 本教學課程使用 **My Workspace**。 若要移除 Power BI 視覺效果，請移至 DataSets，然後按一下垃圾桶圖示來刪除資料集。 本教學課程使用 **contosodataset**。 當您移除資料集時，報表也會一併移除。

### <a name="clean-up-resources-using-azure-cli"></a>使用 Azure CLI 清除資源

若要移除資源群組，請使用 [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete) 命令。

```azurecli-interactive
az group delete --name $resourceGroup
```
### <a name="clean-up-resources-using-powershell"></a>使用 PowerShell 清除資源

若要移除資源群組，請使用 [Remove-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令。 在本教學課程剛開始時，$resourceGroup 已設回 **ContosoIoTRG1**。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name $resourceGroup
```


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已學會如何藉由執行下列工作，使用訊息路由將 IoT 中樞訊息路由至不同目的地。  

> [!div class="checklist"]
> * 使用 Azure CLI 或 PowerShell 設定基礎資源 -- IoT 中樞、儲存體帳戶、服務匯流排佇列和模擬裝置。
> * 在 IoT 中樞內設定儲存體帳戶和服務匯流排佇列的端點及路由。
> * 建立邏輯應用程式，使其在訊息新增至服務匯流排佇列時觸發並傳送電子郵件。
> * 下載並執行模擬 IoT 裝置的應用程式，針對不同路由選項將訊息傳送至中樞。
> * 為傳送至預設端點的資料建立 Power BI 視覺效果。
> * 檢視結果...
> * ...在服務匯流排佇列和電子郵件中。
> * ...在儲存體帳戶中。
> * ...在 Power BI 視覺效果中。

前進至下一個教學課程，以了解如何管理 IoT 裝置的狀態。 

> [!div class="nextstepaction"]
[從後端服務設定您的裝置](tutorial-device-twins.md)

 <!--  [Manage the state of a device](./tutorial-manage-state.md) -->
