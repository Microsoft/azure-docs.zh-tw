---
title: Azure Active Directory Connect 常見問題集 - | Microsoft Docs
description: 本文將回答有關 Azure AD Connect 的常見問題。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 4e47a087-ebcd-4b63-9574-0c31907a39a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/05/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 1fb19c7b74d059d83cc5a51a158dcc6ba9bede23
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306267"
---
# <a name="azure-active-directory-connect-faq"></a>Azure Active Directory Connect 常見問題集

## <a name="general-installation"></a>一般安裝
**問︰如果 Azure Active Directory (Azure AD) 全域系統管理員已啟用雙因素驗證 (2FA)，安裝可正常運作嗎？**  
從 2016 年 2 月的組建開始，系統已可支援此情況。

**問：是否有方法可自動安裝 Azure AD Connect？**  
您只可使用安裝精靈來進行 Azure AD Connect 安裝。 不支援自動和無訊息安裝。

**問：我有一個樹系，但無法連線到其中的網域。如何安裝 Azure AD Connect？**  
從 2016 年 2 月的組建開始，系統已可支援此情況。

**問：Azure Active Directory Domain Services (Azure AD DS) 的健康情況代理程式可在伺服器核心上運作嗎？**  
是。 安裝代理程式之後，您可以使用下列 PowerShell Cmdlet 來完成註冊程序︰ 

`Register-AzureADConnectHealthADDSAgent -Credentials $cred`

**問：Azure AD Connect 是否支援從兩個網域同步至 Azure AD？**  
是，支援這種情況。 請參閱[多個網域](how-to-connect-install-multiple-domains.md)。
 
**問：Azure AD Connect 中的相同 Active Directory 網域可以有多個連接器嗎？**  
不行，相同 AD 網域不能有多個連接器。 

**問：可以將 Azure AD Connect 資料庫移從本機資料庫至遠端 SQL Server 執行個體嗎？**   
是，下列步驟將提供如何執行這項操作的一般指引。 我們目前正在製作更詳細的文件。
1. 備份 LocalDB ADSync 資料庫。
執行此操作最簡單的方式是使用與 Azure AD Connect 安裝在同一部機器上的 SQL Server Management Studio。 連線至 (localdb)\.\ADSync，然後備份 ADSync 資料庫。

2. 將 ADSync 資料庫還原至您的遠端 SQL Server 執行個體。

3. 針對現有的[遠端 SQL 資料庫](how-to-connect-install-existing-database.md)安裝 Azure AD Connect。
   本文將示範如何遷移至使用本機 SQL 資料庫。 如果您要遷移至使用遠端 SQL 資料庫，則在此程序的步驟 5 中，您也必須輸入現有服務帳戶作為 Windows 同步服務執行時的依據。 此同步引擎服務帳戶的描述如下：
   
      **使用現有服務帳戶**：Azure AD Connect 預設會使用虛擬服務帳戶，以供同步處理服務使用。 如果您是使用遠端 SQL Server 執行個體或需要驗證的 Proxy，請使用受控服務帳戶，或使用網域中知道密碼的服務帳戶。 在這類情況下，請輸入要使用的帳戶。 請確定執行安裝的使用者是 SQL 中的系統管理員，以便建立服務帳戶的登入認證。 如需詳細資訊，請參閱 [Azure AD Connect 帳戶與權限](reference-connect-accounts-permissions.md#adsync-service-account)。 
   
      使用最新的組建，SQL 管理員即可執行頻外資料庫佈建，然後由具有資料庫擁有者權限的 Azure AD Connect 管理員進行安裝。 如需詳細資訊，請參閱[使用 SQL 委派的管理員權限安裝 Azure AD Connect](how-to-connect-install-sql-delegation.md)。

為了簡單起見，我們建議安裝 Azure AD Connect 的使用者就是 SQL 中的系統管理員。 不過，在最新的組建中，您現在已可使用委派的 SQL 系統管理員，如[使用 SQL 委派管理員權限安裝 Azure AD Connect](how-to-connect-install-sql-delegation.md)中所述。

## <a name="network"></a>網路
**問：我的防火牆、網路裝置或其他軟硬體會限制在網路上開啟連線的時間。當我使用 Azure AD Connect 時，用戶端逾時閥值的時間應該多長？**  
所有網路軟體、實體裝置或其他軟硬體限制連線開啟時間上限的閥值應該至少為 5 分鐘 (300 秒)，以便讓安裝 Azure AD Connect 用戶端的伺服器與 Azure Active Directory 連線。 此建議也適用於所有先前發行的 Microsoft 身分識別同步處理工具。

**問：是否支援單一標籤網域 (SLD)？**  
雖然非常不建議您使用這種網路設定 ([請參閱文章](https://support.microsoft.com/help/2269810/microsoft-support-for-single-label-domains))，但只要單一層級網域的網路設定可正常運作，便支援搭配使用 Azure AD Connect 同步處理與單一標籤網域。

**問：是否支援使用斷續 AD 網域的樹系？**  
否，Azure AD Connect 不支援包含斷續命名空間的內部部署樹系。

**問：是否支援「有句點」的 NetBIOS 名稱？**  
否，Azure AD Connect 不支援 NetBIOS 名稱包含句點 (.) 的內部部署樹系或/網域。

**問：是否支援純 IPv6 環境？**  
否，Azure AD Connect 不支援純 IPv6 環境。

## <a name="federation"></a>同盟
**問：如果我收到一封電子郵件，要求我更新我的 Office 365 憑證，該怎麼辦？**  
如需有關更新憑證的指引，請參閱[更新憑證](how-to-connect-fed-o365-certs.md)。

**問：我已經針對 Office 365 信賴憑證者設定「自動更新信賴憑證者」。當我的權杖簽署憑證自動換用時，需要採取任何動作嗎？**  
請參考[更新憑證](how-to-connect-fed-o365-certs.md)一文中概述的指導方針。

## <a name="environment"></a>環境
**問：安裝 Azure AD Connect 之後，是否支援重新命名伺服器？**  
否。 變更伺服器名稱會使同步引擎無法連線到 SQL 資料庫執行個體，並且無法啟動此服務。

## <a name="identity-data"></a>身分識別資料
**問：Azure AD 中的 userPrincipalName (UPN) 屬性為什麼不符合內部部署的 UPN？**  
如需詳細資訊，請參閱這些文章：

* [Office 365、Azure 或 Intune 中的使用者名稱不符合內部部署的 UPN 或替代登入識別碼](https://support.microsoft.com/kb/2523192)
* [在您將使用者帳戶的 UPN 變更為使用不同的同盟網域後，Azure Active Directory 同步作業工具未同步處理變更](https://support.microsoft.com/kb/2669550)

您也可以將 Azure AD 設定為允許同步處理引擎更新 UPN，如 [Azure AD Connect 同步處理服務功能](how-to-connect-syncservice-features.md)中所述。

**問：是否支援將內部部署 Azure AD 群組或連絡人物件與現有的 Azure AD 群組或連絡人物件進行大致相符比對？**  
是，此大致相符比對會依據 proxyAddress。 針對未啟用郵件功能的群組，不支援大致相符比對。

**問：是否支援將現有 Azure AD 群組或連絡人物件上的 ImmutableId 屬性手動設定成與內部部署 Azure AD 群組或連絡人物件完全相符？**  
否，目前不支援將現有 Azure AD 群組或連絡人物件上的 ImmutableId 屬性手動設定成與之完全相符。

## <a name="custom-configuration"></a>自訂組態
**問：Azure AD Connect 適用的 PowerShell Cmdlet 記載於何處？**  
除了記載於本網站上的 Cmdlet，在 Azure AD Connect 中找到的其他 PowerShell Cmdlet 不支援客戶使用。

**問：我是否可以使用在 Synchronization Service Manager 中找到的「伺服器匯出/伺服器匯入」選項，在伺服器之間移動組態？**  
否。 此選項不會擷取所有組態設定，因此不應使用。 應改用精靈在第二部伺服器上建立基底組態，並使用同步處理規則編輯器產生 PowerShell 指令碼，以在伺服器之間移動任何自訂規則。 如需詳細資訊，請參閱[變換移轉](how-to-upgrade-previous-version.md#swing-migration)。

**問︰是否可針對 Azure 登入頁面進行密碼快取，而且是否可以因為此快取包含自動完成 ="false" 屬性的密碼輸入元素而避免此快取？**  
目前不支援修改**密碼**欄位的 HTML 屬性，包括自動完成標記。 我們目前正在開發適用於自訂 JavaScript 的功能，可讓您將任何屬性新增至 [密碼] 欄位。

**問︰Azure 登入頁面會顯示先前登入成功的使用者名稱。可以關閉此行為嗎？**  
目前不支援修改**密碼**輸入欄位的 HTML 屬性，包括自動完成標記。 我們目前正在開發適用於自訂 JavaScript 的功能，可讓您將任何屬性新增至 [密碼] 欄位。

**問︰是否辦法避免並行的工作階段？**  
否。

## <a name="auto-upgrade"></a>自動升級

**問：使用自動升級的優點和後果為何？**  
我們建議所有客戶針對其 Azure AD Connect 安裝啟用自動升級。 優點是您一律會收到最新修補程式，包括在 Azure AD Connect 中發現的弱點安全性更新。 升級程序輕鬆無比，而且在有新版本可用時就會自動執行。 有數千個 Azure AD Connect 客戶會透過自動升級來使用每個新版本。

自動升級程序一律會先確定安裝是否符合自動升級資格。 若符合資格，就會執行和測試升級。 此程序也包含尋找規則的自訂變更和特定環境因素。 如果測試顯示升級不成功，則會自動還原到先前的版本。

根據環境的大小，此程序可能需要幾個小時。 正在進行升級時，不會發生任何 Windows Server Active Directory 與 Azure AD 之間的同步處理。

**問：我收到一封電子郵件告訴我，我的自動升級不再有效，我需要安裝新的版本。我為何需要這麼做？**  
在某些情況下，我們去年所發行的 Azure AD Connect 版本，可能已停用您的伺服器上的自動升級功能。 我們已在 Azure AD Connect 1.1.750.0 版中修正此問題。 如果您已受到此問題的影響，您可以藉由執行 PowerShell 指令碼進行修復，或以手動方式升級至最新版 Azure AD Connect 來解決此問題。 

若要執行 PowerShell 指令碼，請[下載指令碼](https://aka.ms/repairaadconnect)，並在系統管理 PowerShell 視窗中的 Azure AD Connect 伺服器上執行指令碼。 若要了解如何執行指令碼，[請檢視此短片](https://aka.ms/repairaadcau)。

若要手動升級，您必須下載並執行最新版的 AADConnect.msi 檔案。
 
-  如果您目前的版本早於 1.1.750.0，[請下載最新版本並進行升級](https://www.microsoft.com/download/details.aspx?id=47594)。
- 如果您的 Azure AD Connect 版本是 1.1.750.0 或更新版本，則不需要執行進一步的動作。 您已使用包含自動升級修正程式的版本。 

**問：我收到一封電子郵件通知我升級至最新版本，以重新啟用自動升級。我目前使用 1.1.654.0 版。是否需要升級？**  
是，您要升級至 1.1.750.0 版或更新版本，以重新啟用自動升級。 [下載最新版本並進行升級](https://www.microsoft.com/download/details.aspx?id=47594)。

**問：我收到一封電子郵件通知我升級至最新版本，以重新啟用自動升級。如果我已使用 PowerShell 來啟用自動升級，我還需要安裝最新版本嗎？**  
是，您仍需要升級至 1.1.750.0 版或更新版本。 使用 PowerShell 啟用自動升級服務，並不能減輕在 1.1.750.0 之前版本中找到的自動升級問題。

**問：我想要升級至較新版本，但我不確定當初是誰安裝 Azure AD Connect，而且沒有使用者名稱和密碼。我需要此資訊嗎？**
您不需要知道原先用來升級 Azure AD Connect 的使用者名稱和密碼。 使用具有全域管理員角色的任何 Azure AD 帳戶即可。

**問：如何找出我目前使用的 Azure AD Connect 版本？**  
若要確認您伺服器上所安裝的 Azure AD Connect 是哪個版本，請移至 [控制台] 並選取 [程式] > [程式和功能]，來查看所安裝的 Microsoft Azure AD Connect 版本，如下所示：

![控制台中的 Azure AD Connect 版本](./media/reference-connect-faq/faq1.png)

**問：如何升級至最新版的 Azure AD Connect？**  
若要了解如何升級至最新版本，請參閱 [Azure AD Connect：從舊版升級到最新版本](how-to-upgrade-previous-version.md)。 

**問：我們去年已升級為最新版的 Azure AD Connect。我們需要再次升級嗎？**  
Azure AD Connect 小組會對此服務提出頻繁的更新。 若要獲取錯誤修正程式、安全性更新及新功能，請務必讓您的伺服器使用最新版本以保持最新狀態。 如果您啟用自動升級，您的軟體版本就會自動更新。 若要尋找 Azure AD Connect 的版本發行歷程記錄，請參閱[Azure AD Connect：版本發行歷程記錄](reference-connect-version-history.md)。

**問：執行升級需要多久時間，而且對我的使用者有何影響？**  
升級所需的時間需視您的租用戶大小而定。 對於較大的組織，可能適合在在晚上或週末執行升級。 升級期間不會發生任何同步處理活動。

**問：我認為我已升級到 Azure AD Connect，但是在 Office 入口網站中，它仍然提及 DirSync。這是為什麼？**  
Office 小組正在處理 Office 入口網站更新，以反映目前的產品名稱。 它不會反映您所使用的同步處理工具。

**問：我的自動升級狀態顯示「擱置」。它為何會擱置？應該加以啟用嗎？**  
在某些情況下，先前版本中引入的錯誤會讓自動升級狀態設定為「擱置」。 以手動方式啟用在技術上是可行的，但需要幾個複雜的步驟。 最佳做法是下載並安裝最新版的 Azure AD Connect。

**問：我的公司有嚴格的變更管理需求，我想要控制其推出的時間。我可以控制何時啟動自動升級嗎？**  
不行，目前尚未有這類功能。 我們正在針對未來版本評估此功能。

**問：如果自動升級失敗，我會收到電子郵件嗎？如何知道它是否成功？**  
您不會收到升級結果的通知。 我們正在針對未來版本評估此功能。

**問：您是否會發佈有關何時打算推出自動升級的時間表？**  
在較新版本的發行程序中，自動升級是第一個步驟。 每當有新版本時，升級就會自動推出。 [Azure AD 藍圖](../fundamentals/whats-new.md)會預先宣布較新的 Azure AD connect 版本。

**問：自動升級也會升級 AAD Connect Health 嗎？**  
是，自動升級也會升級 Azure AD Connect Health。

**問：您也是在預備模式中自動升級 Azure AD Connect 伺服器嗎？**  
是，您可以自動升級處於預備模式的 Azure AD Connect 伺服器。

**問：如果自動升級失敗，而我的 Azure AD Connect 伺服器不會啟動，該怎麼辦？**  
在少數情況下，Azure AD Connect 服務不會在執行升級之後啟動。 在這些情況下，請重新啟動伺服器，這樣通常會修正此問題。 如果 Azure AD Connect 服務還是不會啟動，請開啟支援票證。 如需詳細資訊，請參閱[ 建立服務要求以連絡 Office 365 支援人員](https://blogs.technet.microsoft.com/praveenkumar/2013/07/17/how-to-create-service-requests-to-contact-office-365-support/)。 

**問：我不確定升級至較新的 Azure AD Connect 版本時有何風險。您可以打電話給我來協助我升級嗎？**  
如果您需要升級至較新 Azure AD Connect 版本的協助，請在[建立服務要求以連絡 Office 365 支援人員](https://blogs.technet.microsoft.com/praveenkumar/2013/07/17/how-to-create-service-requests-to-contact-office-365-support/)上開啟支援票證。

## <a name="troubleshooting"></a>疑難排解
**問：如何取得 Azure AD Connect 的說明？**

[搜尋 Microsoft 知識庫 (KB)](https://www.microsoft.com/en-us/search/result.aspx?q=azure+active+directory+connect)

* 針對 Azure AD Connect 支援，在 KB 中搜尋常見協助修正問題的技術解決方案。

[Azure Active Directory 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=WindowsAzureAD)

* 您可以前往 [Azure AD 社群](https://social.msdn.microsoft.com/Forums/azure/en-US/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required) 來搜尋技術問題與解答，或詢問您自己的問題。

[取得 Azure AD 支援](https://docs.microsoft.com/azure/active-directory/active-directory-troubleshooting-support-howto)
