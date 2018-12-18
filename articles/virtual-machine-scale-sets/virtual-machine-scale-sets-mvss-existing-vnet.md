---
title: 在 Azure 擴展集範本中參考現有的虛擬網路 | Microsoft Docs
description: 了解如何將虛擬網路新增到現有的「Azure 虛擬機器擴展集」範本
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 76ac7fd7-2e05-4762-88ca-3b499e87906e
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2017
ms.author: negat
ms.openlocfilehash: eb35975de5864e129f97b614a61487456dd972ef
ms.sourcegitcommit: f46cbcff710f590aebe437c6dd459452ddf0af09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2017
ms.locfileid: "26782366"
---
# <a name="add-reference-to-an-existing-virtual-network-in-an-azure-scale-set-template"></a>在 Azure 擴展集範本中新增對現有虛擬網路的參考

本文說明如何修改[最基本的可行擴展集範本](./virtual-machine-scale-sets-mvss-start.md)以便將其部署至現有虛擬網路，而非建立新的範本。

## <a name="change-the-template-definition"></a>變更範本定義

您可以在[這裡](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)看到最基本的可行擴展集範本，並在[這裡](https://raw.githubusercontent.com/gatneil/mvss/existing-vnet/azuredeploy.json)看到用於將擴展集部署至現有虛擬網路的範本。 讓我們逐步檢查用來建立此範本 (`git diff minimum-viable-scale-set existing-vnet`) 的差異：

首先，新增 `subnetId` 參數。 這個字串會傳遞至擴展集組態，讓擴展集可識別要將虛擬機器部署到的預建子網路。 這個字串的格式必須是︰`/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Network/virtualNetworks/<virtual-network-name>/subnets/<subnet-name>`。 例如，若要將擴展集部署到名稱為 `myvnet`、子網路為 `mysubnet`、資源群組為 `myrg` 和訂用帳戶為 `00000000-0000-0000-0000-000000000000` 的現有虛擬網路，subnetId 會是：`/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myrg/providers/Microsoft.Network/virtualNetworks/myvnet/subnets/mysubnet`。

```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "subnetId": {
+      "type": "string"
     }
   },
```

接下來，從 `resources` 陣列刪除虛擬網路資源，因為您使用現有虛擬網路，因此不需要部署新的資源。

```diff
   "variables": {},
   "resources": [
-    {
-      "type": "Microsoft.Network/virtualNetworks",
-      "name": "myVnet",
-      "location": "[resourceGroup().location]",
-      "apiVersion": "2016-12-01",
-      "properties": {
-        "addressSpace": {
-          "addressPrefixes": [
-            "10.0.0.0/16"
-          ]
-        },
-        "subnets": [
-          {
-            "name": "mySubnet",
-            "properties": {
-              "addressPrefix": "10.0.0.0/16"
-            }
-          }
-        ]
-      }
-    },
```

虛擬網路在部署範本之前就已存在，因此不需要指定從擴展集到虛擬網路的 dependsOn 子句。 刪除下列幾行：

```diff
     {
       "type": "Microsoft.Compute/virtualMachineScaleSets",
       "name": "myScaleSet",
       "location": "[resourceGroup().location]",
       "apiVersion": "2016-04-30-preview",
-      "dependsOn": [
-        "Microsoft.Network/virtualNetworks/myVnet"
-      ],
       "sku": {
         "name": "Standard_A1",
         "capacity": 2
```

最後，傳入使用者所設定的 `subnetId` 參數 (而不是使用 `resourceId` 來取得相同部署中的 vnet 識別碼，這是最基本的可行擴展集範本的作法)。

```diff
                       "name": "myIpConfig",
                       "properties": {
                         "subnet": {
-                          "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
+                          "id": "[parameters('subnetId')]"
                         }
                       }
                     }
```




## <a name="next-steps"></a>後續步驟

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
