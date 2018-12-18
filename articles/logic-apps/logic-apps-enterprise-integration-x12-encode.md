---
title: 將 X12 訊息編碼 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Azure Logic Apps 中，使用 X12 訊息編碼器來驗證 EDI 及轉換 XML 編碼的訊息
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: a01e9ca9-816b-479e-ab11-4a984f10f62d
ms.date: 01/27/2017
ms.openlocfilehash: 3ed5cb61fef5f07913f11c4e4df309d720d5b901
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123500"
---
# <a name="encode-x12-messages-in-azure-logic-apps-with-enterprise-integration-pack"></a>使用 Enterprise Integration Pack 將 Azure Logic Apps 中的 X12 訊息編碼

使用編碼 X12 訊息連接器，可以驗證 EDI 和夥伴特定屬性，交換時將 XML 編碼的訊息轉換成 EDI 交易集，以及要求技術通知、功能通知或兩者。
若要使用此連接器，您必須將連接器新增至邏輯應用程式中的現有觸發程序。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* Azure 帳戶；您可以建立一個 [免費帳戶](https://azure.microsoft.com/free)
* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)。 您必須有整合帳戶才能使用編碼 X12 訊息連接器。
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)
* 已經在整合帳戶中定義的 [X12 合約](logic-apps-enterprise-integration-x12.md)

## <a name="encode-x12-messages"></a>編碼 X12 訊息

1. [建立邏輯應用程式](quickstart-create-first-logic-app-workflow.md)。

2. 編碼 X12 訊息連接器沒有觸發程序，因此您必須新增觸發程序 (例如要求觸發程序) 來啟動邏輯應用程式。 在 Logic Apps 設計工具中，新增觸發程序，然後將動作新增至您的邏輯應用程式。

3.  在搜尋方塊中，輸入 "X12" 做為篩選條件。 選取 [X12 - 依照合約名稱編碼為 X12 訊息] 或是 [X12 - 依照身分識別編碼為 X12 訊息]。
   
    ![搜尋 "x12"](./media/logic-apps-enterprise-integration-x12-encode/x12decodeimage1.png) 

3. 如果您先前未建立與整合帳戶的任何連線，系統將會提示您立即建立該連線。 替連線命名，然後選取您要連線的整合帳戶。 
   
    ![整合帳戶連線](./media/logic-apps-enterprise-integration-x12-encode/x12encodeimage1.png)

    具有星號的屬性為必要項目。

    | 屬性 | 詳細資料 |
    | --- | --- |
    | 連線名稱 * |為連接器輸入任何名稱。 |
    | 整合帳戶 * |輸入整合帳戶的名稱。 確定您的整合帳戶和邏輯應用程式位於相同的 Azure 位置。 |

5.  當您完成時，連線詳細資料看起來類似此範例。 若要完成連線建立，請選擇 [建立]。

    ![整合帳戶連線已建立](./media/logic-apps-enterprise-integration-x12-encode/x12encodeimage2.png)

    現在已建立您的連線。

    ![整合帳戶連線詳細資料](./media/logic-apps-enterprise-integration-x12-encode/x12encodeimage3.png) 

#### <a name="encode-x12-messages-by-agreement-name"></a>由協議名稱編碼 X12 訊息

如果您選擇依照合約名稱將 X12 訊息編碼，請開啟 [X12 合約名稱] 清單，輸入或選取現有的 X12 合約。 輸入 XML 訊息進行編碼。

![輸入 X12 合約名稱和 XML 訊息進行編碼](./media/logic-apps-enterprise-integration-x12-encode/x12encodeimage4.png)

#### <a name="encode-x12-messages-by-identities"></a>由身分識別編碼 X12 訊息

如果您選擇依照身分識別將 X12 訊息編碼，請輸入傳送者識別碼、傳送者限定詞、接收者識別碼和接收者限定詞，如 X12 合約中所設定。 選取要編碼的 XML 訊息。
   
![提供傳送者與收件者的身分識別，選取要編碼的 XML 訊息](./media/logic-apps-enterprise-integration-x12-encode/x12encodeimage5.png) 

## <a name="x12-encode-details"></a>X12 編碼詳細資料

X12 編碼連接器會執行下列工作︰

* 藉由比對傳送者和接收者內容屬性進行合約解析。
* 序列化 EDI 交換，將 XML 編碼訊息轉換為交換中的 EDI 交易集。
* 套用交易集標頭和結尾區段
* 產生交換控制編號、群組控制編號和每個傳出交換的交易集控制編號
* 取代承載資料中的分隔符號
* 驗證 EDI 和夥伴特定屬性
  * 針對訊息結構描述進行交易集資料元素的結構描述驗證
  * 對交易集資料元素執行 EDI 驗證。
  * 對交易集資料元素執行擴充驗證
* 要求技術和/或功能確認 (若已設定)。
  * 標頭驗證後會產生技術確認。 技術確認會報告位址接收者處理交換標頭和結尾的狀態
  * 內文驗證後會產生功能確認。 功能確認會報告處理接收的文件時遇到的每個錯誤

## <a name="view-the-swagger"></a>檢視 Swagger
請參閱 [Swagger 詳細資料](/connectors/x12/)。 

## <a name="next-steps"></a>後續步驟
[深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件") 

