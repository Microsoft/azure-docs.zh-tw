---
title: 轉換 Azure Resource Manager 擴展集範本以使用受控磁碟 | Microsoft Docs
description: 轉換擴展集範本至受控磁碟擴展集範本。
keywords: 虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: bc8c377a-8c3f-45b8-8b2d-acc2d6d0b1e8
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 5/18/2017
ms.author: negat
ms.openlocfilehash: 760e30f5c6f4ecaff299bae1725548a6a7c5184c
ms.sourcegitcommit: f46cbcff710f590aebe437c6dd459452ddf0af09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2017
ms.locfileid: "26781066"
---
# <a name="convert-a-scale-set-template-to-a-managed-disk-scale-set-template"></a>轉換擴展集範本至受控磁碟擴展集範本

使用 Resource Manager 範本來建立不使用受控磁碟之擴展集的客戶可能希望修改它以使用受控磁碟。 本文說明如何使用受控磁碟，來作為 [Azure 快速入門範本](https://github.com/Azure/azure-quickstart-templates) (提供 Resource Manager 範本範例的社群導向存放庫) 的提取要求範例。 您可以在 [https://github.com/Azure/azure-quickstart-templates/pull/2998](https://github.com/Azure/azure-quickstart-templates/pull/2998) 找到完整提取要求，並在下面找到主要差異部分以及說明：

## <a name="making-the-os-disks-managed"></a>將 OS 磁碟設定為受控磁碟

下列 diff 已移除數個與儲存體帳戶相關的變數和磁碟屬性。 儲存體帳戶類型已不再是必要項目 (Standard_LRS 是預設值)，但您仍可視需要加以指定。 針對受控磁碟，只支援 Standard_LRS 與 Premium_LRS。 我們在舊範本中使用新的儲存體帳戶尾碼、唯一字串陣列與 SA 計數來產生儲存體帳戶名稱。 這些變數在新範本中已不再是必要項目，因為受控磁碟會代表客戶自動建立儲存體帳戶。 同樣地，VHD 容器名稱與 OS 磁碟名稱已不再是必要項目，因為受控磁碟會自動命名底層儲存體 Blob 容器與磁碟。

```diff
   "variables": {
-    "storageAccountType": "Standard_LRS",
     "namingInfix": "[toLower(substring(concat(parameters('vmssName'), uniqueString(resourceGroup().id)), 0, 9))]",
     "longNamingInfix": "[toLower(parameters('vmssName'))]",
-    "newStorageAccountSuffix": "[concat(variables('namingInfix'), 'sa')]",
-    "uniqueStringArray": [
-      "[concat(uniqueString(concat(resourceGroup().id, variables('newStorageAccountSuffix'), '0')))]",
-      "[concat(uniqueString(concat(resourceGroup().id, variables('newStorageAccountSuffix'), '1')))]",
-      "[concat(uniqueString(concat(resourceGroup().id, variables('newStorageAccountSuffix'), '2')))]",
-      "[concat(uniqueString(concat(resourceGroup().id, variables('newStorageAccountSuffix'), '3')))]",
-      "[concat(uniqueString(concat(resourceGroup().id, variables('newStorageAccountSuffix'), '4')))]"
-    ],
-    "saCount": "[length(variables('uniqueStringArray'))]",
-    "vhdContainerName": "[concat(variables('namingInfix'), 'vhd')]",
-    "osDiskName": "[concat(variables('namingInfix'), 'osdisk')]",
     "addressPrefix": "10.0.0.0/16",
     "subnetPrefix": "10.0.0.0/24",
     "virtualNetworkName": "[concat(variables('namingInfix'), 'vnet')]",
```


在下列 diff 中，計算 API 的版本已更新為 2016-04-30-preview，這是擴展集的受控磁碟支援所需的最低版本。 您仍可視需要在新 API 版本中搭配舊語法來使用非受控磁碟。 如果您僅更新計算 API 版本但未變更任何其他項目，範本應該可繼續如往常一樣運作。

```diff
@@ -86,7 +74,7 @@
       "version": "latest"
     },
     "imageReference": "[variables('osType')]",
-    "computeApiVersion": "2016-03-30",
+    "computeApiVersion": "2016-04-30-preview",
     "networkApiVersion": "2016-03-30",
     "storageApiVersion": "2015-06-15"
   },
```

下列 diff 已從資源陣列中徹底移除儲存體帳戶資源。 這不再是所需資源，因為受控磁碟會自動建立這些資源。

```diff
@@ -113,19 +101,6 @@
       }
     },
-    {
-      "type": "Microsoft.Storage/storageAccounts",
-      "name": "[concat(variables('uniqueStringArray')[copyIndex()], variables('newStorageAccountSuffix'))]",
-      "location": "[resourceGroup().location]",
-      "apiVersion": "[variables('storageApiVersion')]",
-      "copy": {
-        "name": "storageLoop",
-        "count": "[variables('saCount')]"
-      },
-      "properties": {
-        "accountType": "[variables('storageAccountType')]"
-      }
-    },
     {
       "type": "Microsoft.Network/publicIPAddresses",
       "name": "[variables('publicIPAddressName')]",
       "location": "[resourceGroup().location]",
```

在下列 diff 中，我們可以看到我們正在將從擴展集參考的「相依於」子句移至建立儲存體帳戶的迴圈。 在舊範本中，這可以確保會在開始建立擴展集之前先建立儲存體帳戶，但搭配受控磁碟使用時，此子句已非必要。 VHD 容器屬性以及 OS 磁碟名稱屬性也已經移除，因為這些屬性會自動由受控磁碟在幕後進行處理。 如果您想要使用進階 OS 磁碟，則可在 "osDisk" 設定中新增 `"managedDisk": { "storageAccountType": "Premium_LRS" }`。 只有 VM SKU 中具有大寫或小寫 's' 的 VM 可以使用進階磁碟。

```diff
@@ -183,7 +158,6 @@
       "location": "[resourceGroup().location]",
       "apiVersion": "[variables('computeApiVersion')]",
       "dependsOn": [
-        "storageLoop",
         "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]",
         "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
       ],
@@ -200,16 +174,8 @@
         "virtualMachineProfile": {
           "storageProfile": {
             "osDisk": {
-              "vhdContainers": [
-                "[concat('https://', variables('uniqueStringArray')[0], variables('newStorageAccountSuffix'), '.blob.core.windows.net/', variables('vhdContainerName'))]",
-                "[concat('https://', variables('uniqueStringArray')[1], variables('newStorageAccountSuffix'), '.blob.core.windows.net/', variables('vhdContainerName'))]",
-                "[concat('https://', variables('uniqueStringArray')[2], variables('newStorageAccountSuffix'), '.blob.core.windows.net/', variables('vhdContainerName'))]",
-                "[concat('https://', variables('uniqueStringArray')[3], variables('newStorageAccountSuffix'), '.blob.core.windows.net/', variables('vhdContainerName'))]",
-                "[concat('https://', variables('uniqueStringArray')[4], variables('newStorageAccountSuffix'), '.blob.core.windows.net/', variables('vhdContainerName'))]"
-              ],
-              "name": "[variables('osDiskName')]",
             },
             "imageReference": "[variables('imageReference')]"
           },

```

擴展集設定中並沒有明確的屬性可決定要使用受控或非受控磁碟。 擴展集會根據儲存體設定檔中的屬性來判斷要使用的磁碟。 因此，修改範本以確保擴展集的儲存體設定檔中有正確的屬性非常重要。


## <a name="data-disks"></a>資料磁碟

根據上面的變更，擴展集會針對 OS 磁碟使用受控磁碟，但對於資料磁碟呢？ 若要新增資料磁碟，請在與 "osDisk" 相同層級的 "storageProfile" 下新增 "dataDisks" 屬性。 屬性的值是 JSON 物件清單，其中每個物件都具有屬性 "lun" (對於 VM 上的每個資料磁碟，這必須是唯一的)、"createOption" ("empty" 是目前唯一支援的選項) 與 "diskSizeGB" (以 GB 為單位的磁碟大小，必須大於 0 並小於 1024)，如下列範例中所示： 

```
"dataDisks": [
  {
    "lun": "1",
    "createOption": "empty",
    "diskSizeGB": "1023"
  }
]
```

若在此陣列中指定 `n` 磁碟，擴展集中的每個 VM 都會取得 `n` 資料磁碟。 但是請注意，這些資料磁碟是未經處理的裝置。 它們並未格式化。 客戶可以決定是否要在使用這些磁碟之前先連結、分割或格式化它們。 或者，您也可以在每個資料磁碟物件中指定 `"managedDisk": { "storageAccountType": "Premium_LRS" }`，以指定它應該是進階資料磁碟。 只有 VM SKU 中具有大寫或小寫 's' 的 VM 可以使用進階磁碟。

若要深入了解如何搭配擴展集使用資料磁碟，請參閱[此文章](./virtual-machine-scale-sets-attached-disks.md)。


## <a name="next-steps"></a>後續步驟
如需使用擴展集的範例 Resource Manager 範本，請在 [Azure 快速入門範本 github 儲存機制](https://github.com/Azure/azure-quickstart-templates)中搜尋 "vmss"。

如需一般資訊，請參閱 [擴展集的主要登陸頁面](https://azure.microsoft.com/services/virtual-machine-scale-sets/)。

