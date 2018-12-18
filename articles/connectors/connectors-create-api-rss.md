---
title: 從 Azure Logic Apps 連線到 RSS 摘要 | Microsoft Docs
description: 使用 Azure Logic Apps 將監視和管理 RSS 摘要的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
ms.topic: article
ms.assetid: a10a6277-ed29-4e68-a881-ccdad6fd0ad8
tags: connectors
ms.date: 08/24/2018
ms.openlocfilehash: c8d1122572764dda1fc550a06ae254109e3bf033
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42885793"
---
# <a name="manage-rss-feeds-by-using-azure-logic-apps"></a>使用 Azure Logic Apps 來管理 RSS 摘要

使用 Azure Logic Apps 和 RSS 連接器，您即可為 RSS 摘要建立自動化的工作和工作流程，例如：

* 監視所發佈的 RSS 摘要項目。
* 列出所有 RSS 摘要項目。

RSS (具意網站摘要) 也稱為真正簡易新聞訂閱方式，是一種熱門的 Web 新聞訂閱方式格式，可用來發佈經常更新的內容，例如，部落格文章和頭條新聞。 許多內容發佈者會提供 RSS 摘要供使用者訂閱該內容。 

您可以使用 RSS 觸發程序，以取得 RSS 摘要的回應，並且讓輸出可供其他動作使用。 您可以在邏輯應用程式中使用 RSS 動作來執行具有 RSS 摘要的工作。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* RSS 摘要的 URL

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 要在其中存取 RSS 摘要的邏輯應用程式。 若要開始使用 RSS 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 RSS 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-an-rss-feed"></a>連線到 RSS 摘要

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

   * 針對空白邏輯應用程式，請在搜尋方塊中輸入「rss」作為篩選條件。 在觸發程序清單底下，選取您想要的觸發程序。 

     -或-

   * 若是現有的邏輯應用程式，請在想要新增動作的步驟底下，選擇 [新增步驟]。 在搜尋方塊中，輸入 "rss" 作為篩選條件。 在動作清單底下，選取您想要的動作。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/rss/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)