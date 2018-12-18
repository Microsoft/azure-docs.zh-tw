---
title: 搭配虛擬機器擴展集使用期望狀態組態 | Microsoft Docs
description: 搭配 Azure DSC 擴充功能使用虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: zjalexander
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager
keywords: ''
ms.assetid: c8f047b5-0e6c-4ef3-8a47-f1b284d32942
ms.service: virtual-machine-scale-sets
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 04/05/2017
ms.author: zachal
ms.openlocfilehash: a68a5f31952d636c054b66dc0bb6ec0579cd7192
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30908998"
---
# <a name="using-virtual-machine-scale-sets-with-the-azure-dsc-extension"></a>搭配 Azure DSC 擴充功能使用虛擬機器擴展集
[虛擬機器擴展集](virtual-machine-scale-sets-overview.md)可以搭配 [Azure 期望狀態設定 (DSC)](../virtual-machines/windows/extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 擴充功能處理常式來使用。 虛擬機器擴展集提供一個部署及管理大量虛擬機器的方式，可以因應負載情況來彈性地相應縮小和放大。 當 VM 上線時，可使用 DSC 來設定它們，因為它們正在執行生產環境的軟體。

## <a name="differences-between-deploying-to-virtual-machines-and-virtual-machine-scale-sets"></a>部署至虛擬機器與部署至虛擬機器擴展集之間的差異
適用於虛擬機器擴展集的基礎範本結構與單一 VM 有些微不同。 具體來說，單一 VM 會在 "virtualMachines" 節點下部署擴充功能。 有一個類型為 "extensions" 且已將 DSC 加入範本的項目

```
"resources": [
          {
              "name": "Microsoft.Powershell.DSC",
              "type": "extensions",
              "location": "[resourceGroup().location]",
              "apiVersion": "2015-06-15",
              "dependsOn": [
                  "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
              ],
              "tags": {
                  "displayName": "dscExtension"
              },
              "properties": {
                  "publisher": "Microsoft.Powershell",
                  "type": "DSC",
                  "typeHandlerVersion": "2.20",
                  "autoUpgradeMinorVersion": false,
                  "forceUpdateTag": "[parameters('dscExtensionUpdateTagVersion')]",
                  "settings": {
                      "configuration": {
                          "url": "[concat(parameters('_artifactsLocation'), '/', variables('dscExtensionArchiveFolder'), '/', variables('dscExtensionArchiveFileName'))]",
                          "script": "DscExtension.ps1",
                          "function": "Main"
                      },
                      "configurationArguments": {
                          "nodeName": "[variables('vmName')]"
                      }
                  },
                  "protectedSettings": {
                      "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                  }
              }
          }
      ]
```

虛擬機器擴展集節點具有 "properties" 區段以及 "VirtualMachineProfile"、"extensionProfile" 屬性。 已將 DSC 加入至 "extensions" 下方

```
"extensionProfile": {
            "extensions": [
                {
                    "name": "Microsoft.Powershell.DSC",
                    "properties": {
                        "publisher": "Microsoft.Powershell",
                        "type": "DSC",
                        "typeHandlerVersion": "2.20",
                        "autoUpgradeMinorVersion": false,
                        "forceUpdateTag": "[parameters('DscExtensionUpdateTagVersion')]",
                        "settings": {
                            "configuration": {
                                "url": "[concat(parameters('_artifactsLocation'), '/', variables('DscExtensionArchiveFolder'), '/', variables('DscExtensionArchiveFileName'))]",
                                "script": "DscExtension.ps1",
                                "function": "Main"
                            },
                            "configurationArguments": {
                                "nodeName": "localhost"
                            }
                        },
                        "protectedSettings": {
                            "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                        }
                    }
                }
            ]
```

## <a name="behavior-for-a-virtual-machine-scale-set"></a>虛擬機器擴展集的行為
虛擬機器擴展集的行為與單一 VM 的行為完全相同。 建立新的 VM 時，會使用 DSC 擴充功能自動進行佈建。 如果擴充功能需要較新版本的 WMF，VM 會在上線之前重新開機。 一旦上線之後，它就會下載 DSC 組態 .zip，並在 VM 上佈建它。 您可以在 [Azure DSC 擴充功能概觀](../virtual-machines/windows/extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到更多詳細資料。

## <a name="next-steps"></a>後續步驟
查看 [適用於 DSC 擴充功能的 Azure Resource Manager 範本](../virtual-machines/windows/extensions-dsc-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

了解 [DSC 擴充功能如何安全地處理認證](../virtual-machines/windows/extensions-dsc-credentials.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 

如需有關 Azure DSC 擴充功能處理常式的詳細資訊，請參閱 [Azure 期望狀態組態擴充功能處理常式簡介](../virtual-machines/windows/extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 

如需有關 PowerShell DSC 的詳細資訊，請 [瀏覽 PowerShell 文件中心](https://msdn.microsoft.com/powershell/dsc/overview)。 

