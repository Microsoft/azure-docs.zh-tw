---
title: 在 Azure 擴展集範本中參照自訂映像 | Microsoft Docs
description: 了解如何將自訂映像新增到現有的「Azure 虛擬機器擴展集」範本
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
ms.date: 5/10/2017
ms.author: negat
ms.openlocfilehash: 28d2c080048a7f82e83ad9c1794c9757b330a8c7
ms.sourcegitcommit: f46cbcff710f590aebe437c6dd459452ddf0af09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2017
ms.locfileid: "26780913"
---
# <a name="add-a-custom-image-to-an-azure-scale-set-template"></a>新增自訂映像至 Azure 擴展集範本

本文說明如何修改[最基本的可行擴展集範本](./virtual-machine-scale-sets-mvss-start.md)從自訂映像部署。

## <a name="change-the-template-definition"></a>變更範本定義

您可以在[這裡](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)看到最基本的可行擴展集範本，並在[這裡](https://raw.githubusercontent.com/gatneil/mvss/custom-image/azuredeploy.json)看到用於從自訂映像部署擴展集的範本。 讓我們逐步檢查用來建立此範本 (`git diff minimum-viable-scale-set custom-image`) 的差異：

### <a name="creating-a-managed-disk-image"></a>建立受控磁碟映像

如果您已有自訂的受控磁碟映像 (類型為 `Microsoft.Compute/images` 的資源)，則可略過此節。

首先，新增 `sourceImageVhdUri` 參數，這是 Azure 儲存體中包含所要部署自訂映像的一般化 blob 其 URI。


```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "sourceImageVhdUri": {
+      "type": "string",
+      "metadata": {
+        "description": "The source of the generalized blob containing the custom image"
+      }
     }
   },
   "variables": {},
```

接著，新增類型為 `Microsoft.Compute/images` 的資源，這是以位在 URI `sourceImageVhdUri` 的一般化 blob 為基礎的受控磁碟映像。 此映像必須位在與使用它的擴展集相同的區域中。 在映像的屬性中，指定作業系統類型、blob 的位置 (從 `sourceImageVhdUri` 參數) 及儲存體帳戶類型：

```diff
   "resources": [
     {
+      "type": "Microsoft.Compute/images",
+      "apiVersion": "2016-04-30-preview",
+      "name": "myCustomImage",
+      "location": "[resourceGroup().location]",
+      "properties": {
+        "storageProfile": {
+          "osDisk": {
+            "osType": "Linux",
+            "osState": "Generalized",
+            "blobUri": "[parameters('sourceImageVhdUri')]",
+            "storageAccountType": "Standard_LRS"
+          }
+        }
+      }
+    },
+    {
       "type": "Microsoft.Network/virtualNetworks",
       "name": "myVnet",
       "location": "[resourceGroup().location]",

```

在擴展集資源中，新增參照自訂映像的 `dependsOn` 子句，以確定會先建立映像，擴展集才會嘗試從該映像部署：

```diff
       "location": "[resourceGroup().location]",
       "apiVersion": "2016-04-30-preview",
       "dependsOn": [
-        "Microsoft.Network/virtualNetworks/myVnet"
+        "Microsoft.Network/virtualNetworks/myVnet",
+        "Microsoft.Compute/images/myCustomImage"
       ],
       "sku": {
         "name": "Standard_A1",

```

### <a name="changing-scale-set-properties-to-use-the-managed-disk-image"></a>變更擴展集屬性以使用受控磁碟映像

在擴展集 `storageProfile` 的 `imageReference` 中，不是指定發行者、提供項目、sku 及平台映像版本，而是指定 `Microsoft.Compute/images` 資源的 `id`：

```diff
         "virtualMachineProfile": {
           "storageProfile": {
             "imageReference": {
-              "publisher": "Canonical",
-              "offer": "UbuntuServer",
-              "sku": "16.04-LTS",
-              "version": "latest"
+              "id": "[resourceId('Microsoft.Compute/images', 'myCustomImage')]"
             }
           },
           "osProfile": {
```

在此範例中，使用 `resourceId` 函式取得以相同範本所建立映像的資源識別碼。 如果您已事先建立受控磁碟映像，您應改為提供該映像的識別碼。 此識別碼的格式必須是：`/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Compute/images/<image-name>`。


## <a name="next-steps"></a>後續步驟

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
