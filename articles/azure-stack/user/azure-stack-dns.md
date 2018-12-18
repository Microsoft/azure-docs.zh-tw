---
title: Azure Stack 中的 DNS | Microsoft Docs
description: 在 Azure Stack 中使用 DNS
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.openlocfilehash: acb8b262256031ae8615180e0f55c98cb56b538d
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41946545"
---
# <a name="using-dns-in-azure-stack"></a>在 Azure Stack 中使用 DNS

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 支援下列網域名稱系統 (DNS) 功能：

* DNS 主機名稱解析
* 使用 API 建立和管理 DNS 區域和記錄

## <a name="support-for-dns-hostname-resolution"></a>支援 DNS 主機名稱解析

您可以指定公用 IP 資源的 DNS 網域名稱標籤。 Azure Stack 使用 *domainnamelabel.location*.cloudapp.azurestack.external 作為標籤名稱，並將它對應至 Azure Stack 受控 DNS 伺服器中的公用 IP 位址。

例如，如果您建立公用 IP 資源並以 **contoso** 作為本機 Azure Stack 位置的網域名稱標籤，則[完整網域名稱](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (FQDN) **contoso.local.cloudapp.azurestack.external** 會解析為資源的公用 IP 位址。 您可以使用此 FQDN 來建立自訂網域 CNAME 記錄，其指向 Azure Stack 中的公用 IP 位址。

若要深入了解名稱解析，請參閱 [DNS 解析](https://docs.microsoft.com/azure/dns/dns-for-azure-services?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)一文。

> [!IMPORTANT]
> 您所建立的每個網域名稱標籤在其 Azure Stack 位置中都必須是唯一的。

下一個螢幕擷取畫面顯示 [建立公用 IP 位址] 對話方塊，以便使用入口網站建立公用 IP 位址。

![建立公用 IP 位址](media/azure-stack-whats-new-dns/image01.png)

**範例案例**

您可讓負載平衡器處理來自 Web 應用程式的要求。 負載平衡器的背後是在一或多個虛擬機器上執行的網站。 您可以使用 DNS 名稱 (而不是 IP 位址) 來存取負載平衡的網站。

## <a name="create-and-manage-dns-zones-and-records-using-the-api"></a>使用 API 建立和管理 DNS 區域和記錄

您可以建立和管理 DNS 區域和 Azure Stack 中的記錄。

Azure Stack 提供就像 Azure 的 DNS 服務，使用與 Azure DNS API 一致的 API。  將您的網域裝載於 Azure Stack DNS，您就可以使用相同的認證、API 和工具來管理 DNS 記錄。 您也可以使用與其他 Azure 服務相同的計費方式和支援。

Azure Stack DNS 的基礎結構會比 Azure 更為精簡。 Azure Stack 部署的大小和位置將會影響 DNS 範圍、級別和效能。 這也表示每個部署的效能、可用性、全域發佈和高可用性不盡相同。

## <a name="comparison-with-azure-dns"></a>與 Azure DNS 比較

Azure Stack 中的 DNS 與 Azure 中的 DNS 類似，但有一些您需要了解的主要例外狀況。

* **不支援 AAAA 記錄**

    Azure Stack「不」支援 AAAA 記錄，因為 Azure Stack 並不支援 IPv6 位址。  這是 Azure 和 Azure Stack DNS 之間的主要差異。
* **不是多租用戶**

    Azure Stack 中的 DNS 服務不是多租用戶。 每個租用戶無法建立相同的 DNS 區域。 僅有首個訂用帳戶嘗試建立區域會成功，後續要求皆會失敗。  這是已知問題，也是 Azure 和 Azure Stack DNS 之間的主要差異。 此問題將在未來版本中解決。
* **標記、中繼資料和 Etag**

    Azure Stack 處理標記、中繼資料、Etag 和限制的方式有些許差異。

若要深入了解 Azure DNS，請參閱 [DNS 區域和記錄](../../dns/dns-zones-records.md)。

### <a name="tags-metadata-and-etags"></a>標記、中繼資料和 Etag

**標記**

Azure Stack DNS 支援在 DNS 區域資源上，使用 Azure Resource Manager 標記。 不支援在 DNS 記錄集上使用標記，不過，支援在 DNS 記錄集上使用「中繼資料」作為替代，接下來會詳述。

**Metadata**

Azure Stack DNS 支援使用「中繼資料」來替代記錄集標記，用以標註記錄集。 類似於標記，中繼資料可讓您建立名稱-值組與每個記錄集之間的關聯。 例如，這可在用以記錄每個記錄集目的上非常實用。 與標記不同的是，中繼資料無法用來提供 Azure 帳單篩選過的檢視，也無法在 Azure Resource Manager 原則中指定。

**Etag**

假設有兩個人或兩個處理序同時嘗試修改 DNS 記錄。 何者獲勝？ 獲勝者知道他已覆寫另一人所做的變更嗎？

Azure Stack DNS 使用 Etag 以安全地處理相同資源的並行變更。 Etag 和 Azure Resource Manager「標籤」分開。 每個 DNS 資源 (區域或記錄集) 都有一個相關聯的 Etag。 每當擷取資源時，也會擷取其 Etag。 更新資源時，您可以選擇傳回 Etag，讓 Azure Stack DNS 可以確認伺服器上的 Etag 相符。 因為每次更新資源都會重新產生 Etag，Etag 不符就表示發生並行變更。 建立新的資源時也可以使用 Etag，以確保該資源尚不存在。

根據預設，Azure Stack DNS PowerShell 會使用 Etag 來禁止對區域和記錄集進行並行變更。 選擇性的 *-Overwrite* 參數可以用來停用 Etag 檢查，在此情況下，會覆寫任何已發生的並行變更。

在 Azure Stack DNS REST API 層級上，將會使用 HTTP 標頭指定 Etag。 下表提供它們的行為：

| 頁首 | 行為|
|--------|---------|
| None   | PUT 一定成功 (沒有 Etag 檢查)|
| If-match| 唯有當資源存在且 Etag 符合時，PUT 才會成功|
| If-match *| 唯有當資源存在時，PUT 才會成功|
| If-none-match *| 唯有當資源不存在時，PUT 才會成功|

### <a name="limits"></a>限制

使用 Azure Stack DNS 時，會適用下列的預設限制︰

| 資源| 預設限制|
|---------|--------------|
| 每一訂用帳戶的區域| 100|
| 每一區域的記錄集| 5000|
| 每一記錄集的記錄| 20|

## <a name="next-steps"></a>後續步驟

[適用於 Azure Stack 的 iDNS 簡介](azure-stack-understanding-dns.md)
