---
title: 在 Azure 中的 Linux VM 上使用原則強制執行安全性 | Microsoft Docs
description: 如何將原則套用至 Azure Resource Manager Linux 虛擬機器
services: virtual-machines-linux
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 06778ab4-f8ff-4eed-ae10-26a276fc3faa
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/02/2017
ms.author: singhkay
ms.openlocfilehash: 04a172c40c24e4e0daaf17b495d2320f3a897a47
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975248"
---
# <a name="apply-policies-to-linux-vms-with-azure-resource-manager"></a>使用 Azure Resource Manager 將原則套用至 Linux VM
藉由使用原則，組織可以強制執行整個企業的各種慣例和規則。 強制執行所要的行為有助於降低風險，同時促進組織的成功。 我們會在本文中說明如何使用 Azure Resource Manager 原則來為您組織的虛擬機器定義所要的行為。

如需原則簡介，請參閱[什麼是 Azure 原則？](../../azure-policy/azure-policy-introduction.md)。

## <a name="permitted-virtual-machines"></a>允許的虛擬機器
若要確保您組織的虛擬機器與應用程式相容，您可以針對允許使用哪些作業系統設下限制。 您可以透過以下原則範例，允許只能建立 Ubuntu 14.04.2-LTS 虛擬機器。

```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "in": [
          "Microsoft.Compute/disks",
          "Microsoft.Compute/virtualMachines",
          "Microsoft.Compute/VirtualMachineScaleSets"
        ]
      },
      {
        "not": {
          "allOf": [
            {
              "field": "Microsoft.Compute/imagePublisher",
              "in": [
                "Canonical"
              ]
            },
            {
              "field": "Microsoft.Compute/imageOffer",
              "in": [
                "UbuntuServer"
              ]
            },
            {
              "field": "Microsoft.Compute/imageSku",
              "in": [
                "14.04.2-LTS"
              ]
            },
            {
              "field": "Microsoft.Compute/imageVersion",
              "in": [
                "latest"
              ]
            }
          ]
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

使用萬用字元修改上述原則，來允許任何 Ubuntu LTS 映像： 

```json
{
  "field": "Microsoft.Compute/virtualMachines/imageSku",
  "like": "*LTS"
}
```

如需原則欄位的相關資訊，請參閱[原則別名](../../governance/policy/concepts/definition-structure.md#aliases)。

## <a name="managed-disks"></a>受控磁碟

若要要求使用受控磁碟，請使用以下原則：

```json
{
  "if": {
    "anyOf": [
      {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/virtualMachines"
          },
          {
            "field": "Microsoft.Compute/virtualMachines/osDisk.uri",
            "exists": true
          }
        ]
      },
      {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/VirtualMachineScaleSets"
          },
          {
            "anyOf": [
              {
                "field": "Microsoft.Compute/VirtualMachineScaleSets/osDisk.vhdContainers",
                "exists": true
              },
              {
                "field": "Microsoft.Compute/VirtualMachineScaleSets/osdisk.imageUrl",
                "exists": true
              }
            ]
          }
        ]
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

## <a name="images-for-virtual-machines"></a>虛擬機器的映像

基於安全性理由，您可以要求只在環境中部署已核准的自訂映像。 您可以指定含有已核准映像的資源群組，或已核准的特定映像。

下列範例會從已核准的資源群組要求映像：

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "in": [
                    "Microsoft.Compute/virtualMachines",
                    "Microsoft.Compute/VirtualMachineScaleSets"
                ]
            },
            {
                "not": {
                    "field": "Microsoft.Compute/imageId",
                    "contains": "resourceGroups/CustomImage"
                }
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
} 
```

下列範例會指定已核准的映像識別碼：

```json
{
    "field": "Microsoft.Compute/imageId",
    "in": ["{imageId1}","{imageId2}"]
}
```

## <a name="virtual-machine-extensions"></a>虛擬機器延伸模組

您可能想要禁止使用某些類型的延伸模組。 例如，一個延伸模組可能與某些自訂虛擬機器映像不相容。 下列範例示範如何封鎖特定延伸模組。 它利用發行者和類型來判斷要封鎖哪個延伸模組。

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.Compute/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                "equals": "Microsoft.Compute"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/type",
                "equals": "{extension-type}"

      }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```


## <a name="next-steps"></a>後續步驟
* 定義原則規則 (如上述範例所示) 之後，您必須建立原則定義，並將它指派到某個範圍。 範圍可以是訂用帳戶、資源群組或資源。 若要指派原則，請參閱[使用 Azure 入口網站來指派和管理資源原則](../../azure-policy/assign-policy-definition.md)、[使用 PowerShell 來指派原則](../../azure-policy/assign-policy-definition-ps.md)或[使用 Azure CLI 來指派原則](../../azure-policy/assign-policy-definition-cli.md)。
* 如需資源原則簡介，請參閱[什麼是 Azure 原則？](../../azure-policy/azure-policy-introduction.md)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。
