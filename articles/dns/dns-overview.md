---
title: 什麼是 Azure DNS？
description: 在 Microsoft Azure 上裝載 Azure DNS 服務的概觀。 在 Microsoft Azure 上裝載您的網域。
author: vhorne
manager: jeconnoc
ms.service: dns
ms.topic: overview
ms.date: 9/24/2018
ms.author: victorh
ms.openlocfilehash: f68281f71ed525798410fb80b0d0175d38050c40
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48829914"
---
# <a name="what-is-azure-dns"></a>什麼是 Azure DNS？

Azure DNS 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構提供名稱解析。 只要將您的網域裝載於 Azure，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。

您無法使用 Azure DNS 來購買網域名稱。 如果支付年費，則可以使用 [Azure Web Apps](https://docs.microsoft.com/azure/app-service/custom-dns-web-site-buydomains-web-app#buy-the-domain) 或第三方網域名稱註冊機構來購買網域名稱。 然後，便可以在 Azure DNS 中裝載您的網域以管理記錄。 如需詳細資訊，請參閱 [將網域委派給 Azure DNS](dns-domain-delegation.md) 。

Azure DNS 包含下列功能︰

## <a name="reliability-and-performance"></a>可靠性和效能

Azure DNS 中的 DNS 網域裝載於 Azure 的 DNS 名稱伺服器全球網路。 Azure DNS 會使用「任一傳播」網路，所以每個 DNS 查詢是由最接近的可用 DNS 伺服器回答，而為您的網域提供快速的效能和高可用性。

## <a name="security"></a>安全性

Azure DNS 服務以 Azure Resource Manager 為基礎，可提供下功能：

* [角色型存取控制](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#access-control) - 用以控制哪些人能存取組織的特定動作。

* [活動記錄](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#activity-logs) - 用以監視組織中的使用者修改資源的情形，或在進行疑難排解時發現錯誤。

* [資源鎖定](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources) - 用以鎖定訂用帳戶、資源群組或資源，以防止組織中的其他使用者不小心刪除或修改重要資源。

如需詳細資訊，請參閱[如何保護 DNS 區域和記錄](dns-protect-zones-recordsets.md)。 


## <a name="ease-of-use"></a>容易使用

Azure DNS 服務可為您的 Azure 服務管理 DNS 記錄，也可為您的外部資源提供 DNS。 Azure DNS 已在 Azure 入口網站中進行整合，並且使用與您其他 Azure 服務相同的認證、支援合約和計費。 

DNS 會依據 Azure 所裝載的 DNS 區域數量，以及所接收的 DNS 查詢數量計費。 若要深入了解定價，請參閱 [Azure DNS 定價](https://azure.microsoft.com/pricing/details/dns/)。

您可以使用 Azure 入口網站、Azure PowerShell Cmdlet 和跨平台 Azure CLI 來管理網域和記錄。 需要自動化 DNS 管理的應用程式可以使用 REST API 和 SDK 與服務進行整合。

## <a name="customizable-virtual-networks-with-private-domains"></a>使用私人網域的可自訂虛擬網路

Azure DNS 也支援私人 DNS 區域 (目前處於公開預覽狀態)。 這可讓您在私人虛擬網路中使用自己的自訂網域名稱，而不是 Azure 目前提供的可用名稱。

如需詳細資訊，請參閱[使用私人網域的 Azure DNS](private-dns-overview.md)。

## <a name="alias-records"></a>別名記錄

Azure DNS 支援別名記錄集。 您可以使用別名記錄集以參考 Azure 資源，例如 Azure 共用 IP 位址或流量管理員設定檔。 如果基礎資源的 IP 位址變更，別名記錄集會在 DNS 解析期間自行順暢地更新。 別名記錄集指向服務執行個體，而且服務執行個體與 IP 位址相關聯。 

此外，您可以使用別名記錄，立即將您的 apex 或裸網域 (例如，contoso.com) 指向流量管理員設定檔。

如需詳細資訊，請參閱 [Azure DNS 別名記錄的概觀](dns-alias.md)。


## <a name="next-steps"></a>後續步驟

* 了解 DNS 區域和記錄：[DNS 區域和記錄概觀](dns-zones-records.md)。

* 了解如何在 Azure DNS 中建立區域：[建立 DNS 區域](./dns-getstarted-create-dnszone-portal.md)。

* 如需有關 Azure DNS 的常見問題集，請參閱 [Azure DNS 常見問題集](dns-faq.md)。

