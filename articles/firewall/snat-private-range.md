---
title: Azure 防火牆 SNAT 私人 IP 位址範圍
description: 您可以設定 SNAT 的 IP 位址範圍。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 01/11/2021
ms.author: victorh
ms.openlocfilehash: 0df91680dadbc4ac19299a4df48a585a11f044e8
ms.sourcegitcommit: 3af12dc5b0b3833acb5d591d0d5a398c926919c8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2021
ms.locfileid: "98072236"
---
# <a name="azure-firewall-snat-private-ip-address-ranges"></a>Azure 防火牆 SNAT 私人 IP 位址範圍

Azure 防火牆會針對公用 IP 位址的所有輸出流量提供自動 SNAT。 根據預設，當目的地 IP 位址為每個 [IANA RFC 1918](https://tools.ietf.org/html/rfc1918)的私人 ip 位址範圍時，Azure 防火牆不會使用網路規則進行 SNAT。 無論目的地 IP 位址為何，一律會使用 [透明 proxy](https://wikipedia.org/wiki/Proxy_server#Transparent_proxy) 來套用應用程式規則。

當您將流量直接路由傳送到網際網路時，此邏輯的運作效果很好。 不過，如果您已啟用 [強制通道](forced-tunneling.md)，則會將網際網路系結流量 Snat 轉譯至 AzureFirewallSubnet 中的其中一個防火牆私人 IP 位址，並將來源從內部部署防火牆中隱藏。

如果您的組織使用私人網路的公用 IP 位址範圍，Azure 防火牆會將流量 SNAT 轉譯到 AzureFirewallSubnet 其中一個防火牆私人 IP 位址。 不過，您可以將 Azure 防火牆設定為 **不** 會 SNAT 公用 IP 位址範圍。 例如，若要指定個別的 IP 位址，您可以指定它，如下所示： `192.168.1.10` 。 若要指定 IP 位址範圍，您可以指定它，如下所示： `192.168.1.0/24` 。

- 若要將 Azure 防火牆設定為 **永遠不** 使用 SNAT，而不考慮目的地 ip 位址，請使用 **0.0.0.0/0** 做為您的私人 ip 位址範圍。 使用此設定時，Azure 防火牆永遠不會將流量直接路由傳送到網際網路。 

- 若要將防火牆設定為 **一律** 使用 SNAT （不論目的地位址為何），請使用 **255.255.255.255/32** 做為您的私人 IP 位址範圍。

> [!IMPORTANT]
> 您指定的私人位址範圍只會套用到網路規則。 目前，應用程式規則一律為 SNAT。

> [!IMPORTANT]
> 如果您想要指定自己的私人 IP 位址範圍，並保留預設的 IANA RFC 1918 位址範圍，請確定您的自訂清單仍包含 IANA RFC 1918 範圍。 

## <a name="configure-snat-private-ip-address-ranges---azure-powershell"></a>設定 SNAT 私人 IP 位址範圍-Azure PowerShell

您可以使用 Azure PowerShell 來指定防火牆的私人 IP 位址範圍。

### <a name="new-firewall"></a>新增防火牆

針對新的防火牆，Azure PowerShell Cmdlet 是：

```azurepowershell
$azFw = @{
    Name               = '<fw-name>'
    ResourceGroupName  = '<resourcegroup-name>'
    Location           = '<location>'
    VirtualNetworkName = '<vnet-name>'
    PublicIpName       = '<public-ip-name>'
    PrivateRange       = @("IANAPrivateRanges", "192.168.1.0/24", "192.168.1.10")
}

New-AzFirewall @azFw
```
> [!NOTE]
> 使用部署 Azure 防火牆 `New-AzFirewall` 需要現有的 VNet 和公用 IP 位址。 如需完整部署指南，請參閱 [使用 Azure PowerShell 部署和設定 Azure 防火牆](deploy-ps.md) 。

> [!NOTE]
> IANAPrivateRanges 會擴充至 Azure 防火牆上目前的預設值，而其他範圍則會新增至其中。 若要在您的私用範圍規格中保留 IANAPrivateRanges 預設值，它必須保留在您 `PrivateRange` 的規格中，如下列範例所示。

如需詳細資訊，請參閱 [AzFirewall](/powershell/module/az.network/new-azfirewall?view=azps-3.3.0)。

### <a name="existing-firewall"></a>現有的防火牆

若要設定現有的防火牆，請使用下列 Azure PowerShell Cmdlet：

```azurepowershell
$azfw = Get-AzFirewall -Name '<fw-name>' -ResourceGroupName '<resourcegroup-name>'
$azfw.PrivateRange = @("IANAPrivateRanges","192.168.1.0/24", "192.168.1.10")
Set-AzFirewall -AzureFirewall $azfw
```

## <a name="configure-snat-private-ip-address-ranges---azure-cli"></a>設定 SNAT 私人 IP 位址範圍-Azure CLI

您可以使用 Azure CLI 來指定防火牆的私人 IP 位址範圍。

### <a name="new-firewall"></a>新增防火牆

針對新的防火牆，Azure CLI 命令為：

```azurecli-interactive
az network firewall create \
-n <fw-name> \
-g <resourcegroup-name> \
--private-ranges 192.168.1.0/24 192.168.1.10 IANAPrivateRanges
```

> [!NOTE]
> 使用 Azure CLI 命令部署 Azure 防火牆 `az network firewall create` 需要額外的設定步驟，以建立公用 IP 位址和 IP 設定。 如需完整部署指南，請參閱 [使用 Azure CLI 部署和設定 Azure 防火牆](deploy-cli.md) 。

> [!NOTE]
> IANAPrivateRanges 會擴充至 Azure 防火牆上目前的預設值，而其他範圍則會新增至其中。 若要在您的私用範圍規格中保留 IANAPrivateRanges 預設值，它必須保留在您 `PrivateRange` 的規格中，如下列範例所示。

### <a name="existing-firewall"></a>現有的防火牆

若要設定現有的防火牆，Azure CLI 命令為：

```azurecli-interactive
az network firewall update \
-n <fw-name> \
-g <resourcegroup-name> \
--private-ranges 192.168.1.0/24 192.168.1.10 IANAPrivateRanges
```

## <a name="configure-snat-private-ip-address-ranges---arm-template"></a>設定 SNAT 私人 IP 位址範圍-ARM 範本

若要在 ARM 範本部署期間設定 SNAT，您可以將下列內容新增至 `additionalProperties` 屬性：

```json
"additionalProperties": {
   "Network.SNAT.PrivateRanges": "IANAPrivateRanges , IPRange1, IPRange2"
},
```

## <a name="configure-snat-private-ip-address-ranges---azure-portal"></a>設定 SNAT 私人 IP 位址範圍-Azure 入口網站

您可以使用 Azure 入口網站來指定防火牆的私人 IP 位址範圍。

1. 選取您的資源群組，然後選取您的防火牆。
2. 在 [ **總覽** ] 頁面的 [ **私人 IP 範圍**] 中，選取預設值 [ **IANA RFC 1918**]。

   [ **編輯私人 IP 首碼** ] 頁面隨即開啟：

   :::image type="content" source="media/snat-private-range/private-ip.png" alt-text="編輯私人 IP 首碼":::

1. 依預設，會設定 **IANAPrivateRanges** 。
2. 編輯環境的私人 IP 位址範圍，然後選取 [ **儲存**]。

## <a name="next-steps"></a>後續步驟

- 瞭解 [Azure 防火牆強制通道](forced-tunneling.md)。