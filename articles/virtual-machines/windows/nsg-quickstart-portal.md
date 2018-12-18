---
title: 使用 Azure 入口網站對 VM 開啟連接埠 | Microsoft Docs
description: 了解如何在 Azure 入口網站中使用 Resource Manager 部署模型對 Windows VM 開啟連接埠 / 建立端點
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: f7cf0319-5ee7-435e-8f94-c484bf5ee6f1
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.openlocfilehash: 2820dcabf042d7463f9776b42f277a0457caf3b6
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37929010"
---
# <a name="how-to-open-ports-to-a-virtual-machine-with-the-azure-portal"></a>如何使用 Azure 入口網站開啟虛擬機器的連接埠
[!INCLUDE [virtual-machines-common-nsg-quickstart](../../../includes/virtual-machines-common-nsg-quickstart.md)]

## <a name="quick-commands"></a>快速命令
您也可以 [使用 Azure PowerShell 來執行這些步驟](nsg-quickstart-powershell.md)。

首先，建立您的「網路安全性群組」。 在入口網站中選取一個資源群組，選擇 [新增]，然後搜尋並選取 [網路安全性群組]：

![新增網路安全性群組](./media/nsg-quickstart-portal/add-nsg.png)

輸入您「網路安全性群組」的名稱，選取或建立資源群組，然後選取位置。 完成後，請選取 [建立]：

![建立網路安全性群組](./media/nsg-quickstart-portal/create-nsg.png)

選取您的新「網路安全性群組」。 選取 [輸入安全性規則]，然後選取 [新增] 按鈕來建立規則︰

![新增輸入規則](./media/nsg-quickstart-portal/add-inbound-rule.png)

若要建立允許流量的規則：

- 選取 [基本] 按鈕。 根據預設，[進階] 視窗會提供一些其他組態選項，例如定義特定來源 IP 區塊或連接埠範圍。
- 從下拉式功能表中選擇常見的**服務**，例如 *HTTP*。 您也可以選取 [自訂] 以提供特定連接埠供使用者使用。 
- 如有需要，請變更優先順序或名稱。 優先順序會影響規則的套用順序，數值越低會越早套用規則。
- 當您準備好時，選取 [確定] 以建立規則：

![建立輸入規則](./media/nsg-quickstart-portal/create-inbound-rule.png)

最後一個步驟是將您的「網路安全性群組」與子網路或特定網路介面建立關聯。 讓我們將「網路安全性群組」與子網路建立關聯。 選取 [子網路]，然後選擇 [關聯]：

![將網路安全性群組與子網路建立關聯](./media/nsg-quickstart-portal/associate-subnet.png)

選取您的虛擬網路，然後選取適當的子網路：

![將網路安全性群組與虛擬網路功能建立關聯](./media/nsg-quickstart-portal/select-vnet-subnet.png)

您現在已建立「網路安全性群組」、已建立允許連接埠 80 上流量的輸入規則，並且已將它與子網路建立關聯。 透過連接埠 80 可連線到您連接到該子網路的所有 VM。

## <a name="more-information-on-network-security-groups"></a>網路安全性群組的詳細資訊
這裡的快速命令可讓您使流向您 VM 的流量開始正常運作。 「網路安全性群組」提供許多絕佳的功能和細微性來控制對您資源的存取。 您可以深入了解 [建立網路安全性群組和 ACL 規則](../../virtual-network/tutorial-filter-network-traffic.md)。

針對高可用性 Web 應用程式，您應該將 VM 放在 Azure Load Balancer 後方。 此負載平衡器會將流量分散到所有 VM，並具有提供流量篩選的網路安全性群組。 如需詳細資訊，請參閱[如何平衡 Azure 中 Linux 虛擬機器的負載以建立高可用性應用程式](tutorial-load-balancer.md)。

## <a name="next-steps"></a>後續步驟
在此範例中，您建立了簡單的規則來允許 HTTP 流量。 您可以從下列文章中，找到有關建立更詳細環境的資訊︰

* [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)
* [什麼是網路安全性群組？](../../virtual-network/security-overview.md)