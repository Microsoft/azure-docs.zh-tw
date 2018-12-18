---
title: 修改 Azure 虛擬機器擴展集 | Microsoft Docs
description: 深入了解如何使用 REST API、Azure PowerShell 及 Azure CLI，修改和更新 Azure 虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: e229664e-ee4e-4f12-9d2e-a4f456989e5d
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2018
ms.author: negat
ms.openlocfilehash: 628d407869d24f466b5a7c056d51d76217e29798
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996650"
---
# <a name="modify-a-virtual-machine-scale-set"></a>修改虛擬機器擴展集
在應用程式的整個生命週期中，您可能需要修改或更新您的虛擬機器擴展集。 這些更新可能包括如何更新擴展集的組態，或者變更應用程式組態。 本文說明如何使用 REST API、Azure PowerShell 或 Azure CLI 來修改現有的擴展集。

## <a name="fundamental-concepts"></a>基本概念

### <a name="the-scale-set-model"></a>擴展集模型
擴展集具有「擴展集模型」，可擷取擴展集整體的「預期」狀態。 若要查詢擴展集的模型，您可以使用 

- REST API 執行 [compute/virtualmachinescalesets/get](/rest/api/compute/virtualmachinescalesets/get)，如下所示：

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)：

    ```powershell
    Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
    ```

- Azure CLI 執行 [az vmss show](/cli/azure/vmss#az_vmss_show)：

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet
    ```

- 您也可以使用 [resources.azure.com](https://resources.azure.com) 或語言特定 [Azure SDK](https://azure.microsoft.com/downloads/)。

確切的輸出呈現內容取決於您提供給命令的選項。 下列範例會示範 Azure CLI 的扼要範例輸出：

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "overprovision": true,
  "plan": null,
  "singlePlacementGroup": true,
  "sku": {
    "additionalProperties": {},
    "capacity": 1,
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

這些屬性會整體套用至擴展集。


### <a name="the-scale-set-instance-view"></a>擴展集執行個體檢視
擴展集也具有「擴展集執行個體檢視」，可擷取擴展集整體的目前「執行階段」狀態。 若要查詢擴展集的執行個體檢視，您可以使用：

- REST API 執行 [compute/virtualmachinescalesets/getinstanceview](/rest/api/compute/virtualmachinescalesets/getinstanceview)，如下所示：

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)：

    ```powershell
    Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceView
    ```

- Azure CLI 執行 [az vmss get-instance-view](/cli/azure/vmss#az_vmss_get_instance_view)：

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
    ```

- 您也可以使用 [resources.azure.com](https://resources.azure.com) 或語言特定 [Azure SDK](https://azure.microsoft.com/downloads/)

確切的輸出呈現內容取決於您提供給命令的選項。 下列範例會示範 Azure CLI 的扼要範例輸出：

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
{
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": null,
      "time": "{time}"
    }
  ],
  "virtualMachine": {
    "additionalProperties": {},
    "statusesSummary": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "count": 1
      }
    ]
  }
}
```

這些屬性會提供擴展集內虛擬機器目前執行階段狀態的摘要，例如套用至擴展集之擴充功能的狀態。


### <a name="the-scale-set-vm-model-view"></a>擴展集 VM 模型檢視
就像擴展集有模型檢視一樣，擴展集內的每個 VM 也有自己的模型檢視。 若要查詢擴展集的模型檢視，您可以使用：

- REST API 執行 [compute/virtualmachinescalesetvms/get](/rest/api/compute/virtualmachinescalesetvms/get)，如下所示：

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Get-AzureRmVmssVm](/powershell/module/azurerm.compute/get-azurermvmssvm)：

    ```powershell
    Get-AzureRmVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Azure CLI 執行 [az vmss show](/cli/azure/vmss#az_vmss_show)：

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- 您也可以使用 [resources.azure.com](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/)。

確切的輸出呈現內容取決於您提供給命令的選項。 下列範例會示範 Azure CLI 的扼要範例輸出：

```azurecli
$ az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "name": "{name}",
  "sku": {
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

這些屬性會描述虛擬機器本身的組態，而非擴展集整體的組態。 例如，擴展集模型有 `overprovision` 作為屬性，而擴展集內虛擬機器的模型則沒有。 之所以會有這個差異，是因為過度佈建是適用於擴展集整體的屬性，而不適用於擴展集內個別的虛擬機器 (如需有關過度佈建的詳細資訊，請參閱[擴展集的設計考量](virtual-machine-scale-sets-design-overview.md#overprovisioning))。


### <a name="the-scale-set-vm-instance-view"></a>擴展集 VM 執行個體檢視
就像擴展集有執行個體檢視一樣，擴展集內的每個 VM 也有自己的執行個體檢視。 若要查詢擴展集的執行個體檢視，您可以使用：

- REST API 執行 [ompute/virtualmachinescalesetvms/getinstanceview](/rest/api/compute/virtualmachinescalesetvms/getinstanceview)，如下所示：

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Get-AzureRmVmssVm](/powershell/module/azurerm.compute/get-azurermvmssvm)：

    ```powershell
    Get-AzureRmVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -InstanceView
    ```

- Azure CLI 執行 [az vmss get-instance-view](/cli/azure/vmss#az_vmss_get_instance_view)

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- 您也可以使用 [resources.azure.com](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/)

確切的輸出呈現內容取決於您提供給命令的選項。 下列範例會示範 Azure CLI 的扼要範例輸出：

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
{
  "additionalProperties": {
    "osName": "ubuntu",
    "osVersion": "16.04"
  },
  "disks": [
    {
      "name": "{name}",
      "statuses": [
        {
          "additionalProperties": {},
          "code": "ProvisioningState/succeeded",
          "displayStatus": "Provisioning succeeded",
          "time": "{time}"
        }
      ]
    }
  ],
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "time": "{time}"
    },
    {
      "additionalProperties": {},
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ],
  "vmAgent": {
    "statuses": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready",
        "level": "Info",
        "message": "Guest Agent is running",
        "time": "{time}"
      }
    ],
    "vmAgentVersion": "{version}"
  },
}
```

這些屬性會描述虛擬機器本身目前執行階段狀態的摘要，包括套用至擴展集的所有擴充功能。


## <a name="how-to-update-global-scale-set-properties"></a>如何更新全域擴展集屬性
若要更新全域擴展集屬性，您必須更新擴展集模型中的屬性。 您可以透過下列方式執行此操作：

- REST API 執行 [compute/virtualmachinescalesets/createorupdate](/rest/api/compute/virtualmachinescalesets/createorupdate)，如下所示：

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- 您可以使用來自 REST API 的屬性來部署 Resource Manager 範本，以更新全域擴展集屬性。

- Azure PowerShell 搭配 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss)：

    ```powershell
    Update-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -VirtualMachineScaleSet {scaleSetConfigPowershellObject}
    ```

- Azure CLI 執行 [az vmss update](/cli/azure/vmss#az_vmss_update)：
    - 若要修改屬性：

        ```azurecli
        az vmss update --set {propertyPath}={value}
        ```

    - 若要將物件新增至擴展集內的清單屬性： 

        ```azurecli
        az vmss update --add {propertyPath} {JSONObjectToAdd}
        ```

    - 若要從擴展集內的清單屬性移除物件： 

        ```azurecli
        az vmss update --remove {propertyPath} {indexToRemove}
        ```

    - 如果您先前已使用 `az vmss create` 命令來部署擴展集，則可以再次執行 `az vmss create` 命令來更新擴展集。 請確保 `az vmss create` 命令中的所有屬性都與之前相同，除了您想要修改的屬性之外。

- 您也可以使用 [resources.azure.com](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/)。

更新擴展集模型之後，新的設定就會套用至擴展集內新建立的所有 VM。 不過，擴展集內現有 VM 的模型仍然必須藉由最新的整體擴展集模型來更新至最新狀態。 每個 VM 的模型中都有一個名為 `latestModelApplied` 的布林值屬性，此屬性會指出 VM 是否已藉由最新的整體擴展集模型更新至最新狀態 (`true` 表示 VM 已藉由最新的模型更新至最新狀態)。


## <a name="how-to-bring-vms-up-to-date-with-the-latest-scale-set-model"></a>如何藉由最新的擴展集模型將 VM 更新至最新狀態
擴展集具有「升級原則」，可決定藉由最新擴展集模型將 VM 更新至最新狀態的方式。 升級原則的三個模式為：

- **自動** - 在此模式下，擴展集不保證虛擬機器的關閉順序。 擴展集可能同時關閉所有 VM。 
- **輪流** - 在此模式下，擴展集會分批進行更新，批次之間會有選擇性的暫停時間。
- **手動** - 在此模式下，當您更新擴展集模型時，現有的虛擬機器將不受影響。
 
若要更新現有的 VM，您必須「手動升級」每個現有的 VM。 您可以透過下列方式執行此手動升級：

- REST API 執行 [compute/virtualmachinescalesets/updateinstances](/rest/api/compute/virtualmachinescalesets/updateinstances)，如下所示：

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/manualupgrade?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Update-AzureRmVmssInstance](/powershell/module/azurerm.compute/update-azurermvmssinstance)：
    
    ```powershell
    Update-AzureRmVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Azure CLI 執行 [az vmss update-instances](/cli/azure/vmss#az_vmss_update_instances)

    ```azurecli
    az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
    ```

- 您也可以使用語言特定 [Azure SDK](https://azure.microsoft.com/downloads/)。

>[!NOTE]
> Service Fabric 叢集只能使用「自動」模式，但處理更新的方式不同。 如需詳細資訊，請參閱 [Service Fabric 應用程式升級](../service-fabric/service-fabric-application-upgrade.md)。

有一種全域擴展集屬性修改並不遵守升級原則。 對於擴展集作業系統設定檔 (例如系統管理員使用者名稱和密碼) 的變更，只能在 API 版本 2017-12-01 或更新版本中進行。 這些變更只會套用至變更擴展集模型之後建立的 VM。 若要將現有的 VM 更新至最新狀態，您必須為每個現有的 VM 執行「重新安裝映像」作業。 您可以透過下列方式執行這個重新安裝映像作業：

- REST API 執行 [compute/virtualmachinescalesets/reimage](/rest/api/compute/virtualmachinescalesets/reimage)，如下所示：

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/reimage?api-version={apiVersion}
    ```

- Azure PowerShell 搭配 [Set-AzureRmVmssVm](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmssvm)：

    ```powershell
    Set-AzureRmVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -Reimage
    ```

- Azure CLI 執行 [az vmss reimage](https://docs.microsoft.com/cli/azure/vmss#az_vmss_reimage)：

    ```azurecli
    az vmss reimage --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- 您也可以使用語言特定 [Azure SDK](https://azure.microsoft.com/downloads/)。


## <a name="properties-with-restrictions-on-modification"></a>具有修改限制的屬性

### <a name="create-time-properties"></a>建立階段屬性
有些屬性只能在建立擴展集時設定。 這些屬性包括：

- 可用性區域
- 映像參考發行者
- 映像參考供應項目
- 受控 OS 磁碟儲存體帳戶類型

### <a name="properties-that-can-only-be-changed-based-on-the-current-value"></a>只能根據目前值變更的屬性
有些屬性可以變更，但除非是根據目前的值。 這些屬性包括：

- **singlePlacementGroup** - 如果 singlePlacementGroup 為 true，則可以修改成 false。 不過，如果 singlePlacementGroup 為 false，則**無法**修改成 true。
- **subnet** - 只要擴展集的原始子網路與新子網路位於相同的虛擬網路中，便可以修改擴展集的子網路。

### <a name="properties-that-require-deallocation-to-change"></a>必須解除配置才能變更的屬性
有些屬性只有在已將擴展集內 VM 解除配置的情況下，才能變更成特定值。 這些屬性包括：

- **SKU name (SKU 名稱)** - 如果擴展集目前所在的硬體不支援新的虛擬機器 SKU，您必須先將擴展集內的虛擬機器解除配置，才能修改 SKU 名稱。 如需詳細資訊，請參閱[如何調整 Azure 虛擬機器的大小](../virtual-machines/windows/resize-vm.md)。


## <a name="vm-specific-updates"></a>VM 特定的更新
某些修改可以套用至特定 VM，而無法套用至全域擴展集屬性。 目前唯一支援的虛擬機器特定更新是將資料磁碟連結至擴展集內的虛擬機器，或將資料磁碟與那些虛擬機器中斷連結。 這項功能處於預覽狀態。 如需詳細資訊請，參閱[預覽文件](https://github.com/Azure/vm-scale-sets/tree/master/preview/disk) \(英文\)。


## <a name="scenarios"></a>案例

### <a name="application-updates"></a>應用程式更新
如果透過擴充功能將應用程式部署至擴展集，則更新擴充組態時，會造成應用程式根據升級原則進行更新。 例如，如果您有要在自訂指令碼擴充功能中執行的新版指令碼，則可以更新 fileUris 屬性以指向新的指令碼。 在某些情況下，即使擴充組態未變更 (例如，您已更新指令碼，但未變更指令碼的 URI)，您也可能會想要強制更新。 在這些情況下，您可以修改 forceUpdateTag 來強制更新。 Azure 平台不會解譯這個屬性。 如果您變更此值，並不會影響擴充功能的執行方式。 變更只會強制擴充功能重新執行。 如需有關 forceUpdateTag 的詳細資訊，請參閱[擴充功能的 REST API 文件](/rest/api/compute/virtualmachineextensions/createorupdate)。 請注意，forceUpdateTag 可以搭配所有擴充功能使用，而不只是搭配自訂指令碼擴充功能。

透過自訂映像來部署應用程式也是常見的做法。 在下一節中將會探討此案例。

### <a name="os-updates"></a>OS 更新
如果您使用 Azure 平台映像，您可以修改 imageReference (如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/createorupdate)) 來更新映像。

>[!NOTE]
> 使用平台映像時，通常會指定 "latest" 作為映像參考版本。 當您建立、相應放大及重新安裝映像時，會使用最新可用版本來建立虛擬機器。 不過，這**並不**意謂著作業系統映像會隨著時間在新映像版本發行時自動更新。 獨立的功能目前為預覽狀態，可提供自動的作業系統升級。 如需詳細資訊，請參閱 [自動作業系統升級文件](virtual-machine-scale-sets-automatic-upgrade.md)。

如果您使用自訂映像，您可以更新 imageReference 識別碼 (如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/createorupdate)) 來更新映像。

## <a name="examples"></a>範例

### <a name="update-the-os-image-for-your-scale-set"></a>更新擴展集的 OS 映像
您可能有執行舊版 Ubuntu LTS 16.04 的擴展集。 您想要更新為較新版本的 Ubuntu LTS 16.04 (例如，版本 16.04.201801090)。 映像參考版本屬性不是清單的一部分，因此您可以直接使用下列其中一個命令來修改這些屬性：

- Azure PowerShell 搭配 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss)，如下所示：

    ```powershell
    Update-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -ImageReferenceVersion 16.04.201801090
    ```

- Azure CLI 執行 [az vmss update](/cli/azure/vmss#az_vmss_update_instances)：

    ```azurecli
    az vmss update --resource-group myResourceGroup --name myScaleSet --set virtualMachineProfile.storageProfile.imageReference.version=16.04.201801090
    ```


### <a name="update-the-load-balancer-for-your-scale-set"></a>更新擴展集的負載平衡器
假設您有一個含有 Azure Load Balancer 的擴展集，而您想要以「Azure 應用程式閘道」取代 Azure Load Balancer。 擴展集的負載平衡器和應用程式閘道屬性是清單的一部分，因此您可以使用命令來移除或新增清單元素，而不需直接修改屬性：

- Azure Powershell：

    ```powershell
    # Get the current model of the scale set and store it in a local PowerShell object named $vmss
    $vmss=Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
    
    # Create a local PowerShell object for the new desired IP configuration, which includes the referencerence to the application gateway
    $ipconf = New-AzureRmVmssIPConfig "myNic" -ApplicationGatewayBackendAddressPoolsId /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendAddressPoolName} -SubnetId $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Subnet.Id –Name $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Name
    
    # Replace the existing IP configuration in the local PowerShell object (which contains the references to the current Azure Load Balancer) with the new IP configuration
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0] = $ipconf
    
    # Update the model of the scale set with the new configuration in the local PowerShell object
    Update-AzureRmVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -virtualMachineScaleSet $vmss
    ```

- Azure CLI：

    ```azurecli
    # Remove the load balancer backend pool from the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools 0
    
    # Remove the load balancer backend pool from the scale set model; only necessary if you have NAT pools configured on the scale set
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools 0
    
    # Add the application gateway backend pool to the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].ApplicationGatewayBackendAddressPools '{"id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendPoolName}"}'
    ```

>[!NOTE]
> 這些命令會假設擴展集上只有一個 IP 設定和負載平衡器。 如果有多個，您可能需要使用清單索引而不是 0。


## <a name="next-steps"></a>後續步驟
您也可以使用 [Azure CLI](virtual-machine-scale-sets-manage-cli.md) 或 [Azure PowerShell](virtual-machine-scale-sets-manage-powershell.md)，在擴展集上執行常見管理工作。
