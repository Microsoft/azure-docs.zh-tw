---
title: 在 Azure CDN 端點上預先載入資產 | Microsoft Docs
description: 了解如何預先載入 Azure CDN 端點上的快取內容。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 5ea3eba5-1335-413e-9af3-3918ce608a83
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/12/2018
ms.author: magattus
ms.openlocfilehash: d2dc8ad1e4b7e429dc758a96e49aa4825ae108e5
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091312"
---
# <a name="pre-load-assets-on-an-azure-cdn-endpoint"></a>在 Azure CDN 端點上預先載入資產
[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

根據預設，只有在要求資產時，才會快取資產。 由於 Edge Server 尚未快取內容且必須將要求轉送給原始伺服器，因此來自每個區域的第一個要求所花費的時間會比後續的要求長。 若要避免這個第一個命中延遲，請預先載入您的資產。 除了提供較佳的客戶體驗之外，預先載入已快取的資產也可減少原始伺服器上的網路流量。

> [!NOTE]
> 對於大型事件或同時提供給許多使用者的內容 (例如新影片發行或軟體更新) 來說，將資產預先載入很有用。
> 
> 

本教學課程將逐步引導您在 Azure CDN 邊緣節點上預先載入快取的所有內容。

## <a name="to-pre-load-assets"></a>預先載入資產
1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至包含您希望預先載入之端點的 CDN 設定檔。 設定檔窗格會隨即開啟。
    
2. 按一下清單中的端點。 端點窗格會隨即開啟。
3. 從 CDN 端點窗格中，選取 [載入]。
   
    ![CDN 端點窗格](./media/cdn-preload-endpoint/cdn-endpoint-blade.png)
   
    [載入] 窗格會隨即開啟。
   
    ![CDN 載入窗格](./media/cdn-preload-endpoint/cdn-load-blade.png)
4. 針對 [內容路徑]，輸入每個您希望載入之資產的完整路徑 (例如 `/pictures/kitten.png`)。
   
   > [!TIP]
   > 在您開始輸入文字之後，會出現更多 [內容路徑] 文字方塊，可讓您建置多個資產的清單。 若要將資產從清單中刪除，請選取省略符號 (...) 按鈕，然後選取 [刪除]。
   > 
   > 每個內容路徑都必須是符合下列[規則運算式](https://msdn.microsoft.com/library/az24scfc.aspx)的相對 URL：  
   > - 載入單一檔案路徑：`^(?:\/[a-zA-Z0-9-_.%=\u0020]+)+$`  
   > - 使用查詢字串來載入單一檔案：`^(?:\?[-_a-zA-Z0-9\/%:;=!,.\+'&\u0020]*)?$` 
   > 
   > 由於每個資產都必須有自己的路徑，因此預先載入資產沒有萬用字元功能。
   > 
   > 
   
    ![載入按鈕](./media/cdn-preload-endpoint/cdn-load-paths.png)
5. 輸入完內容路徑時，選取 [載入]。
   

> [!NOTE]
> 每個 CDN 設定檔的限制為每分鐘 10 個載入要求，以及一次可以處理 50 個並行路徑。 每個路徑都有 1024 個字元的路徑長度限制。
> 
> 

## <a name="see-also"></a>另請參閱
* [清除 Azure CDN 端點](cdn-purge-endpoint.md)
* [Azure CDN REST API 參考 - 預先載入端點上的內容](https://docs.microsoft.com/rest/api/cdn/endpoints/loadcontent)
* [Azure CDN REST API 參考 - 從端點清除內容](https://docs.microsoft.com/rest/api/cdn/endpoints/purgecontent)

