---
title: Azure AD Connect：帳戶與權限 | Microsoft 文件
description: 本主題描述使用和建立的帳戶以及所需的權限。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.reviewer: cychua
ms.assetid: b93e595b-354a-479d-85ec-a95553dd9cc2
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 96d8977a63d26576d4d783dd0661409fdcee90f8
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308857"
---
# <a name="azure-ad-connect-accounts-and-permissions"></a>Azure AD Connect：帳戶與權限

## <a name="accounts-used-for-azure-ad-connect"></a>用於 Azure AD Connect 的帳戶

![](media/reference-connect-accounts-permissions/account5.png)

Azure AD Connect 會使用 3 個帳戶，以便將資訊從內部部署或 Windows Server Active Directory 同步處理至 Azure Active Directory。  這些帳戶分別是：

- **AD DS 連接器帳戶**：用來在 Windows Server Active Directory 中讀取/寫入資訊

- **ADSync 服務帳戶**：用來執行同步處理服務，以及存取 SQL 資料庫

- **Azure AD 連接器帳戶**：用來在 Azure AD 中寫入資訊

除了這三個用來執行 Azure AD Connect 的帳戶，您另外還需要下列帳戶以便安裝 Azure AD Connect。  它們是：

- **AD DS 企業系統管理員帳戶**：用來安裝 Azure AD Connect
- **Azure AD 全域系統管理員帳戶**：用來建立 Azure AD Connector 帳戶和設定 Azure AD。

- **SQL SA 帳戶 (選擇性)**：用來在使用完整版 SQL Server 時建立 ADSync 資料庫。  這個 SQL Server 可位於 Azure AD Connect 安裝的本機或遠端。  此帳戶可以是和企業系統管理員相同的帳戶。  SQL 管理員可執行頻外資料庫佈建，然後由具有資料庫擁有者權限的 Azure AD Connect 管理員進行安裝。  如需這方面的資訊，請參閱[使用 SQL 委派的管理員權限安裝 Azure AD Connect](how-to-connect-install-sql-delegation.md)

## <a name="installing-azure-ad-connect"></a>安裝 Azure AD Connect
Azure AD Connect 安裝精靈提供兩個不同的路徑：

* 在 [快速設定] 中，精靈需要更多權限，  以便在不需要建立使用者或設定權限的情況下，輕鬆設定您的組態。
* 在 [自訂設定] 中，精靈會提供更多選擇和選項。 不過，在某些情況下，您必須確定自己擁有正確的權限。



## <a name="express-settings-installation"></a>快速設定安裝
在快速設定中，安裝精靈會要求下列項目：

  - AD DS 企業系統管理員認證
  - Azure AD 全域系統管理員認證

### <a name="ad-ds-enterprise-admin-credentials"></a>AD DS 企業系統管理員認證
AD DS 企業系統管理員帳戶可用來設定內部部署 Active Directory。 這些認證只能在安裝期間使用，安裝完成後則無法使用。 企業管理員而不是網域管理員應該確定可以在所有網域中設定 Active Directory 中的權限。

如果您是從 DirSync 升級，可透過 AD DS 企業系統管理員認證來重設 DirSync 所使用帳戶的密碼。 您也需要 Azure AD 全域管理員認證。

### <a name="azure-ad-global-admin-credentials"></a>Azure AD 全域管理員認證
這些認證只能在安裝期間使用，安裝完成後則無法使用。 其會用來建立 [Azure AD 連接器帳戶](#azure-ad-service-account)，以用於將變更同步處理至 Azure AD。 帳戶也會在 Azure AD 中啟用同步處理做為一項功能。

### <a name="ad-ds-connector-account-required-permissions-for-express-settings"></a>AD DS 連接器帳戶針對快速設定所需的權限
[AD DS 連接器帳戶](#active-directory-account)是為了在 Windows Server AD 中讀取和寫入而建立的，而且在由快速設定建立時，會具有下列權限：

| 權限 | 用於 |
| --- | --- |
| <li>複寫目錄變更</li><li>複寫目錄變更 (全部) |密碼雜湊同步處理 |
| 讀取/寫入所有屬性 (使用者) |匯入和 Exchange 混合 |
| 讀取/寫入所有屬性 (iNetOrgPerson) |匯入和 Exchange 混合 |
| 讀取/寫入所有屬性 (群組) |匯入和 Exchange 混合 |
| 讀取/寫入所有屬性 (連絡人) |匯入和 Exchange 混合 |
| 重設密碼 |啟用密碼回寫的準備工作 |

### <a name="express-installation-wizard-summary"></a>快速安裝精靈摘要

![快速安裝](./media/reference-connect-accounts-permissions/express.png)

下表摘要列出快速安裝精靈頁面、所收集的認證，以及這些認證的用途。

| 精靈頁面 | 收集的認證 | 所需的權限 | 用於 |
| --- | --- | --- | --- |
| N/A |執行安裝精靈的使用者 |本機伺服器的系統管理員 |<li>建立 [ADSync 服務帳戶](#azure-ad-connect-sync-service-account)帳戶，以用來執行同步處理服務。 |
| 連接至 Azure AD |Azure AD 目錄認證 |Azure AD 中的全域管理員角色 |<li>啟用 Azure AD 目錄中的同步處理。</li>  <li>建立在 Azure AD 中用於持續同步處理作業的 [Azure AD 連接器帳戶](#azure-ad-service-account)。</li> |
| 連線到 AD DS |內部部署 Active Directory 認證 |Active Directory 中 Enterprise Admins (EA) 群組成員 |<li>在 Active Directory 中建立 [AD DS 連接器帳戶](#active-directory-account)並對其授與權限。 這個建立的帳戶是在同步處理期間用來讀取和寫入目錄資訊。</li> |


## <a name="custom-installation-settings"></a>自訂安裝設定

使用自訂設定安裝時，精靈會提供更多選擇和選項。 

### <a name="custom-installation-wizard-summary"></a>自訂安裝精靈摘要

下表摘要列出自訂安裝精靈頁面、所收集的認證，以及這些認證的用途。

![快速安裝](./media/reference-connect-accounts-permissions/customize.png)

| 精靈頁面 | 收集的認證 | 所需的權限 | 用於 |
| --- | --- | --- | --- |
| N/A |執行安裝精靈的使用者 |<li>本機伺服器的系統管理員</li><li>如果使用完整的 SQL Server，使用者必須是 SQL 中的系統管理員 (SA)</li> |預設會建立用來作為 [同步引擎服務帳戶](#azure-ad-connect-sync-service-account)的本機帳戶。 只有在管理員未指定特定帳戶時才會建立帳戶。 |
| 安裝同步處理服務，服務帳戶選項 |AD 或本機使用者帳戶認證 |使用者權限會由安裝精靈授與 |如果系統管理員指定帳戶，則此帳戶會做為同步處理服務帳戶。 |
| 連接至 Azure AD |Azure AD 目錄認證 |Azure AD 中的全域管理員角色 |<li>啟用 Azure AD 目錄中的同步處理。</li>  <li>建立在 Azure AD 中用於持續同步處理作業的 [Azure AD 連接器帳戶](#azure-ad-service-account)。</li> |
| 連接您的目錄 |各個連線至 Azure AD 之樹系的內部部署 Active Directory 認證 |權限取決於所啟用的功能，並且可在[建立 AD DS 連接器帳戶](#create-the-ad-dso-connector-account)中找到 |這個帳戶是在同步處理期間用來讀取和寫入目錄資訊。 |
| AD FS 伺服器 |如果執行精靈的使用者登入認證權限不足無法連線，精靈就會收集清單中每個伺服器的認證 |網域系統管理員 |安裝和設定 AD FS 伺服器角色。 |
| Web 應用程式 Proxy 伺服器 |如果執行精靈的使用者登入認證權限不足無法連線，精靈就會收集清單中每個伺服器的認證 |目標電腦上的本機系統管理員 |安裝和設定 WAP 伺服器角色。 |
| Proxy 信任憑證 |Federation Service 信任認證 (Proxy 用來註冊 FS 信任憑證的認證) |網域帳戶是 AD FS 伺服器的本機系統管理員 |FS-WAP 信任憑證的首次註冊。 |
| AD FS 服務帳戶頁面，「使用網域使用者帳戶選項」 |AD 使用者帳戶認證 |網域使用者 |AD 使用者帳戶所提供的認證將做為 AD FS 服務的登入帳戶。 |

### <a name="create-the-ad-ds-connector-account"></a>建立 AD DS 連接器帳戶
在 [連線您的目錄] 頁面上指定的帳戶必須存在於 Active Directory，才能進行安裝。  Azure AD Connect 1.1.524.0 版和更新版本有選項可讓 Azure AD Connect 精靈建立 **AD DS 連接器帳戶**，以用來連線至 Active Directory。  

它也必須獲得必要的權限。 安裝精靈不會驗證權限，在同步處理期間只會發現問題。

您需要的權限取決於您啟用的選用功能。 如果您有多個網域，則必須對樹系中的所有網域授與權限。 如果您未啟用任何一項功能，預設的 **網域使用者** 權限就已足夠。

| 功能 | 權限 |
| --- | --- |
| ms-DS-ConsistencyGuid 功能 |ms-DS-ConsistencyGuid 屬性 (詳情記載於[設計概念 - 使用 ms-DS-ConsistencyGuid 來作為 sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor)) 的寫入權限。 | 
| 密碼雜湊同步處理 |<li>複寫目錄變更</li>  <li>複寫目錄變更 (全部) |
| Exchange 混合式部署 |[Exchange 混合回寫](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback)中記載了使用者、群組和連絡人適用的屬性的寫入權限。 |
| Exchange 郵件公用資料夾 |公用資料夾屬性 (詳情記載於 [Exchange 郵件公用資料夾](reference-connect-sync-attributes-synchronized.md#exchange-mail-public-folder)) 的讀取權限。 | 
| 密碼回寫 |[開始使用密碼管理](../authentication/howto-sspr-writeback.md)中記載了使用者適用的屬性的寫入權限。 |
| 裝置回寫 |[裝置回寫](how-to-connect-device-writeback.md)中所述的使用 PowerShell 指令碼授與權限。 |
| 群組回寫 |讀取、建立、更新和刪除已同步處理之 **Office 365 群組**的群組物件。  如需詳細資訊，請參閱[群組回寫](how-to-connect-preview.md#group-writeback)。|

## <a name="upgrade"></a>升級
當您從一個 Azure AD Connect 版本升級到新版本時，需要下列權限：

>[!IMPORTANT]
>從組建 1.1.484 開始，Azure AD Connect 出現一個迴歸錯誤 (bug)，導致需要有 sysadmin 權限才能升級 SQL 資料庫。  組建 1.1.647 中已更正這個錯誤。  如果想要升級至這個組建，您需要 sysadmin 權限。  Dbo 權限還不夠。  如果您嘗試升級 Azure AD Connect 但沒有 sysadmin 權限，升級會失敗，然後 Azure AD Connect 將無法再正常運作。  Microsoft 知道這個問題，正在設法解決。


| 主體 | 所需的權限 | 用於 |
| --- | --- | --- |
| 執行安裝精靈的使用者 |本機伺服器的系統管理員 |更新二進位檔案。 |
| 執行安裝精靈的使用者 |ADSyncAdmins 的成員 |對同步處理規則和其他組態進行變更。 |
| 執行安裝精靈的使用者 |如果您使用同步處理引擎資料庫的完整 SQL Server: DBO (或類似選項) |變更資料庫層級，例如更新含有新資料行的資料表。 |

## <a name="more-about-the-created-accounts"></a>已建立帳戶的相關資訊
### <a name="ad-ds-connector-account"></a>AD DS 連接器帳戶
如果您使用快速設定，則會在 Active Directory 中建立用於同步處理的帳戶。 建立的帳戶位於樹系根網域的使用者容器中，並且其名稱前面會是 **MSOL_**。 系統會使用不會過期的長複雜密碼建立帳戶。 如果在網域中有密碼原則，請確定允許這個帳戶使用長密碼和複雜密碼。

![AD 帳戶](./media/reference-connect-accounts-permissions/adsyncserviceaccount.png)

如果您使用自訂設定，您就必須負責在開始安裝之前建立帳戶。  請參閱[建立 AD DS 連接器帳戶](#create-the-ad-dso-connector-account)。

### <a name="adsync-service-account"></a>ADSync 服務帳戶
同步處理服務可以在不同帳戶下執行。 它可以在**虛擬服務帳戶** (VSA)、**群組受控服務帳戶** (gMSA/sMSA) 或一般使用者帳戶下執行。 當您執行全新安裝時，2017 年 4 月版之 Connect 的支援選項已變更。 如果您從舊版的 Azure AD Connect 升級，將無法使用這些額外選項。

| 帳戶類型 | 安裝選項 | 說明 |
| --- | --- | --- |
| [虛擬服務帳戶](#virtual-service-account) | 快速和自訂，2017 年 4 月和更新版本 | 這個選項適用於所有快速安裝，但網域控制站上的安裝除外。 若為自訂安裝，除非您使用其他選項，否則這是預設選項。 |
| [群組受控服務帳戶](#group-managed-service-account) | 自訂，2017 年 4 月和更新版本 | 如果您使用遠端 SQL Server，我們建議使用群組受控服務帳戶。 |
| [使用者帳戶](#user-account) | 快速和自訂，2017 年 4 月和更新版本 | 只有在 Windows Server 2008 和網域控制站上安裝時，才會在安裝期間建立前面加上 AAD_ 的使用者帳戶。 |
| [使用者帳戶](#user-account) | 快速和自訂，2017 年 3 月和更早版本 | 在安裝期間，系統會建立本機帳戶，並於帳戶前面加上 AAD_。 而在使用自訂安裝時，您則可以指定另一個帳戶。 |

如果您使用的 Connect 所具有的組建來自 2017 年 3 月或更早版本，由於 Windows 基於安全性理由會摧毀加密金鑰，您不應該重設服務帳戶的密碼。 只有重新安裝 Azure AD Connect 才能將帳戶變更為其他任何帳戶。 如果您升級為來自 2017 年 4 月或更新版本的組建，則系統支援變更服務帳戶的密碼，但是您無法變更所使用的帳戶。

> [!Important]
> 您只能在第一次安裝時設定服務帳戶。 系統不支援在安裝完成之後才變更服務帳戶。

下列資料表列出同步處理服務帳戶的預設、建議和支援選項。

圖例：

- **粗體**代表預設選項，大部分情況下也代表建議選項。
- *斜體* 代表建議選項 (若該選項不是預設選項)。
- 2008 - 安裝在 Windows Server 2008 時的預設選項
- 非粗體 - 支援選項
- 本機帳戶 - 伺服器上的本機使用者帳戶
- 網域帳戶 - 網域使用者帳戶
- sMSA - [獨立受控服務帳戶](https://technet.microsoft.com/library/dd548356.aspx)
- gMSA - [群組受控服務帳戶](https://technet.microsoft.com/library/hh831782.aspx)

| | LocalDB</br>Express | LocalDB/LocalSQL</br>自訂 | 遠端 SQL</br>自訂 |
| --- | --- | --- | --- |
| **獨立/工作群組電腦** | 不支援 | **VSA**</br>本機帳戶 (2008)</br>本機帳戶 |  不支援 |
| **已加入網域的電腦** | **VSA**</br>本機帳戶 (2008) | **VSA**</br>本機帳戶 (2008)</br>本機帳戶</br>網域帳戶</br>sMSA、gMSA | **gMSA**</br>網域帳戶 |
| **網域控制站** | **網域帳戶** | *gMSA*</br>**網域帳戶**</br>sMSA| *gMSA*</br>**網域帳戶**|

#### <a name="virtual-service-account"></a>虛擬服務帳戶
虛擬服務帳戶是特殊的帳戶類型，這種帳戶沒有密碼，並且是由 Windows 進行管理。

![VSA](./media/reference-connect-accounts-permissions/aadsyncvsa.png)

VSA 適用於同步處理引擎和 SQL 位於相同伺服器的情況。 如果您使用遠端 SQL，我們會建議您改用[群組受控服務帳戶](#managed-service-account)。

這項功能需要 Windows Server 2008 R2 或更新版本。 如果您在 Windows Server 2008 上安裝 Azure AD Connect，則安裝會改回使用[使用者帳戶](#user-account)。

#### <a name="group-managed-service-account"></a>群組受控服務帳戶
如果您使用遠端 SQL Server，我們建議使用**群組受控服務帳戶**。 如需如何讓 Active Directory 準備好使用群組受控服務帳戶的詳細資訊，請參閱[群組受控服務帳戶概觀](https://technet.microsoft.com/library/hh831782.aspx)。

若要使用此選項，請在 [安裝必要元件](how-to-connect-install-custom.md#install-required-components) 頁面上，依序選取 [使用現有的服務帳戶] 和 [受控服務帳戶]。  
![VSA](./media/reference-connect-accounts-permissions/serviceaccount.png)  
系統也支援使用[獨立受控服務帳戶](https://technet.microsoft.com/library/dd548356.aspx)。 不過，這些帳戶只能在本機電腦上使用，所以對預設虛擬服務帳戶使用這些帳戶並沒有任何好處。

這項功能需要 Windows Server 2012 或更新版本。 如果您需要使用較舊的作業系統並使用遠端 SQL，則您必須使用[使用者帳戶](#user-account)。

#### <a name="user-account"></a>使用者帳戶
安裝精靈會建立本機服務帳戶 (除非您在自訂設定指定要使用的帳戶)。 此帳戶的前面會加上 **AAD_** 並用做實際同步處理服務的執行身分。 如果您在網域控制站上安裝 Azure AD Connect，則在網域中建立帳戶。 如果有下列情況，**AAD_** 服務帳戶必須位於網域中：
   - 您使用執行 SQL Server 的遠端伺服器
   - 您使用需要驗證的 Proxy

![同步服務帳戶](./media/reference-connect-accounts-permissions/syncserviceaccount.png)

系統會使用不會過期的長複雜密碼建立帳戶。

這個帳戶是安全儲存其他帳戶密碼的方式。 這些其他帳戶的密碼會加密儲存在資料庫中。 加密金鑰的私密金鑰是使用 Windows 資料保護 API (DPAPI) 的密碼編譯服務祕密金鑰加密來保護。

如果您使用完整的 SQL Server，那麼服務帳戶會是為同步引擎所建立的資料庫 DBO。 使用其他權限，服務將無法如預期般運作。 也會建立 SQL 登入。

此帳戶也會獲授與檔案、登錄機碼及與其他同步引擎相關的物件權限。

### <a name="azure-ad-connector-account"></a>Azure AD 連接器帳戶
系統會在 Azure AD 中建立帳戶供同步服務使用。 此帳戶可以由其顯示名稱來識別。

![AD 帳戶](./media/reference-connect-accounts-permissions/aadsyncserviceaccount2.png)

使用帳戶所在伺服器的名稱可以透過使用者名稱的第二個部分來識別。 在圖中，伺服器名稱是 DC1。 如果您有預備伺服器，則每個伺服器會有自己的帳戶。

系統會使用不會過期的長複雜密碼建立帳戶。 且該帳戶會獲得特殊角色**目錄同步處理帳戶**，其僅具有執行目錄同步處理工作的權限。 您無法透過 Azure AD Connect 精靈以外的方式授與特殊的內建角色。 Azure 入口網站會顯示此帳戶具備 [使用者] 角色。

Azure AD 中有 20 個同步服務帳戶的限制。 若要取得 Azure AD 中現有 Azure AD 服務帳戶的清單，請執行下列 Azure AD PowerShell Cmdlet：`Get-AzureADDirectoryRole | where {$_.DisplayName -eq "Directory Synchronization Accounts"} | Get-AzureADDirectoryRoleMember`

若要移除未使用的 Azure AD 服務帳戶，請執行下列 Azure AD PowerShell Cmdlet：`Remove-AzureADUser -ObjectId <ObjectId-of-the-account-you-wish-to-remove>`

## <a name="related-documentation"></a>相關文件
如果您尚未閱讀有關[整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md) 的文件，下表提供相關主題的連結。

|話題 |連結|  
| --- | --- |
|下載 Azure AD Connect | [下載 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771)|
|使用快速設定進行安裝 | [快速安裝 Azure AD Connect](how-to-connect-install-express.md)|
|使用自訂設定進行安裝 | [自訂 Azure AD Connect 安裝](./how-to-connect-install-custom.md)|
|從 DirSync 升級 | [從 Azure AD 同步作業工具 (DirSync) 升級](how-to-dirsync-upgrade-get-started.md)|
|安裝後 | [驗證安裝和指派授權 ](how-to-connect-post-installation.md)|

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
