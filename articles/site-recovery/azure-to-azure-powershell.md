---
title: Azure Site Recovery - 使用 Azure PowerShell 安裝和測試 Azure 虛擬機器的災害復原 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 與 Azure Site Recovery 來設定 Azure 虛擬機器的災害復原。
services: site-recovery
author: bsiva
manager: abhemraj
editor: raynew
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: bsiva
ms.openlocfilehash: 62dd02d53c14635a386a8c6fa3fbfbd6f91a88f7
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37921081"
---
# <a name="set-up-disaster-recovery-for-azure-virtual-machines-using-azure-powershell"></a>使用 Azure PowerShell 來設定 Azure 虛擬機器的災害復原

在本文中，您會了解如何使用 Azure PowerShell 來安裝和測試 Azure 虛擬機器的災害復原。 

您會了解如何：

> [!div class="checklist"]
> - 建立復原服務保存庫。
> - 設定 PowerShell 工作階段的保存庫內容。
> - 讓保存庫準備好開始複寫 Azure 虛擬機器。
> - 建立網路對應。
> - 建立作為複寫虛擬機器目的地的儲存體帳戶。
> - 將 Azure 虛擬機器複寫到復原區域以供災害復原之用。
> - 執行測試容錯移轉、驗證和清除測試容錯移轉。
> - 容錯移轉至復原區域。

> [!NOTE]
> 可透過入口網站使用的案例功能，並非都能透過 Azure PowerShell 來使用。 目前不支援透過 Azure PowerShell 來使用的部分案例功能包括：
> - 能夠複寫使用受控磁碟的 Azure 虛擬機器。
> - 能夠指定應該複寫虛擬機器中的所有磁碟，而不需要明確指定虛擬機器的每個磁碟。  

## <a name="prerequisites"></a>先決條件

開始之前：
- 請確定您了解[情節架構和元件](azure-to-azure-architecture.md)。
- 檢閱所有元件的[支援需求](azure-to-azure-support-matrix.md)。
- 您有 AzureRm PowerShell 模組的 5.7.0 版或更高版本。 如果您需要安裝或升級 Azure PowerShell，請按照此[安裝和設定 Azure PowerShell 指南](/powershell/azureps-cmdlets-docs)的說明。

## <a name="log-in-to-your-microsoft-azure-subscription"></a>登入您的 Microsoft Azure 訂用帳戶

使用 Connect-AzureRmAccount Cmdlet 登入您的 Azure 訂用帳戶

```azurepowershell
Connect-AzureRmAccount
```
選取 Azure 訂用帳戶。 您可以使用 Get AzureRmSubscription Cmdlet 取得您有存取權的 Azure 訂用帳戶的清單。 使用 Select-AzureRmSubscription Cmdlet 選取要使用的 Azure 訂用帳戶。

```azurepowershell
Select-AzureRmSubscription -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

## <a name="get-details-of-the-virtual-machines-to-be-replicated"></a>取得要複寫的虛擬機器詳細資料

在本文範例中，「美國東部」區域的虛擬機器會複寫並復原到「美國西部 2」區域。 所要複寫的虛擬機器是具有 OS 磁碟和單一資料磁碟的虛擬機器。 此範例所使用的虛擬機器名稱是 AzureDemoVM。

```azurepowershell
# Get details of the virtual machine
$VM = Get-AzureRmVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

Write-Output $VM
```

```
ResourceGroupName  : A2AdemoRG
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
Name               : AzureDemoVM
Type               : Microsoft.Compute/virtualMachines
Location           : eastus
Tags               : {}
DiagnosticsProfile : {BootDiagnostics}
HardwareProfile    : {VmSize}
NetworkProfile     : {NetworkInterfaces}
OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState  : Succeeded
StorageProfile     : {ImageReference, OsDisk, DataDisks}
```

取得虛擬機器磁碟的磁碟詳細資料。 稍後在開始複寫虛擬機器時，會用到磁碟詳細資料。

```azurepowershell
$OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
$DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
```

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

建立要在其中建立復原服務保存庫的資源群組。 

> [!IMPORTANT]
> * 復原服務保存庫和受保護的虛擬機器必須位於不同的 Azure 位置。
> * 復原服務保存庫的資源群組和受保護的虛擬機器必須位於不同的 Azure 位置。
> * 復原服務保存庫和其所屬的資源群組可位於相同的 Azure 位置。
 
在本文範例中，受保護的虛擬機器位於「美國東部」區域。 針對災害復原所選取的復原區域為「美國西部 2」區域。 復原服務保存庫和保存庫的資源群組都位於復原區域 (美國西部 2)

```azurepowershell
#Create a resource group for the recovery services vault in the recovery Azure region
New-AzureRmResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"
```
```
ResourceGroupName : a2ademorecoveryrg
Location          : westus2
ProvisioningState : Succeeded
Tags              : 
ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg
```
   
建立復原服務保存庫。 下列範例會在「美國西部 2」區域建立名為 a2aDemoRecoveryVault 的復原服務保存庫。

```azurepowershell
#Create a new Recovery services vault in the recovery region
$vault = New-AzureRmRecoveryServicesVault -Name "a2aDemoRecoveryVault" -ResourceGroupName "a2ademorecoveryrg" -Location "West US 2"

Write-Output $vault
```
```
Name              : a2aDemoRecoveryVault
ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoRecoveryVault
Type              : Microsoft.RecoveryServices/vaults
Location          : westus2
ResourceGroupName : a2ademorecoveryrg
SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
``` 
## <a name="set-the-vault-context"></a>設定保存庫內容

> [!TIP]
> Azure Site Recovery PowerShell 模組 (AzureRm.RecoveryServices.SiteRecovery 模組) 提供對於大多數 Cmdlet 容易使用的別名。 模組中的 Cmdlet 採用 *\<Operation>-**AzureRmRecoveryServicesAsr**\<Object>* 的形式，並且有形式為 *\<Operation>-**ASR**\<Object>* 的對等別名。 本文使用 Cmdlet 別名提高可讀性。

設定要用於 PowerShell 工作階段的保存庫內容。 若要這樣做，請下載保存庫設定檔，並在 PowerShell 工作階段中匯入所下載的檔案來設定保存庫內容。 

設定後，會在所選保存庫的內容中執行 PowerShell 工作階段中的後續 Azure Site Recovery 作業。 

 ```azurepowershell
#Download the vault settings file for the vault.
$Vaultsettingsfile = Get-AzureRmRecoveryServicesVaultSettingsFile -Vault $vault -SiteRecovery -Path C:\users\user\Documents\

#Import the downloaded vault settings file to set the vault context for the PowerShell session.
Import-AzureRmRecoveryServicesAsrVaultSettingsFile -Path $Vaultsettingsfile.FilePath

```
```
ResourceName         ResourceGroupName ResourceNamespace          ResouceType
------------         ----------------- -----------------          -----------
a2aDemoRecoveryVault a2ademorecoveryrg Microsoft.RecoveryServices Vaults     
```

```azurepowershell
#Delete the downloaded vault settings file
Remove-Item -Path $Vaultsettingsfile.FilePath
```
## <a name="prepare-the-vault-to-start-replicating-azure-virtual-machines"></a>讓保存庫準備好開始複寫 Azure 虛擬機器

####<a name="1-create-a-site-recovery-fabric-object-to-represent-the-primarysource-region"></a>1.建立 Site Recovery 網狀架構物件來代表主要 (來源) 區域

保存庫中的網狀架構物件可代表 Azure 區域。 建立目的是要代表保存庫所保護虛擬機器所屬 Azure 區域的網狀架構物件，便是主要的網狀架構物件。 在本文範例中，受保護的虛擬機器位於「美國東部」區域。

> [!NOTE]
> Azure Site Recovery 作業會以非同步方式執行。 您開始作業時，會提交 Azure Site Recovery 作業，並傳回作業追蹤物件。 請使用作業追蹤物件，來取得作業的最新狀態 (Get-ASRJob)，以及監視作業的狀態。

```azurepowershell
#Create Primary ASR fabric
$TempASRJob = New-ASRFabric -Azure -Location 'East US'  -Name "A2Ademo-EastUS" 

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State

$PrimaryFabric = Get-AsrFabric -Name "A2Ademo-EastUS"
```
如果會由相同的保存庫保護多個 Azure 區域的虛擬機器，請為每個來源 Azure 區域各建立一個網狀架構物件。

####<a name="2-create-a-site-recovery-fabric-object-to-represent-the-recovery-region"></a>2.建立 Site Recovery 網狀架構物件來代表復原區域

復原網狀架構物件可代表復原 Azure 位置。 虛擬機器會複寫並復原到 (在發生容錯移轉時) 復原網狀架構所代表的復原區域。 此範例所使用的復原 Azure 區域是「美國西部 2」。

```azurepowershell
#Create Recovery ASR fabric
$TempASRJob = New-ASRFabric -Azure -Location 'West US 2'  -Name "A2Ademo-WestUS" 

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State

$RecoveryFabric = Get-AsrFabric -Name "A2Ademo-WestUS"

```

####<a name="3-create-a-site-recovery-protection-container-in-the-primary-fabric"></a>3.在主要網狀架構中建立 Site Recovery 保護容器

保護容器是用來將複寫的項目群組到網狀架構內的容器。

```azurepowershell
#Create a Protection container in the primary Azure region (within the Primary fabric)
$TempASRJob = New-AzureRmRecoveryServicesAsrProtectionContainer -InputObject $PrimaryFabric -Name "A2AEastUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

Write-Output $TempASRJob.State

$PrimaryProtContainer = Get-ASRProtectionContainer -Fabric $PrimaryFabric -Name "A2AEastUSProtectionContainer"
```
####<a name="4-create-a-site-recovery-protection-container-in-the-recovery-fabric"></a>4.在復原網狀架構中建立 Site Recovery 保護容器

```azurepowershell
#Create a Protection container in the recovery Azure region (within the Recovery fabric)
$TempASRJob = New-AzureRmRecoveryServicesAsrProtectionContainer -InputObject $RecoveryFabric -Name "A2AWestUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"

Write-Output $TempASRJob.State

$RecoveryProtContainer = Get-ASRProtectionContainer -Fabric $RecoveryFabric -Name "A2AWestUSProtectionContainer"
```

####<a name="5-create-a-replication-policy"></a>5.建立複寫原則

```azurepowershell
#Create replication policy
$TempASRJob = New-ASRPolicy -AzureToAzure -Name "A2APolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State

$ReplicationPolicy = Get-ASRPolicy -Name "A2APolicy"
```
####<a name="6-create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container"></a>6.在主要和復原保護容器之間建立保護容器對應

保護容器對應會使用復原保護容器和複寫原則來對應主要的保護容器。 請針對每個會用來在保護容器組之間複寫虛擬機器的複寫原則，各建立一個對應。

```azurepowershell
#Create Protection container mapping between the Primary and Recovery Protection Containers with the Replication policy
$TempASRJob = New-ASRProtectionContainerMapping -Name "A2APrimaryToRecovery" -Policy $ReplicationPolicy -PrimaryProtectionContainer $PrimaryProtContainer -RecoveryProtectionContainer $RecoveryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State

$EusToWusPCMapping = Get-ASRProtectionContainerMapping -ProtectionContainer $PrimaryProtContainer -Name "A2APrimaryToRecovery"
```

####<a name="7-create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover"></a>7.建立用於容錯回復 (在容錯移轉之後反向複寫) 的保護容器對應

在容錯移轉後，您可在準備好讓已容錯移轉的虛擬機器回復到原始 Azure 區域時進行容錯回復。 為了進行容錯回復，系統會將已容錯移轉的虛擬機器，從容錯移轉後的區域反向複寫到原始區域。 在反向複寫時，原始區域和復原區域的角色會對調。 原始區域現在會變成新的復原區域，原本的復原區域現在則會變成主要區域。 反向複寫的保護容器對應會表示原始區域和復原區域的對調角色。

```azurepowershell
#Create Protection container mapping (for failback) between the Recovery and Primary Protection Containers with the Replication policy 
$TempASRJob = New-ASRProtectionContainerMapping -Name "A2ARecoveryToPrimary" -Policy $ReplicationPolicy -PrimaryProtectionContainer $RecoveryProtContainer -RecoveryProtectionContainer $PrimaryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State
```

## <a name="create-cache-storage-accounts-and-target-storage-accounts"></a>建立快取儲存體帳戶和目標儲存體帳戶

快取儲存體帳戶是和所複寫虛擬機器位於相同 Azure 區域的標準儲存體帳戶。 快取儲存體帳戶則會用來在變更移至復原 Azure 區域前，暫時保存複寫變更。 您可以選擇 (但並非必要) 為虛擬機器的不同磁碟指定不同的快取儲存體帳戶。

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzureRmStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

對於未使用受控磁碟的虛擬機器，目標儲存體帳戶是復原區域中要作為虛擬機器磁碟複寫目的地的儲存體帳戶。 目標儲存體帳戶可以是標準儲存體帳戶，也可以是進階儲存體帳戶。 請根據磁碟的資料變更率 (IO 寫入速率) 以及 Azure Site Recovery 針對儲存體類型所支援的變換限制，來選取所需的儲存體帳戶種類。

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzureRmStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage

```

## <a name="create-network-mappings"></a>建立網路對應

網路對應會將主要區域的虛擬網路，對應至復原區域的虛擬網路。 網路對應會指定復原區域中的 Azure 虛擬網路，而主要虛擬網路中的虛擬機器便應容錯移轉至此網路。 一個 Azure 虛擬網路只能對應至復原區域中的單一 Azure 虛擬網路。

- 在復原區域中建立要作為容錯移轉目的地的 Azure 虛擬網路

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzureRmVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzureRmVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzureRmVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```
- 擷取主要虛擬網路 (虛擬機器所連線的 vnet)
   ```azurepowershell
    #Retrieve the virtual network that the virtual machine is connected to
    
    #Get first network interface card(nic) of the virtual machine
    $SplitNicArmId = $VM.NetworkProfile.NetworkInterfaces[0].Id.split("/")

    #Extract resource group name from the ResourceId of the nic
    $NICRG = $SplitNicArmId[4]

    #Extract resource name from the ResourceId of the nic
    $NICname = $SplitNicArmId[-1]

    #Get network interface details using the extracted resource group name and resourec name
    $NIC = Get-AzureRmNetworkInterface -ResourceGroupName $NICRG -Name $NICname

    #Get the subnet ID of the subnet that the nic is connected to
    $PrimarySubnet = $NIC.IpConfigurations[0].Subnet

    # Extract the resource ID of the Azure virtual network the nic is connected to from the subnet ID
    $EastUSPrimaryNetwork = (Split-Path(Split-Path($PrimarySubnet.Id))).Replace("\","/")  
   ```
- 在主要虛擬網路與復原虛擬網路之間建立網路對應
   ```azurepowershell
    #Create an ASR network mapping between the primary Azure virtual network and the recovery Azure virtual network
    $TempASRJob = New-ASRNetworkMapping -AzureToAzure -Name "A2AEusToWusNWMapping" -PrimaryFabric $PrimaryFabric -PrimaryAzureNetworkId $EastUSPrimaryNetwork -RecoveryFabric $RecoveryFabric -RecoveryAzureNetworkId $WestUSRecoveryNetwork
    
    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
            sleep 10; 
            $TempASRJob = Get-ASRJob -Job $TempASRJob
    }
    
    #Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    
   ```
- 建立反方向 (容錯回復) 的網路對應
    ```azurepowershell
    #Create an ASR network mapping for failback between the recovery Azure virtual network and the primary Azure virtual network
    $TempASRJob = New-ASRNetworkMapping -AzureToAzure -Name "A2AWusToEusNWMapping" -PrimaryFabric $RecoveryFabric -PrimaryAzureNetworkId $WestUSRecoveryNetwork -RecoveryFabric $PrimaryFabric -RecoveryAzureNetworkId $EastUSPrimaryNetwork
    
    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
            sleep 10; 
            $TempASRJob = Get-ASRJob -Job $TempASRJob
    }
        
    #Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    ```

## <a name="replicate-azure-virtual-machine"></a>複寫 Azure 虛擬機器

請複寫 Azure 虛擬機器。

```azurepowershell
#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)

#Disk replication configuration for the OS disk
$OSDiskReplicationConfig = New-AzureRmRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $OSDiskVhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Disk replication configuration for data disk
$DataDisk1ReplicationConfig = New-AzureRmRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $DataDisk1VhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzureRmResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.  
$TempASRJob = New-ASRReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){ 
        sleep 10; 
        $TempASRJob = Get-ASRJob -Job $TempASRJob
}


#Check if the Job completed successfully. The updated job state of a successfuly completed job should be "Succeeded"
Write-Output $TempASRJob.State
```

複寫作業啟動成功後，系統就會將虛擬機器資料複寫至復原區域。 

複寫程序一開始會先在復原區域中植入虛擬機器複寫磁碟的複本。 這個階段稱為初始複寫階段。 

初始複寫完成後，複寫會進入差異同步處理階段。 此時，系統會保護虛擬機器，供您對其執行測試容錯移轉作業。 初始複寫完成後，代表虛擬機器的複寫項目狀態會進入「受保護」狀態。

請藉由取得虛擬機器所對應的受保護複寫項目詳細資料，來監視虛擬機器的複寫狀態和複寫健康情況。
```azurepowershell
 Get-ASRReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```
```
FriendlyName ProtectionState ReplicationHealth
------------ --------------- -----------------
AzureDemoVM  Protected       Normal           
```

## <a name="perform-a-test-failover-validate-and-cleanup-test-failover"></a>執行測試容錯移轉、驗證和清除測試容錯移轉

虛擬機器的複寫到達受保護狀態後，就能對虛擬機器 (對虛擬機器的受保護複寫項目) 執行測試容錯移轉作業。

```azurepowershell
#Create a seperate network for test failover (not connected to my DR network)
$TFOVnet = New-AzureRmVirtualNetwork -Name "a2aTFOvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.3.0.0/16"

Add-AzureRmVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $TFOVnet -AddressPrefix "10.3.0.0/20" | Set-AzureRmVirtualNetwork

$TFONetwork= $TFOVnet.Id
```

執行測試容錯移轉。
```azurepowershell
$ReplicationProtectedItem = Get-ASRReplicationProtectedItem -FriendlyName "AzureDemoVM" -ProtectionContainer $PrimaryProtContainer

$TFOJob = Start-ASRTestFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -AzureVMNetworkId $TFONetwork -Direction PrimaryToRecovery


```

等待測試容錯移轉作業完成。
```azurepowershell
Get-ASRJob -Job $TFOJob
```

```
Name             : 3dcb043e-3c6d-4e0e-a42e-8d4245668547
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/3dcb043e-3c6d-4e0e-a42e-8d4245668547
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : TestFailover
DisplayName      : Test failover
ClientRequestId  : 1ef8515b-b130-4452-a44d-91aaf071931c ActivityId: 907bb2bc-ebe6-4732-8b66-77d0546eaba8
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:29:43 AM
EndTime          : 4/25/2018 4:33:06 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   : 
Tasks            : {Prerequisites check for test failover, Create test virtual machine, Preparing the virtual machine, Start the virtual machine}
Errors           : {}
```
測試容錯移轉作業成功完成後，就可以連線至測試容錯移轉虛擬機器，並驗證測試容錯移轉。

在測試容錯移轉虛擬機器上完成測試後，請啟動清除測試容錯移轉作業來清除測試複本。 此作業會刪除測試容錯移轉所建立的虛擬機器測試複本。

```azurepowershell
$Job_TFOCleanup = Start-ASRTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-ASRJob -Job $Job_TFOCleanup | Select State
```
```
State    
-----    
Succeeded
```

## <a name="failover-to-azure"></a>容錯移轉至 Azure

將虛擬機器容錯移轉至特定復原點。

```azurepowershell
$RecoveryPoints = Get-ASRRecoveryPoint -ReplicationProtectedItem $ReplicationProtectedItem

#The list of recovery points returned may not be sorted chronologically and will need to be sorted first, in order to be able to find the oldest or the latest recovery points for the virtual machine.
"{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[-1].RecoveryPointTime
```
```
CrashConsistent 4/24/2018 11:10:25 PM
```


```azurepowershell
#Start the failover job
$Job_Failover = Start-ASRUnplannedFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[-1]

do {
        $Job_Failover = Get-ASRJob -Job $Job_Failover;
        sleep 30;
} while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))

$Job_Failover.State
```
```
Succeeded
```

容錯移轉成功後，就可以認可容錯移轉作業。
```azurepowershell
$CommitFailoverJOb = Start-ASRCommitFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-ASRJOb -Job $CommitFailoverJOb
```

```
Name             : 58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : CommitFailover
DisplayName      : Commit
ClientRequestId  : 10a95d6c-359e-4603-b7d9-b7ee3317ce94 ActivityId: 8751ada4-fc42-4238-8de6-a82618408fcf
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:50:58 AM
EndTime          : 4/25/2018 4:51:01 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   : 
Tasks            : {Prerequisite check, Commit}
Errors           : {}
```

容錯移轉之後，當您準備好返回原始區域時，請使用 Update-AzureRmRecoveryServicesAsrProtectionDirection Cmdlet 啟動受保護複寫項目的反向複寫。

## <a name="next-steps"></a>後續步驟
參閱 [Azure 網站復原 PowerShell 參考](https://docs.microsoft.com/powershell/module/AzureRM.RecoveryServices.SiteRecovery)，了解如何執行其他工作，例如建立復原計劃，以及透過 PowerShell 測試復原計劃的容錯移轉。
