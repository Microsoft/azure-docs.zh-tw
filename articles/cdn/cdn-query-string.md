---
title: 使用查詢字串來控制 Azure CDN 快取行為 - 標準層 | Microsoft Docs
description: Azure CDN 查詢字串快取可控制當 Web 要求包含查詢字串時，要如何快取檔案。 本文說明標準 Azure CDN 產品中的查詢字串快取。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 17410e4f-130e-489c-834e-7ca6d6f9778d
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/11/2018
ms.author: magattus
ms.openlocfilehash: f0dab3dc81c626e3e7f8c79b4142e5eb4f2a1276
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093794"
---
# <a name="control-azure-cdn-caching-behavior-with-query-strings---standard-tier"></a>使用查詢字串來控制 Azure CDN 快取行為 - 標準層
> [!div class="op_single_selector"]
> * [標準層](cdn-query-string.md)
> * [進階層](cdn-query-string-premium.md)
> 

## <a name="overview"></a>概觀
使用 Azure 內容傳遞網路 (CDN)，可以控制 Web 要求內含查詢字串時的檔案快取方式。 在包含查詢字串的 Web 要求中，查詢字串是要求中問號 (?) 之後的部分。 查詢字串可以包含一或多個索引鍵/值組，其中的欄位名稱與其值是以等號 (=) 分隔。 每個索引鍵/值組是以 & 符號分隔。 例如 http:\//www.contoso.com/content.mov?field1=value1&field2=value2。 如果要求的查詢字串中有不止一個索引鍵/值組，其順序無關緊要。 

> [!IMPORTANT]
> Azure CDN 的標準和進階產品提供相同的查詢字串快取功能，但使用者介面不同。 本文說明**來自 Microsoft 的標準 Azure CDN**、**來自 Akamai 的標準 Azure CDN**和**來自 Verizon 的標準 Azure CDN** 的介面。 如需了解使用**來自 Verizon 的進階 Azure CDN** 來進行的查詢字串快取，請參閱[使用查詢字串來控制 Azure CDN 快取行為 - 進階層](cdn-query-string-premium.md)。

查詢字串模式有三種：

- **忽略查詢字串**：預設模式。 在此模式中，CDN 存在點 (POP) 節點會在發生第一次要求時，將來自要求者的查詢字串傳遞給原始伺服器並快取資產。 所有後續對該資產提出並由 POP 提供服務的要求則會忽略查詢字串，直到所快取的資產到期為止。

- **略過查詢字串快取**：在此模式中，不會在 CDN POP 節點快取包含查詢字串的要求。 POP 節點會直接從原始伺服器擷取資產，然後隨著每個要求將其傳遞給要求者。

- **快取所有不重複的 URL**：在此模式中，每個要求都有一個唯一的 URL (包含查詢字串)，會被視為具有專屬快取的唯一資產。 例如，系統會將原始伺服器對 example.ashx?q=test1 要求做出的回應快取在 POP 節點，然後針對後續具有相同查詢字串的快取傳回此回應。 針對 example.ashx?q=test2 的要求，系統會將其快取為具有專屬存留時間設定的個別資產。
   
    >[!IMPORTANT] 
    > 當查詢字串包含會隨著每個要求變更的參數 (例如工作階段識別碼或使用者名稱) 時，請勿使用此模式，因為這會造成快取命中率偏低。

## <a name="changing-query-string-caching-settings-for-standard-cdn-profiles"></a>變更標準 CDN 設定檔的查詢字串快取設定
1. 開啟的 CDN 設定檔，然後選取您想要管理的 CDN 端點。
   
   ![CDN 設定檔端點](./media/cdn-query-string/cdn-endpoints.png)
   
2. 在左側窗格的 [設定] 下方，按一下 [快取規則]。
   
    ![CDN [快取規則] 按鈕](./media/cdn-query-string/cdn-caching-rules-btn.png)
   
3. 在 [查詢字串快取行為] 清單中，選取查詢字串模式，然後按一下 [儲存]。
   
   ![CDN 查詢字串快取選項](./media/cdn-query-string/cdn-query-string.png)

> [!IMPORTANT]
> 由於註冊資訊需要一些時間才能傳播至整個 Azure CDN，因此可能無法立即顯示快取字串設定變更：
> - 若為**來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
> - 若為**來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
> - 若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 



