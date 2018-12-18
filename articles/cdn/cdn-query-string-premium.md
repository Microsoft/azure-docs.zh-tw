---
title: 使用查詢字串來控制 Azure CDN 快取行為 - 進階層 | Microsoft Docs
description: Azure CDN 查詢字串快取可控制當 Web 要求包含查詢字串時，要如何快取檔案。 本文說明「來自 Verizon 的進階 Azure CDN」產品中的查詢字串快取。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 99db4a85-4f5f-431f-ac3a-69e05518c997
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/11/2018
ms.author: magattus
ms.openlocfilehash: 2f0a361d53489e22ccc8e41406e5b86b423ea2f6
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091397"
---
# <a name="control-azure-cdn-caching-behavior-with-query-strings---premium-tier"></a>使用查詢字串來控制 Azure CDN 快取行為 - 進階層
> [!div class="op_single_selector"]
> * [標準層](cdn-query-string.md)
> * [進階層](cdn-query-string-premium.md)
> 

## <a name="overview"></a>概觀
使用 Azure 內容傳遞網路 (CDN)，可以控制 Web 要求內含查詢字串時的檔案快取方式。 在包含查詢字串的 Web 要求中，查詢字串是要求中問號 (?) 之後的部分。 查詢字串可以包含一或多個索引鍵/值組，其中的欄位名稱與其值是以等號 (=) 分隔。 每個索引鍵/值組是以 & 符號分隔。 例如 http:\//www.contoso.com/content.mov?field1=value1&field2=value2。 如果要求的查詢字串中有不止一個索引鍵/值組，其順序無關緊要。 

> [!IMPORTANT]
> 「標準」與「進階」CDN 產品提供相同的查詢字串快取功能，但兩者的使用者介面不同。 本文描述**來自 Verizon 的進階 Azure CDN** 介面。 如需了解使用 Azure CDN 標準產品進行的查詢字串快取，請參閱[使用查詢字串控制 Azure CDN 快取行為 - 標準層](cdn-query-string.md)。
>


查詢字串模式有三種：

- **標準快取**：預設模式。 在此模式中，CDN 存在點 (POP) 節點會在發生第一次要求時，將來自要求者的查詢字串傳遞給原始伺服器並快取資產。 所有後續對該資產提出並由 POP 伺服器提供服務的要求則會忽略查詢字串，直到所快取的資產到期為止。

    >[!IMPORTANT] 
    > 如果已針對此帳戶上的任何路徑啟用權杖授權，則標準快取模式就會是唯一可使用的模式。 

- **無快取**：在此模式中，系統不會在 CDN POP 節點快取包含查詢字串的要求。 POP 節點會直接從原始伺服器擷取資產，然後隨著每個要求將其傳遞給要求者。

- **唯一快取**：在此模式中，每個要求都有一個唯一 URL (包含查詢字串)，會被視為具有專屬快取的唯一資產。 例如，系統會將原始伺服器對 example.ashx?q=test1 要求做出的回應快取在 POP 節點，然後針對後續具有相同查詢字串的快取傳回此回應。 針對 example.ashx?q=test2 的要求，系統會將其快取為具有專屬存留時間設定的個別資產。
   
    >[!IMPORTANT] 
    > 當查詢字串包含會隨著每個要求變更的參數 (例如工作階段識別碼或使用者名稱) 時，請勿使用此模式，因為這會造成快取命中率偏低。

## <a name="changing-query-string-caching-settings-for-premium-cdn-profiles"></a>變更進階 CDN 設定檔的查詢字串快取設定
1. 開啟 CDN 設定檔，然後按一下 [管理]。
   
    ![CDN 設定檔 [管理] 按鈕](./media/cdn-query-string-premium/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
2. 將滑鼠移至 [HTTP 大型] 索引標籤上，然後將滑鼠移至 [快取設定] 飛出功能表上。 按一下 [查詢字串快取]。
   
    查詢字串快取選項隨即顯示。
   
    ![CDN 查詢字串快取選項](./media/cdn-query-string-premium/cdn-query-string.png)
3. 選取查詢字串模式，然後按一下 [更新]。

> [!IMPORTANT]
> 因為註冊資訊需要一段時間才能傳遍 CDN，快取字串設定的變更可能無法立即看見。 傳播通常會在 10 分鐘內完成。
 

