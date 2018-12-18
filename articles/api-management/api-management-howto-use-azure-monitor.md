---
title: 在 Azure API 管理中監視發佈的原則 | Microsoft Docs
description: 依照此教學課程的步驟，了解如何在 Azure API 管理中監視您的 API。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: apimpm
ms.openlocfilehash: b4c885758f572851f058edb6e7851d650faed9f9
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972993"
---
# <a name="monitor-published-apis"></a>監視發佈的 API

您可以使用 Azure 監視器來視覺化、查詢、路由、封存及針對來自 Azure 資源的度量或記錄採取行動。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 檢視活動記錄檔
> * 檢視診斷記錄
> * 檢視 API 的計量 
> * 在 API 收到未經授權的呼叫時，設定警示規則

下列影片示範如何使用 Azure 監視器來監視 API 管理。 

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Monitor-API-Management-with-Azure-Monitor/player]
>
>

## <a name="prerequisites"></a>先決條件

+ 完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
+ 同時也請完成下列教學課程：[匯入和發佈您的第一個 API](import-and-publish.md)。

## <a name="view-metrics-of-your-apis"></a>檢視 API 的計量

API 管理會每分鐘發出計量，讓您近乎即時地了解 API 的狀態和健康情況。 以下是一些可用計量的摘要：

* 容量 (預覽)：協助您決定是否升級/降級 APIM 服務。 計量每分鐘發出，並反映提出報告時的閘道容量。 計量的範圍為 0 到 100，是根據 CPU 和記憶體使用率等閘道資源計算而來。
* 閘道要求總數︰該期間內的 API 要求數目。 
* 成功的閘道要求︰收到 HTTP 成功回應碼的 API 要求數目，這些回應碼包括 304、307 和任何小於 301 的代碼 (例如 200)。
* 失敗的閘道要求︰收到 HTTP 錯誤回應碼的 API 要求數目，這些回應碼包括 400 和任何大於 500 的代碼。
* 未經授權閘道器要求︰收到 401、403 和 429 HTTP 回應碼的 API 要求數目。
* 其他閘道要求︰所收到的 HTTP 回應碼不屬於上述任何類別 (例如 418) 的 API 要求數目。

存取計量：

1. 從靠近頁面底部的功能表中，選取 [計量]。

    ![metrics](./media/api-management-azure-monitor/api-management-metrics-blade.png)

2. 從下拉式清單中，選取您想了解的計量 (您可以新增多個度量)。  
    例如，從可用的度量清單中選取 [閘道要求總數] 和 [失敗的閘道要求]。
3. 該圖表會顯示 API 呼叫的總數。 也會顯示失敗的 API 呼叫數目。

    ![計量圖表](./media/api-management-azure-monitor/apim-monitor-metrics.png)

## <a name="set-up-an-alert-rule-for-unauthorized-request"></a>針對未經授權的要求設定警示規則

您可以進行設定來收到以計量和活動記錄為基礎的警示。 Azure 監視器可讓您將警示設定為在觸發時執行下列動作︰

* 傳送電子郵件通知
* 呼叫 Webhook
* 叫用 Azure 邏輯應用程式

設定警示：

1. 從靠近頁面底部的功能表列中，選取 [警示 (傳統)]。

    ![alerts](./media/api-management-azure-monitor/api-management-alert-rules-blade.png)

2. 選取 [新增計量警示]。
3. 輸入此警示的**名稱**。
4. 選取 [未經授權的閘道要求] 作為要監視的計量。
5. 選取**電子郵件擁有者、參與者和讀取者**。
6. 按 [確定]。
7. 嘗試呼叫沒有 API 金鑰的會議 API。 身為此 API 管理服務的擁有者，您會收到電子郵件警示。 

    > [!TIP]
    > 警示規則也可以在觸發時呼叫 Web Hook 或 Azure Logic App。

    ![set-up-alert](./media/api-management-azure-monitor/set-up-alert.png)

## <a name="activity-logs"></a>活動記錄檔

活動記錄可讓您深入了解 API 管理服務上所執行的作業。 您可以使用活動記錄來判斷 API 管理服務上所執行之任何寫入作業 (PUT、POST、DELETE) 的「內容、對象和時間」。

> [!NOTE]
> 活動記錄不會納入讀取 (GET) 作業，也不會納入透過 Azure 入口網站或原始管理 API 所執行的作業。

您可以在 API 管理服務中存取活動記錄，或在 Azure 監視器中存取所有 Azure 資源的記錄。 

檢視活動記錄：

1. 選取您的 APIM 服務執行個體。
2. 按一下 [活動記錄]。

    ![活動記錄](./media/api-management-azure-monitor/api-management-activity-logs-blade.png)

3. 選取所需的篩選範圍，然後按一下 [套用]。

    ![活動記錄](./media/api-management-azure-monitor/apim-monitor-activity-logs.png)

## <a name="diagnostic-logs"></a>診斷記錄檔

診斷記錄可提供豐富的作業與錯誤資訊，這些資訊對於稽核和疑難排解用途來說很重要。 診斷記錄與活動記錄不同。 活動記錄可讓您深入了解 Azure 資源上所執行的作業。 診斷記錄能讓您了解資源執行的作業。

若要設定診斷記錄：

1. 選取您的 APIM 服務執行個體。
2. 按一下 [ **診斷記錄**]。

    ![診斷記錄](./media/api-management-azure-monitor/api-management-diagnostic-logs-blade.png)

3. 按一下 [開啟診斷]。 您可以將診斷記錄連同計量封存至儲存體帳戶，將它們串流處理至事件中樞，或將它們傳送至 Log Analytics。 

API 管理目前提供關於個別 API 要求的診斷記錄 (每小時提供一批)，且每個要求項目都有下列結構描述︰

```json
{  
    "isRequestSuccess" : "",
    "time": "",
    "operationName": "",
    "category": "",
    "durationMs": ,
    "callerIpAddress": "",
    "correlationId": "",
    "location": "",
    "httpStatusCodeCategory": "",
    "resourceId": "",
    "properties": {   
        "method": "", 
        "url": "", 
        "clientProtocol": "", 
        "responseCode": , 
        "backendMethod": "", 
        "backendUrl": "", 
        "backendResponseCode": ,
        "backendProtocol": "",  
        "requestSize": , 
        "responseSize": , 
        "cache": "", 
        "cacheTime": "", 
        "backendTime": , 
        "clientTime": , 
        "apiId": "",
        "operationId": "", 
        "productId": "", 
        "userId": "", 
        "apimSubscriptionId": "", 
        "backendId": "",
        "lastError": { 
            "elapsed" : "", 
            "source" : "", 
            "scope" : "", 
            "section" : "" ,
            "reason" : "", 
            "message" : ""
        } 
    }      
}  
```

| 屬性  | 類型 | 說明 |
| ------------- | ------------- | ------------- |
| isRequestSuccess | 布林值 | 如果已完成 HTTP 要求，但回應狀態碼在 2xx 或 3xx 範圍內，則為 true |
| 分析 | date-time | 閘道接收 HTTP 要求的時間戳記 |
| operationName | 字串 | 常數值 'Microsoft.ApiManagement/GatewayLogs' |
| category | 字串 | 常數值 'GatewayLogs' |
| durationMs | integer | 從閘道收到要求直到傳入完整回應時的毫秒數 |
| callerIpAddress | 字串 | 立即閘道呼叫端的 IP 位址 (可以是中繼項目) |
| correlationId | 字串 | API 管理所指派的唯一 http 要求識別碼 |
| location | 字串 | 處理要求的閘道所在的 Azure 區域名稱 |
| httpStatusCodeCategory | 字串 | http 回應狀態碼的類別：成功 (301 或更小或 304 或 307)、未經授權 (401、403、429)、錯誤 (400，介於 500 與 600 之間)、其他 |
| ResourceId | 字串 | "API 管理資源的識別碼 /SUBSCRIPTIONS/<subscription>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.APIMANAGEMENT/SERVICE/<name> |
| properties | 物件 | 目前要求的屬性 |
| method | 字串 | 連入要求的 HTTP 方法 |
| url | 字串 | 連入要求的 URL |
| clientProtocol | 字串 | 連入要求的 HTTP 通訊協定版本 |
| responseCode | integer | 傳送至用戶端之 HTTP 回應的狀態碼 |
| backendMethod | 字串 | 傳送至後端之要求的 HTTP 方法 |
| backendUrl | 字串 | 傳送至後端之要求的 URL |
| backendResponseCode | integer | 從後端接收之 HTTP 回應的代碼 |
| backendProtocol | 字串 | 傳送至後端之要求的 HTTP 通訊協定版本 | 
| requestSize | integer | 在要求處理期間從用戶端接收的位元組數目 | 
| responseSize | integer | 在要求處理期間傳送至用戶端的位元組數目 | 
| cache | 字串 | 要求處理中 API 管理快取參與的狀態 (亦即，命中、錯過、無) | 
| cacheTime | integer | 整體 API 管理快取 IO (連線、傳送及接收位元組) 所耗費的毫秒數 | 
| backendTime | integer | 整體後端 IO (連線、傳送及接收位元組) 所耗費的毫秒數 | 
| clientTime | integer | 整體用戶端 IO (連線、傳送及接收位元組) 所耗費的毫秒數 | 
| apiId | 字串 | 目前要求的 API 實體識別碼 | 
| operationId | 字串 | 目前要求的作業實體識別碼 | 
| productId | 字串 | 目前要求的產品實體識別碼 | 
| userId | 字串 | 目前要求的使用者實體識別碼 | 
| apimSubscriptionId | 字串 | 目前要求的訂用帳戶實體識別碼 | 
| backendId | 字串 | 目前要求的後端實體識別碼 | 
| LastError | 物件 | 上次要求處理錯誤 | 
| elapsed | integer | 從閘道收到要求直到發生錯誤時所經過的毫秒數 | 
| 來源 | 字串 | 導致錯誤的原則或處理內部處理常式名稱 | 
| scope | 字串 | 包含導致錯誤之原則的原則文件範圍 | 
| section | 字串 | 包含導致錯誤之原則的原則文件區段 | 
| 原因 | 字串 | 錯誤原因 | 
| Message | 字串 | 錯誤訊息 | 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 檢視活動記錄檔
> * 檢視診斷記錄
> * 檢視 API 的計量
> * 在 API 收到未經授權的呼叫時，設定警示規則

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [追蹤呼叫](api-management-howto-api-inspector.md)
