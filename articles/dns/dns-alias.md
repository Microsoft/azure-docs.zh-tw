---
title: Azure DNS 別名記錄概觀
description: Microsoft Azure DNS 中對別名記錄的支援概觀。
services: dns
author: vhorne
ms.service: dns
ms.topic: article
ms.date: 9/25/2018
ms.author: victorh
ms.openlocfilehash: 05a6a1700de2b8b334b40d9efd9b8d79e9e7241f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957553"
---
# <a name="azure-dns-alias-records-overview"></a>Azure DNS 別名記錄概觀

Azure DNS 別名記錄是 DNS 記錄集上的限定性條件，可讓您參考 DNS 區域內的其他 Azure 資源。 例如，您可以建立別名記錄集，來參考 Azure 公用 IP 位址，而不參考 A 記錄。 由於您的別名記錄集會動態指向 Azure 公用 IP 位址服務執行個體，因此，別名記錄集可以在 DNS 解析期間順暢地進行自行更新。

Azure DNS 區域中的下列記錄類型支援別名記錄集：A、AAAA 和 CNAME。 

> [!NOTE]
> 只有外部端點類型支援適用於流量管理員 A 或 AAAA 記錄類型的別名記錄。 您必須針對流量管理員中的外部端點，適當地提供 IPv4 或 IPv6 位址 (最好是靜態 IP)。

## <a name="capabilities"></a>功能

- **從 DNS A/AAAA 記錄集指向公用 IP 資源**。 您可以建立 A/AAAA 記錄集，並使其成為別名記錄集來指向公用 IP 資源。
- **從 DNS A/AAAA/CNAME 記錄集指向流量管理員設定檔**。 除了能夠從 DNS CNAME 記錄集指向流量管理員設定檔的 CNAME (例如：contoso.trafficmanager.net)，您現在也可以從 DNS 區域中的 A 或 AAAA 記錄集指向具有外部端點的流量管理員設定檔。
   > [!NOTE]
   > 只有外部端點類型支援適用於流量管理員 A 或 AAAA 記錄類型的別名記錄。 您必須針對流量管理員中的外部端點，適當地提供 IPv4 或 IPv6 位址 (最好是靜態 IP)。
- **指向相同區域中的另一個 DNS 記錄集**。 別名記錄可以參考其他相同類型的記錄集。 例如，DNS CNAME 記錄集可以是另一個相同類型的 CNAME 記錄集所用的別名。 如果想要在行為方面將一些記錄集設定為別名，有些設定為非別名，則這種方式相當有用。

## <a name="scenarios"></a>案例
以下提供別名記錄的一些常見案例：

### <a name="prevent-dangling-dns-records"></a>防止 DNS 記錄懸空
在 Azure DNS 區域中，別名記錄可用來密切追蹤 Azure 資源的生命週期 (例如公用 IP 和流量管理員設定檔)。 關於傳統 DNS 記錄的常見問題之一是「記錄懸空」，特別是使用 A/AAAA 或 CNAME 記錄類型。 

使用傳統的 DNS 區域記錄，如果目標 IP 或 CNAME 不再存在，DNS 區域記錄並不會知道此一事實，而您必須手動更新。 在某些組織中，此手動更新可能不會即時發生，或者可能會因為將角色與相關聯的權限層級分隔開來而造成問題。

例如，有權刪除屬於某個應用程式之 CNAME 或 IP 位址的角色，可能沒有足夠的權限來更新 DNS 記錄以指向那些目標。 因此，可能會在刪除 IP 或 CNAME 以及移除指向它的 DNS 記錄之間產生時間延遲，這可能會導致終端使用者中斷。

別名記錄會移除與此案例相關聯的複雜度，並協助避免這類懸空參考。 當 DNS 記錄被限定為別名記錄以指向公用 IP 或流量管理員設定檔時，而且如果會刪除那些基礎資源，則也會在同一時間移除該 DNS 別名記錄。 這可確保終端使用者永遠不會受到中斷影響。

### <a name="update-dns-zones-automatically-when-application-ips-change"></a>當應用程式 IP 變更時自動更新 DNS 區域

與前述案例類似，如果應用程式移動或重新啟動底層虛擬機器，則會在針對底層公用 IP 資源進行 IP 位址變更時自動更新別名記錄。 如果系統將終端使用者導向至另一個具有舊 IP 位址的應用程式，您就能避免潛在的安全性風險。

### <a name="host-load-balanced-applications-at-the-zone-apex"></a>位於區域頂點的主機負載平衡應用程式

DNS 通訊協定可防止在區域頂點 (例如：contoso.com) 指派 A 或 AAAA 記錄以外的任何項目。 這會為有負載平衡的應用程式位於流量管理員後面的應用程式擁有者凸顯一個問題，因為它無法從區域頂點記錄指向流量管理員設定檔。 因此，應用程式擁有者被迫使用因應措施。 例如，位於應用程式層的重新導向，要從區域頂點重新導向至另一個網域 (從 contoso.com 至 www.contoso.com)。 這在重新導向功能方面代表單一失敗點。

透過別名記錄，此問題已不存在。 應用程式擁有者現在可以將其區域頂點記錄指向具有外部端點的流量管理員設定檔。 使用此功能，應用程式擁有者可以指向相同的流量管理員設定檔，以用於其 DNS 區域內的任何其他網域。 例如，只要已將流量管理員設定檔設定為只具有外部端點，contoso.com 與 www.contoso.com 都會指向相同的流量管理員設定檔。

## <a name="next-steps"></a>後續步驟

若要深入了解別名記錄，請參閱下列文章：

- [教學課程：設定別名記錄來參考 Azure 公用 IP 位址](tutorial-alias-pip.md)
- [教學課程：使用流量管理員來設定別名記錄以支援頂點網域名稱](tutorial-alias-tm.md)
- [DNS 常見問題](https://docs.microsoft.com/azure/dns/dns-faq#alias-records)
