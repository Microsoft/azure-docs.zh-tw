---
title: DNS 區域和記錄概觀 - Azure DNS | Microsoft Docs
description: 將 DNS 區域和記錄，裝載於 Microsoft Azure DNS 中的支援概觀。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
ms.assetid: be4580d7-aa1b-4b6b-89a3-0991c0cda897
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 12/18/2017
ms.author: victorh
ms.openlocfilehash: 7f69d77ac7a6c2a17ef2568f0c7edaef2e1ee3f5
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39174289"
---
# <a name="overview-of-dns-zones-and-records"></a>DNS 區域和記錄的概觀

此頁面說明的重要概念，包括網域、DNS 區域、DNS 記錄和記錄集，以及在 Azure DNS 中所受到的支援。

## <a name="domain-names"></a>網域名稱

網域名稱系統是網域階層。 階層從「根」網域開始，其名稱只是 '**.**'。  下面接著最上層網域，例如 'com'、'net'、'org'、'uk' 或 'jp'。  再往下是第二層網域，例如 'org.uk' 或 'co.jp'。 DNS 階層中的網域散佈在全球，裝載在世界各地的 DNS 名稱伺服器上。

網域名稱註冊機構是種組織，可讓您購買網域名稱，例如 'contoso.com'。  購買網域名稱可讓您控制該名稱底下的 DNS 階層，例如將網域名稱 'www.contoso.com' 導向貴公司的網站。 可以讓該註冊機構代表您，將網域裝載在其自有的名稱伺服器上；或允許您指定替代的名稱伺服器。

Azure DNS 提供散佈全球、高可用性的名稱伺服器基礎結構，可用來裝載您的網域。 只要將您的網域裝載於 Azure DNS 中，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具、計費方式和支援來管理 DNS 記錄。

Azure DNS 目前不支援購買網域名稱。 若想要購買網域名稱，必須洽詢第三方網域名稱註冊機構。 註冊機構通常會收取些微年費。 然後便可以在 Azure DNS 裝載這些網域來管理 DNS 記錄。 如需詳細資訊，請參閱 [將網域委派給 Azure DNS](dns-domain-delegation.md) 。

## <a name="dns-zones"></a>DNS 區域

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

## <a name="dns-records"></a>DNS 記錄

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

### <a name="time-to-live"></a>存留時間

存留時間 (或 TTL) 會指定用戶端快取每一筆記錄多長時間後，才會重新查詢。 在上述範例中，TTL 是 3600 秒 (1 小時)。

在 Azure DNS 中，TTL 是針對記錄集而指定，而非針對每一筆記錄，因此相同的值，會套用到該記錄集內的所有記錄。  您可以指定介於 1 到 2,147,483,647 秒之間的任何 TTL 值。

### <a name="wildcard-records"></a>萬用字元記錄

Azure DNS 支援 [萬用字元記錄](https://en.wikipedia.org/wiki/Wildcard_DNS_record)。 萬用字元記錄會傳回具有相符名稱的任何查詢 (除非在非萬用字元記錄集內，還有更接近的相符項目)。 Azure DNS 支援 NS 和 SOA 以外的所有記錄類型的萬用字元記錄集。

若要建立萬用字元記錄集，請使用記錄集名稱 '\*'。 或者，也可以使用最左邊標籤是 '\*' 的名稱，例如 '\*.foo'。

### <a name="caa-records"></a>CAA 記錄

CAA 記錄可讓網域擁有者指定哪些憑證授權單位 (CA) 有權為其網域發行憑證。 這可讓 CA 避免在某些情況下誤發憑證。 CAA 記錄都有三個屬性：
* **旗標**：這是介於 0 到 255 之間的整數，用來表示對每個 [RFC](https://tools.ietf.org/html/rfc6844#section-3) 具有特殊意義的重要旗標
* **標記**：ASCII 字串，可以是下列其中一項：
    * **issue**：當您想要指定有權發行憑證 (所有類型) 的 CA 時，使用此選項
    * **issuewild**：當您想要指定有權發行憑證 (僅限 Wildcard 憑證) 的 CA 時，使用此屬性
    * **iodef**：指定當發生未授權的憑證發行要求時，CA 可通知的電子郵件地址或主機名稱
* **值**：所選特定標記的值

### <a name="cname-records"></a>CNAME 記錄

CNAME 記錄集不能與其他具有相同名稱的記錄集共存。 例如，您無法同時建立具有相對名稱 'www' 的 CNAME 記錄集和具有相對名稱 'www' 的 A 記錄。

因為區域頂點 (名稱 = '\@') 一定會包含建立區域時所建立的 NS 和 SOA 記錄集，您無法在區域頂點建立 CNAME 記錄集。

這些條件約束源自於 DNS 標準，而不是 Azure DNS 的限制。

### <a name="ns-records"></a>NS 記錄

區域頂點 (名稱 '\@') 的 NS 記錄集會在每個 DNS 區域自動建立，並在刪除該區域時自動將其刪除 (無法個別刪除)。

此記錄集包含指派給區域的 Azure DNS 名稱伺服器的名稱。 您可以將其他名稱伺服器新增至此 NS 記錄集，以支援使用多個 DNS 提供者的共同裝載網域。 您也可以修改此記錄集的 TTL 和中繼資料。 不過，您無法移除或修改預先填入的 Azure DNS 名稱伺服器。 

這只適用於區域頂點的 NS 記錄集。 區域中的其他 NS 記錄集 (如用於委派子區域) 可以建立、修改和刪除，沒有條件約束。

### <a name="soa-records"></a>SOA 記錄

在每個區域頂點 (名稱 = '\@') 會自動建立 SOA 記錄集，並在刪除該區域時自動將其刪除。  無法個別建立或刪除 SOA 記錄。

可以修改 SOA 記錄除了 'Host' 屬性以外的所有屬性，因為依照預先設定，該屬性會參考 Azure DNS 所提供的主要名稱伺服器名稱。

### <a name="spf-records"></a>SPF 記錄

[!INCLUDE [dns-spf-include](../../includes/dns-spf-include.md)]

### <a name="srv-records"></a>SRV 記錄

各種服務都會使用 [SRV 記錄](https://en.wikipedia.org/wiki/SRV_record)來指定伺服器位置。 在 Azure DNS 中指定 SRV 記錄時︰

* 必須在記錄集名稱中包括 [服務] 和 [通訊協定]，並在其前方加上底線。  例如，'\_sip.\_tcp.name'。  在區域頂點的記錄，則不需要在記錄名稱中指定 '\@'，只要使用服務與通訊協定即可，例如 '\_sip.\_tcp'。
* 系統已將 [優先順序]、[權數]、[連接埠] 和 [目標]，指定為記錄集內每筆記錄的參數。

### <a name="txt-records"></a>TXT 記錄

TXT 記錄是用來將網域名稱對應到任意文字字串。 用於多個應用程式，特別是與電子郵件設定相關的應用程式，例如[寄件者原則架構 (SPF)](https://en.wikipedia.org/wiki/Sender_Policy_Framework)和 [DomainKeys Identified Mail (DKIM)](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)。

DNS 標準允許單一 TXT 記錄包含多個字串，每個長度最多可達 254 個字元。 使用多個字串時，用戶端會將這些字串連接並視為單一字串。

在呼叫 Azure DNS REST API 時，您必須分別指定每個 TXT 字串。  當使用 Azure 入口網站、PowerShell 或 CLI 介面時，您應該為每筆記錄指定單一字串，如有需要，字串將自動分段成 254 個字元的區段。

DNS 記錄中的多個字串，不應與 TXT 記錄集中的多個 TXT 記錄相混淆。  TXT 記錄集可以包含多個記錄，「每一個」記錄可以包含多個字串。  Azure DNS 支援每個 TXT 記錄集中的 字串總長度上限為 1024 個字元 (所有記錄的總和)。

## <a name="tags-and-metadata"></a>標記和中繼資料

### <a name="tags"></a>標記

標記是名稱-值組的清單，由 Azure Resource Manager 用來標示資源。  Azure Resource Manager 會使用標記來啟用 Azure 帳單篩選過的檢視，也可讓您設定標記需要的原則。 如需標記的詳細資訊，請參閱 [使用標記來組織您的 Azure 資源](../azure-resource-manager/resource-group-using-tags.md)。

Azure DNS 支援在 DNS 區域資源上，使用 Azure Resource Manager 標記。  不支援在 DNS 記錄集上使用標記，不過，支援在 DNS 記錄集上使用「中繼資料」作為替代，如下所述。

### <a name="metadata"></a>中繼資料

Azure DNS 支援使用 '中繼資料' 來替代記錄集標記，用以標註記錄集。  類似於標記，中繼資料可讓您建立名稱-值組與每個記錄集之間的關聯。  這非常有用，例如，可用以記錄每個記錄集的目的。  與標記不同的是，中繼資料無法用來提供 Azure 帳單篩選過的檢視，也無法在 Azure Resource Manager 原則中指定。

## <a name="etags"></a>Etag

假設有兩個人或兩個處理序同時嘗試修改 DNS 記錄。 何者獲勝？ 獲勝者知道他已覆寫另一人所做的變更嗎？

Azure DNS 使用 Etag 以安全地處理相同資源的並行變更。 Etag 和 [Azure Resource Manager「標籤」](#tags)分開。 每個 DNS 資源 (區域或記錄集) 都有一個相關聯的 Etag。 每當擷取資源時，也會擷取其 Etag。 更新資源時，您可以選擇傳回 Etag，讓 Azure DNS 可以確認伺服器上的 Etag 相符。 因為每次更新資源都會重新產生 Etag，Etag 不符就表示發生並行變更。 建立新的資源時也可以使用 Etag，以確保該資源尚不存在。

根據預設，Azure DNS PowerShell 會使用 Etag 來禁止對區域和記錄集進行並行變更。 選擇性的 *-Overwrite* 參數可以用來停用 Etag 檢查，在此情況下，會覆寫任何已發生的並行變更。

在 Azure DNS REST API 層級上是使用 HTTP 標頭指定 Etag。  下表提供它們的行為：

| 頁首 | 行為 |
| --- | --- |
| None |PUT 一定成功 (沒有 Etag 檢查) |
| If-match <etag> |唯有當資源存在且 Etag 符合時，PUT 才會成功 |
| If-match * |唯有當資源存在時，PUT 才會成功 |
| If-none-match * |唯有當資源不存在時，PUT 才會成功 |


## <a name="limits"></a>限制

使用 Azure DNS 時，會適用下列的預設限制︰

[!INCLUDE [dns-limits](../../includes/dns-limits.md)]

## <a name="next-steps"></a>後續步驟

* 若要開始使用 Azure DNS，請學習如何[建立 DNS 區域](dns-getstarted-create-dnszone-portal.md)和[建立 DNS 記錄](dns-getstarted-create-recordset-portal.md)。
* 若要移轉現有的 DNS 區域，請學習如何[匯入和匯出 DNS 區域檔案](dns-import-export.md)。
