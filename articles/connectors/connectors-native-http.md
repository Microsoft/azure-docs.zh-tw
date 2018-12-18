---
title: 使用 Azure Logic Apps 連線到任何 HTTP 端點 | Microsoft Docs
description: 藉由使用 Azure Logic Apps，將與任何 HTTP 端點通訊的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: e11c6b4d-65a5-4d2d-8e13-38150db09c0b
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: e1561e3be95847efccf487c96bd9c9a8104f161b
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106443"
---
# <a name="call-http-or-https-endpoints-with-azure-logic-apps"></a>使用 Azure Logic Apps 來呼叫 HTTP 或 HTTPS 端點

您可以使用 Azure Logic Apps 和超文字傳輸通訊協定 (HTTP) 連接器，建立自動化的工作流程，藉由建置邏輯應用程式來與任何 HTTP 或 HTTPS 端點通訊。 例如，您可以監視網站的服務端點。 當該端點上發生事件時，例如您的網站停止運作，事件就會觸發邏輯應用程式的工作流程，並執行指定的動作。 

您可以使用 HTTP 觸發程序作為工作流程中的第一個步驟，以定期檢查或「輪詢」端點。 在每次檢查時，觸發程序會將呼叫或「要求」傳送至端點。 端點的回應會決定是否執行邏輯應用程式的工作流程。 觸發程序會將回應的任何內容，傳遞至邏輯應用程式中的動作。 

您可以使用 HTTP 動作作為工作流程中的任何其他步驟，在您想要的時候檢查端點。 端點的回應會決定工作流程的剩餘動作如何執行。

如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您想要呼叫的目標端點 URL 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您想要從其中呼叫目標端點的邏輯應用程式。若要開始使用 HTTP 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 HTTP 動作，請以觸發程序啟動邏輯應用程式。

## <a name="add-http-trigger"></a>新增 HTTP 觸發程序

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟空白邏輯應用程式。

1. 在搜尋方塊中輸入 "http" 作為篩選條件。 在觸發程序清單底下，選取 [HTTP] 觸發程序。 

   ![選取 HTTP 觸發程序](./media/connectors-native-http/select-http-trigger.png)

1. 提供您想要包含在目標端點呼叫中的 [HTTP 觸發程序參數和值](../logic-apps/logic-apps-workflow-actions-triggers.md##http-trigger)。 設定您想要觸發程序檢查目標端點的頻率週期。

   ![輸入 HTTP 觸發程序參數](./media/connectors-native-http/http-trigger-parameters.png)

   如需 HTTP 觸發程序、參數和值的詳細資訊，請參閱[觸發程序和動作類型參考](../logic-apps/logic-apps-workflow-actions-triggers.md##http-trigger)。

1. 請使用當引發觸發程序時執行的動作，繼續建置邏輯應用程式的工作流程。

## <a name="add-http-action"></a>新增 HTTP 動作

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 在想要新增 HTTP 動作的最後一個步驟底下，選擇 [新增步驟]。 

   在此範例中，邏輯應用程式會啟動 HTTP 觸發程序作為第一個步驟。

1. 在搜尋方塊中輸入 "http" 作為篩選條件。 在動作清單底下，選取 [HTTP] 動作。

   ![選取 HTTP 動作](./media/connectors-native-http/select-http-action.png)

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 提供您想要包含在目標端點呼叫中的 [HTTP 動作參數和值](../logic-apps/logic-apps-workflow-actions-triggers.md##http-action)。 

   ![輸入 HTTP 動作參數](./media/connectors-native-http/http-action-parameters.png)

1. 完成後，請確保已儲存邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。 

## <a name="authentication"></a>驗證

若要設定驗證，請在動作或觸發程序內選擇 [顯示進階選項]。 如需 HTTP 觸發程序和動作可用驗證類型的詳細資訊，請參閱[觸發程序和動作類型參考](../logic-apps/logic-apps-workflow-actions-triggers.md#connector-authentication)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)