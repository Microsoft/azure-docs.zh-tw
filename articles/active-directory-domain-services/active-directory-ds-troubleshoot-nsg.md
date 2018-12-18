---
title: Azure Active Directory Domain Services：針對網路安全性群組設定進行疑難排解 | Microsoft Docs
description: 針對 Azure AD Domain Services 的 NSG 設定進行疑難排解
services: active-directory-ds
documentationcenter: ''
author: eringreenlee
manager: ''
editor: ''
ms.assetid: 95f970a7-5867-4108-a87e-471fa0910b8c
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/01/2018
ms.author: ergreenl
ms.openlocfilehash: a2acbed81e323718c7d294d87ebf699c35664d02
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502640"
---
# <a name="troubleshoot-invalid-networking-configuration-for-your-managed-domain"></a>針對受控網域的無效網路設定進行移難排解
本文將協助您針對導致下列警示訊息的網路相關設定錯誤進行疑難排解：

## <a name="alert-aadds104-network-error"></a>警示 AADDS104：網路錯誤
**警示訊息：***Microsoft 無法觸達此受控網域的網域控制站。如果虛擬網路上設定的網路安全性群組 (NSG) 封鎖受控網域的存取，就可能發生這種情況。另一個可能的原因，是使用者定義的路由封鎖了來自網際網路的連入流量。*

Azure AD Domain Services 網路錯誤最常見的原因是 NSG 設定無效。 為您虛擬網路設定的「網路安全性群組」(NSG) 必須允許存取[特定連接埠](active-directory-ds-networking.md#ports-required-for-azure-ad-domain-services)。 如果這些連接埠遭到封鎖，Microsoft 便無法監視或更新您的受控網域。 此外，也會影響到 Azure AD 目錄與受控網域之間的同步處理。 建立 NSG 時，請將這些連接埠保持開啟，以避免服務中斷。

### <a name="checking-your-nsg-for-compliance"></a>檢查 NSG 的合規性

1. 在 Azure 入口網站中，瀏覽至 [網路安全性群組](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Network%2FNetworkSecurityGroups) 頁面
2. 從表格中，選擇與已啟用您受控網域的子網路關聯的 NSG。
3. 在左窗格的 [設定] 下，按一下 [輸入安全性規則]
4. 檢閱備妥的規則，並識別哪些規則會封鎖存取[這些連接埠](active-directory-ds-networking.md#ports-required-for-azure-ad-domain-services)
5. 對 NSG 進行以下編輯以確保合規性：刪除規則、新增規則，或建立全新的 NSG。 [新增規則](#add-a-rule-to-a-network-security-group-using-the-azure-portal)或[建立全新符合規範的 NSG](#create-a-nsg-for-azure-ad-domain-services-using-powershell) 的步驟如下

## <a name="sample-nsg"></a>NSG 範例
下表描述的 NSG 範例，能讓受控網域保持安全，同時允許 Microsoft 監視、管理及更新資訊。

![NSG 範例](.\media\active-directory-domain-services-alerts\default-nsg.png)

>[!NOTE]
> Azure AD Domain Services 會要求來自虛擬網路的對外存取不受限制。 建議您不要建立任何額外的 NSG 規則來限制虛擬網路的對外存取。

## <a name="add-a-rule-to-a-network-security-group-using-the-azure-portal"></a>使用 Azure 入口網站將規則新增至網路安全性群組
如果您不想使用 PowerShell，則可使用 Azure 入口網站，手動對 NSG 新增單一規則。 若要在網路安全性群組中建立規則，請完成下列步驟：

1. 在 Azure 入口網站中，瀏覽至 [網路安全性群組](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Network%2FNetworkSecurityGroups) 頁面。
2. 從表格中，選擇與已啟用您受控網域的子網路關聯的 NSG。
3. 在左側面板的 [設定] 底下，按一下 [輸入安全性規則] 或 [輸出安全性規則]。
4. 按一下 [新增] 並填入資訊來建立規則。 按一下 [確定]。
5. 在規則資料表中尋找您的規則，確認規則已建立。


## <a name="create-a-nsg-for-azure-ad-domain-services-using-powershell"></a>使用 PowerShell 為 Azure AD Domain Services 建立 NSG
此 NSG 會設定成允許對 Azure AD Domain Services 所需連接埠的輸入流量，但拒絕任何其他不需要的對內存取。

**先決條件：安裝並設定 Azure PowerShell。** 請依照指示來[安裝 Azure PowerShell 模組並連線至您的 Azure 訂用帳戶](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?toc=%2fazure%2factive-directory-domain-services%2ftoc.json) \(英文\)。

>[!TIP]
> 建議您使用最新版的 Azure PowerShell 模組。 如果您已經安裝舊版的 Azure PowerShell 模組，則請更新至最新版本。
>

依下列步驟使用 PowerShell 來建立新的 NSG。
1. 登入 Azure 訂用帳戶。

  ```PowerShell
  # Log in to your Azure subscription.
  Connect-AzureRmAccount
  ```

2. 建立具有三個規則的 NSG。 下列指令碼會定義 NSG 的三個規則，從而允許系統存取要執行 Azure AD Domain Services 所需使用的連接埠。 接著，此指令碼會建立包含這些規則的新 NSG。 如果在虛擬網路中部署的工作負載需要，請使用相同的格式來新增允許其他輸入流量的額外規則。

  ```PowerShell
  # Allow inbound HTTPS traffic to enable synchronization to your managed domain.
  $SyncRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowSyncWithAzureAD -Description "Allow synchronization with Azure AD" `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 `
  -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 443

  # Allow management of your domain over port 5986 (PowerShell Remoting)
  $PSRemotingRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowPSRemoting -Description "Allow management of domain through port 5986" `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 102 `
  -SourceAddressPrefix 52.180.183.8, 23.101.0.70, 52.225.184.198, 52.179.126.223, 13.74.249.156, 52.187.117.83, 52.161.13.95, 104.40.156.18, 104.40.87.209, 52.180.179.108, 52.175.18.134, 52.138.68.41, 104.41.159.212, 52.169.218.0, 52.187.120.237, 52.161.110.169, 52.174.189.149, 13.64.151.161 -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 5986

  #The following two rules are optional and needed only in certain situations.

  # Allow management of your domain over port 3389 (remote desktop).
  $RemoteDesktopRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowRD -Description "Allow management of domain through port 3389" `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 103 `
  -SourceAddressPrefix 207.68.190.32/27, 13.106.78.32/27, 10.254.32.0/20, 10.97.136.0/22, 13.106.174.32/27, 13.106.4.96/27 -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389

  # Secure LDAP rule, it is recommended to change the source address prefix to include only the IP addresses
  $SecureLDAPRule = New-AzureRmNetworkSecurityRuleConfig -Name SecureLDAP -Description "Allow access through secure LDAP port" `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 104 `
  -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 636

  # Create the NSG with the rules above (if you need the remote desktop rule and secure ldap rule, add it below)
  $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location westus `
  -Name "AADDomainServices-NSG" -SecurityRules $SyncRule, $PSRemotingRule
  ```

3. 最後，將 NSG 與所選擇的 VNet 和子網路建立關聯。

  ```PowerShell
  # Find vnet and subnet
  $Vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $ResourceGroup -Name $VnetName
  $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $Vnet -Name $SubnetName

  # Set the nsg to the subnet and save the changes
  $Subnet.NetworkSecurityGroup = $Nsg
  Set-AzureRmVirtualNetwork -VirtualNetwork $Vnet
  ```

## <a name="full-script-to-create-and-apply-an-nsg-for-azure-ad-domain-services"></a>用以建立及套用 Azure AD Domain Services 之 NSG 的完整指令碼
>[!TIP]
> 建議您使用最新版的 Azure PowerShell 模組。 如果您已經安裝舊版的 Azure PowerShell 模組，則請更新至最新版本。
>

```PowerShell
# Change the following values to match your deployment
$ResourceGroup = "ResourceGroupName"
$Location = "westus"
$VnetName = "exampleVnet"
$SubnetName = "exampleSubnet"

# Log in to your Azure subscription.
Connect-AzureRmAccount

# Allow inbound HTTPS traffic to enable synchronization to your managed domain.
$SyncRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowSyncWithAzureAD -Description "Allow synchronization with Azure AD" `
-Access Allow -Protocol Tcp -Direction Inbound -Priority 101 `
-SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 443

# Allow management of your domain over port 5986 (PowerShell Remoting)
$PSRemotingRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowPSRemoting -Description "Allow management of domain through port 5986" `
-Access Allow -Protocol Tcp -Direction Inbound -Priority 102 `
-SourceAddressPrefix 52.180.183.8, 23.101.0.70, 52.225.184.198, 52.179.126.223, 13.74.249.156, 52.187.117.83, 52.161.13.95, 104.40.156.18, 104.40.87.209, 52.180.179.108, 52.175.18.134, 52.138.68.41, 104.41.159.212, 52.169.218.0, 52.187.120.237, 52.161.110.169, 52.174.189.149, 13.64.151.161 -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 5986

# The following two rules are optional and needed only in certain situations.

# Allow management of your domain over port 3389 (remote desktop).
$RemoteDesktopRule = New-AzureRmNetworkSecurityRuleConfig -Name AllowRD -Description "Allow management of domain through port 3389" `
-Access Allow -Protocol Tcp -Direction Inbound -Priority 103 `
-SourceAddressPrefix 207.68.190.32/27, 13.106.78.32/27, 10.254.32.0/20, 10.97.136.0/22, 13.106.174.32/27, 13.106.4.96/27 -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 3389

# Secure LDAP rule, it is recommended to change the source address prefix to include only the IP addresses
$SecureLDAPRule = New-AzureRmNetworkSecurityRuleConfig -Name SecureLDAP -Description "Allow access through secure LDAP port" `
-Access Allow -Protocol Tcp -Direction Inbound -Priority 104 `
-SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 636

# Create the NSG with the rules above (if you need the remote desktop rule and secure ldap rule, add it below)
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location westus `
-Name "AADDomainServices-NSG" -SecurityRules $SyncRule, $PSRemotingRule

# Find vnet and subnet
$Vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $ResourceGroup -Name $VnetName
$Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $Vnet -Name $SubnetName

# Set the nsg to the subnet and save the changes
$Subnet.NetworkSecurityGroup = $Nsg
Set-AzureRmVirtualNetwork -VirtualNetwork $Vnet
```


## <a name="need-help"></a>需要協助嗎？
請連絡 Azure Active Directory Domain Services 產品小組， [分享意見或尋求支援](active-directory-ds-contact-us.md)。
