---
title: 使用 Azure 網路監看員安全性群組檢視分析網路安全性 - REST API | Microsoft Docs
description: 本文會說明如何使用 PowerShell，利用安全性群組檢視分析虛擬機器的安全性。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: a2f418fe-f5d2-43ed-8dc3-df0ed2a4d4ac
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 0fd96f9bd3027568e81e9c56ddb095297c699683
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39089769"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-rest-api"></a>使用 REST API，利用安全性群組檢視分析虛擬機器的安全性

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [Azure CLI](network-watcher-security-group-view-cli.md)
> - [REST API](network-watcher-security-group-view-rest.md)

安全性群組檢視會傳回套用至虛擬機器之已設定且有效的網路安全性規則。 這項功能可用來稽核及診斷 VM 所設定的網路安全性群組和規則，以確保會正確允許或拒絕流量。 在本文中，我們會說明如何使用 REST API 來擷取有效且已套用至虛擬機器的安全性規則

## <a name="before-you-begin"></a>開始之前

在此案例中，您可以呼叫網路監看員 REST API 來取得的虛擬機器安全性群組檢視。 使用 ARMclient 透過 PowerShell 呼叫 REST API。 您可以在 chocolatey 的 [ARMClient on Chocolatey](https://chocolatey.org/packages/ARMClient) 上找到 ARMClient

此案例假設您已依照[建立網路監看員](network-watcher-create.md)中的步驟建立網路監看員。 此案例也假設已有具有有效虛擬機器的資源群組可供使用。

## <a name="scenario"></a>案例

本文涵蓋的案例會擷取有效且已套用指定虛擬機器的安全性規則。

## <a name="log-in-with-armclient"></a>使用 ARMClient 登入

```PowerShell
armclient login
```

## <a name="retrieve-a-virtual-machine"></a>擷取虛擬機器

執行下列程式碼以傳回虛擬機器。下列程式碼需要變數︰

- **subscriptionId** - 也可使用 **Get-AzureRMSubscription** Cmdlet 來擷取訂用帳戶識別碼。
- **resourceGroupName** - 包含虛擬機器的資源群組名稱。

```powershell
$subscriptionId = '<subscription id>'
$resourceGroupName = '<resource group name>'

armclient get https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Compute/virtualMachines?api-version=2015-05-01-preview
```

所需的資訊是回應中 `Microsoft.Compute/virtualMachines`類型下方的**識別碼**，如下列範例所示︰

```json
...,
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft
.Network/networkInterfaces/{nicName}"
            }
          ]
        },
        "provisioningState": "Succeeded"
      },
      "resources": [
        {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Com
pute/virtualMachines/{vmName}/extensions/CustomScriptExtension"
        }
      ],
      "type": "Microsoft.Compute/virtualMachines",
      "location": "westcentralus",
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute
/virtualMachines/{vmName}",
      "name": "{vmName}"
    }
  ]
}
```

## <a name="get-security-group-view-for-virtual-machine"></a>取得虛擬機器的安全性群組檢視

下列範例會要求目標虛擬機器的安全性群組檢視。 本範例的結果可用於比較依原始所定義的規則及安全性來尋找設定漂移。

```powershell
$subscriptionId = "<subscription id>"
$resourceGroupName = "<resource group name>"
$networkWatcherName = "<network watcher name>"
$targetUri = "<uri of target resource>" # Example: /subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.compute/virtualMachine/$vmName

$requestBody = @"
{
    'targetResourceId': '${targetUri}'

}
"@
armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/securityGroupView?api-version=2016-12-01" $requestBody -verbose
```

## <a name="view-the-response"></a>檢視回應

下列範例是前述命令中所傳回的回應。 結果顯示虛擬機器上所有有效且套用的安全性規則，並細分為 **NetworkInterfaceSecurityRules**、**DefaultSecurityRules**和 **EffectiveSecurityRules** 群組。

```json

{
  "networkInterfaces": [
    {
      "securityRuleAssociations": {
        "networkInterfaceAssociation": {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{nicName}",
          "securityRules": [
            {
              "name": "default-allow-rdp",
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/securityRules/default-allow-rdp",
              "etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
              "properties": {
                "provisioningState": "Succeeded",
                "protocol": "TCP",
                "sourcePortRange": "*",
                "destinationPortRange": "3389",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 1000,
                "direction": "Inbound"
              }
            }
          ]
        },
        "defaultSecurityRules": [
          {
            "name": "AllowVnetInBound",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/defaultSecurityRules/",
            "properties": {
              "provisioningState": "Succeeded",
              "description": "Allow inbound traffic from all VMs in VNET",
              "protocol": "*",
              "sourcePortRange": "*",
              "destinationPortRange": "*",
              "sourceAddressPrefix": "VirtualNetwork",
              "destinationAddressPrefix": "VirtualNetwork",
              "access": "Allow",
              "priority": 65000,
              "direction": "Inbound"
            }
          },
          ...
        ],
        "effectiveSecurityRules": [
          {
            "name": "DefaultOutboundDenyAll",
            "protocol": "All",
            "sourcePortRange": "0-65535",
            "destinationPortRange": "0-65535",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Deny",
            "priority": 65500,
            "direction": "Outbound"
          },
          ...
        ]
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

請瀏覽[使用網路監看員稽核網路安全性群組 (NSG)](network-watcher-security-group-view-powershell.md) 以了解如何自動驗證網路安全性群組。


