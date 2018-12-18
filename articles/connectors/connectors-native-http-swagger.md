---
title: 從 Azure Logic Apps 呼叫 REST Twilio | Microsoft Docs
description: 使用 Azure Logic Apps 中的 HTTP + Swagger 連接器，自動執行可與 REST 端點通訊的工作和工作流程
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, jehollan, LADocs
ms.assetid: eccfd87c-c5fe-4cf7-b564-9752775fd667
tags: connectors
ms.topic: article
ms.date: 07/18/2016
ms.openlocfilehash: e96e271fbb50a2485a22fab061ea160dc00cf3d6
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123167"
---
# <a name="call-rest-endpoints-with-http--swagger-connector-in-azure-logic-apps"></a>透過 Azure Logic Apps 中的 HTTP + Swagger 連接器呼叫 REST 端點

當您在邏輯應用程式工作流程中使用 HTTP + Swagger 動作時，可以透過 [Swagger 文件](https://swagger.io)在任一 REST 端點建立第一級的連接器。 您也可以使用第一級的邏輯應用程式設計工具體驗，擴充邏輯應用程式以呼叫任何 REST 端點。

若要了解如何利用連接器建立邏輯應用程式，請參閱[建立新的邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="use-http--swagger-as-a-trigger-or-an-action"></a>使用 HTTP + Swagger 做為觸發程序或動作

HTTP + Swagger 觸發程序和動作的功能與 [HTTP 動作](connectors-native-http.md)相同，但能從 [Swagger 中繼資料](https://swagger.io)公開 API 的結構和輸出，因此可以提供較佳的邏輯應用程式設計工具體驗。 您也可以使用 HTTP + Swagger 連接器作為觸發程序。 如果您想要實作輪詢觸發程序，請依照[建立自訂 API 來呼叫邏輯應用程式的其他 API、服務和系統](../logic-apps/logic-apps-create-api-app.md#polling-triggers)中所述的輪詢模式進行。

深入了解 [邏輯應用程式觸發程序和動作](connectors-overview.md)。

以下是如何使用 HTTP + Swagger 作業做為邏輯應用程式中工作流程動作的範例。

1. 選取 [新增步驟]  按鈕。
2. 選取 [新增動作]。
3. 在動作搜尋方塊中，輸入 **swagger** 以列出 HTTP + Swagger 動作。
   
    ![選取 HTTP + Swagger 動作](./media/connectors-native-http-swagger/using-action-1.png)
4. 輸入 Swagger 文件的 URL：
   
   * 若要從邏輯應用程式設計工具使用，此 URL 必須是 HTTPS 端點並已啟用 CORS。
   * 如果 Swagger 文件不符合此需求，您可以使用 [已啟用 CORS 的 Azure 儲存體](#hosting-swagger-from-storage) 來儲存文件。
5. 按一下 [下一步]  以從 Swagger 文件讀取和轉譯。
6. 新增 HTTP 呼叫所需的任何參數。
   
    ![完成 HTTP 動作](./media/connectors-native-http-swagger/using-action-2.png)
7. 若要儲存並發佈您的邏輯應用程式，請按一下設計工具工具列中的 [儲存]。

### <a name="host-swagger-from-azure-storage"></a>從 Azure 儲存體託管 Swagger
您可能想要參考未託管的 Swagger 文件，但這並不符合設計工具的安全性及跨原始來源需求。 為解決這個問題，您可以將 Swagger 文件儲存在 Azure 儲存體並啟用 CORS 來參考該文件。  

以下是在 Azure 儲存體中建立、設定和儲存 Swagger 文件的步驟：

1. [使用 Azure Blob 儲存體建立 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md) 若要執行此步驟，請將權限設為 [公用存取]。

2. 在 Blob 啟用 CORS。 

   您可以使用 [這個 PowerShell 指令碼](https://github.com/logicappsio/EnableCORSAzureBlob/blob/master/EnableCORSAzureBlob.ps1) 自動設定此設定。

3. 將 Swagger 檔案上傳至 Blob。 

   您可以從 [Azure 入口網站](https://portal.azure.com)或 [Azure 儲存體總管](http://storageexplorer.com/)等工具執行這個步驟。

4. 參考 Azure Blob 儲存體中文件的 HTTPS 連結。 

   此連結用此格式：

   `https://*storageAccountName*.blob.core.windows.net/*container*/*filename*`

## <a name="technical-details"></a>技術詳細資訊
以下是此 HTTP + Swagger 連接器所支援觸發程序和動作的詳細資料。

## <a name="http--swagger-triggers"></a>HTTP + Swagger 觸發程序
觸發程序是一個事件，可用來啟動邏輯應用程式中定義的工作流程。 [深入了解觸發程序。](connectors-overview.md) HTTP + Swagger 連接器有一個觸發程序。

| 觸發程序 | 說明 |
| --- | --- |
| HTTP + Swagger |進行 HTTP 呼叫並傳回回應內容 |

## <a name="http--swagger-actions"></a>HTTP + Swagger 動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作。](connectors-overview.md) HTTP + Swagger 連接器有一個可能的動作。

| 動作 | 說明 |
| --- | --- |
| HTTP + Swagger |進行 HTTP 呼叫並傳回回應內容 |

### <a name="action-details"></a>動作詳細資料
HTTP + Swagger 連接器隨附一個可能的動作。 以下是關於每個動作、其必要和選擇性輸入欄位，以及與其使用方式相關聯的對應輸出詳細資料的資訊。

#### <a name="http--swagger"></a>HTTP + Swagger
在 Swagger 中繼資料的協助下提出 HTTP 輸出要求。
星號 (*) 表示必要的欄位。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 方法 * |method |要使用的 HTTP 指令動詞。 |
| URI* |uri |HTTP 要求的 URI。 |
| headers |headers |要包含之 HTTP 標頭的 JSON 物件。 |
| body |body |HTTP 要求本文。 |
| 驗證 |驗證 |用於要求的驗證。 如需詳細資訊，請參閱 [HTTP 連接器](connectors-native-http.md#authentication)。 |

**輸出詳細資料**

HTTP 回應

| 屬性名稱 | 資料類型 | 說明 |
| --- | --- | --- |
| headers |物件 |回應標頭 |
| body |物件 |回應物件 |
| Status Code |整數 |HTTP 狀態碼 |

### <a name="http-responses"></a>HTTP 回應
呼叫不同動作時，您可能會收到特定回應。 下表概述對應的回應及說明。

| 名稱 | 說明 |
| --- | --- |
| 200 |OK |
| 202 |已接受 |
| 400 |不正確的要求 |
| 401 |未經授權 |
| 403 |禁止 |
| 404 |找不到 |
| 500 |內部伺服器錯誤。 發生未知錯誤。 |

- - -
## <a name="next-steps"></a>後續步驟

* [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [尋找其他連接器](apis-list.md)