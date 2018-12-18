---
title: 在 Azure 中為可擴充的應用程式建立 VM 和儲存體帳戶 | Microsoft Docs
description: 了解如何部署 VM，以用來執行使用 Azure Blob 儲存體的可擴充應用程式
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.custom: mvc
ms.component: blobs
ms.openlocfilehash: fddf35f4d2a2bc7abef891ee508eaecc9ffb4719
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39396908"
---
# <a name="create-a-virtual-machine-and-storage-account-for-a-scalable-application"></a>為可擴充的應用程式建立虛擬機器和儲存體帳戶

本教學課程是一個系列的第一部分。 本教學課程會示範如何部署應用程式，以透過 Azure 儲存體帳戶上傳和下載大量隨機資料。 當您完成後，您就有可在虛擬機器上執行的主控台應用程式，並可將大量資料上傳和下載到儲存體帳戶。

在系列的第一部分中，您將了解如何：

> [!div class="checklist"]
> * 建立儲存體帳戶
> * 建立虛擬機器
> * 設定自訂指令碼擴充功能

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶
 
此範例會將 50 個大型檔案上傳至 Azure 儲存體帳戶的 Blob 容器。 儲存體帳戶提供唯一命名空間來儲存及存取您的 Azure 儲存體資料物件。 在使用 [New-AzureRmStorageAccount](/powershell/module/AzureRM.Storage/New-AzureRmStorageAccount) 命令所建立的資源群組中建立儲存體帳戶。

在下列命令中，使用您自己的全域唯一名稱來替代見到 `<blob_storage_account>` 預留位置的 Blob 儲存體帳戶。

```powershell-interactive
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName myResourceGroup `
  -Name "<blob_storage_account>" `
  -Location EastUS `
  -SkuName Standard_LRS `
  -Kind Storage `
  -EnableEncryptionService Blob
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

建立虛擬機器組態。 此組態包括部署虛擬機器時所使用的組態，例如虛擬機器映像、大小和驗證組態。 執行此步驟時，系統會提示您輸入認證。 您輸入的值會設定為虛擬機器的使用者名稱和密碼。

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立虛擬機器。

```azurepowershell-interactive
# Variables for common values
$resourceGroup = "myResourceGroup"
$location = "eastus"
$vmName = "myVM"

# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

# Create a virtual network card and associate with public IP address
$nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

# Create a virtual machine configuration
$vmConfig = New-AzureRmVMConfig -VMName myVM -VMSize Standard_DS14_v2 | `
    Set-AzureRmVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzureRmVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzureRmVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzureRmVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig

Write-host "Your public IP address is $($pip.IpAddress)"
```

## <a name="deploy-configuration"></a>部署組態

針對本教學課程，有一些必要的元件必須安裝在虛擬機器上。 自訂指令碼擴充功能會用來執行可完成下列工作的 PowerShell 指令碼：

> [!div class="checklist"]
> * 安裝 .NET core 2.0
> * 安裝 chocolatey
> * 安裝 GIT
> * 複製範例存放庫
> * 還原 NuGet 套件
> * 建立 50 個包含隨機資料的 1GB 檔案

執行下列 Cmdlet 來完成虛擬機器的組態。 完成此步驟需要 5-15 分鐘。

```azurepowershell-interactive
# Start a CustomScript extension to use a simple PowerShell script to install .NET core, dependencies, and pre-create the files to upload.
Set-AzureRMVMCustomScriptExtension -ResourceGroupName myResourceGroup `
    -VMName myVM `
    -Location EastUS `
    -FileUri https://raw.githubusercontent.com/azure-samples/storage-dotnet-perf-scale-app/master/setup_env.ps1 `
    -Run 'setup_env.ps1' `
    -Name DemoScriptExtension
```

## <a name="next-steps"></a>後續步驟

在此系列的第一部分，您已了解如何建立儲存體帳戶、部署虛擬機器，以及為虛擬機器設定必要條件，例如如何：

> [!div class="checklist"]
> * 建立儲存體帳戶
> * 建立虛擬機器
> * 設定自訂指令碼擴充功能

接著請前往此系列的第二個部分，使用指數式重試和平行處理原則來將大量資料上傳至儲存體帳戶。

> [!div class="nextstepaction"]
> [將大量大型檔案平行上傳至儲存體帳戶](storage-blob-scalable-app-upload-files.md)
