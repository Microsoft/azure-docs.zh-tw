---
title: 範例和常見案例 - Azure Logic Apps | Microsoft Docs
description: Azure Logic Apps 的範例、案例、教學課程和逐步解說
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.assetid: e06311bc-29eb-49df-9273-1f05bbb2395c
ms.date: 01/31/18
ms.openlocfilehash: 0d5a7cb97636b9ca3aabf6c4199ce9e41bc1665d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304419"
---
# <a name="common-scenarios-examples-tutorials-and-walkthroughs-for-azure-logic-apps"></a>Azure Logic Apps 的常見情節、範例、教學課程和逐步解說

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) 可協助您協調及整合不同的服務，方法為提供 [100 + 個已備妥可使用的連接器](../connectors/apis-list.md)，範圍從內部部署 SQL Server 或 SAP 到 Microsoft 認知服務。 Logic Apps 服務為「無伺服器」，因此您不必擔心縮放比例或執行個體。 您只需要使用觸發程序和工作流程執行的動作，即可定義觸發程序。 基礎平台可處理調整、可用性和效能。 Logic Apps 特別適用於需要協調跨多個系統之多個動作的使用案例和情節。

為了協助您深入了解 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 支援中的眾多模式和功能，以下提供了常見的範例與情節。

## <a name="popular-starting-points-for-logic-app-workflows"></a>邏輯應用程式工作流程的常用起點

每個邏輯應用程式都是以[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)開頭，並只有一個觸發程序會啟動邏輯應用程式工作流程，並傳遞任何資料作為該觸發程序的一部分。 某些連接器會提供觸發程序，這些類型有：

* 輪詢觸發程序：會定期檢查新資料的服務端點。 當新的資料存在時，觸發程序會使用資料建立並執行新的工作流程執行個體作為輸入。

* 推送觸發程序：在服務端點中接聽資料，並等待特定事件發生。 當事件發生時，會立即引發觸發程序，建立和執行使用任何可用資料作為輸入的新工作流程執行個體。

以下是幾個常用的觸發程序範例：

* 輪詢： 

  * [**排程 - 循環**觸發程序](../connectors/connectors-native-recurrence.md)可讓您設定開始日期和時間，加上引發邏輯應用程式的循環。 
  例如，您可以選取要觸發邏輯應用程式的每週天數和每天時間。

  * 「收到電子郵件時」觸發程序可讓您的邏輯應用程式檢查 Logic Apps 支援的任何電子郵件提供者的新電子郵件，例如 [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md)、[Gmail](https://docs.microsoft.com/connectors/gmail/)、[Outlook.com](https://docs.microsoft.com/connectors/outlook/)，依此類推。

  * [**HTTP** 觸發程序](../connectors/connectors-native-http.md)可讓邏輯應用程式透過 HTTP 進行通訊來檢查指定的服務端點。
  
* 發送：

  * [**要求 / 回應 - 要求**觸發程序](../connectors/connectors-native-reqres.md)可讓邏輯應用程式以某種方式即時接收 HTTP 要求和回應事件。

  * [**HTTP Webhook** 觸發程序](../connectors/connectors-native-webhook.md)可訂閱服務端點，方法為使用該服務註冊回呼 URL。 
  這樣一來，當指定的事件發生時，服務可以只通知觸發程序，讓觸發程序不需要輪詢服務。

接收關於新資料或事件的通知之後，觸發程序會引發、建立新的邏輯應用程式工作流程執行個體，並執行工作流程中的動作。 您可以在整個工作流程中，從觸發程序存取任何資料。 例如，「在新的推文上」觸發程序會將推文內容傳遞到邏輯應用程式執行。 

## <a name="respond-to-triggers-and-extend-actions"></a>回應觸發程序並延伸動作

對於可能沒有已發佈連接器的系統和服務，您也可以延伸邏輯應用程式。

* [建立自訂觸發程序或動作](../logic-apps/logic-apps-create-api-app.md)
* [為工作流程執行設定長時間執行的動作](../logic-apps/logic-apps-create-api-app.md)
* [使用 Webhook 回應外部事件和動作](../logic-apps/logic-apps-create-api-app.md)
* [呼叫、觸發或巢狀處理具有 HTTP 要求同步回應的工作流程](../logic-apps/logic-apps-http-endpoint.md)
* [教學課程︰使用 Logic Apps 和 Power BI 在幾分鐘內建置具有 AI 技術的社交儀表板](http://aka.ms/logicappsdemo)
* [影片︰回應 Twilio SMS Webhook 並傳送文字回應](https://channel9.msdn.com/Blogs/Windows-Azure/Azure-Logic-Apps-Walkthrough-Webhook-Functions-and-an-SMS-Bot)

## <a name="control-flow-error-handling-and-logging-capabilities"></a>控制流程、錯誤處理和記錄功能

邏輯應用程式包含豐富的功能，可用於處理進階控制流程，例如條件、開關、迴圈和範圍。 若要確保解決方案的彈性，您也可以在工作流程中實作錯誤和例外狀況處理。 針對工作流程執行狀態的通知和診斷記錄，Azure Logic Apps 也提供了監視和警示。

* 執行以[條件陳述式](../logic-apps/logic-apps-control-flow-conditional-statement.md)和 [Switch 陳述式](../logic-apps/logic-apps-control-flow-switch-statement.md)為基礎的不同動作
* [使用迴圈重複執行步驟或處理陣列和集合中的項目](../logic-apps/logic-apps-control-flow-loops.md)
* [將動作與範圍群組在一起](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* [在工作流程中撰寫錯誤和例外狀況處理](../logic-apps/logic-apps-exception-handling.md)
* [使用案例︰醫療保健公司如何使用邏輯應用程式的例外狀況處理來處理 HL7 FHIR 工作流程](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
* [開啟現有 Logic Apps 的監視、記錄和警示](../logic-apps/logic-apps-monitor-your-logic-apps.md)
* [建立 Logic Apps 時開啟監視和診斷記錄](../logic-apps/logic-apps-monitor-your-logic-apps-oms.md)

## <a name="deploy-and-manage-logic-apps"></a>部署和管理邏輯應用程式

您可以透過 Visual Studio、Azure DevOps 或其他任何原始檔控制和自動化建置工具，完整地開發及部署邏輯應用程式。 為了支援將工作流程與相依連線部署在資源範本中的功能，邏輯應用程式使用 Azure 資源部署範本。 Visual Studio 工具會自動產生這些範本，您可以將其簽入原始檔控制以便控制版本。

* [使用 Visual Studio 建立和部署邏輯應用程式](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [開啟現有 Logic Apps 的監視、記錄和警示](../logic-apps/logic-apps-monitor-your-logic-apps.md)
* [建立自動部署範本](../logic-apps/logic-apps-create-deploy-template.md)

## <a name="content-types-conversions-and-transformations-within-a-run"></a>執行內的內容類型、轉換 (Conversion) 及轉換 (Transformation)

您可以使用 Azure Logic Apps [工作流程定義語言](http://aka.ms/logicappsdocs)中的許多函數，來存取、轉換 (Convert) 及轉換 (Transform) 多種內容類型。 例如，您可以使用 `@json()` 和 `@xml()` 工作流程運算式，在字串、JSON 和 XML 之間進行轉換 (Convert)。 Logic Apps 引擎會保留內容類型，以支援在服務之間以不失真的方式進行內容傳輸的功能。

* [工作流程運算式在邏輯應用程式中的運作方式](../logic-apps/logic-apps-author-definitions.md)
* [處理非 JSON 內容類型](../logic-apps/logic-apps-content-type.md)，例如 `application/xml`、`application/octet-stream` 和 `multipart/formdata`
* [Azure Logic Apps 的工作流程定義語言結構描述](http://aka.ms/logicappsdocs)

## <a name="other-integrations-and-capabilities"></a>其他整合和功能

邏輯應用程式也提供與許多服務 (例如 Azure Functions、Azure API 管理、Azure App Service) 和自訂 HTTP 端點 (例如 REST 和 SOAP) 的整合。

* [使用 Azure 無伺服器建立即時社交儀表板](../logic-apps/logic-apps-scenario-social-serverless.md)
* [從邏輯應用程式呼叫 Azure Functions](../logic-apps/logic-apps-azure-functions.md)
* [教學課程：使用 Azure Functions 觸發邏輯應用程式](../logic-apps/logic-apps-scenario-function-sb-trigger.md)
* [教學課程：使用 Azure Event Grid 和 Logic Apps 監視虛擬機器變更](../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md)
* [教學課程：建立與 Azure Logic Apps 和 Microsoft 認知服務整合的函式來分析 Twitter 推文情感](../azure-functions/functions-twitter-email.md)
* [教學課程：搭配連線 IoT 中樞和信箱的 Azure Logic Apps 進行 IoT 遠端監視和通知](../iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps.md)
* [部落格︰從邏輯應用程式呼叫 SOAP 端點](https://blogs.msdn.microsoft.com/logicapps/2016/04/07/using-soap-services-with-logic-apps/)

## <a name="end-to-end-scenarios"></a>端對端案例

* [白皮書：使用 Azure 服務 (例如 Logic Apps) 進行端對端案例管理整合](https://aka.ms/enterprise-integration-e2e-case-management-utilities-logic-apps)

## <a name="customer-stories"></a>客戶案例

了解 Azure Logic Apps 以及其他 Azure 服務和 Microsoft 產品如何協助[這些公司](https://aka.ms/logic-apps-customer-stories)提升其靈活度，並藉由簡化、組織、自動化及協調複雜的程序，專注於其核心業務。

## <a name="next-steps"></a>後續步驟

* [使用 JSON 依邏輯應用程式定義建置](../logic-apps/logic-apps-author-definitions.md)
* [處理邏輯應用程式中的錯誤和例外狀況](../logic-apps/logic-apps-exception-handling.md)
* [提交有關改善 Azure Logic Apps 的評論、問題、意見或建議](https://feedback.azure.com/forums/287593-logic-apps)