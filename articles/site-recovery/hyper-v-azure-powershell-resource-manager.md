---
title: 使用 PowerShell 與 Azure Resource Manager 來複寫 Hyper-V VM | Microsoft Docs
description: 使用 PowerShell 和 Azure Resource Manager，透過 Azure Site Recovery 將 Hyper-V VM 至 Azure 的複寫自動化。
services: site-recovery
author: bsiva
manager: abhiag
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: bsiva
ms.openlocfilehash: 721bb725538b0b1f6eb0e7132b99e75491b6f969
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037987"
---
# <a name="set-up-disaster-recovery-to-azure-for-hyper-v-vms-using-powershell-and-azure-resource-manager"></a>針對 Hyper-V VM，使用 PowerShell 和 Azure Resource Manager 設定至 Azure 的災害復原

[Azure Site Recovery](site-recovery-overview.md) 可藉由協調 Azure 虛擬機器 (VM)、內部部署 VM 與實體伺服器的複寫、容錯移轉及復原，為您的商務持續性與嚴重損壞修復 (BCDR) 策略做出貢獻。

本文說明如何搭配 Azure Resource Manager 使用 Windows PowerShell，將 Hyper-V VM 複寫到 Azure。 本文使用的範例示範如何將 Hyper-V 主機上執行的單一 VM 複寫到 Azure。

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell 提供 Cmdlet，讓您使用 Windows PowerShell 管理 Azure。 Azure PowerShell for Azure Resource Manager 提供的 Site Recovery PowerShell Cmdlet 可讓您在 Azure 中保護與復原伺服器。

您不需要是 PowerShell 專家也能使用本文，但您必須了解模組、Cmdlet 和工作階段等基本概念。 閱讀[開始使用 Windows PowerShell](http://technet.microsoft.com/library/hh857337.aspx) 和[搭配使用 Azure PowerShell 與 Azure Resource Manager](../powershell-azure-resource-manager.md)。

> [!NOTE]
> 雲端方案提供者 (CSP) 計畫的 Microsoft 合作夥伴，可以在其客戶各自的 CSP 訂用帳戶 (租用戶訂用帳戶) 設定和管理客戶的伺服器保護。
>
>

## <a name="before-you-start"></a>開始之前
確認您已備妥這些必要條件：

* [Microsoft Azure](https://azure.microsoft.com/) 帳戶。 您可以從 [免費試用](https://azure.microsoft.com/pricing/free-trial/)開始。 此外，您可以參閱 [Azure Site Recovery 管理員價格](https://azure.microsoft.com/pricing/details/site-recovery/)。
* Azure PowerShell 1.0。 如需有關此版本以及如何安裝的資訊，請參閱 [Azure PowerShell 1.0。](https://azure.microsoft.com/)
* [AzureRM.SiteRecovery](https://www.powershellgallery.com/packages/AzureRM.SiteRecovery/) 和 [AzureRM.RecoveryServices](https://www.powershellgallery.com/packages/AzureRM.RecoveryServices/) 模組。 您可以從 [PowerShell](https://www.powershellgallery.com/)

此外，本文所述的特定範例有下列先決條件：

* 執行 Windows Server 2012 R2 或 Microsoft Hyper-V Server 2012 R2 且包含一或多部 VM 的 Hyper-V 主機。 Hyper-V 伺服器應該直接或透過 Proxy 連接到網際網路。
* 您想要複寫的 VM 都應該符合[這些必要條件](hyper-v-azure-support-matrix.md#replicated-vms)。

## <a name="step-1-sign-in-to-your-azure-account"></a>步驟 1：登入您的 Azure 帳戶

1. 開啟 PowerShell 主控台並執行這個命令，登入您的 Azure 帳戶。 此 Cmdlet 會開啟一個網頁並提示您輸入帳戶認證：**Connect-AzureRmAccount**。
    - 或者，您可以使用 **-Credential** 參數，以參數形式將您的帳戶認證加入 **Connect-AzureRmAccount** Cmdlet。
    - 如果您是代表租用戶工作的 CSP 合作夥伴，請使用客戶的 tenantID 或租用戶主要網域名稱將客戶指定為租用戶。 例如：**Connect-AzureRmAccount -Tenant "fabrikam.com"**
2. 由於一個帳戶可以有多個訂用帳戶，因此您必須將要使用的訂用帳戶與帳戶建立關聯：

    `Select-AzureRmSubscription -SubscriptionName $SubscriptionName`

3. 使用下列命令確認您的訂用帳戶已註冊使用 Azure 復原服務提供者和 Site Recovery：

    `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.RecoveryServices` `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.SiteRecovery`

4. 確認 **RegistrationState** 在命令輸出中設為 [已註冊]，您可以繼續執行步驟 2。 如果未設定，請執行下列命令來註冊訂用帳戶中遺漏的提供者：

    `Register-AzureRmResourceProvider -ProviderNamespace Microsoft.SiteRecovery` `Register-AzureRmResourceProvider -ProviderNamespace Microsoft.RecoveryServices`

5. 確認提供者成功使用下列命令註冊：

    `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.RecoveryServices` `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.SiteRecovery`.

## <a name="step-2-set-up-the-vault"></a>步驟 2：設定保存庫

1. 建立將在其中建立保存庫的 Azure Resource Manager 資源群組，或使用現有的資源群組。 建立新的資源群組，如下所示。 $ResourceGroupName 變數包含您想要建立的資源群組名稱，而 $Geo 變數包含要在其中建立資源群組的 Azure 區域 (例如：巴西南部)。

    `New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Geo` 

2. 若要取得您的訂用帳戶中的資源群組清單，請執行 **Get-AzureRmResourceGroup** Cmdlet。
2. 建立新的 Azure 復原服務保存庫，如下所示：

        $vault = New-AzureRmRecoveryServicesVault -Name <string> -ResourceGroupName <string> -Location <string>

    您可以使用 **Get-AzureRmRecoveryServicesVault** Cmdlet 擷取現有保存庫清單。


## <a name="step-3-set-the-recovery-services-vault-context"></a>步驟 3：設定復原服務保存庫內容

設定保存庫內容，如下所示：

`Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault`

## <a name="step-4-create-a-hyper-v-site"></a>步驟 4：建立 Hyper-V 網站

1. 建立新的 Hyper-V 站台，如下所示：

        $sitename = "MySite"                #Specify site friendly name
        New-AzureRmSiteRecoverySite -Name $sitename

2. 這個 Cmdlet 會啟動 Site Recovery 作業來建立網站，並傳回 Site Recovery 作業物件。 等待作業完成，並確認作業已成功完成。
3. 使用**Get-AzureRmSiteRecoveryJob Cmdlet** 以擷取作業物件，並檢查作業的目前狀態。
4. 產生並下載網站的註冊金鑰，如下所示：

    ```
    $SiteIdentifier = Get-AzureRmSiteRecoverySite -Name $sitename | Select -ExpandProperty SiteIdentifier
        Get-AzureRmRecoveryServicesVaultSettingsFile -Vault $vault -SiteIdentifier $SiteIdentifier -SiteFriendlyName $sitename -Path $Path
    ```

5. 將下載的金鑰複製到 Hyper-V 主機。 您需要金鑰向網站註冊 Hyper-V 主機。

## <a name="step-5-install-the-provider-and-agent"></a>步驟 5：安裝提供者和代理程式

1. 從 [Microsoft](https://aka.ms/downloaddra)下載最新版提供者的安裝程式。
2. 在 Hyper-V 主機上執行安裝程式。
3. 在安裝結尾繼續註冊步驟。
4. 當系統提示時，請提供下載金鑰，並完成 Hyper-V 主機註冊。
5. 確認 Hyper-V 主機向網站註冊，如下所示：

        $server =  Get-AzureRmSiteRecoveryServer -FriendlyName $server-friendlyname

## <a name="step-6-create-a-replication-policy"></a>步驟 6：建立複寫原則

啟動前請注意，指定的儲存體帳戶應與保存庫位於相同的 Azure 區域中，而且應該啟用異地複寫。

1. 建立複寫原則，如下所示︰

        $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
        $PolicyName = “replicapolicy”
        $Recoverypoints = 6                    #specify the number of recovery points
        $storageaccountID = Get-AzureRmStorageAccount -Name "mystorea" -ResourceGroupName "MyRG" | Select -ExpandProperty Id

        $PolicyResult = New-AzureRmSiteRecoveryPolicy -Name $PolicyName -ReplicationProvider “HyperVReplicaAzure” -ReplicationFrequencyInSeconds $ReplicationFrequencyInSeconds  -RecoveryPoints $Recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId $storageaccountID

2. 請檢查傳回的作業，以確保複寫原則建立成功。

3. 擷取對應至網站的保護容器，如下所示：

        $protectionContainer = Get-AzureRmSiteRecoveryProtectionContainer
3. 建立保護容器與複寫原則的關聯，如下所示：

     $Policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $PolicyName   $associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy $Policy -PrimaryProtectionContainer $protectionContainer

4. 等到關聯工作順利完成。

## <a name="step-7-enable-vm-protection"></a>步驟 7：啟用 VM 保護

1. 擷取對應至您想要保護之 VM 的保護實體，如下所示：

        $VMFriendlyName = "Fabrikam-app"                    #Name of the VM
        $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName
2. 保護 VM。 如果您要保護的 VM 有多個連接到它的磁碟，請使用 OSDiskName  參數指定作業系統磁碟。

        $Ostype = "Windows"                                 # "Windows" or "Linux"
        $DRjob = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionEntity -Policy $Policy -Protection Enable -RecoveryAzureStorageAccountId $storageaccountID  -OS $OStype -OSDiskName $protectionEntity.Disks[0].Name

3. 在初始複寫後，等待 VM 達到受保護的狀態。 所需時間長短，受到要複寫的資料量和可用的 Azure 上游頻寬等因素影響。 達到受保護的狀態時，作業的 State 和 StateDescription 就會更新，如下所示： 

        PS C:\> $DRjob = Get-AzureRmSiteRecoveryJob -Job $DRjob

        PS C:\> $DRjob | Select-Object -ExpandProperty State
        Succeeded

        PS C:\> $DRjob | Select-Object -ExpandProperty StateDescription
        Completed
4. 更新 VM 角色大小等復原屬性，以及在容錯移轉後要附加 VM NIC 的 Azure 網路。

        PS C:\> $nw1 = Get-AzureRmVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

        PS C:\> $VMFriendlyName = "Fabrikam-App"

        PS C:\> $VM = Get-AzureRmSiteRecoveryVM -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

        PS C:\> $UpdateJob = Set-AzureRmSiteRecoveryVM -VirtualMachine $VM -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

        PS C:\> $UpdateJob = Get-AzureRmSiteRecoveryJob -Job $UpdateJob

        PS C:\> $UpdateJob

        Name             : b8a647e0-2cb9-40d1-84c4-d0169919e2c5
        ID               : /Subscriptions/a731825f-4bf2-4f81-a611-c331b272206e/resourceGroups/MyRG/providers/Microsoft.RecoveryServices/vault
                           s/MyVault/replicationJobs/b8a647e0-2cb9-40d1-84c4-d0169919e2c5
        Type             : Microsoft.RecoveryServices/vaults/replicationJobs
        JobType          : UpdateVmProperties
        DisplayName      : Update the virtual machine
        ClientRequestId  : 805a22a3-be86-441c-9da8-f32685673112-2015-12-10 17:55:51Z-P
        State            : Succeeded
        StateDescription : Completed
        StartTime        : 10-12-2015 17:55:53 +00:00
        EndTime          : 10-12-2015 17:55:54 +00:00
        TargetObjectId   : 289682c6-c5e6-42dc-a1d2-5f9621f78ae6
        TargetObjectType : ProtectionEntity
        TargetObjectName : Fabrikam-App
        AllowedActions   : {Restart}
        Tasks            : {UpdateVmPropertiesTask}
        Errors           : {}



## <a name="step-8-run-a-test-failover"></a>步驟 8：執行測試容錯移轉
1. 執行測試容錯移轉，如下所示：

        $nw = Get-AzureRmVirtualNetwork -Name "TestFailoverNw" -ResourceGroupName "MyRG" #Specify Azure vnet name and resource group

        $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -FriendlyName $VMFriendlyName -ProtectionContainer $protectionContainer

        $TFjob = Start-AzureRmSiteRecoveryTestFailoverJob -ProtectionEntity $protectionEntity -Direction PrimaryToRecovery -AzureVMNetworkId $nw.Id
2. 確認已在 Azure 中建立測試 VM。 在 Azure 中建立測試 VM 之後，測試容錯移轉作業已經暫停。
3. 若要清除和完成測試容錯移轉，請執行：

        $TFjob = Resume-AzureRmSiteRecoveryJob -Job $TFjob

## <a name="next-steps"></a>後續步驟
[深入了解](https://docs.microsoft.com/powershell/module/azurerm.siterecovery) 使用 Azure Resource Manager PowerShell Cmdlet 進行 Azure Site Recovery 的相關資訊。
