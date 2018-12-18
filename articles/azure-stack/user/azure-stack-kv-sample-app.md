---
title: 允許應用程式取出 Azure Stack Key Vault 密碼 | Microsoft Docs
description: 使用範例應用程式來處理 Azure Stack Key Vault
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 3748b719-e269-4b48-8d7d-d75a84b0e1e5
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/15/2018
ms.author: sethm
ms.openlocfilehash: ed02174247de1a99f3d9a4880fd0afa60f867552
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "43050444"
---
# <a name="a-sample-application-that-uses-keys-and-secrets-stored-in-a-key-vault"></a>使用金鑰保存庫中儲存之金鑰和密碼的範例應用程式

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

請依照本文中的步驟執行範例應用程式 (HelloKeyVault)，以從 Azure Stack 中的金鑰保存庫擷取金鑰和密碼。

## <a name="prerequisites"></a>必要條件

您可以從 Azure Stack [開發套件](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop)安裝下列必要項目，或從 Windows 型外部用戶端 (如果您[透過 VPN 連線](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn)) 安裝：

* 安裝 [Azure Stack 相容的 Azure PowerShell 模組](azure-stack-powershell-install.md)。
* 下載[處理 Azure Stack 所需的工具](azure-stack-powershell-download.md)。

## <a name="create-and-get-the-key-vault-and-application-settings"></a>建立並取得金鑰保存庫和應用程式設定

若要準備範例應用程式：

* 在 Azure Stack 中建立金鑰保存庫。
* 在 Azure Active Directory (Azure AD) 中註冊應用程式。

您可以使用 Azure 入口網站或 PowerShell 來準備範例應用程式。 本文說明如何使用 PowerShell 建立金鑰保存庫及註冊應用程式。

>[!NOTE]
>依預設，PowerShell 指令碼會在 Active Directory 中建立新的應用程式。 不過，您可以註冊其中一個現有的應用程式。

 在執行下列指令碼之前，請確實提供 `aadTenantName` 和 `applicationPassword` 變數的值。 若未指定 `applicationPassword` 的值，此指令碼將會產生隨機密碼。

```powershell
$vaultName           = 'myVault'
$resourceGroupName   = 'myResourceGroup'
$applicationName     = 'myApp'
$location            = 'local'

# Password for the application. If not specified, this script will generate a random password during app creation.
$applicationPassword = ''

# Function to generate a random password for the application.
Function GenerateSymmetricKey()
{
    $key = New-Object byte[](32)
    $rng = [System.Security.Cryptography.RNGCryptoServiceProvider]::Create()
    $rng.GetBytes($key)
    return [System.Convert]::ToBase64String($key)
}

Write-Host 'Please log into your Azure Stack user environment' -foregroundcolor Green

$tenantARM = "https://management.local.azurestack.external"
$aadTenantName = "PLEASE FILL THIS IN WITH YOUR AAD TENANT NAME. FOR EXAMPLE: myazurestack.onmicrosoft.com"

# Configure the Azure Stack operator’s PowerShell environment.
Add-AzureRMEnvironment `
  -Name "AzureStackUser" `
  -ArmEndpoint $tenantARM

Set-AzureRmEnvironment `
  -Name "AzureStackAdmin" `
  -GraphAudience "https://graph.windows.net/"

$TenantID = Get-AzsDirectoryTenantId `
  -AADTenantName $aadTenantName `
  -EnvironmentName AzureStackUser

# Sign in to the user portal.
Add-AzureRmAccount `
  -EnvironmentName "AzureStackUser" `
  -TenantId $TenantID `

$now = [System.DateTime]::Now
$oneYearFromNow = $now.AddYears(1)

$applicationPassword = GenerateSymmetricKey

# Create a new Azure AD application.
$identifierUri = [string]::Format("http://localhost:8080/{0}",[Guid]::NewGuid().ToString("N"))
$homePage = "http://contoso.com"

Write-Host "Creating a new AAD Application"
$ADApp = New-AzureRmADApplication `
  -DisplayName $applicationName `
  -HomePage $homePage `
  -IdentifierUris $identifierUri `
  -StartDate $now `
  -EndDate $oneYearFromNow `
  -Password $applicationPassword

Write-Host "Creating a new AAD service principal"
$servicePrincipal = New-AzureRmADServicePrincipal `
  -ApplicationId $ADApp.ApplicationId

# Create a new resource group and a key vault in that resource group.
New-AzureRmResourceGroup `
  -Name $resourceGroupName `
  -Location $location

Write-Host "Creating vault $vaultName"
$vault = New-AzureRmKeyVault -VaultName $vaultName `
  -ResourceGroupName $resourceGroupName `
  -Sku standard `
  -Location $location

# Specify full privileges to the vault for the application.
Write-Host "Setting access policy"
Set-AzureRmKeyVaultAccessPolicy -VaultName $vaultName `
  -ObjectId $servicePrincipal.Id `
  -PermissionsToKeys all `
  -PermissionsToSecrets all

Write-Host "Paste the following settings into the app.config file for the HelloKeyVault project:"
'<add key="VaultUrl" value="' + $vault.VaultUri + '"/>'
'<add key="AuthClientId" value="' + $servicePrincipal.ApplicationId + '"/>'
'<add key="AuthClientSecret" value="' + $applicationPassword + '"/>'
Write-Host

```

下一個螢幕擷取畫面顯示用來建立金鑰保存庫之指令碼的輸出：

![具有存取金鑰的金鑰保存庫](media/azure-stack-kv-sample-app/settingsoutput.png)

記下先前指令碼所傳回 **VaultUrl**、**AuthClientId** 和 **AuthClientSecret** 的值。 您將使用這些值來執行 HelloKeyVault 應用程式。

## <a name="download-and-configure-the-sample-application"></a>下載並設定範例應用程式

下載 Azure [金鑰保存庫用戶端範例](https://www.microsoft.com/en-us/download/details.aspx?id=45343)頁面所提供的金鑰保存庫範例。 將 .zip 檔的內容解壓縮至您的開發工作站。 範例資料夾中有兩個應用程式，本文使用 HelloKeyVault。

若要載入 HelloKeyVault 範例：

* 瀏覽至 **Microsoft.Azure.KeyVault.Samples** > **samples** > **HelloKeyVault** 資料夾。
* 在 Visual Studio 中開啟 HelloKeyVault 應用程式。

### <a name="configure-the-sample-application"></a>設定範例應用程式

在 Visual Studio 中：

* 開啟 HelloKeyVault\App.config 檔案，然後找出 &lt;**appSettings**&gt; 元素。
* 以用來建立金鑰保存庫的傳回值更新 **VaultUrl**、**AuthClientId** 和 **AuthClientSecret** 金鑰。 (依預設，App.config 檔案會包含 *AuthCertThumbprint* 的預留位置。 請將此預留位置取代為 *AuthClientSecret*。)

  ![應用程式設定](media/azure-stack-kv-sample-app/appconfig.png)

* 重建方案。

## <a name="run-the-application"></a>執行應用程式

當您執行 HelloKeyVault 時，應用程式會登入 Azure AD，然後使用 AuthClientSecret 權杖來驗證 Azure Stack 中的金鑰保存庫。

您可以使用 HelloKeyVault 範例來：

* 執行基本作業，例如，對金鑰和密碼執行建立、加密、包裝和刪除等作業。
* 將 *encrypt* 和 *decrypt* 之類的參數傳至 HelloKeyVault，並將指定的變更套用至金鑰保存庫。

## <a name="next-steps"></a>後續步驟

[使用金鑰保存庫密碼部署 VM](azure-stack-kv-deploy-vm-with-secret.md)

[使用 Key Vault 憑證部署 VM](azure-stack-kv-push-secret-into-vm.md)
