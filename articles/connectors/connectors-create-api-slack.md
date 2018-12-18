---
title: 從 Azure Logic Apps 連線到 Slack | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓在 Slack 帳戶中監視檔案及管理通道、群組和訊息的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: 234cad64-b13d-4494-ae78-18b17119ba24
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 7af2db528866d687064e854e00e43e81d2601b2b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042320"
---
# <a name="monitor-and-manage-slack-with-azure-logic-apps"></a>使用 Azure Logic Apps 來監視及管理 Slack

您可以使用 Azure Logic Apps 和 Slack 連接器，建立自動化的工作和工作流程，以便監視 Slack 檔案及管理 Slack 通道、訊息、群組等等，例如：

* 監視新檔案何時建立。
* 建立、列出及加入通道 
* 張貼訊息。
* 建立群組並設定請勿打擾。

您可以使用觸發程序，從您的 Slack 帳戶收到回應，並且讓輸出可供其他動作使用。 您可以使用動作，該動作會使用您的 Slack 帳戶來執行工作。 您也可以讓其他動作使用 Slack 動作的輸出。 例如，建立新檔案時，您可以使用 Office 365 Outlook 連接器來傳送電子郵件。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 [Slack](https://slack.com/) (英文) 帳戶和使用者認證

  您的認證會授權邏輯應用程式建立連線並存取 Slack 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Slack 帳戶的邏輯應用程式。 若要開始使用 Slack 觸發程序，[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 Slack 動作，請使用觸發程序來啟動邏輯應用程式，例如 Slack 觸發程序或其他觸發程序，例如「週期」觸發程序。

## <a name="connect-to-slack"></a>連線至 Slack

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 針對空白邏輯應用程式，請在搜尋方塊中輸入 "slack" 作為篩選條件。 請在觸發程序清單底下，選取您想要的觸發程序。 

   -或-

   若是現有的邏輯應用程式，請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 
   在搜尋方塊中，輸入 "slack" 作為篩選條件。 
   請在動作清單底下，選取您想要的動作。

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 如果系統提示您登入 Slack，請登入您的 Slack 工作區。 

   ![登入 Slack 工作區](./media/connectors-create-api-slack/slack-sign-in-workspace.png)

1. 授權邏輯應用程式的存取。

   ![將存取權授權給 Slack](./media/connectors-create-api-slack/slack-authorize-access.png)

1. 為您選取的觸發程序或動作提供必要的詳細資料。 若要繼續建置邏輯應用程式的工作流程，請新增更多動作。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/slack/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
