---
title: 在應用程式服務中相應放大背景工作角色 - Azure Stack | Microsoft Docs
description: 調整 Azure Stack 應用程式服務的詳細指導方針
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: 3cbe87bd-8ae2-47dc-a367-51e67ed4b3c0
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/08/2018
ms.author: anwestg
ms.reviewer: sethm
ms.openlocfilehash: f4a3cd7d09c18506a85317db7a360b8de3a9630b
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077015"
---
# <a name="app-service-on-azure-stack-add-more-infrastructure-or-worker-roles"></a>App Service on Azure Stack：新增更多基礎結構或背景工作角色

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*  

此文件提供如何調整 App Service on Azure Stack 基礎結構和背景工作角色的相關指示。 其中包含建立額外的背景工作角色以支援任何規模之應用程式的步驟。

> [!NOTE]
> 如果您「Azure Stack 環境」的 RAM 未超過 96 GB，您可能很難增加額外的容量。

根據預設值，Azure Stack 上的 App Service 支援免費和共用背景工作角色層。 若要新增其他背景工作角色層，您需要新增更多背景工作角色。

如果您不確定已透過 Azure Stack 安裝上的預設 App Service 部署了哪個背景工作層，您可以在 [Azure Stack 上的 App Service 概觀](azure-stack-app-service-overview.md)中檢閱其他資訊。

Azure App Service on Azure Stack 會使用虛擬機器擴展集部署所有角色，實質上利用此工作負載的縮放功能。 因此，所有背景工作層的縮放是透過 App Service 管理員完成。

> [!IMPORTANT]
> 目前無法如 Azure Stack 版本資訊中所述，在入口網站中調整虛擬機器擴展集，因此請使用 PowerShell 範例來相應放大規模。
>
>

## <a name="add-additional-workers-with-powershell"></a>使用 PowerShell 來新增額外的背景工作

1. [在 PowerShell 中設定 Azure Stack 管理環境](azure-stack-powershell-configure-admin.md)

2. 使用以下範例來相應放大擴展集的規模：
   ```powershell
   
    ##### Scale out the AppService Role instances ######
   
    # Set context to AzureStack admin.
    Login-AzureRmAccount -EnvironmentName AzureStackAdmin
                                                 
    ## Name of the Resource group where AppService is deployed.
    $AppServiceResourceGroupName = "AppService.local"

    ## Name of the ScaleSet : e.g. FrontEndsScaleSet, ManagementServersScaleSet, PublishersScaleSet , LargeWorkerTierScaleSet,      MediumWorkerTierScaleSet, SmallWorkerTierScaleSet, SharedWorkerTierScaleSet
    $ScaleSetName = "SharedWorkerTierScaleSet"

    ## TotalCapacity is sum of the instances needed at the end of operation. 
    ## e.g. if your VMSS has 1 instance(s) currently and you need 1 more the TotalCapacity should be set to 2
    $TotalCapacity = 2  

    # Get current scale set
    $vmss = Get-AzureRmVmss -ResourceGroupName $AppServiceResourceGroupName -VMScaleSetName $ScaleSetName

    # Set and update the capacity
    $vmss.sku.capacity = $TotalCapacity
    Update-AzureRmVmss -ResourceGroupName $AppServiceResourceGroupName -Name $ScaleSetName -VirtualMachineScaleSet $vmss 
   ```    

   > [!NOTE]
   > 視角色類型和執行個體數目而定，此步驟可能需要幾小時的時間才能完成。
   >
   >

3. 在 App Service 的 [管理] 中監視新角色執行個體的狀態，若要檢查個別角色執行個體的狀態，請按一下清單中的角色類型。

## <a name="add-additional-workers-directly-within-the-app-service-resource-provider-admin"></a>直接在 App Service 資源提供者系統管理內新增其他背景工作角色。

1. 以服務管理員身分登入 Azure Stack 管理入口網站。

2. 瀏覽至 [App Service]。

    ![](media/azure-stack-app-service-add-worker-roles/image01.png)

3. 按一下 [角色]。 您可以在這裡看到所有已部署之 App Service 角色的明細。

4. 以滑鼠右鍵按一下您想要調整的類型列，然後按一下 [ScaleSet]。

    ![](media/azure-stack-app-service-add-worker-roles/image02.png)

5. 按一下 [調整大小]，選取您想要調整為多少執行個體數，然後按一下 [儲存]。

    ![](media/azure-stack-app-service-add-worker-roles/image03.png)

6. Azure App Service on Azure Stack 現在將新增額外的 VM、設定它們、安裝所有必要的軟體，並在完成此程序時將它們標示為就緒。 此程序大約需要 80 分鐘。

7. 您可以檢視 [角色] 刀鋒視窗中的背景工作，來監視整備新角色的進度。

## <a name="result"></a>結果

在將它們完整部署並且準備好後，背景工作會變成可供使用者用來將工作負載部署到當中。 以下顯示的是預設可用的多個定價層範例。 如果特定的背景工作層沒有可用的背景工作，就無法使用選擇對應定價層的選項。

![](media/azure-stack-app-service-add-worker-roles/image04.png)

>[!NOTE]
> 若要相應放大「管理」、「前端」或「發行者」角色的規模，您必須相應放大對應之角色類型的規模。 
>
>

若要相應放大管理、前端或發行者角色，請遵循相同的步驟來選取適當的角色類型。 控制項不會部署為擴展集，因此這兩者應在所有生產環境部署的安裝期間進行部署。

### <a name="next-steps"></a>後續步驟

[設定部署來源](azure-stack-app-service-configure-deployment-sources.md)
