---
title: 從 Azure Logic Apps 連線到 Trello | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓在 Trello 專案中監視及管理清單、面板、卡片的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: fe7a4377-5c24-4f72-ab1a-6d9d23e8d895
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 4ae8d3dff108f14844c31d7b9d0b0871326832a3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046144"
---
# <a name="monitor-and-manage-trello-with-azure-logic-apps"></a>使用 Azure Logic Apps 來監視及管理 Trello

您可以使用 Azure Logic Apps 和 Trello 連接器，建立自動化的工作和工作流程，以便監視及管理 Trello 清單、卡片、面板、小組成員等等，例如：

* 監視新卡片何時新增至面板和清單。 
* 建立、取得及管理面板、卡片和清單。
* 將註解和成員新增至卡片。
* 列出面板、面板標籤、面板上的卡片、卡片註解、卡片成員、 小組成員以及您是其中成員的小組。 
* 取得小組。

您可以使用觸發程序，從您的 Trello 帳戶收到回應，並且讓輸出可供其他動作使用。 您可以使用動作，該動作會使用您的 Trello 帳戶來執行工作。 您也可以讓其他動作使用 Trello 動作的輸出。 例如，當新卡片新增至面板或清單時，您可以使用 Slack 連接器來傳送訊息。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 Trello 帳戶和使用者認證

  您的認證會授權邏輯應用程式建立連線並存取 Trello 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Trello 帳戶的邏輯應用程式。 若要開始使用 Trello 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 Trello 動作，請使用觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-trello"></a>連線至 Trello

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 針對空白邏輯應用程式，請在搜尋方塊中輸入 "trello" 作為篩選條件。 請在觸發程序清單底下，選取您想要的觸發程序。 

   -或-

   若是現有的邏輯應用程式，請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 
   在搜尋方塊中，輸入 "trello" 作為篩選條件。 
   請在動作清單底下，選取您想要的動作。

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 如果系統提示您登入 Trello，請授與邏輯應用程式的存取權並且登入。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/trello/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)