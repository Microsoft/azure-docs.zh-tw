---
title: Azure DNS 常見問題集 | Microsoft Docs
description: 關於 Azure DNS 的常見問題集
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/25/2018
ms.author: victorh
ms.openlocfilehash: 66e04e7f0b272f19788e79805ef06d11e2eda572
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948021"
---
# <a name="azure-dns-faq"></a>Azure DNS 常見問題集

## <a name="about-azure-dns"></a>關於 Azure DNS

### <a name="what-is-azure-dns"></a>什麼是 Azure DNS？

網域名稱系統 (DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。 Azure DNS 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構提供名稱解析。 只要將您的網域裝載於 Azure，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。

Azure DNS 中的 DNS 網域裝載於 Azure 的 DNS 名稱伺服器全球網路。 這會使用「任一傳播」網路，所以每個 DNS 查詢是由最接近的可用 DNS 伺服器回答。 Azure DNS 能為您的網域提供快速的效能與高可用性。

Azure DNS 服務是架構在 Azure Resource Manager 上。 因此可得益於 Resource Manager 的功能，如角色型存取控制、稽核記錄檔、資源鎖定。 您可以透過 Azure 入口網站、Azure PowerShell Cmdlet 和跨平台 Azure CLI 來管理網域和記錄。 需要自動化 DNS 管理的應用程式可以透過 REST API 和 SDK 與服務進行整合。

### <a name="how-much-does-azure-dns-cost"></a>Azure DNS 的費用是多少？

Azure DNS 計費模型所根據的是 Azure DNS 中裝載的 DNS 區域數量，以及它們所接收的 DNS 查詢數量。 根據使用量會提供折扣。

如需詳細資訊，請參閱 [Azure DNS 定價頁面](https://azure.microsoft.com/pricing/details/dns/)。

### <a name="what-is-the-sla-for-azure-dns"></a>Azure DNS 有怎樣的 SLA？

Azure 保證有效的 DNS 要求至少有 99.99% 的時間都至少會從一部 Azure DNS 名稱伺服器收到回應。

如需詳細資訊，請參閱 [Azure DNS SLA 頁面](https://azure.microsoft.com/support/legal/sla/dns)。

### <a name="what-is-a-dns-zone-is-it-the-same-as-a-dns-domain"></a>什麼是「DNS 區域」？ 它與 DNS 網域相同嗎？ 

網域是網域名稱系統中的唯一名稱，例如 'contoso.com'。


DNS 區域用來裝載特定網域的 DNS 記錄。 例如，網域 'contoso.com' 可能包含數筆 DNS 記錄，例如 'mail.contoso.com' (用於郵件伺服器) 和 'www.contoso.com' (用於網站)。 這些記錄會裝載在 'contoso.com' DNS 區域中。

網域名稱「只是一個名稱」，而 DNS 區域則是包含網域名稱之 DNS 記錄的資料資源。 Azure DNS 可讓您裝載 DNS 區域，並在 Azure 中管理網域的 DNS 記錄。 它也提供 DNS 名稱伺服器以回應來自網際網路的 DNS 查詢。

### <a name="do-i-need-to-purchase-a-dns-domain-name-to-use-azure-dns"></a>我是否必須先購買 DNS 網域名稱才能使用 Azure DNS？ 

不一定。

您無須購買網域，就能將 DNS 區域裝載在 Azure DNS 中。 您無須擁有網域名稱，即可隨時建立 DNS 區域。 此區域的 DNS 查詢只有在導向到指派給此區域的 Azure DNS 名稱伺服器時，才會進行解析。

如果您想要將您的 DNS 區域連結到全球 DNS 階層，讓來自世界各地的 DNS 查詢都能找到您的 DNS 區域，並以您的 DNS 記錄回應這些查詢，您就必須購買網域名稱。

## <a name="azure-dns-features"></a>Azure DNS 功能

### <a name="are-there-any-restrictions-when-using-alias-records-for-a-domain-name-apex-with-traffic-manager"></a>使用流量管理員針對網域名稱頂點使用別名記錄時，是否有任何限制？

是。 您必須使用靜態公用 IP 位址來搭配流量管理員。 使用靜態 IP 位址設定**外部端點**目標。 

### <a name="does-azure-dns-support-dns-based-traffic-routing-or-endpoint-failover"></a>Azure DNS 是否支援 DNS 型流量路由或端點容錯移轉？

DNS 型流量路由和端點容錯移轉是由「Azure 流量管理員」所提供。 Azure 流量管理員是一項可以與 Azure DNS 一起使用的個別 Azure 服務。 如需詳細資訊，請參閱[流量管理員概觀](../traffic-manager/traffic-manager-overview.md)。

Azure DNS 僅支援裝載「靜態」DNS 網域，其中所指定 DNS 記錄的每個 DNS 查詢一律會收到相同的 DNS 回應。

### <a name="does-azure-dns-support-domain-name-registration"></a>Azure DNS 是否支援網域名稱註冊？

否。 Azure DNS 目前不支援購買網域名稱。 若想要購買網域，必須洽詢第三方網域名稱註冊機構。 註冊機構通常會收取些微年費。 然後便可以在 Azure DNS 裝載這些網域來管理 DNS 記錄。 如需詳細資訊，請參閱 [將網域委派給 Azure DNS](dns-domain-delegation.md) 。

「網域購買」是我們目前正在 Azure 待辦項目上追蹤的功能。 您可以使用意見反應網站來[註冊此功能的支援](https://feedback.azure.com/forums/217313-networking/suggestions/4996615-azure-should-be-its-own-domain-registrar) \(英文\)。

### <a name="does-azure-dns-support-dnssec"></a>Azure DNS 是否支援 DNSSEC？

否。 Azure DNS 目前不支援 DNSSEC。

DNSSEC 是我們目前正在 Azure DNS 待辦項目上追蹤的功能。 您可以使用意見反應網站來[註冊此功能的支援](https://feedback.azure.com/forums/217313-networking/suggestions/13284393-azure-dns-needs-dnssec-support) \(英文\)。

### <a name="does-azure-dns-support-zone-transfers-axfrixfr"></a>Azure DNS 是否支援區域傳輸 (AXFR/IXFR)？

否。 Azure DNS 目前不支援區域傳輸。 您可以[使用 Azure CLI 將 DNS 區域匯入到 Azure DNS 中](dns-import-export.md)。 接著便可以透過 [Azure DNS 管理入口網站](dns-operations-recordsets-portal.md)、我們的 [REST API](https://docs.microsoft.com/powershell/module/azurerm.dns)、[SDK](dns-sdk.md)、[PowerShell Cmdlet](dns-operations-recordsets.md) 或 [CLI 工具](dns-operations-recordsets-cli.md)，來管理 DNS 記錄。

「區域傳輸」是我們目前正在 Azure DNS 待辦項目上追蹤的功能。 您可以使用意見反應網站來[註冊此功能的支援](https://feedback.azure.com/forums/217313-networking/suggestions/12925503-extend-azure-dns-to-support-zone-transfers-so-it-c) \(英文\)。

### <a name="does-azure-dns-support-url-redirects"></a>Azure DNS 是否支援 URL 重新導向？

否。 URL 重新導向服務實際上並不是 DNS 服務，它們是在 HTTP 層級運作，而不是在 DNS 層級。 有些 DNS 提供者會將 URL 重新導向服務結合成其整體供應項目的一部分。 Azure DNS 目前不支援此做法。

我們目前正在 Azure DNS 待辦項目上追蹤「URL 重新導向」功能。 您可以使用意見反應網站來[註冊此功能的支援](https://feedback.azure.com/forums/217313-networking/suggestions/10109736-provide-a-301-permanent-redirect-service-for-ape) \(英文\)。

### <a name="does-azure-dns-support-extended-ascii-encoding-8-bit-set-for-txt-recordset"></a>Azure DNS 是否支援在 TXT 資料錄集中使用延伸的 ASCII 編碼集 (8 位元)？

是。 如果您使用最新版本的 Azure REST API、SDK、PowerShell 與 CLI，Azure DNS 支援在 TXT 資料錄集中使用延伸的 ASCII 編碼集 (2017-10-01 以前的版本或 SDK 2.1 不支援延伸的 ASCII 集)。 例如，如果使用者提供字串作為 TXT 記錄的值，而此字串具有延伸的 ASCII 字元 \128 (例如："abcd\128efgh")，則 Azure DNS 在內部表示中會使用這個字元的位元組值 (即 128)。 而且在 DNS 解析時，會在回應中傳回此位元組的值。 也請注意，就解析而言，"abc" 和 "\097\098\099" 是可互換的。 

針對 TXT 記錄，我們遵循 [RFC 1035](https://www.ietf.org/rfc/rfc1035.txt) 中的區域檔案主要格式逸出規則。 例如，根據 RFC，'\' 現在實際上會逸出所有東西。 如果您指定 "A\B" 作為 TXT 記錄的值，它就只會表示及解析為 "AB"。 如果您真的想讓 TXT 記錄解析成 "A\B"，則需要將 "\" 再次逸出，也就是 指定為 "A\\B"。 

此支援目前不適用於從 Azure 入口網站所建立的 TXT 記錄。 

## <a name="alias-records"></a>別名記錄

### <a name="what-are-some-scenarios-where-alias-records-are-useful"></a>別名記錄可在哪些案例中派上用場？
請參閱 [Azure DNS 別名記錄概觀](dns-alias.md)中的案例一節。

### <a name="what-record-types-are-supported-for-alias-record-sets"></a>別名記錄集支援哪些記錄類型？
Azure DNS 區域中的下列記錄類型支援別名記錄集：A、AAAA 與 CNAME。 

### <a name="what-resources-are-supported-as-targets-for-alias-record-sets"></a>哪些資源可支援作為別名記錄集的目標？
- **從 DNS A/AAAA 記錄集指向公用 IP 資源**。 您可以建立 A/AAAA 記錄集，並使其成為別名記錄集來指向公用 IP 資源。
- **從 DNS A/AAAA/CNAME 記錄集指向流量管理員設定檔**。 除了能夠從 DNS CNAME 記錄集指向流量管理員設定檔的 CNAME (例如：contoso.trafficmanager.net) 之外，您現在也可以從 DNS 區域中的 A 或 AAAA 記錄集指向具有外部端點的流量管理員設定檔。
- **指向相同區域中的另一個 DNS 記錄集**。 別名記錄可以參考其他相同類型的記錄集。 例如，DNS CNAME 記錄集可以是相同類型的另一個 CNAME 記錄集所用的別名。 如果想要在行為方面將一些記錄集設為別名，有些設為非別名，這種方式相當有用。

### <a name="can-i-create-and-update-alias-records-from-the-azure-portal"></a>我是否可從 Azure 入口網站建立及更新別名記錄？
是。 除了使用 Azure REST API、Azure PowerShell、CLI 與 SDK 之外，您也可以在 Azure 入口網站中建立或管理別名。

### <a name="will-alias-records-help-ensure-my-dns-record-set-is-deleted-when-the-underlying-public-ip-is-deleted"></a>當刪除底層公用 IP 時，別名記錄是否會協助確保我的 DNS 記錄集也會刪除？
是。 事實上，這是別名記錄的核心功能之一。 它們會協助您避免應用程式使用者發生可能的中斷。

### <a name="will-alias-records-help-ensure-my-dns-record-set-is-updated-to-the-correct-ip-address-when-the-underlying-public-ip-address-changes"></a>當底層公用 IP 位址變更時，別名記錄是否會協助確保我的 DNS 記錄集更新為正確的 IP 位址？
是。 如同上一個問題，這是別名記錄的核心功能之一，可協助您避免應用程式可能發生的中斷或安全性風險。

### <a name="are-there-any-restrictions-when-using-alias-record-sets-for-an-a-or-aaaa-records-to-point-to-traffic-manager"></a>針對 A 或 AAAA 記錄指向流量管理員使用別名記錄集時是否有任何限制？
是。 如果您想要指向流量管理員設定檔，作為來自 A 或 AAAA 記錄集的別名，您必須確定流量管理員設定檔只使用外部端點。 當您在流量管理員中建立外部端點時，請務必提供端點的實際 IP 位址。

### <a name="is-there-an-additional-charge-for-using-alias-records"></a>使用別名記錄需要額外付費嗎？
別名記錄是 DNS 記錄集上的限定性條件，使用別名記錄無需額外付費。

## <a name="using-azure-dns"></a>使用 Azure DNS

### <a name="can-i-co-host-a-domain-using-azure-dns-and-another-dns-provider"></a>我是否可以使用 Azure DNS 和另一個 DNS 提供者來共同裝載某個網域？

是。 Azure DNS 支援與其他 DNS 服務共同裝載網域。

若要這麼做，您必須將網域的 NS 記錄 (這可控制哪些提供者會收到該網域的 DNS 查詢) 修改成指向該兩個提供者的名稱伺服器。 您可以在三個地方修改這些 NS 記錄：在 Azure DNS 中、在另一個提供者中，以及在父區域中 (通常是透過網域名稱註冊機構設定)。 如需有關 DNS 委派的詳細資訊，請參閱 [DNS 網域委派](dns-domain-delegation.md)。

此外，您必須確保網域的 DNS 記錄在這兩個 DNS 提供者之間保持同步。 Azure DNS 目前不支援 DNS 區域傳輸。 您必須使用 [Azure DNS 管理入口網站](dns-operations-recordsets-portal.md)、[REST API](https://docs.microsoft.com/powershell/module/azurerm.dns) \(英文\)、[SDK](dns-sdk.md)、[PowerShell Cmdlet](dns-operations-recordsets.md) 或 [CLI 工具](dns-operations-recordsets-cli.md)，來同步處理 DNS 記錄。

### <a name="do-i-have-to-delegate-my-domain-to-all-four-azure-dns-name-servers"></a>我是否必須將我的網域委派給全部四個 Azure DNS 名稱伺服器？

是。 Azure DNS 會為每個 DNS 區域指派四個名稱伺服器，以隔離錯誤和提升恢復能力。 為了符合 Azure DNS SLA 的要求，您必須將您的網域委派給這所有四個名稱伺服器。

### <a name="what-are-the-usage-limits-for-azure-dns"></a>Azure DNS 在使用量上有什麼限制？

使用 Azure DNS 時，會適用下列的預設限制︰

[!INCLUDE [dns-limits](../../includes/dns-limits.md)]

### <a name="can-i-move-an-azure-dns-zone-between-resource-groups-or-between-subscriptions"></a>我是否可以在資源群組之間或訂用帳戶之間移動 Azure DNS 區域？

是。 您可以在資源群組之間或訂用帳戶之間移動 DNS 區域。

移動 DNS 區域時，對 DNS 查詢並沒有影響。 指派給區域的名稱伺服器會維持不變，系統會將 DNS 查詢當作一般輸送量來處理。

如需有關如何移動 DNS 區域的詳細資訊和指示，請參閱[將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。

### <a name="how-long-does-it-take-for-dns-changes-to-take-effect"></a>DNS 變更需要多長的時間才會生效？

新的 DNS 區域和 DNS 記錄通常幾秒內就會在 Azure DNS 名稱伺服器上快速反映出來。

對現有 DNS 記錄的變更可能需要較長的時間，但仍應該會在 60 秒內於 Azure DNS 名稱伺服器上反映出來。 在此情況下，Azure DNS 外的 DNS 快取 (由 DNS 用戶端和 DNS 遞迴解析程式所執行) 也可能影響讓 DNS 變更生效所需的時間。 您可以使用每一個記錄集的「存留時間」(TTL) 屬性來控制這個快取持續時間。

### <a name="how-can-i-protect-my-dns-zones-against-accidental-deletion"></a>我要如何保護我的 DNS 區域免於遭到意外刪除？

Azure DNS 的管理是透過 Azure Resource Manager，因此享有 Azure Resource Manager 所提供的存取控制功能優點。 角色型存取控制可用來控制哪些使用者具備 DNS 區域和記錄集的讀取或寫入存取權。 您可以套用資源鎖定來防止意外修改或刪除 DNS 區域和記錄集。

如需詳細資訊，請參閱[保護 DNS 區域和記錄](dns-protect-zones-recordsets.md)。

### <a name="how-do-i-set-up-spf-records-in-azure-dns"></a>我要如何設定 Azure DNS 中的 SPF 記錄？

[!INCLUDE [dns-spf-include](../../includes/dns-spf-include.md)]

### <a name="do-azure-dns-nameservers-resolve-over-ipv6"></a>Azure DNS Nameserver 是否需透過 IPv6 解析？ 

是。 Azure DNS Nameservers 是雙重堆疊 (具有 IPv4 和 IPv6 位址)。 若要尋找指派至您 DNS 區域的 Azure DNS nameservers IPv6 位址，您可以使用 nslookup 之類的工具 (例如 `nslookup -q=aaaa <Azure DNS Nameserver>`)。

### <a name="how-do-i-set-up-an-international-domain-name-idn-in-azure-dns"></a>我要如何設定 Azure DNS 中的「國際網域名稱」(IDN)？

「國際網域名稱」(IDN) 的運作方式是使用 '[Punycode](https://en.wikipedia.org/wiki/Punycode)' 將每個 DNS 名稱編碼。 建立 DNS 查詢時，會使用這些以 Punycode 編碼的名稱來建立。

您可以透過先將區域名稱或記錄集名稱轉換成 Punycode，來設定 Azure DNS 中的「國際網域名稱」(IDN)。 Azure DNS 目前不支援以 Punycode 作為轉換目標或來源的內建轉換。

## <a name="private-dns"></a>私人 DNS

[!INCLUDE [private-dns-public-preview-notice](../../includes/private-dns-public-preview-notice.md)]

### <a name="does-azure-dns-support-private-domains"></a>Azure DNS 是否支援「私人」網域？
「私人」網域的支援是使用私人區域功能進行實作。  此功能目前以公開預覽的形式提供。  私人區域是使用與網際網路面向 Azure DNS 區域相同的工具使用進行管理，但只能從您指定的虛擬網路內將它們進行解析。  如需詳細資訊，請參閱[概觀](private-dns-overview.md)。

目前 Azure 入口網站上並不支援私人區域。 

如需有關 Azure 中其他內部 DNS 選項的資訊，請參閱 [VM 與角色執行個體的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

### <a name="what-is-the-difference-between-registration-virtual-network-and-resolution-virtual-network-in-the-context-of-private-zones"></a>在私人區域的內容下，註冊虛擬網路和解析虛擬網路之間的差異為何？ 
您可以兩種方式將虛擬網路連結至 DNS 私人區域：作為註冊虛擬網路，或是作為解析虛擬網路。 不管是用哪一種方式，虛擬網路中的虛擬機器都能夠針對私人區域中的記錄成功地進行解析。 不過，如果您將虛擬網路指定為註冊虛擬網路，Azure 將會針對虛擬網路中的虛擬機器，將 DNS 記錄自動註冊 (動態註冊) 至區域。 此外，當註冊虛擬網路中的虛擬機器被刪除時，Azure 也會自動從連結的私人區域中移除對應的 DNS 記錄。 

### <a name="will-azure-dns-private-zones-work-across-azure-regions"></a>Azure DNS 私人區域是否能跨 Azure 區域運作？
是。 私人區域支援在跨 Azure 區域的虛擬網路之間進行 DNS 解析 (就算是沒有明確對虛擬網路進行對等也一樣)，前提是所有虛擬網路都必須針對私人區域指定為解析虛擬網路。 客戶可能需要對虛擬網路進行對等，來使 TCP/HTTP 流量能夠從某個區域流至另一個區域。 

### <a name="is-connectivity-to-the-internet-from-virtual-networks-required-for-private-zones"></a>針對私人區域，虛擬網路是否需要能夠連線至網際網路？
否。 私人區域能與虛擬網路搭配運作，來讓客戶可以在虛擬網路內及虛擬網路之間管理虛擬機器或其他資源的網域。 名稱解析並不需要任何網際網路連線能力。 

### <a name="can-the-same-private-zone-be-used-for-multiple-virtual-networks-for-resolution"></a>相同的私人區域是否可以用於多個虛擬網路以進行解析？ 
是。 客戶可以將最多 10 個解析虛擬網路關聯至單一私人區域。

### <a name="can-a-virtual-network-that-belongs-to-a-different-subscription-be-added-as-a-resolution-virtual-network-to-a-private-zone"></a>是否可以將屬於不同訂用帳戶的虛擬網路，作為解析虛擬網路新增至私人區域？ 
是，前提是使用者在這兩個虛擬網路及私人 DNS 區域上皆擁有寫入作業權限。 寫入權限可能配置給多個 RBAC 角色。 例如，傳統網路參與者 RBAC 角色便具有虛擬網路的寫入權限。 如需 RBAC 角色的詳細資訊，請參閱[角色型存取控制](../role-based-access-control/overview.md)

### <a name="will-the-automatically-registered-virtual-machine-dns-records-in-a-private-zone-be-automatically-deleted-when-the-virtual-machines-are-deleted-by-the-customer"></a>當客戶刪除虛擬機器時，是否也會自動刪除私人區域中之自動註冊的虛擬機器 DNS 記錄？
是。 如果您刪除註冊虛擬網路內的虛擬機器，基於此為註冊虛擬網路的原因，我們將會自動刪除註冊至區域的 DNS 記錄。 

### <a name="can-an-automatically-registered-virtual-machine-record-in-a-private-zone-from-a-registration-virtual-network-be-deleted-manually"></a>(來自註冊虛擬網路的) 私人區域中之自動註冊的虛擬機器記錄是否可以手動刪除？ 
否。 目前，從註冊虛擬網路自動註冊於私人區域中的虛擬機器 DNS 記錄，並無法由客戶查看或編輯。 不過，您可以將這些自動註冊的 DNS 記錄取代 (覆寫) 為於區域中手動建立的 DNS 記錄。 請參閱下列針對此議題的問題和解答。

### <a name="what-happens-when-we-attempt-to-manually-create-a-new-dns-record-into-a-private-zone-that-has-the-same-hostname-as-an-automatically-registered-existing-virtual-machine-in-a-registration-virtual-network"></a>當我嘗試在具有和註冊虛擬網路中 (自動註冊的) 現有虛擬機器相同之主機名稱的私人區域中手動建立新的 DNS 記錄時，會發生什麼事？ 
如果您嘗試在具有和註冊虛擬網路中現有 (自動註冊的) 虛擬機器相同之主機名稱的私人區域中手動建立新的 DNS 記錄，我們將會允許新的 DNS 記錄覆寫自動註冊的虛擬機器記錄。 此外，如果您隨後嘗試從區域刪除此手動建立的 DNS 記錄，只要虛擬機器仍然存在且有附加私人 IP，刪除將會成功，且自動註冊將會再次發生 (DNS 記錄將會在區域中自動重新建立)。 

### <a name="what-happens-when-we-unlink-a-registration-virtual-network-from-a-private-zone-would-the-automatically-registered-virtual-machine-records-from-the-virtual-network-be-removed-from-the-zone-as-well"></a>當我取消註冊虛擬網路和私人區域之間的連結時，會發生什麼事？ 來自虛擬網路的自動註冊虛擬機器記錄是否也會從區域中移除？
是。 如果您解除註冊虛擬網路及私人區域之間的連結 (更新 DNS 區域以移除相關聯的註冊虛擬網路)，Azure 將會從區域移除所有自動註冊的虛擬機器記錄。 

### <a name="what-happens-when-we-delete-a-registration-or-resolution-virtual-network-that-is-linked-to-a-private-zone-do-we-have-to-manually-update-the-private-zone-to-unlink-the-virtual-network-as-a-registration-or-resolution--virtual-network-from-the-zone"></a>當我刪除已連結至私人區域的註冊 (或解析) 虛擬網路時，會發生什麼事？ 我是否需要手動更新私人區域，以取消作為註冊 (或解析) 虛擬網路的虛擬網路與區域之間的連結？
是。 當您在沒有先解除註冊 (或解析) 虛擬網路與私人區域間連結的情況下便刪除該虛擬網路時，Azure 將會允許您進行該刪除作業，但系統將不會自動解除虛擬網路與任何先前已連結之私人區域之間的連結。 您需要手動解除虛擬網路與私人區域之間的連結。 基於這個原因，我們建議在刪除虛擬網路之前，先解除它和您私人區域之間的連結。

### <a name="would-dns-resolution-using-the-default-fqdn-internalcloudappnet-still-work-even-when-a-private-zone-for-example-contosolocal-is-linked-to-a-virtual-network"></a>在私人區域 (例如 contoso.local) 已連結至虛擬網路的情況下，使用預設 FQDN (internal.cloudapp.net) 的 DNS 解析是否能持續運作？ 
是。 私人區域功能並不會取代使用由 Azure 所提供之 internal.cloudapp.net 區域的預設 DNS 解析，而是作為額外的性能或增強功能提供使用。 針對這兩種情況 (無論是仰賴由 Azure 所提供的 internal.cloudapp.net 或您自己的私人區域)，我們建議使用您想要解析之目標區域的 FQDN。 

### <a name="would-the-dns-suffix-on-virtual-machines-within-a-linked-virtual-network-be-changed-to-that-of-the-private-zone"></a>已連結虛擬網路內之虛擬機器上的 DNS 尾碼，是否會變更成私人區域的尾碼？ 
否。 目前，您已連結虛擬網路中虛擬機器上的 DNS 尾碼將會保持為由 Azure 所提供的預設尾碼 ("*.internal.cloudapp.net")。 不過，您可以手動將虛擬機器上的 DNS 尾碼變更為私人區域的尾碼。 

### <a name="are-there-any-limitations-for-private-zones-during-this-preview"></a>此預覽期間的私人區域功能是否有任何限制？
是。 在公開預覽其間，將會存在下列限制：
* 每個私人區域都只能有一個註冊虛擬網路
* 每個私人區域最多可以有 10 個解析虛擬網路
* 特定虛擬網路只能連結到單一私人區域作為註冊虛擬網路
* 特定虛擬網路可以連結到最多 10 個私人區域作為解析虛擬網路
* 如果指定了註冊虛擬網路，在已註冊至私人區域的該虛擬網路中，將無法從 PowerShell/CLI/API 檢視或擷取來自該虛擬網路 VM 的 DNS 記錄，但 VM 記錄實際上已註冊，且將會成功解析
* 反向 DNS 只適用於註冊虛擬網路中的私人 IP 空間
* 未在私人區域中註冊之私人 IP (例如：在作為解析虛擬網路連結至私人區域的虛擬網路中，虛擬機器的私人 IP) 的反向 DNS 將會傳回 "internal.cloudapp.net" 作為 DNS 尾碼，但此尾碼將無法解析。   
* 虛擬網路一開始 (也就是 第一次) 連結到私人區域作為註冊或解析虛擬網路時， 必須是空的 (也就是沒有任何 VM 記錄)。 不過，對於未來要作為註冊或解析虛擬網路連結到其他私人區域的虛擬網路，則可為非空白。 
* 此階段不支援條件式轉送，例如，啟用 Azure 和 OnPrem 網路之間的解析。 如需客戶可如何透過其他機制實現此案例的文件，請參閱 [VM 與角色執行個體的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)

### <a name="are-there-any-quotas-or-limits-on-zones-or-records-for-private-zones"></a>針對私人區域的區域或記錄，是否有任何配額或限制？
針對私人區域，並沒有針對每個訂用帳戶所允許的區域數目，或是每個區域之記錄集數目的個別限制。 公用與私人區域都會計入整體的 DNS 限制，如[這裡](../azure-subscription-service-limits.md#dns-limits)所述

### <a name="is-there-portal-support-for-private-zones"></a>私人區域是否有入口網站支援？
透過非入口網站機制 (API/PowerShell/CLI/SDK) 所建立的私人區域將會顯示於 Azure 入口網站上，但客戶將無法建立新的私人區域或管理虛擬網路之間的關聯性。 此外，針對作為註冊虛擬網路關聯的虛擬網路，自動註冊的 VM 記錄將不會顯示於入口網站上。 

## <a name="next-steps"></a>後續步驟

[深入了解 Azure DNS](dns-overview.md)
<br>
[深入了解將 Azure DNS 用於私人網域](private-dns-overview.md)
<br>
[深入了解 DNS 區域和記錄](dns-zones-records.md)
<br>
[開始使用 Azure DNS](dns-getstarted-portal.md)

