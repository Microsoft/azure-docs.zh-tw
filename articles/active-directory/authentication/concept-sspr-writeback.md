---
title: 內部部署密碼回寫與 Azure AD SSPR 的整合
description: 在內部部署 AD 基礎結構中添加雲端密碼回寫功能
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 8440d8a492105365417190ad286798e0bdf47a0c
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295830"
---
# <a name="what-is-password-writeback"></a>什麼是密碼回寫？

能擁有雲端式密碼重設公用程式很棒，但大部分公司仍有供其使用者使用的內部部署目錄。 Microsoft 支援要如何讓傳統的內部部署 Active Directory (AD) 與雲端中的密碼變更保持同步？ 密碼回寫是透過 [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) 所實現的功能，可將雲端中的密碼變更即時回寫至現有內部部署目錄。

使用下列項目的環境可支援密碼回寫：

* [Active Directory 同盟服務](../hybrid/how-to-connect-fed-management.md) \(英文\)
* [密碼雜湊同步處理](../hybrid/how-to-connect-password-hash-synchronization.md)
* [傳遞驗證](../hybrid/how-to-connect-pta.md)

密碼回寫提供：

* **強制執行內部部署 Active Directory 密碼原則**：當使用者重設其密碼時，系統會進行檢查以確保此動作符合內部部署 Active Directory 原則，然後才將此動作認可至該目錄。 這項檢閱包括檢查歷程記錄、複雜度、有效期、密碼篩選，以及您在本機 Active Directory 中已定義的任何其他密碼限制。
* **零延遲的意見反應**：密碼回寫是一項同步作業。 如果使用者的密碼不符合原則，或因為任何原因而無法重設或變更，他們會立即收到通知。
* **支援從存取面板和 Office 365 變更密碼**：當已同盟的或已同步處理密碼雜湊的使用者變更其已過期或尚未過期的密碼時，系統會將這些密碼回寫到本機 Active Directory 環境。
* **支援在管理員從 Azure 入口網站重設密碼時將密碼回寫**：每當管理員在 [Azure 入口網站](https://portal.azure.com)中重設使用者密碼時，如果該使用者為已同盟的或已同步處理密碼雜湊的使用者，系統就會將密碼回寫至內部部署環境。 Office 管理入口網站目前不支援此功能。
* **不需要任何輸入防火牆規則**：密碼回寫會使用「Azure 服務匯流排」轉送作為基礎通訊通道。 所有通訊都會透過連接埠 443 來輸出。

> [!Note]
> 存在於內部部署 Active Directory 中受保護群組內的使用者帳戶，無法使用密碼回寫。 如需受保護群組的詳細資訊，請參閱 [Active Directory 中的受保護帳戶和群組](https://technet.microsoft.com/library/dn535499.aspx)。

## <a name="licensing-requirements-for-password-writeback"></a>密碼回寫的授權需求

**自助式密碼重設/變更/使用內部部署回寫來解除鎖定是 Azure AD 的進階功能**。 如需授權的詳細資訊，請參閱 [Azure Active Directory 價格網站](https://azure.microsoft.com/pricing/details/active-directory/)。

若要使用密碼回寫，您的租用戶中必須已指派下列其中一項授權：

* Azure AD Premium P1
* Azure AD Premium P2
* Enterprise Mobility + Security E3 或 A3
* Enterprise Mobility + Security E5 或 A5
* Microsoft 365 E3 或 A3
* Microsoft 365 E5 或 A5
* Microsoft 365 F1

> [!WARNING]
> 獨立的 Office 365 授權方案不支援密碼回寫，而且需要您具備上述其中一個方案，這項功能才能運作。
>

## <a name="how-password-writeback-works"></a>密碼回寫的運作方式

當已同盟的或已同步處理密碼雜湊的使用者嘗試在雲端重設或變更其密碼時，會發生下列動作：

1. 系統會執行檢查，以了解使用者擁有哪一種密碼。 如果密碼是在內部部署環境中進行管理的：
   * 系統會執行檢查以了解回寫服務是否已啟動並在執行中。 如果是，使用者可以繼續進行。
   * 如果回寫服務關閉，系統會告知使用者其無法立即重設其密碼。
2. 接著，使用者會通過適當的驗證閘道，然後到達 [重設密碼] 畫面。
3. 使用者選取新的密碼，並加以確認。
4. 當使用者選取 [送出] 時，系統會以在回寫設定程序期間所建立的對稱金鑰加密純文字密碼。
5. 經過加密的密碼會放入承載中，透過 HTTPS 通道傳送到租用戶特定的服務匯流排轉送 (會在回寫設定程序期間為您設定此轉送)。 此轉送受到只有您的內部部署安裝才會知道的隨機產生密碼所保護。
6. 在訊息抵達服務匯流排之後，密碼重設端點會自動甦醒，並看到有擱置中的重設要求。
7. 服務會接著使用雲端錨點屬性來尋找使用者。 若要讓此查閱成功︰

   * 使用者物件必須存在於 Active Directory 連接器空間中。
   * 使用者物件必須連結至對應的 Metaverse (MV) 物件。
   * 使用者物件必須連結至對應的 Azure Active Directory 連接器物件。
   * 從 Active Directory 連接器物件到 MV 的連結上必須有同步處理規則 `Microsoft.InfromADUserAccountEnabled.xxx`。 <br> <br>
   從雲端傳來呼叫時，同步處理引擎會使用 **cloudAnchor** 屬性來查閱 Azure Active Directory 連接器空間物件。 接著，它會依循連結回到 MV 物件，然後再依循連結回到 Active Directory 物件。 因為相同使用者可能會有多個 Active Directory 物件 (多樹系)，所以同步處理引擎需倚賴 `Microsoft.InfromADUserAccountEnabled.xxx` 連結來選出正確的物件。

   > [!Note]
   > 由於這個邏輯的緣故，Azure AD Connect 必須要能夠與主要網域控制站 (PDC) 模擬器通訊，密碼回寫才能運作。 如果需要手動啟用此功能，您可以將 Azure AD Connect 連線到 PDC 模擬器。 在 Active Directory 同步處理連接器的 [屬性] 上按一下滑鼠右鍵，然後選取 [設定目錄分割]。 從該處尋找 [網域控制站連線設定] 區段，然後核取標題為 [只使用慣用的網域控制站] 的方塊。 即使慣用的網域控制站不是 PDC 模擬器，Azure AD Connect 仍會嘗試連線至 PDC 來進行密碼回寫。

8. 找到使用者帳戶之後，系統會嘗試在適當的 Active Directory 樹系中直接重設密碼。
9. 如果密碼設定作業成功，系統會告訴使用者其密碼已變更。
   > [!NOTE]
   > 如果將使用者的密碼雜湊同步處理至 Azure AD 時使用的是密碼雜湊同步處理，內部部署密碼原則就有可能比雲端密碼原則弱。 在此情況下，系統會強制執行內部部署原則。 此原則可確保不論您是使用密碼雜湊同步處理還是同盟來提供單一登入，都會在雲端強制執行內部部署原則。

10. 如果密碼設定作業失敗，系統會顯示錯誤以提示使用者再試一次。 作業可能會因下列原因而失敗：
   * 服務已停止運作。
   * 他們所選的密碼不符合組織原則。
   * 無法在本機 Active Directory 中找到使用者。

    錯誤訊息會對使用者提供指引，讓他們可以嘗試自行解決而不需要系統管理員介入。

## <a name="password-writeback-security"></a>密碼回寫安全性

密碼回寫是一項極為安全的服務。 為了確保資訊會受到保護，系統會啟用四層式安全性模型，如下所述：

* **租用戶特定的服務匯流排轉送**
   * 當您設定服務時，便會設定一個以隨機產生強式密碼保護的租用戶特定服務匯流排轉送，且 Microsoft 永遠無法得知該密碼。
* **受到鎖定的密碼編譯強式密碼加密金鑰**
   * 建立服務匯流排轉送之後，便會建立強式對稱金鑰，以在透過線路傳送密碼時予以加密。 此金鑰只會存在於您公司的雲端密碼存放區中，並受到嚴密鎖定和稽核，就像目錄中的任何其他密碼一樣。
* **業界標準傳輸層安全性 (TLS)**
   1. 當雲端上發生密碼重設或變更作業時，系統便會以公開金鑰將純文字密碼加密。
   2. 加密密碼會放入 HTTPS 訊息中，使用 Microsoft SSL 憑證透過加密通道傳送到您的服務匯流排轉送。
   3. 在訊息抵達服務匯流排之後，您的內部部署代理程式會喚醒服務匯流排，並使用先前產生的強式密碼向其進行驗證。
   4. 內部部署代理程式會拾取加密訊息，然後使用私密金鑰進行解密。
   5. 內部部署代理程式會嘗試透過 AD DS SetPassword API 來設定密碼。 這個步驟可讓您在雲端強制執行 Active Directory 內部部署密碼原則 (例如複雜度、有效期、歷程記錄、篩選等)。
* **訊息到期原則**
   * 如果訊息因內部部署服務已停止運作而停留在服務匯流排中，在數分鐘之後，它就會逾時而被移除。 讓訊息逾時並予以移除可更進一步提升安全性。

### <a name="password-writeback-encryption-details"></a>密碼回寫的加密詳細資料

在使用者提交密碼重設之後，重設要求會先經過數個加密步驟，然後才抵達您的內部部署環境。 這些加密步驟可確保提供最高的服務可靠性和安全性。 這些步驟的說明如下：

* **步驟 1：採用 2048 位元 RSA 金鑰的密碼加密**在使用者提交要寫回到內部部署環境的密碼之後，所提交的密碼本身會以 2048 位元 RSA 金鑰加密。
* **步驟 2：採用 AES-GCM 的套件層級加密**：整個套件 (密碼 + 必要的中繼資料) 會以 AES-GCM 加密。 這個加密可防止任何可直接存取基礎 ServiceBus 通道的人員檢視或竄改內容。
* **步驟 3：所有通訊都會透過 TLS/SSL 進行**：與 ServiceBus 的所有通訊都會在 SSL/TLS 通道中進行。 這個加密可保護內容，免於遭到未經授權的第三方存取。
* **每 6 個月自動變換金鑰**：所有金鑰會每 6 個月變換一次，或在每一次於 Azure AD Connect 上停用再重新啟用密碼回寫時進行變換，以確保最高的服務安全性。

### <a name="password-writeback-bandwidth-usage"></a>密碼回寫頻寬使用量

密碼回寫是一種低頻寬服務，只有在下列情況下，才會將要求傳回給內部部署代理程式︰

* 透過 Azure AD Connect 啟用或停用此功能時會傳送兩則訊息。
* 每 5 分鐘傳送一則訊息作為服務活動訊號 (只要服務正在執行)。
* 每次提交新密碼時會傳送兩則訊息：
   * 第一則訊息是一個執行作業的要求。
   * 第二則訊息包含該作業的結果，並且會在下列情況下傳送︰
      * 每次在使用者自助式密碼重設期間提交新密碼時。
      * 每次在使用者密碼變更作業期間提交新密碼時。
      * 每次在系統管理員所起始的使用者密碼重設 (僅限從 Azure 系統管理員入口網站) 期間提交新密碼時。

#### <a name="message-size-and-bandwidth-considerations"></a>訊息大小和頻寬考量

上述每則訊息的大小通常會小於 1 KB。 即使在負載極大的情況下，密碼回寫服務本身每秒也只會耗用幾千位元的頻寬。 因為每則訊息都以即時方式傳送 (僅限密碼更新作業要求時)，且因為訊息大小是如此的小，所以回寫功能的頻寬使用量會因太小，以致沒有可測得的影響。

## <a name="supported-writeback-operations"></a>支援的回寫作業

下列所有情況都會回寫密碼︰

* **支援的使用者作業**
   * 任何使用者自助式自願變更密碼作業
   * 任何使用者自助式強制變更密碼作業 (例如密碼到期)
   * 任何源自[密碼重設入口網站](https://passwordreset.microsoftonline.com)的使用者自助式密碼重設
* **支援的系統管理員作業**
   * 任何系統管理員自助式自願變更密碼作業
   * 任何系統管理員自助式強制變更密碼作業 (例如密碼到期)
   * 任何源自[密碼重設入口網站](https://passwordreset.microsoftonline.com)的系統管理員自助式密碼重設
   * 系統管理員從 [Azure 入口網站](https://portal.azure.com)起始的任何使用者密碼重設

## <a name="unsupported-writeback-operations"></a>不支援的回寫作業

下列任何情況均*不會*回寫密碼︰

* **不支援的使用者作業**
   * 任何由使用者使用 PowerShell 第 1 版、第 2 版或 Azure AD Graph API 來進行的自有密碼重設
* **不支援的系統管理員作業**
   * 系統管理員從 [Office 管理入口網站](https://portal.office.com)起始的任何使用者密碼重設
   * 任何由系統管理員從 PowerShell 第 1 版、第 2 版或 Azure AD Graph API 起始的使用者密碼重設

## <a name="next-steps"></a>後續步驟

使用下列教學課程啟用密碼回寫：[啟用密碼回寫](tutorial-enable-writeback.md)
