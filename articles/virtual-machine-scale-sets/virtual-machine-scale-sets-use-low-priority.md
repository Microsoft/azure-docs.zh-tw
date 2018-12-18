---
title: 建立使用低優先順序 VM 的 Azure 擴展集 (預覽) | Microsoft Docs
description: 了解如何建立使用低優先順序 VM 的 Azure 虛擬機器擴展集以節省成本
services: virtual-machine-scale-sets
documentationcenter: ''
author: mmccrory
manager: rajraj
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/01/2018
ms.author: memccror
ms.openlocfilehash: c0b4e3e0a924c1353f7732737670dee7ed45a62a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953866"
---
# <a name="low-priority-vms-on-scale-sets-preview"></a>擴展集中的低優先順序 VM (預覽)

在擴展集中使用低優先順序的 VM，可讓您以大幅降低的成本使用我們未運用的容量。 在任何時候，只要 Azure 又需要這些容量，Azure 基礎結構就會收回低優先順序 VM。 因此，低優先順序 VM 對於可因應中斷的工作負載來說很好用，例如批次處理作業、開發/測試環境、大量運算工作負載等。

可使用的未運用容量數量取決於大小、區域、時段等因素。 如果在擴展集上部署低優先順序 VM，當有可使用的容量時 Azure 便會配置 VM，但這些 VM 沒有服務等級協定 (SLA)。 低優先順序擴展集是部署在單一容錯網域中，且不保證高可用性。

## <a name="eviction-policy"></a>收回原則

在建立低優先順序擴展集時，您可以將收回原則設為*解除配置* (預設) 或*刪除*。 

*解除配置*原則會將已收回的虛擬機器移至已停止 (已解除配置) 狀態，讓您可重新部署收回的執行個體。 不過，不保證配置會成功。 已解除配置的 VM 會依擴展集執行個體配額計算，您將需要支付您的基礎磁碟費用。 

如果您想讓低優先順序擴展集中的 VM 在收回時一併刪除，可將收回原則設定為*刪除*。 收回原則設定為刪除的情況下，您可以藉由增加擴展集執行個體計數屬性來建立新的 VM。 Azure 會一同刪除已收回的 VM 與其基礎磁碟，因此您將不需要支付此儲存體的費用。 您也可以使用擴展集的自動調整功能，自動嘗試和補償收回的 VM，不過不保證配置會成功。 建議您只在收回原則設定為刪除時，才讓低優先順序擴展集使用自動調整功能，以節省磁碟成本並避免達到配額限制。 

> [!NOTE]
> 在預覽期間，您能夠使用 [Azure 入口網站](#use-the-azure-portal)和 [Azure Resource Manager 範本](#use-azure-resource-manager-templates)來設定收回原則。 

## <a name="deploying-low-priority-vms-on-scale-sets"></a>在擴展集上部署低優先順序 VM

若要在擴展集上部署低優先順序 VM，可將新的 Priority 旗標設為 Low。 擴展集中的所有 VM 都會設定為低優先順序。 若要建立使用低優先順序 VM 的擴展集，請使用下列其中一個方法︰
- [Azure 入口網站](#use-the-azure-portal)
- [Azure CLI](#use-the-azure-cli-20)
- [Azure PowerShell](#use-azure-powershell)
- [Azure 資源管理員範本](#use-azure-resource-manager-templates)

## <a name="use-the-azure-portal"></a>使用 Azure 入口網站

建立使用低優先順序虛擬機器之擴展集的程序，與[使用者入門文章](quick-create-portal.md)所述程序相同。 當您部署擴展集時，可以選擇設定低優先順序旗標和收回原則：![建立使用低優先順序 VM 的擴展集](media/virtual-machine-scale-sets-use-low-priority/vmss-low-priority-portal.png)

## <a name="use-the-azure-cli"></a>使用 Azure CLI

建立使用低優先順序 VM 之擴展集的程序，與[使用者入門文章](quick-create-cli.md)中所述的程序相同。 只要在 CLI 呼叫中新增 '--priority' 參數，並將它設定為 Low，如下列範例所示：

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Low
```

## <a name="use-azure-powershell"></a>使用 Azure PowerShell

建立使用低優先順序 VM 之擴展集的程序，與[使用者入門文章](quick-create-powershell.md)中所述的程序相同。
只要在 [New-AzureRmVmssConfig](/powershell/module/azurerm.compute/new-azurermvmssconfig) 中新增 '-priority' 參數，並將它設定為 Low，如下列範例所示：

```powershell
$vmssConfig = New-AzureRmVmssConfig `
    -Location "East US 2" `
    -SkuCapacity 2 `
    -SkuName "Standard_DS2" `
    -UpgradePolicyMode Automatic `
    -Priority "Low"
```

## <a name="use-azure-resource-manager-templates"></a>使用 Azure Resource Manager 範本

建立使用低優先順序 VM 之擴展集的程序，與 [Linux](quick-create-template-linux.md) 或 [Windows](quick-create-template-windows.md) 的使用者入門文章中所述的程序相同。 將 'priority' 屬性新增至範本中的 Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile 資源類型，並指定其值為 Low。 請務必使用 *2018-03-01* API 版本或更高版本。 

為了要將收回原則設定為刪除，請新增 'evictionPolicy' 參數，並將它設定為 delete (刪除)。

下列範例會在「美國中西部」建立名為 myScaleSet 的 Linux 低優先順序擴展集，並會在收回擴展集中的 VM 時刪除 VM：

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2018-03-01",
  "sku": {
    "name": "Standard_DS2_v2",
    "capacity": "2"
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
       "priority": "Low",
       "evictionPolicy": "delete",
       "storageProfile": {
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage"
        },
        "imageReference":  {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "16.04-LTS",
          "version": "latest"
        }
      },
      "osProfile": {
        "computerNamePrefix": "myvmss",
        "adminUsername": "azureuser",
        "adminPassword": "P@ssw0rd!"
      }
    }
  }
}
```
## <a name="faq"></a>常見問題集

### <a name="can-i-convert-existing-scale-sets-to-low-priority-scale-sets"></a>是否可以將現有的擴展集轉換成低優先順序的擴展集？
否，僅在建立時支援設定低優先順序旗標。

### <a name="can-i-create-a-scale-set-with-both-regular-vms-and-low-priority-vms"></a>我可以建立使用一般虛擬機器和低優先順序虛擬機器的擴展集嗎？
否，擴展集不支援一個以上的優先順序類型。

### <a name="how-is-quota-managed-for-low-priority-vms"></a>如何為低優先順序虛擬機器管理配額？
低優先順序的虛擬機器和一般虛擬機器共用相同的配額集區。 

### <a name="can-i-use-autoscale-with-low-priority-scale-sets"></a>我是否可以透過低優先順序擴展集使用自動調整？
是，您可以在您的低優先順序擴展集上設定自動調整規則。 如果您的虛擬機器收回，自動調整會嘗試建立新的低優先順序虛擬機器。 請記住，不保證容量足夠。 

### <a name="does-autoscale-work-with-both-eviction-policies-deallocate-and-delete"></a>自動調整是否可以同時使用收回原則 (解除配置和刪除)？
建議您在使用自動調整時，將您的收回原則設為刪除。 這是因為解除配置執行個體是按照擴展集的容量計數進行計算。 使用自動調整時，由於已解除配置收回的執行個體，可能會很快達到目標執行個體計數。 

## <a name="next-steps"></a>後續步驟
您已經建立使用低優先順序 VM 的擴展集，接下來可開始嘗試部署[使用低優先順序自動調整規模的範本](https://github.com/Azure/vm-scale-sets/tree/master/preview/lowpri)。

如需定價詳細資料，請參閱[虛擬機器擴展集定價網頁](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/)。