---
title: 從 Azure Logic Apps 連線至 Project Online | Microsoft Docs
description: 使用 Azure Logic Apps，將監視、建立和管理 Project Online 專案、工作和資源的工作流程自動化
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
ms.topic: article
ms.assetid: 40ce621e-4925-4653-93bb-71ab9abcbdf1
tags: connectors
ms.date: 08/24/2018
ms.openlocfilehash: cfcb53b6e95250a1ccbebfdfcfbff5ec8479504b
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42886948"
---
# <a name="manage-project-online-projects-tasks-and-resources-by-using-azure-logic-apps"></a>使用 Azure Logic Apps 來管理 Project Online 專案、工作和資源

使用 Azure Logic Apps 和 Project Online 連接器，您即可透過 Office 365 為 Project Online 中的專案、工作和資源，建立自動化的工作和工作流程。 工作流程可以執行下列動作和其他工作，例如：

* 監視新建立的專案、工作或資源。 或者，監視新發佈的專案。
* 建立新的專案、工作或資源。
* 列出現有的專案或工作。
* 簽出、簽入或發佈專案。

Project Online 可藉由提供強大的專案管理功能，協助您規劃、設定優先順序以及管理專案和專案組合投資，不受場地和裝置的限制。 您可以使用 Project Online 觸發程序，以取得 Project Online 的回應，並且讓輸出可供其他動作使用。 您可以在邏輯應用程式中使用動作，以在 Project Online 中執行各種工作。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* Project Online，可透過 [Office 365 帳戶](https://www.office.com/)取得。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Project Online 資料的邏輯應用程式。 若要開始使用 Project Online 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 Project Online 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-project-online"></a>連線至 Project Online

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

   * 針對空白邏輯應用程式，在搜尋方塊中輸入「Project Online」作為篩選條件。 
   在觸發程序清單底下，選取您想要的觸發程序。 

     -或-

   * 若是現有的邏輯應用程式，請在想要新增動作的步驟底下，選擇 [新增步驟]。 在搜尋方塊中輸入「Project Online」作為篩選條件。 在動作清單底下，選取您想要的動作。

1. 如果系統提示您登入 Project Online，請立即登入。

   您的認證會授權邏輯應用程式建立與 Project Online 的連線，並存取資料。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/projectonline/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)