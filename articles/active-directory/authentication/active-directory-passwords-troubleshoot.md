---
title: 自助式密碼重設疑難排解 - Azure Active Directory
description: 針對 Azure AD 自助式密碼重設進行疑難排解
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 057af5e0e5b467ab60e8de7534e9f4428b96c3dc
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46298314"
---
# <a name="troubleshoot-self-service-password-reset"></a>針對自助式密碼重設進行疑難排解

您有 Azure Active Directory (Azure AD) 自助式密碼重設 (SSPR) 方面的問題嗎？ 後續資訊可協助您重新運作項目。

## <a name="troubleshoot-self-service-password-reset-errors-that-a-user-might-see"></a>針對使用者可能會看到的自助式密碼重設錯誤進行疑難排解

| Error | 詳細資料 | 技術詳細資訊 |
| --- | --- | --- |
| TenantSSPRFlagDisabled = 9 | 很抱歉，因為您的系統管理員已為貴組織停用密碼重設，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們啟用此功能。 若要深入了解，請參閱[忘記 Azure AD 密碼的說明](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-update-your-own-password#common-problems-and-their-solutions)。 | SSPR_0009：我們已偵測到您的系統管理員尚未啟用「密碼重設」。 請連絡您的系統管理員，並要求他們為組織啟用「密碼重設」。 |
| WritebackNotEnabled = 10 |很抱歉，因為您的系統管理員尚未為貴組織啟用必要的服務，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們檢查貴組織的設定。 若要深入了解必要的服務，請參閱[設定密碼回寫](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-writeback#configure-password-writeback)。 | SSPR_0010：我們已偵測到「密碼回寫」尚未啟用。 請連絡您的系統管理員，並要求他們啟用「密碼回寫」。 |
| SsprNotEnabledInUserPolicy = 11 | 很抱歉，因為您的系統管理員尚未為貴組織設定密碼重設，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們設定密碼重設。 若要深入了解密碼重設設定，請參閱[快速入門：Azure AD 自助式密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started)。 | SSPR_0011：您的組織尚未定義密碼重設原則。 請連絡您的系統管理員，並要求他們定義密碼重設原則。 |
| UserNotLicensed = 12 | 很抱歉，因為缺少貴組織所需要的授權，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們檢查授權指派。 若要深入了解有關授權的詳細資訊，請參閱 [Azure AD 自助式密碼重設的授權需求](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-licensing)。 | SSPR_0012：您的組織沒有執行密碼重設所需的必要授權。 請連絡您的系統管理員，並要求他們檢閱授權指派。 |
| UserNotMemberOfScopedAccessGroup = 13 | 很抱歉，因為您的系統管理員尚未設定您的帳戶以使用密碼重設，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們設定您的帳戶以供密碼重設使用。 若要深入了解密碼重設的帳戶設定，請參閱[為使用者推出密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-best-practices)。 | SSPR_0012：您不是已啟用密碼重設之群組的成員。 請連絡您的系統管理員，並要求加入群組。 |
| UserNotProperlyConfigured = 14 | 很抱歉，因為缺少您帳戶所需要的資訊，此時您無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們為您重設密碼。 當您再次可以存取您的帳戶之後，必須註冊所需的資訊。 若要註冊資訊，請遵循[註冊自助式密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-reset-register)一文。 | SSPR_0014：重設密碼所需要的其他安全性資訊。 若要繼續進行，請連絡您的系統管理員，並要求他們重設您的密碼。 當您可以存取您的帳戶之後，可在 https://aka.ms/ssprsetup 註冊其他安全性資訊。 您的系統管理員可以遵循[設定與閱讀密碼重設的驗證資料](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-data#set-and-read-authentication-data-using-powershell)中的步驟，將其他安全性資訊新增至您的帳戶中。 |
| OnPremisesAdminActionRequired = 29 | 很抱歉，因為貴組織的密碼重設設定發生問題，此時無法重設密碼。 您無法採取進一步的動作來解決這種情況。 請連絡您的系統管理員，並要求他們調查。 若要深入了解潛在問題，請參閱[疑難排解密碼回寫](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-troubleshoot#troubleshoot-password-writeback)。 | SSPR_0029：因為您的內部部署設定發生錯誤，我們無法重設您的密碼。 請連絡您的系統管理員，並要求他們調查。 |
| OnPremisesConnectivityError = 30 | 很抱歉，因為貴組織的連線發生問題，此時我們無法重設密碼。 現在無需採取任何動作，但如果您稍後再試，可能會解決問題。 如果問題持續發生，請連絡您的系統管理員，並要求他們調查。 若要深入了解連線問題，請參閱[針對密碼回寫連線問題進行疑難排解](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-troubleshoot#troubleshoot-password-writeback-connectivity)。 | SSPR_0030：因為您內部部署環境的連線不佳，我們無法重設您的密碼。 請連絡您的系統管理員，並要求他們調查。|


## <a name="troubleshoot-the-password-reset-configuration-in-the-azure-portal"></a>在 Azure 入口網站中，針對密碼重設設定進行疑難排解

| Error | 解決方法 |
| --- | --- |
| 我在 Azure 入口網站中的 Azure AD 之下看不到 [密碼重設] 區段。 | 如果您未將 Azure AD Premium 或 Basic 授權指派給執行此作業的系統管理員，就會發生這種情況。 <br> <br> 將授權指派給所提及的系統管理員帳戶。 您可以遵循[指派、驗證授權及解決其問題](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses)一文中的步驟。|
| 我看不到特定的設定選項。 | 許多 UI 元素只會在需要時出現。 如果您想要看到這些元素，請嘗試啟用所有選項。 |
| 我看不到 [內部部署整合] 索引標籤。 | 只有在您下載 Azure AD Connect 並設定密碼回寫後，這個選項才會出現。 如需詳細資訊，請參閱[利用快速設定開始使用 Azure AD Connect](../hybrid/how-to-connect-install-express.md)。 |

## <a name="troubleshoot-password-reset-reporting"></a>針對密碼重設報告進行疑難排解

| Error | 解決方法 |
| --- | --- |
| 我在 [自助式密碼管理] 稽核事件類別中看不到任何密碼管理活動類型。 | 如果您未將 Azure AD Premium 或 Basic 授權指派給執行此作業的系統管理員，就會發生這種情況。 <br> <br> 您可以將授權指派給所提及的系統管理員帳戶，以解決這個問題。 如需詳細資訊，請參閱[指派、驗證授權及解決其問題](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses)。 |
| 使用者註冊顯示多個時間。 | 目前當使用者註冊時，我們會將所註冊的每一筆個別資料記錄為個別事件。 <br> <br> 如果您需要彙總此資料並以更有彈性的方式檢視，可以下載報告，並在 Excel 中以樞紐分析表開啟資料。

## <a name="troubleshoot-the-password-reset-registration-portal"></a>疑難排解密碼重設註冊入口網站

| Error | 解決方法 |
| --- | --- |
| 目錄未啟用密碼重設功能。 **您的系統管理員尚未為您啟用這項功能。** | 將 [已啟用自助式密碼重設] 旗標切換為 [選取項目] 或 [全部]，然後選取 [儲存]。 |
| 使用者尚未獲得 Azure AD Premium 或 Basic 授權。 **您的系統管理員尚未為您啟用這項功能。** | 如果您未將 Azure AD Premium 或 Basic 授權指派給執行此作業的系統管理員，就會發生這種情況。 <br> <br> 您可以將授權指派給所提及的系統管理員帳戶，以解決這個問題。 如需詳細資訊，請參閱[指派、驗證授權及解決其問題](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses)。|
| 處理要求時發生錯誤。 | 許多問題都會造成此情形，但這個錯誤通常是因為服務中斷或設定問題所導致。 如果您看到這個錯誤，而且它會影響您的業務，請連絡 Microsoft 支援服務人員以尋求其他協助。 |

## <a name="troubleshoot-the-password-reset-portal"></a>疑難排解密碼重設入口網站

| Error | 解決方法 |
| --- | --- |
| 目錄未啟用密碼重設功能。 | 將 [已啟用自助式密碼重設] 旗標切換為 [選取項目] 或 [全部]，然後選取 [儲存]。 |
| 使用者尚未獲得 Azure AD Premium 或 Basic 授權。 | 如果您未將 Azure AD Premium 或 Basic 授權指派給執行此作業的系統管理員，就會發生這種情況。 <br> <br> 如果您將授權指派給所提及的系統管理員帳戶，就可以解決這個問題。 如需詳細資訊，請參閱[指派、驗證授權及解決其問題](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses)。 |
| 目錄已啟用密碼重設，但使用者的驗證資訊遺失或格式不正確。 | 在繼續之前，請確定使用者已在目錄中登記格式正確的連絡資料。 如需詳細資訊，請參閱 [Azure AD 自助式密碼重設所用的資料](howto-sspr-authenticationdata.md)。 |
| 目錄已啟用密碼重設，原則設定為需要兩個驗證方法，但使用者只登記一個連絡資料。 | 在繼續之前，請確定使用者有至少兩個已正確設定的連絡方法。 例如，擁有行動電話號碼和辦公室電話號碼。 |
| 目錄已啟用密碼重設，並已正確設定使用者，但無法連絡到使用者。 | 原因可能是暫時性的服務錯誤，或如果連絡資料不正確，而無法正確偵測到。 <br> <br> 如果使用者等候 10 秒，就會出現「再試一次」和「連絡您的系統管理員」連結。 如果使用者選取「再試一次」，就會重試該呼叫。 如果使用者選取「連絡您的系統管理員」，就會傳送表單電子郵件給其全域系統管理員，要求對該使用者帳戶執行密碼重設。 |
| 使用者從未收到密碼重設 SMS 或來電。 | 原因可能是目錄中的電話號碼格式不正確。 請確定電話號碼的格式是 “+ccc xxxyyyzzzzXeeee”。 <br> <br> 密碼重設並不支援分機號碼，即使您在目錄中指定也是一樣。 在分派呼叫之前，會移除分機號碼。 使用沒有分機號碼的電話號碼，或將分機號碼整合至專用交換機 (PBX) 中的電話號碼。 |
| 使用者從未收到密碼重設電子郵件。 | 這個問題最常見的原因，是垃圾郵件篩選器拒絕了郵件。 請檢查您的垃圾郵件或刪除的郵件資料夾中是否有該電子郵件。 <br> <br> 同時確定您檢查的是正確電子郵件帳戶中的訊息。 |
| 我已設定密碼重設原則，但是當系統管理員帳戶使用密碼重設時，卻未套用該原則。 | Microsoft 會管理及控制系統管理員的密碼重設原則，以確保最高層級的安全性。 |
| 禁止使用者在一天當中嘗試重設密碼太多次。 | 我們實作了自動節流機制來避免使用者在短時間內嘗試重設密碼太多次。 節流發生時機： <br><ul><li>使用者嘗試在一個小時內驗證電話號碼 5 次。</li><li>使用者在一個小時內嘗試使用安全性問題關卡五次。</li><li>使用者在一個小時內嘗試重設相同使用者帳戶的密碼五次。</li></ul>若要修正這個問題，請指示使用者在最後一次嘗試之後等待 24 小時。 使用者接著可以重設其密碼。 |
| 使用者在驗證其電話號碼時看到錯誤。 | 輸入的電話號碼與登記的電話號碼不符時，就會發生這個錯誤。 請確定使用者在嘗試使用電話式方法來重設密碼時，輸入了完整的電話號碼，包括區碼和國碼。 |
| 處理要求時發生錯誤。 | 許多問題都會造成此情形，但這個錯誤通常是因為服務中斷或設定問題所導致。 如果您看到這個錯誤，而且它會影響您的業務，請連絡 Microsoft 支援服務人員以尋求其他協助。 |
| 內部部署原則違規 | 密碼不符合內部部署的 Active Directory 密碼原則。 |
| 密碼未遵循模糊原則 | 使用的密碼存在於禁止的密碼清單中，不得使用。 |

## <a name="troubleshoot-password-writeback"></a>針對密碼回寫進行疑難排解

| Error | 解決方法 |
| --- | --- |
| 無法內部部署啟動密碼重設服務。 Azure AD Connect 電腦的應用程式事件記錄中出現錯誤 6800。 <br> <br> 上架之後，同盟、傳遞驗證或密碼雜湊同步使用者無法重設其密碼。 | 啟用「密碼回寫」時，同步處理引擎會透過與雲端上線服務通訊，呼叫回寫程式庫來執行設定 (上線)。 在上線期間或啟動 Windows Communication Foundation (WCF) 端點來進行「密碼回寫」時，如果發生任何錯誤，都會導致 Azure AD Connect 電腦的事件記錄中出現錯誤。 <br> <br> 在 Azure AD Sync (ADSync) 服務重新啟動期間，若已設定回寫，就會啟動 WCF 端點。 不過，如果端點啟動失敗，我們會記錄事件 6800，並讓同步處理服務啟動。 出現此事件即表示「密碼回寫」端點並未啟動。 此事件 (6800) 的事件記錄詳細資料，以及 PasswordResetService 元件所產生的事件記錄項目會指出為何無法啟動端點。 如果「密碼回寫」仍然無法運作，請檢閱這些事件記錄錯誤，然後嘗試重新啟動 Azure AD Connect。 如果此問題持續發生，請嘗試先將「密碼回寫」停用再重新啟用。
| 當使用者嘗試在已啟用密碼回寫時重設密碼或解除鎖定帳戶，作業會失敗。 <br> <br> 此外，在解除鎖定作業發生之後，您會在 Azure AD Connect 事件記錄中看到事件，其中包含：「Synchronization Engine returned an error hr=800700CE, message=The filename or extension is too long」。 | 尋找 Azure AD Connect 的 Active Directory 帳戶，並將密碼重設，讓它包含不超過 127 個字元。 然後從 [開始] 功能表開啟 [同步處理服務]。 瀏覽至 [連接器]，然後尋找 [Active Directory 連接器]。 選取它，然後選取 [屬性]。 瀏覽至 [認證] 頁面，然後輸入新密碼。 選取 [確定] 以關閉頁面。 |
| 在 Azure AD Connect 安裝程序的最後一個步驟，您會看到一個錯誤，指出無法設定「密碼回寫」。 <br> <br> Azure AD Connect 應用程式事件記錄包含錯誤 32009，以及「取得授權權杖時發生錯誤」文字。 | 下列兩種情況會發生此錯誤： <br><ul><li>針對在 Azure AD Connect 安裝程序開始時所指定的全域系統管理員帳戶，指定了錯誤的密碼。</li><li>針對在 Azure AD Connect 安裝程序開始時所指定的全域系統管理員帳戶，嘗試使用同盟使用者。</li></ul> 若要修正此問題，請確定您並未針對在安裝程序開始時所指定的全域系統管理員使用同盟帳戶。 也請確定所指定的密碼正確。 |
| Azure AD Connect 電腦的事件記錄包含透過執行 PasswordResetService 所擲回的錯誤 32002。 <br> <br> 此錯誤指出：「連線到服務匯流排時發生錯誤。 權杖提供者無法提供安全性權杖。」 | 您的內部部署環境無法連線到雲端的 Azure 服務匯流排端點。 這個錯誤是因為防火牆規則封鎖連往特定連接埠或網址的輸出連線所導致。 如需詳細資訊，請參閱[連線必要條件](../hybrid/how-to-connect-install-prerequisites.md)。 在您更新這些規則之後，請重新啟動 Azure AD Connect 電腦，「密碼回寫」應該就會再次開始運作。 |
| 運作一段時間之後，同盟、傳遞驗證或密碼雜湊同步使用者無法重設其密碼。 | 在某些罕見的情況下，Azure AD Connect 重新啟動後，「密碼回寫」服務可能仍無法重新啟動。 在這些情況下，請先檢查「密碼回寫」是否在內部部署環境中顯示為已啟用。 您可以使用 Azure AD Connect 精靈或 PowerShell 來檢查 (請參閱上一節「做法」)。 如果此功能顯示為已啟用，請再次嘗試透過 UI 或 PowerShell 啟用或停用此功能。 如果這麼做沒有效，請嘗試完整解除安裝再重新安裝 Azure AD Connect。 |
| 同盟、傳遞驗證或密碼雜湊同步使用者若嘗試重設其密碼，會在嘗試提交密碼後看到錯誤。 此錯誤表示發生服務問題。 <br ><br> 除了這個問題之外，在密碼重設作業期間，您可能會在內部部署的事件記錄中看到關於管理代理程式存取遭拒的錯誤。 | 如果您在事件記錄中看到這些錯誤，請確認在設定時於精靈中指定的 Active Directory 管理代理程式 (ADMA) 帳戶具有密碼回寫的必要權限。 <br> <br> 這項權限在授與後，最多需要一小時的時間，才會透過網域控制站 (DC) 上的 `sdprop` 背景工作往下傳遞。 <br> <br> 若要讓密碼重設正常運作，必須在要重設密碼的使用者物件安全性描述元上為權限加上戳記。 在使用者物件上出現此權限之前，密碼重設會繼續因存取遭拒訊息而失敗。 |
| 同盟、傳遞驗證或密碼雜湊同步使用者若嘗試重設其密碼，會在嘗試提交密碼後看到錯誤。 此錯誤表示發生服務問題。 <br> <br> 除了這個問題之外，在密碼重設作業期間，您可能會在 Azure AD Connect 服務的事件記錄中，看到錯誤指出「找不到物件」錯誤。 | 這個錯誤通常表示同步處理引擎找不到 Azure AD 連接器空間中的使用者物件，或連結的 Metaverse (MV) 或 Azure AD 連接器空間物件。 <br> <br> 若要針對這個問題進行疑難排解，請確定使用者已確實透過 Azure AD Connect 的目前執行個體從內部部屬同步處理到 Azure AD，並檢查連接器空間和 MV 中物件的狀態。 確認 Active Directory 憑證服務 (AD CS) 物件會透過 “Microsoft.InfromADUserAccountEnabled.xxx” 規則連線到 MV 物件。|
| 同盟、傳遞驗證或密碼雜湊同步使用者若嘗試重設其密碼，會在嘗試提交密碼後看到錯誤。 此錯誤表示發生服務問題。 <br> <br> 除了這個問題之外，在密碼重設作業期間，您可能會在 Azure AD Connect 服務的事件記錄中看到錯誤，指出「找到多個相符項目」錯誤。 | 此錯誤指出同步處理引擎偵測到 MV 物件透過 “Microsoft.InfromADUserAccountEnabled.xxx” 連線到多個 AD CS 物件。 這表示使用者在多個樹系中啟用帳戶。 密碼回寫不支援此案例。 |
| 密碼作業因設定錯誤而失敗。 應用程式事件記錄中包含 Azure AD Connect 錯誤 6329 和文字：「0x8023061f (作業失敗，因為此管理代理程式上未啟用密碼同步處理)」。 | 如果在已經啟用密碼回寫功能之後，變更 Azure AD Connect 設定以新增 Active Directory 樹系 (或移除現有樹系再重新加入)，就會發生此錯誤。 位於在這些最近新增樹系中的使用者，其密碼作業會失敗。 若要修正此問題，請在完成樹系設定變更之後，將「密碼回寫」功能先停用，然後再重新啟用。 |

## <a name="password-writeback-event-log-error-codes"></a>密碼回寫事件記錄錯誤碼

針對「密碼回寫」問題進行疑難排解時，最佳做法是檢查 Azure AD Connect 電腦上的「應用程式事件記錄」。 這個事件記錄包含「密碼回寫」兩個相關來源的事件。 PasswordResetService 來源會說明與「密碼回寫」作業相關的作業和問題。 ADSync 來源會說明與 Active Directory 環境中的密碼設定相關的作業和問題。

### <a name="if-the-source-of-the-event-is-adsync"></a>如果事件來源是 ADSync

| 代碼 | 名稱或訊息 | 說明 |
| --- | --- | --- |
| 6329 | BAIL: MMS(4924) 0x80230619：「有一項限制讓密碼無法變更為目前指定的密碼。」 | 當「密碼回寫」服務嘗試在本機目錄上設定不符合密碼有效期、歷程記錄、複雜度或網域篩選需求的密碼時，就會發生此事件。 <br> <br> 如果您有最短的密碼使用期限，且最近在該時段內變更過密碼，就必須在到達網域中指定的使用期限後，才能再次變更密碼。 若要進行測試，最短使用期限應該設定為 0。 <br> <br> 如果已啟用密碼歷程記錄需求，則必須選取最近 N 次未用過的密碼，其中 N 是密碼歷程記錄設定。 如果您選取最近 N 次用過的密碼，則會在此案例中看到失敗。 若要進行測試，密碼歷程記錄應該設定為 0。 <br> <br> 如果您有密碼複雜度需求，則會在使用者嘗試變更或重設密碼時強制執行這些需求。 <br> <br> 如果您啟用密碼篩選功能，當使用者選取的密碼不符合篩選準則時，重設或變更作業就會失敗。 |
| 6329 | MMS(3040): admaexport.cpp(2837): 伺服器未包含 LDAP 密碼原則控制項。 | 如果未在 DC 上啟用 LDAP_SERVER_POLICY_HINTS_OID 控制項 (1.2.840.113556.1.4.2066)，就會發生此問題。 若要使用密碼回寫功能，您必須啟用該控制項。 若要這麼做，DC 必須是 Windows Server 2008 (含最新的 SP) 或更新版本。 如果您的 DC 是 2008 (R2 之前) 版，則也必須套用 [Hotfix KB2386717](http://support.microsoft.com/kb/2386717)。 |
| HR 8023042 | 同步處理引擎傳回的錯誤 hr= 80230402，訊息=嘗試取得物件失敗，因為存在具有相同錨點的重複項目。 | 在多個網域中啟用相同的使用者識別碼時，就會發生此錯誤。 例如，如果您在同步處理的帳戶和資源樹系中皆存在相同的使用者識別碼，且在每個樹系中皆為啟用時。 <br> <br> 如果您使用非唯一的錨點屬性 (例如別名或 UPN)，而且兩個使用者共用該相同的錨點屬性時，也會發生此錯誤。 <br> <br> 若要解決這個問題，請確定您的網域內沒有重複的使用者，且每個使用者皆使用唯一的錨點屬性。 |

### <a name="if-the-source-of-the-event-is-passwordresetservice"></a>如果事件來源為 PasswordResetService

| 代碼 | 名稱或訊息 | 說明 |
| --- | --- | --- |
| 31001 | PasswordResetStart | 這個事件表示內部部署服務偵測到源自雲端、針對同盟、傳遞驗證或密碼雜湊同步使用者所提出的密碼重設要求。 這個事件是每個密碼重設回寫作業的第一個事件。 |
| 31002 | PasswordResetSuccess | 這個事件表示使用者在密碼重設作業期間選取了新的密碼。 我們認為這個密碼符合公司的密碼需求。 密碼已成功地寫回至本機的 Active Directory 環境。 |
| 31003 | PasswordResetFail | 此事件表示使用者選取了密碼，且密碼已成功送達內部部署環境。 但是，當我們嘗試在本機 Active Directory 環境中設定密碼時，發生失敗。 此失敗有幾個可能原因： <br><ul><li>使用者的密碼不符合使用期限、歷程記錄、複雜度或網域篩選需求。 若要解決這個問題，請建立新的密碼。</li><li>ADMA 服務帳戶沒有適當的權限，無法對所提及的使用者帳戶設定新密碼。</li><li>使用者的帳戶位於不允許密碼設定作業的受保護群組中，例如網域或企業系統管理員群組。</li></ul>|
| 31004 | OnboardingEventStart | 如果您啟用搭配 Azure AD Connect 運作的「密碼回寫」，就會發生此事件，且我們已開始將貴組織在「密碼回寫」Web 服務上線。 |
| 31005 | OnboardingEventSuccess | 這個事件表示上線程序已順利完成，「密碼回寫」功能已可供使用。 |
| 31006 | ChangePasswordStart | 這個事件表示內部部署服務偵測到源自雲端、針對同盟、傳遞驗證或密碼雜湊同步使用者所提出的密碼變更要求。 這個事件是每個密碼變更回寫作業的第一個事件。 |
| 31007 | ChangePasswordSuccess | 這個事件表示使用者在密碼變更作業期間選取了新的密碼，我們判斷此密碼符合公司的密碼需求，因此該密碼已成功回寫至本機 Active Directory 環境。 |
| 31008 | ChangePasswordFail | 這個事件表示使用者選取了密碼，該密碼已成功抵達內部部署環境，但是當我們嘗試在本機 Active Directory 環境中設定密碼時，發生了失敗狀況。 此失敗有幾個可能原因： <br><ul><li>使用者的密碼不符合使用期限、歷程記錄、複雜度或網域篩選需求。 若要解決這個問題，請建立新的密碼。</li><li>ADMA 服務帳戶沒有適當的權限，無法對所提及的使用者帳戶設定新密碼。</li><li>使用者的帳戶位於不允許密碼設定作業的受保護群組中，例如網域或企業系統管理員。</li></ul> |
| 31009 | ResetUserPasswordByAdminStart | 內部部署服務偵測到源自系統管理員，代表使用者且針對同盟、傳遞驗證或密碼雜湊同步使用者所提出的密碼重設要求。 這個事件是每個由系統管理員所起始之密碼重設回寫作業的第一個事件。 |
| 31010 | ResetUserPasswordByAdminSuccess | 系統管理員在系統管理員起始密碼重設作業期間選取了新的密碼。 我們認為這個密碼符合公司的密碼需求。 密碼已成功地寫回至本機的 Active Directory 環境。 |
| 31011 | ResetUserPasswordByAdminFail | 系統管理員代表使用者選取了密碼。 密碼已成功送達內部部署環境。 但是，當我們嘗試在本機 Active Directory 環境中設定密碼時，發生失敗。 此失敗有幾個可能原因： <br><ul><li>使用者的密碼不符合使用期限、歷程記錄、複雜度或網域篩選需求。 若要解決這個問題，請嘗試新的密碼。</li><li>ADMA 服務帳戶沒有適當的權限，無法對所提及的使用者帳戶設定新密碼。</li><li>使用者的帳戶位於不允許密碼設定作業的受保護群組中，例如網域或企業系統管理員。</li></ul>  |
| 31012 | OffboardingEventStart | 如果您停用搭配 Azure AD Connect 運作的「密碼回寫」，就會發生此事件，此事件表示我們已開始將貴組織在「密碼回寫」Web 服務下架。 |
| 31013| OffboardingEventSuccess| 這個事件表示下架程序已順利完成，「密碼回寫」功能已順利停用。 |
| 31014| OffboardingEventFail| 這個事件表示下架程序未成功。 這可能是因為雲端上或是設定期間所指定之內部部署系統管理員帳戶的權限錯誤。 如果您在停用密碼回寫時，嘗試使用同盟的雲端全域管理員，也會發生錯誤。 若要修正此問題，請檢查您的系統管理權限，並確定您在設定「密碼回寫」功能時，未使用已同盟的帳戶。|
| 31015| WriteBackServiceStarted| 這個事件表示已成功啟動密碼回寫服務。 已準備好接受來自雲端的密碼管理要求。|
| 31016| WriteBackServiceStopped| 這個事件表示密碼回寫服務已停止。 來自雲端的任何密碼管理要求都不會成功。|
| 31017| AuthTokenSuccess| 這個事件表示我們已順利擷取 Azure AD Connect 設定期間所指定之全域系統管理員的授權權杖，以便開始下架或上架程序。|
| 31018| KeyPairCreationSuccess| 這個事件表示我們已成功建立密碼加密金鑰。 此金鑰是用來加密從雲端傳送至您內部部署環境的密碼。|
| 32000| UnknownError| 這個事件表示密碼管理作業期間發生未知的錯誤。 請查看事件中的例外狀況文字，以獲得詳細資訊。 如果您遇到問題，請嘗試先將「密碼回寫」停用，然後再重新啟用。 如果這麼做沒有用，請將事件記錄複本連同內部指定的追蹤識別碼交給支援工程師。|
| 32001| ServiceError| 這個事件表示連線到雲端密碼重設服務時發生錯誤。 當內部部署服務無法連線到密碼重設 web 服務時，通常就會發生這個錯誤。|
| 32002| ServiceBusError| 這個事件表示連線到租用戶的服務匯流排執行個體時發生錯誤。 如果您在內部部署環境中封鎖了輸出連線，就會發生這個問題。 請檢查您的防火牆來確保您允許透過 TCP 443 及連到 https://ssprsbprodncu-sb.accesscontrol.windows.net/ 的連線，然後再試一次。 如果仍遇到問題，請嘗試先將「密碼回寫」停用，然後再重新啟用。|
| 32003| InPutValidationError| 這個事件表示傳遞到我們的 Web 服務 API 的輸入無效。 請重試一次此作業。|
| 32004| DecryptionError| 這個事件表示在解密從雲端抵達的密碼時發生錯誤。 原因可能是雲端服務和內部部署環境的解密金鑰不符。 若要解決此問題，請先將您內部部署環境中的「密碼回寫」停用，然後再重新啟用。|
| 32005| ConfigurationError| 上架期間，我們將租用戶特定的資訊儲存在內部部署環境的設定檔中。 這個事件表示儲存此檔案時發生錯誤，或啟動服務時發生檔案讀取錯誤。 若要修正此問題，請嘗試先將「密碼回寫」停用，然後再重新啟用，以強制重寫此組態檔。|
| 32007| OnBoardingConfigUpdateError| 上架期間，我們會從雲端傳送資料到內部部署密碼重設服務。 接著，該資料會寫入記憶體內檔案中，然後再傳送至同步處理服務，安全地儲存在磁碟上。 這個事件表示在記憶體中寫入或更新該資料時發生問題。 若要修正此問題，請嘗試先將「密碼回寫」停用，然後再重新啟用，以強制重寫此組態檔。|
| 32008| ValidationError| 這個事件表示我們從密碼重設 Web 服務收到的回應無效。 若要修正此問題，請嘗試先將「密碼回寫」停用，然後再重新啟用。|
| 32009| AuthTokenError| 這個事件表示我們無法取得 Azure AD Connect 設定期間所指定之全域系統管理員帳戶的授權權杖。 造成這個錯誤的原因可能是全域系統管理員帳戶所指定的使用者名稱或密碼錯誤。 如果所指定的全域管理員帳戶已同盟，也可能會發生這個錯誤。 若要修正此問題，請以正確的使用者名稱和密碼重新執行設定，並確保系統管理員是受控 (僅限雲端或已密碼同步處理的) 帳戶。|
| 32010| CryptoError| 這個事件表示在產生密碼加密金鑰或解密從雲端服務抵達的密碼時發生錯誤。 此錯誤可能表示您的環境有問題。 請查看事件記錄的詳細資料，深入了解此問題並加以解決。 您也可以嘗試先將「密碼回寫」服務停用，然後再重新啟用。|
| 32011| OnBoardingServiceError| 這個事件表示內部部署服務與密碼重設 Web 服務無法正確地通訊，來起始上架程序。 這可能是因為有防火牆規則，或是取得租用戶的驗證權杖時發生問題。 若要修正此問題，請確定您沒有封鎖透過 TCP 443 和 TCP 9350-9354 或連到 https://ssprsbprodncu-sb.accesscontrol.windows.net/ 的輸出連線。 並請確認您用來上架的 Azure AD 系統管理員帳戶並未同盟。|
| 32013| OffBoardingError| 這個事件表示內部部署服務與密碼重設 Web 服務無法正確地通訊，來起始下架程序。 這可能是因為有防火牆規則，或是取得租用戶的授權權杖時發生問題。 若要修正此問題，請確定您沒有封鎖透過 443 或連到 https://ssprsbprodncu-sb.accesscontrol.windows.net/ 的輸出連線，且您要用來下架的 Azure Active Directory 管理帳戶並未同盟。|
| 32014| ServiceBusWarning| 這個事件表示我們必須重試連線到租用戶的服務匯流排執行個體。 在正常情況下，您不必在意此事件，但如果您多次看到此事件，請考慮檢查服務匯流排網路連線，尤其是如果它是高延遲或低頻寬連線的話。|
| 32015| ReportServiceHealthError| 為了監視「密碼回寫」服務的健康情況，我們每隔 5 分鐘就會傳送一次活動訊號資料給我們的密碼重設 Web 服務。 這個事件表示在將此健全狀況資訊傳回到雲端 Web 服務時發生錯誤。 此健康情況資訊不包含物件識別資訊 (OII) 或個人識別資訊 (PII) 資料，純綷只有活動訊號和基本服務統計資料，以便我們可以在雲端中提供服務狀態資訊。|
| 33001| ADUnKnownError| 這個事件表示 Active Directory 傳回未知的錯誤。 如需詳細資訊，請檢查來自 ADSync 來源之事件的 Azure AD Connect 伺服器事件記錄。|
| 33002| ADUserNotFoundError| 這個事件表示在內部部署目錄中找不到嘗試重設或變更密碼的使用者。 當已在內部部署但未在雲端中刪除使用者時，就可能發生這個錯誤。 如果同步處理發生問題，也可能會發生這個錯誤。如需詳細資訊，請檢查您的同步處理記錄，以及最後幾次執行的同步處理詳細資料。|
| 33003| ADMutliMatchError| 當密碼重設或變更要求源自雲端時，我們會使用 Azure AD Connect 設定程序期間所指定的雲端錨點，來判斷如何將該要求往回連結到內部部署環境中的使用者。 這個事件表示我們發現內部部署目錄中有兩位使用者具有相同的雲端錨點屬性。 如需詳細資訊，請檢查您的同步處理記錄，以及最後幾次執行的同步處理詳細資料。|
| 33004| ADPermissionsError| 這個事件表示 Active Directory 管理代理程式 (ADMA) 服務帳戶沒有適當的指定帳戶權限，因此無法設定新密碼。 請確定使用者樹系中的 ADMA 帳戶有樹系中所有物件的重設和變更密碼權限。 如需有關如何設定權限的詳細資訊，請參閱步驟 4：設定適當的 Active Directory 權限。|
| 33005| ADUserAccountDisabled| 這個事件表示我們嘗試對內部部署中已停用的帳戶重設或變更密碼。 請啟用帳戶，然後再試一次此作業。|
| 33006| ADUserAccountLockedOut| 這個事件表示我們嘗試對內部部署中已鎖定的帳戶重設或變更密碼。 當使用者在短時間內嘗試進行變更或重設密碼作業太多次，就可能發生鎖定。 請解除鎖定帳戶，然後再試一次此作業。|
| 33007| ADUserIncorrectPassword| 這個事件表示使用者在執行密碼變更作業時，指定的目前密碼不正確。 請指定正確的目前密碼，然後再試一次。|
| 33008| ADPasswordPolicyError| 當「密碼回寫」服務嘗試在本機目錄上設定不符合密碼有效期、歷程記錄、複雜度或網域篩選需求的密碼時，就會發生此事件。 <br> <br> 如果您有最短的密碼使用期限，且最近在該時段內變更過密碼，就必須在到達網域中指定的使用期限後，才能再次變更密碼。 若要進行測試，最短使用期限應該設定為 0。 <br> <br> 如果已啟用密碼歷程記錄需求，則必須選取最近 N 次未用過的密碼，其中 N 是密碼歷程記錄設定。 如果您選取最近 N 次用過的密碼，則會在此案例中看到失敗。 若要進行測試，密碼歷程記錄應該設定為 0。 <br> <br> 如果您有密碼複雜度需求，則會在使用者嘗試變更或重設密碼時強制執行這些需求。 <br> <br> 如果您啟用密碼篩選功能，當使用者選取的密碼不符合篩選準則時，重設或變更作業就會失敗。|
| 33009| ADConfigurationError| 這個事件表示 Active Directory 的設定有問題，因此在將密碼回寫到內部部署目錄時發生問題。 如需所發生錯誤的詳細資訊，請檢查 Azure AD Connect 電腦的應用程式事件記錄中來自 ADSync 服務的訊息。|

## <a name="troubleshoot-password-writeback-connectivity"></a>針對密碼回寫連線問題進行疑難排解

如果 Azure AD Connect 的「密碼回寫」元件發生服務中斷，以下是您可用來解決此問題的一些快速步驟：

* [檢查網路連線](#confirm-network-connectivity)
* [重新啟動 Azure AD Connect 同步處理服務](#restart-the-azure-ad-connect-sync-service)
* [將密碼回寫功能先停用再重新啟用](#disable-and-re-enable-the-password-writeback-feature)
* [安裝最新版的 Azure AD Connect](#install-the-latest-azure-ad-connect-release)
* [疑難排解密碼回寫](#troubleshoot-password-writeback)

一般而言，若要以最快速的方式復原服務，建議您依上述順序執行這些步驟。

### <a name="confirm-network-connectivity"></a>檢查網路連線

最常見的失敗點是防火牆和 Proxy 連接埠，以及閒置逾時的設定不正確。 

對於 Azure AD Connect 1.1.443.0 版和更新版本，您需要擁有下列各項的輸出 HTTPS 存取權：

   - passwordreset.microsoftonline.com
   - servicebus.windows.net

如需詳細資訊，請參閱 [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Microsoft Azure Datacenter IP 範圍) 的更新清單，該清單會在每星期三更新、隔週星期一生效。

如需詳細資訊，請在 [Azure AD Connect 的必要條件](../hybrid/how-to-connect-install-prerequisites.md)一文中檢閱連線必要條件。



### <a name="restart-the-azure-ad-connect-sync-service"></a>重新啟動 Azure AD Connect 同步處理服務

如需解決服務的連線問題或其他暫時性問題，請重新啟動 Azure AD Connect 同步處理服務：

   1. 以系統管理員身分，在執行 Azure AD Connect 的伺服器上選取 [啟動]。
   2. 在搜尋欄位中輸入 **services.msc**並選取 [輸入]。
   3. 找出 **Microsoft Azure AD Sync** 項目。
   4. 以滑鼠右鍵按一下服務項目，選取 [重新啟動]，然後等候作業完成。

   ![重新啟動 Azure AD Sync 服務][Service restart]

這些步驟會重新建立您與雲端服務的連線，解決您可能會遇到的任何中斷問題。 如果重新啟動 ADSync 服務並未解決您的問題，建議您接下來嘗試先將「密碼回寫」功能停用，然後再重新啟用。

### <a name="disable-and-re-enable-the-password-writeback-feature"></a>先將「密碼回寫」功能停用，然後再重新啟用

若要解決連線問題，請先將「密碼回寫」功能停用，然後再重新啟用：

   1. 以系統管理員身分，開啟 [Azure AD Connect 設定精靈]。
   2. 在 [連線到 Azure AD] 中，輸入您的「Azure AD 全域系統管理員認證」。
   3. 在 [連線到 AD DS] 中，輸入您的「AD Domain Services 系統管理員認證」。
   4. 在 [專門識別您的使用者] 中，選取 [下一步] 按鈕。
   5. 在 [選用功能] 中，清除 [密碼回寫] 核取方塊。
   6. 在其餘的對話方塊頁面上逐一選取 [下一步] 而不變更任何項目，直到到達 [準備好設定] 頁面為止。
   7. 確認 [準備好設定] 頁面顯示 [密碼回寫]選項為 [停用]，然後選取綠色的 [設定] 按鈕來認可變更。
   8. 在 [已完成] 中，清除 [立即同步處理] 選項，然後選取 [完成] 來關閉精靈。
   9. 重新開啟 [Azure AD Connect 設定精靈]。
   10. 重複步驟 2-8，但請確定您已在 [選用功能] 頁面上選取 [密碼回寫] 選項來重新啟用服務。

這些步驟會重新建立您與雲端服務的連線，解決您可能會遇到的任何中斷問題。

如果先將「密碼回寫」功能停用然後再重新啟用並未解決您的問題，建議您重新安裝 Azure AD Connect。

### <a name="install-the-latest-azure-ad-connect-release"></a>安裝最新版的 Azure AD Connect

重新安裝 Azure AD Connect 可以解決我們的雲端服務和您的本機 Active Directory 環境之間的設定和連線問題。

建議您只在嘗試過上述前兩個步驟後，才執行此步驟。

> [!WARNING]
> 如果您已自訂現成可用的同步處理規則，先備份這些規則，然後再繼續進行升級，並於完成後以手動方式重新部署這些規則。

   1. 從 [Microsoft 下載中心](http://go.microsoft.com/fwlink/?LinkId=615771)下載最新版的 Azure AD Connect。
   2. 由於您已安裝 Azure AD Connect，需要執行就地升級，即可將 Azure AD Connect 安裝更新為最新版。
   3. 執行下載的封裝，並遵循螢幕上的指示來更新您的 Azure AD Connect 電腦。

先前步驟應該會重新建立您與雲端服務的連線，解決您可能會遇到的任何中斷問題。

如果安裝最新版 Azure AD Connect 伺服器並未解決您的問題，最後一步是建議您在安裝最新版本後，試著先將「密碼回寫」停用，然後再重新啟用。

## <a name="verify-that-azure-ad-connect-has-the-required-permission-for-password-writeback"></a>確認 Azure AD Connect 具有執行密碼回寫所需的權限

Azure AD Connect 需要 Active Directory **重設密碼**權限才能執行密碼回寫。 若要了解 Azure AD Connect 是否有給定內部部署 Active Directory 使用者帳戶的必要權限，您可以使用 Windows 有效權限功能：

   1. 登入 Azure AD Connect 伺服器，並啟動 **Synchronization Service Manager**，方法是選取 [開始] > [同步處理服務]。
   2. 在 [連接器] 索引標籤下，選取內部部署 [Active Directory Domain Services] 連接器，然後選取 [屬性]。  

   ![有效權限 - 步驟 2](./media/active-directory-passwords-troubleshoot/checkpermission01.png)  
  
   3. 在快顯視窗中，選取 [連線到 Active Directory 樹系] 索引標籤，然後記下 [使用者名稱] 屬性。 這個屬性是 Azure AD Connect 用來執行目錄同步作業的 AD DS 帳戶。 若要讓 Azure AD Connect 能夠執行密碼回寫，AD DS 帳戶必須有「重設密碼」權限。  
   
   ![有效權限 - 步驟 3](./media/active-directory-passwords-troubleshoot/checkpermission02.png) 
  
   4. 登入內部部署網域控制站，然後啟動 **Active Directory 使用者和電腦**應用程式。
   5. 選取 [檢視]，並確定 [進階功能] 選項已啟用。  
   
   ![有效權限 - 步驟 5](./media/active-directory-passwords-troubleshoot/checkpermission03.png) 
  
   6. 尋找您需要確認的 Active Directory 使用者帳戶。 以滑鼠右鍵按一下帳戶名稱，然後選取 [屬性]。  
   
   ![有效權限 - 步驟 6](./media/active-directory-passwords-troubleshoot/checkpermission04.png) 

   7. 在快顯視窗中，移至 [安全性] 索引標籤，然後選取 [進階]。  
   
   ![有效權限 - 步驟 7](./media/active-directory-passwords-troubleshoot/checkpermission05.png) 
   
   8. 在 [系統管理員的進階安全性設定] 快顯視窗中，移至 [有效存取權] 索引標籤。
   9. 選取 [選取使用者]，選取 Azure AD Connect 所使用的 AD DS 帳戶 (請參閱步驟 3)，然後再選取 [檢視有效存取權]。  
   
   ![有效權限 - 步驟 9](./media/active-directory-passwords-troubleshoot/checkpermission06.png) 
  
   10. 向下捲動並尋找 [重設密碼]。 如果該項目有核取記號，AD DS 帳戶就有權限可重設選定 Active Directory 使用者帳戶的密碼。  
   
   ![有效權限 - 步驟 10](./media/active-directory-passwords-troubleshoot/checkpermission07.png)  

## <a name="azure-ad-forums"></a>Azure AD 論壇

如果您有關於 Azure AD 和自助式密碼重設的一般問題，您可以在 [Azure AD 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=WindowsAzureAD)尋求社群協助。 社群的成員包括工程師、產品經理、MVP 和夥伴 IT 專業人員。

## <a name="contact-microsoft-support"></a>連絡 Microsoft 支援

如果找不到問題的答案，我們的支援團隊一直都能進一步協助您。

為了正確地提供協助，我們會要求您在開啟案例時，提供盡可能詳細的資料。 這些詳細資料包括：

* **錯誤的一般描述**：錯誤為何？ 注意到何種行為？ 我們如何才能重現錯誤？ 請提供盡可能詳細的資料。
* **頁面**：您注意到錯誤時的所在頁面？ 盡可能包含 URL 和頁面的螢幕擷取畫面。
* **支援碼**：使用者看到錯誤時所產生的支援碼？
    * 若要找到支援碼，請重現錯誤，然後按一下畫面底部的 [支援碼] 連結，將所產生的 GUID 傳送給支援工程師。

    ![尋找畫面底部的支援碼][Support code]

    * 如果您所在的頁面底部沒有支援碼，請選取 F12，搜尋 SID 和 CID，然後將這兩個結果傳送給支援工程師。
* **日期、時間和時區**：請包含發生錯誤的精確日期和時間 (含時區)。
* **使用者識別碼**：看到錯誤的使用者是誰？ 例如 user@contoso.com。
    * 這是同盟使用者嗎？
    * 這是傳遞驗證使用者嗎？
    * 這是密碼雜湊同步使用者嗎？
    * 這是否僅限雲端的使用者？
* **授權**：使用者是否已獲得 Azure AD Premium 或 Basic 授權？
* **應用程式事件記錄**：如果您使用密碼回寫，而且錯誤位於您的內部部署基礎結構中，請包含來自 Azure AD Connect 伺服器的應用程式事件記錄壓縮複本。



[Service restart]: ./media/active-directory-passwords-troubleshoot/servicerestart.png "重新啟動 Azure AD Sync 服務"
[Support code]: ./media/active-directory-passwords-troubleshoot/supportcode.png "支援碼位於視窗右下角"

## <a name="next-steps"></a>後續步驟

下列文章提供有關透過 Azure AD 重設密碼的其他資訊：

* [如何完成 SSPR 成功首度發行？](howto-sspr-deployment.md)
* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)
* [您有授權問題嗎？](concept-sspr-licensing.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [哪些驗證方法可供使用者使用？](concept-sspr-howitworks.md#authentication-methods)
* [使用 SSPR 的原則選項有哪些？](concept-sspr-policy.md)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [如何回報 SSPR 中的活動？](howto-sspr-reporting.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)
