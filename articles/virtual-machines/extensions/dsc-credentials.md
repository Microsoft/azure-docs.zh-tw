---
title: 使用期望的狀態設定將認證傳遞至 Azure
description: 了解如何使用 PowerShell 期望狀態設定 (DSC) 將認證安全地傳遞至 Azure 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: bobbytreed
manager: carmonm
editor: ''
tags: azure-resource-manager
keywords: dsc
ms.assetid: ea76b7e8-b576-445a-8107-88ea2f3876b9
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 05/02/2018
ms.author: robreed
ms.openlocfilehash: 52e115aa7f54eccc2be4e500c544aa38ca3bc32d
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45631271"
---
# <a name="pass-credentials-to-the-azure-dscextension-handler"></a>將認證傳遞至 Azure DSCExtension 處理常式

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

本文涵蓋適用於 Azure 的期望狀態設定 (DSC) 擴充功能。 如需 DSC 擴充處理常式的概觀，請參閱 [Azure 期望狀態設定擴充功能處理常式簡介](dsc-overview.md)。

## <a name="pass-in-credentials"></a>傳入認證

您可能需要設定過程中，於使用者內容中設定使用者帳戶、存取服務，或安裝程式。 若要執行這些動作，您需要提供認證。

您可以使用 DSC 設定參數化的設定。 在參數化的設定中，會將認證傳入至設定並安全地儲存在 .mof 檔案中。 Azure 延伸模組處理常式會提供憑證的自動管理功能，藉以簡化認證的管理。

下列 DSC 設定指令碼會建立包含指定密碼的本機使用者帳戶︰

```powershell
configuration Main
{
    param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [PSCredential]
        $Credential
    )
    Node localhost {
        User LocalUserAccount
        {
            Username = $Credential.UserName
            Password = $Credential
            Disabled = $false
            Ensure = "Present"
            FullName = "Local User Account"
            Description = "Local User Account"
            PasswordNeverExpires = $true
        }
    }
}
```

請務必將 **node localhost** 納入為設定的一部分。 延伸模組處理常式專門尋找 **node localhost** 陳述式。 如果缺少此陳述式，將無法執行下列步驟。 也請務必納入 typecast **[PsCredential]**。 這個特定的類型會觸發擴充功能對認證進行加密。

若要將此指令碼發佈至 Azure Blob 儲存體︰

`Publish-AzureRmVMDscConfiguration -ConfigurationPath .\user_configuration.ps1`

若要設定 Azure DSC 擴充功能，並提供認證︰

```powershell
$configurationName = 'Main'
$configurationArguments = @{ Credential = Get-Credential }
$configurationArchive = 'user_configuration.ps1.zip'
$vm = Get-AzureRmVM -Name 'example-1'

$vm = Set-AzureRmVMDscExtension -VMName $vm -ConfigurationArchive $configurationArchive -ConfigurationName $configurationName -ConfigurationArgument @configurationArguments

$vm | Update-AzureRmVM
```

## <a name="how-a-credential-is-secured"></a>如何保護認證

執行此程式碼會提示您輸入認證。 提供認證之後，它會暫時儲存在記憶體中。 使用 **Set-AzureRmVMDscExtension** Cmdlet 發佈認證時，認證會透過 HTTPS 傳輸至 VM。 在 VM 中，Azure 會使用本機 VM 憑證儲存在磁碟上加密的認證。 認證會在記憶體中短暫地解密後再重新加密，以便傳遞給 DSC。

此流程與[使用不含延伸模組處理常式的安全組態](/powershell/dsc/securemof)不同。 Azure 環境提供一個透過憑證以安全傳輸組態資料的方式。 使用 DSC 擴充處理常式時，您不需要在 **ConfigurationData** 中提供 **$CertificatePath** 或 **$CertificateID**/ **$Thumbprint** 項目。

## <a name="next-steps"></a>後續步驟

- 取得 [Azure DSC 擴充處理常式簡介](dsc-overview.md)。
- 查看 [適用於 DSC 擴充功能的 Azure Resource Manager 範本](dsc-template.md)。
- 如需有關 PowerShell DSC 的詳細資訊，請移至 [PowerShell 文件中心](/powershell/dsc/overview)。
- 如需更多您可以使用 PowerShell DSC 進行管理的功能，以及更多 DSC 資源，請瀏覽 [PowerShell 資源庫](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)。