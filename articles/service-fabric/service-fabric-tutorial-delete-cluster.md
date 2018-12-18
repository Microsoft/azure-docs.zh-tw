---
title: 刪除 Azure 中的 Service Fabric 叢集 | Microsoft Docs
description: 在本教學課程中，您會了解如何刪除 Azure 裝載的 Service Fabric 叢集及其所有資源。 您可以刪除包含叢集的資源群組，或選擇性地刪除資源。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/26/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 70c5fa5de627b69623b1cce6929615f4e99e2a05
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47410796"
---
# <a name="tutorial-remove-a-service-fabric-cluster-running-in-azure"></a>教學課程：移除 Azure 中執行的 Service Fabric 叢集

本教學課程是系列中的第四部分，示範如何刪除 Azure 中執行的 Service Fabric 叢集。 若要完全刪除 Service Fabric 叢集，您也必須刪除叢集所使用的資源。 做法有兩種︰刪除叢集所在的資源群組 (這會刪除叢集資源和資源群組中的所有其他資源)，或明確刪除叢集資源和其相關聯資源 (而不是資源群組中的其他資源)。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 刪除資源群組及其所有資源
> * 選擇性地刪除資源群組中的資源

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 在 Azure 上，使用範本建立安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)或 [Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
> * [將叢集相應縮小或相應放大](service-fabric-tutorial-scale-cluster.md)
> * [升級叢集的執行階段](service-fabric-tutorial-upgrade-cluster.md)
> * 刪除叢集

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* 安裝 [Azure PowerShell 模組 4.1 版或更新版本](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)或 [Azure CLI](/cli/azure/install-azure-cli)。
* 在 Azure 上，建立安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)或 [Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)

## <a name="delete-the-resource-group-containing-the-service-fabric-cluster"></a>刪除包含 Service Fabric 叢集的資源群組
刪除叢集及其取用之所有資源的最簡單方式，就是刪除資源群組。

登入 Azure 並選取您要移除叢集的訂用帳戶識別碼。  您可以登入[Azure 入口網站](http://portal.azure.com)找到您的訂用帳戶識別碼。 使用 [Remove-AzureRMResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) Cmdlet 或 [az group delete](/cli/azure/group?view=azure-cli-latest#az_group_delete) 命令刪除資源群組和所有叢集資源。

```powershell
Connect-AzureRmAccount
Set-AzureRmContext -SubscriptionId <guid>
$groupname = "sfclustertutorialgroup"
Remove-AzureRmResourceGroup -Name $groupname -Force
```

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="selectively-delete-the-cluster-resource-and-the-associated-resources"></a>選擇性地刪除叢集資源和相關聯的資源
如果您的資源群組只有您想要刪除之 Service Fabric 叢集相關的資源，則很容易就可以刪除整個資源群組。 如果您想要選擇性地刪除資源群組中的資源，並確保資源未與叢集相關聯，請遵循這些步驟。

列出資源群組中的資源：

```powershell
Connect-AzureRmAccount
Set-AzureRmContext -SubscriptionId <guid>
$groupname = "sfclustertutorialgroup"
Get-AzureRmResource -ResourceGroupName $groupname | ft
```

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="sfclustertutorialgroup"
az resource list --resource-group $ResourceGroupName
```

對於您想要刪除的每個資源，執行下列指令碼︰

```powershell
Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "<Resource Type>" -ResourceGroupName $groupname -Force
```

```azurecli
az resource delete --name "<name of the Resource>" --resource-type "<Resource Type>" --resource-group $ResourceGroupName
```

若要刪除叢集資源，請執行下列指令碼︰

```powershell
Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "Microsoft.ServiceFabric/clusters" -ResourceGroupName $groupname -Force
```

```azurecli
az resource delete --name "<name of the Resource>" --resource-type "Microsoft.ServiceFabric/clusters" --resource-group $ResourceGroupName
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 刪除資源群組及其所有資源
> * 選擇性地刪除資源群組中的資源

現在您已完成本教學課程，請嘗試下列作業：
* 了解如何使用 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 檢查及管理 Service Fabric 叢集。
* 了解如何修補叢集節點的 [Windows 作業系統](service-fabric-patch-orchestration-application.md)或 [Linux 作業系統](service-fabric-patch-orchestration-application-linux.md)。
* 了解如何彙總並收集 [Windows 叢集](service-fabric-diagnostics-event-aggregation-wad.md)或 [Linux 叢集](service-fabric-diagnostics-event-aggregation-lad.md)的事件及[設定 Log Analytics](service-fabric-diagnostics-oms-setup.md) 以監視叢集事件。