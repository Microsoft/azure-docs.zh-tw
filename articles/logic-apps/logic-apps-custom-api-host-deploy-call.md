---
title: 從 Azure Logic Apps 部署及呼叫 web API 與 REST API | Microsoft Docs
description: 部署並呼叫 Web API 與 REST API，以便與 Azure Logic Apps 中的工作流程進行系統整合
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, stepsic, LADocs
ms.topic: article
ms.assetid: f113005d-0ba6-496b-8230-c1eadbd6dbb9
ms.date: 05/26/2017
ms.openlocfilehash: 0d53c8355fadf53c81676a1fe3c71f8e0b046630
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126563"
---
# <a name="deploy-and-call-custom-apis-from-workflows-in-azure-logic-apps"></a>在 Azure Logic Apps 中從工作流程部署和呼叫自訂 API

在您[建立可在 Logic Apps 工作流程中所使用的自訂 API](./logic-apps-create-api-app.md) 之後，您在呼叫之前必須先部署您的 API。 您可以將 API 部署為 [web 應用程式](../app-service/app-service-web-overview.md)，但請考慮將您的 API 部署為 [API 應用程式](../app-service/app-service-web-tutorial-rest-api.md)，如此一來，當您在雲端中及內部部署建置、裝載並自訂 API 時，可讓您的作業更容易。 您不需要在 API 中變更任何程式碼 - 只需將您的程式碼部署至 API 應用程式。 您可以將 API 裝載在 [Azure App Service](../app-service/app-service-web-overview.md) 上，這是一個平台即服務 (PaaS) 供應項目，提供擴充性高且簡便的 API 裝載服務。

且雖然您可以從邏輯應用程式呼叫任何 API，但請新增描述您 API 之作業和參數的 [OpenAPI (先前為 Swagger) 中繼資料](http://swagger.io/specification/)以獲得最佳體驗。 OpenAPI 檔案可協助您更輕鬆地整合 API 工作，並改善 Logic Apps。

## <a name="deploy-your-api-as-a-web-app-or-api-app"></a>將您的 API 部署為 web 應用程式或 API 應用程式

請先將您的 API 部署為 web 應用程式或 API 應用程式到 Azure App Service 後，才能從邏輯應用程式呼叫自訂 API。 此外，若要讓 Logic Apps Designer 可讀取您的 OpenAPI 檔案，請設定 API 定義屬性，並開啟 web 應用程式或 API 應用程式的[跨原始資源共用 (CORS)](../app-service/app-service-web-overview.md)。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取您的 Web 應用程式或 API 應用程式。

2. 在開啟的應用程式功能表之 [API] 下方，選擇 [API 定義]。 將 [API 定義位置] 設定為 OpenAPI swagger.json 檔案的 URL。

   通常，URL 會以這種格式出現：`https://{name}.azurewebsites.net/swagger/docs/v1)`

   ![您自訂 API 的 OpenAPI 檔案連結](./media/logic-apps-custom-api-deploy-call/custom-api-swagger-url.png)

3. 在 [API] 下，選擇 [CORS]。 將 [允許的來源] 的 CORS 原則設定為 **'*'** (全部允許)。

   此設定允許來自 Logic App Designer 的要求。

   ![允許來自 Logic App Designer 對您自訂 API 的要求](./media/logic-apps-custom-api-deploy-call/custom-api-cors.png)

如需詳細資訊，請參閱[在 Azure App Service 中裝載具有 CORS 支援的 RESTful API](../app-service/app-service-web-tutorial-rest-api.md)。

## <a name="call-your-custom-api-from-logic-app-workflows"></a>從邏輯應用程式工作流程呼叫自訂 API

在您設定 API 定義屬性和 CORS 之後，您自訂 API 的觸發程序和動作應該就可供您用來包含在邏輯應用程式工作流程中。 

*  若要檢視具有 OpenAPI URL 的網站，您可以在 Logic Apps Designer 中瀏覽訂用帳戶網站。

*  若要藉由指向 OpenAPI 文件來檢視可用的動作和輸入，請使用 [HTTP + Swagger 動作](../connectors/connectors-native-http-swagger.md)。

*  若要呼叫任何 API，包含沒有或未公開 OpenAPI 文件的 API，您一律可以使用 [HTTP 動作](../connectors/connectors-native-http.md)建立要求。

## <a name="next-steps"></a>後續步驟

* [自訂連接器概觀](../logic-apps/custom-connector-overview.md)