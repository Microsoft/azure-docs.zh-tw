---
title: Azure 資源提供者註冊錯誤 | Microsoft Docs
description: 描述如何解決 Azure 資源提供者註冊錯誤。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 03/09/2018
ms.author: tomfitz
ms.openlocfilehash: b90009c1cd08a1004e58c4b9f25cd6350712fbcd
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358603"
---
# <a name="resolve-errors-for-resource-provider-registration"></a>解決資源提供者註冊的錯誤

本文描述使用您先前未在訂用帳戶中使用的資源提供者時，可能會遇到的錯誤。

## <a name="symptom"></a>徵狀

在部署資源時，您可能會收到下列錯誤代碼和訊息︰

```
Code: NoRegisteredProviderFound
Message: No registered resource provider found for location {location}
and API version {api-version} for type {resource-type}.
```

或者，您可能會收到類似訊息，指出︰

```
Code: MissingSubscriptionRegistration
Message: The subscription is not registered to use namespace {resource-provider-namespace}
```

錯誤訊息應可提供給您支援的位置和 API 版本建議。 您可以將您的範本變更為其中一個建議的值。 Azure 入口網站或正在使用的命令列介面會自動註冊大部分的提供者；但並非全部。 如果您未曾使用特定的資源提供者，您可能需要註冊該提供者。

## <a name="cause"></a>原因

您會因為下列三個原因其中一個而收到此錯誤︰

1. 尚未向您的訂用帳戶註冊此資源提供者
1. 此資源類型不支援 API 版本
1. 此資源類型不支援位置

## <a name="solution-1---powershell"></a>解決方案 1：PowerShell

針對 PowerShell，請使用 **Get-AzureRmResourceProvider** 來查看您的註冊狀態。

```powershell
Get-AzureRmResourceProvider -ListAvailable
```

若要註冊提供者，請使用 **Register-AzureRmResourceProvider** ，並提供您想要註冊的資源提供者名稱。

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Cdn
```

若要取得支援特定資源類型的位置，請使用︰

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
```

若要取得支援特定資源類型的 API 版本，請使用︰

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
```

## <a name="solution-2---azure-cli"></a>解決方案 2：Azure CLI

若要查看是否已註冊該提供者，請使用 `az provider list` 命令。

```azurecli-interactive
az provider list
```

若要註冊資源提供者，請使用 `az provider register` 命令，然後指定要註冊的*命名空間*。

```azurecli-interactive
az provider register --namespace Microsoft.Cdn
```

若要查看資源類型所支援的位置和 API 版本，請使用︰

```azurecli-interactive
az provider show -n Microsoft.Web --query "resourceTypes[?resourceType=='sites'].locations"
```

## <a name="solution-3---azure-portal"></a>解決方案 3：Azure 入口網站

您可以看到註冊狀態，並透過入口網站註冊資源提供者命名空間。

1. 對於您的訂用帳戶，選取**資源提供者**。

   ![選取資源提供者](./media/resource-manager-register-provider-errors/select-resource-provider.png)

1. 查看資源提供者的清單，並視需要選取 [註冊] 連結以註冊您嘗試部署的資源提供者類型。

   ![列出資源提供者](./media/resource-manager-register-provider-errors/list-resource-providers.png)
