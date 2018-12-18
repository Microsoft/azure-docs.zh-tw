---
title: 設定 Azure VM 的 WinRM 存取 | Microsoft Docs
description: 設定 WinRM 存取給 Resource Manager 部署模型中建立的 Azure 虛擬機器使用。
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 9718e85b-d360-4621-90b8-0b0b84a21208
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 06/16/2016
ms.author: kasing
ms.openlocfilehash: 5fa82dd4a85ff2e62848df0fdc6006922005a84b
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30914540"
---
# <a name="setting-up-winrm-access-for-virtual-machines-in-azure-resource-manager"></a>在 Azure Resource Manager 中設定虛擬機器的 WinRM 存取
## <a name="winrm-in-azure-service-management-vs-azure-resource-manager"></a>Azure Service Management 與 Azure Resource Manager 中的 WinRM

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-rm-include.md)]

* 如需 Azure Resource Manager 的概觀，請參閱[本文章](../../azure-resource-manager/resource-group-overview.md)
* 如需 Azure Service Management 與 Azure Resource Manager 之間的差異性，請參閱 [本文章](../../resource-manager-deployment-model.md)

在兩個堆疊之間設定 WinRM 組態的主要差異在於如何在 VM 上安裝憑證。 在 Azure Resource Manager 堆疊中，憑證會模型化為「金鑰保存庫資源提供者」所管理的資源。 因此，使用者必須提供自己的憑證，並在 VM 中使用憑證之前先將它上傳至金鑰保存庫。

以下是您對 VM 設定 WinRM 連線必須採取的步驟

1. 建立金鑰保存庫
2. 建立自我簽署憑證
3. 將您的自我簽署憑證上傳至金鑰保存庫
4. 取得金鑰保存庫中您的自我簽署憑證的 URL
5. 在建立 VM 時參考您的自我簽署憑證的 URL

## <a name="step-1-create-a-key-vault"></a>步驟 1︰建立金鑰保存庫
您可以使用下列命令來建立金鑰保存庫

```
New-AzureRmKeyVault -VaultName "<vault-name>" -ResourceGroupName "<rg-name>" -Location "<vault-location>" -EnabledForDeployment -EnabledForTemplateDeployment
```

## <a name="step-2-create-a-self-signed-certificate"></a>步驟 2：建立自我簽署憑證
您可以使用下列 PowerShell 指令碼建立自我簽署憑證

```
$certificateName = "somename"

$thumbprint = (New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation Cert:\CurrentUser\My -KeySpec KeyExchange).Thumbprint

$cert = (Get-ChildItem -Path cert:\CurrentUser\My\$thumbprint)

$password = Read-Host -Prompt "Please enter the certificate password." -AsSecureString

Export-PfxCertificate -Cert $cert -FilePath ".\$certificateName.pfx" -Password $password
```

## <a name="step-3-upload-your-self-signed-certificate-to-the-key-vault"></a>步驟 3：將自我簽署憑證上傳至金鑰保存庫
在將憑證上傳至於步驟 1 中建立的金鑰保存庫之前，必須先將它轉換成 Microsoft.Compute 資源提供者可以了解的格式。 下面的 PowerShell 指令碼可讓您這麼做

```
$fileName = "<Path to the .pfx file>"
$fileContentBytes = Get-Content $fileName -Encoding Byte
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)

$jsonObject = @"
{
  "data": "$filecontentencoded",
  "dataType" :"pfx",
  "password": "<password>"
}
"@

$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

$secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText –Force
Set-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>" -SecretValue $secret
```

## <a name="step-4-get-the-url-for-your-self-signed-certificate-in-the-key-vault"></a>步驟 4：取得金鑰保存庫中您的自我簽署憑證的 URL
Microsoft.Compute 資源提供者在佈建 VM 時，需要金鑰保存庫內密碼的 URL。 這可讓 Microsoft.Compute 資源提供者下載密碼，並在 VM 上建立對等憑證。

> [!NOTE]
> 密碼的 URL 也必須包含版本。 範例 URL 如下所示：https://contosovault.vault.azure.net:443/secrets/contososecret/01h9db0df2cd4300a20ence585a6s7ve
> 
> 

#### <a name="templates"></a>範本
您可以使用下列程式碼取得範本中的 URL 連結

    "certificateUrl": "[reference(resourceId(resourceGroup().name, 'Microsoft.KeyVault/vaults/secrets', '<vault-name>', '<secret-name>'), '2015-06-01').secretUriWithVersion]"

#### <a name="powershell"></a>PowerShell
您可以使用下列 PowerShell 命令取得此 URL 

    $secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id

## <a name="step-5-reference-your-self-signed-certificates-url-while-creating-a-vm"></a>步驟 5：在建立 VM 時參考您的自我簽署憑證的 URL
#### <a name="azure-resource-manager-templates"></a>Azure Resource Manager 範本
透過範本建立 VM 時，憑證會在密碼區段和 winRM 區段中參考，如下所示：

    "osProfile": {
          ...
          "secrets": [
            {
              "sourceVault": {
                "id": "<resource id of the Key Vault containing the secret>"
              },
              "vaultCertificates": [
                {
                  "certificateUrl": "<URL for the certificate you got in Step 4>",
                  "certificateStore": "<Name of the certificate store on the VM>"
                }
              ]
            }
          ],
          "windowsConfiguration": {
            ...
            "winRM": {
              "listeners": [
                {
                  "protocol": "http"
                },
                {
                  "protocol": "https",
                  "certificateUrl": "<URL for the certificate you got in Step 4>"
                }
              ]
            },
            ...
          }
        },

上述的範例範本可以在這裡找到： [201-vm-winrm-keyvault-windows](https://azure.microsoft.com/documentation/templates/201-vm-winrm-keyvault-windows)

此範本的原始程式碼位於 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows)

#### <a name="powershell"></a>PowerShell
    $vm = New-AzureRmVMConfig -VMName "<VM name>" -VMSize "<VM Size>"
    $credential = Get-Credential
    $secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
    $vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName "<Computer Name>" -Credential $credential -WinRMHttp -WinRMHttps -WinRMCertificateUrl $secretURL
    $sourceVaultId = (Get-AzureRmKeyVault -ResourceGroupName "<Resource Group name>" -VaultName "<Vault Name>").ResourceId
    $CertificateStore = "My"
    $vm = Add-AzureRmVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore $CertificateStore -CertificateUrl $secretURL

## <a name="step-6-connecting-to-the-vm"></a>步驟 6︰連接到 VM
在您可以連接至 VM 之前，您必須確定您的電腦已設定 WinRM 遠端管理。 以系統管理員身分啟動 PowerShell 並執行下列命令來確認您已設定完畢。

    Enable-PSRemoting -Force

> [!NOTE]
> 如果上述程式碼無法運作，您可能需要確定 WinRM 服務正在執行。 您可以使用 `Get-Service WinRM`
> 
> 

安裝完成之後，您可以使用下列命令連接到 VM

    Enter-PSSession -ConnectionUri https://<public-ip-dns-of-the-vm>:5986 -Credential $cred -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck) -Authentication Negotiate
