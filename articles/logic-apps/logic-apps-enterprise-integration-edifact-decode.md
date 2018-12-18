---
title: 將 EDIFACT 訊息解碼 - Azure Logic Apps | Microsoft Docs
description: 針對採用 Enterprise Integration Pack 的 Azure Logic Apps，使用 EDIFACT 訊息解碼器驗證 EDI 及產生通知
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 0e61501d-21a2-4419-8c6c-88724d346e81
ms.date: 01/27/2017
ms.openlocfilehash: b101922d15a3f90c29eff51c223d2ea7dc30ddf2
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125347"
---
# <a name="decode-edifact-messages-for-azure-logic-apps-with-the-enterprise-integration-pack"></a>使用企業整合套件將 Azure Logic Apps 的 EDIFACT 訊息解碼

使用 Decode EDIFACT 訊息連接器，您可以驗證 EDI 和夥伴特定的屬性、將交換分割為交易集或保留整個交換，並產生已處理交易的通知。 若要使用此連接器，您必須將連接器新增至邏輯應用程式中的現有觸發程序。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* Azure 帳戶；您可以建立一個 [免費帳戶](https://azure.microsoft.com/free)
* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)。 您必須有整合帳戶才能使用解碼 EDIFACT 訊息連接器。 
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)
* 已經在整合帳戶中定義的 [EDIFACT 合約](logic-apps-enterprise-integration-edifact.md)

## <a name="decode-edifact-messages"></a>解碼 EDIFACT 訊息

1. [建立邏輯應用程式](quickstart-create-first-logic-app-workflow.md)。

2. 解碼 EDIFACT 訊息連接器沒有觸發程序，因此您必須新增觸發程序 (例如要求觸發程序) 來啟動邏輯應用程式。 在 Logic Apps 設計工具中，新增觸發程序，然後將動作新增至您的邏輯應用程式。

3. 在搜尋方塊中，輸入 "EDIFACT" 做為篩選條件。 選取 [將 EDIFACT 訊息解碼]。
   
    ![搜尋 EDIFACT](./media/logic-apps-enterprise-integration-edifact-decode/edifactdecodeimage1.png)

3. 如果您先前未建立與整合帳戶的任何連線，系統將會提示您立即建立該連線。 替連線命名，然後選取您要連線的整合帳戶。
   
    ![建立整合帳戶](./media/logic-apps-enterprise-integration-edifact-decode/edifactdecodeimage2.png)

    具有星號的屬性為必要項目。

    | 屬性 | 詳細資料 |
    | --- | --- |
    | 連線名稱 * |為連接器輸入任何名稱。 |
    | 整合帳戶 * |輸入整合帳戶的名稱。 確定您的整合帳戶和邏輯應用程式位於相同的 Azure 位置。 |

4. 當您完成連線建立時，請選擇 [建立]。 您的連線詳細資料看起來類似此範例：

    ![整合帳戶詳細資料](./media/logic-apps-enterprise-integration-edifact-decode/edifactdecodeimage3.png)  

5. 建立您的連線之後 (如此範例所示)，請選取要解碼的 EDIFACT 一般檔案訊息。

    ![整合帳戶連線已建立](./media/logic-apps-enterprise-integration-edifact-decode/edifactdecodeimage4.png)  

    例如︰

    ![選取 EDIFACT 一般檔案訊息以便解碼](./media/logic-apps-enterprise-integration-edifact-decode/edifactdecodeimage5.png)  

## <a name="edifact-decoder-details"></a>EDIFACT 解碼器詳細資料

解碼 EDIFACT 連接器會執行下列工作︰ 

* 針對交易夥伴協議驗證信封。
* 比對傳送者辨識符號和識別項，以及接收者辨識符號和識別項，以解析合約。
* 在交換具有多個交易時，根據合約的接收設定組態，將交換分割成多筆交易。
* 解譯交換。
* 驗證 EDI 和夥伴特定屬性，包括：
  * 驗證交換信封結構
  * 針對控制結構描述進行信封的結構描述驗證
  * 針對訊息結構描述進行交易集資料元素的結構描述驗證
  * 對交易集資料元素執行 EDI 驗證
* 驗證交換、群組和交易集控制編號並未重複 (若已設定) 
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
* 產生技術 (控制) 和/或功能確認 (若已設定)。
  * 技術確認或 CONTRL ACK 會報告完整接收的交換所進行的語法檢查結果。
  * 功能確認會確認接受或拒絕已接收的交換或群組

## <a name="view-swagger-file"></a>檢視 Swagger 檔案
若要檢視 EDIFACT 連接器的 Swagger 詳細資料，請參閱 [EDIFACT](/connectors/edifact/)。

## <a name="next-steps"></a>後續步驟
[深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件") 

