---
title: 包含檔案
description: 包含檔案
services: storage
author: jboeshart
ms.service: storage
ms.topic: include
ms.date: 06/05/2018
ms.author: jaboes
ms.custom: include file
ms.openlocfilehash: b2561f4b1b5ef27f389114c85f0646b968f7765e
ms.sourcegitcommit: 301855e018cfa1984198e045872539f04ce0e707
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36269556"
---
# <a name="using-managed-disks-in-azure-resource-manager-templates"></a>在 Azure Resource Manager 範本中使用受控磁碟

本文逐步解說當使用 Azure Resource Manager 範本佈建虛擬機器時，受控與非受控磁碟之間的差異。 這些範例有助於您將使用非受控磁碟的現有範本，更新為使用受控磁碟。 如需參考，我們會使用 [101-vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows) \(英文\) 範本作為指南。 如果您想要直接進行比較，您可以看到使用[受控磁碟](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows/azuredeploy.json) \(英文\) 的範本以及使用[非受控磁碟](https://github.com/Azure/azure-quickstart-templates/tree/93b5f72a9857ea9ea43e87d2373bf1b4f724c6aa/101-vm-simple-windows/azuredeploy.json) \(英文\) 的先前版本。

## <a name="unmanaged-disks-template-formatting"></a>非受控磁碟範本格式

一開始，讓我們先看一下如何部署非受控磁碟。 建立非受控磁碟時，您需要有一個儲存體帳戶來保存 VHD 檔案。 您可以建立新儲存體帳戶，或使用現有儲存體帳戶。 此文章說明如何建立新儲存體帳戶。 請在資源區塊中建立一個儲存體帳戶資源，如下所示。

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2016-01-01",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage",
    "properties": {}
}
```

在虛擬機器物件的儲存體帳戶中新增相依性，以確保在虛擬機器建立前先建立相依性。 在 `storageProfile` 區段中，指定 VHD 位置的完整 URI，它會參照儲存體帳戶，而且 OS 磁碟與所有資料磁碟都需要它。

```json
{
    "apiVersion": "2015-06-15",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "name": "osdisk",
                "vhd": {
                    "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "name": "datadisk1",
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "vhd": {
                        "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/datadisk1.vhd')]"
                    },
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

## <a name="managed-disks-template-formatting"></a>受控磁碟範本格式

有了 Azure 受控磁碟，磁碟會變成最上層資源，且不再需要使用者建立儲存體帳戶。 受控磁碟是在 `2016-04-30-preview` API 版本中首次公開，在後續的 API 版本中皆可取得，而且現在它們是預設的磁碟類型。 以下幾節會逐步解說預設設定，並詳細說明如何進一步自訂您的磁碟。

> [!NOTE]
> 建議使用 `2016-04-30-preview` 以後的 API 版本，因為 `2016-04-30-preview` 和 `2017-03-30` 之間有重大變更。
>
>

### <a name="default-managed-disk-settings"></a>預設的受控磁碟設定

若要建立使用受控磁碟的 VM，您不再需要建立儲存體帳戶資源，而且可以更新虛擬機器資源，如下所示。 請特別注意，`apiVersion` 會反映出 `2017-03-30` 和 `osDisk` 和 `dataDisks` 不再參照 VHD 的特定 URI。 若部署而不指定其他屬性，磁碟將會使用 [標準 LRS 儲存體](../articles/storage/common/storage-redundancy.md) 。 若未指定名稱，它的 OS 磁碟格式為 `<VMName>_OsDisk_1_<randomstring>`，而每個資料磁碟的格式為 `<VMName>_disk<#>_<randomstring>`。 預設已停用 Azure 磁碟加密；OS 磁碟的快取為 [讀取/寫入]，資料磁碟的快取則為 [無]。 您可能會注意到，在下列範例中仍有儲存體帳戶相依性，但這只適用於儲存體的診斷，而且磁碟儲存體並不需要相依性。

```json
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="using-a-top-level-managed-disk-resource"></a>使用最上層受控磁碟資源

作為在虛擬機器物件中指定磁碟組態的替代方式，您可以建立最上層磁碟資源，並在建立虛擬機器時連結它。 例如，您可以建立如下所示的磁碟資源，用來作為資料磁碟。

```json
{
    "type": "Microsoft.Compute/disks",
    "name": "[concat(variables('vmName'),'-datadisk1')]",
    "apiVersion": "2017-03-30",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "properties": {
        "creationData": {
            "createOption": "Empty"
        },
        "diskSizeGB": 1023
    }
}
```

在 VM 物件中，參照這個要連結的磁碟物件。 指定 `managedDisk` 屬性中所建立受控磁碟的資源識別碼，以允許在建立 VM 時連結磁碟。 VM 資源的 `apiVersion` 設為 `2017-03-30`。 在磁碟資源上新增了相依性，以確保在建立 VM 之前成功建立相依性。 

```json
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
        "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "lun": 0,
                    "name": "[concat(variables('vmName'),'-datadisk1')]",
                    "createOption": "attach",
                    "managedDisk": {
                        "id": "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
                    }
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="create-managed-availability-sets-with-vms-using-managed-disks"></a>為使用受控磁碟的 VM 建立受控可用性設定組

若要為使用受控磁碟的 VM 建立受控可用性設定組，請將 `sku` 物件新增至可用性設定組資源，並將 `name` 屬性設定為 `Aligned`。 這個屬性可確保每個 VM 的磁碟彼此之間都充分隔離，以避免發生單一失敗點。 另請注意，可用性設定組資源的 `apiVersion` 是設定成 `2017-03-30`。

```json
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/availabilitySets",
    "location": "[resourceGroup().location]",
    "name": "[variables('avSetName')]",
    "properties": {
        "PlatformUpdateDomainCount": 3,
        "PlatformFaultDomainCount": 2
    },
    "sku": {
        "name": "Aligned"
    }
}
```

### <a name="standard-ssd-disks"></a>標準 SSD 磁碟

以下是在 Resource Manager 範本中建立標準 SSD 磁碟所需的參數：

* Microsoft.Compute 的 apiVersion 必須設定為 `2018-04-01` (或更新版本)
* 將 managedDisk.storageAccountType 指定為 `StandardSSD_LRS`

下列範例示範使用標準 SSD 磁碟的虛擬機器 properties.storageProfile.osDisk 區段：

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

如需如何使用範本建立標準 SSD 磁碟的完整範本範例，請參閱[從具有標準 SSD 資料磁碟的 Windows 映像建立虛擬機器](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/)。

### <a name="additional-scenarios-and-customizations"></a>其他案例和自訂

若要尋找有關 REST API 規格的完整資訊，請檢閱[建立受控磁碟 REST API 文件](/rest/api/manageddisks/disks/disks-create-or-update)。 您會找到其他案例，以及可透過範本部署提交至 API 之預設及可接受的值。 

## <a name="next-steps"></a>後續步驟

* 如需使用受控磁碟的完整範本，請瀏覽下列「Azure 快速入門存放庫」連結。
    * [使用受控磁碟的 Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows) \(英文\)
    * [使用受控磁碟的 Linux VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux) \(英文\)
    * [受控磁碟範本的完整清單](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md) \(英文\)
* 瀏覽 [Azure 受控磁碟概觀](../articles/virtual-machines/windows/managed-disks-overview.md)文件，深入了解受控磁碟。
* 瀏覽 [Microsoft.Compute/virtualMachines 範本參考](/azure/templates/microsoft.compute/virtualmachines)文件，檢閱虛擬機器資源的範本參考文件。
* 瀏覽 [Microsoft.Compute/disks 範本參考](/azure/templates/microsoft.compute/disks)文件，檢閱磁碟資源的範本參考文件。
* 如需如何在 Azure 虛擬機器擴展集內使用受控磁碟的相關資訊，請瀏覽[使用資料磁碟搭配擴展集](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks)文件。
