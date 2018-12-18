---
title: 連線到 Box - Azure Logic Apps | Microsoft Docs
description: 使用 Box REST API 和 Azure Logic Apps 建立及管理檔案
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 11/07/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: b5c8c18c6d02710646560f29d4bc7b5784f730a2
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746872"
---
# <a name="create-and-manage-files-in-box-with-azure-logic-apps"></a>使用 Azure Logic Apps 在 Box 中建立和管理檔案

本文說明如何使用 Box 連接器，從邏輯應用程式內部在 Box 中建立和管理檔案。 這樣一來，您就可以建立邏輯應用程式，將用於管理檔案和進行其他動作的工作和工作流程自動化，例如：

* 根據您從 Box 所取得的資料，來建置您的商務流程。 

* 在建立或更新檔案時觸發自動化的工作和工作流程。

* 執行動作來複製檔案、刪除檔案等等。 

  當這些動作取得回應時，就能讓輸出可供其他動作使用。 
  舉例來說，當 Box 中的某個檔案變更時，您可以使用 Office 365 在電子郵件中傳送該檔案。

## <a name="prerequisites"></a>必要條件

* [Box 帳戶](https://www.box.com/home)

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您要在其中存取 Box 帳戶的邏輯應用程式。 若要使用 Box 觸發程序啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識。
如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps](../logic-apps/logic-apps-overview.md)。

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/box/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)