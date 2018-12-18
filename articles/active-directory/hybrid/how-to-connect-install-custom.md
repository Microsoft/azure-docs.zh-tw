---
title: Azure AD Connect：自訂安裝 | Microsoft Docs
description: 這份文件詳述了 Azure AD Connect 的自訂安裝選項。 使用下列指示，透過 Azure AD Connect 安裝 Active Directory。
services: active-directory
keywords: 何謂 Azure AD Connect、安裝 Active Directory、Azure AD 的必要元件
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 6d42fb79-d9cf-48da-8445-f482c4c536af
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/04/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 95a0ffc6deecb7fd3bd6fce740f578352fdf2d66
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48888089"
---
# <a name="custom-installation-of-azure-ad-connect"></a>自訂 Azure AD Connect 安裝
當您想要更多安裝選項時，可使用 Azure AD Connect **自訂設定** 。 如果您有多個樹系，或如果您想要設定未涵蓋在快速安裝中的選用功能，可使用它。 只要是[**快速安裝**](how-to-connect-install-express.md)選項不能滿足部署或拓撲的情況，就可使用它。

在開始安裝 Azure AD Connect 之前，請務必要[下載 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771) 並完成 [Azure AD Connect：硬體和必要條件](how-to-connect-install-prerequisites.md)中的必要條件步驟。 另外，也請確定您具有 [Azure AD Connect 帳戶與權限](reference-connect-accounts-permissions.md)中所述的必要帳戶。

如果自訂的設定不符合拓撲，例如若要升級 DirSync，請參閱 [相關文件](#related-documentation) 中的其他案例。

## <a name="custom-settings-installation-of-azure-ad-connect"></a>Azure AD Connect 的自訂設定安裝
### <a name="express-settings"></a>快速設定
在此頁面上，按一下 [自訂]  以啟動自訂設定安裝。

### <a name="install-required-components"></a>安裝必要的元件
安裝同步處理服務時，您可以將選用組態區段保持未核取狀態，Azure AD Connect 會自動設定所有項目。 它會設定 SQL Server 2012 Express LocalDB 執行個體、建立適當的群組，以及指派權限。 如果您想要變更預設值，則可以使用下表了解可用的選用組態選項。

![必要的元件](./media/how-to-connect-install-custom/requiredcomponents.png)

| 選用組態 | 說明 |
| --- | --- |
| 使用現有的 SQL Server |可讓您指定 SQL Server 名稱和執行個體名稱。 如果您已經有想要使用的 ad 資料庫伺服器，請選擇這個選項。 如果您的 SQL Server 未啟用瀏覽，請在 [執行個體名稱]  中輸入執行個體名稱加上逗號及連接埠號碼。 |
| 使用現有的服務帳戶 |Azure AD Connect 預設會使用虛擬服務帳戶，以供同步處理服務使用。 如果您是使用遠端 SQL Server 或需要驗證的 Proxy，則需要使用**受控服務帳戶**，或使用網域中知道密碼的服務帳戶。 在這類情況下，請輸入要使用的帳戶。 請確定執行安裝的使用者為 SQL 中的 SA，才可建立服務帳戶的登入。  請參閱 [Azure AD Connect 帳戶與權限](reference-connect-accounts-permissions.md#adsync-service-account)。 </br></br>使用最新的組建，SQL 管理員即可執行頻外資料庫佈建，然後由具有資料庫擁有者權限的 Azure AD Connect 管理員進行安裝。  如需詳細資訊，請參閱[使用 SQL 委派的管理員權限安裝 Azure AD Connect](how-to-connect-install-sql-delegation.md)。|
| 指定自訂同步群組 |Azure AD Connect 預設會在安裝同步處理服務時，建立四個伺服器的本機群組。 這些群組如下：[系統管理員] 群組、[操作員] 群組、[瀏覽] 群組和 [密碼重設群組]。 您可以在此指定自己的群組。 群組必須位於伺服器本機上，不能位於網域中。 |

### <a name="user-sign-in"></a>使用者登入
在安裝必要元件後，系統會要求您選取使用者的單一登入方法。 下表提供可用選項的簡短說明。 如需登入方法的完整說明，請參閱[使用者登入](plan-connect-user-signin.md)。

![使用者登入](./media/how-to-connect-install-custom/usersignin4.png)

| 單一登入選項 | 說明 |
| --- | --- |
| 密碼雜湊同步處理 |使用者可使用他們在內部部署網路中使用的相同密碼登入 Microsoft Cloud 服務，例如 Office 365。 使用者密碼會以密碼雜湊的形式同步至 Azure AD，並在雲端中進行驗證。 如需詳細資訊，請參閱[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)。 |
|傳遞驗證|使用者可使用他們在內部部署網路中使用的相同密碼登入 Microsoft Cloud 服務，例如 Office 365。  使用者密碼會傳遞至內部部署 Active Directory 網域控制站進行驗證。
| 與 AD FS 同盟 |使用者可使用他們在內部部署網路中使用的相同密碼登入 Microsoft Cloud 服務，例如 Office 365。  系統會將使用者重新導向至他們的內部部署 AD FS 執行個體以進行登入，並在內部部署中進行驗證。 |
| 與 PingFederate 同盟|使用者可使用他們在內部部署網路中使用的相同密碼登入 Microsoft Cloud 服務，例如 Office 365。  系統會將使用者重新導向至他們的內部部署 PingFederate 執行個體以進行登入，並在內部部署中進行驗證。 |
| 請勿設定 |不會安裝和設定任何使用者登入功能。 如果您已經有第三方的同盟伺服器或另一個現有的適當方案，請選擇此選項。 |
|啟用單一登入|此選項同時適用於密碼雜湊同步處理和傳遞驗證，並可為公司網路上的桌上型電腦使用者提供單一登入體驗。 如需詳細資訊，請參閱[單一登入](how-to-connect-sso.md)。 </br>請注意，AD FS 客戶無法使用此選項，因為 AD FS 已提供相同層級的單一登入。</br>

### <a name="connect-to-azure-ad"></a>連接至 Azure AD
在 [連接至 Azure AD] 畫面中，輸入全域系統管理員的帳戶和密碼。 如果您在前一個頁面選取 [與 AD FS 同盟]，請勿以您打算啟用同盟的網域中的帳戶登入。 建議使用 Azure AD 租用戶內預設 **onmicrosoft.com** 網域中的帳戶。

此帳戶只會用來在 Azure AD 中建立服務帳戶，而且在精靈完成後便不會使用。  
![使用者登入](./media/how-to-connect-install-custom/connectaad.png)

如果您的全域管理員帳戶已啟用 MFA，您需要在登入快顯視窗中再次提供密碼，並完成 MFA 認證。 認證可能是提供驗證碼或撥打電話。  
![使用者登入 MFA](./media/how-to-connect-install-custom/connectaadmfa.png)

全域管理員帳戶也可以啟用 [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md)。

如果您收到錯誤訊息，而且有連線問題，請參閱[針對連線問題進行疑難排解](tshoot-connect-connectivity.md)。

## <a name="pages-under-the-sync-section"></a>[同步] 一節下的頁面

### <a name="connect-your-directories"></a>連接您的目錄
若要連線到您的 Active Directory 網域服務，Azure AD Connect 需要樹系名稱和具有足夠權限的帳戶認證。

![連線目錄](./media/how-to-connect-install-custom/connectdir01.png)

輸入樹系名稱並按一下 [新增目錄] 之後，快顯對話方塊隨即出現並提示您使用下列選項︰

| 選項 | 說明 |
| --- | --- |
| 建立新帳戶 | 如果您想要 Azure AD Connect 精靈建立 Azure AD Connect 在目錄同步處理期間連線至 AD 樹系所需的 AD DS 帳戶，請選取此選項。 選取此選項後，輸入企業管理帳戶的使用者名稱和密碼。 Azure AD Connect 精靈將會使用提供的企業管理帳戶來建立所需的 AD DS 帳戶。 您可以用 NetBios 或 FQDN 格式輸入網域部分，也就是 FABRIKAM\administrator 或 fabrikam.com\administrator。 |
| 使用現有帳戶 | 如果您想要提供現有的 AD DS 帳戶，以便 Azure AD Connect 在目錄同步處理期間用於連線至 AD 樹系，請選取此選項。 您可以用 NetBios 或 FQDN 格式輸入網域部分，也就是 FABRIKAM\syncuser 或 fabrikam.com\syncuser。 此帳戶可以是一般使用者帳戶，因為我們只需要預設的讀取權限。 不過，視您的情況而定，也可能需要更多權限。 如需詳細資訊，請參閱 [Azure AD Connect 帳戶與權限](reference-connect-accounts-permissions.md##create-the-ad-ds-connector-account)。 |

![連線目錄](./media/how-to-connect-install-custom/connectdir02.png)

### <a name="azure-ad-sign-in-configuration"></a>Azure AD 登入組態
此頁面可讓您檢閱內部部署 AD DS 中存在的 UPN 網域，以及已在 Azure AD 中驗證的 UPN 網域。 此頁面也可讓您設定要用於 userPrincipalName 的屬性。

![未驗證的網域](./media/how-to-connect-install-custom/aadsigninconfig2.png)  
檢閱每一個標示為**未新增**和**未驗證**的網域。 確定您所使用的網域皆已在 Azure AD 中完成驗證。 驗證好網域時，按一下 [重新整理] 符號。 如需詳細資訊，請參閱[新增並驗證網域](../active-directory-domains-add-azure-portal.md)

**UserPrincipalName** ：屬性 userPrincipalName 是使用者登入 Azure AD 和 Office 365 時會使用的屬性。 使用的網域 (也稱為 UPN 尾碼)，應該會在同步處理使用者前於 Azure AD 中進行驗證。 Microsoft 建議保留預設屬性 userPrincipalName。 如果此屬性不可路由傳送且無法驗證，則可以選取另一個屬性。 例如，您可以選取電子郵件做為保存登入識別碼的屬性。 使用 userPrincipalName 之外的其他屬性稱為 **替代 ID**。 替代 ID 屬性值必須遵循 RFC822 標準。 替代 ID 可與密碼雜湊同步處理、傳遞驗證和同盟搭配使用。 此屬性不得在 Active Directory 中定義為多重值 (即使它只有單一值)。

>[!NOTE]
> 當您啟用傳遞驗證時，您必須至少有一個已驗證網域才能繼續執行精靈。

> [!WARNING]
> 使用替代 ID 會與所有 Office 365 工作負載不相容。 如需詳細資訊，請參閱 [設定替代的登入 ID](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)。
>
>

### <a name="domain-and-ou-filtering"></a>網域和 OU 篩選
預設會同步所有網域和 OU。 如果您不想將部分網域或 OU 同步處理至 Azure AD，您可以取消選取這些網域和 OU。  
![DomainOU 篩選](./media/how-to-connect-install-custom/domainoufiltering.png)  
精靈的這個頁面會設定網域型和 OU 型篩選。 如果您打算進行變更，那麼在進行這些變更之前，請參閱[網域型篩選](how-to-connect-sync-configure-filtering.md#domain-based-filtering)和 [OU 型篩選](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering)。 某些 OU 對此功能而言是必要的，因此必須加以選取。

如果您使用 OU 型篩選搭配 1.1.524.0 版之前的 Azure AD Connect，預設會同步處理稍後新增的 OU。 如果您希望不要同步處理新的 OU，則可以在精靈完成 [OU 型篩選](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering)後加以設定。 對於 Azure AD Connect 1.1.524.0 版或之後的版本，您可以指出是否想要同步處理新的 OU。

如果您打算使用[群組型篩選](#sync-filtering-based-on-groups)，則確定已包含具有群組的 OU 包含且不會使用 OU 篩選進行篩選。 OU 篩選會在群組型篩選之前評估。

由於防火牆限制，也可能無法連線到某些網域。 依預設不會選取這些網域，而且會有警告。  
![無法連線到網域](./media/how-to-connect-install-custom/unreachable.png)  
如果您看到此警告，請確定這些網域確實無法連線，因此預期會有警告。

### <a name="uniquely-identifying-your-users"></a>唯一識別您的使用者

#### <a name="select-how-users-should-be-identified-in-your-on-premises-directories"></a>選取在內部部署目錄中要如何識別使用者
跨樹系比對功能可讓您定義 AD DS 樹系中的使用者在 Azure AD 中的顯示方式。 使用者可能會在整個樹系中只顯示一次，或是具有啟用和停用帳戶的組合。 使用者也可能顯示為某些樹系中的連絡人。

![唯一](./media/how-to-connect-install-custom/unique2.png)

| 設定 | 說明 |
| --- | --- |
| [使用者在所有樹系中只會顯示一次](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |在 Azure AD 中，所有使用者會都建立為個別物件。 在 Metaverse 中這些物件不會聯結。 |
| [郵件屬性](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |如果郵件屬性在不同樹系中具有相同的值，則此選項就會聯結使用者和連絡人。 如果已透過 GALSync 建立了您的連絡人，請使用此選項。 如果選擇此選項，則不會將未填入其 Mail 屬性的 User 物件同步處理至 Azure AD。 |
| [ObjectSID 與 msExchangeMasterAccountSID/ msRTCSIP-OriginatorSid](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |此選項會聯結帳戶樹系中已啟用的使用者與資源樹系中已停用的使用者。 在 Exchange 中，此組態稱為連結信箱。 如果您只使用 Lync 而 Exchange 不存在資源樹系中，也可以使用此選項。 |
| sAMAccountName 與 MailNickName |此選項會在預期可以找到使用者登入 ID 的屬性中聯結。 |
| 特定的屬性 |此選項可讓您選取您的屬性。 如果選擇此選項，則不會將未填入其 (已選取) 屬性的 User 物件同步處理至 Azure AD。 **限制：** 確定選擇的是已可在 Metaverse 中找到的屬性。 如果您選擇自訂屬性 (並非在 Metaverse 中)，精靈將無法完成。 |

#### <a name="select-how-users-should-be-identified-with-azure-ad---source-anchor"></a>選取要如何使用 Azure AD 識別使用者 - 來源錨點
屬性 sourceAnchor 是使用者物件存留期間都不會變更的屬性。 它是連結內部部署使用者與 Azure AD 中使用者的主要金鑰。

| 設定 | 說明 |
| --- | --- |
| 讓 Azure 為我管理來源錨點 | 如果您想要 Azure AD 為您挑選屬性，請選取此選項。 如果您選取此選項，Azure AD Connect 精靈會套本文的 [Azure AD Connect︰設計概念 - 使用 ms-DS-ConsistencyGuid 作為 sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor) 一節所述的 sourceAnchor 屬性選取邏輯。 自訂安裝完成之後，此精靈會通知您哪些屬性已被選為來源錨點屬性。 |
| 特定的屬性 | 如果您希望指定現有的 AD 屬性作為 sourceAnchor 屬性，請選取此選項。 |

因為無法改變屬性，所以您必須規劃並使用好的屬性。 objectGUID 就是不錯的選項。 只要使用者帳戶沒有在樹系/網域之間移動，此屬性就不會改變。 請避免使用會在某人結婚或變更指派時改變的屬性。 因為不可以使用帶有 @-sign 的屬性，所以無法使用 email 和 userPrincipalName。 屬性也有區分大小寫，因此在樹系間移動物件時，請務必保留大寫/小寫。 二進位屬性會以 base64 編碼，但其他屬性類型則會維持未編碼狀態。 在同盟情況以及部分 Azure AD 介面中，此屬性也稱為 immutableID。 您可以在[設計概念](plan-connect-design-concepts.md#sourceanchor)中找到關於來源錨點的詳細資訊。

### <a name="sync-filtering-based-on-groups"></a>根據群組進行同步處理篩選
篩選群組功能可讓您只同步處理一小部分的物件來進行試驗。 若要使用這項功能，請在內部部署 Active Directory 中建立此目的專用的群組。 然後新增應該同步處理至 Azure AD 做為直接成員的使用者和群組。 您稍後可以在此群組中新增和移除使用者，藉此維護應該要顯示在 Azure AD 中的物件清單。 所有您想要同步處理的物件，都必須是直接隸屬於群組的成員。 使用者、群組、連絡人及電腦/裝置全都必須是直接成員。 系統不會解析巢狀群組成員資格。 當您新增群組做為成員時，只會新增群組本身而不會新增其成員。

![同步處理篩選](./media/how-to-connect-install-custom/filter2.png)

> [!WARNING]
> 這項功能僅適用於支援試驗部署。 請勿將其用於成熟的生產部署。
>
>

在成熟的生產部署中，維護含有所有要同步處理之物件的單一群組將非常困難。 您應該改用[設定篩選](how-to-connect-sync-configure-filtering.md)中的其中一種方法。

### <a name="optional-features"></a>選用功能
此畫面可讓您針對特定情況選取選用功能。

>[!WARNING]
>Azure AD Connect **1.0.8641.0** 版和較舊版本會依賴 Azure 存取控制服務來進行密碼回寫。  這項服務將於 **2018 年 11 月 7 日**淘汰。  如果您使用上述任何版本的 Azure AD Connect 並已啟用密碼回寫，當服務淘汰後，使用者可能會無法變更或重設其密碼。 我們不會在這些版本的 Azure AD Connect 上支援密碼回寫。
>
>如需 Azure 存取控制服務的詳細資訊，請參閱[操作說明：從 Azure 存取控制服務遷移](../develop/active-directory-acs-migration.md)
>
>若要下載最新版的 Azure AD Connect，請按一下[這裡](https://www.microsoft.com/en-us/download/details.aspx?id=47594)。

![選用功能](./media/how-to-connect-install-custom/optional2.png)

> [!WARNING]
> 如果您目前啟用 DirSync 或 Azure AD Sync，請不要在 Azure AD Connect 中啟動任何回寫功能。



| 選用功能 | 說明 |
| --- | --- |
| Exchange 混合部署 |Exchange 混合部署功能允許在內部部署和 Office 365 中並存 Exchange 信箱。 Azure AD Connect 會將一組特定的[屬性](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback)從 Azure AD 同步處理回內部部署目錄。 |
| Exchange 郵件公用資料夾 | 「Exchange 郵件公用資料夾」功能可讓您將擁有郵件功能的公用資料夾物件從內部部署 Active Directory 同步處理到 Azure AD。 |
| Azure AD 應用程式和屬性篩選 |透過啟用 Azure AD 應用程式和屬性篩選，即可調整這組同步處理的屬性。 這個選項會在精靈中另外新增兩個組態頁面。 如需詳細資訊，請參閱 [Azure AD 應用程式和屬性篩選](#azure-ad-app-and-attribute-filtering)。 |
| 密碼雜湊同步處理 |如果您選取同盟做為登入解決方案，您可以啟用此選項。 密碼雜湊同步處理可作為備份選項。 如需其他資訊，請參閱[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)。 </br></br>如果您已選取傳遞驗證，則預設也會啟用此選項，確保能夠支援舊版用戶端並作為備用選項。 如需其他資訊，請參閱[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)。|
| 密碼回寫 |透過啟用密碼回寫，在 Azure AD 中產生的密碼變更會回寫至內部部署目錄。 如需詳細資訊，請參閱[開始使用密碼管理](../authentication/quickstart-sspr.md)。 |
| 群組回寫 |如果您使用 [Office 365 群組]  功能，就可以在內部部署的 Active Directory 中顯示這些群組。 只有當您內部部署的 Active Directory 中已經有 Exchange 時，才能使用此選項。 如需詳細資訊，請參閱[群組回寫](how-to-connect-preview.md#group-writeback)。 |
| 裝置回寫 |在條件式存取情況下，可讓您將 Azure AD 中的裝置物件回寫到內部部署的 Active Directory。 如需詳細資訊，請參閱[在 Azure AD Connect 中啟用裝置回寫](how-to-connect-device-writeback.md)。 |
| 目錄擴充屬性同步處理 |透過啟用目錄擴充屬性同步處理，指定的屬性將會同步處理至 Azure AD。 如需詳細資訊，請參閱[目錄擴充](how-to-connect-sync-feature-directory-extensions.md)。 |

### <a name="azure-ad-app-and-attribute-filtering"></a>Azure AD 應用程式和屬性篩選
如果您想要限制要將哪些屬性同步處理至 Azure AD，請先選取您將會使用的服務。 如果您在此頁面上進行組態變更，則必須重新執行安裝精靈來明確選取新的服務。

![選用功能應用程式](./media/how-to-connect-install-custom/azureadapps2.png)

根據在上一個步驟中選取的服務，此頁面會顯示將同步處理的所有屬性。 這份清單是正在同步處理的所有物件類型的組合。 如果其中有一些您不需要同步處理的特定屬性，您可以取消選取那些屬性。

![選用功能屬性](./media/how-to-connect-install-custom/azureadattributes2.png)

> [!WARNING]
> 移除可能影響功能的屬性。 如需最佳做法和建議，請參閱[同步處理的屬性](reference-connect-sync-attributes-synchronized.md#attributes-to-synchronize)。
>
>

### <a name="directory-extension-attribute-sync"></a>目錄擴充屬性同步處理
您可以使用組織新增的自訂屬性或 Active Directory 中的其他屬性在 Azure AD 中擴充結構描述。 若要使用這項功能，請選取 [選用功能] 頁面上的 [目錄擴充屬性同步處理]。 您可以在此頁面上選取更多要同步處理的屬性。

>[!NOTE]
>[可用屬性] 方塊會區分大小寫。

![目錄擴充](./media/how-to-connect-install-custom/extension2.png)

如需詳細資訊，請參閱[目錄擴充](how-to-connect-sync-feature-directory-extensions.md)。

### <a name="enabling-single-sign-on-sso"></a>啟用單一登入 (SSO)
設定單一登入以用於密碼同步處理或傳遞驗證是一道簡單的程序，您只需要對每個要同步至 Azure AD 的樹系完成一次即可。 設定程序包含兩個步驟，如下所示︰

1.  在內部部署 Active Directory 中建立所需的電腦帳戶。
2.  設定用戶端機器的內部網路區域，以支援單一登入。

#### <a name="create-the-computer-account-in-active-directory"></a>在 Active Directory 中建立電腦帳戶
對於在 Azure AD Connect 中新增的每個樹系，您必須提供網域系統管理員認證，才能在每個樹系中建立電腦帳戶。 認證只會用來建立帳戶，不會存放或用於其他任何作業。 只需在 Azure AD Connect 精靈的 [啟用單一登入] 頁面上新增認證，如下所示︰

![啟用單一登入](./media/how-to-connect-install-custom/enablesso.png)

>[!NOTE]
>如果您不想對特定樹系使用單一登入，您可以略過該樹系。

#### <a name="configure-the-intranet-zone-for-client-machines"></a>設定用戶端機器的內部網路區域
若要確保用戶端會自動登入內部網路區域，您必須確定兩個 URL 屬於內部網路區域。 這可確保加入網域的電腦會在連接到公司網路時自動將 Kerberos 票證傳送至 Azure AD。
在具有群組原則管理工具的電腦上。

1.  開啟 [群組原則管理工具]
2.  編輯會套用至所有使用者的群組原則。 例如，預設網域原則。
3.  瀏覽至 **User Configuration\Administrative Templates\Windows Components\Internet Explorer\Internet Control Panel\Security Page**，並根據下圖選取 [指派網站到區域清單]。
4.  啟用原則，並在對話方塊中輸入下列項目。

        Value: `https://autologon.microsoftazuread-sso.com`  
        Data: 1  


5.  看起來應該會像下面這樣：  
![內部網路區域](./media/how-to-connect-install-custom/sitezone.png)

6.  按兩次 [確定]。

## <a name="configuring-federation-with-ad-fs"></a>設定與 AD FS 同盟
使用 Azure AD Connect 設定 AD FS 的作業很簡單，只需要按幾下就能完成。 進行設定之前必須先具備下列項目。

* 做為同盟伺服器的 Windows Server 2012 R2 或更新版本伺服器，且已啟用遠端管理
* 做為 Web 應用程式 Proxy 伺服器的 Windows Server 2012 R2 或更新版本伺服器，且已啟用遠端管理
* 您想要使用的 Federation Service 名稱 (例如 sts.contoso.com) 的 SSL 憑證

>[!NOTE]
>即使您未使用 Azure AD Connect 來管理您的同盟信任，您也可以使用 Azure AD Connect 更新 AD FS 伺服器陣列的 SSL 憑證。

### <a name="ad-fs-configuration-pre-requisites"></a>AD FS 組態必要條件
若要使用 Azure AD Connect 設定 AD FS 伺服器陣列，請確定已在遠端伺服器啟用 WinRM。 確定您已完成[同盟必要條件](how-to-connect-install-prerequisites.md#prerequisites-for-federation-installation-and-configuration)中的其他工作。 此外，請完成[表 3 - Azure AD Connect 和同盟伺服器/WAP](reference-connect-ports.md#table-3---azure-ad-connect-and-ad-fs-federation-serverswap) 中列出的連接埠需求。

### <a name="create-a-new-ad-fs-farm-or-use-an-existing-ad-fs-farm"></a>建立新的 AD FS 伺服器陣列或使用現有的 AD FS 伺服器陣列
您可以使用現有的 AD FS 伺服器陣列，或選擇建立新的 AD FS 伺服器陣列。 如果您選擇建立新的伺服器陣列，就必須提供 SSL 憑證。 如果 SSL 憑證有密碼保護，則系統會提示您輸入密碼。

![AD FS 伺服器陣列](./media/how-to-connect-install-custom/adfs1.png)

如果您選擇使用現有的 AD FS 伺服器陣列，將會前往設定 AD FS 與 Azure AD 之間信任關係的畫面。

>[!NOTE]
>Azure AD Connect 僅可用於管理一個 AD FS 伺服器陣列。 如果您與 Azure AD 的現有同盟信任是設定在選取的 AD FS 伺服器陣列上，則 Azure AD Connect 會從頭建立一次信任。

### <a name="specify-the-ad-fs-servers"></a>指定 AD FS 伺服器
輸入想要在其中安裝 AD FS 的伺服器。 您可以根據容量規劃需求，加入一或多部伺服器。 請先將所有 AD FS 伺服器 (WAP 伺服器則不必) 加入 Active Directory，再執行這項設定。 Microsoft 建議安裝一部專門用於測試和試驗部署的 AD FS 伺服器。 然後在完成初始設定之後透過再次執行 Azure AD Connect，新增及部署更多伺服器以符合您的調整需求。

> [!NOTE]
> 請先確認所有伺服器均已加入 AD 網域，再執行這項設定。
>
>

![AD FS 伺服器](./media/how-to-connect-install-custom/adfs2.png)

### <a name="specify-the-web-application-proxy-servers"></a>指定 Web 應用程式 Proxy 伺服器
輸入您要做為 Web 應用程式 Proxy 伺服器的伺服器。 Web 應用程式 Proxy 伺服器會部署在您的 DMZ (外部網路對應) 中，且支援來自外部網路的驗證要求。 您可以根據容量規劃需求，加入一或多部伺服器。 Microsoft 建議安裝一部專門用於測試和試驗部署的 Web 應用程式 Proxy 伺服器。 然後在完成初始設定之後透過再次執行 Azure AD Connect，新增及部署更多伺服器以符合您的調整需求。 我們建議準備同樣數目的 Proxy 伺服器，以滿足來自內部網路的驗證需求。

> [!NOTE]
> <li> 如果您使用的帳戶不是 WAP 伺服器上的本機系統管理員，則系統會提示您提供系統管理員認證。</li>
> <li> 執行此步驟之前，請確認 Azure AD Connect 伺服器和 Web 應用程式 Proxy 伺服器之間有 HTTP/HTTPS 連線。</li>
> <li> 請確認 Web 應用程式伺服器和 AD FS 伺服器之間的 HTTP/HTTPS 連線是否允許流入驗證要求。</li>
>

![Web 應用程式](./media/how-to-connect-install-custom/adfs3.png)

系統會提示您輸入認證，讓 Web 應用程式伺服器可以建立與 AD FS 伺服器的安全連線。 這些認證必須是 AD FS 伺服器上的本機系統管理員。

![Proxy](./media/how-to-connect-install-custom/adfs4.png)

### <a name="specify-the-service-account-for-the-ad-fs-service"></a>指定 AD FS 服務的服務帳戶
AD FS 服務需要網域服務帳戶來驗證使用者，以及在 Active Directory 中查閱使用者資訊。 它可支援兩種類型的服務帳戶：

* **群組受控服務帳戶** ：在 Windows Server 2012 的 Active Directory 網域服務中導入。 此類型的帳戶可為 AD FS 之類的服務提供單一帳戶，而不需要定期更新帳戶密碼。 如果您在 AD FS 伺服器所屬的網域中已經有 Windows Server 2012 網域控制站，請使用此選項。
* **網域使用者帳戶** ：此類型的帳戶會要求您提供密碼，並在密碼變更或到期時定期更新密碼。 只有當您在 AD FS 伺服器所屬的網域中沒有 Windows Server 2012 網域控制站時，才能使用此選項。

如果您選取了 [群組受控服務帳戶]，而這項功能從未在 Active Directory 中使用過，系統會提示您輸入企業系統管理員認證。 這些認證會用來啟動金鑰存放區，並在 Active Directory 中啟用這項功能。

> [!NOTE]
> Azure AD Connect 會執行檢查，以偵測 AD FS 服務是否已經註冊為網域中的 SPN。  AD DS 不允許同時註冊重複的 SPN。  如果找到重複的 SPN，請先移除此 SPN，才能繼續執行。

![AD FS 服務帳戶](./media/how-to-connect-install-custom/adfs5.png)

### <a name="select-the-azure-ad-domain-that-you-wish-to-federate"></a>選取您想要建立同盟的 Azure AD 網域
此組態會用來設定 AD FS 與 Azure AD 之間的同盟關係。 它會設定 AD FS 將安全性權杖簽發給 Azure AD，並將 Azure AD 設定為信任來自此特定 AD FS 執行個體的權杖。 此頁面只能讓您在初始安裝中設定單一網域。 您可稍後再次執行 Azure AD Connect 以設定其他網域。

![Azure AD 網域](./media/how-to-connect-install-custom/adfs6.png)

### <a name="verify-the-azure-ad-domain-selected-for-federation"></a>驗證所選取用於同盟的 Azure AD 網域
當您選取要同盟的網域時，Azure AD Connect 會提供您所需資訊以供您驗證尚未驗證的網域。 請參閱[新增並驗證網域](../active-directory-domains-add-azure-portal.md)以了解如何使用這項資訊。

![Azure AD 網域](./media/how-to-connect-install-custom/verifyfeddomain.png)

> [!NOTE]
> AD Connect 會嘗試在設定階段驗證網域。 如果您繼續進行設定，但未加入必要的 DNS 記錄，精靈將無法完成設定。
>
>

## <a name="configuring-federation-with-pingfederate"></a>設定與 PingFederate 的同盟
使用 Azure AD Connect 設定 PingFederate 的作業很簡單，只需要按幾下就能完成。 但您必須符合下列必要條件。
- PingFederate 8.4 或更新版本。  如需詳細資訊，請參閱 [PingFederate 與 Azure Active Directory 和 Office 365 的整合](https://docs.pingidentity.com/bundle/O365IG20_sm_integrationGuide/page/O365IG_c_integrationGuide.html)
- 您想要使用的 Federation Service 名稱 (例如 sts.contoso.com) 的 SSL 憑證

### <a name="verify-the-domain"></a>驗證網域
選取 [與 PingFederate 同盟] 後，系統會要求您驗證要建立同盟的網域。  從下拉式方塊中選取網域。

![驗證網域](./media/how-to-connect-install-custom/ping1.png)

### <a name="export-the-pingfederate-settings"></a>匯出 PingFederate 設定


PingFederate 必須設定為每個同盟 Azure 網域的同盟伺服器。  按一下 [匯出設定] 按鈕，然後與 PingFederate 管理員共用這項資訊。  同盟伺服器管理員會更新組態，然後提供 PingFederate 伺服器的 URL 和連接埠號碼，讓 Azure AD Connect 能夠驗證中繼資料設定。  

![驗證網域](./media/how-to-connect-install-custom/ping2.png)

若有任何驗證問題，請連絡您的 PingFederate 管理員加以解決。  下列範例說明與 Azure 之間沒有有效信任關係的 PingFederate 伺服器：

![信任](./media/how-to-connect-install-custom/ping5.png)




### <a name="verify-federation-connectivity"></a>驗證同盟連線能力
Azure AD Connect 會嘗試驗證在上一個步驟中從 PingFederate 中繼資料擷取的驗證端點。  Azure AD Connect 會先嘗試使用本機 DNS 伺服器解析這些端點。  接著，它會嘗試使用外部 DNS 提供者解析端點。  若有任何驗證問題，請連絡您的 PingFederate 管理員加以解決。  

![驗證連線能力](./media/how-to-connect-install-custom/ping3.png)

### <a name="verify-federation-login"></a>驗證同盟登入
最後，您可以藉由登入同盟網域，來驗證新設定的同盟登入流程。 如果登入成功，即表示與 PingFederate 的同盟已成功設定。
![驗證登入](./media/how-to-connect-install-custom/ping4.png)

## <a name="configure-and-verify-pages"></a>設定並確認頁面
設定會在此頁面上進行。

> [!NOTE]
> 在繼續安裝之前，如果已設定同盟，請確定您已設定[同盟伺服器的名稱解析](how-to-connect-install-prerequisites.md#name-resolution-for-federation-servers)。
>
>


![準備設定](./media/how-to-connect-install-custom/readytoconfigure2.png)

### <a name="staging-mode"></a>預備模式
您可以在預備模式下同時設定新的同步處理伺服器。 僅支援將一部同步處理伺服器匯出到雲端中的一個目錄。 但如果您想要從另一部伺服器移動，例如執行 DirSync 的伺服器，可以啟用預備模式中的 Azure AD Connect。 啟用時，同步處理引擎會照常匯入和同步處理資料，但它不會將任何項目匯出至 Azure AD 或 AD。 在預備模式下，將會停用功能密碼同步處理和密碼回寫。

![預備模式](./media/how-to-connect-install-custom/stagingmode.png)

在預備模式中，可以對同步處理引擎進行所需的變更，並檢閱要匯出的項目。 當此組態看起來設定良好時，請再次執行安裝精靈，並停用預備模式。 資料現在會從這個伺服器匯出至 Azure AD。 同時請務必停用其他伺服器，如此才能只讓一部伺服器主動匯出。

如需詳細資訊，請參閱[預備模式](how-to-connect-sync-operations.md#staging-mode)。

### <a name="verify-your-federation-configuration"></a>驗證同盟組態
當您按一下 [驗證] 按鈕時，Azure AD Connect 會為您驗證 DNS 設定。

**內部網路連線能力檢查**

* 解決同盟 FQDN：Azure AD Connect 會檢查 DNS 是否可以解析同盟 FQDN，以確保連線能力。 如果 Azure AD Connect 無法解析 FQDN，驗證將會失敗。 確保同盟服務 FQDN 有 DNS 記錄存在，才能成功完成驗證。
* DNS A 記錄：Azure AD Connect 會檢查您的同盟服務是否有 A 記錄。 如果沒有使用 A 記錄，驗證將會失敗。 為您的同盟 FQDN 建立 A 記錄 (而非 CNAME 記錄)，才能成功完成驗證。

**外部網路連線能力檢查**

* 解決同盟 FQDN：Azure AD Connect 會檢查 DNS 是否可以解析同盟 FQDN，以確保連線能力。

![完成](./media/how-to-connect-install-custom/completed.png)

![驗證](./media/how-to-connect-install-custom/adfs7.png)

若要驗證端對端驗證已成功執行，您應手動執行下列一或多項測試：

* 在同步處理完成後，請使用「在 Azure AD Connect 中驗證同盟登入的其他工作」，對您選擇的內部部署使用者帳戶進行驗證。
* 驗證您可以在內部網路中從已加入網域的電腦使用瀏覽器進行登入：連線至 https://myapps.microsoft.com，並使用您已登入的帳戶驗證登入。 內建的 AD DS 系統管理員帳戶未同步處理，不能用於驗證。
* 驗證您可以從外部網路的裝置登入。 在家用電腦或行動裝置上連線至 https://myapps.microsoft.com，並提供您的認證。
* 驗證豐富型用戶端登入。 連線至 https://testconnectivity.microsoft.com，選擇 [Office 365] 索引標籤，然後選擇 [Office 365 單一登入測試]。

## <a name="troubleshooting"></a>疑難排解
下一節包含遇到 Azure AD Connect 安裝問題時，您可以使用的疑難排解和資訊。

### <a name="the-adsync-database-already-contains-data-and-cannot-be-overwritten"></a>「ADSync 資料庫已包含資料，而且無法覆寫」
如果您使用 Azure AD Connect 自訂安裝，並在 [安裝必要元件] 頁面上選取 [使用現有 SQL 伺服器] 選項，則可能會出現錯誤，其中指出 **ADSync 資料庫已包含資料，而且無法覆寫。請移除現有資料庫，然後再試一次。**

![Error](./media/how-to-connect-install-custom/error1.png)

這是因為對於上述文字方塊中指定的 SQL 伺服器而言，其 SQL 執行個體上已經有名為 **ADSync** 的現有資料庫。

這通常會在您解除安裝 Azure AD Connect 之後發生。  當您解除安裝時，資料庫不會從 SQL Server 中刪除。

若要修正此問題，請先確認 Azure AD Connect 在解除安裝前所使用的 **ADSync** 資料庫已不會再使用。

接下來，建議您在刪除資料庫前先加以備份。

最後，您必須刪除資料庫。  您可以使用 **Microsoft SQL Server Management Studio** 並連線至 SQL 執行個體，來完成此動作。 尋找 **ADSync** 資料庫，以滑鼠右鍵按一下該項目，然後從捷徑功能表中選取 [刪除]。  按一下 [確定] 按鈕來將其刪除。

![Error](./media/how-to-connect-install-custom/error2.png)

當您刪除 **ADSync** 資料庫之後，您可以按一下 [安裝] 按鈕來重試安裝。

## <a name="next-steps"></a>後續步驟
安裝完成之後，請先登出 Windows 再重新登入，才能使用 Synchronization Service Manager 或同步處理規則編輯器。

安裝了 Azure AD Connect 之後，您可以 [驗證安裝和指派授權](how-to-connect-post-installation.md)。

深入了解這些在安裝時啟用的功能︰[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)和 [Azure AD Connect Health](how-to-connect-health-sync.md)。

深入了解這些常見主題︰[排程器和如何觸發同步處理](how-to-connect-sync-feature-scheduler.md)。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。