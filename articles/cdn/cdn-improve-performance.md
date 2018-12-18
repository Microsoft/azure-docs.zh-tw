---
title: 在 Azure CDN 中壓縮檔案以改善效能 | Microsoft Docs
description: 了解如何藉由在 Azure CDN 中壓縮檔案來改善檔案傳輸速度並增加頁面載入效能。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: af1cddff-78d8-476b-a9d0-8c2164e4de5d
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/13/2018
ms.author: magattus
ms.openlocfilehash: 2468462170f970cd597dd1296417d5b93a88c2ec
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997264"
---
# <a name="improve-performance-by-compressing-files-in-azure-cdn"></a>在 Azure CDN 中壓縮檔案以改善效能
檔案壓縮是簡單且有效的方法，可提升檔案傳輸速度，並且在檔案從伺服器傳送出去之前先減少其大小，以增加頁面載入效能。 檔案壓縮可降低頻寬成本，並提供回應速度更快的體驗給使用者。

有兩種方式可啟用檔案壓縮︰

- 在原始伺服器上啟用壓縮。 在此情況下，Azure CDN 會傳遞壓縮的檔案，遞交給提出要求的用戶端。
- 直接在 CDN POP 伺服器上啟用壓縮 (動態壓縮)。 在此情況下，CDN 會壓縮檔案並將其提供給終端使用者，即使原始伺服器未壓縮這些檔案也是如此。

> [!IMPORTANT]
> Azure CDN 組態變更會需要一些時間才能傳播至整個網路： 
- 若為**來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
- 若為**來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
- 若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
>
> 如果這是您第一次設定 CDN 端點壓縮，請先等候 1-2 小時再進行疑難排解，確定壓縮設定已傳播至 POP。
> 
> 

## <a name="enabling-compression"></a>啟用壓縮
標準和高階 CDN 層提供相同的壓縮功能，但兩者的使用者介面不同。 如需有關標準和高階 CDN 層之間差異的詳細資訊，請參閱 [Azure CDN 概觀](cdn-overview.md)。

### <a name="standard-cdn-profiles"></a>標準 CDN 設定檔 
> [!NOTE]
> 本節適用於**來自 Microsoft 的標準 Azure CDN**、**來自 Verizon 的標準 Azure CDN** 及**來自 Akamai 的標準 Azure CDN** 設定檔。
> 
> 

1. 從 CDN 設定檔頁面選取您想要管理的 CDN 端點。
   
    ![CDN 設定檔端點](./media/cdn-file-compression/cdn-endpoints.png)
   
    隨即開啟 [CDN 端點] 頁面。
2. 選取 [壓縮]。

    ![CDN 壓縮選取項目](./media/cdn-file-compression/cdn-compress-select-std.png)
   
    壓縮頁面隨即開啟。
3. 選取 [開啟] 以開啟壓縮。
   
    ![CDN 檔案壓縮選項](./media/cdn-file-compression/cdn-compress-standard.png)
4. 請使用預設的 MIME 類型，或是新增或移除 MIME 類型以修改清單。
   
   > [!TIP]
   > 即使可行，我們依然不建議對壓縮格式套用壓縮。 例如，ZIP、MP3、MP4 或 JPG。
   > 
   
   > [!NOTE]
   > 目前不支援在 Microsoft 提供的 Azure CDN 標準中修改 MIME 類型的預設清單。
   > 
 
5. 完成變更之後，請選取 [儲存]。

### <a name="premium-cdn-profiles"></a>進階 CDN 設定檔
> [!NOTE]
> 本節僅適用於**來自 Verizon 的 Azure CDN 進階**設定檔。
> 

1. 從 CDN 設定檔頁面選取 [管理]。
   
    ![CDN 管理選取](./media/cdn-file-compression/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
2. 將滑鼠移至 [HTTP 大型] 索引標籤上，然後將滑鼠移至 [快取設定] 飛出視窗上。 選取 [壓縮]。

    ![CDN 壓縮選取項目](./media/cdn-file-compression/cdn-compress-select.png)
   
    壓縮選項隨即顯示。
   
    ![CDN 檔案壓縮選項](./media/cdn-file-compression/cdn-compress-files.png)
3. 選取 [啟用壓縮] 以啟用壓縮。 在 [檔案類型] 方塊中，輸入您想要壓縮成逗號分隔清單 (無空格) 的 MIME 類型。
   
   > [!TIP]
   > 即使可行，我們依然不建議對壓縮格式套用壓縮。 例如，ZIP、MP3、MP4 或 JPG。
   > 
    
4. 完成變更之後，請選取 [更新]。

## <a name="compression-rules"></a>壓縮規則

### <a name="azure-cdn-standard-from-microsoft-profiles"></a>來自 Microsoft 的標準 Azure CDN

如果是**來自 Microsoft 的標準 Azure CDN** 設定檔，只會壓縮符合資格的檔案。 若要符合壓縮的資格，檔案必須：屬於已[設定壓縮](#enabling-compression)的 MIME 類型。
- 大於 1 KB - 小於 8 MB

這些設定檔支援下列壓縮編碼：
- gzip (GNU zip)
- brotli 
 
如果該要求支援多種壓縮類型，優先使用 brotli 壓縮。

當資產的要求指定 gzip 壓縮且快取中的要求結果遺失時，Azure CDN 會在原始伺服器上對資產執行 gzip 壓縮。 之後會從快取提供壓縮的檔案。

### <a name="azure-cdn-from-verizon-profiles"></a>來自 Verizon 的 Azure CDN 設定檔

若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，只會壓縮合格的檔案。 若要符合壓縮，檔案必須︰
- 超過 128 個位元組
- 小於 1 MB
 
這些設定檔支援下列壓縮編碼：
- gzip (GNU zip)
- DEFLATE
- bzip2
- brotli 
 
如果該要求支援多種壓縮類型，那些壓縮類型的優先順序均高於 brotli 壓縮。

當資產的要求指定 brotli 壓縮 (HTTP 標頭為 `Accept-Encoding: br`) 且快取中的要求結果遺失時，Azure CDN 會直接在 POP 伺服器上對資產執行 brotli 壓縮。 之後會從快取提供壓縮的檔案。

### <a name="azure-cdn-standard-from-akamai-profiles"></a>來自 Akamai 的標準 Azure CDN 設定檔

若為**來自 Akamai 的標準 Azure CDN** 設定檔，所有檔案都符合壓縮資格。 然而，檔案必須為已[設定壓縮](#enabling-compression)的 MIME 類型。

這些設定檔只支援 gzip 壓縮編碼。 當設定檔端點要求 gzip 編碼檔案時，一律都是要求來自原點的檔案，不論用戶端是否提出要求。 

## <a name="compression-behavior-tables"></a>壓縮行為資料表
下列資料表描述每個案例的 Azure CDN 壓縮行為：

### <a name="compression-is-disabled-or-file-is-ineligible-for-compression"></a>已停用壓縮或檔案不適合進行壓縮
| 用戶端要求的格式 (透過 Accept-Encoding 標頭) | 快取的檔案格式 | CDN 對用戶端的回應 | 附註&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
| --- | --- | --- | --- |
| 已壓縮 |已壓縮 |已壓縮 | |
| 已壓縮 |未壓縮 |未壓縮 | |
| 已壓縮 |不快取 |已壓縮或未壓縮 |原點回應可判斷 CDN 是否執行壓縮。 |
| 未壓縮 |已壓縮 |未壓縮 | |
| 未壓縮 |未壓縮 |未壓縮 | |
| 未壓縮 |不快取 |未壓縮 | |

### <a name="compression-is-enabled-and-file-is-eligible-for-compression"></a>已啟用壓縮且檔案適合進行壓縮
| 用戶端要求的格式 (透過 Accept-Encoding 標頭) | 快取的檔案格式 | CDN 對用戶端的回應 | 注意 |
| --- | --- | --- | --- |
| 已壓縮 |已壓縮 |已壓縮 |支援格式之間的 CDN 轉碼。 |
| 已壓縮 |未壓縮 |已壓縮 |CDN 執行壓縮。 |
| 已壓縮 |不快取 |已壓縮 |如果原點傳回未壓縮的檔案，則 CDN 會執行壓縮。 <br/>**來自 Verizon 的 Azure CDN** 會傳遞第一次要求中的未壓縮檔案，然後壓縮及快取檔案以供後續要求之需。 <br/>具有 `Cache-Control: no-cache` 標頭的檔案永遠不會經過壓縮。 |
| 未壓縮 |已壓縮 |未壓縮 |CDN 執行解壓縮。 |
| 未壓縮 |未壓縮 |未壓縮 | |
| 未壓縮 |不快取 |未壓縮 | |

## <a name="media-services-cdn-compression"></a>媒體服務 CDN 壓縮
對於啟用媒體服務 CDN 串流的端點來說，預設會針對下列 MIME 類型啟用壓縮： 
- application/vnd.ms-sstr+xml 
- application/dash+xml
- application/vnd.apple.mpegurl
- application/f4m+xml 

## <a name="see-also"></a>另請參閱
* [CDN 檔案壓縮疑難排解](cdn-troubleshoot-compression.md)    

