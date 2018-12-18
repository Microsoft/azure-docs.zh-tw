---
title: 疑難排解 - 針對 Azure Front Door Service 組態的問題進行疑難排解 | Microsoft Docs
description: 在本教學課程中，您會了解如何自行排解您可能會面臨的一些 Front Door 常見問題。
services: frontdoor
documentationcenter: ''
author: sharad4u
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/22/2018
ms.author: sharadag
ms.openlocfilehash: 78370b004d18f70ae4d485f3ad7cfd910e6dd70a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045788"
---
# <a name="troubleshooting-common-routing-issues"></a>針對常見路由問題進行疑難排解
本文說明如何排解您可能在 Azure Front Door Service 組態中面臨的一些常見路由問題。 

## <a name="hostname-not-routing-to-backend-and-returns-400-status-code"></a>主機名稱無法路由傳送至後端並傳回 400 狀態碼


### <a name="symptom"></a>徵狀
- 您已建立 Front Door，但前端主機的要求會傳回 HTTP 400 狀態碼。

 - 您已建立從自訂網域到您所設定前端主機的 DNS 對應。 不過，將要求傳送至自訂網域主機名稱會傳回 HTTP 400 狀態碼，而且似乎不會路由傳送至您已設定的後端。

### <a name="cause"></a>原因
- 如果您尚未針對已新增為前端主機的自訂網域設定路由規則，則可能出現此徵狀。 即使已經為 Front Door 子網域 (*.azurefd.net) 下您的自訂網域具有 DNS 對應的前端主機設定路由規則，仍必須為該前端主機明確地新增路由規則。

### <a name="troubleshooting-steps"></a>疑難排解步驟
- 新增從自訂網域到所需後端集區的路由規則。

## <a name="request-to-frontend-hostname-returns-404-status-code"></a>對前端主機名稱的要求會傳回 404 狀態碼

### <a name="symptom"></a>徵狀
- 您已建立 Front Door，並設定了前端主機、至少有一個後端的後端集區，以及可將前端主機連到後端集區的路由規則。 將要求傳送至已設定的前端主機時，您的內容似乎無法使用，因為傳回了 HTTP 404 狀態碼。

### <a name="cause"></a>原因
這個徵狀的可能原因有幾個，包括︰
 - 後端不是公眾對應後端，而且 Front Door 服務看不到它。

- 後端設定錯誤，這導致 Front Door 服務傳送錯誤的要求 (也就是，您的後端只接受 HTTP，但是您還沒取消核取允許 HTTPS，所以 Front Door 會嘗試轉送 HTTPS 要求)。
- 後端會拒絕已透過要求轉送至後端的主機標頭。
- 尚未完整部署後端的組態。

### <a name="troubleshooting-steps"></a>疑難排解步驟
1. 部署時間
    - 確定您已等候 ~10 分鐘讓組態進行部署。

2. 檢查後端設定
     - 瀏覽到要求應路由傳送至的後端集區 (取決於您設定路由規則的方式) ，並確認「後端主機類型」與後端主機名稱都正確。 如果後端是自訂主機，請確保您的拼字正確。 

     - 檢查您的 HTTP 和 HTTPS 連接埠。 在大部分情況下，80 和 443 (分別) 正確無誤，不需要任何變更。 不過，您的後端有可能並未如此設定，而且是在不同的連接埠上接聽。

    - 請檢查針對前端主機應路由傳送至的後端所設定的 [後端主機標頭]。 在大部分情況下，此標頭應該與 [後端主機標頭] 相同。 不過，如果後端預期有不同的值，則不正確的值可能會導致各種 HTTP 4xx 狀態碼。 如果您輸入後端的 IP 位址，則可能必須將 [後端主機標頭] 設定為後端的主機名稱。


3. 檢查路由規則設定
     - 瀏覽到應會從有問題的前端主機名稱路由傳送至後端集區的路由規則。 確定接受的通訊協定設定正確，若非如此，請確定已正確設定 Front Door 在轉送要求時會使用的通訊協定。 [接受的通訊協定] 可決定 Front Door 應該接受的要求，而 [進階] 索引標籤下的 [轉送通訊協定] 可決定 Front Door 應使用哪個通訊協定將要求轉送至後端。
          - 例如，如果後端只接受 HTTP 要求，則下列組態會有效：
               - [接受的通訊協定] 為 HTTP 和 HTTPS。 [轉送通訊協定] 為 HTTP。 比對要求無法運作，因為 HTTPS 是允許的通訊協定，而如果要求以 HTTPS 形式傳入，Front Door 會嘗試使用 HTTPS 轉送它。

               - [接受的通訊協定] 為 HTTP。 [轉送通訊協定] 為比對要求或 HTTPS。

   - 按一下路由規則組態窗格頂端的 [進階] 索引標籤。 預設會停用 [URL 重寫]，而如果您想要縮小您要提供的後端裝載資源範圍，則應該只使用此欄位。 停用時，Front Doo 會轉送它所接收的相同要求路徑。 此欄位可能設定不正確，而 Front Door 向無法使用的後端要求資源，因而傳回 HTTP 404 狀態碼。

