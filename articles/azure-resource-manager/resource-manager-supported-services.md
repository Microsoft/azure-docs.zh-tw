---
title: Azure 資源提供者和資源類型 | Microsoft Docs
description: 說明支援資源管理員、其結構描述及可用 API 版本的資源提供者，以及可裝載資源的區域。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 3c7a6fe4-371a-40da-9ebe-b574f583305b
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/25/2017
ms.author: tomfitz
ms.openlocfilehash: 811bb40816339dbe7097e429722625a3ae5c95c0
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358195"
---
# <a name="resource-providers-and-types"></a>資源提供者和類型

部署資源時，您經常需要擷取有關資源提供者和類型的資訊。 在本文中，您將了解：

* 在 Azure 中檢視所有資源提供者
* 檢查資源提供者的註冊狀態
* 註冊資源提供者
* 檢視資源提供者的資源類型
* 檢視資源類型的有效位置
* 檢視資源類型的有效 API 版本

您可以透過入口網站、PowerShell 或 Azure CLI 執行下列步驟。

## <a name="powershell"></a>PowerShell

若要查看 Azure 中的所有資源提供者，以及您訂用帳戶的登錄狀態，請使用：

```powershell
Get-AzureRmResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

傳回的結果類似於：

```powershell
ProviderNamespace                RegistrationState
-------------------------------- ------------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

註冊資源提供者可將您的訂用帳戶設定為可搭配資源提供者使用。 註冊範圍一律是訂用帳戶。 許多資源提供者都會預設為自動註冊。 不過，您可能需要手動註冊某些資源提供者。 若要註冊資源提供者，您必須有權執行資源提供者的 `/register/action` 作業。 這項作業包含在「參與者」和「擁有者」角色中。

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
```

傳回的結果類似於：

```powershell
ProviderNamespace : Microsoft.Batch
RegistrationState : Registering
ResourceTypes     : {batchAccounts, operations, locations, locations/quotas}
Locations         : {West Europe, East US, East US 2, West US...}
```

當訂用帳戶中仍有資源提供者的資源類型時，您無法取消註冊該資源提供者。

若要查看特定資源提供者的資訊，請使用：

```powershell
Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
```

傳回的結果類似於：

```powershell
{ProviderNamespace : Microsoft.Batch
RegistrationState : Registered
ResourceTypes     : {batchAccounts}
Locations         : {West Europe, East US, East US 2, West US...}

...
```

若要查看資源提供者的資源類型，請使用：

```powershell
(Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes.ResourceTypeName
```

它會傳回：

```powershell
batchAccounts
operations
locations
locations/quotas
```

API 版本會對應至資源提供者所發行的 REST API 作業版本。 當資源提供者啟用新功能時，它會發行新版本的 REST API。 

若要取得資源類型的可用 API 版本，請使用：

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).ApiVersions
```

它會傳回：

```powershell
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

所有區域都支援資源管理員，但您部署的資源可能無法在所有區域中受到支援。 此外，您的訂用帳戶上可能會有一些限制，以防止您使用某些支援該資源的區域。 

若要取得資源類型支援的位置，請使用：

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

它會傳回：

```powershell
West Europe
East US
East US 2
West US
...
```

## <a name="azure-cli"></a>Azure CLI
若要查看 Azure 中的所有資源提供者，以及您訂用帳戶的登錄狀態，請使用：

```azurecli
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

傳回的結果類似於：

```azurecli
Provider                         Status
-------------------------------- ----------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

註冊資源提供者可將您的訂用帳戶設定為可搭配資源提供者使用。 註冊範圍一律是訂用帳戶。 許多資源提供者都會預設為自動註冊。 不過，您可能需要手動註冊某些資源提供者。 若要註冊資源提供者，您必須有權執行資源提供者的 `/register/action` 作業。 這項作業包含在「參與者」和「擁有者」角色中。

```azurecli
az provider register --namespace Microsoft.Batch
```

它會傳回一則訊息說明註冊持續進行中。

當訂用帳戶中仍有資源提供者的資源類型時，您無法取消註冊該資源提供者。

若要查看特定資源提供者的資訊，請使用：

```azurecli
az provider show --namespace Microsoft.Batch
```

傳回的結果類似於：

```azurecli
{
    "id": "/subscriptions/####-####/providers/Microsoft.Batch",
    "namespace": "Microsoft.Batch",
    "registrationsState": "Registering",
    "resourceTypes:" [
        ...
    ]
}
```

若要查看資源提供者的資源類型，請使用：

```azurecli
az provider show --namespace Microsoft.Batch --query "resourceTypes[*].resourceType" --out table
```

它會傳回：

```azurecli
Result
---------------
batchAccounts
operations
locations
locations/quotas
```

API 版本會對應至資源提供者所發行的 REST API 作業版本。 當資源提供者啟用新功能時，它會發行新版本的 REST API。 

若要取得資源類型的可用 API 版本，請使用：

```azurecli
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].apiVersions | [0]" --out table
```

它會傳回：

```azurecli
Result
---------------
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

所有區域都支援資源管理員，但您部署的資源可能無法在所有區域中受到支援。 此外，您的訂用帳戶上可能會有一些限制，以防止您使用某些支援該資源的區域。 

若要取得資源類型支援的位置，請使用：

```azurecli
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" --out table
```

它會傳回：

```azurecli
Result
---------------
West Europe
East US
East US 2
West US
...
```

## <a name="portal"></a>入口網站

訂用帳戶若要查看 Azure 中的所有資源提供者，以及您訂用帳戶的登錄狀態，請選取 [訂用帳戶]。

![選取 [訂用帳戶]](./media/resource-manager-supported-services/select-subscriptions.png)

選擇要檢視的訂用帳戶。

![指定訂用帳戶](./media/resource-manager-supported-services/subscription.png)

選取**資源提供者**並檢視可用資源提供者的清單。

![顯示資源提供者](./media/resource-manager-supported-services/show-resource-providers.png)

註冊資源提供者可將您的訂用帳戶設定為可搭配資源提供者使用。 註冊範圍一律是訂用帳戶。 許多資源提供者都會預設為自動註冊。 不過，您可能需要手動註冊某些資源提供者。 若要註冊資源提供者，您必須有權執行資源提供者的 `/register/action` 作業。 這項作業包含在「參與者」和「擁有者」角色中。 若要註冊資源提供者，請選取 [註冊]。

![註冊資源提供者](./media/resource-manager-supported-services/register-provider.png)

當訂用帳戶中仍有資源提供者的資源類型時，您無法取消註冊該資源提供者。

若要查看特定資源提供者的資訊，請選取 [所有服務]。

![選取所有服務](./media/resource-manager-supported-services/more-services.png)

搜尋**資源總管**並從可用的選項加以選取。

![選取 [資源總管]](./media/resource-manager-supported-services/select-resource-explorer.png)

選取 [提供者]。

![選取 [提供者]](./media/resource-manager-supported-services/select-providers.png)

選取資源提供者和您想要檢視的資源類型。

![選取 [資源類型]](./media/resource-manager-supported-services/select-resource-type.png)

所有區域都支援資源管理員，但您部署的資源可能無法在所有區域中受到支援。 此外，您的訂用帳戶上可能會有一些限制，以防止您使用某些支援該資源的區域。 資源總管會顯示資源類型的有效位置。

![顯示位置](./media/resource-manager-supported-services/show-locations.png)

API 版本會對應至資源提供者所發行的 REST API 作業版本。 當資源提供者啟用新功能時，它會發行新版本的 REST API。 資源總管會顯示資源類型的有效 API 版本。

![顯示 API 版本](./media/resource-manager-supported-services/show-api-versions.png)

## <a name="next-steps"></a>後續步驟
* 若要了解如何建立資源管理員範本，請參閱 [編寫 Azure 資源管理員範本](resource-group-authoring-templates.md)。
* 若要了解如何部署資源，請參閱 [使用 Azure 資源管理員範本部署應用程式](resource-group-template-deploy.md)。
* 若要檢視資源提供者的作業，請參閱 [Azure REST API](/rest/api/)。

