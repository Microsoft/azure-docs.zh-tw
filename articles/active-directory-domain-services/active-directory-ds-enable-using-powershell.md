---
title: 使用 PowerShell 啟用 Azure Active Directory Domain Services | Microsoft Docs
description: 使用 PowerShell 啟用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: d4bc5583-6537-4cd9-bc4b-7712fdd9272a
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/06/2017
ms.author: maheshu
ms.openlocfilehash: ee3f65b7afde19a1f9c730334043cc7dae9ea791
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503803"
---
# <a name="enable-azure-active-directory-domain-services-using-powershell"></a>使用 PowerShell 啟用 Azure Active Directory Domain Services
本文說明如何使用 PowerShell 啟用 Azure Active Directory (AD) Domain Services。

## <a name="task-1-install-the-required-powershell-modules"></a>工作 1：安裝必要的 PowerShell 模組

### <a name="install-and-configure-azure-ad-powershell"></a>安裝並設定 Azure AD PowerShell
請遵循文章中的指示[安裝 Azure AD PowerShell 模組並連線至 Azure AD](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?toc=%2fazure%2factive-directory-domain-services%2ftoc.json)。

### <a name="install-and-configure-azure-powershell"></a>安裝並設定 Azure PowerShell
請遵循文章中的指示[安裝 Azure PowerShell 模組並連線至 Azure 訂用帳戶](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?toc=%2fazure%2factive-directory-domain-services%2ftoc.json)。


## <a name="task-2-create-the-required-service-principal-in-your-azure-ad-directory"></a>工作 2：在 Azure AD 目錄中建立必要的服務主體
輸入下列 PowerShell 命令，為 Azure AD 目錄中的 Azure AD Domain Services 建立所需的服務主體。
```powershell
# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId “2565bd9d-da50-47d4-8b85-4c97f669dc36”
```

## <a name="task-3-create-and-configure-the-aad-dc-administrators-group"></a>工作 3：建立和設定「AAD DC 系統管理員」群組
下一個工作是建立系統管理員群組，將用來在受控網域上委派系統管理工作。
```powershell
# Create the delegated administration group for AAD Domain Services.
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"
```

現在您已建立群組，請將幾個使用者新增至群組。
```powershell
# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso100.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

## <a name="task-4-register-the-azure-ad-domain-services-resource-provider"></a>工作 4：註冊 Azure AD Domain Services 資源提供者
輸入下列 PowerShell 命令以註冊 Azure AD Domain Services 的資源提供者：
```powershell
# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.AAD
```

## <a name="task-5-create-a-resource-group"></a>工作 5：建立資源群組
輸入下列 PowerShell 命令以建立資源群組：
```powershell
$ResourceGroupName = "ContosoAaddsRg"
$AzureLocation = "westus"

# Create the resource group.
New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation
```

在此資源群組中，您可以建立虛擬網路與 Azure AD Domain Services 的受控網域。


## <a name="task-6-create-and-configure-the-virtual-network"></a>工作 6：建立和設定虛擬網路
現在，請建立您要在其中啟用 Azure AD Domain Services 的虛擬網路。 請確認您已建立 Azure AD Domain Services 的專用子網路。 請勿將工作負載虛擬機器部署到此專用子網路。

輸入下列 PowerShell 命令來建立具有 Azure AD Domain Services 專用子網路的虛擬網路。

```powershell
$ResourceGroupName = "ContosoAaddsRg"
$VnetName = "DomainServicesVNet_WUS"

# Create the dedicated subnet for AAD Domain Services.
$AaddsSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location westus `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
```


## <a name="task-7-provision-the-azure-ad-domain-services-managed-domain"></a>工作 7：佈建 Azure AD Domain Services 的受控網域
輸入下列 PowerShell 命令，針對您的目錄啟用 Azure AD Domain Services：

```powershell
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "contoso100.com"
$ResourceGroupName = "ContosoAaddsRg"
$VnetName = "DomainServicesVNet_WUS"
$AzureLocation = "westus"

# Enable Azure AD Domain Services for the directory.
New-AzureRmResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -ApiVersion 2017-06-01 -Force -Verbose
```

> [!WARNING]
> **佈建受控網域之後，別忘了其他設定步驟。**
> 佈建受控網域之後，您仍然需要完成下列工作：
> * 為虛擬網路**[更新 DNS 設定](active-directory-ds-getting-started-dns.md)**，讓虛擬機器可以找到受管理的網域來進行網域聯結或驗證。
* **[啟用 Azure AD Domain Services 的密碼同步化](active-directory-ds-getting-started-password-sync.md)**，讓使用者可使用他們的公司認證來登入受管理的網域。
>


## <a name="powershell-script"></a>PowerShell 指令碼
用來執行本文中所列出所有工作的 PowerShell 指令碼如下所示。 複製指令碼，並將它儲存至使用 '.ps1' 副檔名的檔案。 在 PowerShell 中或使用 PowerShell 整合式指令碼環境 (ISE) 執行指令碼。

> [!NOTE]
> **執行此指令碼所需的權限**若要啟用 Azure AD Domain Services，您必須是 Azure AD 目錄的全域管理員。 此外，您必須要有虛擬網路的「參與者」權限，才可啟用 Azure AD Domain Services。
>

```powershell
# Change the following values to match your deployment.
$AaddsAdminUserUpn = "admin@contoso100.onmicrosoft.com"
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "contoso100.com"
$ResourceGroupName = "ContosoAaddsRg"
$VnetName = "DomainServicesVNet_WUS"
$AzureLocation = "westus"

# Connect to your Azure AD directory.
Connect-AzureAD

# Login to your Azure subscription.
Connect-AzureRmAccount

# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId “2565bd9d-da50-47d4-8b85-4c97f669dc36”

# Create the delegated administration group for AAD Domain Services.
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"

# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq '$AaddsAdminUserUpn'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId

# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.AAD

# Create the resource group.
New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation

# Create the dedicated subnet for AAD Domain Services.
$AaddsSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $AzureLocation `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet

# Enable Azure AD Domain Services for the directory.
New-AzureRmResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -ApiVersion 2017-06-01 -Force -Verbose
```

> [!WARNING]
> **佈建受控網域之後，別忘了其他設定步驟。**
> 佈建受控網域之後，您仍然需要完成下列工作：
> * 為虛擬網路更新 DNS 設定，讓虛擬機器可以找到受控網域來進行網域聯結或驗證。
* 啟用 Azure AD Domain Services 的密碼同步化，讓使用者可使用他們的公司認證來登入受控網域。
>

## <a name="next-steps"></a>後續步驟
建立受控網域之後，請執行下列設定工作，如此一來您就可以使用受控網域：

* [更新虛擬網路的 DNS 伺服器設定，以指向受控網域](active-directory-ds-getting-started-dns.md)
* [為受控網域啟用密碼同步化](active-directory-ds-getting-started-password-sync.md)
