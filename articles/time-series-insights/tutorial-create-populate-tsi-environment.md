---
title: 建立 Azure 時間序列深入解析環境
description: 了解如何建立已從模擬裝置填入資料的時間序列深入解析環境。
services: time-series-insights
author: ashannon7
ms.service: time-series-insights
ms.topic: tutorial
ms.date: 06/04/2018
ms.author: anshan
manager: cshankar
ms.openlocfilehash: 46fecf6f1b2826f0e23b7b6c4f4511ce9a662d1c
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46365684"
---
# <a name="tutorial-create-an-azure-time-series-insights-environment"></a>教學課程：建立 Azure 時間序列深入解析環境

此教學課程將逐步引導您建立已從模擬裝置填入資料的時間序列深入解析 (TSI) 環境。 在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 TSI 環境 
> * 建立包含 IoT 中樞的裝置模擬解決方案
> * 將 TSI 環境連線至 IoT 中樞
> * 執行裝置模擬以將資料串流處理至 TSI 環境中
> * 驗證模擬的遙測資料

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-show-you-how-to-use-an-azure-iot-solution-accelerator-to-generate-data-that-can-be-used-to-get-started-with-time-series-insightsbr"></a>在此影片中，我們會說明如何使用「Azure IoT 解決方案加速器」來產生可用來開始使用「時間序列深入解析」的資料。</br>

> [!VIDEO https://www.youtube.com/embed/6ehNf6AJkFo]

## <a name="prerequisites"></a>先決條件

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。 

您的 Azure 登入帳戶也必須是訂用帳戶的「擁有者」角色成員。 如需詳細資料，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](/azure/role-based-access-control/role-assignments-portal)

## <a name="overview"></a>概觀

TSI 環境是收集及儲存裝置資料之處。 儲存在 TSI 環境中之後，您可以使用 [TSI 總管](time-series-quickstart.md)和 [TSI 查詢 API](/rest/api/time-series-insights/time-series-insights-reference-queryapi) 來查詢和分析資料。

和所有裝置一樣 (無論模擬或實體)，IoT 中樞也可作為連線點，供裝置安全地連線至 Azure 雲端並傳輸資料。 如 [TSI 概觀](time-series-insights-overview.md)中的討論，IoT 中樞也可作為事件來源，將資料串流處理至 TSI 環境中。 

此教學課程也會使用 [IoT 解決方案加速器](/azure/iot-accelerators/)，以產生範例遙測資料並串流處理至 IoT 中樞。 IoT 解決方案加速器提供預先設定的企業級解決方案，可讓您加快自訂 IoT 解決方案的開發速度。 

## <a name="create-a-tsi-environment"></a>建立 TSI 環境

首先，在您的 Azure 訂用帳戶中建立 TSI 環境：

1. 使用您的 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)。  
2. 選取左上方的 [+ 建立資源]。  
3. 選取 [物聯網] 類別，然後選取 [時間序列深入解析]。  
   
   [![選取時間序列深入解析環境資源](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png#lightbox)

4. 在 [時間序列深入解析環境] 頁面上，填入必要參數：
   
   參數|說明
   ---|---
   **環境名稱** | 選擇 TSI 環境的唯一名稱。 TSI 總管和查詢 API 會使用此名稱。
   **訂用帳戶** | 訂用帳戶是 Azure 資源的容器。 請選擇您要用來建立 TSI 環境的訂用帳戶。
   **資源群組** | 資源群組是 Azure 資源的容器。 請為 TSI 環境資源選擇現有的資源群組，或建立新的群組。
   **位置** | 選擇 TSI 環境的資料中心區域。 為了避免增加頻寬成本和延遲時間，建議您將 TSI 環境設置於與其他 IoT 資源相同的區域中。
   **定價 SKU** | 選擇所需的輸送量。 若要有最低成本和入門級的容量，請選取 S1。
   **容量** | 容量是套用至輸入速率、儲存體容量以及與選取之 SKU 相關聯成本的乘數。  您可以在環境建立後變更其容量。 如需最低成本，選取 1 作為容量。 

   完成後，請按一下 [建立] 開始佈建程序。

   ![建立時間序列深入解析環境資源](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-params.png)

5. 您可以查看 [通知] 面板以監視部署完成進度，其所需時間應該少於一分鐘：  

   ![時間序列深入解析環境部署成功](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-deployment-succeeded.png)

## <a name="create-a-device-simulation"></a>建立裝置模擬

接下來，請建立裝置模擬解決方案，此解決方案將產生測試資料以填入您的 TSI 環境中：

1. 在個別的視窗/索引標籤中移至 https://www.azureiotsolutions.com，使用相同的 Azure 訂用帳戶進行登入，然後選取 [裝置模擬] 加速器：

   ![執行裝置模擬加速器](media/tutorial-create-populate-tsi-environment/sa-main.png)

2. 在 [建立裝置模擬解決方案] 頁面上輸入必要的參數：

   參數|說明
   ---|---
   **解決方案名稱** | 一個唯一值，用來建立新的資源群組。 列出的 Azure 資源會在建立後指派給資源群組。
   **訂用帳戶** | 指定在上一節中用來建立 TSI 環境的相同訂用帳戶。
   **區域** | 指定在上一節中用來建立 TSI 環境的相同區域。 
   **部署選擇性 Azure 資源** | 將 [IoT 中樞] 保留為已勾選，因為模擬裝置將會用它來連接/串流處理資料。

   完成後，請按一下 [建立解決方案] 以佈建解決方案的 Azure 資源，其所需時間大約為 6-7 分鐘：

   ![佈建裝置模擬解決方案](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution.png)

3. 佈建完成後，新的解決方案上方的文字將會從「佈建中...」變更為「就緒」：

   >[!IMPORTANT]
   > 此時還不可按 [啟動] 按鈕！ 但請將此網頁保持為開啟，因為您稍後將會返回。

   ![裝置模擬解決方案佈建完成](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard-ready.png)

4. 現在，請返回 Azure 入口網站，並在您的訂用帳戶中檢查新建立的資源。 在入口網站的 [資源群組] 頁面中，您會看到使用上一個步驟中提供的**解決方案名稱**建立的新資源群組。 也請留意為了支援裝置模擬解決方案而建立的所有資源：

   [![裝置模擬解決方案資源](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png)](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png#lightbox)

## <a name="connect-the-tsi-environment-to-the-iot-hub"></a>將 TSI 環境連線至 IoT 中樞 

此時，您已了解如何建立兩組資源 (分別屬於其本身的資源群組)： 
- 空的 TSI 環境 
- 解決方案加速器所產生的裝置模擬解決方案資源，包括 IoT 中樞

前文提過，模擬裝置必須連線至 IoT 中樞，以串流處理裝置資料。 若要使資料流入 TSI 環境中，您必須同時對 IoT 中樞和 TSI 環境進行組態變更。 

### <a name="iot-hub-configuration-define-a-consumer-group"></a>IoT 中樞組態：定義取用者群組

IoT 中樞提供多種可讓您與其他執行者共用功能的端點。 「事件」端點提供讓其他應用程式在資料串流處理至 IoT 中樞執行個體時取用資料的方法。 具體來說，「取用者群組」會提供讓應用程式接聽 IoT 中樞並從中提取資料的機制。 

接著，您可以在裝置模擬解決方案的 IoT 中樞「事件」端點上定義新的「取用者群組」屬性。 

1. 在 Azure 入口網站中，移至您為裝置模擬解決方案建立之資源群組的 [概觀] 頁面，然後選取 IoT 中樞資源：

   [![裝置模擬解決方案資源群組](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png#lightbox)

   也請記下為該解決方案產生的 IoT 中樞資源的 [名稱]，因為您後續將會參照它。

2. 向下捲動並選取 [端點] 頁面，然後選取 [事件] 端點。 在端點的 [屬性] 頁面上，在 $Default" 取用者群組下輸入端點的唯一名稱，然後按一下 [儲存]：

   [![裝置模擬解決方案 IoT 中樞端點](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png#lightbox)

### <a name="tsi-environment-configuration-define-an-event-source"></a>TSI 環境組態：定義事件來源

現在，將新的 IoT 中樞「取用者群組」事件端點連線至 TSI 環境，作為「事件來源」。

1. 移至您為 TSI 環境建立之資源群組的 [概觀] 頁面，然後選取 TSI 環境：

   [![TSI 環境資源群組和環境](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png#lightbox)

2. 在 TSI 環境頁面上選取 [事件來源]，然後按一下 [+ 新增]：

   ![TSI 環境概觀](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add.png)

3. 在 [新增事件來源] 頁面上輸入必要參數： 

   參數|說明
   ---|---
   **事件來源名稱** | 需要唯一值，用來為事件來源命名。
   **來源** | 選取 [IoT 中樞]。
   **匯入選項** | 使用預設的 [從可用的訂用帳戶使用 IoT 中樞]。 此選項會使下一個下拉式清單填入可用的訂用帳戶。
   **訂用帳戶** | 選取您用來建立 TSI 環境和裝置模擬資源的相同訂用帳戶。
   **IoT 中樞名稱** | 應預設為您先前記下的 IoT 中樞名稱。 否則，請選取正確的 IoT 中樞。
   **IoT 中樞原則名稱** | 保留為預設值 "iothubowner"。
   **IoT 中樞取用者群組** | 應預設為您先前建立的 IoT 中樞取用者群組的名稱。 否則，請選取正確的取用者群組名稱。 
   **事件序列化格式** | 保留為預設值 "JSON"。
   **時間戳記屬性名稱** | 指定為 "timestamp"。

   完成後，請按一下 [建立] 以新增事件來源。 當您返回資源群組的 [概觀] 頁面時，您將會看到 TSI 環境資源與「時間序列深入解析事件來源」資源。

   ![TSI 環境的新事件來源](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add-event-source.png)

## <a name="run-a-device-simulation-to-stream-data-into-the-tsi-environment"></a>執行裝置模擬以將資料串流處理至 TSI 環境中

所有組態工作皆已完成，現在您可以在 TSI 環境中填入來自模擬裝置的範例資料。

您可以回想在[建立裝置模擬區段](#create-a-device-simulation)時，加速器建立了數個 Azure 資源用以支援解決方案。 連同先前所討論的 IoT 中樞，也產生了 Azure App Service Web 應用程式，以建立及傳輸模擬裝置遙測資料。

1. 返回您的[解決方案加速器儀表板](https://www.azureiotsolutions.com/Accelerators#dashboard)。 如有必要，請使用您在此教學課程中使用的相同 Azure 帳戶重新登入。 現在，您可以按一下「裝置模擬」解決方案下的 [啟動] 按鈕：

     ![解決方案加速器儀表板](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard.png)

2. 裝置模擬 Web 應用程式會在此時啟動，且可能需要幾秒鐘來完成初始載入。 系統也會提示您表示同意，為 Web 應用程式授與「將您登入並讀取您的設定檔」的權限。 此權限可讓應用程式擷取支援應用程式運作所需的使用者設定檔資訊：

     ![裝置模擬 Web 應用程式同意](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png)

3. 在 [模擬設定] 頁面載入後，請輸入必要的參數： 

   參數|說明
   ---|---
   **目標 IoT 中樞** | 選取 [使用預先佈建的 IoT 中樞]。
   **裝置型號** | 選取 [Chiller]。
   **裝置數目**  | 在 [數量] 下輸入 1000。
   **遙測頻率** | 輸入 10 秒。
   **模擬持續時間** | 選取 [結束時間:]，然後輸入 5 分鐘。

   完成後，請按一下 [開始模擬]。 模擬總共將執行 5 分鐘，從 1000 個模擬裝置產生資料，每 10 秒一次。  

   ![裝置模擬設定](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png)

4. 模擬執行時，您會注意到 [訊息總數] 和 [每秒訊息數] 欄位大約每隔 10 秒會更新一次。 模擬大約會在 5 分鐘後結束，並且返回**模擬設定**。

   ![裝置模擬執行中](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png)

## <a name="verify-the-telemetry-data"></a>驗證遙測資料

在最後的這一節中，您會驗證遙測資料是否已產生並儲存在 TSI 環境中。 若要驗證資料，您可以使用 [時間序列深入解析] 總管來查詢和分析遙測資料。

1. 返回 TSI 環境的資源群組 [概觀] 頁面，然後再次選取 TSI 環境：

   [![TSI 環境資源群組和環境](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png#lightbox)

2. 在 TSI 環境的 [概觀] 頁面上，按一下 [時間序列深入解析總管 URL]，以開啟 TSI 總管：

   [![TSI 總管](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png#lightbox)

3. TSI 總管會使用您的 Azure 入口網站帳戶進行載入和驗證。 在初始檢視中，您可以看到 TSI 環境中確實已填入模擬遙測資料的圖表區域。 若要篩選為較小的時間範圍，請選取左上方的下拉式清單。 接著，輸入足以跨越裝置模擬持續時間的時間範圍。 然後按一下搜尋放大鏡：

   [![TSI 總管時間範圍篩選](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png#lightbox)

4. 縮小時間範圍，可讓圖表放大顯示傳輸至 IoT 中樞與 TSI 環境的資料明顯暴增的區域。 同時也請留意右上方的**串流處理完成**文字，此處會顯示找到的事件總數。 您也可以拖曳 [間隔大小] 滑桿，來控制圖表的繪圖資料細微度：

   [![TSI 總管中已篩選過時間範圍的檢視](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png#lightbox)

5. 最後，您也可以按一下某個區域以篩選範圍，然後按一下滑鼠右鍵，再使用**瀏覽事件**在表格式 [事件] 檢視中顯示事件的詳細資料：

   [![TSI 總管中已篩選過時間範圍的檢視和事件](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png#lightbox)

## <a name="clean-up-resources"></a>清除資源

此教學課程建立了數個執行中的 Azure 服務，用以支援 TSI 環境和裝置模擬解決方案。 如果您想要放棄並 (或) 延遲完成此教學課程系列，建議您刪除所有資源，以避免產生不必要的成本。 

從 Azure 入口網站的左側功能表：

1. 按一下 [資源群組] 圖示，然後選取您為 TSI 環境建立的資源群組。 在頁面頂端按一下 [刪除資源群組]、輸入資源群組的名稱，然後按一下 [刪除]。 
2. 按一下 [資源群組] 圖示，然後選取裝置模擬解決方案加速器所建立的資源群組。 在頁面頂端按一下 [刪除資源群組]、輸入資源群組的名稱，然後按一下 [刪除]。 

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 TSI 環境 
> * 建立包含 IoT 中樞的裝置模擬解決方案
> * 將 TSI 環境連線至 IoT 中樞
> * 執行裝置模擬以將資料串流處理至 TSI 環境中
> * 驗證模擬的遙測資料

您現在已了解如何建立您自己的 TSI 環境，並了解如何建置從 TSI 環境中取用資料的 Web 應用程式：

> [!div class="nextstepaction"]
> [建立 Azure 時間序列深入解析單一頁面 Web 應用程式](tutorial-create-tsi-sample-spa.md)


