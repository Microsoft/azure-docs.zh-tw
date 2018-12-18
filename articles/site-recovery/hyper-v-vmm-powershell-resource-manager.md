---
title: 使用 PowerShell (Azure Resource Manager) 將 Virtual Machine Manager 雲端中的 Hyper-V VM 複寫至次要網站 | Microsoft Docs
description: 說明如何使用 PowerShell (Resource Manager) 將 Virtual Machine Manager 雲端中的 Hyper-V VM 複寫至次要 Virtual Machine Manager 網站
services: site-recovery
author: sujaytalasila
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: sutalasi
ms.openlocfilehash: 0fecc7ba48daf396c3d25969cdda5891bdf08232
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37917960"
---
# <a name="replicate-hyper-v-vms-to-a-secondary-site-by-using-powershell-resource-manager"></a>使用 PowerShell (Resource Manager) 將 Hyper-V VM 複寫至次要網站

本文說明如何自動執行相關步驟，使用 [Azure Site Recovery](site-recovery-overview.md) 將 System Center Virtual Machine Manager 雲端中的 Hyper-V VM 複寫至次要內部部署網站中的 Virtual Machine Manager 雲端。

## <a name="prerequisites"></a>先決條件

- 檢閱[案例架構和元件](hyper-v-vmm-architecture.md)。
- 檢閱所有元件的[支援需求](site-recovery-support-matrix-to-sec-site.md)。
- 請確定 Virtual Machine Manager 伺服器和 Hyper-V 主機符合[支援需求](site-recovery-support-matrix-to-sec-site.md)。
- 確認您要複寫的 VM 符合[複寫的機器支援](site-recovery-support-matrix-to-sec-site.md)


## <a name="prepare-for-network-mapping"></a>準備網路對應

[網路對應](hyper-v-vmm-network-mapping.md)會在來源與目標雲端中的內部部署 Virtual Machine Manager VM 網路之間對應。 對應具有下列功能：

- 在容錯移轉之後將 VM 連線至適當的目標 VM 網路。 
- 以最佳方式將複本 VM 放在目標 Hyper-V 主機伺服器上。 
- 如果您未設定網路對應，複本 VM 將不會在容錯移轉之後連線到 VM 網路。

根據下列指示準備 Virtual Machine Manager：

* 確定您的來源和目標 Virtual Machine Manager 伺服器上有 [Virtual Machine Manager 邏輯網路](https://docs.microsoft.com/system-center/vmm/network-logical)：

    - 來源伺服器上的邏輯網路應該與 Hyper-V 主機所在的來源雲端相關聯。
    - 目標伺服器上的邏輯網路應該與目標雲端相關聯。
* 確定您的來源和目標 Virtual Machine Manager 伺服器上有 [ 邏輯網路](https://docs.microsoft.com/system-center/vmm/network-virtual)。 VM 網路應連結至每個位置的邏輯網路。
* 將來源 Hyper-V 主機上的 VM 連線至來源 VM 網路。 

## <a name="prepare-for-powershell"></a>準備 PowerShell

確定 Azure PowerShell 已經準備就緒：

- 如果您已經使用 PowerShell，請升級至 0.8.10 版或更新版本。 [深入了解](/powershell/azureps-cmdlets-docs)如何設定 PowerShell。
- 在您安裝並設定 PowerShell 後，請檢閱[服務 Cmdlet](/powershell/azure/overview)。
- 若要深入了解如何在 PowerShell 中使用參數值、輸入和輸出，請閱讀[開始使用](/powershell/azure/get-started-azureps)指南。

## <a name="set-up-a-subscription"></a>設定訂用帳戶
1. 從 PowerShell，登入您的 Azure 帳戶。

        $UserName = "<user@live.com>"
        $Password = "<password>"
        $SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
        $Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
        Connect-AzureRmAccount #-Credential $Cred
2. 使用訂用帳戶識別碼擷取您的訂用帳戶清單。 記下要建立復原服務保存庫的訂用帳戶識別碼。 

        Get-AzureRmSubscription
3. 進行訂用帳戶的保存庫設定。

        Set-AzureRmContext –SubscriptionID <subscriptionId>

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫
1. 如果您沒有 Azure Resource Manager 資源群組，請建立一個。

        New-AzureRmResourceGroup -Name #ResourceGroupName -Location #location
2. 建立新的復原服務保存庫。 將保存庫物件儲存在稍後會用到的變數中。 

        $vault = New-AzureRmRecoveryServicesVault -Name #vaultname -ResouceGroupName #ResourceGroupName -Location #location
   
    您可以在保存庫物件建立後，使用 Get-AzureRMRecoveryServicesVault 加以擷取。

## <a name="set-the-vault-context"></a>設定保存庫內容
1. 擷取現有的保存庫。

       $vault = Get-AzureRmRecoveryServicesVault -Name #vaultname
2. 設定保存庫內容。

       Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault

## <a name="install-the-site-recovery-provider"></a>安裝 Site Recovery 提供者
1. 在 Virtual Machine Manager 機器上，執行下列命令來建立目錄：

       New-Item c:\ASR -type directory
2. 使用下載的提供者安裝檔案將檔案解壓縮。

       pushd C:\ASR\
       .\AzureSiteRecoveryProvider.exe /x:. /q
3. 安裝提供者，並等待安裝完成。

       .\SetupDr.exe /i
       $installationRegPath = "hklm:\software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
       do
       {
         $isNotInstalled = $true;
         if(Test-Path $installationRegPath)
         {
           $isNotInstalled = $false;
         }
       }While($isNotInstalled)

4. 在保存庫中註冊伺服器。

       $BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
       pushd $BinPath
       $encryptionFilePath = "C:\temp\".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice

## <a name="create-and-associate-a-replication-policy"></a>建立複寫原則並產生關聯
1. 建立複寫原則 (在此案例中，是針對 Hyper-V 2012 R2 建立)，如下所示︰

        $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
        $PolicyName = “replicapolicy”
        $RepProvider = HyperVReplica2012R2
        $Recoverypoints = 24                    #specify the number of hours to retain recovery pints
        $AppConsistentSnapshotFrequency = 4 #specify the frequency (in hours) at which app consistent snapshots are taken
        $AuthMode = "Kerberos"  #options are "Kerberos" or "Certificate"
        $AuthPort = "8083"  #specify the port number that will be used for replication traffic on Hyper-V hosts
        $InitialRepMethod = "Online" #options are "Online" or "Offline"

        $policyresult = New-AzureRmSiteRecoveryPolicy -Name $policyname -ReplicationProvider $RepProvider -ReplicationFrequencyInSeconds $Replicationfrequencyinseconds -RecoveryPoints $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours $AppConsistentSnapshotFrequency -Authentication $AuthMode -ReplicationPort $AuthPort -ReplicationMethod $InitialRepMethod

    > [!NOTE]
    > Virtual Machine Manager 雲端可以包含執行不同 Windows Server 版本的 Hyper-V 主機，但複寫原則僅適用於特定版本的作業系統。 如果您有執行不同作業系統的主機，請針對每個系統建立不同的複寫原則。 例如︰如果您有五部主機在 Windows Server 2012 上執行，有三部主機在 Windows Server 2012 R2 上執行，請建立兩個複寫原則。 您可以為每種類型的作業系統各建立一個。

2. 擷取主要保護容器 (主要 Virtual Machine Manager 雲端) 和復原保護容器 (復原 Virtual Machine Manager 雲端)。

       $PrimaryCloud = "testprimarycloud"
       $primaryprotectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloud;  

       $RecoveryCloud = "testrecoverycloud"
       $recoveryprotectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $RecoveryCloud;  
3. 使用好記的名稱，擷取您所建立的複寫原則。

       $policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $policyname
4. 開始建立保護容器 (Virtual Machine Manager 雲端) 與複寫原則的關聯。

       $associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy     $Policy -PrimaryProtectionContainer $primaryprotectionContainer -RecoveryProtectionContainer $recoveryprotectionContainer
5. 等候原則關聯工作完成。 若要檢查工作是否完成，請使用下列 PowerShell 程式碼片段：

       $job = Get-AzureRmSiteRecoveryJob -Job $associationJob

       if($job -eq $null -or $job.StateDescription -ne "Completed")
       {
         $isJobLeftForProcessing = $true;
       }

6. 完成工作處理之後，執行下列命令：

       if($isJobLeftForProcessing)
       {
         Start-Sleep -Seconds 60
       }
       }While($isJobLeftForProcessing)

若要檢查作業是否完成，請執行[監視活動](#monitor-activity)中的步驟。

##  <a name="configure-network-mapping"></a>設定網路對應
1. 此命令擷取目前保存庫的伺服器。 此命令會將 Site Recovery 伺服器儲存在 $Servers 陣列變數中。

        $Servers = Get-AzureRmSiteRecoveryServer
2. 執行此命令來擷取來源 Virtual Machine Manager 伺服器和目標 Virtual Machine Manager 伺服器的網路。

        $PrimaryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[0]        

        $RecoveryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[1]

    > [!NOTE]
    > 來源 Virtual Machine Manager 伺服器在伺服器陣列中可以是第一部或第二部伺服器。 檢查 Virtual Machine Manager 伺服器名稱，並適當地擷取網路。


3. 此 Cmdlet 會在主要網路與復原網路之間建立對應。 其會將主要網路指定為 $PrimaryNetworks 的第一個元素。 其會將復原網路指定為 $RecoveryNetworks 的第一個元素。

        New-AzureRmSiteRecoveryNetworkMapping -PrimaryNetwork $PrimaryNetworks[0] -RecoveryNetwork $RecoveryNetworks[0]


## <a name="enable-protection-for-vms"></a>為 VM 啟用保護
正確設定伺服器、雲端和網路後，就可以對雲端中的 VM 啟用保護。

1. 若要啟用保護，請執行下列命令以擷取保護容器：

          $PrimaryProtectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloudName
2. 依照下列方式取得保護實體 (VM)：

           $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -friendlyName $VMName -ProtectionContainer $PrimaryProtectionContainer
3. 啟用 VM 複寫。

          $jobResult = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionentity -Protection Enable -Policy $policy

## <a name="run-a-test-failover"></a>執行測試容錯移轉

若要測試您的部署，請針對單一虛擬機器執行測試容錯移轉。 您也可以建立包含多個 VM 的復原計劃，並針對計劃執行容錯移轉。 測試容錯移轉會在隔離的網路中模擬您的容錯移轉與復原機制。

1. 擷取作為 VM 容錯移轉目標的 VM。

       $Servers = Get-AzureRmSiteRecoveryServer
       $RecoveryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[1]

2. 執行測試容錯移轉。

   針對單一 VM：

        $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -FriendlyName $VMName -ProtectionContainer $PrimaryprotectionContainer

        $jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -VMNetwork $RecoveryNetworks[1]
    
   針對復原計劃：

        $recoveryplanname = "test-recovery-plan"

        $recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

        $jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -Recoveryplan $recoveryplan -VMNetwork $RecoveryNetworks[1]

若要檢查作業是否完成，請執行[監視活動](#monitor-activity)中的步驟。

## <a name="run-planned-and-unplanned-failovers"></a>執行計劃性或非計劃性容錯移轉

1. 執行計劃性容錯移轉。

   針對單一 VM：

        $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

        $jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity

   針對復原計劃：

        $recoveryplanname = "test-recovery-plan"

        $recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

        $jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -Recoveryplan $recoveryplan

2. 執行非計劃性容錯移轉。

   針對單一 VM：
        
        $protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

        $jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity

   針對復原計劃：

        $recoveryplanname = "test-recovery-plan"

        $recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

        $jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity

## <a name="monitor-activity"></a>監視活動
使用下列命令監視容錯移轉活動。 等候作業之間的處理完成。

    Do
    {
        $job = Get-AzureSiteRecoveryJob -Id $associationJob.JobId;
        Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
        if($job -eq $null -or $job.StateDescription -ne "Completed")
        {
            $isJobLeftForProcessing = $true;
        }

    if($isJobLeftForProcessing)
        {
            Start-Sleep -Seconds 60
        }
    }While($isJobLeftForProcessing)



## <a name="next-steps"></a>後續步驟

[深入了解](/powershell/module/azurerm.recoveryservices.backup/#recovery)使用 Resource Manager PowerShell Cmdlet 進行 Site Recovery 的相關資訊。
