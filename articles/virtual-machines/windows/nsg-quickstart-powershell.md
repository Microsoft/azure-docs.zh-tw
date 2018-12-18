---
title: 使用 Azure PowerShell 對 VM 開啟連接埠 | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 部署模式和 Azure PowerShell 對 Windows VM 開啟連接埠 / 建立端點
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: cf45f7d8-451a-48ab-8419-730366d54f1e
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.openlocfilehash: f79db8cdec0aa48ae300aff4c58072fb6afdc932
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932777"
---
# <a name="how-to-open-ports-and-endpoints-to-a-vm-in-azure-using-powershell"></a>如何使用 PowerShell 對 Azure 中的 VM 開啟連接埠與端點
[!INCLUDE [virtual-machines-common-nsg-quickstart](../../../includes/virtual-machines-common-nsg-quickstart.md)]

## <a name="quick-commands"></a>快速命令
若要建立網路安全性群組和 ACL 規則，您需要 [安裝最新版的 Azure PowerShell](/powershell/azureps-cmdlets-docs)。 您也可以 [使用 Azure 入口網站來執行這些步驟](nsg-quickstart-portal.md)。

登入您的 Azure 帳戶：

```powershell
Connect-AzureRmAccount
```

在下列範例中，請以您自己的值取代參數名稱。 範例參數名稱包括 myResourceGroup、myNetworkSecurityGroup 和 myVnet。

使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) 建立規則。 下列範例會建立名為 myNetworkSecurityGroupRule 的規則，以允許連接埠 80  上的 tcp 流量︰

```powershell
$httprule = New-AzureRmNetworkSecurityRuleConfig `
    -Name "myNetworkSecurityGroupRule" `
    -Description "Allow HTTP" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority "100" `
    -SourceAddressPrefix "Internet" `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 80
```

接著，使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) 建立網路安全性群組，並依下列方式指派您剛才建立的 HTTP 規則。 下列範例建立名為 myNetworkSecurityGroup 的網路安全性群組：

```powershell
$nsg = New-AzureRmNetworkSecurityGroup `
    -ResourceGroupName "myResourceGroup" `
    -Location "EastUS" `
    -Name "myNetworkSecurityGroup" `
    -SecurityRules $httprule
```

現在，讓我們將「網路安全性群組」指派給子網路。 下列範例使用 [Get-AzureRmVirtualNetwork](/powershell/module/azurerm.network/get-azurermvirtualnetwork) 將名為 myVnet 的現有虛擬網路指派至變數 $vnet：

```powershell
$vnet = Get-AzureRmVirtualNetwork `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVnet"
```

使用 [Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig) 將網路安全性群組與子網路建立關聯。 下列範例會將名為 mySubnet 的子網路與您的網路安全性群組建立關聯：

```powershell
$subnetPrefix = $vnet.Subnets|?{$_.Name -eq 'mySubnet'}

Set-AzureRmVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name "mySubnet" `
    -AddressPrefix $subnetPrefix.AddressPrefix `
    -NetworkSecurityGroup $nsg
```

最後，使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork) 更新虛擬網路，以便讓變更生效︰

```powershell
Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```


## <a name="more-information-on-network-security-groups"></a>網路安全性群組的詳細資訊
這裡的快速命令可讓您使流向您 VM 的流量開始正常運作。 「網路安全性群組」提供許多絕佳的功能和細微性來控制對您資源的存取。 您可以深入了解 [建立網路安全性群組和 ACL 規則](tutorial-virtual-network.md#secure-network-traffic)。

針對高可用性 Web 應用程式，您應該將 VM 放在 Azure Load Balancer 後方。 此負載平衡器會將流量分散到所有 VM，並具有提供流量篩選的網路安全性群組。 如需詳細資訊，請參閱[如何平衡 Azure 中 Linux 虛擬機器的負載以建立高可用性應用程式](tutorial-load-balancer.md)。

## <a name="next-steps"></a>後續步驟
在此範例中，您建立了簡單的規則來允許 HTTP 流量。 您可以從下列文章中，找到有關建立更詳細環境的資訊︰

* [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)
* [什麼是網路安全性群組？](../../virtual-network/security-overview.md)
* [負載平衡器的 Azure Resource Manager 概觀](../../load-balancer/load-balancer-arm.md)

