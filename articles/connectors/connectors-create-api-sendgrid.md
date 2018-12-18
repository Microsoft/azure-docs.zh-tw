---
title: 從 Azure Logic Apps 連線到 SendGrid | Microsoft Docs
description: 使用 Azure Logic Apps 將傳送電子郵件和管理 SendGrid 中郵寄清單的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: bc4f1fc2-824c-4ed7-8de8-e82baff3b746
ms.topic: article
tags: connectors
ms.date: 08/24/2018
ms.openlocfilehash: c8747210a77879d551e323a7c0e46a9ab013fa3f
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42887184"
---
# <a name="send-emails-and-manage-mailing-lists-in-sendgrid-by-using-azure-logic-apps"></a>使用 Azure Logic Apps 傳送電子郵件和管理 SendGrid 中的郵寄清單

使用 Azure Logic Apps 和 SendGrid 連接器，您即可建立可傳送電子郵件和管理收件者清單的自動化工作和工作流程，例如：

* 傳送電子郵件。
* 在清單中新增收件者。
* 取得、新增和管理全域隱藏項目。

您可以在邏輯應用程式中使用 SendGrid 動作以執行這些工作。 您也可以讓其他動作使用 SendGrid 動作的輸出。 

此連接器只提供動作，因此若要啟動邏輯應用程式，請選取個別觸發程序，例如「週期」觸發程序。 例如，如果您要定期地在清單中新增收件者，則可藉由使用 Office 365 Outlook 連接器或 Outlook.com 連接器，來傳送關於收件者和清單的電子郵件。
如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* [SendGrid 帳戶](https://www.sendgrid.com/)和 [SendGrid API 金鑰](https://sendgrid.com/docs/ui/account-and-settings/api-keys/)

   API 金鑰會授權邏輯應用程式建立連線並存取 SendGrid 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 SendGrid 帳戶的邏輯應用程式。 若要使用 SendGrid 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-sendgrid"></a>連線至 SendGrid

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

   * 請在想要新增 SMTP 動作的最後一個步驟底下，選擇 [新增步驟]。 

     -或-

   * 請在想要新增動作的步驟之間，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 在搜尋方塊中，輸入「sendgrid」作為篩選條件。 在動作清單底下，選取您想要的動作。

1. 為您的連線提供一個名稱。 

1. 輸入 SendGrid API 金鑰，然後選擇 [建立]。

1. 為您選取的動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/sendgrid/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)