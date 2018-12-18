---
title: 清除 Azure CDN 端點 | Microsoft Docs
description: 了解如何清除 Azure CDN 端點的所有快取內容。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 0b50230b-fe82-4740-90aa-95d4dde8bd4f
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: magattus
ms.openlocfilehash: a3777533fc967e1974b99375496dd3777fa9fb3a
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093845"
---
# <a name="purge-an-azure-cdn-endpoint"></a>清除 Azure CDN 端點
## <a name="overview"></a>概觀
Azure CDN 邊緣節點會快取資產，直到資產的存留時間 (TTL) 到期。  資產的 TTL 到期之後，當用戶端從邊緣節點要求資產時，邊緣節點會建立資產新的更新的複本以服務用戶端的要求並儲存重新整理快取。

若要確定使用者一律會取得最新的資產複本，最佳作法是為每個更新設定資產版本，然後將它們發佈為新的 URL。  CDN 會立即為下一個用戶端要求擷取新的資產。  有時您可能想要清除所有邊緣節點的快取內容，並強制它們全部擷取新的更新的資產。  可能是因為您的 Web 應用程式更新，或快速更新包含不正確資訊的資產。

> [!TIP]
> 請注意，清除只會清除 CDN 邊緣伺服器上的快取內容。  任何下游快取，例如 proxy 伺服器和本機瀏覽器快取，仍然可能含有檔案的快取複本。  當您設定檔案的存留時間時，請務必記住這一點。  您可以強制下游用戶端要求最新版本的檔案，方法是每次更新下游用戶端時提供唯一名稱給它，或是運用 [查詢字串快取](cdn-query-string.md)。  
> 
> 

本教學會逐步引導您清除端點的所有邊緣節點的資產。

## <a name="walkthrough"></a>逐步介紹
1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽到包含您希望清除之端點的 CDN 設定檔。
2. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [清除] 按鈕。
   
    ![CDN 設定檔刀鋒視窗](./media/cdn-purge-endpoint/cdn-profile-blade.png)
   
    會開啟 [清除] 刀鋒視窗。
   
    ![CDN 清除刀鋒視窗](./media/cdn-purge-endpoint/cdn-purge-blade.png)
3. 在 [清除] 刀鋒視窗中，從 [URL] 下拉式清單中選取希望清除的服務位址。
   
    ![清除表單](./media/cdn-purge-endpoint/cdn-purge-form.png)
   
   > [!NOTE]
   > 您也可以按一下 [CDN 端點] 刀鋒視窗中的 [清除] 按鈕，來開啟 [清除] 刀鋒視窗。  在此情況下，[URL]  欄位會預先填入該特定端點的服務位址。
   > 
   > 
4. 選取您希望從邊緣節點清除的資產。  如果您希望清除所有資產，請按一下 [清除]  核取方塊。  或者，在 [路徑] 文字方塊中輸入每個您想要清除之資產的路徑。 路徑支援下列格式。
    1. **單一 URL 清除**︰藉由指定完整 URL (含或不含副檔名) 來清除個別資產，例如 `/pictures/strasbourg.png`、`/pictures/strasbourg`
    2. **萬用字元清除**︰星號 (\*) 可作為萬用字元。 清除路徑中有 `/*` 之端點下的所有資料夾、子資料夾和檔案，或指定後接 `/*` 的資料夾來清除特定資料夾下的所有子資料夾和檔案，例如 `/pictures/*`。  請注意，來自 Akamai 的 Azure CDN 目前不支援萬用字元清除。 
    3. **根網域清除**︰清除路徑中有 "/" 之端點的根目錄。
   
   > [!TIP]
   > 路徑必須加以指定才能清除，且必須是符合下列[規則運算式](https://msdn.microsoft.com/library/az24scfc.aspx)的相對 URL。 **來自 Akamai 的 Azure CDN** 目前不支援**全部清除**和**萬用字元清除**。
   > > 單一 URL 清除 `@"^\/(?>(?:[a-zA-Z0-9-_.%=\(\)\u0020]+\/?)*)$";`  
   > > 查詢字串 `@"^(?:\?[-\@_a-zA-Z0-9\/%:;=!,.\+'&\(\)\u0020]*)?$";`  
   > > 萬用字元清除 `@"^\/(?:[a-zA-Z0-9-_.%=\(\)\u0020]+\/)*\*$";`。 
   > 
   > 在您輸入文字之後會出現更多 [路徑] 文字方塊，讓您能夠建立多個資產的清單。  按一下刪節號 (...) 按鈕可以將資產從清單刪除。
   > 
5. 按一下 [清除]  按鈕。
   
    ![清除按鈕](./media/cdn-purge-endpoint/cdn-purge-button.png)

> [!IMPORTANT]
> 清除要求需要大約 2-3 分鐘來處理**來自 Verizon 的 Azure CDN** (標準和進階)，大約 7 分鐘來處理**來自 Akamai 的 Azure CDN**。  Azure CDN 隨時都有在設定檔層級 50 個並行清除要求的限制。 
> 
> 

## <a name="see-also"></a>另請參閱
* [在 Azure CDN 端點上預先載入資產](cdn-preload-endpoint.md)
* [Azure CDN REST API 參考資料 - 清除或預先載入端點](https://msdn.microsoft.com/library/mt634451.aspx)

