---
title: Azure Functions 概觀 | Microsoft Docs
description: 了解如何使用 Azure Functions，在數分鐘內最佳化非同步的工作負載。
services: functions
documentationcenter: na
author: mattchenderson
manager: jeconnoc
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.assetid: 01d6ca9f-ca3f-44fa-b0b9-7ffee115acd4
ms.service: azure-functions
ms.devlang: multiple
ms.topic: overview
ms.date: 10/03/2017
ms.author: glenga
ms.custom: H1Hack27Feb2017, mvc
ms.openlocfilehash: f9b60b9365f8856db8ffd8e14d95d46af3808ffb
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49116482"
---
# <a name="an-introduction-to-azure-functions"></a>Azure Functions 簡介  
Azure Functions 是可在雲端輕鬆執行程式碼片段或「函數」的解決方案。 您可以只撰寫處理手邊問題所需的程式碼，而不需擔心要執行它的整個應用程式或基礎結構。 Functions 可讓開發更有生產力，而且您可以使用您選擇的開發語言，例如 C#、F#、Node.js、Java 或 PHP。 只需對您的程式碼執行的時間付費，並信任 Azure 視需要調整。 Azure Functions 可讓您在 Microsoft Azure 上開發 [無伺服器](https://azure.microsoft.com/solutions/serverless/)應用程式。

本主題提供 Azure Functions 的高階概觀。 如果您想要直接進入正題並開始使用 Functions，請從 [建立您的第一個 Azure Functions](functions-create-first-azure-function.md) 著手。 如果您要尋找更多有關 Functions 的技術資訊，請參閱 [開發人員參考](functions-reference.md)。

## <a name="features"></a>特性
以下是 Functions 的一些主要功能︰

* **語言選擇**  - 使用選擇的 C#、F# 或 Javascript 撰寫函式。 請參閱[支援的語言](supported-languages.md)了解其他選項。
* **使用即付費價格模式** - 只對執行您的程式碼所花的時間付費。 請參閱[價格區段](#pricing)中的使用情況主控方案選項。  
* **自備相依性** - Functions 支援 NuGet 和 NPM，以便您使用您最愛的程式庫。  
* **整合式安全性** - 利用 OAuth 提供者 (如 Azure Active Directory、Facebook、Google、Twitter 和 Microsoft 帳戶) 保護 HTTP 觸發的函數。  
* **簡化整合** - 輕鬆地利用 Azure 服務和軟體即服務 (SaaS) 供應項目。 請參閱[整合區段](#integrations)以取得相關範例。  
* **彈性開發** - 直接在入口網站中撰寫函式的程式碼，或透過 [GitHub](../app-service/scripts/app-service-cli-continuous-deployment-github.md)、[Azure DevOps Services](../app-service/scripts/app-service-cli-continuous-deployment-vsts.md) 與其他[支援的開發工具](../app-service/app-service-deploy-local-git.md)設定連續整合並部署您的程式碼。  
* **開放原始碼** - Functions 執行階段是開放原始碼的平台並 [可在 GitHub 上取得](https://github.com/azure/azure-webjobs-sdk-script)。  

## <a name="what-can-i-do-with-functions"></a>我可以用 Functions 來做什麼？
Functions 是處理資料、整合系統、使用物聯網 (IoT)，以及建置簡單 API 和微服務的絕佳解決方案。 考慮將 Functions 用於如下的工作：映像或訂單處理、檔案維護，或者您要排程執行的任何工作。 

Functions 提供範本，可讓您開始使用重要的案例，包括下列案例︰

* **HTTPTrigger** - 使用 HTTP 要求觸發程式碼的執行。 如需範例，請參閱[建立您的第一個函式](functions-create-first-azure-function.md)。
* **TimerTrigger** - 在預先定義的排程執行清除或其他批次工作。 如需範例，請參閱[建立由計時器觸發的函式](functions-create-scheduled-function.md)。
* **CosmosDBTrigger** - 當 Azure Cosmos DB 文件在 NoSQL 資料庫的集合中新增或更新時進行處理。 如需詳細資訊，請參閱 [Azure Cosmos DB 繫結](functions-bindings-cosmosdb-v2.md)。
* **BlobTrigger** - 在新增至容器時，處理 Azure 儲存體 blob。 您可以使用此函式調整映像大小。 如需詳細資訊，請參閱 [Blob 儲存體繫結](functions-bindings-storage-blob.md)。
* **QueueTrigger** - 在訊息送達 Azure 儲存體佇列中時回應。 如需詳細資訊，請參閱 [Azure 佇列儲存體繫結](functions-bindings-storage-queue.md)。
* **EventGridTrigger** -  回應傳遞到 Azure 事件方格中之訂用帳戶的事件。 支援可接收事件 (包含篩選) 的訂用帳戶型模型。 適合建置事件型架構的解決方案。 如需範例，請參閱[使用事件方格自動調整已上傳映像的大小](../event-grid/resize-images-on-storage-blob-upload-event.md)。
* **EventHubTrigger** - 回應傳送到 Azure 事件中樞的事件。 特別適合用於應用程式檢測、使用者經驗或工作流程處理及物聯網 (IoT) 案例。 如需詳細資訊，請參閱[事件中樞繫結](functions-bindings-event-hubs.md)。
* **ServiceBusQueueTrigger** - 將程式碼連接至其他 Azure 服務或內部部署服務，方法是接聽訊息佇列。 如需詳細資訊，請參閱[服務匯流排繫結](functions-bindings-service-bus.md)。
* **ServiceBusTopicTrigger** - 將程式碼連接至其他 Azure 服務或內部部署服務，方法是訂閱主題。 如需詳細資訊，請參閱[服務匯流排繫結](functions-bindings-service-bus.md)。

Azure Functions 支援「觸發」，這是開始執行您的程式碼的方式，以及「繫結」，這是針對輸入和輸出資料簡化編碼的方式。 如需 Azure Functions 提供的觸發和繫結的詳細說明，請參閱 [Azure Functions 觸發和繫結開發人員參考](functions-triggers-bindings.md)。

## <a name="integrations"></a>整合
Azure Functions 可以與各種 Azure 和協力廠商服務整合。 這些服務可以觸發您的函式並開始執行，或做為您程式碼的輸入和輸出。 Azure Functions 支援下列服務整合：

* Azure Cosmos DB
* Azure 事件中心
* Azure Event Grid
* Azure 通知中心
* Azure 服務匯流排 (佇列和主題)
* Azure 儲存體 (Blob、佇列和資料表)
* 內部部署 (使用服務匯流排)
* Twilio (SMS 訊息)

## <a name="pricing"></a>Functions 的計費方式
Azure Functions 有兩種定價方案。 請選擇最符合您需求的方案： 

* **使用情況方案**：當您的函式執行時，Azure 會提供所有必要的運算資源。 您不必擔心資源管理，您只需為您的程式碼執行時間支付費用。 
* **App Service 方案** - 執行函式就像執行 Web 應用程式。 當您已準備對其他應用程式使用 App Service 時，您可以在相同方案上執行您的函數，不會產生額外的費用。 

如需主控方案的詳細資訊，請參閱 [Azure Functions 主控方案比較](functions-scale.md)。 在 [Functions 價格](https://azure.microsoft.com/pricing/details/functions/)頁面上可取得完整的價格詳細資料。

## <a name="next-steps"></a>後續步驟
* [建立您的第一個Azure Functions](functions-create-first-azure-function.md)  
  直接進入正題並使用 Azure Functions 快速入門建立您的第一個函數。 
* [Azure Functions 開發人員參考](functions-reference.md)  
  提供更多有關 Azure Functions 執行階段的技術資訊，以及可供撰寫函數程式碼及定義觸發程序和繫結時參考。
* [測試 Azure Functions](functions-test-a-function.md)  
  說明可用於測試函式的各種工具和技巧。
* [如何調整 Azure 函式](functions-scale.md)  
  討論 Azure Functions 可用的服務方案，包括使用情況主控方案，以及如何選擇正確的方案。 
* [深入了解 Azure App Service](../app-service/app-service-web-overview.md)  
  Azure Functions 會利用 Azure App Service 來執行核心功能，例如部署、環境變數和診斷。 

