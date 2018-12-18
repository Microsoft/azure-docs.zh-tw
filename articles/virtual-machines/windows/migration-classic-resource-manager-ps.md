---
title: 使用 PowerShell 移轉到 Resource Manager | Microsoft Docs
description: 本文逐步解說使用 Azure PowerShell 命令進行平台支援之 IaaS 資源 (例如虛擬機器 (VM)、虛擬網路 (VNET) 和儲存體帳戶) 移轉 (從傳統移轉至 Azure Resource Manager (ARM))
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 2b3dff9b-2e99-4556-acc5-d75ef234af9c
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 03/30/2017
ms.author: kasing
ms.openlocfilehash: 9cfdd6828a6d7ec699501a485519f843c59d0422
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36291923"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-powershell"></a>使用 Azure PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager
以下步驟說明如何使用 Azure PowerShell 命令，將基礎結構即服務 (IaaS) 資源從傳統部署模型移轉至 Azure Resource Manager 部署模型。

如果您想要的話，也可以使用 [Azure 命令列介面 (Azure CLI)](../linux/migration-classic-resource-manager-cli.md)來移轉資源。

* 如需了解有關支援之移轉案例的背景，請參閱 [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](migration-classic-resource-manager-overview.md)。
* 如需詳細的指導方針和移轉逐步解說，請參閱 [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](migration-classic-resource-manager-deep-dive.md)。
* [檢閱最常見的移轉錯誤](migration-classic-resource-manager-errors.md)

<br>
下列流程圖會識別在移轉程序期間執行步驟所需的順序

![Screenshot that shows the migration steps](media/migration-classic-resource-manager/migration-flow.png)

## <a name="step-1-plan-for-migration"></a>步驟 1︰為移轉做規劃
以下是您評估將 IaaS 資源從傳統移轉至 Resource Manager 時，我們所建議的一些最佳做法：

* 將 [支援及不支援的功能和組態](migration-classic-resource-manager-overview.md)看一遍。 如果您的虛擬機器使用不支援的組態或功能，建議您等到宣布支援該組態/功能之後，再進行移轉。 或者，移除該功能或移出該組態以利移轉進行 (如果這麼做符合您的需求)。
* 如果您是使用自動化指令碼來部署現今的基礎結構和應用程式，請使用這些指令碼來嘗試建立相似的測試設定以進行移轉。 或者，您也可以使用 Azure 入口網站來設定範例環境。

> [!IMPORTANT]
> 目前不支援將應用程式閘道從傳統環境移轉至 Resource Manager。 若要使用應用程式閘道來移轉傳統虛擬網路，請先移除閘道，再執行「準備」作業來移動網路。 在完成移轉之後，於 Azure Resource Manager 中重新連接閘道。
>
>如果 ExpressRoute 閘道連線至另一個訂用帳戶中的 ExpressRoute 線路，則無法自動移轉。 在這種情況下，請移除 ExpressRoute 閘道，移轉虛擬網路，然後重新建立閘道。 如需相關步驟和詳細資訊，請參閱[將 ExpressRoute 線路和相關聯的虛擬網路從傳統部署模型移轉至 Resource Manager 部署模型](../../expressroute/expressroute-migration-classic-resource-manager.md)。

## <a name="step-2-install-the-latest-version-of-azure-powershell"></a>步驟 2：安裝最新版的 Azure PowerShell
Azure PowerShell 的主要安裝選項有兩個：[PowerShell 資源庫](https://www.powershellgallery.com/profiles/azure-sdk/)或 [Web Platform Installer (WebPI)](http://aka.ms/webpi-azps)。 WebPI 接收每月更新。 PowerShell 資源庫則是持續接收更新。 本文是以 Azure PowerShell 2.1.0 為基礎。

如需安裝指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

<br>

## <a name="step-3-ensure-that-you-are-an-administrator-for-the-subscription-in-azure-portal"></a>步驟 3：確定您在 Azure 入口網站中是訂用帳戶的共同管理員
若要執行此移轉，必須在 [Azure 入口網站](https://portal.azure.com)中將您新增為訂用帳戶的共同管理員。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 [中樞] 功能表中，選取 [訂用帳戶] 。 如果您沒有看到，請選取 [所有服務]。
3. 尋找適當的訂用帳戶項目，然後查看 [我的角色] 欄位。 對於共同管理員而言，這個值應該是 [帳戶管理員]。

如果您無法新增共同管理員，請連絡服務管理員或訂用帳戶的共同管理員，以將您新增為共同管理員。   

## <a name="step-4-set-your-subscription-and-sign-up-for-migration"></a>步驟 4︰設定您的訂用帳戶並註冊以進行移轉
首先，開啟 PowerShell 提示字元。 針對移轉，您必須為傳統和 Resource Manager 模型設定您的環境。

登入您的 Resource Manager 模型帳戶。

```powershell
    Connect-AzureRmAccount
```

請使用下列命令來取得可用的訂用帳戶：

```powershell
    Get-AzureRMSubscription | Sort Name | Select Name
```

設定目前工作階段的 Azure 訂用帳戶。 這個範例會將預設訂用帳戶名稱設定為 [我的 Azure 訂用帳戶]。 將範例訂用帳戶名稱取代為您自己的名稱。

```powershell
    Select-AzureRmSubscription –SubscriptionName "My Azure Subscription"
```

> [!NOTE]
> 註冊是一次性步驟，但您必須在嘗試移轉之前完成。 如果不註冊，您會看到下列錯誤訊息：
>
> *不正確的要求︰訂用帳戶未針對移轉進行註冊。*

請使用下列命令向移轉資源提供者註冊：

```powershell
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

請等候 5 分鐘讓註冊完成。 您可以使用下列命令來檢查核准狀態：

```powershell
    Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

請先確定 RegistrationState 是 `Registered` ，再繼續進行。

現在，登入您的傳統模型帳戶。

```powershell
    Add-AzureAccount
```

請使用下列命令來取得可用的訂用帳戶：

```powershell
    Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

設定目前工作階段的 Azure 訂用帳戶。 這個範例會將預設訂用帳戶設定為 [我的 Azure 訂用帳戶]。 將範例訂用帳戶名稱取代為您自己的名稱。

```powershell
    Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```

<br>

## <a name="step-5-make-sure-you-have-enough-azure-resource-manager-virtual-machine-vcpus-in-the-azure-region-of-your-current-deployment-or-vnet"></a>步驟 5︰確定您目前的部署或 VNET 的 Azure 區域中有足夠的 Azure Resource Manager 虛擬機器 vCPU
您可以使用下列 PowerShell 命令來檢查您目前在 Azure Resource Manager 中擁有的 vCPU 數目。 若要深入了解 vCPU 配額，請參閱[限制和 Azure Resource Manager](../../azure-subscription-service-limits.md#limits-and-the-azure-resource-manager)。

此範例會檢查**美國西部**區域的可用性。 將範例區域名稱取代為您自己的名稱。

```powershell
Get-AzureRmVMUsage -Location "West US"
```

## <a name="step-6-run-commands-to-migrate-your-iaas-resources"></a>步驟 6︰執行命令來移轉 IaaS 資源
* [移轉雲端服務中的 VM (不在虛擬網路中)](#step-61-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network)
* [移轉虛擬網路中的 VM](#step-61-option-2---migrate-virtual-machines-in-a-virtual-network)
* [移轉儲存體帳戶](#step-62-migrate-a-storage-account)

> [!NOTE]
> 下述所有作業都是等冪的。 如果您有不支援的功能或組態錯誤以外的任何問題，建議您重新嘗試準備、中止或認可作業。 平台將會重新嘗試該動作。


### <a name="step-61-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>步驟 6.1：選項 1 - 移轉雲端服務中的虛擬機器 (不在虛擬網路中)
使用下列命令來取得雲端服務清單，然後選擇您想要移轉的雲端服務。 如果雲端服務中的 VM 是在虛擬網路中，或是具有 Web 角色或背景工作角色，命令就會傳回錯誤訊息。

```powershell
    Get-AzureService | ft Servicename
```

取得雲端服務的部署名稱。 在此範例中，服務名稱是 **My Service**。 將範例服務名稱取代為您自己的服務名稱。

```powershell
    $serviceName = "My Service"
    $deployment = Get-AzureDeployment -ServiceName $serviceName
    $deploymentName = $deployment.DeploymentName
```

準備好雲端服務中的虛擬機器以進行移轉。 有兩個選項可供您選擇。

* **選項 1：將 VM 移轉到平台建立的虛擬網路**

    首先，請使用下列命令來驗證您是否可以移轉雲端服務︰

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    $validate.ValidationMessages
    ```

    下列命令會顯示封鎖移轉的任何警告和錯誤。 如果驗證成功，您便可以繼續進行下列「準備」步驟：

    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    ```
* **選項 2：移轉到 Resource Manager 部署模型中的現有虛擬網路**

    這個範例會將資源群組名稱設定為 **myResourceGroup**，將虛擬網路名稱設定為 **myVirtualNetwork** 以及將子網路名稱設定為 **mySubNet**。 將此範例中的名稱取代為您自己的資源名稱。

    ```powershell
    $existingVnetRGName = "myResourceGroup"
    $vnetName = "myVirtualNetwork"
    $subnetName = "mySubNet"
    ```

    首先，請使用下列命令來驗證您是否可以移轉虛擬網路︰

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
    $validate.ValidationMessages
    ```

    下列命令會顯示封鎖移轉的任何警告和錯誤。 如果驗證成功，您便可以繼續進行下列「準備」步驟：

    ```powershell
        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        -VirtualNetworkName $vnetName -SubnetName $subnetName
    ```

上述其中一個選項的「準備」作業成功之後，請查詢 VM 的移轉狀態。 確定它們是處於 `Prepared` 狀態。

此範例中會將 VM 名稱設定為 **myVM**。 將範例名稱取代為您自己的 VM 名稱。

```powershell
    $vmName = "myVM"
    $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
    $vm.VM.MigrationState
```

使用 PowerShell 或 Azure 入口網站來檢查已備妥之資源的組態。 如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

```powershell
    Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName
```

如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

```powershell
    Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName
```

### <a name="step-61-option-2---migrate-virtual-machines-in-a-virtual-network"></a>步驟 6.1：選項 2 - 移轉虛擬網路中的虛擬機器

若要移轉虛擬網路中的虛擬機器，您將需要移轉虛擬網路。 虛擬機器會自動隨著虛擬網路移轉。 選取您想要移轉的虛擬網路。
> [!NOTE]
> [移轉單一傳統虛擬機器](migrate-single-classic-to-resource-manager.md)，其做法是使用虛擬機器的 VHD (OS 和資料) 檔案，建立具有受控磁碟的新 Resource Manager 虛擬機器。
<br>

> [!NOTE]
> 虛擬網路名稱可能與新入口網站中顯示的名稱不同。 新 Azure 入口網站會將名稱顯示為 `[vnet-name]`，但實際虛擬網路名稱類型為 `Group [resource-group-name] [vnet-name]`。 移轉之前，使用 `Get-AzureVnetSite | Select -Property Name` 命令查閱實際虛擬網路名稱，或在舊的 Azure 入口網站中檢視它。 

這個範例會將虛擬網路名稱設定為 **myVnet**。 將範例虛擬網路名稱取代為您自己的名稱。

```powershell
    $vnetName = "myVnet"
```

> [!NOTE]
> 如果虛擬網路包含 Web 角色或背景工作角色，或有具備不支援之組態的 VM，您就會收到驗證錯誤訊息。

首先，請使用下列命令來驗證您是否可以移轉虛擬網路︰

```powershell
    Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

下列命令會顯示封鎖移轉的任何警告和錯誤。 如果驗證成功，您便可以繼續進行下列「準備」步驟：

```powershell
    Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

請使用 Azure PowerShell 或 Azure 入口網站來檢查已備妥之虛擬機器的組態。 如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

```powershell
    Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

```powershell
    Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```

### <a name="step-62-migrate-a-storage-account"></a>步驟 6.2：移轉儲存體帳戶
完成虛擬機器的移轉之後，建議您先執行下列先決條件檢查，再移轉儲存體帳戶。

> [!NOTE]
> 如果您的儲存體帳戶有沒有相關聯的磁碟或 VM 資料，您可以直接跳到**驗證儲存體帳戶並開始移轉**一節。

* **先決條件會檢查您已移轉的任何 VM 或儲存體帳戶是否具有磁碟資源**
    * **移轉磁碟儲存於儲存體帳戶的傳統虛擬機器**

        下列命令會傳回儲存體帳戶中所有傳統 VM 磁碟的 RoleName 和 DiskName 屬性。 RoleName 是磁碟所連線的虛擬機器名稱。 如果下列命令傳回磁碟，請確定這些磁碟所連結的虛擬機器會在移轉儲存體帳戶之前移轉。
        ```powershell
         $storageAccountName = 'yourStorageAccountName'
          Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
          DiskName | Format-List -Property RoleName, DiskName

        ```
    * **刪除儲存於儲存體帳戶的未連結傳統 VM 磁碟**

        使用下列命令，尋找儲存體帳戶中未連線的傳統 VM 磁碟︰

        ```powershell
            $storageAccountName = 'yourStorageAccountName'
            Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Where-Object -Property AttachedTo -EQ $null | Format-List -Property DiskName  

        ```
        如果上述命令傳回磁碟，則使用下列命令刪除這些磁碟︰

        ```powershell
           Remove-AzureDisk -DiskName 'yourDiskName'
        ```
    * **刪除儲存於儲存體帳戶的 VM 映像**

        下列命令會傳回將作業系統磁碟儲存於儲存體帳戶的所有 VM 映像。
         ```powershell
            Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageAccountName)`
                                    } | Select-Object -Property ImageName, ImageLabel
         ```
         下列命令會傳回將資料磁碟儲存於儲存體帳戶的所有 VM 映像。
         ```powershell

            Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
                                             -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)}).Count -gt 0 `
                                            } | Select-Object -Property ImageName, ImageLabel
         ```
        使用下列命令，刪除上述命令所傳回的所有 VM 映像︰
        ```powershell
        Remove-AzureVMImage -ImageName 'yourImageName'
        ```
* **驗證儲存體帳戶並開始移轉**

    請使用下列命令來驗證每個要移轉的儲存體帳戶。 在此範例中，儲存體帳戶名稱是 **myStorageAccount**。 將範例名稱取代為您自己的儲存體帳戶名稱。

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName
    ```

    下一個步驟是準備儲存體帳戶以進行移轉

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName
    ```

    請使用 Azure PowerShell 或 Azure 入口網站來檢查已備妥之儲存體帳戶的組態。 如果您尚未準備好進行移轉，而想要回到舊狀態，請使用下列命令：

    ```powershell
        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName
    ```

    如果備妥的組態看起來沒問題，您就可以繼續進行並使用下列命令來認可資源：

    ```powershell
        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName
    ```

## <a name="next-steps"></a>後續步驟
* [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager) 的概觀](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [將 IaaS 資源從傳統移轉至 Azure Resource Manager 的規劃](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 CLI 將 IaaS 資源從傳統移轉至 Azure Resource Manager](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [用於協助將 IaaS 資源從傳統移轉至 Azure Resource Manager 的社群工具](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱最常見的移轉錯誤](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [檢閱有關將 IaaS 資源從傳統移轉至 Azure Resource Manager 的常見問題集](migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
