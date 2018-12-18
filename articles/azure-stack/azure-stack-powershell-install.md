---
title: 安裝適用於 Azure Stack 的 PowerShell | Microsoft Docs
description: 瞭解如何安裝適用於 Azure Stack 的 PowerShell。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: article
ms.date: 08/10/2018
ms.author: sethm
ms.reviewer: thoroet
ms.openlocfilehash: 2619f959dbefba84ea1a4d5aa974055998b78b5a
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "41946607"
---
# <a name="install-powershell-for-azure-stack"></a>安裝適用於 Azure Stack 的 PowerShell

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您將必須安裝與 Azure Stack 相容的 PowerShell 模組，才能與您的雲端搭配運作。 啟用相容性時，會透過名為「API 設定檔」的功能來啟用。

API 設定檔提供一個管理 Azure 與 Azure Stack 之間版本差異的方式。 API 版本設定檔是一組具有特定 API 版本的 Azure Resource Manager PowerShell 模組。 每個雲端平台都有一組支援的 API 版本設定檔。 例如，Azure Stack 支援特定日期的設定檔版本 (例如 **2017-03-09-profile**)，而 Azure 則支援**最新的** API 版本設定檔。 當您安裝設定檔時，會安裝與所指定設定檔對應的 Azure Resource Manager PowerShell 模組。

您可以在已連線至網際網路、部分連線或已中斷連線的情況下，安裝與 Azure Stack 相容的 PowerShell 模組。 在本文中，我們將逐步解說針對這些情況安裝適用於 Azure Stack 之 PowerShell 的詳細指示。

## <a name="1-verify-your-prerequisites"></a>1.驗證您的必要條件

開始使用 Azure Stack 和 PowerShell 之前，您必須先備妥幾個需求項目。

- **PowerShell 5.0 版**  
若要檢查您的版本，請執行 $PSVersionTable.PSVersion 並比較**主要**版本。 如果您沒有 PowerShell 5.0，請遵循[連結](https://docs.microsoft.com/powershell/scripting/setup/installing-windows-powershell?view=powershell-6#upgrading-existing-windows-powershell)以升級至 PowerShell 5.0。

  > [!Note]  
  > PowerShell 5.0 需要 Windows 機器。

- **從已提升權限的提示字元執行 PowerShell**  
  您將必須能夠以系統管理權限執行 PowerShell。

- **PowerShell 資源庫存取權**  
  您將必須能夠存取 [PowerShell 資源庫](https://www.powershellgallery.com)。 此資源庫是 PowerShell 內容的中央存放庫。 **PowerShellGet** 模組包含用於探索、安裝、更新及發佈 PowerShell 成品 (例如來自「PowerShell 資源庫」和其他私人存放庫的模組、DSC 資源、角色功能及指令碼) 的 Cmdlet。 如果您是在已中斷連線的情況下使用 PowerShell，您將必須從可連線至網際網路的機器擷取資源，然後將它們儲存在已中斷連線之機器可存取的位置中。


<!-- Nuget? -->

## <a name="2-validate-if-the-powershell-gallery-is-accessible"></a>2.驗證是否可存取 PowerShell 資源庫

驗證 PSGallery 是否已註冊為存放褲。

> [!Note]  
> 此步驟需要存取網際網路。 

開啟已提升權限的 PowerShell 提示字元，然後執行下列 Cmdlet：

````PowerShell  
Import-Module -Name PowerShellGet -ErrorAction Stop
Import-Module -Name PackageManagement -ErrorAction Stop
Get-PSRepository -Name "PSGallery"
````

如果存放庫未註冊，請開啟提升權限的 PowerShell 工作階段，並執行下列命令：

```PowerShell
Register-PsRepository -Default
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
```

## <a name="3-uninstall-existing-versions-of-the-azure-stack-powershell-modules"></a>3.解除安裝現有的 Azure Stack PowerShell 模組版本

在安裝必要版本之前，請確定解除安裝任何先前安裝的 Azure Stack AzureRM PowerShell 模組。 您可以使用下列兩種方法之一來解除安裝它們：

1. 若要將現有的 AzureRM PowerShell 模組解除安裝，請關閉所有作用中的 PowerShell 工作階段，然後執行下列 Cmdlet：

  ````PowerShell  
    Uninstall-Module AzureRM.AzureStackAdmin -Force
    Uninstall-Module AzureRM.AzureStackStorage -Force
    Uninstall-Module -Name AzureStack -Force
  ````

2. 從 `C:\Program Files\WindowsPowerShell\Modules` 和 `C:\Users\AzureStackAdmin\Documents\WindowsPowerShell\Modules` 資料夾中刪除開頭為 `Azure` 的所有資料夾。 刪除這些資料夾會移除任何現有的 PowerShell 模組。

## <a name="4-connected-install-powershell-for-azure-stack-with-internet-connectivity"></a>4.已連線：在具備網際網路連線能力的情況下安裝適用於 Azure Stack 的 PowerShell

Azure Stack 需要 **2017-03-09-profile** API 版本設定檔 (藉由安裝 **AzureRM.Bootstrapper** 模組即可取得)。 除了 AzureRM 模組外，您也應該安裝 Azure Stack 特有的 Azure PowerShell 模組。 

執行下列 PowerShell 指令碼，在開發工作站上安裝這些模組：

  - **1.4.0 版** (Azure Stack 1804 或更新版本)

    ```PowerShell  
    # Install the AzureRM.Bootstrapper module. Select Yes when prompted to install NuGet 
    Install-Module -Name AzureRm.BootStrapper 

    # Install and import the API Version Profile required by Azure Stack into the current PowerShell session. 
    Use-AzureRmProfile -Profile 2017-03-09-profile -Force 

    # Install Module Version 1.4.0 if Azure Stack is running 1804 at a minimum 
    Install-Module -Name AzureStack -RequiredVersion 1.4.0
    ```

- **1.2.11 版** (1804 之前的版本)

    ```PowerShell  
    # Install the AzureRM.Bootstrapper module. Select Yes when prompted to install NuGet 
    Install-Module -Name AzureRm.BootStrapper 

    # Install and import the API Version Profile required by Azure Stack into the current PowerShell session. 
    Use-AzureRmProfile -Profile 2017-03-09-profile -Force 

    # Install Module Version 1.2.11 if Azure Stack is running a lower version than 1804 
    Install-Module -Name AzureStack -RequiredVersion 1.2.11 
    ```

執行下列命令來確認安裝：

```PowerShell  
Get-Module -ListAvailable | where-Object {$_.Name -like "Azs*"}
```

如果安裝成功，輸出中會顯示 AzureRM 和 AzureStack 模組。

## <a name="5-disconnected-install-powershell-without-an-internet-connection"></a>5.已中斷連線：在沒有網際網路連線的情況下安裝 PowerShell

在中斷連線的情況中，您必須先將 PowerShell 模組下載至具有網際網路連線的電腦，然後將其傳送至 Azure Stack 開發套件進行安裝。

登入具有網際網路連線能力的電腦，然後根據您的 Azure Stack 版本，使用下列指令碼將 Azure Resource Manager 和 Azure Stack 套件下載到您的本機電腦。


  - **1.3.0 版** (Azure Stack 1804 或更新版本)
  
    > [!Note]  
    若要從 1.2.11 版升級，請參閱[移轉指南](https://aka.ms/azspowershellmigration)。

    ````PowerShell  
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop

      $Path = "<Path that is used to save the packages>"
      Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.4.0
    ````

  - **1.2.11 版** (1804 之前的版本)

    ````PowerShell  
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop

      $Path = "<Path that is used to save the packages>"

      Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 1.2.11
    ````

2. 將下載的套件複製到 USB 裝置。

3. 登入工作站，並將套件從 USB 裝置複製到工作站上的位置。

4. 現在您必須註冊此位置作為預設存放庫，並從此存放庫安裝 AzureRM 與 AzureStack 模組：

   ```PowerShell
   #requires -Version 5
   #requires -RunAsAdministrator
   #requires -Module PowerShellGet
   #requires -Module PackageManagement

   $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
   $RepoName = "MyNuGetSource"

   Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation  -InstallationPolicy Trusted

   Install-Module AzureRM -Repository $RepoName

   Install-Module AzureStack -Repository $RepoName 
   ```

## <a name="6-configure-powershell-to-use-a-proxy-server"></a>6.設定 PowerShell 使用 Proxy 伺服器

在需要 Proxy 伺服器才能存取網際網路的情況下，您必須先將 PowerShell 設定成使用現有的 Proxy 伺服器。

1. 開啟提升權限的 PowerShell 提示字元。
2. 執行下列命令：

````PowerShell  
  #To use Windows credentials for proxy authentication
  [System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials

  #Alternatively, to prompt for separate credentials that can be used for #proxy authentication

  [System.Net.WebRequest]::DefaultWebProxy.Credentials = Get-Credential
````

## <a name="next-steps"></a>後續步驟

 - [從 GitHub 下載 Azure Stack 工具](azure-stack-powershell-download.md)
 - [設定 Azure Stack 使用者的 PowerShell 環境](user/azure-stack-powershell-configure-user.md)  
 - [設定 Azure Stack 操作員的 PowerShell 環境](azure-stack-powershell-configure-admin.md) 
 - [在 Azure Stack 中管理 API 版本設定檔](user/azure-stack-version-profiles.md)  
