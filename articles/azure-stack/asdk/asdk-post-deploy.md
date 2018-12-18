---
title: Azure Stack 開發套件 (ASDK) 的部署後設定| Microsoft Docs
description: 說明在安裝 Azure Stack 開發套件 (ASDK) 後建議的設定變更。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: d3bfe2c472d48a68bd818ac06874db136528b470
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43840264"
---
# <a name="post-asdk-installation-configuration-tasks"></a>ASDK 安裝後設定工作

在[安裝 Azure Stack 開發套件 (ASDK)](asdk-install.md) 後，您必須進行一些建議的安裝後設定變更。

## <a name="install-azure-stack-powershell"></a>安裝 Azure Stack PowerShell

您需要與 Azure Stack 相容的 Azure PowerShell 模組，才能搭配 Azure Stack 使用。

Azure Stack 的 PowerShell 命令是透過 PowerShell 資源庫進行安裝。 若要註冊 PSGallery 存放庫，請開啟提升權限的 PowerShell 工作階段，並執行下列命令：

``` Powershell
Set-PSRepository `
  -Name "PSGallery" `
  -InstallationPolicy Trusted
```

您可以使用 API 版本設定檔來指定 Azure Stack 相容 AzureRM 模組。  API 版本設定檔提供一個管理 Azure 與 Azure Stack 之間版本差異的方式。 API 版本設定檔是一組具有特定 API 版本的 AzureRM PowerShell 模組。 可透過「PowerShell 資源庫」取得的 **AzureRM.Bootstrapper** 模組會提供使用 API 版本設定檔所需的 PowerShell Cmdlet。

不論是否有對 ASDK 的主控電腦的網際網路連線能力，您都可以安裝最新的 Azure Stack PowerShell 模組：

> [!IMPORTANT]
> 在安裝必要版本之前，請務必[解除安裝任何現有的 Azure PowerShell 模組](.\.\azure-stack-powershell-install.md#3-uninstall-existing-versions-of-the-azure-stack-powershell-modules)。

- **具有來自 ASDK 主機電腦的網際網路連線**。 執行下列 PowerShell 指令碼，以在開發套件安裝上安裝這些模組：

  ``` PowerShell
  # Install the AzureRM.Bootstrapper module. Select Yes when prompted to install NuGet. 
  Install-Module `
    -Name AzureRm.BootStrapper

  # Install and import the API Version Profile required by Azure Stack into the current PowerShell session.
  Use-AzureRmProfile `
    -Profile 2017-03-09-profile -Force

  # Install Azure Stack Module Version 1.4.0. If running a pre-1804 version of Azure Stack, change the -RequiredVersion value to 1.2.11.
  Install-Module -Name AzureStack -RequiredVersion 1.4.0 

  ```

  如果安裝成功，輸出中會顯示 AzureRM 和 AzureStack 模組。

- **不具有來自 ASDK 主機電腦的網際網路連線**。 在中斷連線的案例中，您必須先使用下列 PowerShell 命令，將 PowerShell 模組下載至具有網際網路連線的機器：

  ```PowerShell
  $Path = "<Path that is used to save the packages>"

  Save-Package `
    -ProviderName NuGet `
    -Source https://www.powershellgallery.com/api/v2 `
    -Name AzureRM `
    -Path $Path `
    -Force `
    -RequiredVersion 1.2.11

  Save-Package `
    -ProviderName NuGet `
    -Source https://www.powershellgallery.com/api/v2 `
    -Name AzureStack `
    -Path $Path `
    -Force `
  # Install Azure Stack Module Version 1.4.0. If running a pre-1804 version of Azure Stack, change the -RequiredVersion value to 1.2.11.  
    -RequiredVersion 1.4.0
  ```

  接著，將下載的套件複製到 ASDK 電腦，然後註冊該位置作為預設存放庫，並從此存放庫安裝 AzureRM 與 AzureStack 模組：

    ```PowerShell  
    $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
    $RepoName = "MyNuGetSource"

    Register-PSRepository `
      -Name $RepoName `
      -SourceLocation $SourceLocation `
      -InstallationPolicy Trusted

    Install-Module AzureRM `
      -Repository $RepoName

    Install-Module AzureStack `
      -Repository $RepoName
    ```

## <a name="download-the-azure-stack-tools"></a>下載 Azure Stack 工具

[AzureStack-Tools](https://github.com/Azure/AzureStack-Tools) 是裝載 PowerShell 模組的 GitHub 存放庫，可用來管理資源並部署至 Azure Stack。 若要取得這些工具，請複製 GitHub 存放庫或下載 AzureStack-Tools 資料夾，方法是執行下列指令碼：

  ```PowerShell
  # Change directory to the root directory. 
  cd \

  # Enforce usage of TLSv1.2 to download the Azure Stack tools archive from GitHub
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
  invoke-webrequest `
    https://github.com/Azure/AzureStack-Tools/archive/master.zip `
    -OutFile master.zip

  # Expand the downloaded files.
  expand-archive master.zip `
    -DestinationPath . `
    -Force

  # Change to the tools directory.
  cd AzureStack-Tools-master
  ```

## <a name="validate-the-asdk-installation"></a>驗證 ASDK 安裝
若要確保 ASDK 部署成功，您可以依照下列步驟來使用 Test-AzureStack Cmdlet：

1. 以 AzureStack\AzureStackAdmin 身分登入 ASDK 主機電腦。
2. 以系統管理員身分開啟 PowerShell (非 PowerShell ISE)。
3. 執行： `Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint`
4. 執行： `Test-AzureStack`

這些測試需要幾分鐘才能完成。 如果安裝成功，輸出看起來像這樣：

![test-azurestack](media/asdk-post-deploy/test-azurestack.png)

如果發生失敗，請依照疑難排解步驟來取得協助。

## <a name="reset-the-password-expiration-policy"></a>重設密碼到期原則 
為確定開發套件主機的密碼不會在評估期間結束前到期，請在部署 ASDK 之後遵循下列步驟進行。

### <a name="to-change-the-password-expiration-policy-from-powershell"></a>若要從 Powershell 變更密碼到期原則：
從已提升權限的 PowerShell 主控台，執行以下命令：

```powershell
Set-ADDefaultDomainPasswordPolicy -MaxPasswordAge 180.00:00:00 -Identity azurestack.local
```

### <a name="to-change-the-password-expiration-policy-manually"></a>若要手動變更密碼到期原則：
1. 在開發套件主機上，開啟 [群組原則管理] (GPMC.MMC)，並瀏覽至 [群組原則管理] – [樹系：azurestack.local] – [網域] – [azurestack.local]。
2. 以滑鼠右鍵按一下 [預設網域原則]，然後按一下 [編輯]。
3. 在 [群組原則管理編輯器] 中，瀏覽至 [電腦設定] – [原則] – [Windows 設定] – [安全性設定] – [帳戶原則] – [密碼原則]。
4. 在右窗格中，按兩下 [密碼最長使用期限]。
5. 在 [密碼最長使用期限內容] 對話方塊中，將 [密碼到期日] 值變更為 **180**，然後按一下 [確定]。

![群組原則管理主控台](media/asdk-post-deploy/gpmc.png)


## <a name="next-steps"></a>後續步驟
[向 Azure 註冊 ASDK](asdk-register.md)
