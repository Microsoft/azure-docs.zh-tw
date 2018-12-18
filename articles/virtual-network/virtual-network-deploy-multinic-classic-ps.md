---
title: 建立具有多個 NIC 的 VM (傳統) - Azure PowerShell | Microsoft Docs
description: 了解如何使用 PowerShell 建立具有多個 NIC 的 VM (傳統)。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: 6e50f39a-2497-4845-a5d4-7332dbc203c5
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/22/2018
ms.author: genli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ca4e9e77d0e0ca62c04fbbfe132a41fb3e01df46
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38477653"
---
# <a name="create-a-vm-classic-with-multiple-nics-using-powershell"></a>使用 PowerShell 建立具有多個 NIC 的 VM (傳統)

[!INCLUDE [virtual-network-deploy-multinic-classic-selectors-include.md](../../includes/virtual-network-deploy-multinic-classic-selectors-include.md)]

您可以在 Azure 中建立虛擬機器 (VM) 並將多個網路介面 (NIC) 連接至每個 VM。 有多個 NIC 時，可透過各個 NIC 分隔不同的流量類型。 例如，一個 NIC 可能與網際網路進行通訊，而另一個 NIC 則只與未連線到網際網路的內部資源進行通訊。 透過多個 NIC 分隔網路流量是許多網路虛擬設備 (例如應用程式交付和 WAN 最佳化解決方案) 所需的功能。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何使用 [Resource Manager 部署模型](../virtual-machines/windows/multiple-nics.md)執行這些步驟。

[!INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

在下列步驟中，WEB 伺服器使用名為 *IaaSStory* 的資源群組，而 DB 伺服器使用名為 *IaaSStory-BackEnd* 的資源群組。

## <a name="prerequisites"></a>先決條件

您需要建立 *IaaSStory* 資源群組，其中含有此案例的所有必要資源，才能建立 DB 伺服器。 若要建立這些資源，請完成下列步驟。 依照[建立虛擬網路](virtual-networks-create-vnet-classic-netcfg-ps.md)文章中的步驟建立虛擬網路。

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-the-back-end-vms"></a>建立後端 VM
後端 VM 有賴於建立下列資源：

* **後端子網路**。 資料庫伺服器會是另外的子網路的一部分，以隔離流量。 下面的指令碼需要這個子網路位在名為 *WTestVnet*的 vnet 中。
* **資料磁碟的儲存體帳戶**。 為取得更佳的效能，資料庫伺服器上的資料磁碟會使用需要進階儲存體帳戶的固態硬碟 (SSD) 技術。 請確定 Azure 的部署位置，以支援進階儲存體。
* **可用性設定組**。 所有的資料庫伺服器都會加入單一的可用性設定組，確保在維護期間至少有一部 VM 啟動並執行。

### <a name="step-1---start-your-script"></a>步驟 1：啟動指令碼
[這裡](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/virtual-network-deploy-multinic-classic-ps.ps1)可以下載所使用之完整的 PowerShell 指令碼。 請遵循下列步驟來變更指令碼來讓指令碼在環境中運作。

1. 根據上述 [必要條件](#Prerequisites)中已部署的現有資源群組來變更下列變數的值。

    ```powershell
    $location              = "West US"
    $vnetName              = "WTestVNet"
    $backendSubnetName     = "BackEnd"
    ```

2. 根據後端部署要使用的值，變更下列變數值。

    ```powershell
    $backendCSName         = "IaaSStory-Backend"
    $prmStorageAccountName = "iaasstoryprmstorage"
    $avSetName             = "ASDB"
    $vmSize                = "Standard_DS3"
    $diskSize              = 127
    $vmNamePrefix          = "DB"
    $dataDiskSuffix        = "datadisk"
    $ipAddressPrefix       = "192.168.2."
    $numberOfVMs           = 2
    ```

### <a name="step-2---create-necessary-resources-for-your-vms"></a>步驟 2：為 VM 建立必要的資源
您需要為所有 VM 的資料磁碟建立新的雲端服務和儲存體帳戶。 您也需要指定影像及 VM 的本機系統管理員帳戶。 若要建立這些資源，請完成下列步驟：

1. 建立新的雲端服務。

    ```powershell
    New-AzureService -ServiceName $backendCSName -Location $location
    ```

2. 建立新的進階儲存體帳戶。

    ```powershell
    New-AzureStorageAccount -StorageAccountName $prmStorageAccountName `
    -Location $location -Type Premium_LRS
    ```
3. 設定前文中建立的儲存體帳戶，做為訂用帳戶的目前儲存體帳戶。

    ```powershell
    $subscription = Get-AzureSubscription | where {$_.IsCurrent -eq $true}  
    Set-AzureSubscription -SubscriptionName $subscription.SubscriptionName `
    -CurrentStorageAccountName $prmStorageAccountName
    ```

4. 選取 VM 影像。

    ```powershell
    $image = Get-AzureVMImage `
    | where{$_.ImageFamily -eq "SQL Server 2014 RTM Web on Windows Server 2012 R2"} `
    | sort PublishedDate -Descending `
    | select -ExpandProperty ImageName -First 1
    ```

5. 設定本機系統管理員帳戶認證。

    ```powershell
    $cred = Get-Credential -Message "Enter username and password for local admin account"
    ```

### <a name="step-3---create-vms"></a>步驟 3：建立 VM
您需要使用迴圈建立所需數量的 VM，並在迴圈中建立必要的 NIC 和 VM。 若要建立 NIC 和 VM，請執行下列步驟。

1. 啟動 `for` 迴圈，根據 `$numberOfVMs` 變數值，視需要的次數重複命令來建立一部 VM 和兩個 NIC。

    ```powershell
    for ($suffixNumber = 1; $suffixNumber -le $numberOfVMs; $suffixNumber++){
    ```

2. 建立指定 VM 影像、大小和可用性設定組的 `VMConfig` 物件。

    ```powershell
    $vmName = $vmNamePrefix + $suffixNumber
    $vmConfig = New-AzureVMConfig -Name $vmName `
        -ImageName $image `
        -InstanceSize $vmSize `
        -AvailabilitySetName $avSetName
    ```

3. 將 VM 佈建為 Windows VM。

    ```powershell
    Add-AzureProvisioningConfig -VM $vmConfig -Windows `
        -AdminUsername $cred.UserName `
        -Password $cred.GetNetworkCredential().Password
    ```

4. 設定預設 NIC，並指派它一個靜態 IP 位址。

    ```powershell
    Set-AzureSubnet         -SubnetNames $backendSubnetName -VM $vmConfig
    Set-AzureStaticVNetIP   -IPAddress ($ipAddressPrefix+$suffixNumber+3) -VM $vmConfig
    ```

5. 為每部 VM 加入第二個 NIC。

    ```powershell
    Add-AzureNetworkInterfaceConfig -Name ("RemoteAccessNIC"+$suffixNumber) `
    -SubnetName $backendSubnetName `
    -StaticVNetIPAddress ($ipAddressPrefix+(53+$suffixNumber)) `
    -VM $vmConfig
    ```
    
6. 為每部 VM 建立資料磁碟。

    ```powershell
    $dataDisk1Name = $vmName + "-" + $dataDiskSuffix + "-1"    
    Add-AzureDataDisk -CreateNew -VM $vmConfig `
    -DiskSizeInGB $diskSize `
    -DiskLabel $dataDisk1Name `
    -LUN 0

    $dataDisk2Name = $vmName + "-" + $dataDiskSuffix + "-2"   
    Add-AzureDataDisk -CreateNew -VM $vmConfig `
    -DiskSizeInGB $diskSize `
    -DiskLabel $dataDisk2Name `
    -LUN 1
    ```

7. 建立每部 VM 並結束迴圈。

    ```powershell
    New-AzureVM -VM $vmConfig `
    -ServiceName $backendCSName `
    -Location $location `
    -VNetName $vnetName
    }
    ```

### <a name="step-4---run-the-script"></a>步驟 4：執行指令碼
既然您已根據需求下載並變更指令碼，請執行指令碼來建立有多個 NIC 的後端資料庫 VM。

1. 儲存您的指令碼，然後從 **PowerShell** 命令提示字元或 **PowerShell ISE** 執行它。 您會看到初始的輸出，如下所示。

        OperationDescription    OperationId                          OperationStatus

        New-AzureService        xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded
        New-AzureStorageAccount xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded
        
        WARNING: No deployment found in service: 'IaaSStory-Backend'.
2. 填寫認證提示中所需的資訊，並按一下 [確定] 。 會傳回下列輸出。

        New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded
        New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded

### <a name="step-5---configure-routing-within-the-vms-operating-system"></a>步驟 5 - 設定 VM 的作業系統內的路由

Azure DHCP 會將預設閘道指派給連接至虛擬機器的第一個 (主要) 網路介面。 Azure 不會將預設閘道指派給連接至虛擬機器的其他 (次要) 網路介面。 因此，依預設，您無法與次要網路介面中子網路之外的資源進行通訊。 不過，次要網路介面可與它們的子網路之外的資源通訊。 若要設定次要網路介面的路由，請參閱下列文章：

- [設有多個 NIC 的 Windows VM](../virtual-machines/windows/multiple-nics.md#configure-guest-os-for-multiple-nics
)

- [設有多個 NIC 的 Linux VM](../virtual-machines/linux/multiple-nics.md#configure-guest-os-for-multiple-nics
)
