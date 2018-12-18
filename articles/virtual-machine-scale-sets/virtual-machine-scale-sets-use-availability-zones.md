---
title: 建立使用可用性區域的 Azure 擴展集 | Microsoft Docs
description: 了解如何建立使用可用性區域的 Azure 虛擬機器擴展集，以提升系統中斷時的備援能力
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm
ms.devlang: na
ms.topic: article
ms.date: 08/08/2018
ms.author: cynthn
ms.openlocfilehash: 18d9a2dade271e61fa3db423da610a7f982aa47b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949667"
---
# <a name="create-a-virtual-machine-scale-set-that-uses-availability-zones"></a>建立使用可用性區域的虛擬機器擴展集

如要保護虛擬機器擴展集，使其免於資料中心層級的失敗，您可以建立跨可用性區域的擴展集。 支援可用性區域的 Azure 區域至少會有三個不同的區域，每個區域都有自己的獨立電源、網路和冷卻系統。 如需詳細資訊，請參閱[可用性區域概觀](../availability-zones/az-overview.md)。

## <a name="availability-considerations"></a>可用性考量

從 API 版本 *2017-12-01* 開始，當您將擴展集部署至一個或多個區域時，可以選擇使用「最大分配」或「靜態的 5 個容錯網域分配」來進行部署。 使用最大分配時，擴展集會盡量將 VM 分配至每個區域中的多個容錯網域。 您可以在每個區域中五個以上或以下的容錯網域中進行此分配。 若使用「靜態的 5 個容錯網域分配」，擴展集會確切地在每個區域的五個容錯網域中分配 VM。 如果擴展集無法在每個區域中找到五個不同的容錯網域，來滿足配置要求，則要求會失敗。

**我們建議您使用最大分配來為大部分工作負載進行部署**，因為此方法在大部分情況下能提供最佳分配。 如果您需要將複本分配到不同硬體的獨立單元，我們建議您在可用性區域之間進行分配，並在每個區域中使用最大分配。

使用最大分配時，無論您在多少個容錯網域間分配 VM，在擴展集 VM 執行個體檢視和執行個體中繼資料中只會看到一個容錯網域。 每個區域中的分配都是隱含的。

若要使用最大分配，請將 *platformFaultDomainCount* 設為 *1*。 若要使用靜態的五個容錯網域分配，請將 *platformFaultDomainCount* 設為 *5*。 在 API 版本 *2017-12-01* 中，單一區域和跨區域擴展集的 *platformFaultDomainCount* 預設值為 *1*。 目前，只有靜態的五個容錯網域分配可支援區域性擴展集。

### <a name="placement-groups"></a>放置群組

當您部署規模集時，也可以選擇使用每個可用性區域的單一[放置群組](./virtual-machine-scale-sets-placement-groups.md)進行部署，或者每個可用性區域使用多個進行部署。 針對區域擴展集，選項是在該區域中有單一放置群組，或者在該區域中有多個放置群組。 針對大部分的工作負載，我們建議您使用多個放置群組，如此便能有更大的擴展性。 在 API 版本 *2017-12-01* 中，擴展集預設為單一區域和跨區域擴展集可有多個放置群組，但針對區域性擴展集，其預設為單一放置群組。

> [!NOTE]
> 如果您使用最大分配，您必須使用多個放置群組。

### <a name="zone-balancing"></a>區域平衡

最後，針對跨多個區域部署的擴展集，您也可以選擇「最佳區域平衡」或「嚴格區域平衡」。 對擴展集而言，如果每個區域有相同數目的 VM，或是與所有其他區域中的 VM 數目差異為 +\\- 1 個，則此擴展集即為「平衡」。 例如︰

- 擴展集的區域 1 中有 2 個 VM，區域 2 中有 3 個 VM，而區域 3 中有 3 個 VM，則視為平衡。 只有一個區域的 VM 計數不同，且只比其他區域少 1 個。 
- 擴展集的區域 1 中有 1 個 VM，區域 2 中有 3 個 VM，而區域 3 中有 3 個 VM，則視為不平衡。 區域 1 的 VM 數目比區域 2 和 3 少 2 個。

也有可能發生擴展集中的 VM 建立成功，但這些 VM 上的延伸模組卻無法部署。 判斷擴展集是否平衡時，仍會將這些擴充失敗的 VM 計算在內。 例如，針對區域 1 中有 3 個 VM，區域 2 中有 3 個 VM，以及區域 3 中有 3 個 VM 的擴展集，即使區域 1 中的所有擴充都失敗，但區域 2 和區域 3 中的所有擴充都成功，則此擴展集將視為平衡。

若使用最佳區域平衡，擴展集會嘗試相應縮小或相應放大以維持平衡。 不過，如果基於某些原因而不能這麼做 (例如，如果某一個區域無法運作，擴展集不能在該區域中建立新的 VM)，該擴展集會允許暫時性的不平衡，以成功進行相應縮小或放大。在後續的相應放大嘗試上，該擴展集會將 VM 新增至其擴展集需要更多 VM 以取得平衡的區域。 同樣地，在後續的相應縮小嘗試上，該擴展集會將 VM 從其擴展集需要減少 VM 以取得平衡的區域中移除。 使用「嚴格區域平衡」，擴展集無法嘗試任何相應縮小或放大，如果這樣做會導致不平衡。

若要使用最佳區域平衡，請將 *zoneBalance* 設為 *false*。 在 API 版本 *2017-12-01* 中，這項設定是預設值。 若要使用嚴格區域平衡，請將 *zoneBalance* 設為 *true*。

## <a name="single-zone-and-zone-redundant-scale-sets"></a>單一區域和區域備援擴展集

在 Azure 區域中部署虛擬機器擴展集時，您可以選擇要使用一或多個可用性區域。

當您在單一區域中建立擴展集時，您可以控制要在哪個區域執行所有 VM 執行個體，且擴展集只會在該區域內進行管理並自動調整。 區域備援擴展集可讓您建立跨多個區域的單一擴展集。 當您建立 VM 執行個體時，系統預設會將它們平均分散到各個區域。 若其中一個區域發生中斷，擴展集不會為了增加容量而自動相應放大。 最佳做法是設定以 CPU 或記憶體使用量為基礎的自動調整規則。 自動調整規則可讓擴展集在該區域遺失 VM 執行個體時，於其餘運作中的區域相應增加新的執行個體，來作為回應。

若要使用可用性區域，您必須在[支援的 Azure 區域](../availability-zones/az-overview.md#regions-that-support-availability-zones)中建立擴展集。 您可以使用下列其中一個方法，來建立使用可用性區域的擴展集：

- [Azure 入口網站](#use-the-azure-portal)
- [Azure CLI](#use-the-azure-cli-20)
- [Azure PowerShell](#use-azure-powershell)
- [Azure 資源管理員範本](#use-azure-resource-manager-templates)

## <a name="use-the-azure-portal"></a>使用 Azure 入口網站

建立使用可用性區域之擴展集的流程，與[使用者入門文章](quick-create-portal.md)中所述的流程相同。 當您選取支援的 Azure 區域時，就可以在一或多個可用區域中建立擴展集，如下列範例所示：

![在單一可用性區域中建立擴展集](media/virtual-machine-scale-sets-use-availability-zones/vmss-az-portal.png)

擴展集和支援資源 (例如，Azure 負載平衡器和公用 IP 位址) 都會建立在您指定的單一區域中。

## <a name="use-the-azure-cli"></a>使用 Azure CLI

建立使用可用性區域之擴展集的流程，與[使用者入門文章](quick-create-cli.md)中所述的流程相同。 若要使用可用性區域，您必須在支援的 Azure 區域中建立擴展集。

將 `--zones` 參數新增至 [az vmss create](/cli/azure/vmss#az_vmss_create) 命令，並指定要使用的區域 (例如區域 1、2 或 3)。 下列範例會在區域 1 中建立名為 myScaleSet 的單一區域擴展集：

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --zones 1
```

如需單一區域擴展集和網路資源的完整範例，請參閱[這個 CLI 指令碼範例](https://github.com/Azure/azure-docs-cli-python-samples/blob/master/virtual-machine-scale-sets/create-single-availability-zone/create-single-availability-zone.sh.)

### <a name="zone-redundant-scale-set"></a>區域備援擴展集

若要建立區域備援擴展集，請使用標準 SKU 公用 IP 位址和負載平衡器。 為了增強備援能力，標準 SKU 會建立區域備援網路資源。 如需詳細資訊，請參閱 [Azure Load Balancer Standard 概觀](../load-balancer/load-balancer-standard-overview.md)和[標準 Load Balancer 和可用性區域](../load-balancer/load-balancer-standard-availability-zones.md)。

若要建立區域備援擴展集，請使用 `--zones` 參數來指定多個區域。 下列範例會跨區域 1、2、3 建立名為 myScaleSet 的區域備援擴展集：

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --zones 1 2 3
```

在您指定的區域中建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。 如需區域備援擴展集和網路資源的完整範例，請參閱[這個 CLI 指令碼範例](https://github.com/Azure/azure-docs-cli-python-samples/blob/master/virtual-machine-scale-sets/create-zone-redundant-scale-set/create-zone-redundant-scale-set.sh)

## <a name="use-azure-powershell"></a>使用 Azure PowerShell

若要使用可用性區域，您必須在支援的 Azure 區域中建立擴展集。 將 `-Zone` 參數新增至 [New-AzureRmVmssConfig](/powershell/module/azurerm.compute/new-azurermvmssconfig) 命令，並指定要使用的區域 (例如區域 1、2 或 3)。

下列範例會在「美國東部 2」區域 *1* 中建立名為 *myScaleSet* 的單一區域擴展集。 系統會自動建立虛擬網路、公用 IP 位址和負載平衡器的 Azure 網路資源。 出現提示時，請為擴展集中的 VM 執行個體提供適當的系統管理認證：

```powershell
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS2" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicy "Automatic" `
  -Zone "1"
```

### <a name="zone-redundant-scale-set"></a>區域備援擴展集

若要建立區域備援擴展集，請使用 `-Zone` 參數來指定多個區域。 下列範例會跨「美國東部 2」區域 *1、2、3* 建立名為 *myScaleSet* 的區域備援擴展集。 系統會自動建立虛擬網路、公用 IP 位址和負載平衡器的區域備援 Azure 網路資源。 出現提示時，請為擴展集中的 VM 執行個體提供適當的系統管理認證：

```powershell
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS2" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicy "Automatic" `
  -Zone "1", "2", "3"
```

## <a name="use-azure-resource-manager-templates"></a>使用 Azure 資源管理員範本

建立使用可用性區域之擴展集的流程，與 [Linux](quick-create-template-linux.md) 或 [Windows](quick-create-template-windows.md) 的使用者入門文章中所述的流程相同。 若要使用可用性區域，您必須在支援的 Azure 區域中建立擴展集。 將 `zones` 屬性新增至範本中的 Microsoft.Compute/virtualMachineScaleSets 資源類型，並指定要使用的區域 (例如區域 1、2 或 3)。

下列範例會在「美國東部 2」區域 1 中建立名為 myScaleSet 的 Linux 單一區域擴展集：

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2017-12-01",
  "zones": ["1"],
  "sku": {
    "name": "Standard_A1",
    "capacity": "2"
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
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

如需單一區域擴展集和網路資源的完整範例，請參閱[這個 Resource Manager 範本範例](https://github.com/Azure/vm-scale-sets/blob/master/preview/zones/singlezone.json)

### <a name="zone-redundant-scale-set"></a>區域備援擴展集

若要建立區域備援擴展集，請在 Microsoft.Compute/virtualMachineScaleSets 資源類型的 `zones` 屬性中指定多個值。 下列範例會跨「美國東部 2」區域 1、2、3 建立名為 myScaleSet 的區域備援擴展集：

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2017-12-01",
  "zones": [
        "1",
        "2",
        "3"
      ]
}
```

如果您建立的是公用 IP 位址或負載平衡器，請指定「"sku": { "name": "Standard" }"」屬性，以建立區域備援網路資源。 您還必須建立網路安全性群組和規則，以便允許所有流量。 如需詳細資訊，請參閱 [Azure Load Balancer Standard 概觀](../load-balancer/load-balancer-standard-overview.md)和[標準 Load Balancer 和可用性區域](../load-balancer/load-balancer-standard-availability-zones.md)。

如需區域備援擴展集和網路資源的完整範例，請參閱[這個 Resource Manager 範本範例](https://github.com/Azure/vm-scale-sets/blob/master/preview/zones/multizone.json)

## <a name="next-steps"></a>後續步驟

現在您已在可用性區域中建立擴展集，接下來您可以了解如何[在虛擬機器擴展集上部署應用程式](tutorial-install-apps-cli.md)或[搭配虛擬機器擴展集使用自動調整](tutorial-autoscale-cli.md)。
