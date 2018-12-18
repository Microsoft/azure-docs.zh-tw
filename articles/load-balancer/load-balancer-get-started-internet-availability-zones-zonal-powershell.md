---
title: 使用 Azure PowerShell 來建立具有區域公用 IP 位址前端的公用 Load Balancer Standard | Microsoft Docs
description: 了解如何使用 Azure PowerShell 來建立具有區域公用 IP 位址前端的公用 Load Balancer Standard
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/26/2018
ms.author: kumud
ms.openlocfilehash: dbb4176ac61cf707b28cddc98db80a1188be3cc8
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31592125"
---
#  <a name="create-a-public-load-balancer-standard-with-zonal-frontend-using-azure-powershell"></a>使用 Azure PowerShell 來建立具有區域前端的公用 Load Balancer Standard

本文會逐步說明如何使用標準公用 IP 位址，來建立具有區域前端的公用 [Load Balancer Standard](https://aka.ms/azureloadbalancerstandard)。 若要了解可用性區域如何與標準 Load Balancer 搭配運作，請參閱[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!NOTE]
> 針對精選的 Azure 資源和區域及 VM 大小系列有提供「可用性區域」支援。 如需如何開始使用，以及有哪些 Azure 資源、區域和 VM 大小系列可供用來試用可用性區域，請參閱[可用性區域概觀](https://docs.microsoft.com/azure/availability-zones/az-overview)。 如需支援，您可以透過 [StackOverflow](https://stackoverflow.com/questions/tagged/azure-availability-zones) 與我們聯繫或[開啟 Azure 支援票證](../azure-supportability/how-to-create-azure-support-request.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="log-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Connect-AzureRmAccount
```

## <a name="create-resource-group"></a>建立資源群組

使用下列命令建立資源群組：

```powershell
New-AzureRmResourceGroup -Name myResourceGroupZLB -Location westeurope
```

## <a name="create-a-public-ip-standard"></a>建立 Public IP Standard 
使用下列命令建立 Public IP Standard：

```powershell
$publicIp = New-AzureRmPublicIpAddress -ResourceGroupName myResourceGroupZLB -Name 'myPublicIPZonal' `
  -Location westeurope -AllocationMethod Static -Sku Standard -zone 1
```

## <a name="create-a-front-end-ip-configuration-for-the-website"></a>為網站建立前端 IP 組態

使用下列命令來建立前端 IP 組態：

```powershell
$feip = New-AzureRmLoadBalancerFrontendIpConfig -Name 'myFrontEnd' -PublicIpAddress $publicIp
```

## <a name="create-the-back-end-address-pool"></a>建立後端位址集區

使用下列命令來建立後端位址集區：

```powershell
$bepool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'
```

## <a name="create-a-load-balancer-probe-on-port-80"></a>在連接埠 80 上建立負載平衡器探查

使用下列命令在連接埠 80 上建立負載平衡器地健康情況探查：

```powershell
$probe = New-AzureRmLoadBalancerProbeConfig -Name 'myHealthProbe' -Protocol Http -Port 80 `
  -RequestPath / -IntervalInSeconds 360 -ProbeCount 5
```

## <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則
 使用下列命令建立負載平衡器規則：

```powershell
   $rule = New-AzureRmLoadBalancerRuleConfig -Name HTTP -FrontendIpConfiguration $feip -BackendAddressPool  $bepool -Probe $probe -Protocol Tcp -FrontendPort 80 -BackendPort 80
```

## <a name="create-a-load-balancer"></a>建立負載平衡器
使用下列命令建立 Load Balancer Standard：

```powershell
$lb = New-AzureRmLoadBalancer -ResourceGroupName myResourceGroupZLB -Name 'MyLoadBalancer' -Location westeurope `
  -FrontendIpConfiguration $feip -BackendAddressPool $bepool `
  -Probe $probe -LoadBalancingRule $rule -Sku Standard
```

## <a name="next-steps"></a>後續步驟
- 深入了解[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。



