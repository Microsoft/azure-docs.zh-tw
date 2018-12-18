---
title: 從 Azure Logic Apps 連線到 SMTP | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓透過 SMTP (簡易郵件傳輸通訊協定) 帳戶傳送電子郵件的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: d4141c08-88d7-4e59-a757-c06d0dc74300
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 90af33574093cfbe529093c7091ee6988f043aa6
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052017"
---
# <a name="send-email-from-your-smtp-account-with-azure-logic-apps"></a>使用 Azure Logic Apps 從您的 SMTP 帳戶傳送電子郵件

您可以使用 Azure Logic Apps 和簡易郵件傳輸通訊協定 (SMTP) 連接器，建立自動化的工作和工作流程，以便從您的 SMTP 帳戶傳送電子郵件。 您也可以讓其他動作使用 SMTP 動作的輸出。 例如，在您的 SMTP 傳送電子郵件之後，您可以使用 Slack 連接器，在 Slack 中通知您的小組。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 SMTP 帳戶和使用者認證

  您的認證會授權邏輯應用程式建立連線並存取 SMTP 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 SMTP 帳戶的邏輯應用程式。 若要使用 SMTP 動作，請使用觸發程序來啟動邏輯應用程式，例如 Salesforce 觸發程序 (如果您有 Salesforce 帳戶)。

  例如，您可以使用**記錄建立時機** Salesforce 觸發程序來啟動邏輯應用程式。 
  此觸發程序會在每次於 Salesforce 中建立新記錄 (例如潛在客戶) 時引發。 
  然後，您可以使用 SMTP [傳送電子郵件] 動作，來追蹤此觸發程序。 這樣一來，在建立新記錄時，邏輯應用程式會從您的 SMTP 帳戶，傳送與新記錄相關的電子郵件。

## <a name="connect-to-smtp"></a>連接到 SMTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 請在想要新增 SMTP 動作的最後一個步驟底下，選擇 [新增步驟]。 

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 在搜尋方塊中，輸入 "smtp" 作為篩選條件。 請在動作清單底下，選取您想要的動作。

1. 出現提示時，請提供這項連線資訊：

   | 屬性 | 必要 | 說明 |
   |----------|----------|-------------|
   | 連線名稱 | 是 | 與 SMTP 伺服器的連線名稱 | 
   | **SMTP 伺服器位址** | 是 | SMTP 伺服器的位址 | 
   | **使用者名稱** | 是 | SMTP 帳戶的使用者名稱 | 
   | **密碼** | 是 | SMTP 帳戶的密碼 | 
   | **SMTP 伺服器連接埠** | 否 | SMTP 伺服器上您想要使用的特定連接埠 | 
   | **是否啟用 SSL？** | 否 | 開啟或關閉 SSL 加密。 | 
   |||| 

1. 為您選取的動作提供必要的詳細資料。 

1. 儲存您的邏輯應用程式，或繼續建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/smtpconnector/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)