---
title: 教學課程：在 Azure 中使用 SSL 憑證來保護 Windows 網頁伺服器 | Microsoft Docs
description: 在本教學課程中，您將了解如何搭配使用 Azure PowerShell 與 Azure Key Vault 中儲存的 SSL 憑證，來保護執行 IIS 網頁伺服器的 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: e42f8f36acdaa5e84f4a3087cac7016867622c21
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338819"
---
# <a name="tutorial-secure-a-web-server-on-a-windows-virtual-machine-in-azure-with-ssl-certificates-stored-in-key-vault"></a>教學課程：在 Azure 中使用 Key Vault 內儲存的 SSL 憑證，來保護 Windows 虛擬機器上的網頁伺服器

若要保護網頁伺服器，您可以使用安全通訊端層 (SSL) 憑證將 Web 流量加密。 這些 SSL 憑證可儲存在 Azure Key Vault，並且能夠讓您將憑證安全地部署到 Azure 中的 Windows 虛擬機器 (VM)。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 Azure Key Vault
> * 產生或上傳憑證至 Key Vault
> * 建立 VM 並安裝 IIS 網頁伺服器
> * 將憑證插入 VM 並使用 SSL 繫結來設定 IIS

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


## <a name="overview"></a>概觀
Azure Key Vault 會保護密碼編譯金鑰和祕密，這類憑證或密碼。 Key Vault 有助於簡化憑證管理程序，並可讓您掌控用來存取這些憑證的金鑰。 您可以在 Key Vault 內建立自我簽署憑證，或上傳您目前已經擁有的受信任憑證。

您不必使用包含了內建憑證的自訂 VM 映像，而是要將憑證插入執行中的 VM。 此程序可確保您在部署期間安裝在網頁伺服器上的憑證會是最新的。 如果您更新或取代憑證，您就不必另外再建立新的自訂 VM 映像。 當您建立其他 VM 時，系統會自動插入最新的憑證。 在整個過程中，憑證絕對不會離開 Azure 平台，或在指令碼、命令列記錄或範本中公開。


## <a name="create-an-azure-key-vault"></a>建立 Azure Key Vault
建立 Key Vault 和憑證之前，請先使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 來建立資源群組。 下列範例會在「美國東部」位置建立名為 myResourceGroupSecureWeb 的資源群組：

```azurepowershell-interactive
$resourceGroup = "myResourceGroupSecureWeb"
$location = "East US"
New-AzureRmResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

接著，使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/new-azurermkeyvault) 建立 Key Vault。 每個 Key Vault 需要唯一的名稱，而且應該全部小寫。 使用您自己唯一的 Key Vault 名稱來取代下列範例中的 `mykeyvault`：

```azurepowershell-interactive
$keyvaultName="mykeyvault"
New-AzureRmKeyVault -VaultName $keyvaultName `
    -ResourceGroup $resourceGroup `
    -Location $location `
    -EnabledForDeployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>產生憑證並儲存於 Key Vault
若要在生產環境中使用，您應該使用 [Import-AzureKeyVaultCertificate](/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate) 來匯入由受信任的提供者所簽署的有效憑證。 在本教學課程中，下列範例示範如何使用 [Add-AzureKeyVaultCertificate](/powershell/module/azurerm.keyvault/add-azurekeyvaultcertificate) 來產生自我簽署憑證，而且該憑證會使用透過 [New-AzureKeyVaultCertificatePolicy](/powershell/module/azurerm.keyvault/new-azurekeyvaultcertificatepolicy) 所得到的預設憑證原則。 

```azurepowershell-interactive
$policy = New-AzureKeyVaultCertificatePolicy `
    -SubjectName "CN=www.contoso.com" `
    -SecretContentType "application/x-pkcs12" `
    -IssuerName Self `
    -ValidityInMonths 12

Add-AzureKeyVaultCertificate `
    -VaultName $keyvaultName `
    -Name "mycert" `
    -CertificatePolicy $policy 
```


## <a name="create-a-virtual-machine"></a>建立虛擬機器
使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 來設定 VM 的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在您可以使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 下列範例會在 *EastUS* 位置中建立名為 *myVM* 的 VM。 如果它們尚不存在，則會建立支援網路資源。 為了允許安全 Web 流量，此 Cmdlet 也會開啟連接埠 *443*。

```azurepowershell-interactive
# Create a VM
New-AzureRmVm `
    -ResourceGroupName $resourceGroup `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred `
    -OpenPorts 443

# Use the Custom Script Extension to install IIS
Set-AzureRmVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server -IncludeManagementTools"}'
```

系統需要花幾分鐘的時間來建立 VM。 最後一個步驟會使用 Azure 自訂指令碼擴充功能，利用 [Set-AzureRmVmExtension](/powershell/module/azurerm.compute/set-azurermvmextension) 來安裝 IIS 網頁伺服器。


## <a name="add-a-certificate-to-vm-from-key-vault"></a>將憑證從 Key Vault 新增至 VM
若要將憑證從 Key Vault 新增至 VM，請使用 [Get-AzureKeyVaultSecret](/powershell/module/azurerm.keyvault/get-azurekeyvaultsecret) 來取得憑證的識別碼。 使用 [Add-AzureRmVMSecret](/powershell/module/azurerm.compute/add-azurermvmsecret) 將憑證新增至 VM：

```azurepowershell-interactive
$certURL=(Get-AzureKeyVaultSecret -VaultName $keyvaultName -Name "mycert").id

$vm=Get-AzureRmVM -ResourceGroupName $resourceGroup -Name "myVM"
$vaultId=(Get-AzureRmKeyVault -ResourceGroupName $resourceGroup -VaultName $keyVaultName).ResourceId
$vm = Add-AzureRmVMSecret -VM $vm -SourceVaultId $vaultId -CertificateStore "My" -CertificateUrl $certURL

Update-AzureRmVM -ResourceGroupName $resourceGroup -VM $vm
```


## <a name="configure-iis-to-use-the-certificate"></a>設定 IIS 以使用憑證
再次搭配 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) 來使用自訂指令碼擴充功能，以更新 IIS 組態。 這次的更新會套用從 Key Vault 插入到 IIS 的憑證，並設定 Web 繫結：

```azurepowershell-interactive
$PublicSettings = '{
    "fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/secure-iis.ps1"],
    "commandToExecute":"powershell -ExecutionPolicy Unrestricted -File secure-iis.ps1"
}'

Set-AzureRmVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString $publicSettings
```


### <a name="test-the-secure-web-app"></a>測試安全的 Web 應用程式
使用 [Get-AzureRmPublicIPAddress](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得 VM 的公用 IP 位址。 下列範例會取得稍早建立之 `myPublicIP` 的 IP 位址︰

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName $resourceGroup -Name "myPublicIPAddress" | select "IpAddress"
```

現在，您可以開啟 Web 瀏覽器，並在網址列輸入 `https://<myPublicIP>`。 若要在使用自我簽署憑證時接受安全性警告，請依序按一下 [詳細資料] 與 [繼續瀏覽網頁]：

![接受 Web 瀏覽器安全性警告](./media/tutorial-secure-web-server/browser-warning.png)

接著會顯示受保護的 IIS 網站，如下列範例所示：

![檢視執行中安全的 IIS 網站](./media/tutorial-secure-web-server/secured-iis.png)


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已使用儲存在 Azure Key Vault 中的 SSL 憑證來保護 IIS 網頁伺服器。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Azure Key Vault
> * 產生或上傳憑證至 Key Vault
> * 建立 VM 並安裝 IIS 網頁伺服器
> * 將憑證插入 VM 並使用 SSL 繫結來設定 IIS

用以下連結查看預先建立的虛擬機器指令碼範例。

> [!div class="nextstepaction"]
> [Windows 虛擬機器指令碼範例](./powershell-samples.md)
