---
title: 適用於 B2B 企業整合的 AS2 訊息 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 AS2 訊息
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: c9b7e1a9-4791-474c-855f-988bd7bf4b7f
ms.date: 06/08/2017
ms.openlocfilehash: 2604cdd6bf758858328c2d30fc4cde535f0a7148
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124657"
---
# <a name="exchange-as2-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 AS2 訊息

您必須先建立 AS2 合約並將該合約儲存在您的整合帳戶中，才可以交換 AS2 訊息。 以下是如何建立 AS2 合約的步驟。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)，以及在 [企業身分識別] 之下設定 AS2 限定詞。

> [!NOTE]
> 當您建立合約時，合約檔案中的內容必須與合約類型相符。    

在您[建立整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)並[新增夥伴](logic-apps-enterprise-integration-partners.md)之後，您可以依照下列步驟建立 AS2 合約。

## <a name="create-an-as2-agreement"></a>建立 AS2 合約

1.  登入 [Azure 入口網站](http://portal.azure.com "Azure 入口網站")。  

2. 在主要 Azure 功能表上，選取 [所有服務]。 在搜尋方塊中輸入「整合」，然後選取 [整合帳戶]。

   ![尋找您的整合帳戶](./media/logic-apps-enterprise-integration-as2/overview-1.png)

   > [!TIP]
   > 如果沒有看到 [所有服務]，您可能必須先展開功能表。 在摺疊功能表的頂端，選取 [顯示文字標籤]。

3. 在 [整合帳戶] 底下，選取要在其中建立合約的整合帳戶。

   ![選取您要在其中建立合約的整合帳戶](./media/logic-apps-enterprise-integration-overview/overview-3.png)

4. 選擇 [合約] 圖格。 如果您沒有 [合約] 圖格，請先新增圖格。

    ![選擇 [合約] 圖格](./media/logic-apps-enterprise-integration-as2/agreement-1.png)

5. 在 [合約] 之下，選擇 [新增]。

    ![選擇 [新增]](./media/logic-apps-enterprise-integration-as2/agreement-2.png)

6. 在 [新增] 之下，輸入合約的 [名稱]。 針對 [合約類型]，選取 **AS2**。 選取合約的 [主機夥伴]、[主機身分識別]、[來賓夥伴] 和 [來賓身分識別]。

    ![提供合約詳細資料](./media/logic-apps-enterprise-integration-as2/agreement-3.png)  

    | 屬性 | 說明 |
    | --- | --- |
    | 名稱 |合約的名稱 |
    | 合約類型 | 應該是 AS2 |
    | 主控夥伴 |合約需要主控夥伴和來賓夥伴。 主機夥伴代表設定合約的組織。 |
    | 主控身分識別 |主控夥伴的識別碼 |
    | 來賓夥伴 |合約需要主控夥伴和來賓夥伴。 來賓夥伴代表與主控夥伴有生意往來的組織。 |
    | 來賓身分識別 |來賓合作夥伴的識別碼 |
    | 接收設定 |這些屬性會套用到合約所接收的所有訊息。 |
    | 傳送設定 |這些屬性會套用到合約所傳送的所有訊息。 |

## <a name="configure-how-your-agreement-handles-received-messages"></a>設定合約處理所收到訊息的方式

您現在已經設定合約屬性，您可以設定此合約如何識別及處理您透過此合約從夥伴接收的內送訊息。

1.  在 [新增] 之下，選取 [接收設定]。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

    ![設定 [接收設定]](./media/logic-apps-enterprise-integration-as2/agreement-4.png)

2. 您也可以透過選取 [覆寫訊息屬性]，覆寫內送訊息的屬性。

3. 若要要求所有內送訊息都必須經過簽署，請選取 [訊息必須經過簽署]。 從 [憑證] 清單中選取現有的[來賓夥伴公開憑證](../logic-apps/logic-apps-enterprise-integration-certificates.md)，以便驗證訊息上的簽章。 如果您沒有憑證，請建立一個。

4.  若要要求加密所有內送訊息，請選取 [訊息必須加密]。 從 [憑證] 清單，選取現有[主機夥伴私人憑證](../logic-apps/logic-apps-enterprise-integration-certificates.md)，以便將內送訊息解密。 如果您沒有憑證，請建立一個。

5. 若要要求訊息必須壓縮，請選取 [訊息必須壓縮]。

6. 若要針對已接收的訊息傳送同步郵件處置通知 (MDN)，請選取 [傳送 MDN]。

7. 若要針對已接收的訊息傳送已簽署的 MDN，請選取 [傳送經過簽署的 MDN]。

8. 若要針對已接收的訊息傳送非同步 MDN，請選取 [傳送非同步 MDN]。

9. 完成後，請務必選擇 [確定] 以儲存設定。

您的合約現在已準備好處理符合您所選設定的內送訊息。

| 屬性 | 說明 |
| --- | --- |
| 覆寫訊息屬性 |指出所接收訊息中可被覆寫的屬性。 |
| 訊息應該簽署 |要求訊息必須經過數位簽署。 設定來賓合作夥伴公開憑證以進行簽章驗證。  |
| 訊息應該加密 |要求訊息必須經過加密。 未加密的訊息會遭到拒絕。 設定主控夥伴私人憑證以進行訊息解密。  |
| 訊息應該壓縮 |要求訊息必須經過壓縮。 未壓縮的訊息會遭到拒絕。 |
| MDN 文字 |要傳送給郵件寄件者的預設郵件處置通知 (MDN)。 |
| 傳送 MDN |要求傳送 MDN。 |
| 傳送簽署的 MDN |要求簽署 MDN。 |
| MIC 演算法 |選取要用於簽署訊息的演算法。 |
| 傳送非同步 MDN | 要求以非同步方式傳送訊息。 |
| URL | 指定要傳送 MDN 的 URL。 |

## <a name="configure-how-your-agreement-sends-messages"></a>設定合約傳送訊息的方式

您可以設定此合約如何識別及處理您透過此合約傳送給夥伴的外寄訊息。

1.  在 [新增] 之下，選取 [傳送設定]。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

    ![設定 [傳送設定] 屬性](./media/logic-apps-enterprise-integration-as2/agreement-51.png)

2. 若要將經過簽署的訊息傳送給夥伴，請選取 [啟用訊息簽署]。 若要簽署訊息，請在 [MIC 演算法] 清單中，選取「主機夥伴私人憑證 MIC 演算法」。 並且在 [憑證] 清單中，選取現有[主機夥伴私人憑證](../logic-apps/logic-apps-enterprise-integration-certificates.md)。

3. 若要將加密的訊息傳送給夥伴，請選取 [啟用訊息加密]。 若要加密訊息，請在 [加密演算法] 清單中，選取「來賓夥伴公開憑證演算法」。
並且在 [憑證] 清單中，選取現有[來賓夥伴公開憑證](../logic-apps/logic-apps-enterprise-integration-certificates.md)。

4. 若要壓縮訊息，請選取 [啟用訊息壓縮]。

5. 若要將 HTTP content-type 標頭展開為單一行，請選取 [展開 HTTP 標頭]。

6. 若要接收所傳送訊息的同步 MDN，請選取 [要求 MDN]。

7. 若要接收所傳送訊息的已簽署 MDN，請選取 [要求經過簽署的 MDN]。

8. 若要接收所傳送訊息的同步 MDN，請選取 [要求非同步 MDN]。 若選取此選項，請輸入 MDN 傳送目標的 URL。

9. 若要要求接收的不可否認性，請選取 [啟用 NRR]。  

10. 若要指定要用於 MIC 或簽署外寄 AS2 訊息或 MDN 的標題中之演算法格式，選取 **SHA2 演算法格式**。  

11. 完成後，請務必選擇 [確定] 以儲存設定。

您的合約現在已準備好處理符合您所選設定的外寄訊息。

| 屬性 | 說明 |
| --- | --- |
| 啟用訊息簽署 |要求傳送自合約的所有訊息都必須經過簽署。 |
| MIC 演算法 |用於簽署訊息的演算法。 設定主控夥伴私人憑證 MIC 演算法以簽署訊息。 |
| 憑證 |選取要用於簽署訊息的憑證。 設定主控夥伴私人憑證以簽署訊息。 |
| 啟用訊息加密 |要求傳送自此合約的所有訊息都必須經過加密。 設定來賓合作夥伴公開憑證演算法以加密訊息。 |
| 加密演算法 |要用於訊息加密的加密演算法。 設定來賓合作夥伴公開憑證以加密訊息。 |
| 憑證 |要用於加密訊息的憑證。 設定來賓合作夥伴私人憑證以加密訊息。 |
| 啟用訊息壓縮 |要求傳送自此合約的所有訊息都必須經過壓縮。 |
| 展開 HTTP 標頭 |請將 HTTP 內容類型標頭放在一行中。 |
| 要求 MDN |針對傳送自此合約的所有訊息要求 MDN。 |
| 要求簽署的 MDN |要求傳送到此合約的所有 MDN 都必須經過簽署。 |
| 要求非同步 MDN |要求將非同步 MDN 傳送到此合約。 |
| URL |指定要傳送 MDN 的 URL。 |
| 啟用 NRR |要求接收的不可否認性 (NRR)，這是一種通訊屬性，它會提供資料已由預訂收件者接收的證據。 |
| SHA2 演算法格式 |選取要用於 MIC 或簽署外寄 AS2 訊息或 MDN 的標題中之演算法格式 |

## <a name="find-your-created-agreement"></a>尋找您建立的合約

1. 完成所有合約屬性的設定之後，請在 [新增] 頁面中選擇 [確定]，以完成合約建立並回到整合帳戶。

    您新增的合約現在顯示於您的 [合約] 清單中。

2. 您也可以在整合帳戶概觀中檢視您的合約。 在整合帳戶功能表上，選擇 [概觀]，然後選取 [合約] 圖格。 

   ![選擇 [合約] 圖格來檢視所有合約](./media/logic-apps-enterprise-integration-as2/agreement-6.png)

## <a name="view-the-swagger"></a>檢視 Swagger
請參閱 [Swagger 詳細資料](/connectors/as2/)。 

## <a name="next-steps"></a>後續步驟
* [深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件")  
