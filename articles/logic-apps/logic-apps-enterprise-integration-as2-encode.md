---
title: 將 AS2 訊息編碼 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 與 Enterprise Integration Pack 將 AS 訊息編碼
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 332fb9e3-576c-4683-bd10-d177a0ebe9a3
ms.date: 08/08/2018
ms.openlocfilehash: 6bb19199929a004ee5668a3a6e057a69c24dd752
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43122708"
---
# <a name="encode-as2-messages-with-azure-logic-apps-and-enterprise-integration-pack"></a>使用 Azure Logic Apps 與 Enterprise Integration Pack 將 AS2 訊息編碼

若要在傳輸訊息時建立安全性和可靠性，請使用編碼 AS2 訊息連接器。 此連接器可透過訊息處置通知 (MDN) 提供數位簽章、加密和通知，這也可能導致支援不可否認性。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* Azure 帳戶；您可以建立一個 [免費帳戶](https://azure.microsoft.com/free)
* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)。 您必須有整合帳戶才能使用編碼 AS2 訊息連接器。
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)
* 已經在整合帳戶中定義的 [AS2 合約](logic-apps-enterprise-integration-as2.md)

## <a name="encode-as2-messages"></a>編碼 AS2 訊息

1. [建立邏輯應用程式](quickstart-create-first-logic-app-workflow.md)。

2. 編碼 AS2 訊息連接器沒有觸發程序，因此您必須新增觸發程序 (例如要求觸發程序) 來啟動邏輯應用程式。 在 Logic Apps 設計工具中，新增觸發程序，然後將動作新增至您的邏輯應用程式。

3.  在搜尋方塊中，輸入 "AS2" 做為篩選條件。 選取 [AS2 - 編碼 AS2 訊息]。
   
    ![搜尋 "AS2"](./media/logic-apps-enterprise-integration-as2-encode/as2decodeimage1.png)

4. 如果您先前未建立與整合帳戶的任何連線，系統將會提示您立即建立該連線。 替連線命名，然後選取您要連線的整合帳戶。 
   
    ![建立整合帳戶連線](./media/logic-apps-enterprise-integration-as2-encode/as2encodeimage1.png)  

    具有星號的屬性為必要項目。

    | 屬性 | 詳細資料 |
    | --- | --- |
    | 連線名稱 * |為連接器輸入任何名稱。 |
    | 整合帳戶 * |輸入整合帳戶的名稱。 確定您的整合帳戶和邏輯應用程式位於相同的 Azure 位置。 |

5.  當您完成時，連線詳細資料看起來類似此範例。 若要完成連線建立，請選擇 [建立]。
   
    ![整合連線詳細資料](./media/logic-apps-enterprise-integration-as2-encode/as2encodeimage2.png)

6. 建立您的連線之後 (如此範例所示)，提供您合約中設定的 **[AS2-From]** 、**[AS2-To]** 識別碼詳細資訊，以及 **[內文]** \(這是訊息承載)。
   
    ![提供必要欄位](./media/logic-apps-enterprise-integration-as2-encode/as2encodeimage3.png)

## <a name="as2-encoder-details"></a>AS2 編碼器詳細資料

編碼 AS2 連接器會執行下列工作︰ 

* 套用 AS2/HTTP 標頭
* 簽署外寄訊息 (若已設定)
* 加密外寄訊息 (若已設定)
* 壓縮訊息 (若已設定)
* 以 MIME 標頭傳輸檔案名稱 (若已設定)


  > [!NOTE]
  > 如果您使用 Azure 金鑰保存庫來管理憑證，請確定您已設定允許**加密**作業的金鑰。
  > 否則，AS2 編碼會失敗。
  >
  > ![金鑰保存庫解密](media/logic-apps-enterprise-integration-as2-encode/keyvault1.png)

## <a name="try-this-sample"></a>嘗試此範例

若要嘗試部署可完整運作的邏輯應用程式和範例 AS2 案例，請參閱 [AS2 邏輯應用程式範本和案例](https://azure.microsoft.com/documentation/templates/201-logic-app-as2-send-receive/)。

## <a name="view-the-swagger"></a>檢視 Swagger
請參閱 [Swagger 詳細資料](/connectors/as2/)。 

## <a name="next-steps"></a>後續步驟
[深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件") 

