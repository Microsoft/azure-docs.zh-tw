---
title: Azure Resource Manager 範本中的虛擬機器 | Microsoft Azure
description: 進一步了解在 Azure Resource Manager 範本中定義虛擬機器資源的方式。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: f63ab5cc-45b8-43aa-a4e7-69dc42adbb99
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: cynthn
ms.openlocfilehash: eb88501c5daf0b79d22f4407a372c4606a173db1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987691"
---
# <a name="virtual-machines-in-an-azure-resource-manager-template"></a>Azure Resource Manager 範本中的虛擬機器

本文描述套用至虛擬機器的 Azure Resource Manager 範本的各個部分。 本文不會說明建立虛擬機器的完整範本；您需要儲存體帳戶、網路介面、公用 IP 位址與虛擬網路的資源定義。 如需有關如何將這些資源一起定義的詳細資訊，請參閱 [Resource Manager 範本逐步解說](../../azure-resource-manager/resource-manager-template-walkthrough.md)。

有許多[組件庫中的範本](https://azure.microsoft.com/documentation/templates/?term=VM)包含 VM 資源。 此處並未說明可以包含在範本中的所有項目。

此範例顯示用來建立指定數目之 VM 範本的一般資源區段︰

```json
"resources": [
  { 
    "apiVersion": "2016-04-30-preview", 
    "type": "Microsoft.Compute/virtualMachines", 
    "name": "[concat('myVM', copyindex())]", 
    "location": "[resourceGroup().location]",
    "copy": {
      "name": "virtualMachineLoop", 
      "count": "[parameters('numberOfInstances')]"
    },
    "dependsOn": [
      "[concat('Microsoft.Network/networkInterfaces/myNIC', copyindex())]" 
    ], 
    "properties": { 
      "hardwareProfile": { 
        "vmSize": "Standard_DS1" 
      }, 
      "osProfile": { 
        "computername": "[concat('myVM', copyindex())]", 
        "adminUsername": "[parameters('adminUsername')]", 
        "adminPassword": "[parameters('adminPassword')]" 
      }, 
      "storageProfile": { 
        "imageReference": { 
          "publisher": "MicrosoftWindowsServer", 
          "offer": "WindowsServer", 
          "sku": "2012-R2-Datacenter", 
          "version": "latest" 
        }, 
        "osDisk": { 
          "name": "[concat('myOSDisk', copyindex())]",
          "caching": "ReadWrite", 
          "createOption": "FromImage" 
        },
        "dataDisks": [
          {
            "name": "[concat('myDataDisk', copyindex())]",
            "diskSizeGB": "100",
            "lun": 0,
            "createOption": "Empty"
          }
        ] 
      }, 
      "networkProfile": { 
        "networkInterfaces": [ 
          { 
            "id": "[resourceId('Microsoft.Network/networkInterfaces',
              concat('myNIC', copyindex()))]" 
          } 
        ] 
      },
      "diagnosticsProfile": {
        "bootDiagnostics": {
          "enabled": "true",
          "storageUri": "[concat('https://', variables('storageName'), '.blob.core.windows.net')]"
        }
      } 
    },
    "resources": [ 
      { 
        "name": "Microsoft.Insights.VMDiagnosticsSettings", 
        "type": "extensions", 
        "location": "[resourceGroup().location]", 
        "apiVersion": "2016-03-30", 
        "dependsOn": [ 
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
        ], 
        "properties": { 
          "publisher": "Microsoft.Azure.Diagnostics", 
          "type": "IaaSDiagnostics", 
          "typeHandlerVersion": "1.5", 
          "autoUpgradeMinorVersion": true, 
          "settings": { 
            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
            variables('wadmetricsresourceid'), 
            concat('myVM', copyindex()),
            variables('wadcfgxend')))]", 
            "storageAccount": "[variables('storageName')]" 
          }, 
          "protectedSettings": { 
            "storageAccountName": "[variables('storageName')]", 
            "storageAccountKey": "[listkeys(variables('accountid'), 
              '2015-06-15').key1]", 
            "storageAccountEndPoint": "https://core.windows.net" 
          } 
        } 
      },
      {
        "name": "MyCustomScriptExtension",
        "type": "extensions",
        "apiVersion": "2016-03-30",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
        ],
        "properties": {
          "publisher": "Microsoft.Compute",
          "type": "CustomScriptExtension",
          "typeHandlerVersion": "1.7",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "fileUris": [
              "[concat('https://', variables('storageName'),
                '.blob.core.windows.net/customscripts/start.ps1')]" 
            ],
            "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
          }
        }
      } 
    ]
  } 
]
``` 

> [!NOTE] 
>此範例仰賴先前建立的儲存體帳戶。 您可以透過從範本部署它來建立儲存體帳戶。 此範例也會仰賴網路介面和其會在範本中定義的相依資源。 這些資源不會顯示在範例中。
>
>

## <a name="api-version"></a>API 版本

當您使用範本部署資源時，必須指定要使用的 API 版本。 此範例會使用這個 apiVersion 項目顯示虛擬機器資源︰

```
"apiVersion": "2016-04-30-preview",
```

您在範本中指定的 API 版本會影響您可以在範本中定義的屬性。 一般而言，您應在建立範本時選取最新的 API 版本。 對於現有的範本，您可以決定是否繼續使用舊的 API 版本，或更新您的範本以便最新版本利用新功能。

使用這些機會來取得最新的 API 版本︰

- REST API - [列出所有資源提供者](https://docs.microsoft.com/rest/api/resources/providers#Providers_List)
- PowerShell - [Get-AzureRmResourceProvider](/powershell/module/azurerm.resources/get-azurermresourceprovider)
- Azure CLI - [az provider show](https://docs.microsoft.com/cli/azure/provider#az_provider_show)

## <a name="parameters-and-variables"></a>參數和變數

[參數](../../resource-group-authoring-templates.md)方便您執行它時指定範本的值。 此參數章節用於範例中︰

```        
"parameters": {
  "adminUsername": { "type": "string" },
  "adminPassword": { "type": "securestring" },
  "numberOfInstances": { "type": "int" }
},
```

當您部署範例範本時，您要輸入每個 VM 上系統管理員帳戶的名稱和密碼值，以及要建立的 VM 數目。 您可以選擇指定使用範本管理之不同檔案中的參數值，或當系統提示時提供值。

[變數](../../resource-group-authoring-templates.md)方便您設定範本中整個重複使用或會隨著時間而改變的值。 此變數章節用於範例中︰

```
"variables": { 
  "storageName": "mystore1",
  "accountid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name,
  '/providers/','Microsoft.Storage/storageAccounts/', variables('storageName'))]", 
  "wadlogs": "<WadCfg> 
    <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> 
      <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> 
      <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > 
        <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" />
      </WindowsEventLog>", 
  "wadperfcounters": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\">
      <PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\">
        <annotation displayName=\"Threads\" locale=\"en-us\"/>
      </PerformanceCounterConfiguration>
    </PerformanceCounters>", 
  "wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters'), 
    '<Metrics resourceId=\"')]", 
  "wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name , 
    '/providers/', 'Microsoft.Compute/virtualMachines/')]", 
  "wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/>
    <MetricAggregation scheduledTransferPeriod=\"PT1M\"/>
    </Metrics></DiagnosticMonitorConfiguration>
    </WadCfg>"
}, 
```

當您部署範例範本時，變數值會用於名稱與先前建立之儲存體帳戶的識別碼。 變數也用來提供診斷延伸模組的設定。 使用[建立 Azure Resource Manager 範本的最佳作法](../../resource-manager-template-best-practices.md)可協助您決定要如何組織您範本中的變數與參數。

## <a name="resource-loops"></a>資源迴圈

您的應用程式需要一部以上的虛擬機器時，您可以在範本中使用複製項目。 此選擇性項目會透過建立您指定為參數的 VM 數目來執行迴圈︰

```
"copy": {
  "name": "virtualMachineLoop", 
  "count": "[parameters('numberOfInstances')]"
},
```

另外，在範例中請注意指定某些資源的值時會使用迴圈索引。 例如，如果您輸入三個執行個體計數，則作業系統磁碟的名稱為 myOSDisk1、myOSDisk2 和 myOSDisk3：

```
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
}
```

> [!NOTE] 
>此範例會使用虛擬機器的受控磁碟。
>
>

請記住，在範本中建立一個資源的迴圈，可能會在建立或存取其他資源時要求您使用迴圈。 例如，多個 VM 無法使用相同的網路介面，因此如果您的範本透過建立三個 VM 來執行迴圈，則它也必須透過建立三個網路介面來執行迴圈。 將網路介面指派至 VM 時，迴圈索引會用來識別它︰

```
"networkInterfaces": [ { 
  "id": "[resourceId('Microsoft.Network/networkInterfaces',
    concat('myNIC', copyindex()))]" 
} ]
```

## <a name="dependencies"></a>相依性

大部分的資源相依於其他資源才能正確地運作。 虛擬機器必須與虛擬網路相關聯，且需要網路介面才能執行它。 [ependsOn](../../resource-group-define-dependencies.md) 項目用來確定網路介面在 VM 建立之前已備妥可供使用：

```
"dependsOn": [
  "[concat('Microsoft.Network/networkInterfaces/', 'myNIC', copyindex())]" 
],
```

Resource Manager 會以平行方式部署任何不依存於另一個要部署資源的資源。 設定相依性時請謹慎，因為您可能會指定非必要相依性而不小心降低您的部署。 相依性可以鏈結多個資源。 例如，網路介面取決於公用 IP 位址和虛擬網路資源。

如何知道是否需要相依性？ 看看您在範本中設定的值。 如果虛擬機器資源定義點中的項目指向在相同範本中部署的另一個資源，則您需要相依性。 例如，範例虛擬機器會定義網路設定檔︰

```
"networkProfile": { 
  "networkInterfaces": [ { 
    "id": "[resourceId('Microsoft.Network/networkInterfaces',
      concat('myNIC', copyindex())]" 
  } ] 
},
```

若要設定這個屬性，網路介面必須存在。 因此，您需要相依性。 您在另一個資源 (父系) 內定義一項資源 (子系) 時，也需要設定相依性。 例如，診斷設定和自訂指令碼延伸模組都定義為虛擬機器的子資源。 在虛擬機器存在之後，它們才能建立。 因此，這兩個資源都會標示為相依於虛擬機器。

## <a name="profiles"></a>設定檔

定義虛擬機器資源時，會使用數個設定檔項目。 某些是必要的而有些則是選擇性。 例如，hardwareProfile、osProfile、storageProfile 和 networkProfile 元素是必要的，但 diagnosticsProfile 為選擇性。 這些設定檔會定義設定，例如︰
   
- [大小](sizes.md)
- [名稱](/azure/architecture/best-practices/naming-conventions)和認證
- 磁碟和[作業系統設定](cli-ps-findimage.md)
- [網路介面](../../virtual-network/virtual-network-deploy-multinic-classic-ps.md) 
- 開機診斷

## <a name="disks-and-images"></a>磁碟和映像
   
在 Azure 中，VHD 檔案可以代表[磁碟或映像](about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 當 VHD 檔案中的作業系統特製化為特定 VM 時，它就是磁碟。 當 VHD 檔案中的作業系統一般化為用來建立許多 VM 時，它就是映像。   
    
### <a name="create-new-virtual-machines-and-new-disks-from-a-platform-image"></a>從平台映像建立新的虛擬機器和新的磁碟

當您建立 VM 時，您必須決定要使用哪個作業系統。 ImageReference 項目用來定義新 VM 的作業系統。 此範例示範適用於 Windows Server 作業系統的定義︰

```
"imageReference": { 
  "publisher": "MicrosoftWindowsServer", 
  "offer": "WindowsServer", 
  "sku": "2012-R2-Datacenter", 
  "version": "latest" 
},
```

如果您想要建立 Linux 作業系統，您可以使用此定義︰

```
"imageReference": {
  "publisher": "Canonical",
  "offer": "UbuntuServer",
  "sku": "14.04.2-LTS",
  "version": "latest"
},
```

作業系統磁碟的組態設定會使用 OsDisk 項目指派。 此範例會定義新的受控磁碟，其快取模式設定為 [ReadWrite]，而此磁碟會從[平台映像](cli-ps-findimage.md)建立：

```
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
},
```

### <a name="create-new-virtual-machines-from-existing-managed-disks"></a>從現有受控磁碟建立新的虛擬機器

如果您想要從現有磁碟建立虛擬機器，移除 imageReference 和 osProfile 項目，並定義這些磁碟的設定︰

```
"osDisk": { 
  "osType": "Windows",
  "managedDisk": { 
    "id": "[resourceId('Microsoft.Compute/disks', [concat('myOSDisk', copyindex())])]" 
  }, 
  "caching": "ReadWrite",
  "createOption": "Attach" 
},
```

### <a name="create-new-virtual-machines-from-a-managed-image"></a>從受控映像建立新的虛擬機器

如果您想要從受控映像建立虛擬機器，請變更 imageReference 元素並定義這些磁碟設定︰

```
"storageProfile": { 
  "imageReference": {
    "id": "[resourceId('Microsoft.Compute/images', 'myImage')]"
  },
  "osDisk": { 
    "name": "[concat('myOSDisk', copyindex())]",
    "osType": "Windows",
    "caching": "ReadWrite", 
    "createOption": "FromImage" 
  }
},
```

### <a name="attach-data-disks"></a>連接資料磁碟

您可以選擇性地將資料磁碟新增至 VM。 [磁碟數量](sizes.md)取決於您使用的作業系統磁碟大小。 將 VM 大小設定為 Standard_DS1_v2，則可以新增到它們的資料磁碟數目上限為 2。 在範例中，有一個受控資料磁碟會新增至每部 VM：

```
"dataDisks": [
  {
    "name": "[concat('myDataDisk', copyindex())]",
    "diskSizeGB": "100",
    "lun": 0, 
    "caching": "ReadWrite",
    "createOption": "Empty"
  }
],
```

## <a name="extensions"></a>延伸模組

雖然[延伸模組](extensions-features.md)是不同的資源，它們會緊密繫結至 VM。 延伸模組可新增為 VM 的子資源或不同的資源。 範例會示範要新增至 VM 的[診斷延伸模組](extensions-diagnostics-template.md)：

```
{ 
  "name": "Microsoft.Insights.VMDiagnosticsSettings", 
  "type": "extensions", 
  "location": "[resourceGroup().location]", 
  "apiVersion": "2016-03-30", 
  "dependsOn": [ 
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
  ], 
  "properties": { 
    "publisher": "Microsoft.Azure.Diagnostics", 
    "type": "IaaSDiagnostics", 
    "typeHandlerVersion": "1.5", 
    "autoUpgradeMinorVersion": true, 
    "settings": { 
      "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
      variables('wadmetricsresourceid'), 
      concat('myVM', copyindex()),
      variables('wadcfgxend')))]", 
      "storageAccount": "[variables('storageName')]" 
    }, 
    "protectedSettings": { 
      "storageAccountName": "[variables('storageName')]", 
      "storageAccountKey": "[listkeys(variables('accountid'), 
        '2015-06-15').key1]", 
      "storageAccountEndPoint": "https://core.windows.net" 
    } 
  } 
},
```

此延伸模組資源會使用 storageName 變數和診斷變數來提供值。 如果您想要變更此延伸模組所收集的資料，可以將更多效能計數器新增至 wadperfcounters 變數。 您也可以選擇將診斷資料放入與 VM 磁碟所儲存位置不同的儲存體帳戶。

有許多您可以在 VM 上安裝的延伸模組，而最有用的可能是[自訂指令碼延伸模組](extensions-customscript.md)。 在範例中，名為 start.ps1 的 PowerShell 指令碼第一次啟動時會在每個 VM 上執行︰

```
{
  "name": "MyCustomScriptExtension",
  "type": "extensions",
  "apiVersion": "2016-03-30",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.7",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "[concat('https://', variables('storageName'),
          '.blob.core.windows.net/customscripts/start.ps1')]" 
      ],
      "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
    }
  }
}
```

Start.ps1 指令碼可以完成許多設定工作。 例如，範例中新增至 VM 的資料磁碟未初始化；您可以使用自訂指令碼將它們初始化。 如果您有多個啟動工作要執行時，可以使用 start.ps1 檔案來呼叫 Azure 儲存體中的其他 PowerShell 指令碼。 範例會使用 PowerShell，但您可以使用任何您所使用的作業系統上可用的指令碼方法。

您可以從入口網站中的延伸模組設定看到已安裝延伸模組的狀態︰

![取得延伸模組狀態](./media/template-description/virtual-machines-show-extensions.png)

您也可以使用 **Get-AzureRmVMExtension** PowerShell 命令、**vm extension get** Azure CLI 命令或 **Get extension information** REST API 取得延伸模組資訊。

## <a name="deployments"></a>部署

當您部署範本時，Azure 會追蹤您部署為群組的資源，並自動指派此部署群組的名稱。 部署的名稱與範本的名稱相同。

如果您想知道部署中的資源狀態，可以使用 Azure 入口網站中的資源群組刀鋒視窗︰

![取得部署資訊](./media/template-description/virtual-machines-deployment-info.png)
    
使用相同的範本來建立資源或更新現有的資源並不是問題。 當您使用命令來部署範本時，有機會說您想要使用的[模式](../../resource-group-template-deploy.md)。 模式可以設定為 [完成] 或 [累加]。 預設值是執行累加式更新。 使用 [完成] 模式請小心，因為您可能會不小心刪除資源。 當您將模式設定為 [完成] 時，Resource Manager 就會刪除不在範本中的資源群組之任何資源。

## <a name="next-steps"></a>後續步驟

- 使用[編寫 Azure Resource Manager 範本](../../resource-group-authoring-templates.md)來建立專屬範本。
- 部署您使用[利用 Resource Manager 範本搭配 Azure PowerShell 建立 Windows 虛擬機器](ps-template.md)建立的範本。
- 檢閱[使用 Azure PowerShell 模組來建立和管理 Windows VM](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，以了解如何管理您建立的 VM。
