---
title: 從 Azure Logic Apps 連線到 Wunderlist | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓在 Wunderlist 帳戶中監視及管理清單、工作、提醒等等的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: e4773ecf-3ad3-44b4-a1b5-ee5f58baeadd
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 7226b59504c7112c039061ab0c184fe14f6e59d0
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918666"
---
# <a name="monitor-and-manage-wunderlist-by-using-azure-logic-apps"></a>藉由使用 Azure Logic Apps 來監視及管理 Wunderlist

您可以使用 Azure Logic Apps 和 Wunderlist 連接器，建立自動化的工作和工作流程，以便在您的 Wunderlist 帳戶中監視及管理待辦事項清單、提醒等等，還有其他動作，例如：

* 監視新工作何時建立、工作何時到期，或者提醒何時發生。
* 建立及管理清單、附註、工作、子工作等等。
* 設定提醒。
* 取得清單、工作、子工作、提醒、檔案、附註、註解等等。

[Wunderlist](https://www.wunderlist.com/) 是一項服務，可協助您規劃、管理及完成您的專案、待辦事項清單和工作 - 在任何裝置、任何位置。 您可以使用觸發程序，從您的 Wunderlist 帳戶收到回應，並且讓輸出可供其他動作使用。 您可以使用動作，該動作會使用您的 Wunderlist 帳戶來執行工作。 您也可以讓其他動作使用 Wunderlist 動作的輸出。 例如，當新工作到期時，您可以使用 Slack 連接器張貼訊息。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 Wunderlist 帳戶和使用者認證

   您的認證會授權邏輯應用程式建立連線並存取 Wunderlist 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Yammer 帳戶的邏輯應用程式。 若要開始使用 Wunderlist 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 Wunderlist 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-wunderlist"></a>連線至 Wunderlist

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

   * 針對空白邏輯應用程式，請在搜尋方塊中輸入 "wunderlist" 作為篩選條件。 
   請在觸發程序清單底下，選取您想要的觸發程序。 

     -或-

   * 針對現有的邏輯應用程式： 
   
     * 請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 

       -或-

     * 請在想要新增動作的步驟之間，將指標移至步驟之間的箭號。 
     選擇顯示的加號 (**+**)，然後選取 [新增動作]。
     
       在搜尋方塊中，輸入 "wunderlist" 作為篩選條件。 
       請在動作清單底下，選取您想要的動作。

1. 如果系統提示您登入 Wunderlist，請立即登入以便允許存取。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/wunderlist/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)