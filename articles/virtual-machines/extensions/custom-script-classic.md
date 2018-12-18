---
title: Windows VM 上的自訂指令碼擴充功能 | Microsoft Docs
description: 使用自訂指令碼擴充功能，在遠端 Windows VM 上執行 PowerShell 指令碼，將 Azure VM 組態工作自動化
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: ebb7340a-8f61-4d3c-a290-d7bf8de2d0bd
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 01/17/2017
ms.author: roiyz
ms.openlocfilehash: 8eb7822962988b02f09c2a2ea31b745ef01d5533
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451845"
---
# <a name="custom-script-extension-for-windows-using-the-classic-deployment-model"></a>使用傳統部署模型自訂適用於 Windows 的指令碼擴充功能

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有兩種： [Resource Manager](../../azure-resource-manager/resource-manager-deployment-model.md) 和傳統。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何[使用 Resource Manager 模型執行這些步驟](custom-script-windows.md)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../includes/virtual-machines-classic-portal.md)]

「自訂指令碼擴充功能」會在 Azure 虛擬機器上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。 「自訂指令碼擴充功能」會與 Azure Resource Manager 範本整合，您也可以使用 Azure CLI、PowerShell、Azure 入口網站或「Azure 虛擬機器 REST API」來執行它。

本文件詳細說明如何透過 Azure PowerShell 模組、Azure Resource Manager 範本使用自訂指令碼擴充功能，同時也詳細說明 Windows 系統上的疑難排解步驟。

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

可以對 Windows Server 2008 R2、2012、2012 R2 和 2016 版執行適用於 Windows 的自訂指令碼擴充功能。

### <a name="script-location"></a>指令碼位置

指令碼必須儲存在 Azure 儲存體或任何可透過有效 URL 存取的其他位置。

### <a name="internet-connectivity"></a>網際網路連線

適用於 Windows 的自訂指令碼擴充功能會要求目標虛擬機器連接到網際網路。 

## <a name="extension-schema"></a>擴充功能結構描述

下列 JSON 顯示自訂指令碼擴充功能的結構描述。 擴充功能需要指令碼位置 (Azure 儲存體或其他具有有效 URL 的位置)，以及可供執行的命令。 如果使用 Azure 儲存體做為指令碼來源，則需要 Azure 儲存體帳戶名稱和帳戶金鑰。 這些項目應被視為敏感性資料，並在擴充功能保護的設定組態中指定。 Azure VM 擴充功能保護的設定資料會經過加密，只會在目標虛擬機器上解密。

```json
{
    "name": "config-app",
    "type": "Microsoft.ClassicCompute/virtualMachines/extensions",
    "location": "[resourceGroup().location]",
    "apiVersion": "2015-06-01",
    "properties": {
        "publisher": "Microsoft.Compute",
        "extension": "CustomScriptExtension",
        "version": "1.8",
        "parameters": {
            "public": {
                "fileUris": "[myScriptLocation]"
            },
            "private": {
                "commandToExecute": "[myExecutionString]"
            }
        }
    }
}
```

### <a name="property-values"></a>屬性值

| 名稱 | 值 / 範例 |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.Compute |
| 擴充功能 | CustomScriptExtension |
| typeHandlerVersion | 1.8 |
| fileUris (例如) | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1 |
| commandToExecute (例如) | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 |

## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 上一節詳述的 JSON 結構描述可以用於 Azure Resource Manager 範本，以在 Azure Resource Manager 範本部署期間執行自訂指令碼擴充功能。 在 [GitHub](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows) 可以找到包含自訂指令碼擴充功能的範例範本。

## <a name="powershell-deployment"></a>PowerShell 部署

`Set-AzureVMCustomScriptExtension` 命令可以用來將自訂指令碼擴充功能新增至現有的虛擬機器。 如需詳細資訊，請參閱 [Set-AzureRmVMCustomScriptExtension ](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmcustomscriptextension)。

```powershell
# create vm object
$vm = Get-AzureVM -Name 2016clas -ServiceName 2016clas1313

# set extension
Set-AzureVMCustomScriptExtension -VM $vm -FileUri myFileUri -Run 'create-file.ps1'

# update vm
$vm | Update-AzureVM
```

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure PowerShell 模組，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請執行下列命令。

```powershell
Get-AzureVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

擴充功能執行輸出會記錄至目標虛擬機器上下列目錄中的檔案。

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

指令碼本身會下載到目標虛擬機器上的下列目錄。

```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads
```

### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
