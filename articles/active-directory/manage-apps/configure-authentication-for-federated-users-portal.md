---
title: 使用主領域探索原則為應用程式設定登入自動加速 | Microsoft Docs
description: 說明什麼是 Azure AD 租用戶，以及如何透過 Azure Active Directory 管理 Azure。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/08/2018
ms.author: barbkess
ms.openlocfilehash: f24be44b00f9c4e789e8d4797f6a0516dcfe940f
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494052"
---
# <a name="configure-azure-active-directory-sign-in-behavior-for-an-application-by-using-a-home-realm-discovery-policy"></a>使用主領域探索原則為應用程式設定 Azure Active Directory 登入行為

下列文件簡介如何為同盟使用者設定 Azure Active Directory 驗證行為。   其中涵蓋自動加速組態，以及同盟網域中的使用者驗證限制。

## <a name="home-realm-discovery"></a>主領域探索
主領域探索 (HRD) 是讓 Azure Active Directory (Azure AD) 判斷使用者登入時應在何處驗證的程序。  使用者在登入 Azure AD 租用戶以存取資源或 Azure AD 一般登入頁面時，會輸入使用者名稱 (UPN)。 Azure AD 會使用該值來探索使用者需在何處登入。 

使用者可能需要被導向至下列其中一處進行驗證：

- 使用者的主租用戶 (可能與使用者嘗試存取之資源的租用戶相同)。 

- Microsoft 帳戶。  使用者是資源租用戶中的來賓。

-  內部部署身分識別提供者，例如 Active Directory Federation Services (AD FS)。

- 與 Azure AD 租用戶同盟的另一個身分識別提供者。

## <a name="auto-acceleration"></a>自動加速 
某些組織會將其 Azure Active Directory 租用戶中的網域設為與另一個 IdP (例如用於使用者驗證的 AD FS) 同盟。  

使用者在登入應用程式時將會先看到 Azure AD 登入頁面。 如果使用者是在同盟網域中，則輸入 UPN 後，系統便會將其導向提供該網域的 IdP 登入頁面。 在某些情況下，系統管理員在使用者登入特定應用程式時可能會想要將其導向至登入頁面。 

因此，使用者即可略過初始的 Azure Active Directory 頁面。 此程序稱為「登入自動加速」。

當租用戶與另一個用於登入的 IdP 同盟時，自動加速可簡化使用者的登入程序。  您可以為個別的應用程式設定自動加速。

>[!NOTE]
>如果您為應用程式設定自動加速，來賓使用者將會無法登入。 如果您將使用者直接導向至同盟 IdP 進行驗證，他們就無法回到 Azure Active Directory 登入頁面。 來賓使用者可能會需要被導向至其他租用戶或外部 IdP (如 Microsoft 帳戶)，但由於他們將略過主領域探索步驟，因此無法登入該應用程式。  

有兩種方式可控制同盟 IdP 的自動加速：   

- 在應用程式的驗證要求上使用網域提示。 
- 設定主領域探索原則以啟用自動加速。

### <a name="domain-hints"></a>網域提示    
網域提示是應用程式的驗證要求中包含的指示詞。 它們可用來加快使用者進入其同盟 IdP 登入頁面的速度。 或者，多租用戶應用程式也可使用網域提示來加速使用者直接前往其租用戶的品牌化 Azure AD 登入頁面。  

例如，應用程式 "largeapp.com" 可能讓客戶存取位於自訂 URL "contoso.largeapp.com" 的應用程式。 應用程式也可在驗證要求中包含 contoso.com 網域提示。 

網域提示語法會根據所使用的通訊協定而有所不同，且通常會設定於應用程式中。

**WS-同盟**：查詢字串中的 whr=contoso.com。

**SAML**：包含網域提示的 SAML 驗證要求，或是查詢字串 whr=contoso.com。

**Open ID Connect**：查詢字串 domain_hint=contoso.com。 

如果網域提示包含在應用程式的驗證要求中，而租用戶與該網域同盟，則 Azure AD 會嘗試將登入重新導向至為該網域設定的 IdP。 

如果網域提示未參考已驗證的同盟網域，則系統會加以忽略，並叫用標準的主領域探索。

如需關於使用 Azure Active Directory 所支援的網域提示進行自動加速的詳細資訊，請參閱[企業行動力 + 安全性部落格](https://cloudblogs.microsoft.com/enterprisemobility/2015/02/11/using-azure-ad-to-land-users-on-their-custom-login-page-from-within-your-app/) (英文)。

>[!NOTE]
>如果網域提示包含在驗證要求中，則其存在將覆寫 HRD 原則中為該應用程式設定的自動加速。

### <a name="home-realm-discovery-policy-for-auto-acceleration"></a>用於自動加速的主領域探索原則
某些應用程式未沒有相關機制可設定它們所發出的驗證要求。 在這種情況下，就無法使用網域提示來控制自動加速。 您可以透過原則設定自動加速來達成相同的行為。  

## <a name="enable-direct-authentication-for-legacy-applications"></a>為繼承應用程式啟用直接驗證
應用程式驗證使用者的最佳做法是使用 Azure Active Directory 程式庫和互動式登入。 程式庫會負責同盟使用者流程。  有時寫入繼承應用程式並非為了要了解同盟。 應用程式不會執行主領域探索，也不會與正確的同盟端點互動以驗證使用者。 您也可選擇使用 HRD 原則啟用會提交使用者名稱/密碼認證的特定繼承應用程式，以直接透過 Azure Active Directory 進行驗證。 您必須啟用密碼雜湊同步。 

> [!IMPORTANT]
> 只要在您已啟用密碼雜湊同步，且您知道即使有內部部署 IdP 實作的任何原則，也可驗證該應用程式時，才能啟用直接驗證。 如果您基於任何原因而關閉密碼雜湊同步，或關閉 AD Connect 的目錄同步作業，則您應移除此原則，以避免使用者透過過時的密碼雜湊進行直接驗證。

## <a name="set-hrd-policy"></a>設定 HRD 原則
若要在應用程式上設定 HRD 原則以執行同盟登入自動加速或直接以雲端為基礎的應用程式，共有三個步驟如下：

1. 建立 HRD 原則。

2. 尋找服務主體以附加原則。

3. 將原則附加至服務主體。 

只有在特定應用程式附加至服務主體時，原則才會生效。 

任何時候服務主體都只能有一個作用中的 HRD 原則。  

您可直接使用 Microsoft Azure Active Directory 圖形 API 或 Azure Active Directory PowerShell cmdlets 建立及管理 HRD 原則。

管理原則的 Graph API 說明於 MSDN 上的[原則相關作業](https://msdn.microsoft.com/library/azure/ad/graph/api/policy-operations)一文。

以下是範例 HRD 原則定義：
    
 ```
   {  
    "HomeRealmDiscoveryPolicy":
    {  
    "AccelerateToFederatedDomain":true,
    "PreferredDomain":"federated.example.edu",
    "AllowCloudPasswordValidation":true
    }
   }
```

原則類型是 "HomeRealmDiscoveryPolicy"。

**AccelerateToFederatedDomain** 是選用項目。 如果 **AccelerateToFederatedDomain** 為 false，則原則對於自動加速沒有作用。 如果 **AccelerateToFederatedDomain** 為 true，且租用戶中只有一個已驗證的同盟網域，則系統會將使用者直接導向同盟 IdP 登入。 如果值為 true，且租用戶中有多個已驗證的網域，則必須指定 **PreferredDomain**。

**PreferredDomain** 是選用項目。 **PreferredDomain** 應指出要加速的網域。 如果租用戶只有一個同盟網域，則可加以省略。  如果省略，而且有多個已驗證的同盟網域，則原則沒有任何作用。

 如果已指定 **PreferredDomain**，它必須符合租用戶的已驗證同盟網域。 應用程式的所有使用者都必須能夠登入該網域。

**AllowCloudPasswordValidation** 是選用項目。 如果 **AllowCloudPasswordValidation** 為 true，則應用程式可直接向 Azure Active Directory 權杖端點提供使用者名稱/密碼認證，以驗證同盟使用者。 只有在密碼雜湊同步已啟用時，此功能才能作用。

### <a name="priority-and-evaluation-of-hrd-policies"></a>HRD 原則的優先順序和評估
您可以建立 HRD 原則，並將其指派給特定的組織及服務主體。 這意謂著可以將多個原則套用至某個特定的應用程式。 生效的 HRD 原則會遵循下列規則：


- 如果驗證要求中存有網域提示，則系統會忽略任何自動加速 HRD 原則。 此時會使用網域提示所指定的行為。

- 否則，如果已將原則明確指派給服務主體，就會強制執行該原則。 

- 如果沒有網域提示，且未將任何原則明確指派給服務主體，則會強制執行明確指派給該服務主體之父組織的原則。 

- 如果沒有網域提示，且未指派原則給服務主體或組織，則會使用預設 HRD 行為。

## <a name="tutorial-for-setting-hrd-policy-on-an-application"></a>在應用程式上設定 HRD 原則的教學課程 
我們將使用 Azure AD PowerShell Cmdlet 逐步解說一些案例，包括：


- 設定 HRD 原則讓應用程式在具有單一同盟網域的租用戶中執行自動加速。

- 設定 HRD 原則讓應用程式在租用戶已驗證的多個網域之一執行自動加速。

- 設定 HRD 原則，讓繼承應用程式向 Azure Active Directory 針對同盟使用者執行直接的使用者名稱/密碼驗證。

- 列出已設定原則的應用程式。


### <a name="prerequisites"></a>必要條件
在下列範例中，您將建立、更新、連結和刪除 Azure AD 中應用程式服務主體上的原則。

1.  若要開始，請下載最新的 Azure AD PowerShell Cmdlet 預覽版。 

2.  下載 Azure AD PowerShell Cmdlet 之後，請執行 Connect 命令以使用您的系統管理員帳戶登入 Azure AD。

    ``` powershell
    Connect-AzureAD -Confirm
    ```
3.  執行下列命令以查看您組織中的所有原則：

    ``` powershell
    Get-AzureADPolicy
    ```

如果沒有傳回任何內容，表示您的租用戶中未建立任何原則。

### <a name="example-set-hrd-policy-for-an-application"></a>範例：為應用程式設定 HRD 原則 

在此範例中，您建立的原則若指派給應用程式可執行以下功能： 
- 如果您的租用戶中有單一網域，則使用者在登入應用程式時能自動加速前往 AD FS 登入畫面。 
- 如果您的租用戶中有多個同盟網域，則可讓使用者自動加速前往 AD FS 登入畫面。
- 已具有指派原則的應用程式可讓同盟使用者直接透過非互動式使用者名稱/密碼登入至 Azure Active Directory。

#### <a name="step-1-create-an-hrd-policy"></a>步驟 1：建立 HRD 原則

如果您的租用戶中有單一網域，則下列原則可讓使用者在登入應用程式時，自動加速前往 AD FS 登入畫面。

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true}}") -DisplayName BasicAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```
如果您的租用戶中有多個同盟網域，則下列原則可讓使用者自動加速前往 AD FS 登入畫面。 如果您有多個同盟網域都會進行應用程式使用者驗證，則您必須指定要自動加速的網域。

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true, `"PreferredDomain`":`"federated.example.edu`"}}") -DisplayName MultiDomainAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```

若要為特定應用程式建立原則，讓同盟使用者直接向 Azure Active Directory 進行使用者名稱/密碼驗證，請執行下列命令：

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AllowCloudPasswordValidation`":true}}") -DisplayName EnableDirectAuthPolicy -Type HomeRealmDiscoveryPolicy
```


若要查看您的新原則並取得其 **ObjectID**，請執行下列命令：

``` powershell
Get-AzureADPolicy
```


建立 HRD 原則後若要套用，可將其指派給多個應用程式服務主體。

#### <a name="step-2-locate-the-service-principal-to-which-to-assign-the-policy"></a>步驟 2：尋找服務主體以對其指派原則  
您需要原則要指派到之服務主體的 **ObjectID**。 有幾種方式可尋找服務主體的 **ObjectID**。    

您可以使用入口網站，或者您可以查詢 [Microsoft Graph](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity)。 您可以移至 [Graph Explorer 工具](https://developer.microsoft.com/graph/graph-explorer)並登入您的 Azure AD 帳戶，以查看您所有組織的服務主體。 由於您使用 PowerShell，您可以使用 get-AzureADServicePrincipal Cmdlet 列出服務主體及其識別碼。

#### <a name="step-3-assign-the-policy-to-your-service-principal"></a>步驟 3：將原則指派給服務主體  
在您取得要設定自動加速之應用程式的服務主體 **ObjectID** 後，請執行下列命令。 此命令會將您在步驟 1 中建立的 HRD 原則與步驟 2 中找到的服務主體產生關聯。

``` powershell
Add-AzureADServicePrincipalPolicy -Id <ObjectID of the Service Principal> -RefObjectId <ObjectId of the Policy>
```

您可以為要新增原則的每個服務主體重複此命令。

如果指派給應用程式的是 HomeRealmDiscovery 原則，則您就無法新增第二個原則。  如為此情況，請變更指派給應用程式的主領域探索原則定義，以新增其他參數。

#### <a name="step-4-check-which-application-service-principals-your-hrd-policy-is-assigned-to"></a>步驟 4：檢查您的 HRD 原則指派給哪些應用程式服務主體
若要檢查哪些應用程式已設定 HRD 原則，請使用 **Get-AzureADPolicyAppliedObject** cmdlet。 將您想檢查之原則的 **ObjectID** 傳遞給它。

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```
#### <a name="step-5-youre-done"></a>步驟 5：大功告成！
要檢查新原則是否正常運作，請嘗試應用程式。

### <a name="example-list-the-applications-for-which-hrd-policy-is-configured"></a>範例：列出已設定 HRD 原則的應用程式

#### <a name="step-1-list-all-policies-that-were-created-in-your-organization"></a>步驟 1：列出您組織中建立的所有原則 

``` powershell
Get-AzureADPolicy
```

請記下要列出指派之原則的 **ObjectID**。

#### <a name="step-2-list-the-service-principals-to-which-the-policy-is-assigned"></a>步驟 2：列出已被指派原則的服務主體。  

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```

### <a name="example-remove-an-hrd-policy-for-an-application"></a>範例：移除應用程式的 HRD 原則
#### <a name="step-1-get-the-objectid"></a>步驟 1：取得 ObjectID
使用前一個範例來取得原則的 ObjectId，以及要移除該原則之應用程式服務主體的 **ObjectID**。 

#### <a name="step-2-remove-the-policy-assignment-from-the-application-service-principal"></a>步驟 2：從應用程式服務主體移除原則指派  

``` powershell
Remove-AzureADApplicationPolicy -ObjectId <ObjectId of the Service Principal>  -PolicyId <ObjectId of the policy>
```

#### <a name="step-3-check-removal-by-listing-the-service-principals-to-which-the-policy-is-assigned"></a>步驟 3：列出已被指派原則的服務主體，檢查移除是否成功 

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```
## <a name="next-steps"></a>後續步驟
- 如需如何在 Azure AD 中進行驗證的詳細資訊，請參閱 [Azure AD 的驗證案例](../develop/authentication-scenarios.md)。
- 如需使用者單一登入的詳細資訊，請參閱[搭配 Azure Active Directory 的應用程式存取和單一登入](configure-single-sign-on-portal.md)。
- 如需所有開發人員相關內容的概觀，請瀏覽 [Active Directory 開發人員指南](../develop/azure-ad-developers-guide.md)。
