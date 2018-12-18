---
title: 將應用程式部署到 Azure 虛擬機器擴展集 | Microsoft Docs
description: 了解如何將應用程式部署到擴展集中的 Linux 和 Windows 虛擬機器執行個體
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: f8892199-f2e2-4b82-988a-28ca8a7fd1eb
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: cynthn
ms.openlocfilehash: 86bb231a4eb9468ccc92c182b099714950b2a81c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979430"
---
# <a name="deploy-your-application-on-virtual-machine-scale-sets"></a>在虛擬機器擴展集上部署您的應用程式
若要在擴展集的虛擬機器 (VM) 執行個體上執行應用程式，您需要先安裝應用程式元件和必要的檔案。 本文介紹如何在擴展集中建置執行個體的自訂 VM 映像，或在現有的 VM 執行個體上自動執行安裝指令碼。 您也將了解如何跨擴展集管理應用程式或作業系統更新。


## <a name="build-a-custom-vm-image"></a>組建自訂的 VM 映像
使用其中一個 Azure 平台映像在擴展集中建立執行個體時，不會安裝或設定其他軟體。 您可以自動安裝這些元件，但將 VM 執行個體佈建到擴展集會花費較長的時間。 若您將多項組態變更套用至 VM 執行個體，則會增管理這些設定指令碼和工作的負擔。

若要減少組態管理和佈建 VM 的時間，您可以建立自訂的 VM 映像，以便在擴展集中佈建執行個體後立即準備好執行應用程式。 若要進一步了解如何建立及使用含擴展集的自訂 VM 映像，請參閱下列教學課程：

- [Azure CLI](tutorial-use-custom-image-cli.md)
- [Azure PowerShell](tutorial-use-custom-image-powershell.md)


## <a name="already-provisioned"></a>安裝包含自訂指令碼延伸模組的應用程式
自訂指令碼擴充功能會在 Azure VM 上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。 若要進一步了解如何建立及使用含擴展集的自訂 VM 映像，請參閱下列教學課程：

- [Azure CLI](tutorial-install-apps-cli.md)
- [Azure PowerShell](tutorial-install-apps-powershell.md)
- [Azure Resource Manager 範本](tutorial-install-apps-template.md)


## <a name="install-an-app-to-a-windows-vm-with-powershell-dsc"></a>使用 PowerShell DSC 將應用程式安裝到 Windows VM
[PowerShell 預期狀態設定 (DSC)](https://msdn.microsoft.com/powershell/dsc/overview) 是一個管理平台，可定義目標電腦的設定。 DSC 設定會定義要在電腦上安裝的項目，以及設定主機的方式。 本機設定管理員 (LCM) 引擎會在每個目標節點上執行，這些節點會以推送組態作為基礎來處理要求的動作。

PowerShell DSC 延伸模組可讓您在擴展集中使用 PowerShell 來自訂 VM 執行個體。 下列範例：

- 指示 VM 執行個體從 GitHub 下載 DSC 套件 - *https://github.com/Azure-Samples/compute-automation-configurations/raw/master/dsc.zip*
- 設定延伸模組來執行安裝指令碼 - `configure-http.ps1`
- 使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) 取得擴展集的相關資訊
- 使用 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss) 將延伸模組套用到 VM 執行個體

會將 DSC 延伸模組套用至名為 *myResourceGroup* 的資源群組中的 *myScaleSet* VM 執行個體。 輸入您自己的名稱，如下所示：

```powershell
# Define the script for your Desired Configuration to download and run
$dscConfig = @{
  "wmfVersion" = "latest";
  "configuration" = @{
    "url" = "https://github.com/Azure-Samples/compute-automation-configurations/raw/master/dsc.zip";
    "script" = "configure-http.ps1";
    "function" = "WebsiteTest";
  };
}

# Get information about the scale set
$vmss = Get-AzureRmVmss `
                -ResourceGroupName "myResourceGroup" `
                -VMScaleSetName "myScaleSet"

# Add the Desired State Configuration extension to install IIS and configure basic website
$vmss = Add-AzureRmVmssExtension `
    -VirtualMachineScaleSet $vmss `
    -Publisher Microsoft.Powershell `
    -Type DSC `
    -TypeHandlerVersion 2.24 `
    -Name "DSC" `
    -Setting $dscConfig

# Update the scale set and apply the Desired State Configuration extension to the VM instances
Update-AzureRmVmss `
    -ResourceGroupName "myResourceGroup" `
    -Name "myScaleSet"  `
    -VirtualMachineScaleSet $vmss
```

如果擴展集上的升級原則是*手動*，則使用 [Update-AzureRmVmssInstance](/powershell/module/azurerm.compute/update-azurermvmssinstance) 更新您的 VM 執行個體。 此 Cmdlet 會將更新的擴展集設定套用至 VM 執行個體，並安裝您的應用程式。


## <a name="install-an-app-to-a-linux-vm-with-cloud-init"></a>使用 cloud-init 將應用程式安裝到 Linux VM
[Cloud-init (英文)](https://cloudinit.readthedocs.io/en/latest/index.html) 是在 Linux VM 初次開機時，廣泛用來自訂它們的方法。 您可以使用 cloud-init 來安裝封裝和寫入檔案，或者設定使用者和安全性。 當 cloud-init 在初次開機程序期間執行時，不需要使用任何額外的步驟或必要的代理程式來套用您的組態。

Cloud-init 也適用於散發套件。 例如，您不使用 **apt-get install** 或 **yum install** 來安裝套件。 您可以改為定義要安裝的套件清單。 Cloud-init 會針對您選取的散發套件自動使用原生的套件管理工具。

如需詳細資訊，包括 *cloud-init.txt* 範例檔，請參閱[使用 cloud-init 自訂 Azure VM](../virtual-machines/linux/using-cloud-init.md)。

若要建立擴展集並使用 cloud-init 檔案，請將 `--custom-data` 參數新增至 [az vmss create](/cli/azure/vmss#az_vmss_create) 命令，並指定 cloud-init 檔案的名稱。 下列範例會在 *myResourceGroup* 中建立名為 *myScaleSet* 的擴展集，並使用名為 *cloud-init.txt* 的檔案設定 VM 執行個體。 輸入您自己的名稱，如下所示：

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys
```


### <a name="install-applications-with-os-updates"></a>安裝應用程式與作業系統更新
當新的作業系統版本可用時，您可以使用或建立新的自訂映像，並[將作業系統升級部署到](virtual-machine-scale-sets-upgrade-scale-set.md)擴展集。 每個 VM 執行個體會升級為您指定的最新映像。 您可以使用預先安裝應用程式的自訂映像、自訂指令碼延伸模組，或 PowerShell，以便在執行升級時自動開放使用應用程式。 執行此程序時可能需要規劃應用程式維護，確認不存在任何版本相容性問題。

如果您使用預先安裝應用程式的自訂 VM 映像，您可以整合應用程式更新與部署管線來建立新映像，並跨擴展集部署作業系統升級。 這個方法可讓管線挑選最新的應用程式組建、建立及驗證 VM 映像，然後在擴展集中升級 VM 執行個體。 若要執行部署管線，跨自訂 VM 映像來建立和部署應用程式更新，您可以[使用 Azure DevOps Services 建立 Packer 映像及進行部署](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)，或使用另一個平台，例如 [Spinnaker](https://www.spinnaker.io/) 或 [Jenkins](https://jenkins.io/)。


## <a name="next-steps"></a>後續步驟
當您建置應用程式並將其部署到擴展集時，您可以檢閱[擴展集設計概觀](virtual-machine-scale-sets-design-overview.md)。 如需如何管理擴展集的詳細資訊，請參閱[使用 PowerShell 管理您的擴展集](virtual-machine-scale-sets-windows-manage.md)。
