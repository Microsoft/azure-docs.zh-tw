---
title: 從 Azure Logic Apps 連線到 Twitter | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓監視及管理推文，加上從您的 Twitter 帳戶取得跟隨者、您跟隨的使用者、其他使用者、時間軸等等相關資訊的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: 8bce2183-544d-4668-a2dc-9a62c152d9fa
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: eea70d979a69a4855b6eeb892d1705ecadaa8434
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918640"
---
# <a name="monitor-and-manage-twitter-by-using-azure-logic-apps"></a>藉由使用 Azure Logic Apps 來監視及管理 Twitter

您可以使用 Azure Logic Apps 和 Twitter 連接器，建立自動化工作和工作流程，以便監視及管理您在 Twitter 中關心的資料，例如推文、跟隨者、使用者和跟隨的使用者、時間軸等等，以及其他動作，例如：

* 監視、貼文和搜尋推文。
* 取得例如跟隨者、跟隨的使用者、時間軸等等的資料。

您可以使用觸發程序，從您的 Twitter 帳戶收到回應，並且讓輸出可供其他動作使用。 您可以使用動作，該動作會使用您的 Twitter 帳戶來執行工作。 您也可以讓其他動作使用 Twitter 動作的輸出。 例如，當具有特定主題標籤的新推文出現時，您可以使用 Slack 連接器來傳送訊息。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 Twitter 帳戶和使用者認證

   您的認證會授權邏輯應用程式建立連線並存取 Twitter 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Twitter 帳戶的邏輯應用程式。 若要開始使用 Twitter 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 Twitter 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-twitter"></a>連接到 Twitter

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

   * 針對空白邏輯應用程式，請在搜尋方塊中輸入 "twitter" 作為篩選條件。 
   請在觸發程序清單底下，選取您想要的觸發程序。 

     -或-

   * 針對現有的邏輯應用程式： 
   
     * 請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 

       -或-

     * 請在想要新增動作的步驟之間，將指標移至步驟之間的箭號。 
     選擇顯示的加號 (**+**)，然後選取 [新增動作]。
     
       在搜尋方塊中，輸入 "twitter" 作為篩選條件。 
       請在動作清單底下，選取您想要的動作。

1. 如果系統提示您登入 Twitter，請立即登入，方可授與邏輯應用程式存取權。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="examples"></a>範例

### <a name="twitter-trigger-when-a-new-tweet-is-posted"></a>Twitter 觸發程序：有新推文張貼時

此觸發程序會在偵測到新推文時 (例如，具有主題標籤 #Seattle 時)，啟動邏輯應用程式工作流程。 所以舉例來說，當找到這些推文時，您可以將具有推文內容的檔案新增至儲存體，例如藉由使用 Dropbox 連接器新增到 Dropbox 帳戶。 

您可以選擇性地包含條件，要求合格的推文必須來自至少具有指定跟隨者數目的使用者。

**企業範例**：您可以使用此觸發程序來監視與貴公司有關的推文，以及將推文內容上傳至 SQL 資料庫。

### <a name="twitter-action-post-a-tweet"></a>Twitter 動作：張貼推文

此動作會張貼推文，但是您可以設定動作，讓推文包含由先前所述觸發程序找到的推文內容。 

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/twitterconnector/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
