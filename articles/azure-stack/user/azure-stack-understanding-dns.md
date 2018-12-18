---
title: 了解 Azure Stack 中的 DNS | Microsoft Docs
description: 了解 Azure Stack 中的 DNS 特性與功能
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-09/28/2018started-article
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: scottnap
ms.openlocfilehash: 783262a5b55bd645ae3b85c1f00434648d7ee35f
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47584962"
---
# <a name="introducing-idns-for-azure-stack"></a>適用於 Azure Stack 的 iDNS 簡介

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

iDNS 是 Azure Stack 網路功能，可讓您解析外部 DNS 名稱 (例如， http://www.bing.com.) ，也可讓您註冊內部虛擬網路名稱。 如此一來，您就可以依名稱 (而非 IP 位址) 解析相同虛擬網路上的 VM。 此方法不需要提供自訂 DNS 伺服器項目。 如需 DNS 的詳細資訊，請參閱 [Azure DNS 概觀](https://docs.microsoft.com/azure/dns/dns-overview)。

## <a name="what-does-idns-do"></a>iDNS 的用途為何？

您可以使用 Azure Stack 中的 iDNS 取得下列功能，而不需要指定自訂的 DNS 伺服器項目：

- 適用於租用戶工作負載的共用 DNS 名稱解析服務。
- 適用於租用戶虛擬網路內的名稱解析和 DNS 登錄的權威 DNS 服務。
- 從租用戶 VM 解析網際網路名稱的遞迴 DNS 服務。 租用戶不再需要指定自訂的 DNS 項目，就可以解析網際網路名稱 (例如 www.bing.com)。

您仍然可以沿用您自己的 DNS，也可以使用自訂的 DNS 伺服器。 不過，藉由使用 iDNS，您可以解析網際網路 DNS 名稱並連線到相同虛擬網路中的其他 VM，而不需要建立自訂 DNS 項目。

## <a name="what-doesnt-idns-do"></a>iDNS 禁止的作業為何？

iDNS 不允許您針對可從虛擬網路外部解析的名稱，建立 DNS 記錄。

在 Azure 中，您可以選擇指定與公用 IP 位址相關聯的 DNS 名稱標籤。 您可以選擇標籤 (首碼)，但 Azure 會根據您建立公用 IP 位址所在的區域，選擇尾碼。

![DNS 名稱標籤的範例](media/azure-stack-understanding-dns-in-tp2/image3.png)

如上圖所示，Azure 將會在 DNS 中，為區域 **westus.cloudapp.azure.com** 底下指定的 DNS 名稱標籤，建立 “A” 記錄。 首碼和尾碼可合併構成[完整網域名稱](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (FQDN)，此名稱可以從公用網際網路上的任何位置解析。

Azure Stack 僅支援用於內部名稱登錄的 iDNS，因此無法執行下列作業：

- 在現有的託管 DNS 區域 (例如 local.azurestack.external) 底下建立 DNS 記錄。
- 建立 DNS 區域 (例如 Contoso.com)。
- 在您自己的自訂 DNS 區域底下建立記錄。
- 支援購買網域名稱。

## <a name="next-steps"></a>後續步驟

[在 Azure Stack 中使用 DNS](azure-stack-dns.md)
