---
title: 在 Windows PowerShell 中調整 Azure 雲端服務 |Microsoft Docs
description: (傳統) 了解如何使用 PowerShell 在 Azure 中相應放大或縮小 Web 角色或背景工作角色。
services: cloud-services
documentationcenter: ''
author: mmccrory
manager: timlt
editor: ''
ms.assetid: ee37dd8c-6714-4c61-adb8-03d6bbf76c9a
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/01/2016
ms.author: mmccrory
ms.openlocfilehash: a7ae8ff202d403dff19b8c9a6a09492235db27ac
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22998003"
---
# <a name="how-to-scale-a-cloud-service-in-powershell"></a>如何在 PowerShell 中調整雲端服務

您可以使用 Windows PowerShell 來新增或移除執行個體，以相應放大或縮小 Web 角色或背景工作角色。  

## <a name="log-in-to-azure"></a>登入 Azure

您必須先登入，才能透過 PowerShell 在訂用帳戶上執行任何作業︰

```powershell
Add-AzureAccount
```

如果您的帳戶有多個相關聯的訂用帳戶，視您的雲端服務所在的位置而定，您可能需要變更目前的訂用帳戶。 若要檢查目前的訂用帳戶，請執行：

```powershell
Get-AzureSubscription -Current
```

如果您需要變更目前的訂用帳戶，請執行︰

```powershell
Set-AzureSubscription -SubscriptionId <subscription_id>
```

## <a name="check-the-current-instance-count-for-your-role"></a>檢查您的角色目前的執行個體計數

若要檢查您的角色目前的狀態，請執行。

```powershell
Get-AzureRole -ServiceName '<your_service_name>' -RoleName '<your_role_name>'
```

您應該會取回角色的相關資訊，包括其目前的作業系統版本和執行個體計數。 在此案例中，角色具有單一執行個體。

![角色的相關資訊](./media/cloud-services-how-to-scale-powershell/get-azure-role.png)

## <a name="scale-out-the-role-by-adding-more-instances"></a>新增更多執行個體以相應放大角色

若要相應放大您的角色，請以 **Count** 參數將所需的執行個體數目傳遞至 **Set-AzureRole** Cmdlet︰

```powershell
Set-AzureRole -ServiceName '<your_service_name>' -RoleName '<your_role_name>' -Slot <target_slot> -Count <desired_instances>
```

在佈建和啟動新的執行個體時，Cmdlet 區塊會暫時凍結。 在此期間，如果您開啟新的 PowerShell 視窗，然後如稍早所示呼叫 **Get-AzureRole**，您會看到新的目標執行個體計數。 如果您在入口網站中檢查角色狀態，應該會看到新的執行個體正在啟動︰

![VM instance starting in portal](./media/cloud-services-how-to-scale-powershell/role-instance-starting.png)

當新的執行個體啟動後，Cmdlet 將會順利返回︰

![角色執行個體成功增加](./media/cloud-services-how-to-scale-powershell/set-azure-role-success.png)

## <a name="scale-in-the-role-by-removing-instances"></a>移除執行個體以相應縮小角色

您可以用相同方式移除執行個體，以相應縮小角色。 將 **Set-AzureRole**的 **Count** 參數設定為您希望相應縮小作業完成之後存在的執行個體數目。

## <a name="next-steps"></a>後續步驟

您無法從 PowerShell 設定自動調整雲端服務。 若要這麼做，請參閱[如何自動調整雲端服務](cloud-services-how-to-scale-portal.md)。
