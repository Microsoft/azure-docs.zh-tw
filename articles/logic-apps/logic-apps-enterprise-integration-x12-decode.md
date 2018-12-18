---
title: 將 X12 訊息解碼 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Azure Logic Apps 中，使用 X12 訊息解碼器來驗證 EDI 並產生通知
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 4fd48d2d-2008-4080-b6a1-8ae183b48131
ms.date: 01/27/2017
ms.openlocfilehash: e3d2a458c2cece5e3f01fdb9e3d403b3fb78dd2b
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43121640"
---
# <a name="decode-x12-messages-in-azure-logic-apps-with-enterprise-integration-pack"></a>使用 Enterprise Integration Pack 將 Azure Logic Apps 中的 X12 訊息解碼

使用 Decode X12 訊息連接器，您可以對照交易夥伴協議來驗證信封、驗證 EDI 和夥伴特定的屬性、將交換分割為交易集或保留整個交換，並產生已處理交易的通知。 若要使用此連接器，您必須將連接器新增至邏輯應用程式中的現有觸發程序。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* Azure 帳戶；您可以建立一個 [免費帳戶](https://azure.microsoft.com/free)
* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)。 您必須有整合帳戶才能使用解碼 X12 訊息連接器。
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)
* 已經在整合帳戶中定義的 [X12 合約](logic-apps-enterprise-integration-x12.md)

## <a name="decode-x12-messages"></a>解碼 X12 訊息

1. [建立邏輯應用程式](quickstart-create-first-logic-app-workflow.md)。

2. 解碼 X12 訊息連接器沒有觸發程序，因此您必須新增觸發程序 (例如要求觸發程序) 來啟動邏輯應用程式。 在 Logic Apps 設計工具中，新增觸發程序，然後將動作新增至您的邏輯應用程式。

3.  在搜尋方塊中，輸入 "X12" 做為篩選條件。 選取 [X12 - 將 X12 訊息]。
   
    ![搜尋 "x12"](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage1.png)  

3. 如果您先前未建立與整合帳戶的任何連線，系統將會提示您立即建立該連線。 替連線命名，然後選取您要連線的整合帳戶。 

    ![提供整合帳戶連線詳細資料](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage4.png)

    具有星號的屬性為必要項目。

    | 屬性 | 詳細資料 |
    | --- | --- |
    | 連線名稱 * |為連接器輸入任何名稱。 |
    | 整合帳戶 * |輸入整合帳戶的名稱。 確定您的整合帳戶和邏輯應用程式位於相同的 Azure 位置。 |

5.  當您完成時，連線詳細資料看起來類似此範例。 若要完成連線建立，請選擇 [建立]。
   
    ![整合帳戶連線詳細資料](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage5.png) 

6. 建立您的連線之後 (如此範例所示)，請選取要解碼的 X12 一般檔案訊息。

    ![整合帳戶連線已建立](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage6.png) 

    例如︰

    ![選取 X12 一般檔案訊息進行解碼](media/logic-apps-enterprise-integration-x12-decode/x12decodeimage7.png) 

   > [!NOTE]
   > 實際訊息內容或訊息陣列的酬載 (正確或不正確) 都是 Base64 編碼的。 因此，您必須指定處理此內容的運算式。
   > 以下是以 XML 處理內容的範例，您可在程式碼檢視中輸入，或是使用設計工具中的運算式產生器來輸入。
   > ``` json
   > "content": "@xml(base64ToBinary(item()?['Payload']))"
   > ```
   > ![內容範例](media/logic-apps-enterprise-integration-x12-decode/content-example.png)
   >


## <a name="x12-decode-details"></a>X12 解碼詳細資料

X12 解碼連接器會執行下列工作︰

* 針對交易夥伴合約驗證信封
* 驗證 EDI 和夥伴特定屬性
  * EDI 結構驗證，以及擴充的結構描述驗證
  * 驗證交換信封的結構。
  * 針對控制結構描述進行信封的結構描述驗證。
  * 針對訊息結構描述進行交易集資料元素的結構描述驗證。
  * 對交易集資料元素執行 EDI 驗證 
* 驗證交換、群組和交易集控制編號並未重複
  * 針對先前已接收的交換檢查交換控制編號。
  * 針對交換中的其他群組控制編號檢查群組控制編號。
  * 針對該群組中其他交易集控制編號檢查交易集控制編號。
* 將交換分割為交易集，或保留整個交換︰
  * 將交換分割為交易集 - 暫止發生錯誤的交易集︰將交換分割為交易集，並剖析每個交易集。 
  X12 Decode 動作只會輸出未通過 `badMessages` 驗證的交易集，並將剩餘的交易輸出到 `goodMessages`。
  * 將交換分割為交易集 - 暫止發生錯誤的交換︰將交換分割為交易集，並剖析每個交易集。 
  如果交換中有一或多個交易集無法通過驗證，X12 Decode 動作會將該交換中的所有交易集輸出到 `badMessages`。
  * 保留交換 - 暫止發生錯誤的交易集︰保留交換並處理整個批次交換。 
  X12 Decode 動作只會輸出未通過 `badMessages` 驗證的交易集，並將剩餘的交易輸出到 `goodMessages`。
  * 保留交換 - 暫止發生錯誤的交換︰保留交換並處理整個批次交換。 
  如果交換中有一或多個交易集無法通過驗證，X12 Decode 動作會將該交換中的所有交易集輸出到 `badMessages`。 
* 產生技術和/或功能確認 (若已設定)。
  * 標頭驗證後會產生技術確認。 技術確認會報告位址接收者處理交換標頭和結尾的狀態。
  * 內文驗證後會產生功能確認。 功能確認會報告處理接收的文件時遇到的每個錯誤

## <a name="view-the-swagger"></a>檢視 Swagger
請參閱 [Swagger 詳細資料](/connectors/x12/)。 

## <a name="next-steps"></a>後續步驟
[深入了解企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md "了解企業整合套件") 

