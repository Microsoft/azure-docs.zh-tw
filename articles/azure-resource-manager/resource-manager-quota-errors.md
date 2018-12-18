---
title: Azure 配額錯誤 |Microsoft Docs
description: 描述如何解決資源配額錯誤。
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
ms.openlocfilehash: 6d9048ae531abedb89b70989ce1c962357c514cd
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34357039"
---
# <a name="resolve-errors-for-resource-quotas"></a>解決資源配額的錯誤

本文描述當部署資源時，可能會遇到的配額錯誤。

## <a name="symptom"></a>徵狀

如果您部署的範本會建立超過 Azure 配額的資源，您會看到類似以下的部署錯誤：

```
Code=OperationNotAllowed
Message=Operation results in exceeding quota limits of Core.
Maximum allowed: 4, Current in use: 4, Additional requested: 2.
```

或者，您可能會看到：

```
Code=ResourceQuotaExceeded
Message=Creating the resource of type <resource-type> would exceed the quota of <number>
resources of type <resource-type> per resource group. The current resource count is <number>,
please delete some resources of this type before creating a new one.
```

## <a name="cause"></a>原因

配額在每個資源群組、訂用帳戶、帳戶和其他範圍套用。 例如，您的訂用帳戶可能設定為要限制區域的核心數目。 如果您嘗試部署超過允許核心數目的虛擬機器，您會收到錯誤訊息指出已超過配額。
如需完整的配額資訊，請參閱 [Azure 訂用帳戶和服務限制、配額與條件約束](../azure-subscription-service-limits.md)。

## <a name="troubleshooting"></a>疑難排解

### <a name="azure-cli"></a>Azure CLI

針對 Azure CLI，使用 `az vm list-usage` 命令以找出虛擬機器配額。

```azurecli
az vm list-usage --location "South Central US"
```

它會傳回：

```azurecli
[
  {
    "currentValue": 0,
    "limit": 2000,
    "name": {
      "localizedValue": "Availability Sets",
      "value": "availabilitySets"
    }
  },
  ...
]
```

### <a name="powershell"></a>PowerShell

針對 PowerShell，使用 **Get-AzureRmVMUsage** 命令以找出虛擬機器配額。

```powershell
Get-AzureRmVMUsage -Location "South Central US"
```

它會傳回：

```powershell
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional Cores                         0   100 Count
Virtual Machines                             0 10000 Count
```

## <a name="solution"></a>解決方法

若要要求增加配額，請移至入口網站並提出支援問題。 請在支援問題中，要求提高您要部署的區域的配額。

> [!NOTE]
> 請記住，對於資源群組，配額適用於每個個別區域，而不是整個訂用帳戶。 如果您需要在美國西部部署 30 個核心，就必須要求在美國西部擁有 30 個資源管理員核心。 如果您需要在任何具有存取權限的區域中部署 30 個核心，就應該要求在所有區域中擁有 30 個 Resource Manager 核心。
>
>

1. 選取 [訂用帳戶]。

   ![訂用帳戶](./media/resource-manager-quota-errors/subscriptions.png)

2. 選取需要增加配額的訂用帳戶。

   ![選取訂用帳戶](./media/resource-manager-quota-errors/select-subscription.png)

3. 選取 [使用量 + 配額]

   ![選取使用量和配額](./media/resource-manager-quota-errors/select-usage-quotas.png)

4. 在右上角，選取 [要求增加配額]。

   ![要求增加配額](./media/resource-manager-quota-errors/request-increase.png)

5. 填寫需要增加的配額類型的表單。

   ![填寫表單](./media/resource-manager-quota-errors/forms.png)