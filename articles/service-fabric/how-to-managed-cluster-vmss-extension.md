---
title: '將虛擬機器擴展集延伸模組新增至 Service Fabric 受控叢集節點類型 (preview) '
description: 以下說明如何將虛擬機器擴展集擴充功能新增 Service Fabric 受控叢集節點類型
ms.topic: article
ms.date: 09/28/2020
ms.openlocfilehash: 64df4b82795f382e176d66dc61470296447b9e29
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98788073"
---
# <a name="add-a-virtual-machine-scale-set-extension-to-a-service-fabric-managed-cluster-node-type-preview"></a>將虛擬機器擴展集延伸模組新增至 Service Fabric 受控叢集節點類型 (preview) 

Service Fabric 受控叢集中的每個節點類型都是由虛擬機器擴展集支援。 這可讓您將 [虛擬機器擴展集延伸](../virtual-machines/extensions/overview.md) 模組新增至 Service Fabric 受控叢集節點類型。

您可以使用 [AzServiceFabricManagedNodeTypeVMExtension](/powershell/module/az.servicefabric/add-azservicefabricmanagednodetypevmextension) PowerShell 命令，將虛擬機器擴展集延伸模組新增至節點類型。

或者，您可以在 Azure Resource Manager 範本中，Service Fabric 受控叢集節點類型上的虛擬機器擴展集擴充功能，例如：

```json
{
    "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
    "apiVersion": "[variables('sfApiVersion')]",
    "name": "[concat(parameters('clusterName'), '/', parameters('nodeTypeName'))]",
    "dependsOn": [
        "[concat('Microsoft.ServiceFabric/managedclusters/', parameters('clusterName'))]"
    ],
    "location": "[resourceGroup().location]",
    "properties": {
        "isPrimary": true,
        "vmInstanceCount": 3,
        "dataDiskSizeGB": 100,
        "vmSize": "Standard_D2",
        "vmImagePublisher": "MicrosoftWindowsServer",
        "vmImageOffer": "WindowsServer",
        "vmImageSku": "2019-Datacenter",
        "vmImageVersion": "latest",
        "vmExtensions": [{
            "name": "ExtensionA",
            "properties": {
                "publisher": "ExtensionA.Publisher",
                "type": "KeyVaultForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                }
            }
        }]
    }
}
```

如需設定受管理叢集節點類型 Service Fabric 的詳細資訊，請參閱 [受控叢集節點類型](/azure/templates/microsoft.servicefabric/2020-01-01-preview/managedclusters/nodetypes)。

## <a name="next-steps"></a>後續步驟

若要深入了解 Service Fabric 受控叢集，請參閱：

> [!div class="nextstepaction"]
> [Service Fabric 受控叢集常見問題集](./faq-managed-cluster.md)
