---
title: 建立 Azure 內部負載平衡器 - PowerShell 傳統 | Microsoft Docs
description: 了解如何在傳統部署模型中使用 PowerShell 建立內部負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: 3be93168-3787-45a5-a194-9124fe386493
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2017
ms.author: genli
ms.openlocfilehash: 8b896705d90b51c056172c285a00dabeed54ebf2
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38697129"
---
# <a name="get-started-creating-an-internal-load-balancer-classic-using-powershell"></a>開始使用 PowerShell 建立內部負載平衡器 (傳統)

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-classic-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-ilb-classic-cli.md)
> * [雲端服務](../load-balancer/load-balancer-get-started-ilb-classic-cloud.md)

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何[使用 Resource Manager 模型執行這些步驟](load-balancer-get-started-ilb-arm-ps.md)。

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-an-internal-load-balancer-set-for-virtual-machines"></a>建立虛擬機器的內部負載平衡器集

若要建立內部負載平衡器集，以及向其傳送流量的伺服器，您必須執行下列動作：

1. 建立將會是連入流量端點的內部負載平衡執行個體，連入流量會在負載平衡集合的不同伺服器之間進行負載平衡。
2. 新增對應到虛擬機器 (將會接收連入流量) 的端點。
3. 設定即將傳送流量進行負荷平衡的伺服器將其流量傳送到內部負載平衡執行個體的虛擬 IP (VIP) 位址。

### <a name="step-1-create-an-internal-load-balancing-instance"></a>步驟 1︰建立內部負載平衡執行個體

在現有的雲端服務或在區域虛擬網路下部署的雲端服務中，您可以使用下列 Windows PowerShell 命令來建立內部負載平衡執行個體：

```powershell
$svc="<Cloud Service Name>"
$ilb="<Name of your ILB instance>"
$subnet="<Name of the subnet within your virtual network>"
$IP="<The IPv4 address to use on the subnet-optional>"

Add-AzureInternalLoadBalancer -ServiceName $svc -InternalLoadBalancerName $ilb –SubnetName $subnet –StaticVNetIPAddress $IP
```

請注意，使用 [Add-AzureEndpoint](https://msdn.microsoft.com/library/dn495300.aspx) Windows PowerShell Cmdlet 會使用 DefaultProbe 參數集。 如需其他參數集的詳細資訊，請參閱 [Add-AzureEndpoint](https://msdn.microsoft.com/library/dn495300.aspx)。

### <a name="step-2-add-endpoints-to-the-internal-load-balancing-instance"></a>步驟 2：將端點加入至內部負載平衡執行個體

下列是一個範例：

```powershell
$svc="mytestcloud"
$vmname="DB1"
$epname="TCP-1433-1433"
$lbsetname="lbset"
$prot="tcp"
$locport=1433
$pubport=1433
$ilb="ilbset"
Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -Lbset $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM
```

### <a name="step-3-configure-your-servers-to-send-their-traffic-to-the-new-internal-load-balancing-endpoint"></a>步驟 3：設定您的伺服器將其流量傳送到新的內部負載平衡端點

您必須設定其流量即將進行負載平衡的伺服器使用內部負載平衡執行個體的新 IP 位址 (VIP)。 這是內部負載平衡執行個體所接聽的位址。 在大部分情況下，您只需要針對內部負載平衡執行個體的 VIP 新增或修改 DNS 記錄。

如果您在建立內部負載平衡執行個體的過程中指定 IP 位址，則您已具有 VIP。 否則，您可以使用下列命令查看 VIP：

```powershell
$svc="<Cloud Service Name>"
Get-AzureService -ServiceName $svc | Get-AzureInternalLoadBalancer
```

若要使用這些命令，請填入值並移除 < 和 >。 下列是一個範例：

```powershell
$svc="mytestcloud"
Get-AzureService -ServiceName $svc | Get-AzureInternalLoadBalancer
```

在 Get-AzureInternalLoadBalancer 命令的顯示中，請記下 IP 位址，並對伺服器或 DNS 記錄進行必要的變更，以確保流量會被傳送到 VIP。

> [!NOTE]
> Microsoft Azure 平台會對各種管理案例使用靜態、可公開路由傳送的 IPv4 位址。 IP 位址是 168.63.129.16。 此 IP 位址不應該遭到任何防火牆封鎖，因為可能會造成非預期的行為。
> 採用 Azure 內部負載平衡，來自負載平衡器的監視探查會使用此 IP 位址，藉此判斷虛擬機器在負載平衡集的健全狀態。 如果網路安全性群組已用來限制傳輸至內部負載平衡集中 Azure 虛擬機器的流量，或已套用至虛擬網路子網路，請確定您已新增網路安全性規則，允許來自 168.63.129.16 的流量。

## <a name="example-of-internal-load-balancing"></a>內部負載平衡的範例

若要為兩個範例組態逐步完成建立負載平衡集合的端對端程序，請參閱下列各節。

### <a name="an-internet-facing-multi-tier-application"></a>網際網路面向的多層式應用程式

您想要為一組網際網路面向的 Web 伺服器提供負載平衡資料庫服務。 這兩組伺服器都會裝載於單一 Azure 雲端服務。 進入 TCP 通訊埠 1433 的 Web 伺服器流量必須分配到資料庫層中的兩部虛擬機器。 圖 1 顯示組態。

![資料庫層的內部負載平衡集合](./media/load-balancer-internal-getstarted/IC736321.png)

組態包含下列各項：

* 裝載虛擬機器的現有雲端服務會命名為 mytestcloud。
* 兩個現有的資料庫伺服器會命名為 DB1、DB2。
* Web 層中的 Web 伺服器會使用私人 IP 位址連接到資料庫層中的資料庫伺服器。 另一個選項是為虛擬網路使用您自己的 DNS，並手動註冊內部負載平衡器集的 A 記錄。

下列命令會設定名為 **ILBset** 的新內部負載平衡執行個體，並將端點新增至對應到兩部資料庫伺服器的虛擬機器：

```powershell
$svc="mytestcloud"
$ilb="ilbset"
Add-AzureInternalLoadBalancer -ServiceName $svc -InternalLoadBalancerName $ilb
$prot="tcp"
$locport=1433
$pubport=1433
$epname="TCP-1433-1433"
$lbsetname="lbset"
$vmname="DB1"
Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -LbSetName $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

$epname="TCP-1433-1433-2"
$vmname="DB2"
Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -LbSetName $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM
```

## <a name="remove-an-internal-load-balancing-configuration"></a>移除內部負載平衡組態

若要將虛擬機器從內部負載平衡器執行個體的端點中移除，請使用下列命令：

```powershell
$svc="<Cloud service name>"
$vmname="<Name of the VM>"
$epname="<Name of the endpoint>"
Get-AzureVM -ServiceName $svc -Name $vmname | Remove-AzureEndpoint -Name $epname | Update-AzureVM
```

若要使用這些命令，請填入值並移除 < 和 >。

下列是一個範例：

```powershell
$svc="mytestcloud"
$vmname="DB1"
$epname="TCP-1433-1433"
Get-AzureVM -ServiceName $svc -Name $vmname | Remove-AzureEndpoint -Name $epname | Update-AzureVM
```

若要將內部負載平衡器執行個體從雲端服務中移除，請使用下列命令：

```powershell
$svc="<Cloud service name>"
Remove-AzureInternalLoadBalancer -ServiceName $svc
```

若要使用這些命令，請填入值並移除 < 和 >。

下列是一個範例：

```powershell
$svc="mytestcloud"
Remove-AzureInternalLoadBalancer -ServiceName $svc
```

## <a name="additional-information-about-internal-load-balancer-cmdlets"></a>內部負載平衡器 Cmdlet 的其他資訊

若要取得內部負載平衡 Cmdlet 的詳細資訊，請在 Windows PowerShell 提示字元中執行下列命令：

```powershell
Get-Help New-AzureInternalLoadBalancerConfig -full
Get-Help Add-AzureInternalLoadBalancer -full
Get-Help Get-AzureInternalLoadbalancer -full
Get-Help Remove-AzureInternalLoadBalancer -full
```

## <a name="next-steps"></a>後續步驟

[使用來源 IP 同質性設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)

