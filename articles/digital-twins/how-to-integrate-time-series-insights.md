---
title: 與 Azure 時間序列深入解析整合
titleSuffix: Azure Digital Twins
description: 瞭解如何設定從 Azure 數位 Twins 到 Azure 時間序列深入解析的事件路由。
author: alexkarcher-msft
ms.author: alkarche
ms.date: 1/19/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 24b4f56e5798acc4d9bd0962be7059a359958645
ms.sourcegitcommit: 65cef6e5d7c2827cf1194451c8f26a3458bc310a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2021
ms.locfileid: "98573236"
---
# <a name="integrate-azure-digital-twins-with-azure-time-series-insights"></a>整合 Azure 數位 Twins 與 Azure 時間序列深入解析

在本文中，您將瞭解如何整合 Azure 數位 Twins 與 [Azure 時間序列深入解析 (TSI) ](../time-series-insights/overview-what-is-tsi.md)。

本文所述的解決方案可讓您收集及分析 IoT 解決方案的歷程記錄資料。 Azure Digital Twins 是將資料送入時間序列深入解析的絕佳功能，因為其可讓您將多個資料流相互關聯，並將資訊標準化，再將其傳送至時間序列深入解析。 

## <a name="prerequisites"></a>必要條件

您必須要有 **Azure 數位 Twins 實例**，才能設定與時間序列深入解析之間的關聯性。 此實例應設定為根據資料更新數位對應項資訊，因為您需要更新對應項資訊幾次，才能看到時間序列深入解析中所追蹤的資料。 

如果您還沒有此設定，您可以遵循 Azure 數位 Twins [*教學課程：連接端對端解決方案*](./tutorial-end-to-end.md)來建立它。 本教學課程將逐步引導您設定 Azure 數位 Twins 實例，以搭配虛擬 IoT 裝置來觸發數位對應項更新。

## <a name="solution-architecture"></a>方案架構

您將透過下列路徑將時間序列深入解析附加至 Azure 數位 Twins。

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-integrate-time-series-insights/diagram-simple.png" alt-text="在端對端案例中，反白顯示時間序列深入解析的 Azure 服務視圖" lightbox="media/how-to-integrate-time-series-insights/diagram-simple.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="create-a-route-and-filter-to-twin-update-notifications"></a>建立對應項更新通知的路由與篩選

當對應項的狀態更新時，Azure 數位 Twins 實例可以發出對應項 [更新事件](how-to-interpret-event-data.md) 。 在本節中，您將建立 Azure 數位 Twins [**事件路由**](concepts-route-events.md) ，將這些更新事件導向至 Azure [事件中樞](../event-hubs/event-hubs-about.md) 以進行進一步的處理。

Azure 數位 Twins [*教學課程：連接端對端解決方案*](./tutorial-end-to-end.md) 會逐步解說一個案例，在此案例中，會使用溫度計來更新代表房間的數位對應項上的溫度屬性。 此模式依賴對應項更新，而不是從 IoT 裝置轉送遙測，讓您有彈性地變更基礎資料來源，而不需要更新您的時間序列深入解析邏輯。

1. 首先，建立事件中樞命名空間，此命名空間將會從您的 Azure 數位 Twins 實例接收事件。 您可以使用下列 Azure CLI 指示，或使用 Azure 入口網站： [*快速入門：使用 Azure 入口網站建立事件中樞*](../event-hubs/event-hubs-create.md)。 若要查看哪些區域支援事件中樞，請造訪 [*依區域提供的 Azure 產品*](https://azure.microsoft.com/global-infrastructure/services/?products=event-hubs)。

    ```azurecli-interactive
    az eventhubs namespace create --name <name for your Event Hubs namespace> --resource-group <resource group name> -l <region>
    ```

2. 在命名空間內建立事件中樞，以接收對應項變更事件。 指定事件中樞的名稱。

    ```azurecli-interactive
    az eventhubs eventhub create --name <name for your Twins event hub> --resource-group <resource group name> --namespace-name <Event Hubs namespace from above>
    ```

3. 使用傳送和接收許可權建立 [授權規則](/cli/azure/eventhubs/eventhub/authorization-rule?view=azure-cli-latest&preserve-view=true#az-eventhubs-eventhub-authorization-rule-create) 。 指定規則的名稱。

    ```azurecli-interactive
        az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group <resource group name> --namespace-name <Event Hubs namespace from above> --eventhub-name <Twins event hub name from above> --name <name for your Twins auth rule>
    ```

4. 建立 Azure 數位 Twins [端點](concepts-route-events.md#create-an-endpoint) ，將您的事件中樞連結至您的 Azure 數位 Twins 實例。

    ```azurecli-interactive
    az dt endpoint create eventhub --endpoint-name <name for your Event Hubs endpoint> --eventhub-resource-group <resource group name> --eventhub-namespace <Event Hubs namespace from above> --eventhub <Twins event hub name from above> --eventhub-policy <Twins auth rule from above> -n <your Azure Digital Twins instance name>
    ```

5. 在 Azure Digital Twins 中建立[路由](concepts-route-events.md#create-an-event-route)，以將對應項更新事件傳送至您的端點。 此路由中的篩選器只允許將對應項更新訊息傳遞至您的端點。

    >[!NOTE]
    >目前 Cloud Shell 有 **已知問題** 會影響這些命令群組：`az dt route`、`az dt model`、`az dt twin`。
    >
    >若要解決此問題，請在執行命令之前，先在 Cloud Shell 中執行 `az login`；或使用[本機 CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true)，而不是 Cloud Shell。 如需這方面的詳細資訊，請參閱[疑難排解：Azure Digital Twins 中的已知問題](troubleshoot-known-issues.md#400-client-error-bad-request-in-cloud-shell)。

    ```azurecli-interactive
    az dt route create -n <your Azure Digital Twins instance name> --endpoint-name <Event Hub endpoint from above> --route-name <name for your route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
    ```

在繼續之前，請記下您的 *事件中樞命名空間* 和 *資源群組*，因為您將在本文稍後使用它們來建立另一個事件中樞。

## <a name="create-a-function-in-azure"></a>在 Azure 中建立函式

接下來，您將使用 Azure Functions 在函數應用程式中建立 **事件中樞觸發的函式** 。 您可以使用在端對端教學課程中建立的函數應用程式 ([*教學課程：連接端對端解決方案*](./tutorial-end-to-end.md)) 或您自己的解決方案。 

此函式會將這些對應項更新事件從其原始格式轉換為 json 物件，其中只包含來自您 twins 的已更新和新增值。

如需使用事件中樞搭配 Azure Functions 的詳細資訊，請參閱 [*Azure Functions 的 Azure 事件中樞觸發程式*](../azure-functions/functions-bindings-event-hubs-trigger.md)。

在您已發佈的函式應用程式中，以下列程式碼取代函式程式碼。

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/updateTSI.cs":::

接著，此函式會將它所建立的 JSON 物件傳送至第二個事件中樞，而您將連接到時間序列深入解析。

稍後，您也會設定此函式將用來連線到您自己的事件中樞的某些環境變數。

## <a name="send-telemetry-to-an-event-hub"></a>將遙測傳送至事件中樞

您現在將建立第二個事件中樞，並設定您的函式將其輸出串流至該事件中樞。 此事件中樞接著將會連線到時間序列深入解析。

### <a name="create-an-event-hub"></a>建立事件中樞

若要建立第二個事件中樞，您可以使用下列 Azure CLI 指示，或使用 Azure 入口網站： [*快速入門：使用 Azure 入口網站建立事件中樞*](../event-hubs/event-hubs-create.md)。

1. 從本文稍早準備您的 *事件中樞命名空間* 和 *資源組* 名

2. 建立新的事件中樞。 指定事件中樞的名稱。

    ```azurecli-interactive
    az eventhubs eventhub create --name <name for your TSI event hub> --resource-group <resource group name from earlier> --namespace-name <Event Hubs namespace from earlier>
    ```
3. 使用傳送和接收許可權建立 [授權規則](/cli/azure/eventhubs/eventhub/authorization-rule?view=azure-cli-latest&preserve-view=true#az-eventhubs-eventhub-authorization-rule-create) 。 指定規則的名稱。

    ```azurecli-interactive
        az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group <resource group name> --namespace-name <Event Hubs namespace from earlier> --eventhub-name <TSI event hub name from above> --name <name for your TSI auth rule>
    ```

## <a name="configure-your-function"></a>設定您的函式

接下來，您必須在您的函數應用程式中設定舊版的環境變數，其中包含您所建立之事件中樞的連接字串。

### <a name="set-the-twins-event-hub-connection-string"></a>設定對應項事件中樞連接字串

1. 使用您針對 Twins 中樞所建立的授權規則，取得 Twins [事件中樞連接字串](../event-hubs/event-hubs-get-connection-string.md)。

    ```azurecli-interactive
    az eventhubs eventhub authorization-rule keys list --resource-group <resource group name> --namespace-name <Event Hubs namespace> --eventhub-name <Twins event hub name from earlier> --name <Twins auth rule from earlier>
    ```

2. 使用您取得的連接字串結果，在您的函數應用程式中建立包含連接字串的應用程式設定：

    ```azurecli-interactive
    az functionapp config appsettings set --settings "EventHubAppSetting-Twins=<Twins event hub connection string>" -g <resource group> -n <your App Service (function app) name>
    ```

### <a name="set-the-time-series-insights-event-hub-connection-string"></a>設定時間序列深入解析事件中樞連接字串

1. 使用您先前為時間序列深入解析中樞建立的授權規則，取得 TSI [事件中樞連接字串](../event-hubs/event-hubs-get-connection-string.md)：

    ```azurecli-interactive
    az eventhubs eventhub authorization-rule keys list --resource-group <resource group name> --namespace-name <Event Hubs namespace> --eventhub-name <TSI event hub name> --name <TSI auth rule>
    ```

2. 在您的函數應用程式中，建立包含您連接字串的應用程式設定：

    ```azurecli-interactive
    az functionapp config appsettings set --settings "EventHubAppSetting-TSI=<TSI event hub connection string>" -g <resource group> -n <your App Service (function app) name>
    ```

## <a name="create-and-connect-a-time-series-insights-instance"></a>建立及連接時間序列深入解析執行個體

接下來，您將設定時間序列深入解析實例，以接收第二個事件中樞的資料。 請遵循下列步驟，如需此程式的詳細資訊，請參閱 [*教學課程：設定 Azure 時間序列深入解析 GEN2 PAYG 環境*](../time-series-insights/tutorials-set-up-tsi-environment.md)。

1. 在 Azure 入口網站中，開始建立時間序列深入解析資源。 
    1. 選取 **Gen2 (L1)** 定價層。
    2. 您將需要為此環境選擇 **時間序列識別碼** 。 您的時間序列識別碼最多可以有三個值，您將在時間序列深入解析中用來搜尋資料。 在本教學課程中，您可以使用 **$dtId**。 在 [*選擇時間序列識別碼的最佳做法中，*](../time-series-insights/how-to-select-tsid.md)深入瞭解如何選取識別碼值。
    
        :::image type="content" source="media/how-to-integrate-time-series-insights/create-twin-id.png" alt-text="時間序列深入解析環境的建立入口網站 UX。已選取 Gen2 (L1) 定價層，且時間序列識別碼屬性名稱為 $dtId" lightbox="media/how-to-integrate-time-series-insights/create-twin-id.png":::

2. 選取 **[下一步：事件來源]** ，然後選取上述的事件中樞資訊。 您也需要建立新的事件中樞取用者群組。
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/event-source-twins.png" alt-text="時間序列深入解析環境事件來源的建立入口網站 UX。您正在使用上述的事件中樞資訊來建立事件來源。您也會建立新的取用者群組。" lightbox="media/how-to-integrate-time-series-insights/event-source-twins.png":::

## <a name="begin-sending-iot-data-to-azure-digital-twins"></a>開始將 IoT 資料傳送至 Azure 數位 Twins

若要開始將資料傳送到時間序列深入解析，您必須開始更新 Azure 數位 Twins 中的數位對應項屬性，並變更資料值。 使用 [az dt](/cli/azure/ext/azure-iot/dt/twin?view=azure-cli-latest&preserve-view=true#ext-azure-iot-az-dt-twin-update) 對應項 update 命令。

如果您使用端對端教學課程 ([*教學課程：連接端對端解決方案*](tutorial-end-to-end.md)) 以協助設定環境，您可以從範例執行 *devicesimulator.exe* 專案來開始傳送模擬的 IoT 資料。 本教學課程的「 [*設定和執行模擬*](tutorial-end-to-end.md#configure-and-run-the-simulation) 」一節中的指示。

## <a name="visualize-your-data-in-time-series-insights"></a>將時間序列深入解析中的資料視覺化

現在，資料應該流入您的時間序列深入解析實例，並準備好進行分析。 請遵循下列步驟來探索即將推出的資料。

1. 在 [Azure 入口網站](https://portal.azure.com) 中開啟您的時間序列深入解析實例 (您可以在入口網站的搜尋列中搜尋您的實例名稱) 。 造訪執行個體概觀中所顯示的「時間序列深入解析總管 URL」。
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/view-environment.png" alt-text="在時間序列深入解析環境的 [總覽] 索引標籤中，選取時間序列深入解析 explorer URL":::

2. 在 [explorer] 中，您會看到來自 Azure 數位 Twins 的三個 twins 顯示于左邊。 選取 [ _**thermostat67**_]，選取 [ **溫度**]，然後按 [ **新增**]。

    :::image type="content" source="media/how-to-integrate-time-series-insights/add-data.png" alt-text="選取 * * thermostat67 * *，選取 * * 溫度 * *，然後按 [新增] * *":::

3. 您現在應該會看到來自控溫器的初始溫度讀數，如下所示。 系統會針對 *room21* 和 *floor1* 更新相同的溫度讀數，而且您可以將這些資料流程一併呈現出來。
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/initial-data.png" alt-text="初始溫度資料會在 TSI explorer 中繪製圖形。這是介於68到85之間的隨機值行":::

4. 如果您讓模擬執行得更久，視覺效果看起來會像這樣：
    
    :::image type="content" source="media/how-to-integrate-time-series-insights/day-data.png" alt-text="每個對應項的溫度資料會以不同色彩的三個平行線繪製。":::

## <a name="next-steps"></a>後續步驟

預設會在時間序列深入解析中將數位 twins 儲存為一般階層，但是可以使用模型資訊和組織的多層級階層進行擴充。 若要深入瞭解此程式，請參閱： 

* [*教學課程：定義和套用模型*](../time-series-insights/tutorials-set-up-tsi-environment.md#define-and-apply-a-model) 

您可以撰寫自訂邏輯，使用已儲存在 Azure 數位 Twins 中的模型和圖形資料來自動提供此資訊。 若要深入瞭解如何管理、升級及從 twins 圖形中取得資訊，請參閱下列參考：

* [*如何：管理數位對應項*](./how-to-manage-twin.md)
* [*How to：查詢對應項圖形*](./how-to-query-graph.md)