---
title: 在傳送至 Azure 事件中樞的即時事件中視覺化資料異常 | Microsoft Docs
description: 教學課程 - 在傳送至 Microsoft Azure 事件中樞的即時事件中視覺化資料異常
services: event-hubs
author: ShubhaVijayasarathy
manager: timlt
ms.author: shvija
ms.date: 08/08/2018
ms.topic: tutorial
ms.service: event-hubs
ms.custom: mvc
ms.openlocfilehash: 04a9a3b3df44814d680f01595d70ced08a946591
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40004108"
---
# <a name="tutorial-visualize-data-anomalies-in-real-time-events-sent-to-azure-event-hubs"></a>教學課程：在傳送至 Azure 事件中樞的即時事件中視覺化資料異常

透過 Azure 事件中樞，您可以使用 Azure 串流分析檢查內送資料並提取異常，然後在 Power BI 中視覺化這些異常。 假設您有數千部裝置不斷地將即時資料傳送到事件中樞，並且每秒新增多達數百萬個事件。 您要如何在這麼多資料中檢查出異常或錯誤？ 例如，裝置正在傳送信用卡交易，而您需要擷取每 5 秒就在多個國家/地區中進行多個交易的位置，這該怎麼做呢？ 會發生此狀況可能是因為有人竊取信用卡，然後同時使用這些信用卡在世界各地購買物品。 

在本教學課程中，您將模擬此範例。 您將執行應用程式來建立信用卡交易，並將其傳送到事件中樞。 然後，您會使用 Azure 串流分析來讀取即時資料流，串流分析會分隔有效交易和無效交易，並使用 Power BI 以視覺方式識別標記為無效的交易。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 建立事件中樞命名空間
> * 建立事件中樞
> * 執行會傳送信用卡交易的應用程式
> * 設定串流分析作業來處理這些交易
> * 設定 Power BI 視覺效果以顯示結果

若要完成此教學課程，您需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前[建立免費帳戶][]。

## <a name="prerequisites"></a>先決條件

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

- 安裝 [Visual Studio](https://www.visualstudio.com/)。 
- 您需要 Power BI 帳戶來分析串流分析作業的輸出。 您可以[免費試用 Power BI](https://app.powerbi.com/signupredirect?pbi_source=web)。

## <a name="set-up-resources"></a>設定資源

針對此教學課程，您必須建立事件中樞命名空間和事件中樞。 您可以使用 Azure CLI 或 Azure PowerShell 來建立這些資源。 請將所有資源建立在相同的資源群組和位置。 如此一來，您就可以在結束時，透過刪除資源群組的單一步驟來移除所有項目。

下列各節會說明如何執行這些必要步驟。 請依照 CLI 或 PowerShell 指示來執行下列步驟：

1. 建立[資源群組](../azure-resource-manager/resource-group-overview.md)。 

2. 建立事件中樞命名空間。 

3. 建立事件中樞。

> [!NOTE]
> 每個指令碼中都會設定稍後在本教學課程中需要的變數。 其中包括資源群組名稱 ($resourceGroup)、事件中樞命名空間 (**$eventHubNamespace**)，以及事件中樞名稱 (**$eventHubName**)。 稍後在本文中提到這些變數時，會在其開頭加上貨幣符號 ($)，因此您可以知道這些變數已設定於指令碼中。

<!-- some day they will approve the tab control; 
  When that happens, put CLI and PSH in tabs. -->

### <a name="set-up-your-resources-using-azure-cli"></a>使用 Azure CLI 設定資源

複製此指令碼並貼到 Cloud Shell。 如果您已登入，它會一次執行一行指令碼。

必須是全域唯一的變數會與 `$RANDOM` 串連。 執行指令碼並設定變數時，系統會產生隨機數字字串，並將其串連至固定字串的結尾，使其成為唯一。

```azurecli-interactive
# Set the values for location and resource group name.
location=westus
resourceGroup=ContosoResourcesEH

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
eventHubNamespace=ContosoEHNamespace$RANDOM
echo "Event Hub Namespace = " $eventHubNamespace

# Create the Event Hubs namespace.
az eventhubs namespace create --resource-group $resourceGroup \
   --name $eventHubNamespace \
   --location $location \
   --sku Standard

# The event hub name must be globally unique, so add a random number to the end.
eventHubName=ContosoEHhub$RANDOM
echo "event hub name = " $eventHubName

# Create the event hub.
az eventhubs eventhub create --resource-group $resourceGroup \
    --namespace-name $eventHubNamespace \
    --name $eventHubName \
    --message-retention 3 \
    --partition-count 2

# Get the connection string that authenticates the app with the Event Hubs service.
connectionString=$(az eventhubs namespace authorization-rule keys list \
   --resource-group $resourceGroup \
   --namespace-name $eventHubNamespace \
   --name RootManageSharedAccessKey \
   --query primaryConnectionString \
   --output tsv)
echo "Connection string = " $connectionString 
```

### <a name="set-up-your-resources-using-azure-powershell"></a>使用 Azure PowerShell 設定資源

複製此指令碼並貼到 Cloud Shell。 如果您已登入，它會一次執行一行指令碼。

必須是全域唯一的變數會與 `$(Get-Random)` 串連。 執行指令碼並設定變數時，系統會產生隨機數字字串，並將其串連至固定字串的結尾，使其成為唯一。

```azurepowershell-interactive
# Log in to Azure account.
Login-AzureRMAccount

# Set the values for the location and resource group.
$location = "West US"
$resourceGroup = "ContosoResourcesEH"

# Create the resource group to be used  
#   for all resources for this tutorial.
New-AzureRmResourceGroup -Name $resourceGroup -Location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
$eventHubNamespace = "contosoEHNamespace$(Get-Random)"
Write-Host "Event Hub Namespace is " $eventHubNamespace

# The event hub name must be globally unique, so add a random number to the end.
$eventHubName = "contosoEHhub$(Get-Random)"
Write-Host "Event hub Name is " $eventHubName

# Create the Event Hubs namespace.
New-AzureRmEventHubNamespace -ResourceGroupName $resourceGroup `
     -NamespaceName $eventHubNamespace `
     -Location $location

# Create the event hub.
$yourEventHub = New-AzureRmEventHub -ResourceGroupName $resourceGroup `
    -NamespaceName $eventHubNamespace `
    -Name $eventHubName `
    -MessageRetentionInDays 3 `
    -PartitionCount 2

# Get the event hub key, and retrieve the connection string from that object.
# You need this to run the app that sends test messages to the event hub.
$eventHubKey = Get-AzureRmEventHubKey -ResourceGroupName $resourceGroup `
    -Namespace $eventHubNamespace `
    -AuthorizationRuleName RootManageSharedAccessKey

# Save this value somewhere local for later use.
Write-Host "Connection string is " $eventHubKey.PrimaryConnectionString
```

## <a name="run-app-to-produce-test-event-data"></a>執行應用程式以產生測試用的事件資料

[GitHub 上的事件中樞範例](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet)包括為您產生測試資料的[異常偵測器應用程式](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/AnomalyDetector)。 其藉由將信用卡交易寫入事件中樞來模擬信用卡的使用，包括偶爾在多個地點對相同信用卡寫入數個交易，使其標記為異常。 若要執行此應用程式，請遵循下列步驟： 

1. 從 GitHub 下載 [Azure 事件中樞範例](https://github.com/Azure/azure-event-hubs/archive/master.zip)，並將它解壓縮到本機。

2. 移至 \azure-event-hubs-master\samples\DotNet\AnomalyDetector\ and double-click on AnomalyDetector.sln 資料夾，以在 Visual Studio 中開啟解決方案。 

3. 開啟 Program.cs，並以執行指令碼時儲存的連接字串取代**事件中樞連接字串**。 

4. 以您的事件中樞名稱取代**事件中樞名稱**。 按 F5 鍵執行應用程式。 它會開始將事件傳送至事件中樞，並持續到傳送 1000 個事件為止。 有幾個執行個體中必須執行該應用程式，以便您擷取資料。 若有需要，這些情況將在下列指示中指出。

## <a name="set-up-azure-stream-analytics"></a>設定 Azure 串流分析

現在您可以將資料串流至事件中樞。 若要在 Power BI 視覺效果中使用該資料，請先設定串流分析作業，以擷取稍後會填入 Power BI 視覺效果的資料。

### <a name="create-the-stream-analytics-job"></a>建立串流分析作業

1. 在 Azure 入口網站中，按一下 [建立資源]。 在搜尋方塊中輸入**串流分析**，然後按 [Enter]。 選取 [串流分析作業]。 按一下串流分析作業窗格上的 [建立]。 

2. 輸入作業的以下資訊：

   **作業名稱**：使用 **contosoEHjob**。 此欄位是作業的名稱，必須具有全域唯一性。

   **訂用帳戶**︰選取您的訂用帳戶。

   **資源群組**︰使用事件中樞所用的相同資源群組 (**ContosoResourcesEH**)。

   **位置**：使用安裝指令碼中使用的相同位置 (**West US**)。

   ![顯示如何建立新 Azure 串流分析作業的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-add-job.png)

    對於其餘欄位，請採用預設值。 按一下頁面底部的 [新增] 。 

### <a name="add-an-input-to-the-stream-analytics-job"></a>將輸入新增至串流分析作業

如果您不在入口網站的 [串流分析作業] 窗格中，您可以按一下入口網站中的 [資源群組] 來回到串流分析作業，然後選取您的資源群組 (**ContosoResourcesEH**)。 此動作會顯示群組中的所有資源，接著您就可以選取您的串流分析作業。 

串流分析作業的輸入是來自事件中樞的信用卡交易。

> [!NOTE]
> 在前面章節中，啟動指令碼內已設定以貨幣符號 ($) 開頭的變數值。 您必須在此使用相同值來指定這些欄位，也就是事件中樞命名空間和事件中樞名稱。

1. 在 [作業拓撲] 之下，按一下 [輸入]。

2. 在 [輸入] 窗格中，按一下 [新增資料流輸入] 並選取事件中樞。 在顯示的畫面上，填寫下列欄位：

   **輸入別名**：使用 **contosoinputs**。 此欄位是輸入資料流的名稱，會在定義資料的查詢時使用。

   **訂用帳戶**︰選取您的訂用帳戶。

   **事件中樞命名空間**：選取您的事件中樞命名空間 ($**eventHubNamespace**)。 

   **事件中樞名稱**：按一下 [使用現有項目]，然後選取事件中樞 ($**eventHubName**)。

   **事件中樞原則名稱**：選取 [RootManageSharedAccessKey]。

   **事件中樞取用者群組**：將此欄位保留空白，以使用預設的取用者群組。

   對於其餘欄位，請採用預設值。

   ![顯示如何將輸入資料流新增至串流分析作業的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-inputs.png)

5. 按一下 [檔案] 。

### <a name="add-an-output-to-the-stream-analytics-job"></a>將輸出新增至串流分析作業

1. 在 [作業拓撲] 之下，按一下 [輸出]。 此欄位是輸出資料流的名稱，會在定義資料的查詢時使用。

2. 在 [輸出] 窗格中，按一下 [新增]，然後選取 [Power BI]。 在顯示的畫面上，完成下列欄位：

   **輸出別名**：使用 **contosooutputs**。 此欄位是輸出的唯一別名。 

   **資料集名稱**：使用 **contosoehdataset**。 此欄位是 Power BI 中要使用的資料集名稱。 

   **資料表名稱**：使用 **contosoehtable**。 此欄位是 Power BI 中要使用的資料表名稱。 

   對於其餘欄位，請採用預設值。

   ![顯示如何為串流分析作業設定輸出的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-outputs.png)

3. 按一下 [授權]，然後登入您的 Power BI 帳戶。

4. 對於其餘欄位，請採用預設值。

5. 按一下 [檔案] 。

### <a name="configure-the-query-of-the-stream-analytics-job"></a>設定串流分析作業的查詢

此查詢會用來擷取最後會傳送至 Power BI 視覺效果的資料。 它會使用先前設定作業時定義的 **contosoinputs** 和 **contosooutputs**。 此查詢會擷取視為詐騙的信用卡交易，也就是同樣每五秒的時間間隔內，相同信用卡號碼在不同地點中有多筆交易。

1. 在 [作業拓撲] 之下，按一下 [查詢]。

2. 將查詢取代為下列其中一個： 

   ```SQL
   /* criteria for fraud:
      credit card purchases with the same card
      in different locations within 5 seconds
   */
   SELECT System.Timestamp AS WindowEnd, 
     COUNT(*) as FraudulentUses      
   INTO contosooutputs
   FROM contosoinputs CS1 TIMESTAMP BY [Timestamp]
       JOIN contosoinputs CS2 TIMESTAMP BY [Timestamp]
       /* where the credit card # is the same */
       ON CS1.CreditCardId = CS2.CreditCardId
       /* and time between the two is between 0 and 5 seconds */
       AND DATEDIFF(second, CS1, CS2) BETWEEN 0 AND 5
       /* where the location is different */
   WHERE CS1.Location != CS2.Location
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. 按一下 [檔案] 。

### <a name="test-the-query-for-the-stream-analytics-job"></a>測試串流分析作業的查詢 

1. 在設定及執行測試時，執行異常偵測器應用程式來將資料傳送至事件中樞。 

2. 在 [查詢] 窗格中，按一下 **contosoinputs** 輸入旁邊的點，然後選取 [來自輸入的範例資料]。

3. 指定您想要取得三分鐘的資料，然後按一下 [確定]。 等待資料已取樣的通知。

4. 按一下 [測試]，並確定開始產生結果。 結果會顯示在查詢下窗格右下方的 [結果] 區段。

5. 關閉 [查詢] 窗格。

### <a name="run-the-stream-analytics-job"></a>執行串流分析作業

在串流分析作業中，依序按一下 [啟動]、[立即] 及 [啟動]。 成功啟動作業後，作業狀態會從 [已停止] 變更為 [執行中]。

## <a name="set-up-the-power-bi-visualizations"></a>設定 Power BI 視覺效果

1. 在設定 Power BI 視覺效果時，執行異常偵測器應用程式來將資料傳送至事件中樞。 您可能需要執行數次，因為每次執行時只會產生 1000 筆交易。

2. 登入您的 [Power BI](https://powerbi.microsoft.com/) 帳戶。

3. 移至 [我的工作區]。

4. 按一下 [資料集]。

   您應該會看到在建立串流分析作業輸出時所指定的資料集 (**contosoehdataset**)。 第一次執行可能需要 5-10 分鐘才會出現資料集。

5. 按一下 [儀表板]，然後按一下 [建立]，並選取 [儀表板]。

   ![儀表板與建立按鈕的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-dashboard.png)

6. 指定儀表板名稱，然後按一下 [建立]。 使用 [信用卡異常]。

   ![指定儀表板名稱的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-name.png)

7. 在 [儀表板] 頁面上，按一下 [新增磚]，在 [即時資料] 區段中選取 [自訂串流資料]，然後按 [下一步]。

   ![指定磚來源的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-real-time-data.png)

8. 選取您的資料集 (**contosoehdataset**)，然後按 [下一步]。

   ![指定資料集的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-select-dataset.png)

9. 選取 [卡片] 作為視覺效果類型。 在 [欄位] 下按一下 [新增值]，然後選取 [fraudulentuses]。

   ![指定視覺效果類型和欄位的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-tile.png)

   按 [下一步] 。

10. 將標題設為 [詐騙用量]，並將副標題設為 [過去幾分鐘內的總和]。 按一下 [套用]。 它會將磚儲存到您的儀表板中。

    ![指定儀表板磚標題和副標題的螢幕擷取畫面。](./media/event-hubs-tutorial-visualize-anomalies/power-bi-tile-details.png)

11. 新增另一個視覺效果。 再次重複前幾個步驟：

   * 按一下 [新增磚]。
   * 選取 [自訂串流資料]。 
   * 按 [下一步] 。
   * 選取您的資料集，然後按 [下一步]。 

12. 在 [視覺效果類型] 下選取 [折線圖]。

13. 在 [軸] 下方按一下 [新增值]，然後選取 [windowend]。 

14. 在 [值] 下方按一下 [新增值]，然後選取 [fraudulentuses]。

15. 在 [要顯示的時間範圍] 下，選取過去 5 分鐘。 按 [下一步] 。

16. 指定 [顯示一段時間內的詐騙用量] 作為標題，並將磚的副標題保留空白，然後按一下 [套用]。 您會回到儀表板。

17. 再次執行異常偵測器應用程式，以將一些資料傳送至事件中樞。 分析資料時，您會看到 [詐騙用量] 磚發生變化，而折線圖上會顯示資料。 

    ![顯示 Power BI 結果的螢幕擷取畫面](./media/event-hubs-tutorial-visualize-anomalies/power-bi-results.png)

## <a name="clean-up-resources"></a>清除資源

如果您要移除所有已建立的資源，請移除 Power BI 視覺效果資料，然後刪除資源群組。 刪除資源群組會刪除群組內含的所有資源。 在此案例中，這會移除事件中樞、事件中樞命名空間、串流分析作業和資源群組本身。 

### <a name="clean-up-resources-in-the-power-bi-visualization"></a>清除 Power BI 視覺效果中的資源

登入您的 Power BI 帳戶。 移至 [我的工作區]。 在您的儀表板名稱列上，按一下 [垃圾桶] 圖示。 接著，移至 [資料集]，然後按一下垃圾桶圖示來刪除資料集 (**contosoehdataset**)。

### <a name="clean-up-resources-using-azure-cli"></a>使用 Azure CLI 清除資源

若要移除資源群組，請使用 [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete) 命令。

```azurecli-interactive
az group delete --name $resourceGroup
```

### <a name="clean-up-resources-using-powershell"></a>使用 PowerShell 清除資源

若要移除資源群組，請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：
> [!div class="checklist"]
> * 建立事件中樞命名空間
> * 建立事件中樞
> * 執行應用程式來模擬事件，並將這些事件傳送到事件中樞
> * 設定串流分析作業來處理傳送至中樞的事件
> * 設定 Power BI 視覺效果以顯示結果

請前往下一篇文章，以深入了解 Azure 事件中樞。

> [!div class="nextstepaction"]
> [開始在 .NET Standard 中傳送訊息至 Azure 事件中樞](event-hubs-dotnet-standard-getstarted-send.md)

[建立免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
