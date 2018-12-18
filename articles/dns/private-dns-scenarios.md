---
title: Azure DNS 私人區域的案例
description: 使用 Azure DNS 私人區域的常見案例概觀。
services: dns
author: vhorne
ms.service: dns
ms.topic: article
ms.date: 03/15/2018
ms.author: victorh
ms.openlocfilehash: d84da36ad6b1ef3e2a507a0944aac583861d5ccb
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39162162"
---
# <a name="azure-dns-private-zones-scenarios"></a>Azure DNS 私人區域案例
Azure DNS 私人區域能在虛擬網路內及虛擬網路之間提供名稱解析。 在本文中，我們會檢視可透過此功能達成的一些常見案例。 

[!INCLUDE [private-dns-public-preview-notice](../../includes/private-dns-public-preview-notice.md)]

## <a name="scenario-name-resolution-scoped-to-a-single-virtual-network"></a>案例：以單一虛擬網路作為範圍的名稱解析
在此案例中，您在 Azure 中擁有具數個 Azure 資源 (包括虛擬機器 (VM)) 的虛擬網路。 您想要透過特定的網域名稱 (DNS 區域) 從虛擬網路內解析資源，且名稱解析必須為私用並無法從網際網路存取。 此外，您需要 Azure 將 VNET 內的 VM 自動註冊至 DNS 區域。 

此案例的示意圖如下。 名為 "A" 的虛擬網路包含兩個 VM (VNETA-VM1 和 VNETA-VM2)。 這兩個 VM 都有相關聯的私人 IP。 在您建立名為 contoso.com 的私人區域，並將此虛擬網路連結為註冊虛擬網路之後，Azure DNS 將會在所顯示的區域中自動建立兩個 A 記錄。 接下來，來自 VNETA-VM1 要解析 VNETA-VM2.contoso.com 的 DNS 查詢，將會接收到包含 VNETA-VM2 私人 IP 的 DNS 回應。 此外，針對從 VNETA-VM2 所發出之 VNETA-VM1 (10.0.0.1) 私人 IP 的反向 DNS 查詢 (PTR)，將會如預期般接收到包含 VNETA-VM1 名稱的 DNS 回應。 

![單一虛擬網路解析](./media/private-dns-scenarios/single-vnet-resolution.png)

## <a name="scenario-name-resolution-across-virtual-networks"></a>案例：跨虛擬網路的名稱解析

此為較常見的案例，其中您會需要將私人區域與多個虛擬網路關聯。 此案例適用於中樞和支點模型之類的架構。中樞和支點模型具有一個中央中樞虛擬網路，以及連線至該虛擬網路的多個其他支點虛擬網路。 中央中樞虛擬網路可以作為註冊虛擬網路連結至私人區域，而支點虛擬網路則可以作為解析虛擬網路進行連結。 

下圖顯示此案例的簡單版本，其中只有兩個虛擬網路：A 和 B。我們將 A 指定為註冊虛擬網路，並將 B 指定為解析虛擬網路。 此目的是為了讓這兩個虛擬網路都共用相同的區域，contoso.com。 在區域建立完畢，且解析及註冊虛擬網路皆已連結至該區域後，Azure 將會自動註冊來自虛擬網路 A 之 VM (VNETA-VM1 和 VNETA-VM2) 的 DNS 記錄。您也可以針對位於解析虛擬網路 B 中的 VM，將 DNS 記錄手動新增至區域。透過此設定，您將能針對順向及反向 DNS 查詢觀察到下列行為：
* 來自解析虛擬網路 B 中之 VNETB-VM1 的 DNS 查詢 (針對 VNETA-VM1.contoso.com)，將會接收到包含 VNETA-VM1 之私人 IP 的 DNS 回應。
* 來自解析虛擬網路 B 中之 VNETB-VM2 的反向 DNS (PTR) 查詢 (針對 10.1.0.1)，將會接收到包含 VNETB-VM1.contoso.com 之 FQDN 的 DNS 回應。 原因是反向 DNS 查詢的範圍為相同的虛擬網路。 
* 來自解析虛擬網路 B 中之 VNETB-VM3 的反向 DNS (PTR) 查詢 (針對 10.0.0.1)，將會接收到 NXDOMAIN。 原因是反向 DNS 查詢的範圍僅為相同的虛擬網路。 


![多個虛擬網路解析](./media/private-dns-scenarios/multi-vnet-resolution.png)

## <a name="scenario-split-horizon-functionality"></a>案例：水平分割功能

此案例中的使用案例為您想要針對相同的 DNS 區域，根據用戶端所在的位置 (位於 Azure 內或是網際網路上) 實現不同的 DNS 解析行為。 例如，您的應用程式可能有具不同功能或行為的私人及公用版本，但您想要針對這兩個版本使用相同的網域名稱。 此案例可以透過使用 Azure DNS 建立具有相同名稱的公用 DNS 區域及私人區域來實現。

下圖會描述此案例。 您擁有具兩個 VM (VNETA-VM1 和 VNETA-VM2) 的虛擬網路 A，這兩個 VM 皆已配置私人 IP 及公用 IP。 您建立稱為 contoso.com 的公用 DNS 區域，並將這些 VM 的公用 IP 註冊為區域內的 DNS 記錄。 您也建立一樣稱為 contoso.com 的私人 DNS 區域，並將 A 指定為註冊虛擬網路。 Azure 會在 A 記錄至私人區域時自動註冊 VM，並指向其私人 IP。

當有網路網路用戶端發出 DNS 查詢以查閱 VNETA-VM1.contoso.com 時，Azure 將會傳回來自公用區域的公用 IP 記錄。 如果相同的 DNS 查詢是由來自相同虛擬網路 A 的另一個 VM (例如 VNETA-VM2) 所發出，Azure 將會傳回來自私人區域的私人 IP 記錄。 

![裂腦解析](./media/private-dns-scenarios/split-brain-resolution.png)

## <a name="next-steps"></a>後續步驟
若要深入了解私人 DNS 區域，請參閱[使用 Azure DNS 私人網域](private-dns-overview.md)。

了解如何在 Azure DNS 中[建立私人 DNS 區域](./private-dns-getstarted-powershell.md)。

如需深入了解 DNS 區域和記錄，請瀏覽：[DNS 區域和記錄的概觀](dns-zones-records.md)。

了解 Azure 的一些其他重要[網路功能](../networking/networking-overview.md)。

