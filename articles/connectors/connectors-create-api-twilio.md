---
title: 從 Azure Logic Apps 連線到 Twilio | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓透過您的 Twilio 帳戶管理全域簡訊、多媒體訊息和 IP 訊息的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: 43116187-4a2f-42e5-9852-a0d62f08c5fc
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: db7677042737ea1377af54cc02ee1c82c05435c8
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047569"
---
# <a name="manage-messages-in-twilio-with-azure-logic-apps"></a>使用 Azure Logic Apps 管理 Twilio 中的訊息

您可以使用 Azure Logic Apps 和 Twilio 連接器，建立自動化的工作和工作流程，以便取得、傳送及列出 Twilio 中的訊息，其中包含全域簡訊、多媒體訊息及 IP 訊息。 您可以使用這些動作，利用您的 Twilio 帳戶來執行工作。 您也可以讓其他動作使用 Twilio 動作的輸出。 例如，當新訊息送達時，您可以使用 Slack 連接器傳送訊息內容。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 從 [Twilio](https://www.twilio.com/)： 

  * 您的 Twilio 帳戶識別碼和[驗證權杖](https://support.twilio.com/hc/en-us/articles/223136027-Auth-Tokens-and-How-to-Change-Them) (英文)，可以在 Twilio 儀表板上找到

    您的認證會授權邏輯應用程式建立連線，並且從邏輯應用程式存取 Twilio 帳戶。 
    如果您是使用 Twilio 試用帳戶，則只能傳送簡訊給「已驗證」的電話號碼。

  * 已驗證的 Twilio 電話號碼，以便傳送簡訊

  * 已驗證的 Twilio 電話號碼，以便接收簡訊

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Twilio 帳戶的邏輯應用程式。 若要使用 Twilio 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-twilio"></a>連線至 Twilio

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 選擇路徑： 

     * 請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 

       -或-

     * 請在想要新增動作的步驟之間，將指標移至步驟之間的箭號。 
     選擇顯示的加號 (**+**)，然後選取 [新增動作]。
     
       在搜尋方塊中，輸入 "twilio" 作為篩選條件。 
       請在動作清單底下，選取您想要的動作。

1. 為您的連線提供必要的詳細資料，然後選擇 [建立]：

   * 用於連線的名稱
   * 您的 Twilio 帳戶識別碼 
   * 您的 Twilio 存取 (驗證) 權杖

1. 為您選取的動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/twilio/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)