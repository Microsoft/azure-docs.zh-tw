---
title: 針對傳回 404 狀態碼的 Azure CDN 端點進行疑難排解 | Microsoft Docs
description: 針對 Azure CDN 端點的 404 回應碼進行疑難排解。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: b588a1eb-ab69-4fc7-ae4d-157c3e46f4a8
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 1cffef5bbda475032ee7ff07188ab0d9d52846ea
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33766099"
---
# <a name="troubleshooting-azure-cdn-endpoints-that-return-a-404-status-code"></a>針對傳回 404 狀態碼的 Azure CDN 端點進行疑難排解
本文可讓您針對傳回 404 HTTP 回應狀態碼的 Azure 內容傳遞網路 (CDN) 端點問題進行疑難排解。

如果在本文章中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您也可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。

## <a name="symptom"></a>徵狀
您已建立 CDN 設定檔和端點，但您的內容似乎無法在 CDN 上使用。 嘗試透過 CDN URL 存取內容的使用者收到 HTTP 404 狀態碼。 

## <a name="cause"></a>原因
有幾個可能的原因，包括︰

* CDN 看不到檔案的來源。
* 端點設定錯誤，導致 CDN 找錯位置。
* 主機正在拒絕來自 CDN 的主機標頭。
* 端點沒有足夠的時間散佈到整個 CDN。

## <a name="troubleshooting-steps"></a>疑難排解步驟
> [!IMPORTANT]
> 建立 CDN 端點之後，無法立即使用，因為註冊需要時間透過 CDN 進行散佈：
> - 若為 **來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
> - 若為**來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
> - 若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 90 分鐘內完成傳播。 
> 
> 如果您完成這份文件中的步驟，而仍然收到 404 回應；請考慮在開啟支援票證之前，等候數小時後再次進行檢查。
> 
> 

### <a name="check-the-origin-file"></a>請檢查來源檔案
首先，確認來源伺服器上有要快取的檔案，而且可在網際網路上公開存取。 要做到這一點，最快的做法是在私人或匿名工作階段中開啟瀏覽器，並直接瀏覽至檔案。 輸入 URL，或將此 URL 貼入網址方塊，並查看是否會產生您所預期的檔案。 例如，假設您在 Azure 儲存體帳戶中有一個檔案，其可在 https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt 存取。 如果您成功載入這個檔案的內容，它便通過測試。

![成功！](./media/cdn-troubleshoot-endpoint/cdn-origin-file.png)

> [!WARNING]
> 雖然這是驗證檔案已公開可用最快且最簡單的方式；在您組織中的某些網路組態讓這個檔案看似公開可用的；但事實上，只有您網路內的使用者才看得見 (即便其已裝載在 Azure 中)。 若要確保情況並非如此，請使用外部瀏覽器測試此檔案，例如並未連線到您組織網路的行動裝置，或 Azure 中的虛擬機器。
> 
> 

### <a name="check-the-origin-settings"></a>請檢查來源設定
在您確認檔案在網際網路上公開可用之後，請驗證您的來源設定。 在 [Azure 入口網站](https://portal.azure.com)，瀏覽至您的 CDN 設定檔，然後選取您要進行疑難排解的端點。 從產生的 [端點] 頁面，選取來源。  

![來源反白的端點頁面](./media/cdn-troubleshoot-endpoint/cdn-endpoint.png)

[來源] 頁面隨即出現。 

![來源頁面](./media/cdn-troubleshoot-endpoint/cdn-origin-settings.png)

#### <a name="origin-type-and-hostname"></a>來源類型和主機名稱
確認 [來源類型] 和 [來源主機名稱] 的值均正確無誤。 在此範例 (https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt) 中，URL 的主機名稱部分是 cdndocdemo.blob.core.windows.net，這是正確的。 因為 Azure 儲存體、Web 應用程式和雲端服務來源會使用 [來源主機名稱] 欄位的下拉式清單值，所以拼字錯誤不成問題。 不過，如果您使用自訂來源，請確定您的主機名稱的拼字正確。

#### <a name="http-and-https-ports"></a>HTTP 和 HTTPS 連接埠
檢查您的 **HTTP** 和 **HTTPS 連接埠**。 在大部分情況下，80 和 443 皆正確，而且您不需要進行任何變更。  不過，如果原始伺服器是透過不同連接埠連接，那麼就必須在這裡表示。 如果您不確定，請檢視來源檔案的 URL。 HTTP 和 HTTPS 規格使用連接埠 80 和 443 作為預設值。 在範例 URL (https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt) 中，未指定連接埠，因此會假設預設值為 443 且設定正確無誤。  

不過，假如您在稍早測試的來源檔案 URL 為 http:\//www.contoso.com:8080/file.txt。 請注意，主機名稱區段結尾的 :8080。 該數字會指示瀏覽器使用連接埠 8080 來連線到位於 www.contoso.com 的 Web 伺服器，因此您必須在 [HTTP 連接埠] 欄位中輸入 8080。 請務必注意，這些連接埠設定只會影響端點用來從來源擷取資訊的連接埠。

> [!NOTE]
> **來自 Akamai 的標準 Azure CDN** 端點不允許來源的完整 TCP 連接埠範圍。  如需不允許的原始連接埠清單，請參閱 [來自 Akamai 的 Azure CDN 允許的原始連接埠](https://msdn.microsoft.com/library/mt757337.aspx)。  
> 
> 

### <a name="check-the-endpoint-settings"></a>請檢查端點設定
在 [端點] 頁面上，選取 [設定] 按鈕。

![以反白顯示設定按鈕的端點頁面](./media/cdn-troubleshoot-endpoint/cdn-endpoint-configure-button.png)

CDN 端點 [設定] 頁面隨即出現。

![[設定] 頁面](./media/cdn-troubleshoot-endpoint/cdn-configure.png)

#### <a name="protocols"></a>通訊協定
針對 **通訊協定**，請確認已選取用戶端所使用的通訊協定。 因為用戶端使用的通訊協定與用來存取來源的通訊協定相同，因此正確設定來源連接埠 (如上一節所示) 相當重要。 不論來源連接埠為何，只會透過預設的 HTTP 和 HTTPS 連接埠 (80 和 443) 接聽 CDN 端點。

讓我們回到包含 http:\//www.contoso.com:8080/file.txt 的假設範例。  您應該還記得之前 Contoso 指定 8080 作為其 HTTP 連接埠，但我們也假設其指定 44300 作為 HTTPS 連接埠。  如果他們建立名為 contoso 的端點，其 CDN 端點的主機名稱會是 contoso.azureedge.net。  http:\//contoso.azureedge.net/file.txt 的要求為 HTTP 要求，因此端點會使用連接埠 8080 上的 HTTP 從來源擷取該要求。  透過 HTTPS 的安全要求 (https:\//contoso.azureedge.net/file.txt) 會導致當從來源擷取檔案時，端點會使用在連接埠 44300 上的 HTTPS。

#### <a name="origin-host-header"></a>原始主機標頭
**原始主機標頭** 為隨著每個要求傳送至來源的主機標頭值。  在大部分情況下，這應該和我們稍早驗證的是相同的 **原始主機名稱** 。  此欄位中不正確的值通常不會造成 404 狀態；但根據來源預期的結果，可能會導致其他 4xx 狀態。

#### <a name="origin-path"></a>原始路徑
最後，我們應該確認 **原始路徑**。  依預設這會是空白。  如果您想要縮小在 CDN 上使用原始裝載資源的範圍，應該只使用此欄位。  

在範例端點中，我們希望能夠使用儲存體帳戶上的所有資源，因此將 [來源路徑] 保留空白。  這表示，對 https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt 的要求會導致從端點連線到要求 /publicblob/lorem.txt 的 cdndocdemo.core.windows.net。  同樣地，對 https:\//cdndocdemo.azureedge.net/donotcache/status.png 的要求會導致端點從原點要求 /donotcache/status.png。

但如果您不想在您的來源上的每個路徑使用 CDN 呢？  假設您只想要公開 publicblob 路徑。  如果我們在 [來源路徑] 欄位中輸入 /publicblob，將導致端點在對來源提出每個要求之前，都要插入 /publicblob。  這表示，對 https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt 的要求，現在實際上會取得 URL 的要求部分 /publicblob/lorem.txt，並將 /publicblob 附加至開頭。 這會導致從來源對 /publicblob/publicblob/lorem.txt 進行要求。  如果該路徑未解析為實際檔案，來源會傳回 404 狀態。  實際上，在此範例中擷取 lorem.txt 的正確 URL 會是 https:\//cdndocdemo.azureedge.net/lorem.txt。  請注意，我們完全不會納入/publicblob 路徑，因為 URL 的要求部分是 /lorem.txt，且端點會新增 /publicblob，進而導致 /publicblob/lorem.txt 成為傳遞至來源的要求。

