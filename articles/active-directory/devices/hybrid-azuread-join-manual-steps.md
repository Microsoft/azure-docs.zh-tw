---
title: 手動設定混合式 Azure Active Directory 已加入裝置 | Microsoft Docs
description: 了解如何手動設定混合式 Azure Active Directory 已加入裝置。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/25/2018
ms.author: markvi
ms.reviewer: sandeo
ms.openlocfilehash: 4e3b7aff97cbcebe34e6af4755900e8888c5e57d
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352792"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-joined-devices-manually"></a>教學課程：手動設定混合式 Azure Active Directory 已加入裝置 

使用 Azure Active Directory (Azure AD) 中的裝置管理，您可以確保使用者會從符合安全性與合規性之標準的裝置來存取您的資源。 如需詳細資料，請參閱 [Azure Active Directory 中的裝置管理簡介](overview.md)。


> [!TIP]
> 如果使用 Azure AD Connect 是您可使用的選項，請參閱[選取您的案例](hybrid-azuread-join-plan.md#select-your-scenario)。 使用 Azure AD Connect，您可以大幅簡化混合式 Azure AD Join 的設定。



如果您有內部部署 Active Directory 環境，而且您想要將加入網域的裝置加入 Azure AD，您可以藉由設定混合式 Azure AD 已加入裝置來完成。 在本教學課程中，您將了解如何手動為您的裝置設定混合式 Azure AD Join。

> [!div class="checklist"]
> * 必要條件
> * 組態步驟
> * 設定服務連接點
> * 設定宣告的發行
> * 啟用舊版 Windows 裝置
> * 確認加入的裝置
> * 對您的實作進行疑難排解
 




## <a name="prerequisites"></a>必要條件

本教學課程假設您已熟悉：
    
-  [Azure Active Directory 中的裝置管理簡介](../device-management-introduction.md)
    
-  [如何規劃混合式 Azure Active Directory Join 實作](hybrid-azuread-join-plan.md)

-  [如何控制裝置的混合式 Azure AD Join](hybrid-azuread-join-control.md)


開始在組織中啟用混合式 Azure AD 已加入裝置之前，您必須先確定：

- 您執行的是最新版的 Azure AD Connect。

- Azure AD Connect 已將您要加入混合式 Azure AD 之裝置的電腦物件同步處理至 Azure AD。 如果電腦物件屬於特定組織單位 (OU)，則您也必須在 Azure AD Connect 中設定這些 OU 的同步處理。

  

Azure AD Connect：

- 保持內部部署 Active Directory (AD) 中的電腦帳戶和 Azure AD 中的裝置物件之間的關聯。 
- 啟用其他裝置相關功能，例如 Windows Hello 企業版。

請確定下列 URL 可從組織網路內的電腦存取，以將電腦註冊至 Azure AD：

- https://enterpriseregistration.windows.net

- https://login.microsoftonline.com 允許
- https://device.login.microsoftonline.com

- 組織的 STS (同盟網域)

如果尚未完成，使用者的本機內部網路設定中應該包括組織的 STS (適用於同盟網域)。

如果您的組織打算使用隨選即用 SSO，那麼您必須能從組織內部的電腦連線到下列 URL，也必須將其新增至使用者的本機內部網路區域：

- https://autologon.microsoftazuread-sso.com

- 此外，請在使用者的內部網路區域啟用下列設定：[允許透過指令碼更新狀態列]。

如果您的組織以內部部署 AD 使用受控 (非同盟) 設定，而不是使用 ADFS 與 Azure AD 同盟，則在 Windows 10 上的混合式 Azure AD，就要靠 AD 中的電腦物件與 Azure AD 同步才能加入。 若要在 Azure AD Connect 同步處理組態中進行同步，請確定已啟用其中包含需要加入混合式 Azure AD 電腦物件的任何組織單位 (OU)。

針對 Windows 10 1703 版或更早版本的裝置，如果組織需要透過輸出 Proxy 存取網際網路，您就必須實作 Web Proxy 自動探索 (WPAD)，以在 Azure AD 中註冊 Windows 10 電腦。 

從 Windows 10 1803 起，即使同盟網域中使用 AD FS 的裝置混嘗試合式 Azure AD 聯結失敗，而且如果 Azure AD Connect 已設定為要將電腦/裝置物件同步處理至 Azure AD，則此裝置會使用已同步處理的電腦/裝置來嘗試完成混合式 Azure AD 聯結。

## <a name="configuration-steps"></a>組態步驟

您可以設定混合式 Azure AD 已加入裝置，以用於各種類型的 Windows 裝置平台。 本主題包含所有一般組態案例所需的步驟。  

使用下表來取得您的案例所需步驟的概觀︰  



| 步驟                                      | 現行 Windows 和密碼雜湊同步處理 | 現行 Windows 和同盟 | 舊版 Windows |
| :--                                        | :-:                                    | :-:                            | :-:                |
| 設定服務連接點 | ![勾選][1]                            | ![勾選][1]                    | ![勾選][1]        |
| 設定宣告的發行           |                                        | ![勾選][1]                    | ![勾選][1]        |
| 啟用非 Windows 10 裝置      |                                        |                                | ![勾選][1]        |
| 確認加入的裝置          | ![勾選][1]                            | ![勾選][1]                    | ![勾選][1]        |



## <a name="configure-service-connection-point"></a>設定服務連接點

您的裝置會在註冊期間使用服務連接點 (SCP) 物件來探索 Azure AD 租用戶資訊。 在內部部署 Active Directory (AD) 中，用於混合式 Azure AD 已加入裝置的 SCP 物件必須存在於電腦樹系的組態命名內容資料分割中。 每個樹系只有一個組態命名內容。 在多樹系 Active Directory 組態中，服務連接點必須存在於包含已加入網域電腦的所有樹系中。

您可以使用 [**Get-ADRootDSE**](https://technet.microsoft.com/library/ee617246.aspx) Cmdlet 來擷取樹系的組態命名內容。  

如果樹系的 Active Directory 網域名稱為 fabrikam.com，則組態命名內容為：

`CN=Configuration,DC=fabrikam,DC=com`

在您的樹系中，用於自動註冊已加入網域裝置的 SCP 物件位於︰  

`CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,[Your Configuration Naming Context]`

視您部署 Azure AD Connect 的方式而定，可能已經設定 SCP 物件。
您可以使用下列 Windows PowerShell 指令碼，確認物件是否存在並擷取探索值︰ 

    $scp = New-Object System.DirectoryServices.DirectoryEntry;

    $scp.Path = "LDAP://CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,CN=Configuration,DC=fabrikam,DC=com";

    $scp.Keywords;

**$scp.Keywords** 的輸出會顯示 Azure AD 租用戶資訊，例如：

    azureADName:microsoft.com
    azureADId:72f988bf-86f1-41af-91ab-2d7cd011db47

如果服務連接點不存在，請在 Azure AD Connect 伺服器上執行 `Initialize-ADSyncDomainJoinedComputerSync` Cmdlet 加以建立。 需要企業系統管理員認證才能執行此 Cmdlet。  
此 Cmdlet：

- 在 Azure AD Connect 連線到的 Active Directory 樹系中建立服務連接點。 
- 要求您指定 `AdConnectorAccount` 參數。 這是在 Azure AD Connect 中設定為 Active Directory 連接器帳戶的帳戶。 


以下指令碼顯示使用此 Cmdlet 的範例。 在此指令碼中，`$aadAdminCred = Get-Credential` 會要求您輸入使用者名稱。 您必須提供使用者主體名稱 (UPN) 格式 (`user@example.com`) 的使用者名稱。 


    Import-Module -Name "C:\Program Files\Microsoft Azure Active Directory Connect\AdPrep\AdSyncPrep.psm1";

    $aadAdminCred = Get-Credential;

    Initialize-ADSyncDomainJoinedComputerSync –AdConnectorAccount [connector account name] -AzureADCredentials $aadAdminCred;

`Initialize-ADSyncDomainJoinedComputerSync` Cmdlet：

- 使用 Active Directory PowerShell 模組和 DS 工具，此模組依賴網域控制站上執行的 Active Directory Web 服務。 執行 Windows Server 2008 R2 和更新版本的網域控制站可支援 Active Directory Web 服務。
- 只有 **MSOnline PowerShell 模組 1.1.166.0 版**才支援。 若要下載此模組，請使用這個[連結](https://msconfiggallery.cloudapp.net/packages/MSOnline/1.1.166.0/)。   
- 若未安裝 AD DS 工具，則 `Initialize-ADSyncDomainJoinedComputerSync` 會失敗。  您可透過 [功能] - [遠端伺服器管理工具] - [角色管理工具] 下的「伺服器管理員」安裝 AD DS 工具。

對於執行 Windows Server 2008 或更早版本的網域控制站，請使用下列指令碼來建立服務連接點。

在多樹系組態中，您應該使用下列指令碼在電腦所在的樹系中建立服務連接點︰
 
    $verifiedDomain = "contoso.com"    # Replace this with any of your verified domain names in Azure AD
    $tenantID = "72f988bf-86f1-41af-91ab-2d7cd011db47"    # Replace this with you tenant ID
    $configNC = "CN=Configuration,DC=corp,DC=contoso,DC=com"    # Replace this with your AD configuration naming context

    $de = New-Object System.DirectoryServices.DirectoryEntry
    $de.Path = "LDAP://CN=Services," + $configNC
    $deDRC = $de.Children.Add("CN=Device Registration Configuration", "container")
    $deDRC.CommitChanges()

    $deSCP = $deDRC.Children.Add("CN=62a0ff2e-97b9-4513-943f-0d221bd30080", "serviceConnectionPoint")
    $deSCP.Properties["keywords"].Add("azureADName:" + $verifiedDomain)
    $deSCP.Properties["keywords"].Add("azureADId:" + $tenantID)

    $deSCP.CommitChanges()

在上述指令碼中，

- `$verifiedDomain = "contoso.com"` 是要使用您 Azure AD 中其中一個已驗證網域名稱來取代的預留位置。 您必須擁有該網域，才能使用它。

如需已驗證之網域名稱的詳細資料，請參閱[將自訂網域名稱新增至 Azure Active Directory](../active-directory-domains-add-azure-portal.md)。  
若要取得已驗證之公司網域的清單，您可以使用 [Get-AzureADDomain](/powershell/module/Azuread/Get-AzureADDomain?view=azureadps-2.0) Cmdlet。 

![Get-AzureADDomain](./media/hybrid-azuread-join-manual-steps/01.png)

## <a name="setup-issuance-of-claims"></a>設定宣告的發行

在同盟 Azure AD 組態中，裝置倚賴 Active Directory Federation Services (AD FS) 或協力廠商內部部署同盟伺服器向 Azure AD 進行驗證。 裝置會進行驗證以取得向 Azure Active Directory 裝置註冊服務 (Azure ADS) 註冊的存取權杖。

現行 Windows 裝置會使用整合式 Windows 驗證向內部部署同盟服務所裝載的作用中 WS-Trust 端點 (1.3 或 2005 版) 進行驗證。

> [!NOTE]
> 使用 AD FS 時，必須啟用 **adfs/services/trust/13/windowstransport** 或 **adfs/services/trust/2005/windowstransport**。 如果您使用 Web 驗證 Proxy，也需確定已透過 Proxy 發佈此端點。 您可以在 AD FS 管理主控台的 [服務] > [端點] 底下看到已啟用的端點。
>
>如果您沒有以 AD FS 做為您的內部部署同盟伺服器，請依照廠商的指示來確定支援 WS-Trust 1.3 或 2005 端點，而這些端點是透過中繼資料交換檔 (MEX) 發佈。

下列宣告必須存在於 Azure DRS 所收到的權杖中，才能完成裝置註冊。 Azure DRS 會使用其中一些資訊在 Azure AD 中建立裝置物件，而 Azure AD Connect 接著會使用此資訊將新建立的裝置物件與內部部署電腦帳戶建立關聯。

* `http://schemas.microsoft.com/ws/2012/01/accounttype`
* `http://schemas.microsoft.com/identity/claims/onpremobjectguid`
* `http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid`

如果您有多個經過驗證的網域名稱，您必須為電腦提供下列宣告︰

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid`

如果您已發出 ImmutableID 宣告 (例如，替代登入 ID)，您必須為電腦提供一個對應宣告：

* `http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`

在下列各節中，您可找到下列相關資訊：
 
- 每個宣告應該具有的值
- 定義在 AD FS 中看起來如何

此定義可協助您確認值是否存在，或者您是否需要加以建立。

> [!NOTE]
> 如果您未將 AD FS 用於您的內部部署同盟伺服器，請依照廠商的指示來建立適當組態以發出這些宣告。

### <a name="issue-account-type-claim"></a>發出帳戶類型宣告

**`http://schemas.microsoft.com/ws/2012/01/accounttype`** - 此宣告必須包含 **DJ** 這個值，此值可將裝置識別為已加入網域的電腦。 在 AD FS 中，您可以新增發行轉換規則，如下所示︰

    @RuleName = "Issue account type for domain-joined computers"
    c:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value = "DJ"
    );

### <a name="issue-objectguid-of-the-computer-account-on-premises"></a>發出內部部署電腦帳戶的 objectGUID

**`http://schemas.microsoft.com/identity/claims/onpremobjectguid`** - 此宣告必須包含內部電腦帳戶的 **objectGUID** 值。 在 AD FS 中，您可以新增發行轉換規則，如下所示︰

    @RuleName = "Issue object GUID for domain-joined computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        store = "Active Directory", 
        types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"), 
        query = ";objectguid;{0}", 
        param = c2.Value
    );
 
### <a name="issue-objectsid-of-the-computer-account-on-premises"></a>發出內部部署電腦帳戶的 objectSID

**`http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid`** - 此宣告必須包含內部部署電腦帳戶的 **objectSid** 值。 在 AD FS 中，您可以新增發行轉換規則，如下所示︰

    @RuleName = "Issue objectSID for domain-joined computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(claim = c2);

### <a name="issue-issuerid-for-computer-when-multiple-verified-domain-names-in-azure-ad"></a>當 Azure AD 中有多個經過驗證的網域名稱時，發出電腦的 issuerID

**`http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid`** - 此宣告必須包含與發行權杖的內部部署同盟服務 (AD FS 或協力廠商) 連線之任何已驗證網域名稱的統一資源識別項 (URI)。 在 AD FS 中，您可以在上述內容之後，以該特定順序新增如下所示的發行轉換規則。 請注意，需要有一個規則可為使用者明確發出此規則。 下列規則中會新增用來識別使用者與電腦驗證的優先規則。

    @RuleName = "Issue account type with the value User when its not a computer"
    NOT EXISTS(
    [
        Type == "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value == "DJ"
    ]
    )
    => add(
        Type = "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value = "User"
    );
    
    @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
    c1:[
        Type == "http://schemas.xmlsoap.org/claims/UPN"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value == "User"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", 
        Value = regexreplace(
        c1.Value, 
        ".+@(?<domain>.+)", 
        "http://${domain}/adfs/services/trust/"
        )
    );
    
    @RuleName = "Issue issuerID for domain-joined computers"
    c:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", 
        Value = "http://<verified-domain-name>/adfs/services/trust/"
    );


在上述宣告中，

- `<verified-domain-name>` 是要使用您 Azure AD 中其中一個已驗證網域名稱來取代的預留位置。 例如，值 = "http://contoso.com/adfs/services/trust/"



如需已驗證之網域名稱的詳細資料，請參閱[將自訂網域名稱新增至 Azure Active Directory](../active-directory-domains-add-azure-portal.md)。  

若要取得已驗證之公司網域的清單，您可以使用 [Get-MsolDomain](/powershell/module/msonline/get-msoldomain?view=azureadps-1.0) Cmdlet。 

![Get-MsolDomain](./media/hybrid-azuread-join-manual-steps/01.png)

### <a name="issue-immutableid-for-computer-when-one-for-users-exist-eg-alternate-login-id-is-set"></a>當使用者存在 ImmutableID (例如已設定替代登入識別碼) 時，請發出電腦的 ImmutableID

**`http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`** - 此宣告必須包含電腦的有效值。 在 AD FS 中，您可以新增發行轉換規則，如下所示︰

    @RuleName = "Issue ImmutableID for computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ] 
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        store = "Active Directory", 
        types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"), 
        query = ";objectguid;{0}", 
        param = c2.Value
    );

### <a name="helper-script-to-create-the-ad-fs-issuance-transform-rules"></a>建立 AD FS 發行轉換規則的協助程式指令碼

下列指令碼可協助您建立上面所述的發行轉換規則。

    $multipleVerifiedDomainNames = $false
    $immutableIDAlreadyIssuedforUsers = $false
    $oneOfVerifiedDomainNames = 'example.com'   # Replace example.com with one of your verified domains
    
    $rule1 = '@RuleName = "Issue account type for domain-joined computers"
    c:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value = "DJ"
    );'

    $rule2 = '@RuleName = "Issue object GUID for domain-joined computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        store = "Active Directory", 
        types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"), 
        query = ";objectguid;{0}", 
        param = c2.Value
    );'

    $rule3 = '@RuleName = "Issue objectSID for domain-joined computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(claim = c2);'

    $rule4 = ''
    if ($multipleVerifiedDomainNames -eq $true) {
    $rule4 = '@RuleName = "Issue account type with the value User when it is not a computer"
    NOT EXISTS(
    [
        Type == "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value == "DJ"
    ]
    )
    => add(
        Type = "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value = "User"
    );
    
    @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
    c1:[
        Type == "http://schemas.xmlsoap.org/claims/UPN"
    ]
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2012/01/accounttype", 
        Value == "User"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", 
        Value = regexreplace(
        c1.Value, 
        ".+@(?<domain>.+)", 
        "http://${domain}/adfs/services/trust/"
        )
    );
    
    @RuleName = "Issue issuerID for domain-joined computers"
    c:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", 
        Value = "http://' + $oneOfVerifiedDomainNames + '/adfs/services/trust/"
    );'
    }

    $rule5 = ''
    if ($immutableIDAlreadyIssuedforUsers -eq $true) {
    $rule5 = '@RuleName = "Issue ImmutableID for computers"
    c1:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", 
        Value =~ "-515$", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ] 
    && 
    c2:[
        Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", 
        Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
    ]
    => issue(
        store = "Active Directory", 
        types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"), 
        query = ";objectguid;{0}", 
        param = c2.Value
    );'
    }

    $existingRules = (Get-ADFSRelyingPartyTrust -Identifier urn:federation:MicrosoftOnline).IssuanceTransformRules 

    $updatedRules = $existingRules + $rule1 + $rule2 + $rule3 + $rule4 + $rule5

    $crSet = New-ADFSClaimRuleSet -ClaimRule $updatedRules 

    Set-AdfsRelyingPartyTrust -TargetIdentifier urn:federation:MicrosoftOnline -IssuanceTransformRules $crSet.ClaimRulesString 

### <a name="remarks"></a>備註 

- 此指令碼會將規則附加至現有的規則。 請勿執行此指令碼兩次，因為會新增這組規則兩次。 先確定這些宣告沒有對應的規則存在 (在對應的條件下)，才能再次執行指令碼。

- 如果您有多個經過驗證的網域名稱 (如 Azure AD 入口網站中所示，或透過 Get-MsolDomains cmdlet)，將指令碼中 **$multipleVerifiedDomainNames** 的值設定為 **$true**。 此外，也務必移除經由 Azure AD Connect 或透過其他方式所建立的任何現有 issuerid 宣告。 此規則的範例如下︰


        c:[Type == "http://schemas.xmlsoap.org/claims/UPN"]
        => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)",  "http://${domain}/adfs/services/trust/")); 

- 如果您已對使用者帳戶發出 **ImmutableID** 宣告，請將指令碼中 **$immutableIDAlreadyIssuedforUsers** 的值設定為 **$true**。

## <a name="enable-windows-down-level-devices"></a>啟用舊版 Windows 裝置

如果有些已加入網域的裝置是舊版 Windows 裝置，您需要︰

- 在 Azure AD 中設定原則，讓使用者可以註冊裝置。
 
- 設定內部部署同盟服務來發出宣告，以支援**整合式 Windows 驗證 (IWA)** 來進行裝置註冊。
 
- 將 Azure AD 裝置驗證端點新增至近端內部網路區域，以避免在驗證裝置時出現憑證提示。

### <a name="set-policy-in-azure-ad-to-enable-users-to-register-devices"></a>在 Azure AD 中設定原則，讓使用者可以註冊裝置

若要註冊舊版 Windows 裝置，您需要確定已設定可允許使用者在 Azure AD 中註冊裝置的設定。 在 Azure 入口網站中，您可以在底下找到此設定：

`Azure Active Directory > Users and groups > Device settings`
    
下列原則必須設定為 [全部]：**使用者可以向 Azure AD 註冊其裝置**

![註冊裝置](./media/hybrid-azuread-join-manual-steps/23.png)


### <a name="configure-on-premises-federation-service"></a>設定內部部署同盟服務 

在收到對 Azure AD 信賴憑證者 (持有 resouce_params 參數且編碼值如下所示) 的驗證要求時，內部部署同盟服務必須支援發出 **authenticationmehod** 和 **wiaormultiauthn** 宣告︰

    eyJQcm9wZXJ0aWVzIjpbeyJLZXkiOiJhY3IiLCJWYWx1ZSI6IndpYW9ybXVsdGlhdXRobiJ9XX0

    which decoded is {"Properties":[{"Key":"acr","Value":"wiaormultiauthn"}]}

收到這類要求時，內部部署同盟服務必須先使用整合式 Windows 驗證來驗證使用者，並且在成功時發出下列兩個宣告︰

    http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/windows
    http://schemas.microsoft.com/claims/wiaormultiauthn

在 AD FS 中，您必須新增可傳遞驗證方法的發行轉換規則。  

**若要新增此規則︰**

1. 在 AD FS 管理主控台中，移至 `AD FS > Trust Relationships > Relying Party Trusts`。
2. 在 [Microsoft Office 365 身分識別平台] 信賴憑證者信任物件上按一下滑鼠右鍵，然後選取 [編輯宣告規則]。
3. 在 [發佈轉換規則] 索引標籤上，選取 [新增規則]。
4. 在 [宣告規則] 範本清單中，選取 [使用自訂規則傳送宣告]。
5. 選取 [下一步] 。
6. 在 [宣告規則名稱] 方塊中，輸入**驗證方法宣告規則**。
7. 在 [宣告規則] 方塊中，輸入下列規則︰

    `c:[Type == "http://schemas.microsoft.com/claims/authnmethodsreferences"] => issue(claim = c);`

8. 在同盟伺服器上，以 Azure AD 信賴憑證者信任物件的信賴憑證者物件名稱取代 **\<RPObjectName\>** 之後，輸入下列 PowerShell 命令。 此物件通常名為「Microsoft Office 365 身分識別平台」。
   
    `Set-AdfsRelyingPartyTrust -TargetName <RPObjectName> -AllowedAuthenticationClassReferences wiaormultiauthn`

### <a name="add-the-azure-ad-device-authentication-end-point-to-the-local-intranet-zones"></a>將 Azure AD 裝置驗證端點新增至近端內部網路區域

若要避免在註冊裝置中的使用者向 Azure AD 進行驗證時出現憑證提示時，您可以將原則推送到已加入網域的裝置，進而將下列 URL 新增至 Internet Explorer 的近端內部網路區域︰

`https://device.login.microsoftonline.com`


## <a name="verify-joined-devices"></a>確認加入的裝置

您可以在 [Azure Active Directory PowerShell 模組](/powershell/azure/install-msonlinev1?view=azureadps-2.0) 中使用 [Get-MsolDevice](https://docs.microsoft.com/powershell/msonline/v1/get-msoldevice) Cmdlet，以查看您組織中已加入成功的裝置。

此 Cmdlet 的輸出會顯示已註冊和已加入 Azure AD 的裝置。 若要取得所有裝置，請使用 **-All** 參數，然後使用 **deviceTrustType** 屬性進行篩選。 已加入網域的裝置具有**已加入網域**這個值。



## <a name="troubleshoot-your-implementation"></a>對您的實作進行疑難排解

如果您為已加入網域的 Windows 裝置執行混合式 Azure AD Join 時遇到問題，請參閱：

- [對現行 Windows 裝置的混合式 Azure AD Join 進行疑難排解](troubleshoot-hybrid-join-windows-current.md)
- [對舊版 Windows 裝置的混合式 Azure AD Join 進行疑難排解](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>後續步驟

* [Azure Active Directory 中的裝置管理簡介](overview.md)



<!--Image references-->
[1]: ./media/hybrid-azuread-join-manual-steps/12.png
