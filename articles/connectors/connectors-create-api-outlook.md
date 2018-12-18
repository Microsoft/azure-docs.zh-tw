---
title: 連線到 Outlook.com - Azure Logic Apps | Microsoft Docs
description: 使用 Outlook.com REST API 和 Azure Logic Apps 管理電子郵件、行事曆和連絡人
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.assetid: 87113c85-d158-4dd5-9ed5-5748130003d6
ms.topic: article
ms.date: 08/18/2016
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 8030ab9d317c1deefaf441008b9022c4a26bb17c
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746157"
---
# <a name="manage-email-calendars-and-contacts-in-outlookcom-with-azure-logic-apps"></a>使用 Azure Logic Apps 在 Outlook.com 中管理電子郵件、行事曆和連絡人

本文說明如何使用 Box 連接器，從邏輯應用程式內部建立和管理 Outlook.com 帳戶。 這樣一來，您就可以建立邏輯應用程式，來自動執行 Outlook.com 帳戶的工作和工作流程，例如：

* 傳送電子郵件。 
* 排程會議。
* 新增連絡人。 

如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps](../logic-apps/logic-apps-overview.md)。

## <a name="prerequisites"></a>必要條件

* [Outlook.com 帳戶](https://outlook.live.com/owa/)

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您要在其中存取 Outlook.com 帳戶的邏輯應用程式。 若要使用 Outlook.com 觸發程序啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識。

## <a name="connect-to-outlookcom"></a>連接到 Outlook.com

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

[!INCLUDE [Connect to Outlook.com](../../includes/connectors-create-api-outlook.md)]

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/outlook/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)