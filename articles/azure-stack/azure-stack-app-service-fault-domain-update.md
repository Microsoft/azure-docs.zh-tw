---
title: App Service on Azure Stack：容錯網域更新 | Microsoft Docs
description: 如何跨容錯網域，在 Azure Stack 上轉散發 Azure App Service
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/29/2018
ms.author: anwestg
ms.openlocfilehash: 53766099f283f802482fe8e84144502d386b1d69
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39440146"
---
# <a name="how-to-redistribute-azure-app-service-on-azure-stack-across-fault-domains"></a>如何跨容錯網域，在 Azure Stack 上轉散發 Azure App Service

「適用於：Azure Stack 整合系統」

使用 1802 更新，Azure Stack 現在支援跨容錯網域散發工作負載，這是對於高可用性很重要的一項功能。

>[!IMPORTANT]
>若要利用容錯網域支援，您必須將 Azure Stack 整合系統更新為 1802。 本文件僅適用於在 1802 更新之前就已完成的 App Service 資源提供者部署。 如果您在 1802 更新套用到 Azure Stack 之後，才部署 App Service on Azure Stack，則資源提供者已經跨容錯網域散發。

## <a name="rebalance-an-app-service-resource-provider-across-fault-domains"></a>跨容錯網域重新平衡 App Service 資源提供者

若要針對 App Service 資源提供者轉散布部署的擴展集，您必須為每個擴展集執行本文中的步驟。 根據預設，擴展集名稱是：

* ManagementServersScaleSet
* FrontEndsScaleSet
* PublishersScaleSet
* SharedWorkerTierScaleSet
* SmallWorkerTierScaleSet
* MediumWorkerTierScaleSet
* LargeWorkerTierScaleSet

>[!NOTE]
> 如果某些背景工作角色層擴展集中未部署任何執行個體，您就不需要重新平衡那些擴展集。 當您在未來將其相應放大時，擴展集將會正確平衡。

若要相應放大擴展集，請遵循下列步驟：

1. 登入 Azure Stack 系統管理員入口網站。
1. 選取 [更多服務]。
1. 在 [計算] 下選取 [虛擬機器擴展集]。 部署為 App Service 部署一部分的現有擴展集會與執行個體計數資訊一起列出。 下列螢幕擷取畫面顯示擴展集範例。

      ![列在虛擬機器擴展集 UX 中的 Azure App Service 擴展集][1]

1. 相應放大每個集合。 例如，如果您在擴展集中有 3 個現有執行個體，您必須相應放大為 6 個，以便跨容錯網域部署 3 個新執行個體。 下列 PowerShell 範例顯示如何相應放大擴展集。

   ```powershell
   Add-AzureRmAccount -EnvironmentName AzureStackAdmin 

   # Get current scale set
   $vmss = Get-AzureRmVmss -ResourceGroupName "AppService.local" -VMScaleSetName "SmallWorkerTierScaleSet"

   # Set and update the capacity of your scale set
   $vmss.sku.capacity = 6
   Update-AzureRmVmss -ResourceGroupName AppService.local" -Name "SmallWorkerTierScaleSet" -VirtualMachineScaleSet $vmss
   ```

   >[!NOTE]
   >視角色類型和執行個體數目而定，此步驟可能需要幾小時的時間才能完成。

1. 在 [App Service 系統管理角色] 中，監視新角色執行個體的狀態。 若要檢查角色執行個體的狀態，請在清單中選取角色類型

    ![Azure App Service on Azure Stack 角色][2]

1. 當新的角色執行個體狀態為 [就緒] 時，返回 [虛擬機器擴展集] 並**刪除**舊的角色執行個體。

1. 針對**每個**虛擬機器擴展集重複這些步驟。

## <a name="next-steps"></a>後續步驟

您也可以嘗試其他[平台即服務 (PaaS) 服務](azure-stack-tools-paas-services.md)。

* [SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)
* [MySQL 資源提供者](azure-stack-mysql-resource-provider-deploy.md)

<!--Image references-->
[1]: ./media/azure-stack-app-service-fault-domain-update/app-service-scale-sets.png
[2]: ./media/azure-stack-app-service-fault-domain-update/app-service-roles.png
