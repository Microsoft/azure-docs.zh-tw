---
title: 從 Azure Logic Apps 連線到 SharePoint | Microsoft Docs
description: 藉由使用 Azure Logic Apps，將在 SharePoint Online 或 SharePoint Server 內部部署中監視及管理資源的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: e0ec3149-507a-409d-8e7b-d5fbded006ce
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 5a3968e1aec98092767712220d8aaf676aba89cd
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43053065"
---
# <a name="monitor-and-manage-sharepoint-resources-with-azure-logic-apps"></a>使用 Azure Logic Apps 來監視及管理 SharePoint 資源

您可以使用 Azure Logic Apps 和 SharePoint 連接器，建立自動化的工作和工作流程，以便在 SharePoint Online 或 SharePoint Server 內部部署中監視及管理資源，例如檔案、資料夾、清單、項目、人員等等，例如：

* 監視檔案或項目何時建立、變更或刪除。
* 建立、取得、更新或刪除項目。
* 新增、取得或刪除附件。 從附件取得內容。
* 建立、複製、更新或刪除檔案。 
* 更新檔案屬性。 取得檔案的內容、中繼資料或屬性。
* 列出或擷取資料夾。
* 取得清單或清單檢視。
* 設定內容核准狀態。
* 解析人員。
* 將 HTTP 要求傳送到 SharePoint。
* 取得實體的值。

您可以使用觸發程序，從 SharePoint 收到回應，並且讓輸出可供其他動作使用。 您可以在邏輯應用程式中使用動作，在 SharePoint 中執行工作。 您也可以讓其他動作使用 SharePoint 動作的輸出。 例如，如果您定期從 SharePoint 擷取檔案，可以使用 Slack 連接器將訊息傳送給您的小組。
如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 SharePoint 網站位址和使用者認證

  您的認證會授權邏輯應用程式建立連線，並存取 SharePoint 帳戶。 

* 在將邏輯應用程式連線至內部部署系統 (例如，SharePoint Server) 之前，您必須先[安裝及設定內部部署資料閘道](../logic-apps/logic-apps-gateway-install.md)。 如此一來，您就可以在為邏輯應用程式建立 SharePoint Server 連線時，指定使用您的閘道安裝。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 SharePoint 帳戶的邏輯應用程式。 若要開始使用 SharePoint 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 SharePoint 動作，請使用觸發程序來啟動您的邏輯應用程式，例如 Salesforce 觸發程序 (如果您有 Salesforce 帳戶)。

  例如，您可以使用「記錄建立時機」Salesforce 觸發程序來啟動邏輯應用程式。 
  此觸發程序會在每次於 Salesforce 中建立新記錄 (例如潛在客戶) 時引發。 
  然後，您可以使用 SharePoint「建立檔案」動作，來追蹤此觸發程序。 這樣一來，在建立新記錄時，邏輯應用程式會在 SharePoint 中建立檔案，該檔案具有新記錄的相關資訊。

## <a name="connect-to-sharepoint"></a>連線至 SharePoint

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 針對空白邏輯應用程式，請在搜尋方塊中輸入 "sharepoint" 作為篩選條件。 在觸發程序清單底下，選取您想要的觸發程序。 

   -或-

   若是現有的邏輯應用程式，請在想要新增 SharePoint 動作的最後一個步驟底下，選擇 [新增步驟]。 
   在搜尋方塊中，輸入 "sharepoint" 作為篩選條件。 
   在動作清單底下，選取您想要的動作。

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 當系統提示您登入時，請提供必要的連線資訊。 如果您使用 SharePoint Server，請確定選取 [透過內部部署資料閘道來連線]。 完成之後，請選擇 [建立]。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/sharepoint/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
