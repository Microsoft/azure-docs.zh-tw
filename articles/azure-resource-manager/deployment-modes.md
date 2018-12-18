---
title: Azure Resource Manager 部署模式 | Microsoft Docs
description: 說明如何指定是否要透過 Azure Resource Manager 使用完整或累加部署模式。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/07/2018
ms.author: tomfitz
ms.openlocfilehash: c8c6c5499e1cea04bc5bdffbb5c07b53b96182e2
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42141874"
---
# <a name="azure-resource-manager-deployment-modes"></a>Azure Resource Manager 部署模式
部署您的資源時，您可以指定部署是累加式更新或完整更新。  這兩種模式的主要差異在於 Resource Manager 如何處理資源群組中未在範本內的現有資源。
預設模式為累加。

## <a name="incremental-and-complete-deployments"></a>累加部署與完整部署
部署資源時：

* 在完整模式中，Resource Manager 會**刪除**現存於資源群組中但未在範本內指定的資源。 
* 在累加模式中，Resource Manager 會讓現存於資源群組中但未在範本內指定的資源**保持不變**。

在這兩種模式中，Resource Manager 都會嘗試佈建範本內指定的所有資源。 如果資源已存在於資源群組中，且其設定保持不變，此作業不會導致任何變更。 如果您變更資源設定，則會以這些新設定佈建資源。 如果您嘗試更新現有資源的位置或類型，部署會失敗並發生錯誤。 相反地，請以您需要的位置或類型部署新的資源。

## <a name="example-result"></a>範例結果

若要說明增量和完整模式之間的差異，請考慮下列案例。

**現有資源群組**包含︰

* 資源 A
* 資源 B
* 資源 C

**範本**定義：

* 資源 A
* 資源 B
* 資源 D

部署於**累加**模式時，資源群組具有：

* 資源 A
* 資源 B
* 資源 C
* 資源 D

若部署在**完整**模式中，資源 C 會遭到刪除。 資源群組具有：

* 資源 A
* 資源 B
* 資源 D

## <a name="set-deployment-mode"></a>設定部署模式

若要在使用 PowerShell 部署時設定部署模式，使用 `Mode` 參數。

```powershell
New-AzureRmResourceGroupDeployment `
  -Mode Complete `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json 
```

若要在使用 Azure CLI 部署時設定部署模式，使用 `mode` 參數。

```azurecli-interactive
az group deployment create \
  --name ExampleDeployment \
  --mode Complete \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

使用[已連結或巢狀範本](resource-group-linked-templates.md)時，您必須將 `mode` 屬性設為 `Incremental`。 只有根層級範本支援完整部署模式。

```json
"resources": [
  {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          <nested-template-or-external-template>
      }
  }
]
```

## <a name="next-steps"></a>後續步驟
* 若要了解如何建立資源管理員範本，請參閱 [編寫 Azure 資源管理員範本](resource-group-authoring-templates.md)。
* 若要了解如何部署資源，請參閱 [使用 Azure 資源管理員範本部署應用程式](resource-group-template-deploy.md)。
* 若要檢視資源提供者的作業，請參閱 [Azure REST API](/rest/api/)。

