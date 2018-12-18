---
title: 了解虛擬機器擴展集範本 | Microsoft Docs
description: 了解如何為虛擬機器擴展集建立最基本的可行擴展集範本
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
ms.date: 06/01/2017
ms.author: negat
ms.openlocfilehash: 5cd495d1332c71d7eae775f933b73d98826f10e4
ms.sourcegitcommit: f46cbcff710f590aebe437c6dd459452ddf0af09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2017
ms.locfileid: "26781250"
---
# <a name="learn-about-virtual-machine-scale-set-templates"></a>了解虛擬機器擴展集範本
[Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#template-deployment)是部署相關資源群組的絕佳方式。 本教學課程系列說明如何建立最基本的可行擴展集範本，以及如何修改此範本來配合各種案例。 所有範例皆來自這個 [GitHub 存放庫](https://github.com/gatneil/mvss)。 

此範本已刻意簡化。 如需較完整的擴展集範本範例，請參閱 [Azure 快速入門範本 GitHub 存放庫](https://github.com/Azure/azure-quickstart-templates)，然後搜尋包含 `vmss` 字串的資料夾。

如果您已經熟悉如何建立範本，您可以跳到「後續步驟」一節，以了解如何修改此範本。

## <a name="review-the-template"></a>檢閱範本

使用 GitHub 以檢閱最基本的可行擴展集範本 [azuredeploy.json](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)。

在本教學課程中，讓我們檢查差異 (`git diff master minimum-viable-scale-set`)，逐步建立最基本的可行擴展集範本。

## <a name="define-schema-and-contentversion"></a>定義 $schema 和 contentVersion
首先，請定義範本中的 `$schema` 和 `contentVersion`。 `$schema` 元素會定義範本語言的版本，並用於 Visual Studio 語法醒目提示及類似的驗證功能。 `contentVersion` 元素不是由 Azure 使用。 相反地，它可協助您追蹤範本版本。

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
```
## <a name="define-parameters"></a>定義參數
接著，定義兩個參數：`adminUsername` 和 `adminPassword`。 參數是您在部署時指定的值。 `adminUsername` 參數就是一個 `string`，但是由於 `adminPassword` 是祕密，因此請賦予它 `securestring` 類型。 稍後，這些參數會被傳送到擴展集組態。

```json
  "parameters": {
    "adminUsername": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    }
  },
```
## <a name="define-variables"></a>定義變數
Resource Manager 範本也可讓您定義以後要在範本中使用的變數。 範例不會使用任何變數，因此 JSON 物件是空白的。

```json
  "variables": {},
```

## <a name="define-resources"></a>定義資源
接下來是範本的資源區段。 在這裡，您會定義實際想要部署的項目。 與 `parameters` 和 `variables` (這些是 JSON 物件) 不同，`resources` 是一個 JSON 物件的 JSON 清單。

```json
   "resources": [
```

所有資源都必須要有 `type`、`name`、`apiVersion` 及 `location` 屬性。 這個範例的第一個資源具有類型 `Microsft.Network/virtualNetwork`、名稱`myVnet`，和 apiVersion `2016-03-30`。 (若要了解資源類型的最新 API 版本，請參閱 [Azure REST API 文件](https://docs.microsoft.com/rest/api/)。)

```json
     {
       "type": "Microsoft.Network/virtualNetworks",
       "name": "myVnet",
       "apiVersion": "2016-12-01",
```

## <a name="specify-location"></a>指定位置
若要指定虛擬網路的位置，請使用 [Resource Manager 範本函式](../azure-resource-manager/resource-group-template-functions.md)。 此函式必須括在引號和方括號內，如下所示︰`"[<template-function>]"`。 在此案例中，請使用 `resourceGroup` 函式。 此函式不接受任何引數並且會傳回 JSON 物件，此物件含有這項部署之目的地資源群組的相關中繼資料。 資源群組是由使用者在部署時所設定。 接著會使用 `.location` 將此值編入此 JSON 物件中來作為索引，以從此 JSON 物件取得位置。

```json
       "location": "[resourceGroup().location]",
```

## <a name="specify-virtual-network-properties"></a>指定虛擬網路屬性
每個 Resource Manager 資源都有自己的 `properties` 區段，用於該資源的特定組態。 在此案例中，請指定虛擬網路應該有一個使用私人 IP 位址範圍 `10.0.0.0/16` 的子網路。 擴展集一律是包含在一個子網路內。 它不能跨子網路。

```json
       "properties": {
         "addressSpace": {
           "addressPrefixes": [
             "10.0.0.0/16"
           ]
         },
         "subnets": [
           {
             "name": "mySubnet",
             "properties": {
               "addressPrefix": "10.0.0.0/16"
             }
           }
         ]
       }
     },
```

## <a name="add-dependson-list"></a>新增 dependsOn 清單
除了必要的 `type`、`name`、`apiVersion` 和 `location` 屬性，每個資源都能擁有選用的 `dependsOn` 字串清單。 這份清單會指定此部署中的哪些其他資源必須在部署此資源之前完成。

在此案例中，這個清單中只有一個元素，即上述範例中的虛擬網路。 之所以指定這項相依性，是因為擴展集必須先有網路存在，才能建立任何 VM。 如此一來，擴展集才能從先前在網路屬性中指定的 IP 位址範圍，為這些 VM 提供私人 IP 位址。 dependsOn 清單中每個字串的格式是 `<type>/<name>`。 使用與先前在虛擬網路資源定義中所使用的相同 `type` 和 `name`。

```json
     {
       "type": "Microsoft.Compute/virtualMachineScaleSets",
       "name": "myScaleSet",
       "apiVersion": "2016-04-30-preview",
       "location": "[resourceGroup().location]",
       "dependsOn": [
         "Microsoft.Network/virtualNetworks/myVnet"
       ],
```
## <a name="specify-scale-set-properties"></a>指定擴展集屬性
擴展集有許多屬性，可以在擴展集中自訂 VM。 如需這些屬性的完整清單，請參閱[擴展集 REST API 文件](https://docs.microsoft.com/rest/api/virtualmachinescalesets/create-or-update-a-set)。 本教學課程只會設定一些常用的屬性。
### <a name="supply-vm-size-and-capacity"></a>提供 VM 大小和容量
擴展集需要知道要建立的 VM 大小 (即「SKU 名稱」)，以及要建立多少個這類 VM (即「SKU 容量」)。 若要查看有哪些可用的 VM 大小，請參閱 [VM 大小文件](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-sizes)。

```json
       "sku": {
         "name": "Standard_A1",
         "capacity": 2
       },
```

### <a name="choose-type-of-updates"></a>選擇更新類型
擴展集也需要知道如何處理擴展集上的更新。 目前有兩個選項：`Manual` 和 `Automatic`。 如需有關兩者之間差異的詳細資訊，請參閱有關[如何升級擴展集](./virtual-machine-scale-sets-upgrade-scale-set.md)的文件。

```json
       "properties": {
         "upgradePolicy": {
           "mode": "Manual"
         },
```

### <a name="choose-vm-operating-system"></a>選擇 VM 作業系統
擴展集需要知道要放置 VM 的作業系統。 在此，請使用已完全修補的 Ubuntu 16.04-LTS 映像來建立虛擬機器。

```json
         "virtualMachineProfile": {
           "storageProfile": {
             "imageReference": {
               "publisher": "Canonical",
               "offer": "UbuntuServer",
               "sku": "16.04-LTS",
               "version": "latest"
             }
           },
```

### <a name="specify-computernameprefix"></a>指定 computerNamePrefix
擴展集會部署多個 VM。 請指定 `computerNamePrefix`，而不是指定每個虛擬機器名稱。 擴展集會針對每個 VM 在此名稱前置詞附加索引，因此 VM 名稱的格式會是 `<computerNamePrefix>_<auto-generated-index>`。

在下列程式碼片段中，使用先前的參數來設定擴展集內所有虛擬機器的系統管理員使用者名稱和密碼。 此程序會使用 `parameters` 範本函式。 此函式會接受字串，該字串指定要參考哪些參數，並且輸出該參數的值。

```json
           "osProfile": {
             "computerNamePrefix": "vm",
             "adminUsername": "[parameters('adminUsername')]",
             "adminPassword": "[parameters('adminPassword')]"
           },
```

### <a name="specify-vm-network-configuration"></a>指定 VM 網路組態
最後，為擴展集內的虛擬機器指定網路設定。 在此案例中，您只需要指定稍早建立之子網路的識別碼。 這會告訴擴展集將網路介面放入這個子網路。

您可以使用 `resourceId` 範本函式來取得包含該子網路的虛擬網路 ID。 此函式會接受資源的類型和名稱，然後傳回該資源的完整識別碼。 此 ID 的格式為：`/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/<resourceProviderNamespace>/<resourceType>/<resourceName>`

不過，只有虛擬網路的識別碼還不夠。 請提供擴展集虛擬機器應該位於的特定子網路。 若要這樣做，請將 `/subnets/mySubnet` 串連至虛擬網路的 ID。 結果是會產生子網路的完整 ID。 使用 `concat` 函式來執行這項操作，此函式會接受一系列字串，然後傳回其串連結果。

```json
           "networkProfile": {
             "networkInterfaceConfigurations": [
               {
                 "name": "myNic",
                 "properties": {
                   "primary": "true",
                   "ipConfigurations": [
                     {
                       "name": "myIpConfig",
                       "properties": {
                         "subnet": {
                           "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
                         }
                       }
                     }
                   ]
                 }
               }
             ]
           }
         }
       }
     }
   ]
}

```

## <a name="next-steps"></a>後續步驟

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
