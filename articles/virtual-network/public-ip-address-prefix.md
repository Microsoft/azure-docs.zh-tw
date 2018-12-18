---
title: Azure 公用 IP 位址首碼 | Microsoft Docs
description: 了解什麼是 Azure 公用 IP 位址首碼，以及它如何協助您為資源指派可預測的的公用 IP 位址。
services: virtual-network
documentationcenter: na
author: anavinahar
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: anavin
ms.openlocfilehash: 5bbe0709f89ca198b0571526291f700c99e9e59f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966770"
---
# <a name="public-ip-address-prefix"></a>公用 IP 位址首碼

公用 IP 位址首碼是 Azure 中公用端點的 IP 位址保留範圍。 Azure 會根據您指定的數量，為您的訂用帳戶配置連續的位址範圍。 如果您不熟悉如何使用公用位址，請參閱[公用 IP 位址](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses)。

將會從每個 Azure 區域的位址集區指派公用 IP 位址。 您可以[下載](https://www.microsoft.com/download/details.aspx?id=41653) Azure 用於每個區域的範圍清單。 例如，40.121.0.0/16 是 Azure 在美國東部區域使用的 100 多個範圍之一。 此範圍也包括 40.121.0.1 - 40.121.255.254 的可用位址。

您可以透過指定名稱與希望首碼包含的位址數目，在 Azure 區域與訂用帳戶中建立公用 IP 位址首碼。 例如，如果您建立 /28 的公用 IP 位址首碼，Azure 將為您配置其中一個範圍的 16 個位址。 在建立範圍之前，您不知道 Azure 會指派哪一個範圍，但位址是連續的。 公用 IP 位址首碼需要付費。 如需詳細資訊，請參閱[公用 IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses)。

> [!IMPORTANT]
> 公用 IP 位址首碼在有限的區域內為公開預覽。 您可以[了解預覽的意義](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 公用 IP 位址首碼目前已於下列區域提供：美國中西部、美國西部、美國西部 2、美國中部、北歐、西歐與東南亞。 如需區域的更新清單，請瀏覽 [Azure 更新](https://azure.microsoft.com/updates/?product=virtual-network)。

## <a name="why-create-a-public-ip-address-prefix"></a>為什麼要建立公用 IP 位址首碼？

當您建立公用 IP 位址資源時，Azure 會從區域中使用的任何範圍指派可用的公用 IP 位址。 Azure 指派位址後，您就會知道是哪個位址，但在 Azure 指派位址之前，您不知道可能指派的地址。 例如，當您或您的業務合作夥伴設定允許特定 IP 位址的防火牆規則時，這可能會造成問題。 每次為資源指派新的公用 IP 位址時，都必須將該位址加入至防火牆規則中。 當您從公用 IP 位址首碼將位址指派給您的資源時，不需要每次指派其中一個位址時就更新防火牆規則，因為整個範圍可以新增至規則中。

## <a name="benefits"></a>優點

- 您可以從已知範圍建立公用 IP 位址資源。
- 您或您的業務合作夥伴可以建立防火牆規則，其範圍包括您目前獲指派的公用 IP 位址，以及尚未分配的位址。 這樣就不需要在將位址指派給新資源時變更防火牆規則。
- 您可以建立之範圍的預設大小是 /28 或 16 個 IP 位址。
- 您可以建立多少範圍沒有任何限制，但是，Azure 訂用帳戶中可以擁有的靜態公用 IP 位址的數目上限有限制。 因此，您所建立的範圍數目不能包含比訂用帳戶中的位址更多的靜態公用 IP 位址。 如需詳細資訊，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)。
- 使用首碼中的位址建立的位址，可以指派給任何可為其指派公用 IP 位址的 Azure 資源。
- 您可以輕鬆查看該範圍內已配置與尚未配置的 IP 位址。

## <a name="scenarios"></a>案例
您可以將下列資源與首碼中的靜態公用 IP 位址產生關聯：

|資源|案例|步驟|
|---|---|---|
|虛擬機器| 將公用 IP 從首碼與您在 Azure 中的虛擬機器產生關聯，可以減少在防火牆中建立 IP 允許清單的管理負荷。 您可以使用單一防火牆規則，將整個首碼列入白名單。 當您在調整 Azure 中的虛擬機器規模時，可以關聯來自相同首碼的 IP，進而節省成本、時間與管理額外負荷。| 將首碼中的 IP 與您的虛擬機器產生關聯：1. [建立首碼](manage-public-ip-address-prefix.md)。 2. [從首碼建立 IP](manage-public-ip-address-prefix.md)。 3. [將 IP 與您的虛擬機器網路介面產生關聯](virtual-network-network-interface-addresses.md#add-ip-addresses)。
| 負載平衡器 | 將首碼中的公用 IP 與您的前端 IP 設定或負載平衡器的連出規則產生關聯，以確保簡化您的 Azure 公用 IP 位址空間。 您可以透過清理連出連線，使其源自公用 IP 首碼定義的連續 IP 位址範圍，來簡化您的案例。 | 將首碼中的 IP 與您的負載平衡器產生關聯：1. [建立首碼](manage-public-ip-address-prefix.md)。 2. [從首碼建立 IP](manage-public-ip-address-prefix.md)。 3. 建立負載平衡器時，選取或更新上面步驟 2 中建立的 IP 作為您負載平衡器的前端 IP。 |
| Azure 防火牆 | 您可以為連出 SNAT 使用首碼中的公用 IP。 這表示所有連出虛擬網路流量都會轉譯為 [Azure 防火牆](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)公用 IP。 由於此 IP 來自於預定的首碼，因此很容易就能事先知道 Azure 中的公用 IP 將是什麼樣子。 | 1.[建立首碼](manage-public-ip-address-prefix.md)。 2. [從首碼建立 IP](manage-public-ip-address-prefix.md)。 3. 當您[部署 Azure 防火牆](../firewall/tutorial-firewall-deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-the-firewall)時，請務必選取您之前從首碼中配置的 IP。|

## <a name="constraints"></a>條件約束

- 您無法指定首碼的 IP 位址。 Azure 會根據您指定的大小為首碼配置 IP 位址。
- 建立首碼之後，就無法變更範圍。
- 範圍僅適用於 IPv4 位址。 範圍不包含 IPv6 位址。
- 只能從首碼的範圍指派使用標準 SKU 建立的靜態公用 IP 位址。 若要深入了解公用 IP 位址 SKU，請參閱[公用 IP 位址](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses)。
- 範圍中的位址只能指派給 Azure Resource Manager 資源。 無法將位址指派給傳統部署模型中的資源。
- 從首碼建立的所有公用 IP 位址必須與首碼存在於相同 Azure 區域和訂用帳戶中，並且必須指派給相同區域和訂用帳戶中的資源。
- 如果首碼中的任何位址指派給與資源相關聯的公用 IP 位址資源，則無法刪除該首碼。 首先，中斷已指派來自首碼 IP 位址之所有公用 IP 位址資源的關聯。


## <a name="next-steps"></a>後續步驟

- [建立](manage-public-ip-address-prefix.md)公用 IP 位址首碼
