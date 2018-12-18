---
title: Azure Multi-Factor Authentication 常見問題集 | Microsoft Docs
description: 與 Azure Multi-Factor Authentication 相關的常見問題與答案。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: b4fc67acae83573db772923a94cdbb6b1c4a301f
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159836"
---
# <a name="frequently-asked-questions-about-azure-multi-factor-authentication"></a>與 Azure Multi-Factor Authentication 相關的常見問題

此常見問題集回答關於 Azure Multi-Factor Authentication 和使用 Multi-Factor Authentication 服務的常見問題。 並將問題細分為一般服務問題、計費模式、使用者體驗和疑難排解。

## <a name="general"></a>一般

**問：Azure Multi-Factor Authentication Server 如何處理使用者資料？**

使用 Multi-Factor Authentication Server，使用者資料只會儲存在內部部署伺服器。 雲端中不會儲存任何持續性的使用者資料。 使用者執行雙步驟驗證時，Multi-Factor Authentication Server 會將資料傳送到 Azure Multi-Factor Authentication 雲端服務以供驗證。 Multi-Factor Authentication Server 與 Multi-Factor Authentication 雲端服務之間的通訊會透過傳出連接埠 443，使用安全通訊端層 (SSL) 或傳輸層安全性 (TLS) 通訊。

驗證要求傳送至雲端服務時，會收集資料以用於驗證和使用方式報告。 雙步驟驗證記錄檔中包含的資料欄位如下：

* **唯一識別碼** (使用者名稱或內部部署 Multi-Factor Authentication Server 識別碼兩者之一)
* **名字和姓氏** (選擇性)
* **電子郵件地址** (選擇性)
* **電話號碼** (進行語音通話或簡訊驗證時)
* **裝置權杖** (執行行動應用程式驗證時)
* **驗證模式**
* **驗證結果**
* **Multi-Factor Authentication Server 名稱**
* **Multi-Factor Authentication Server IP**
* **用戶端 IP** (如果有的話)

您可以在 Multi-Factor Authentication Server 中設定選擇性欄位。

驗證結果 (成功或拒絕) 和拒絕原因會與驗證資料一起儲存。 可在驗證和使用方式報告中取得此資料。

**問：何種 SMS 簡短程式碼可用來傳送 SMS 訊息給我的使用者？**

在美國，Microsoft 會使用下列 SMS 簡短程式碼：

   * 97671
   * 69829
   * 51789
   * 99399

在加拿大，Microsoft 會使用下列 SMS 簡短程式碼：

   * 759731 
   * 673801

Microsoft 不保證相同號碼會提供一致的 SMS 或以語音為基礎的多重要素驗證即時交付。 為了使用者的利益，Microsoft 可能會在調整路由來改善 SMS 傳遞能力時，隨時新增或移除簡短程式碼。 Microsoft 不支援美國和加拿大以外之國家/地區的簡短程式碼

## <a name="billing"></a>計費

大部分計費問題可以從 [Multi-Factor Authentication 價格頁面](https://azure.microsoft.com/pricing/details/multi-factor-authentication/)或有關[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)的文件中得到解答。

**問︰若為了驗證而撥打電話或傳送簡訊，我的組織是否須為此付費？**

不用，您不需為透過 Azure Multi-Factor Authentication 對使用者撥打個別通話或傳送簡訊支付費用。 如果您使用的 MFA 提供者是以驗證次數計費，則您需要為每次驗證支付費用，但無須支付使用方式的費用。

根據不同的個人電話服務，您的使用者可能需要為接聽的電話或收到的簡訊支付費用。

**問：若使用以使用者為單位的計費模式，我需要為所有啟用的使用者付費嗎？或是只須為執行雙步驟驗證的使用者付費？**

無論使用者是否在該月執行雙步驟驗證，計費是以設定要使用 Multi-Factor Authentication 的使用者數量作為基礎。

**問：Multi-Factor Authentication 是如何計費？**

當您建立每個使用者或每個授權的 MFA 提供者時，貴組織的 Azure 訂用帳戶會以使用方式每個月計費作為基礎。 此計費模式類似虛擬機器或網站使用 Azure 的計費方式。

當您購買 Azure Multi-factor Authentication 的訂用帳戶時，貴組織僅支付每位使用者的年度授權費。 MFA 授權和 Office 365、Azure AD Premium 或 Enterprise Mobility + Security 組合會以這種方式計費。 

若要深入了解您的選項，請參閱[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)。

**問：是否有免費版本的 Azure Multi-Factor Authentication？**

在某些情況下，是的。

適用於 Azure 系統管理員的 Multi-Factor Authentication 提供免費的 Azure MFA 功能子集，讓您可存取 Microsoft 線上服務，包括 Azure 和 Office 365 系統管理員入口網站。 若 Azure Active Directory 執行個體中的全域系統管理員沒有透過 MFA 授權、組合或以獨立使用作為基礎的提供者取得完整版 Azure MFA，則僅適用此供應項目。 如果您的管理員是使用免費版本，然後您購買完整版的 Azure MFA，就會自動將所有全域管理員提升為付費版本。

適用於 Office 365 使用者的 Multi-Factor Authentication 提供免費的 Azure MFA 功能子集，讓您可存取 Office 365 服務，包括 Exchange Online 和 SharePoint Online。 若使用者擁有指派的 Office 365 授權，但相對應的 Azure Active Directory 執行個體沒有透過 MFA 授權、組合或獨立使用型提供者取得完整版 Azure MFA 時，則適用此供應項目。

**問：我的組織是否能隨時在「每位使用者」與「每次驗證」耗用量計費模式之間切換？**

如果貴組織購買的 MFA 是使用量計費型獨立服務，則您可以在建立 MFA 提供者時選擇計費模式。 您無法在 MFA 提供者建立之後變更計費模型。 不過，您可以刪除 MFA 提供者，然後建立一個使用不同計費模式的 MFA 提供者。

MFA 提供者建立時，可與 Azure Active Directory (或稱為「Azure AD 租用戶」) 連結。 如果目前的 MFA 提供者已連結到 Azure AD 租用戶，您就可以安全地將 MFA 提供者刪除，並建立連結至相同 Azure AD 租用戶的 MFA 提供者。 或者，如果您購買的 MFA、Azure AD Premium 或 Enterprise Mobility + Security (EMS) 授權已足夠讓啟用 MFA 的所有使用者使用，您可以一併刪除 MFA 提供者。

如果您的 MFA 提供者未連結至 Azure AD 租用戶，或是您將新的 MFA 提供者連結至不同 Azure AD 租用戶，則使用者設定和組態選項並不會進行轉移。 此外，現有 Azure MFA 伺服器也需要使用新 MFA 提供者產生的啟用認證重新啟動。 重新啟用 MFA 伺服器以將其連結至新的 MFA 提供者，並不會影響電話和簡訊驗證，但所有使用者的行動裝置應用程式通知將會停止運作，直到他們重新啟動行動裝置應用程式。

於[開始使用 Azure Multi-Factor Auth 提供者](concept-mfa-authprovider.md)中深入了解 MFA 提供者。

**問：我的組織是否能隨時在「根據使用量計費」和「訂用帳戶」(根據授權計費模式) 之間切換？**

在某些情況下，是的。

如果您的目錄有*根據使用者計費*的 Azure Multi-Factor Authentication 提供者，您可以新增 MFA 授權。 擁有授權的使用者不會根據每位使用者使用量計費。 沒有授權的使用者仍可透過 MFA 提供者啟用 MFA。 如果您購買授權，並將其指派給所有設定使用 Multi-Factor Authentication 的使用者，您就可以刪除 Azure Multi-Factor Authentication 提供者。 如果您之後會有多於授權的使用者，您隨時都可以建立其他根據使用者計費的 MFA 提供者。

如果您的目錄有依驗證次數計費的 Azure Multi-Factor Authentication 提供者，只要您的 MFA 提供者與訂用帳戶連結，就一律要為每次驗證支付費用。 您可以將 MFA 授權指派給使用者，但您仍需要為每次雙步驟驗證要求支付費用，無論發出此要求的人是否擁有指派的 MFA 授權。

**問：我的組織是否必須使用和同步處理身分識別才能使用 Azure Multi-Factor Authentication？**

如果貴組織使用根據使用量計費的模型，則 Azure Active Directory 為選擇性項目，並非必要。 如果您的 MFA 提供者未連結至 Azure AD 租用戶，您可以只將 Multi-Factor Authentication Server 部署在內部。

授權模式需要 Azure Active Directory，因為當您購買授權並指派給目錄中的使用者時，會將授權加入 Azure AD 租用戶。

## <a name="manage-and-support-user-accounts"></a>管理和支援使用者帳戶

**問：如果我的使用者沒在手機上收到回應，或是沒帶著手機，有什麼處理方式是我該告訴他們的？**

我們希望您的所有使用者並非只設定一種驗證方式。 請告訴他們試著重新登入，但在登入頁面上選取不同驗證方法。

您可以將使用者引導至[終端使用者疑難排解指南](../user-help/multi-factor-authentication-end-user-troubleshoot.md)。

**問︰如果我的使用者無法登入他們的帳戶，我該怎麼做？**

您可以讓使用者再次進行註冊程序，以重設使用者的帳戶。 深入了解 [在雲端使用 Azure Multi-Factor Authentication 管理使用者和裝置設定](howto-mfa-userdevicesettings.md)。

**問：如果我的其中一個使用者遺失手機，而該手機上正在使用應用程式密碼，我該怎麼做？**

請刪除該使用者的所有應用程式密碼，以防止未經授權的存取。 當使用者有替代裝置後，他們可以重新建立密碼。 深入了解 [在雲端使用 Azure Multi-Factor Authentication 管理使用者和裝置設定](howto-mfa-userdevicesettings.md)。

**問：如果使用者無法登入非瀏覽器的應用程式該怎麼辦？**

如果貴組織仍然使用舊版的用戶端，但您已[允許使用應用程式密碼](howto-mfa-mfasettings.md#app-passwords)，則您的使用者無法使用他們的使用者名稱和密碼登入這些舊版用戶端。 相反地，他們需要[設定應用程式密碼](../user-help/multi-factor-authentication-end-user-app-passwords.md)。 您的使用者必須先清除 (刪除) 登入資訊、重新啟動應用程式，然後使用他們的使用者名稱和*應用程式密碼* (不是他們慣用的密碼) 登入。

如果貴組織沒有舊版用戶端，則不應該允許使用者建立應用程式密碼。

> [!NOTE]
> 適用於 Office 2013 用戶端的新式驗證
>
> 只有不支援最新驗證方式的應用程式需要應用程式密碼。 Office 2013 用戶端支援最新的驗證通訊協定，但需要設定。 新的 Office 用戶端會自動支援最新的驗證通訊協定。 如需詳細資訊，請參閱 [發表 Office 2013 新式驗證公開預覽](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。

**問︰我的使用者說他們有時會沒收到簡訊，或回覆雙向簡訊時，驗證逾時。**

因為有些無法控制的因素可能會影響服務可靠性，所以不保證會傳遞簡訊，以及在雙向簡訊中收到回覆。 這些因素包含目的地國家/地區、行動電信業者和訊號強度。

如果您的使用者時常無法收到簡訊，請告訴他們改用行動裝置應用程式或撥打電話的方式。 行動應用程式可以透過行動數據和 Wi-Fi 連接接收通知。 此外，即使裝置收不到訊號，行動應用程式可以產生驗證碼。 Microsoft 驗證器應用程式適用於 [Android](http://go.microsoft.com/fwlink/?Linkid=825072)、[IOS](http://go.microsoft.com/fwlink/?Linkid=825073)和 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)。

如果您必須使用簡訊，則建議盡可能使用單向簡訊，而不要使用雙向簡訊。 單向簡訊比較可靠，並可防止使用者因為回覆從其他國家/地區傳送的簡訊而產生全域簡訊費用。

**問︰可變更使用者必須在系統逾時前，輸入簡訊上驗證碼的時間限制嗎？**

在某些情況下是可以的。 

若為具有 Azure MFA Server v7.0 或更高版本的單向 SMS，您可以設定登錄機碼來設定逾時設定。 MFA 雲端服務傳送文字訊息之後，驗證碼 (或單次密碼) 就會傳回 MFA 伺服器。 依預設，MFA 伺服器會將程式碼儲存在記憶體中達 300 秒。 如果使用者在 300 秒內未輸入代碼，則他們的驗證會遭到拒絕。 若要變更預設逾時設定，請使用下列步驟：

1. 請前往 HKLM\Software\Wow6432Node\Positive Networks\PhoneFactor。
2. 建立名為 **pfsvc_pendingSmsTimeoutSeconds** 的DWORD 登錄機碼，並設定您希望 Azure MFA Server 儲存單次密碼的時間 (以秒為單位)。

>[!TIP] 
>如果您有多個 MFA 伺服器，只有處理原始驗證要求的 MFA 伺服器會知道已傳送給使用者的驗證碼。 當使用者輸入程式碼時，用來進行驗證的驗證要求必須傳送到相同的伺服器。 如果程式碼驗證傳送到不同的伺服器，驗證會遭到拒絕。 

若為使用 Azure MFA Server 的雙向簡訊，您可以在 MFA 管理入口網站中設定逾時設定。 如果使用者未在定義的逾時期限內回應簡訊，其驗證就會遭到拒絕。 

若為在雲端中使用 Azure MFA 的單向簡訊 (包括 AD FS 配接器或網路原則伺服器延伸模組)，您就無法設定逾時設定。 Azure AD 會儲存驗證程式碼達 180 秒。 

**問：我是否可以搭配 Azure Multi-Factor Authentication Server 使用硬體權杖？**

如果您正在使用 Azure Multi-Factor Authentication Server，則可以匯入協力廠商的 Open Authentication (OATH) 一次性限時密碼 (TOTP) 權杖，然後將它們用於雙步驟驗證。

如果您將秘密金鑰放在 CSV 檔案中，並匯入至 Azure Multi-Factor Authentication Server，則可以使用屬於 OATH TOTP 權杖的 ActiveIdentity 權杖。 只要用戶端系統可接受使用者輸入，您就可以搭配下列服務來使用 OATH 權杖：Active Directory Federation Services (ADFS)、Internet Information Server (IIS) 表單型驗證，以及遠端驗證撥號使用者服務 (RADIUS)。

您可以匯入下列格式的第三方 OATH TOTP 權杖︰  

- 可攜式對稱金鑰容器 (PSKC)  
- 如果檔案包含序號、Base 32 格式的秘密金鑰和時間間隔，則為 CSV  

**問：我是否可以使用 Azure Multi-Factor Authentication Server 保護「終端機服務」的安全？**

可以，但如果您使用的是 Windows Server 2012 R2 或更新版本，則只能使用遠端桌面閘道 (RD 閘道) 保護「終端機服務」。

Windows Server 2012 R2 中的安全性變更已改變 Azure Multi-Factor Authentication Server 在 Windows Server 2012 及舊版本中，對本機安全性授權 (LSA) 安全性套件的連接方式。 對於 Windows Server 2012 或較舊版本中的終端機服務版本，您可以[使用 Windows 驗證來保護應用程式](howto-mfaserver-windows.md#to-secure-an-application-with-windows-authentication-use-the-following-procedure)。 如果您是使用 Windows Server 2012 R2，則需要 RD 閘道。

**問：我已在 MFA Server 中設定顯示來電號碼，但我的使用者仍接到使用匿名來電的 Multi-Factor Authentication。**

當透過公共電話網路發出 Multi-Factor Authentication 通話時，有時會透過不支援來電顯示的電信業者路由傳送。 因此，即使 Multi-Factor Authentication 系統一律會傳送來電顯示號碼，但並不保證一定會顯示來電號碼。

**問：為什麼我的使用者收到須註冊安全性資訊的提示？**
使用者收到須註冊安全性資訊的原因有下列幾項︰

- 使用者已透過他們 Azure AD 中的系統管理者啟用 MFA，但尚未在他們的帳戶中註冊安全性資訊。
- 使用者已在 Azure AD 中啟用自助式密碼重設。 如果他們之後忘記密碼，安全性資訊將會協助他們重設密碼。
- 使用者存取具有條件式存取原則的應用程式來要求 MFA，但先前未註冊過 MFA。
- 使用者正在透過 Azure AD (包括 Azure AD Join) 註冊裝置，而貴組織要求使用 MFA 進行裝置註冊，但使用者先前未註冊過 MFA。
- 使用者正在 Windows 10 上註冊 Windows Hello 企業版 (需使用 MFA)，但先前未註冊過 MFA。
- 組織建立已套用至使用者的 MFA 註冊原則並加以啟用。
- 使用者先前已註冊過 MFA，但選擇的驗證方法已由系統管理員停用。 使用者必須因此再次進行 MFA 註冊，以選取新的預設驗證方法。

## <a name="errors"></a>Errors

**問：如果使用者在使用行動應用程式通知時看到「驗證要求不適用於已啟用的帳戶」錯誤訊息，應該怎麼辦？**

請告訴他們遵循這個從行動應用程式移除其帳戶的程序，然後重新新增帳戶︰

1. 移至 [您的 Azure 入口網站設定檔](https://account.activedirectory.windowsazure.com/profile/) ，並使用您的組織帳戶登入。
2. 選取 [其他安全性驗證]。
3. 將現有帳戶從行動應用程式移除。
4. 按一下 [設定] ，然後依照指示來重新設定行動應用程式。

**問：如果使用者在登入非瀏覽器應用程式時看到 0x800434D4L 錯誤訊息，應該怎麼辦？**

當您嘗試登入非瀏覽器應用程式 (安裝於本機電腦) 時，發生 0x800434D4L 錯誤，無法使用需要雙步驟驗證的帳戶。

此種錯誤的因應措施是使用不同的使用者帳戶，進行系統管理和非系統管理作業。 您稍後可以連結系統管理帳戶與非系統管理帳戶之間的信箱，以便使用非系統管理帳戶登入 Outlook。 如需這個解決方案的詳細資訊，請了解如何[讓系統管理員能夠開啟及檢視使用者信箱的內容](http://help.outlook.com/141/gg709759.aspx?sl=1)。

## <a name="next-steps"></a>後續步驟

如果這裡沒有您問題的解答，請將它留在頁面底部的註解中。 或者，以下是取得協助的一些其他選項︰

* 在 [Microsoft 支援知識庫](https://www.microsoft.com/en-us/Search/result.aspx?form=mssupport&q=phonefactor&form=mssupport)搜尋常見技術問題的解決方案。
* 在 [Azure Active Directory 論壇](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)中，搜尋及瀏覽來自社群的技術問題與解答，或是提出您的問題。
* 如果您是舊版 PhoneFactor 的客戶且有問題或需要協助重設密碼，請使用 [重設密碼](mailto:phonefactorsupport@microsoft.com) 連結來建立支援案例。
* 請透過 [Azure Multi-Factor Authentication Server (PhoneFactor) 支援](https://support.microsoft.com/oas/default.aspx?prid=14947)連絡支援專業人員。 連絡我們時，請盡量包含有關問題的最多資訊，這樣會十分有幫助。 您可以提供的資訊包含您看到錯誤的頁面、特定錯誤碼、特定工作階段識別碼，以及看到錯誤的使用者的識別碼。
