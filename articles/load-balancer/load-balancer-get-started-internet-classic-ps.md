---
title: 建立網際網路對向負載平衡器 - Azure PowerShell 傳統 | Microsoft Docs
description: 了解如何使用 PowerShell 在傳統模式中建立網際網路面向的負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
tags: azure-service-management
ms.assetid: 73e8bfa4-8086-4ef0-9e35-9e00b24be319
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2017
ms.author: genli
ms.openlocfilehash: 07d3658ff86a46875a57cb3359a60661911e0c8b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38539864"
---
# <a name="get-started-creating-an-internet-facing-load-balancer-classic-in-powershell"></a>開始在 PowerShell 中建立網際網路面向的負載平衡器 (傳統)

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-internet-classic-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-internet-classic-cli.md)
> * [Azure 雲端服務](../load-balancer/load-balancer-get-started-internet-classic-cloud.md)

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

> [!IMPORTANT]
> 使用 Azure 資源之前，請務必了解 Azure 目前有 Azure Resource Manager 和「傳統」兩種部署模型。 在使用任何 Azure 資源之前，請先確認您了解 [部署模型和工具](../azure-classic-rm.md) 。 您可以按一下本文頂端的索引標籤，檢視不同工具的文件。 本文涵蓋之內容包括傳統部署模型。 您也可以 [了解如何使用 Azure 資源管理員建立網際網路面向的負載平衡器](load-balancer-get-started-internet-arm-ps.md)。

[!INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## <a name="set-up-load-balancer-using-powershell"></a>使用 PowerShell 設定負載平衡器

若要使用 PowerShell 設定負載平衡器，請完成下列步驟：

1. 如果您從未用過 Azure PowerShell，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) ，並遵循其中的所有指示登入 Azure，然後選取您的訂用帳戶。
2. 建立虛擬機器之後，您可以使用 PowerShell cmdlet，將負載平衡器新增至相同雲端服務內的虛擬機器。

在下列範例中，您會將稱為 "webfarm" 的負載平衡器集新增至雲端服務 "mytestcloud" (或 myctestcloud.cloudapp.net)，並將負載平衡器的端點新增至名為 "web1" 和 "web2" 的虛擬機器。 負載平衡器會接收連接埠 80 的流量，並使用 TCP 負載平衡本機端點 (在此案例中為連接埠 80) 所定義之虛擬機器之間的流量。

### <a name="step-1"></a>步驟 1

為第一部 VM "web1" 建立負載平衡端點

```powershell
Get-AzureVM -ServiceName "mytestcloud" -Name "web1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 80 -LBSetName "WebFarm" -ProbePort 80 -ProbeProtocol "http" -ProbePath '/' | Update-AzureVM
```

### <a name="step-2"></a>步驟 2

使用相同的負載平衡器集名稱，為第二部 VM "web2" 建立另一個端點

```powershell
Get-AzureVM -ServiceName "mytestcloud" -Name "web2" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 80 -LBSetName "WebFarm" -ProbePort 80 -ProbeProtocol "http" -ProbePath '/' | Update-AzureVM
```

## <a name="remove-a-virtual-machine-from-a-load-balancer"></a>從負載平衡器移除虛擬機器

您可以使用 Remove-AzureEndpoint 從負載平衡器移除虛擬機器端點

```powershell
Get-azureVM -ServiceName mytestcloud  -Name web1 |Remove-AzureEndpoint -Name httpin | Update-AzureVM
```

## <a name="next-steps"></a>後續步驟

您也可以[開始建立內部負載平衡器](load-balancer-get-started-ilb-classic-ps.md)，以及為特定負載平衡器的網路流量行為設定何種[分配模式](load-balancer-distribution-mode.md)。

如果您的應用程式需要讓負載平衡器後方的伺服器保持連接狀態，您可以深入了解 [負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)。 當您使用 Azure 負載平衡器時，該文章可幫助您了解閒置連線行為。
