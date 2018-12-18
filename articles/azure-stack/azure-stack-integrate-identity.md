---
title: Azure Stack 資料中心整合 - 身分識別
description: 了解如何將 Azure Stack AD FS 與您的資料中心 AD FS 整合
services: azure-stack
author: jeffgilb
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 08/07/2018
ms.author: jeffgilb
ms.reviewer: wfayed
keywords: ''
ms.openlocfilehash: 9bbe55e08d7a005d38c5608df39f9285d79eb203
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41947954"
---
# <a name="azure-stack-datacenter-integration---identity"></a>Azure Stack 資料中心整合 - 身分識別
您可以使用 Azure Active Directory (Azure AD) 或 Active Directory Federation Services (AD FS) 作為識別提供者來部署 Azure Stack。 請先選擇識別提供者，才能部署 Azure Stack。 使用 AD FS 的部署也稱為在中斷連線模式中部署 Azure Stack。

下表顯示這兩個身分識別選擇間的差異：

||已中斷與網際網路連線|已連線至網際網路|
|---------|---------|---------|
|計費|必須是容量<br> 僅限 Enterprise 合約 (EA)|容量或預付型方案<br>EA 或雲端解決方案提供者 (CSP)|
|身分識別|必須是 AD FS|Azure AD 或 AD FS|
|Marketplace |支援<br>BYOL 授權|支援<br>BYOL 授權|
|註冊|建議，需要卸除式媒體<br> 及個別的連接裝置。|自動化|
|修補和更新|必要，需要卸除式媒體<br> 及個別的連接裝置。|可以直接從網際網路將更新套件<br> 下載到 Azure Stack。|

> [!IMPORTANT]
> 若沒有重新部署整個 Azure Stack 解決方案，則無法切換身分識別提供者。

## <a name="active-directory-federation-services-and-graph"></a>Active Directory 同盟服務和 Graph

使用 AD FS 部署可讓現有 Active Directory 樹系中的身分識別來驗證 Azure Stack 中的資源。 這個現有的 Active Directory 樹系需要 AD FS 部署，以允許建立 AD FS 同盟信任。

驗證是身分識別的一部分。 若要在 Azure Stack 中管理角色型存取控制 (RBAC)，必須設定 Graph 元件。 委派資源的存取權後，Graph 元件會使用 LDAP 通訊協定來查閱現有 Active Directory 樹系中的使用者帳戶。

![Azure Stack AD FS 架構](media/azure-stack-integrate-identity/Azure-Stack-ADFS-architecture.png)

現有的 AD FS 是帳戶安全性權杖服務 (STS)，會將宣告傳送至 Azure Stack AD FS (資源 STS)。 在 Azure Stack 中，自動化會對現有 AD FS 的中繼資料端點建立宣告提供者信任。

在現有 AD FS 中，必須設定一個信賴憑證者信任。 這個步驟不是由自動化完成，而是必須由操作員設定。 Azure Stack 中繼資料端點已記錄在 AzureStackStampDeploymentInfo.JSON 檔案中，或透過執行命令 `Get-AzureStackInfo` 的特殊權限端點。

信賴憑證者信任設定也會要求您設定由 Microsoft 提供的宣告轉換規則。

對於 Graph 設定，必須提供具有現有 Active Directory 讀取權限的服務帳戶。 需要此帳戶做為輸入，自動化才能啟用 RBAC 情節。

在最後一個步驟，已針對預設提供者訂用帳戶設定了新擁有者。 登入 Azure Stack 管理員入口網站時，此帳戶具有所有資源的完整存取權。

需求：


|元件|需求|
|---------|---------|
|圖形|Microsoft Active Directory 2012/2012 R2/2016|
|AD FS|Windows Server 2012/2012 R2/2016|

## <a name="setting-up-graph-integration"></a>設定 Graph 整合

Graph 僅支援單一 Active Directory 樹系的整合。 如果存在多個樹系，則只有在設定中指定的樹系會用來擷取使用者和群組。

需要下列資訊，做為自動化參數的輸入：


|參數|說明|範例|
|---------|---------|---------|
|CustomADGlobalCatalog|您想整合的<br>目標 Active Directory 樹系的 FQDN|Contoso.com|
|CustomADAdminCredentials|具有 LDAP 讀取權限的使用者|YOURDOMAIN\graphservice|

### <a name="create-user-account-in-the-existing-active-directory-optional"></a>在現有 Active Directory 中建立使用者帳戶 (選擇性)

您可以選擇性在現有的 Active Directory 中建立用於 Graph 服務的帳戶。 如果還沒有想要使用的帳戶，請執行此步驟。

1. 在現有的 Active Directory 中，建立下列使用者帳戶 (建議)：
   - **使用者名稱**：graphservice
   - **密碼**：使用強式密碼<br>將密碼設定為永不到期。

   不需要任何特殊權限或成員資格。

#### <a name="trigger-automation-to-configure-graph"></a>觸發自動化來設定圖形

針對此程序，請使用您資料中心網路內能夠與 Azure Stack 中具特殊權限端點通訊的電腦。

2. 開啟一個已提高權限的 Windows PowerShell 工作階段 (以系統管理員身分執行)，然後連線到具特殊權限端點的 IP 位址。 使用適用於 **CloudAdmin** 的認證進行驗證。

   ```PowerShell  
   $creds = Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

3. 您已連接到特殊權限端點，請執行下列命令： 

   ```PowerShell  
   Register-DirectoryService -CustomADGlobalCatalog contoso.com
   ```

   出現提示時，指定要用於 Graph 服務的使用者帳戶之認證 (例如 graphservice)。 Register-DirectoryService Cmdlet 的輸入必須是樹系名稱 / 樹系中的根網域，而不是樹系中的其他網域。

   > [!IMPORTANT]
   > 等待認證快顯 (特殊權限端點中不支援 Get-Credential)，然後輸入 Graph 服務帳戶認證。

#### <a name="graph-protocols-and-ports"></a>Graph 通訊協定和連接埠

Azure Stack 中的 Graph 服務使用以下通訊協定和連接埠來與可寫入的「通用類別目錄伺服器」(GC) 和「金鑰發佈中心」(KDC) (可在目標 Active Directory 樹系中處理登入要求) 通訊。

Azure Stack 中的 Graph 服務會使用下列通訊協定和連接埠來與目標 Active Directory 通訊：

|類型|Port|通訊協定|
|---------|---------|---------|
|LDAP|389|TCP 和 UDP|
|LDAP SSL|636|TCP|
|LDAP GC|3268|TCP|
|LDAP GC SSL|3269|TCP|

## <a name="setting-up-ad-fs-integration-by-downloading-federation-metadata"></a>藉由下載同盟中繼資料來設定 AD FS 整合

需要下列資訊，作為自動化參數的輸入：

|參數|說明|範例|
|---------|---------|---------|
|CustomAdfsName|宣告提供者的名稱。<cr>在 AD FS 登陸頁面上的顯示方式。|Contoso|
|CustomAD<br>FSFederationMetadataEndpointUri|同盟中繼資料連結|https://ad01.contoso.com/federationmetadata/2007-06/federationmetadata.xml|


### <a name="trigger-automation-to-configure-claims-provider-trust-in-azure-stack"></a>觸發自動化以在 Azure Stack 中設定宣告提供者信任

針對此程序，請使用能夠與 Azure Stack 中具特殊權限端點通訊的電腦。 預期 Azure Stack 會信任帳戶 **STS AD FS** 所使用的憑證。

1. 開啟已提高權限的 Windows PowerShell 工作階段，然後連線到特殊權限端點。

   ```PowerShell  
   $creds = Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. 既然您已連接到特殊權限端點，請使用適用於您環境的參數執行下列命令：

   ```PowerShell  
   Register-CustomAdfs -CustomAdfsName Contoso -CustomADFSFederationMetadataEndpointUri https://win-SQOOJN70SGL.contoso.com/federationmetadata/2007-06/federationmetadata.xml
   ```

3. 使用適用於您環境的參數，執行下列命令來更新預設提供者訂用帳戶的擁有者：

   ```PowerShell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "administrator@contoso.com"
   ```

## <a name="setting-up-ad-fs-integration-by-providing-federation-metadata-file"></a>藉由提供同盟中繼資料檔案設定 AD FS 整合

從 1807 版開始，如果下列任一個條件成立，便請使用此方法：

- 相較於 Azure Stack 中的其他所有端點，憑證鏈結對 AD FS 而言不同。
- 從 Azure Stack 的 AD FS 執行個體到現有 AD FS 伺服器沒有網路連線。

需要下列資訊，作為自動化參數的輸入：


|參數|說明|範例|
|---------|---------|---------|
|CustomAdfsName|宣告提供者的名稱。 在 AD FS 登陸頁面上的顯示方式。|Contoso|
|CustomADFSFederationMetadataFileContent|中繼資料內容|$using:federationMetadataFileContent|



### <a name="create-federation-metadata-file"></a>建立同盟中繼資料檔案

對於下列程序，您必須使用與現有 AD FS 部署具有網路連線的電腦，它會成為帳戶 STS。 此外，必須安裝必要的憑證。

1. 開啟提升權限的 Windows PowerShell 工作階段，並使用適用於您環境的參數執行下列命令：

   ```PowerShell  
   [XML]$Metadata = Invoke-WebRequest -URI https://win-SQOOJN70SGL.contoso.com/federationmetadata/2007-06/federationmetadata.xml -UseBasicParsing

   $Metadata.outerxml|out-file c:\metadata.xml
   ```

2. 將中繼資料複製到能夠與具特殊權限端點通訊的電腦。

### <a name="trigger-automation-to-configure-claims-provider-trust-in-azure-stack"></a>觸發自動化以在 Azure Stack 中設定宣告提供者信任

在此程序中，請使用可與 Azure Stack 中具特殊權限端點通訊、且可存取您在上一個步驟中所建立中繼資料檔案的電腦。

1. 開啟已提高權限的 [Windows PowerShell] 工作階段。

   ```PowerShell  
   $federationMetadataFileContent = get-content c:\metadata.cml
   $creds=Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   Register-CustomAdfs -CustomAdfsName Contoso -CustomADFSFederationMetadataFileContent $using:federationMetadataFileContent
   ```

2. 使用適用於您環境的參數，執行下列命令來更新預設提供者訂用帳戶的擁有者：

   ```PowerShell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "administrator@contoso.com"
   ```

## <a name="configure-relying-party-on-existing-ad-fs-deployment-account-sts"></a>在現有的 AD FS 部署上設定信賴憑證者 (帳戶 STS)

Microsoft 提供可設定信賴憑證者信任 (包括宣告轉換規則) 的指令碼。 使用指令碼是選擇性的，因為您可以手動執行命令。

您可以從 Github 上的 [Azure Stack Tools](https://github.com/Azure/AzureStack-Tools/tree/vnext/DatacenterIntegration/Identity) 下載協助程式指令碼。

如果您決定要手動執行命令，請遵循下列步驟：

1. 將下列內容複製到您的資料中心的 AD FS 執行個體或伺服器陣列成員上的 .txt 檔案中 (例如，儲存為 c:\ClaimRules.txt)：

   ```text
   @RuleTemplate = "LdapClaims"
   @RuleName = "Name claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"), query = ";userPrincipalName;{0}", param = c.Value);

   @RuleTemplate = "LdapClaims"
   @RuleName = "UPN claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"), query = ";userPrincipalName;{0}", param = c.Value);

   @RuleTemplate = "LdapClaims"
   @RuleName = "ObjectID claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid"]
   => issue(Type = "http://schemas.microsoft.com/identity/claims/objectidentifier", Issuer = c.Issuer, OriginalIssuer = c.OriginalIssuer, Value = c.Value, ValueType = c.ValueType);

   @RuleName = "Family Name and Given claim"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
   => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname", "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname"), query = ";sn,givenName;{0}", param = c.Value);

   @RuleTemplate = "PassThroughClaims"
   @RuleName = "Pass through all Group SID claims"
   c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"]
   => issue(claim = c);

   @RuleTemplate = "PassThroughClaims"
   @RuleName = "Pass through all windows account name claims"
   c:[Type == http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname]
   => issue(claim = c);
   ```

2. 若要啟用 Windows Forms 式驗證，請以提高權限使用者的身分開啟 Windows PowerShell 工作階段，並執行下列命令：

   ```PowerShell  
   Set-AdfsProperties -WIASupportedUserAgents @("MSAuthHost/1.0/In-Domain","MSIPC","Windows Rights Management Client","Kloud")
   ```

3. 若要新增信賴憑證者信任，請在您的 AD FS 執行個體或伺服器陣列成員上執行下列 Windows PowerShell 命令。 請務必更新 AD FS 端點，並指向步驟 1 中建立的檔案。

   **針對 AD FS 2016**

   ```PowerShell  
   Add-ADFSRelyingPartyTrust -Name AzureStack -MetadataUrl "https://YourAzureStackADFSEndpoint/FederationMetadata/2007-06/FederationMetadata.xml" -IssuanceTransformRulesFile "C:\ClaimIssuanceRules.txt" -AutoUpdateEnabled:$true -MonitoringEnabled:$true -enabled:$true -AccessControlPolicyName "Permit everyone"
   ```

   **針對 AD FS 2012/2012 R2**

   ```PowerShell  
   Add-ADFSRelyingPartyTrust -Name AzureStack -MetadataUrl "https://YourAzureStackADFSEndpoint/FederationMetadata/2007-06/FederationMetadata.xml" -IssuanceTransformRulesFile "C:\ClaimIssuanceRules.txt" -AutoUpdateEnabled:$true -MonitoringEnabled:$true -enabled:$true
   ```

   > [!IMPORTANT]
   > 使用 Windows Server 2012 或 2012 R2 AD FS 時，必須使用 AD FS MMC 嵌入式管理單元來設定發行授權規則。

4. 使用 Internet Explorer 或 Edge 瀏覽器來存取 Azure Stack 時，您必須忽略權杖繫結。 否則，登入嘗試會失敗。 在您的 AD FS 執行個體或伺服器陣列成員上，執行下列命令：

   > [!note]  
   > 使用 Windows Server 2012 或 2012 R2 AD FS 時，不適用此步驟。 您可以放心略過此命令，並繼續進行整合。

   ```PowerShell  
   Set-AdfsProperties -IgnoreTokenBinding $true
   ```

5. Azure Stack 入口網站和工具 (Visual Studio) 需要有重新整理權杖。 必須藉由仰賴憑證者信任來設定這些權杖。 開啟提升權限的 Windows PowerShell 工作階段，並執行下列命令：

   ```PowerShell  
   Set-ADFSRelyingPartyTrust -TargetName AzureStack -TokenLifeTime 1440
   ```

## <a name="spn-creation"></a>建立 SPN

有許多情節需要使用服務主體名稱 (SPN) 進行驗證。 以下是一些範例：

- Azure Stack 的 AD FS 部署 CLI 使用方式
- 使用 AD FS 部署時的 System Center Management Pack for Azure Stack
- 使用 AD FS 部署時 Azure Stack 中的資源提供者
- 各種應用程式
- 需要非互動式登入

> [!Important]  
> AD FS 僅支援互動式登入工作階段。 如果您的自動化案例需要非互動式登入，則必須使用 SPN。

如需有關建立 SPN 的詳細資訊，請參閱[為 AD FS 建立服務主體](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals#create-service-principal-for-ad-fs)。


## <a name="troubleshooting"></a>疑難排解

### <a name="configuration-rollback"></a>設定復原

如果發生錯誤，使得環境處於您不再可以驗證的狀態，即可使用回復選項。

1. 開啟提升權限的 Windows PowerShell 工作階段，並執行下列命令：

   ```PowerShell  
   $creds = Get-Credential
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. 然後執行下列 Cmdlet：

   ```PowerShell  
   Reset-DatacenterIntegationConfiguration
   ```

   執行回復動作之後，會回復所有組態變更。 只能使用內建的 **CloudAdmin** 使用者進行驗證。

   > [!IMPORTANT]
   > 您必須設定預設提供者訂用帳戶的原始擁有者

   ```PowerShell  
   Set-ServiceAdminOwner -ServiceAdminOwnerUpn "azurestackadmin@[Internal Domain]"
   ```

### <a name="collecting-additional-logs"></a>收集其他記錄檔

如果任一個 Cmdlet 失敗，您可以使用 `Get-Azurestacklogs` Cmdlet 來收集其他記錄檔。

1. 開啟提升權限的 Windows PowerShell 工作階段，並執行下列命令：

   ```PowerShell  
   $creds = Get-Credential
   Enter-pssession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $creds
   ```

2. 然後執行下列 Cmdlet：

   ```PowerShell  
   Get-AzureStackLog -OutputPath \\myworstation\AzureStackLogs -FilterByRole ECE
   ```


## <a name="next-steps"></a>後續步驟

[整合外部監視解決方案](azure-stack-integrate-monitor.md)
