---
title: 教學課程 - Azure 中 Windows VM 的高可用性 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure PowerShell 在可用性設定組中部署高可用性的虛擬機器
documentationcenter: ''
services: virtual-machines-windows
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: tutorial
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: fb10390731310db7b87e9ad101531c4404acf21b
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783268"
---
# <a name="tutorial-create-and-deploy-highly-available-virtual-machines-with-azure-powershell"></a>教學課程：使用 Azure PowerShell 建立及部署高可用性的虛擬機器

在本教學課程中，您會學到如何使用稱為「可用性設定組」的功能，增加 Azure 虛擬機器解決方案的可用性和可靠性。 可用性設定組可確保您在 Azure 上部署的 VM 會分散到叢集中多個各自獨立的硬體節點。 這麼做可以確保當 Azure 發生硬體或軟體故障時，受到影響的只會是一部分的 VM 子集，您整體的解決方案則會維持可用且正常運作。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立可用性設定組
> * 在可用性設定組中建立 VM
> * 檢查可用的 VM 大小
> * 檢查 Azure Advisor

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="availability-set-overview"></a>可用性設定組概觀

可用性設定組是一種可在 Azure 中使用的邏輯群組功能，用以確保其中所放置的 VM 資源在部署到 Azure 資料中心時會彼此隔離。 Azure 可確保您在可用性設定組中所放置的 VM，會橫跨多部實體伺服器、計算機架、儲存體單位和網路交換器來執行。 如果硬體或 Azure 軟體發生故障時，只有一部分的 VM 子集會受到影響，整體的應用程式則會保持運作狀態，可供客戶繼續使用。 如果您想要建置可靠的雲端解決方案，就一定要使用可用性設定組功能。

請設想典型的 VM 型解決方案，在此解決方案中，您可能有 4 部前端 Web 伺服器，以及 2 部後端 VM。 在使用 Azure 時，建議您先定義兩個可用性設定組，再部署 VM︰一個可用性設定組用來放置 Web 層，另一個可用性設定組用來放置後端層。 當您建立新的 VM 時，您便可以將可用性設定組指定為 az vm create 命令的參數，Azure 會自動確保您在可用性設定組內所建立的 VM，會跨多個實體硬體資源來隔離。 如果其中一個用來執行 Web 伺服器或後端 VM 的實體硬體發生問題，Web 伺服器和後端 VM 的其他執行個體會繼續執行，因為它們位於不同硬體上。

如果您想要在 Azure 內部署可靠的 VM 架構解決方案，請使用可用性設定組。

## <a name="create-an-availability-set"></a>建立可用性設定組

您可以使用 [New-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/new-azurermavailabilityset) 建立可用性設定組。 在此範例中，我們將針對 *myResourceGroupAvailability* 資源群組中名為 *myAvailabilitySet* 的可用性設定組，同時將更新數目和容錯網域數目設為 *2*。

建立資源群組。

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroupAvailability -Location EastUS
```

使用 [New-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/new-azurermavailabilityset) 搭配 `-sku aligned` 參數建立受控的可用性設定組。

```azurepowershell-interactive
New-AzureRmAvailabilitySet `
   -Location "EastUS" `
   -Name "myAvailabilitySet" `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Sku aligned `
   -PlatformFaultDomainCount 2 `
   -PlatformUpdateDomainCount 2
```

## <a name="create-vms-inside-an-availability-set"></a>建立位於可用性設定組內的 VM
您必須將 VM 建立於可用性設定組內，才能確保 VM 會在硬體中正確地分散。 您無法在建立可用性設定組之後，將現有的 VM 加入至其中。 

位置中的硬體已分為多個更新網域和容錯網域。 **更新網域**是一組虛擬機器和可同時重新啟動的基礎實體硬體。 相同**容錯網域**中的 VM 會共用一般儲存體以及通用電源和網路交換器。 

您在透過 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM 時，會使用 `-AvailabilitySetName` 參數來指定可用性設定組的名稱。

首先，使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 設定 VM 的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在，請使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 在可用性設定組中建立兩個 VM。

```azurepowershell-interactive
for ($i=1; $i -le 2; $i++)
{
    New-AzureRmVm `
        -ResourceGroupName "myResourceGroupAvailability" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress$i" `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred
}
```

`-AsJob` 參數會以背景工作建立 VM，因此會傳回 PowerShell 提示。 您可以使用 `Job` Cmdlet 檢視背景作業的詳細資料。 建立及設定這兩部 VM 需要幾分鐘的時間。 完成後，您會有兩個分散於基礎硬體上的虛擬機器。 

如果您在入口網站中移至 [資源群組] > [myResourceGroupAvailability] > [myAvailabilitySet] 來查看可用性設定組，您應會看到 VM 分散在 2 個容錯和更新網域上。

![入口網站中的可用性設定組](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>檢查可用的 VM 大小 

您稍後可以將更多 VM 加入至可用性設定組，但是需要知道硬體上有哪些可用的 VM 大小。 針對可用性設定組，使用 [Get-AzureRMVMSize](/powershell/module/azurerm.compute/get-azurermvmsize) 來列出硬體叢集上所有可用的大小。

```azurepowershell-interactive
Get-AzureRmVMSize `
   -ResourceGroupName "myResourceGroupAvailability" `
   -AvailabilitySetName "myAvailabilitySet"
```

## <a name="check-azure-advisor"></a>檢查 Azure Advisor 

您也可以使用 Azure Advisor，以取得改善 VM 可用性方式的詳細資訊。 Azure Advisor 可協助您遵循最佳做法來最佳化 Azure 部署。 它可分析您的資源組態和使用量遙測，然後建議可協助您改善 Azure 資源的成本效益、效能、高可用性和安全性的解決方案。

登入 [Azure 入口網站](https://portal.azure.com)，選取 [所有服務]，然後輸入 **Advisor**。 Advisor 儀表板會顯示所選取訂用帳戶的個人化建議。 如需詳細資訊，請參閱[開始使用 Azure Advisor](../../advisor/advisor-get-started.md)。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立可用性設定組
> * 在可用性設定組中建立 VM
> * 檢查可用的 VM 大小
> * 檢查 Azure Advisor

請前進到下一個教學課程，以了解虛擬機器擴展集。

> [!div class="nextstepaction"]
> [建立 VM 擴展集](tutorial-create-vmss.md)


