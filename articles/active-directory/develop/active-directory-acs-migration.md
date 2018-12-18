---
title: 從 Azure 存取控制服務移轉 | Microsoft Docs
description: 深入了解搬移 Azure 存取控制服務 (ACS) 中應用程式和服務的選項。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jlu, annaba, hirsin
ms.openlocfilehash: 59856418adde1ea29a0513a1ca7c0c60531768d8
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036536"
---
# <a name="how-to-migrate-from-the-azure-access-control-service"></a>操作說明：從 Azure 存取控制服務遷移

Microsoft Azure 存取控制服務 (ACS) 是 Azure Active Directory (Azure AD) 的一項服務，將於 2018 年 11 月 7 日淘汰。 目前使用存取控制的應用程式和服務，必須在淘汰前完全移轉到其他驗證機制。 本文說明給目前客戶的建議移轉選項，協助您規劃汰換存取控制的事宜。 如果您目前未使用存取控制，不需要採取任何動作。

## <a name="overview"></a>概觀

存取控制是雲端驗證服務，能以簡單的方式驗證和授權使用者，使其得以存取您的 Web 應用程式和服務。 它讓您不需要考慮您的程式碼也能享有許多驗證和授權功能。 存取控制的主要使用者包括 Microsoft .NET 用戶端、ASP.NET Web 應用程式及 Windows Communication Foundation (WCF) Web 服務的開發人員和架構設計師。

存取控制的使用案例可以分成三個主要類別：

- 驗證特定的 Microsoft 雲端服務，包括 Azure 服務匯流排和 Dynamics CRM。 用戶端應用程式可以從存取控制取得權杖，用來驗證這些服務以執行各種動作。
- 新增驗證方式至自訂和預先封裝 (如 SharePoint) 的 Web 應用程式。 透過存取控制的「被動」驗證方式，Web 應用程式支援以 Microsoft 帳戶 (前身為 Live ID)，以及 Google、Facebook、Yahoo、Azure AD 及 Active Directory 同盟服務 (AD FS) 等帳戶進行單一登入。
- 使用存取控制簽發的權杖保護自訂 Web 服務。 透過「主動」驗證方式，Web 服務可確保惟有通過存取控制之驗證的已知用戶端能存取自身服務。

每個使用案例及其建議的移轉策略會在後續各節中討論。

> [!WARNING]
> 在大部分情況下，將現有應用程式和服務移轉至較新的技術需要大幅變更程式碼。 建議您立即開始規劃及執行移轉作業，以避免任何可能的中斷或停機。

存取控制包含下列元件：

- 安全權杖服務 (STS)，可接收驗證要求然後簽發安全性權杖。
- Azure 傳統入口網站，用以建立、刪除、啟用及停用存取控制的命名空間。
- 個別的存取控制管理入口網站，用以自訂及設定存取控制命名空間。
- 管理服務，用以自動化入口網站功能。
- 權杖轉換規則引擎，用以定義複雜邏輯以控制存取控制簽發之權杖的輸出格式。

若要使用這些元件，您必須建立一或多個存取控制命名空間。 「命名空間」是應用程式和服務進行通訊的專用存取控制執行個體。 命名空間會與其他所有存取控制客戶隔離。 其他存取控制客戶會使用他們自己的命名空間。 存取控制中的命名空間有專用 URL，看起來像這樣：

```HTTP
https://<mynamespace>.accesscontrol.windows.net
```

與 STS 和管理作業的所有通訊都透過此 URL 完成。 不同的路徑有不同的用途。 若要判斷應用程式和服務是否使用存取控制，請監視所有進入 https://&lt;namespace&gt;.accesscontrol.windows.net 的流量。 所有進入該 URL 的流量都由存取控制處理，而且必須停止。 

但所有進入 `https://accounts.accesscontrol.windows.net` 的流量除外。 進入此 URL 的流量已經由其他服務處理，因此**不**受「存取控制」的淘汰影響。 

如需有關存取控制的詳細資訊，請參閱[存取控制服務 2.0 (已封存)](https://msdn.microsoft.com/library/hh147631.aspx)。

## <a name="find-out-which-of-your-apps-will-be-impacted"></a>找出哪些應用程式會受影響

遵循這一節中的步驟，找出哪些應用程式會受到 ACS 淘汰影響。

### <a name="download-and-install-acs-powershell"></a>下載並安裝 ACS PowerShell

1. 移至 PowerShell 資源庫並下載 [Acs.Namespaces](https://www.powershellgallery.com/packages/Acs.Namespaces/1.0.2)。
1. 藉由執行來安裝模組

    ```powershell
    Install-Module -Name Acs.Namespaces
    ```

1. 藉由執行來取得所有可能的命令

    ```powershell
    Get-Command -Module Acs.Namespaces
    ```

    若要取得特定命令的說明，請執行：

    ```
     Get-Help [Command-Name] -Full
    ```
    
    其中 `[Command-Name]` 是 ACS 命令的名稱。

### <a name="list-your-acs-namespaces"></a>列出您的 ACS 命名空間

1. 使用 **Connect-AcsAccount** Cmdlet 連線至 ACS。
  
    您可能需要先執行 `Set-ExecutionPolicy -ExecutionPolicy Bypass`，才可以執行命令，而且成為這些訂用帳戶的管理員才能執行命令。

1. 使用 **Get-AcsSubscription** Cmdlet 列出可用的 Azure 訂用帳戶。
1. 使用 **Get-AcsNamespace** Cmdlet 列出您的 ACS 命名空間。

### <a name="check-which-applications-will-be-impacted"></a>檢查哪些應用程式會受影響

1. 使用上一個步驟中的命名空間並移至 `https://<namespace>.accesscontrol.windows.net`

    例如，如果其中一個命名空間是 contoso-test，請移至 `https://contoso-test.accesscontrol.windows.net`

1. 在 [信任關係] 之下，選取 [信賴憑證者應用程式] 以查看會受 ACS 淘汰影響的應用程式清單。
1. 針對您擁有的任何其他 ACS 命名空間重複步驟 1-2。

## <a name="retirement-schedule"></a>淘汰排程

截至 2017 年 11 月止，所有存取控制元件均完全受到支援且可運作。 唯一的限制是您[不能透過 Azure 傳統入口網站建立新的存取控制命名空間](https://azure.microsoft.com/blog/acs-access-control-service-namespace-creation-restriction/)。

以下是存取控制元件的淘汰排程：

- **2017 年 11 月**：Azure 傳統入口網站中的 Azure AD 管理員體驗[已淘汰](https://blogs.technet.microsoft.com/enterprisemobility/2017/09/18/marching-into-the-future-of-the-azure-ad-admin-experience-retiring-the-azure-classic-portal/)。 目前，可在以下新的專用 URL 進行「存取控制」的命名空間管理：`http://manage.windowsazure.com?restoreClassic=true`。 該 URL 可以用來檢視現有的命名空間、啟用和停用命名空間，若您有需要，也可以刪除命名空間。
- **2018 年 4 月 2 日**：Azure 傳統入口網站已完全淘汰，這意謂著透過任何 URL 都無法進行「存取控制」命名空間管理。 屆時，您將無法停用或啟用、刪除或列舉存取控制命名空間。 不過，「存取控制」管理入口網站將會在 `https://\<namespace\>.accesscontrol.windows.net` 運作並提供完整功能。 存取控制的其他所有元件都將繼續正常運作。
- **2018 年 11 月 7 日**：所有「存取控制」元件將永久關閉。 這包括存取控制管理入口網站、管理服務、STS 和權杖轉換規則引擎。 屆時，所有傳送給存取控制 (網指 \<namespace\>.accesscontrol.windows.net) 的要求都會失敗。 在那之前，請確實將所有現有的應用程式和服務移轉至其他技術。

## <a name="migration-strategies"></a>移轉策略

下列各節說明從存取控制移轉到其他 Microsoft 技術的初略建議。

### <a name="clients-of-microsoft-cloud-services"></a>Microsoft 雲端服務的用戶端

接受存取控制簽發之權杖的每項 Microsoft 雲端服務，現在支援至少一個其他驗證方式。 每項服務適合的驗證機制不盡相同。 建議您參閱每項服務的特定文件，了解官方指導。 為了方便起見，以下提供每一組文件：

| 服務 | 指引 |
| ------- | -------- |
| Azure 服務匯流排 | [移轉至共用存取簽章](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-migrate-acs-sas) |
| Azure 服務匯流排轉送 | [移轉至共用存取簽章](https://docs.microsoft.com/azure/service-bus-relay/relay-migrate-acs-sas) |
| Azure 受控快取 | [移轉至 Azure Redis 快取](https://docs.microsoft.com/azure/redis-cache/cache-faq#which-azure-cache-offering-is-right-for-me) |
| Azure DataMarket | [移轉至認知服務 API](https://docs.microsoft.com/azure/machine-learning/studio/datamarket-deprecation) |
| BizTalk 服務 | [移轉至 Azure App Service 的邏輯應用程式功能](https://docs.microsoft.com/azure/machine-learning/studio/datamarket-deprecation) |
| Azure 媒體服務 | [移轉至 Azure AD 驗證](https://azure.microsoft.com/blog/azure-media-service-aad-auth-and-acs-deprecation/) |
| Azure 備份 | [升級 Azure 備份代理程式](https://docs.microsoft.com/azure/backup/backup-azure-file-folder-backup-faq) |

<!-- Dynamics CRM: Migrate to new SDK, Dynamics team handling privately -->
<!-- Azure RemoteApp deprecated in favor of Citrix: http://www.zdnet.com/article/microsoft-to-drop-azure-remoteapp-in-favor-of-citrix-remoting-technologies/ -->
<!-- Exchange push notifications are moving, customers don't need to move -->
<!-- Retail federation services are moving, customers don't need to move -->
<!-- Azure StorSimple: TODO -->
<!-- Azure SiteRecovery: TODO -->

### <a name="sharepoint-customers"></a>SharePoint 客戶

SharePoint 2013、2016 和 SharePoint Online 客戶已長期使用 ACS 在雲端、內部部署及混合式案例中進行驗證。 有些 SharePoint 功能和使用案例將會受到 ACS 淘汰影響，有些則不會。 下表針對一些利用 ACS 的最常用 SharePoint 功能，摘要說明移轉指導方針：

| 功能 | 指引 |
| ------- | -------- |
| 從 Azure AD 驗證使用者 | 先前，Azure AD 並不支援 SharePoint 進行驗證所需的 SAML 1.1 權杖，因此 ACS 被用來作為讓 SharePoint 與 Azure AD 權杖格式相容的媒介。 現在，您可以[在內部部署應用程式上使用 Azure AD 應用程式庫 SharePoint 將 SharePoint 直接連線到 Azure AD](https://docs.microsoft.com/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial)。 |
| [內部部署 SharePoint 中的應用程式驗證和伺服器對伺服器驗證](https://technet.microsoft.com/library/jj219571(v=office.16).aspx) | 不受 ACS 淘汰影響；無須進行任何變更。 | 
| [SharePoint 增益集 (提供者裝載和 SharePoint 裝載) 的低信任度授權](https://docs.microsoft.com/sharepoint/dev/sp-add-ins/three-authorization-systems-for-sharepoint-add-ins) | 不受 ACS 淘汰影響；無須進行任何變更。 |
| [SharePoint 雲端混合式搜尋](https://blogs.msdn.microsoft.com/spses/2015/09/15/cloud-hybrid-search-service-application/) | 不受 ACS 淘汰影響；無須進行任何變更。 |

### <a name="web-applications-that-use-passive-authentication"></a>使用被動驗證的 Web 應用程式

存取控制針對使用存取控制進行使用者驗證的 Web 應用程式，提供下列功能給 Web 應用程式開發人員和架構設計師使用：

- 與 Windows Identity Foundation (WIF) 深入整合。
- 與 Google、Facebook、Yahoo、Azure Active Directory、AD FS 帳戶及 Microsoft 帳戶建立同盟。
- 支援下列驗證通訊協定：OAuth 2.0 Draft 13、WS-Trust 及 Web 服務同盟 (WS-同盟)。
- 支援下列權杖格式：JSON Web 權杖 (JWT)、SAML 1.1、SAML 2.0 和簡單 Web 權杖 (SWT)。
- 整合至 WIF 的首頁領域探索體驗，可以讓使用者選擇用來登入的帳戶類型。 這項體驗由 Web 應用程式裝載並且可完全自訂。
- 權杖轉換可以讓您對 Web 應用程式從存取控制收到的宣告進行多種自訂設定，包括：
    - 從識別提供者傳遞宣告。
    - 新增額外的自訂宣告。
    - 在某些情況下，可以使用簡單的 if-then 邏輯簽發宣告。

很抱歉，目前尚未有提供以上所有同等功能的服務。 請評估您需要的存取控制功能有哪些，然後再選擇要使用 [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/)、[Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) (Azure AD B2C) 或其他雲端驗證服務。

#### <a name="migrate-to-azure-active-directory"></a>移轉至 Azure Active Directory

將應用程式和服務直接與 Azure AD 整合是值得考慮的方法。 Azure AD 是 Microsoft 工作或學校帳戶的雲端式識別提供者。 Azure AD 是 Office 365、Azure 和其他產品的識別提供者。 它提供與存取控制類似的同盟驗證功能，但是僅支援部分存取控制功能。 

主要範例是與社交識別提供者的同盟，例如 Facebook、Google 和 Yahoo。 如果您的使用者使用這些類型的認證登入，Azure AD 則非合適的解決方案。 

Azure AD 也不一定支援與存取控制完全相同的驗證通訊協定。 舉例來說，雖然存取控制和 Azure AD 都支援 OAuth，不過每種實作之間都有些微差異。 在移轉時，不同的實作會需要您修改程式碼。

但是，Azure AD 確實為存取控制客戶提供幾個潛在優勢。 它能以原生方式支援雲端裝載的 Microsoft 工作或學校帳戶，而這些帳戶的使用者通常是存取控制客戶。 

Azure AD 租用戶也可以透過 AD FS，與內部部署 Active Directory 的一或多個執行個體同盟。 這樣一來，您的應用程式便能驗證雲端式使用者和內部部署裝載的使用者。 它也支援 WS-同盟通訊協定，讓使用 WIF 的 Web 應用程式整合作業變得相對簡單。

下表比較存取控制的 Web 應用程式相關功能，以及 Azure AD 的同等功能。 

以整體而言，如果您只讓使用者用 Microsoft 工作或學校帳戶登入，Azure Active Directory 可能是最合適的移轉選擇。

| 功能 | 存取控制支援 | Azure AD 支援 |
| ---------- | ----------- | ---------------- |
| **帳戶類型** | | |
| Microsoft 工作或學校帳戶 | 支援 | 支援 |
| Windows Server Active Directory 和 AD FS 的帳戶 |- 與 Azure AD 租用戶建立同盟則可支援 <br />- 與 AD FS 建立直接同盟則可支援 | 僅透過與 Azure AD 租用戶的同盟支援 | 
| 其他企業身分識別管理系統的帳戶 |- 與 Azure AD 租用戶建立同盟也許可支援 <br />- 建立直接同盟則可支援 | 可能透過與 Azure AD 租用戶的同盟 |
| 個人用 Microsoft 帳戶 | 支援 | 只能透過 Azure AD v2.0 OAuth 通訊協定提供支援，其他通訊協定則不行 | 
| Facebook、Google、Yahoo 帳戶 | 支援 | 無法經由任何方式提供支援 |
| **通訊協定與 SDK 相容性** | | |
| WIF | 支援 | 支援，但是提供的指示有限 |
| WS-同盟 | 支援 | 支援 |
| OAuth 2.0 | 支援 Draft 13 | 支援 RFC 6749 (最新型的規格) |
| WS-Trust | 支援 | 不支援 |
| **權杖格式** | | |
| JWT | 在搶鮮版 (Beta) 中支援 | 支援 |
| SAML 1.1 | 支援 | 預覽 |
| SAML 2.0 | 支援 | 支援 |
| SWT | 支援 | 不支援 |
| **自訂** | | |
| 可自訂的首頁領域探索/帳戶挑選 UI | 可下載的程式碼，可以納入應用程式 | 不支援 |
| 上傳自訂權杖簽署憑證 | 支援 | 支援 |
| 在權杖中自訂宣告 |- 從識別提供者傳遞輸入宣告<br />- 從識別提供者取得宣告形式的存取權杖<br />- 根據輸入宣告值簽發輸出宣告<br />- 簽發具有常數值的輸出宣告 |- 無法從同盟識別提供者傳遞宣告<br />- 無法從識別提供者取得宣告形式的存取權杖<br />- 無法根據輸入宣告值簽發輸出宣告<br />- 可以簽發具有常數值的輸出宣告<br />- 可以根據同步至 Azure AD 的使用者屬性簽發輸出宣告 |
| **自動化** | | |
| 自動化設定和管理工作 | 使用存取控制管理服務則可支援 | 使用 Microsoft Graph 和 Azure AD 圖形 API 則可支援 |

如果您決定 Azure AD 是最適合您應用程式和服務的移轉方式，請瞭解以下兩種讓應用程式與 Azure AD 整合。

若要使用 WS-同盟或 WIF 來與 Azure AD 整合，我們建議您依照[為不在資源庫內的應用程式設定同盟單一登入](https://docs.microsoft.com/azure/active-directory/application-config-sso-how-to-configure-federated-sso-non-gallery)所述的方法執行。 本文說明設定 Azure AD 來使用 SAML 單一登入的方法，但是同樣適用於 WS-同盟。 如欲採用此方法需要 Azure AD Premium 授權。 此方法有兩個優點：

- 您有 Azure AD 權杖自訂的完整彈性。 您可以自訂 Azure AD 簽發的宣告，使其符合存取控制簽發的宣告， 尤以使用者識別碼或名稱識別碼宣告最為重要。 若要在變更技術後繼續接收您使用者的一致使用者識別碼，請確認 Azure AD 簽發的使用者識別碼與存取控制簽發的識別碼相符。
- 您可以為您的應用程式設定專用的權杖簽署憑證與存留期。

> [!NOTE]
> 此方法需要 Azure AD Premium 授權。 如果您是存取控制客戶，而且需要 Premium 授權來為應用程式設定單一登入，請與我們連絡。 我們很樂意提供開發人員授權予您使用。

另一個方法是參考[此程式碼範例](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation)，當中的 WS-同盟設定指示稍有不同。 此程式碼範例不使用 WIF，而是使用 ASP.NET 4.5 OWIN 中介軟體。 不過，應用程式註冊指示適用於使用 WIF 的應用程式，而且不需要 Azure AD Premium 授權。 

如果您選擇此方法，請參閱並理解 [Azure AD 中的簽署金鑰變換](https://docs.microsoft.com/azure/active-directory/develop/active-directory-signing-key-rollover)一文。 這個方法會使用 Azure AD 全域簽署金鑰來簽發權杖。 根據預設，WIF 不會自動重新整理簽署金鑰。 當 Azure AD 轉換其全域簽署金鑰時，您的 WIF 實作必須準備好接受變更。 如需詳細資訊，請參閱 [Azure AD 中簽署金鑰變換的相關重要資訊](https://msdn.microsoft.com/en-us/library/azure/dn641920.aspx)。

如果您可以透過 OpenID Connect 或 OAuth 通訊協定與 Azure AD 整合，建議採用該方法。 我們的 [Azure AD 開發人員指南](https://aka.ms/aaddev)有許多關於如何將 Azure AD 整合至 Web 應用程式的文件和指導。

#### <a name="migrate-to-azure-active-directory-b2c"></a>移轉至 Azure Active Directory B2C

要考量的其他移轉路徑是 Azure AD B2C。 Azure AD B2C 是雲端驗證服務，就像存取控制一樣，可讓開發人員將其驗證和身分識別管理邏輯外包給雲端服務。 它是付費服務 (有免費和進階層)，專為作用中使用者多達數百萬的取用者互動應用程式而設計。

就像存取控制一樣，Azure AD B2C 其中一個最具吸引力的功能是支援許多不同類型的帳戶。 有了 Azure AD B2C，您可以讓使用者用 Microsoft 帳戶或 Facebook、Google、LinkedIn、GitHub 或 Yahoo 等帳戶登入。 Azure AD B2C 還支援「本機帳戶」，或使用者特別為您應用程式建立的使用者名稱與密碼。 Azure AD B2C 還提供豐富的擴充性供您自訂登入流程。 

不過，Azure AD B2C 無法支援存取控制客戶可能需要的各種驗證通訊協定和權杖格式。 您也不能使用 Azure AD B2C 從識別提供者、Microsoft 等取得權杖，以及查詢使用者的其他相關資訊。

下表比較存取控制的 Web 應用程式相關功能，以及 Azure AD B2C 的同等功能。 在高階中，*如果您的應用程式是針對取用者，或者如果它支援不同類型的帳戶，則 Azure AD B2C 對於您的移轉可能是正確的選擇。*

| 功能 | 存取控制支援 | Azure AD B2C 支援 |
| ---------- | ----------- | ---------------- |
| **帳戶類型** | | |
| Microsoft 工作或學校帳戶 | 支援 | 透過自訂原則支援  |
| Windows Server Active Directory 和 AD FS 的帳戶 | 與 AD FS 建立直接同盟則可支援 | 以自訂原則建立 SAML 同盟則可支援 |
| 其他企業身分識別管理系統的帳戶 | 透過 WS-同盟建立直接同盟則可支援 | 以自訂原則建立 SAML 同盟則可支援 |
| 個人用 Microsoft 帳戶 | 支援 | 支援 | 
| Facebook、Google、Yahoo 帳戶 | 支援 | 原生支援 Facebook 和 Google，Yahoo 則是透過自訂原則建立 OpenID Connect 同盟則可支援 |
| **通訊協定與 SDK 相容性** | | |
| Windows Identity Foundation (WIF) | 支援 | 不支援 |
| WS-同盟 | 支援 | 不支援 |
| OAuth 2.0 | 支援 Draft 13 | 支援 RFC 6749 (最新型的規格) |
| WS-Trust | 支援 | 不支援 |
| **權杖格式** | | |
| JWT | 在搶鮮版 (Beta) 中支援 | 支援 |
| SAML 1.1 | 支援 | 不支援 |
| SAML 2.0 | 支援 | 不支援 |
| SWT | 支援 | 不支援 |
| **自訂** | | |
| 可自訂的首頁領域探索/帳戶挑選 UI | 可下載的程式碼，可以納入應用程式 | 透過自訂 CSS 可完全自訂 UI |
| 上傳自訂權杖簽署憑證 | 支援 | 支援透過自訂原則自訂簽署金鑰 (非憑證) |
| 在權杖中自訂宣告 |- 從識別提供者傳遞輸入宣告<br />- 從識別提供者取得宣告形式的存取權杖<br />- 根據輸入宣告值簽發輸出宣告<br />- 簽發具有常數值的輸出宣告 |- 可以從識別提供者傳遞宣告；某些宣告需有自訂原則<br />- 無法從識別提供者取得宣告形式的存取權杖<br />- 可以透過自訂原則根據輸入宣告值簽發輸出宣告<br />- 可以透過自訂原則簽發具有常數值的輸出宣告 |
| **自動化** | | |
| 自動化設定和管理工作 | 使用存取控制管理服務則可支援 |- 允許透過 Azure AD 圖形 API 建立使用者<br />- 無法以程式設計方式建立 B2C 租用戶、應用程式或原則 |

如果您決定 Azure AD B2C 是適合您應用程式和服務的移轉方式，請先了解下列資源的內容：

- [Azure AD B2C 文件](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview)
- [Azure AD B2C 自訂原則](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview-custom)
- [Azure AD B2C 價格](https://azure.microsoft.com/pricing/details/active-directory-b2c/)

#### <a name="migrate-to-ping-identity-or-auth0"></a>移轉至 Ping Identity 或 Auth0

在某些情況下，您可能會發現若不大幅變更程式碼，Azure AD 與 Azure AD B2C 便無法取代 Web 應用程式中的存取控制。 幾個常見的範例可能包括：

- 使用 WIF 或 WS-同盟來以 Google，或 Facebook 等社交識別提供者登入的 Web 應用程式。
- 透過 WS-同盟通訊協定與企業識別提供者執行直接同盟的 Web 應用程式。
- 要求社交識別提供者 (例如 Google 或 Facebook) 所簽發之存取權杖，必須以宣告形式儲存於由存取控制所簽發之權杖中的 Web 應用程式。
- 使用 Azure AD，或 Azure AD B2C 無法重現之複雜權杖轉換規則的 Web 應用程式。
- 使用 ACS 來集中管理與許多不同識別提供者之同盟的多租用戶 Web 應用程式

在這些情況下，您可能會想考慮將 Web 應用程式移轉至另一個雲端驗證服務。 我們建議您探索下列選項。 以下每個選項均提供類似存取控制的功能：

|     |     | 
| --- | --- |
| ![Auth0](./media/active-directory-acs-migration/rsz_auth0.png) | [Auth0](https://auth0.com/acs) 是彈性的雲端識別服務，不僅建立了[適用於存取控制客戶的高階移轉指南](https://auth0.com/acs)，而且幾乎能支援 ACS 支援的所有功能。 |
| ![Ping](./media/active-directory-acs-migration/rsz_ping.png) | [Ping Identity](https://www.pingidentity.com) 提供兩種類似 ACS 的解決方案。 PingOne 是支援許多與 ACS 相同之功能的雲端識別服務，而 PingFederate 則是類似的內部部署身分識別產品，不過彈性更大。 如需有關使用這些產品的詳細資料，請參閱 [Ping 的 ACS 淘汰指導](https://www.pingidentity.com/en/company/blog/2017/11/20/migrating_from_microsoft_acs_to_ping_identity.html)。 |

我們與 Ping Identity 和 Auth0 合作的目的是，確保所有存取控制客戶都能獲得從存取控制中，盡可能簡單地搬移其應用程式和服務的移轉方式。

<!--

## Sharepoint 2010, 2013, 2016

TODO: Azure AD only, use Azure AD SAML 1.1 tokens, when we bring it back online.
Other IDPs: use Auth0? https://auth0.com/docs/integrations/sharepoint.

-->

### <a name="web-services-that-use-active-authentication"></a>使用主動驗證的 Web 服務

存取控制針對受到存取控制簽發之權杖保護的 Web 服務，提供下列功能：

- 在您的存取控制命名空間中註冊一或多個服務識別。 使用服務識別要求權杖。
- 可透過使用下列類型的認證，來支援 OAuth WRAP 和 OAuth 2.0 Draft 13 通訊協定要求權杖：
    - 為服務識別建立簡單密碼
    - 使用對稱金鑰或 X509 憑證的已簽署 SWT
    - 由受信任的識別提供者 (通常是 AD FS 執行個體) 簽發的 SAML 權杖
- 支援下列權杖格式：JWT、SAML 1.1、SAML 2.0 和 SWT。
- 簡單權杖轉換規則。

存取控制中的服務識別通常用於實作伺服器對伺服器驗證。 

#### <a name="migrate-to-azure-active-directory"></a>移轉至 Azure Active Directory

我們對於這類驗證流程的建議是移轉至 [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/)。 Azure AD 是 Microsoft 工作或學校帳戶的雲端式識別提供者。 Azure AD 是 Office 365、Azure 和其他產品的識別提供者。 

透過授與 OAuth 用戶端認證的 Azure AD 實作，您也可以使用 Azure AD 進行伺服器對伺服器驗證。 下表比較存取控制的伺服器對伺服器驗證功能，以及 Azure AD 的同等功能。

| 功能 | 存取控制支援 | Azure AD 支援 |
| ---------- | ----------- | ---------------- |
| 如何註冊 Web 服務 | 在存取控制管理入口網站中建立信賴憑證者 | 在 Azure 入口網站中建立 Azure AD Web 應用程式 |
| 如何註冊用戶端 | 在存取控制管理入口網站中建立服務識別 | 在 Azure 入口網站中建立另一個 Azure AD Web 應用程式 |
| 使用的通訊協定 |- OAuth WRAP 通訊協定<br />- 授與 OAuth 2.0 Draft 13 用戶端認證 | OAuth 2.0 用戶端認證授與 |
| 用戶端驗證方法 |- 簡單密碼<br />- 已簽署的 SWT<br />- 來自同盟識別提供者的 SAML 權杖 |- 簡單密碼<br />- 已簽署的 JWT |
| 權杖格式 |- JWT<br />- SAML 1.1<br />- SAML 2.0<br />- SWT<br /> | 僅限 JWT |
| 權杖轉換 |- 新增自訂宣告<br />- 簡單的 if-then 宣告簽發邏輯 | 新增自訂宣告 | 
| 自動化設定和管理工作 | 使用存取控制管理服務則可支援 | 使用 Microsoft Graph 和 Azure AD 圖形 API 則可支援 |

如需有關實作伺服器對伺服器案例的指導，請參閱下列資源：

- [Azure AD 開發人員指南](https://aka.ms/aaddev)的服務對服務章節
- [使用簡單密碼用戶端認證的精靈程式碼範例](https://github.com/Azure-Samples/active-directory-dotnet-daemon)
- [使用憑證用戶端認證的精靈程式碼範例](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)

#### <a name="migrate-to-ping-identity-or-auth0"></a>移轉至 Ping Identity 或 Auth0

在某些情況下，您可能會發現若不大幅變更程式碼，Azure AD 用戶端認證與 OAuth 授與實作便無法取代 Web 應用程式中的存取控制。 幾個常見的範例可能包括：

- 使用非 JWT 權杖格式的伺服器對伺服器驗證。
- 使用外部識別提供者提供之輸入權杖的伺服器對伺服器驗證。
- 使用 Azure AD 無法重現之權杖轉換規則的伺服器對伺服器驗證。

在這些情況下，您可能需考慮將 Web 應用程式移轉至另一個雲端驗證服務。 我們建議您探索下列選項。 以下每個選項均提供類似存取控制的功能：

|     |     | 
| --- | --- |
| ![Auth0](./media/active-directory-acs-migration/rsz_auth0.png) | [Auth0](https://auth0.com/acs) 是彈性的雲端識別服務，不僅建立了[適用於存取控制客戶的高階移轉指南](https://auth0.com/acs)，而且幾乎能支援 ACS 支援的所有功能。 |
| ![Ping](./media/active-directory-acs-migration/rsz_ping.png) | [Ping Identity](https://www.pingidentity.com) 提供兩種類似 ACS 的解決方案。 PingOne 是支援許多與 ACS 相同之功能的雲端識別服務，而 PingFederate 則是類似的內部部署身分識別產品，不過彈性更大。 如需有關使用這些產品的詳細資料，請參閱 [Ping 的 ACS 淘汰指導](https://www.pingidentity.com/en/company/blog/2017/11/20/migrating_from_microsoft_acs_to_ping_identity.html)。 |

我們與 Ping Identity 和 Auth0 合作的目的是，確保所有存取控制客戶都能獲得從存取控制中，盡可能簡單地搬移其應用程式和服務的移轉方式。

## <a name="questions-concerns-and-feedback"></a>問題、考量與意見反應

我們了解許多存取控制客戶在閱讀本文之後依然無法決定使用哪個的移轉方式。 您可能需要一些有助於判斷合適計劃的協助或指導。 如果想要討論您的移轉案例和問題，請在本頁面上留言。
