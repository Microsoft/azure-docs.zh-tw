---
title: 以使用者的身分使用 PowerShell 連線到 Azure Stack | Microsoft Docs
description: 連線到使用者 Azure Stack 執行個體的步驟。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: F4ED2238-AAF2-4930-AA7F-7C140311E10F
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/07/2018
ms.author: sethm
ms.reviewer: Balsu.G
ms.openlocfilehash: acdad9788737f4f552cedc1b5f42e03e2288dba8
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44159074"
---
# <a name="connect-to-azure-stack-with-powershell-as-a-user"></a>以使用者的身分使用 PowerShell 連線到 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

本文提供連線到您 Azure Stack 執行個體的步驟。 您必須連線，才能使用 PowerShell 來管理 Azure Stack 資源。 例如，您可以使用 PowerShell 來訂閱供應項目、建立虛擬機器及部署 Azure Resource Manager 範本。 以執行 PowerShell Cmdlet。

進行設定：
  - 確定您具有必要項目。
  - 與 Azure Active Directory (Azure AD) 或「Active Directory 同盟服務」(AD FS) 連線。 
  - 註冊資源提供者。
  - 測試連線能力。

## <a name="prerequisites"></a>必要條件

從[開發套件](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop)，或從 Windows 型外部用戶端 (如果您[透過 VPN 連線](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn))，設定下列先決條件：

* 安裝 [Azure Stack 相容的 Azure PowerShell 模組](azure-stack-powershell-install.md)。
* 下載[處理 Azure Stack 所需的工具](azure-stack-powershell-download.md)。

務必使用 Azure Stack 組態中的值取代下列指令碼變數：

- **Azure AD 租用戶名稱**  
  用來管理 Azure Stack 的 Azure AD 租用戶名稱，例如 yourdirectory.onmicrosoft.com。
- **Azure Resource Manager 端點**  
  針對 Azure Stack 開發套件，此值會設定為 https://management.local.azurestack.external。 若要取得 Azure Stack 整合式系統的這個值，請與您的服務提供者連絡。

## <a name="connect-with-azure-ad"></a>與 Azure AD 連線

  ```PowerShell
  $AADTenantName = "yourdirectory.onmicrosoft.com"
  $ArmEndpoint = "https://management.local.azurestack.external"

  # Register an Azure Resource Manager environment that targets your Azure Stack instance
  Add-AzureRMEnvironment `
    -Name "AzureStackUser" `
    -ArmEndpoint $ArmEndpoint

  $AuthEndpoint = (Get-AzureRmEnvironment -Name "AzureStackUser").ActiveDirectoryAuthority.TrimEnd('/')
  $TenantId = (invoke-restmethod "$($AuthEndpoint)/$($AADTenantName)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

  # Sign in to your environment
  Login-AzureRmAccount `
    -EnvironmentName "AzureStackUser" `
    -TenantId $TenantId
   ```

## <a name="connect-with-ad-fs"></a>與 AD FS 連線

  ```PowerShell  
  $ArmEndpoint = "https://management.local.azurestack.external"

  # Register an Azure Resource Manager environment that targets your Azure Stack instance
  Add-AzureRMEnvironment `
    -Name "AzureStackUser" `
    -ArmEndpoint $ArmEndpoint

  $AuthEndpoint = (Get-AzureRmEnvironment -Name "AzureStackUser").ActiveDirectoryAuthority.TrimEnd('/')
  $tenantId = (invoke-restmethod "$($AuthEndpoint)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

  # Sign in to your environment
  Login-AzureRmAccount `
    -EnvironmentName "AzureStackUser" `
    -TenantId $tenantId
  ```

## <a name="register-resource-providers"></a>註冊資源提供者

沒有透過入口網站部署任何資源的新使用者訂用帳戶，並不會自動註冊資源提供者。 執行下列指令碼，即可明確註冊資源提供者：

```PowerShell  
foreach($s in (Get-AzureRmSubscription)) {
        Select-AzureRmSubscription -SubscriptionId $s.SubscriptionId | Out-Null
        Write-Progress $($s.SubscriptionId + " : " + $s.SubscriptionName)
Get-AzureRmResourceProvider -ListAvailable | Register-AzureRmResourceProvider -Force
    }
```

## <a name="test-the-connectivity"></a>測試連線

當您將一切設定妥當時，使用 PowerShell 在 Azure Stack 中建立資源，以測試連線能力。 在測試時，建立應用程式的資源群組並新增虛擬機器。 若要建立名為 "MyResourceGroup" 的資源群組，請執行下列命令：

```PowerShell  
New-AzureRmResourceGroup -Name "MyResourceGroup" -Location "Local"
```

## <a name="next-steps"></a>後續步驟

- [開發適用於 Azure Stack 的範本](azure-stack-develop-templates.md)
- [使用 PowerShell 部署範本](azure-stack-deploy-template-powershell.md)
- 如果想要針對雲端操作員環境設定 PowerShell，請參閱[設定 Azure Stack 操作員的 PowerShell 環境](../azure-stack-powershell-configure-admin.md)一文。