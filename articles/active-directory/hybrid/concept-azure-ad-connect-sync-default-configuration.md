---
title: Azure AD Connect 同步：了解預設組態 | Microsoft Docs
description: 本文說明 Azure AD Connect Sync 的預設組態。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: ed876f22-6892-4b9d-acbe-6a2d112f1cd1
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: bd708d279649138fcb17362491da4eb7539c478b
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308747"
---
# <a name="azure-ad-connect-sync-understanding-the-default-configuration"></a>Azure AD Connect 同步處理：了解預設組態
本文說明現成可用的組態規則。 其中說明這些規則以及這些規則對組態有何影響。 本文也會引導您完成 Azure AD Connect 同步處理的預設組態。其目的是讓讀者了解組態模型 (名為宣告式佈建) 在實際範例中的運作情形。 本文假設您已使用安裝精靈安裝並設定 Azure AD Connect Sync。

若要了解組態模型的詳細資訊，請參閱 [了解宣告式佈建](concept-azure-ad-connect-sync-declarative-provisioning.md)。

## <a name="out-of-box-rules-from-on-premises-to-azure-ad"></a>從內部部署至 Azure AD 的現成可用規則
現成可用的組態中包含下列運算式。

### <a name="user-out-of-box-rules"></a>使用者現成可用的規則
這些規則也會套用至 iNetOrgPerson 物件類型。

使用者物件必須符合下列條件，才會進行同步處理：

* 必須具有 sourceAnchor。
* 在 Azure AD 中建立物件之後，即無法變更 sourceAnchor。 如果值在內部部署中變更，則物件會停止同步處理，直到 sourceAnchor 重新變更為原先的值。
* 必須有已填入的 accountEnabled (userAccountControl) 屬性。 在內部部署 Active Directory 中，一律會存在並填入此屬性。

下列使用者物件 **不會** 同步處理至 Azure AD：

* `IsPresent([isCriticalSystemObject])`。 請確定 Active Directory 中多項現成可用的物件 (例如內建的系統管理員帳戶) 不會同步處理。
* `IsPresent([sAMAccountName]) = False`。 請確定沒有 sAMAccountName 屬性的使用者物件不會同步處理。 這種情況實際上只會發生在從 NT4 升級的網域中。
* `Left([sAMAccountName], 4) = "AAD_"`，`Left([sAMAccountName], 5) = "MSOL_"`。 不要同步處理 Azure AD Connect Sync 和較早版本所使用的服務帳戶。
* 請勿同步處理不會在 Exchange Online 中運作的 Exchange 帳戶。
  * `[sAMAccountName] = "SUPPORT_388945a0"`
  * `Left([mailNickname], 14) = "SystemMailbox{"`
  * `(Left([mailNickname], 4) = "CAS_" && (InStr([mailNickname], "}") > 0))`
  * `(Left([sAMAccountName], 4) = "CAS_" && (InStr([sAMAccountName], "}")> 0))`
* 請勿同步處理不會在 Exchange Online 中運作的物件。
  `CBool(IIF(IsPresent([msExchRecipientTypeDetails]),BitAnd([msExchRecipientTypeDetails],&H21C07000) > 0,NULL))`  
  此位元遮罩 (&amp;H21C07000) 會篩選掉下列物件：
  * 擁有郵件功能的公用資料夾 (自 1.1.524.0 版起為預覽版功能)
  * 系統服務員信箱
  * 信箱資料庫信箱 (系統信箱)
  * 萬用安全性群組 (不會對使用者套用，但因舊版因素而存在)
  * 非萬用群組 (不會對使用者套用，但因舊版因素而存在)
  * 信箱計劃
  * 探索信箱
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`。 請不要同步處理任何複寫犧牲者物件。

適用的屬性規則如下：

* `sourceAnchor <- IIF([msExchRecipientTypeDetails]=2,NULL,..)`。 不會從連結的信箱提供 SourceAnchor 屬性。 根據假設，如果已找到連結的信箱，實際的帳戶會在稍後加入。
* 只有在屬性 **mailNickName** 具有值時，才會同步處理 Exchange 的相關屬性。
* 如果有多個樹系，則會依下列順序使用屬性：
  1. 會從具有已啟用帳戶的樹系提供登入的相關屬性 (例如 userPrincipalName)。
  2. 可以在 Exchange GAL (全域通訊清單) 中找到的屬性，則會從具有 Exchange 信箱的樹系提供。
  3. 如果找不到信箱，則這些屬性可來自於任何樹系。
  4. Exchange 相關屬性 (技術屬性不會顯示在 GAL 中) 會從 `mailNickname ISNOTNULL`的樹系提供。
  5. 如果有多個樹系會符合其中一個規則，則會使用連接器 (樹系) 的建立順序 (日期/時間) 來決定由哪個樹系提供屬性。

### <a name="contact-out-of-box-rules"></a>連絡人現成可用的規則
連絡人物件必須符合下列條件，才會進行同步處理：

* 連絡人必須擁有郵件功能。 這會使用下列規則來驗證：
  * `IsPresent([proxyAddresses]) = True)`。 必須填入 proxyAddresses 屬性。
  * 可在 proxyAddresses 屬性或郵件屬性中找到主要電子郵件地址。 存在的 \@ 可用來證實內容是電子郵件地址。 下列其中一個規則必須評估為 True。
    * `(Contains([proxyAddresses], "SMTP:") > 0) && (InStr(Item([proxyAddresses], Contains([proxyAddresses], "SMTP:")), "@") > 0))`。 是否有含有 "SMTP:" 的項目，如果有，是否可在字串中找到 \@？
    * `(IsPresent([mail]) = True && (InStr([mail], "@") > 0)`。 是否已填入郵件屬性，如果是，是否可在字串中找到 \@？

下列連絡人物件 **不會** 同步處理至 Azure AD：

* `IsPresent([isCriticalSystemObject])`。 請確定沒有標記為重要的連絡人物件進行同步處理。 不應該是任何使用預設組態的項目。
* `((InStr([displayName], "(MSOL)") > 0) && (CBool([msExchHideFromAddressLists])))`。
* `(Left([mailNickname], 4) = "CAS_" && (InStr([mailNickname], "}") > 0))`。 這些物件無法在 Exchange Online 中運作。
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`。 請不要同步處理任何複寫犧牲者物件。

### <a name="group-out-of-box-rules"></a>群組現成可用的規則
群組物件必須符合下列條件，才會進行同步處理：

* 擁有的成員必須少於 50,000 個。 此計數為內部部署群組中的成員數目。
  * 如果在第一次開始同步處理之前擁有較多成員，則不會同步處理群組。
  * 如果成員數目在它最初建立之後有所成長，在達到 50,000 個成員時，它會停止同步處理，直到成員資格計數再次低於 50,000。
  * 注意：Azure AD 也會強制執行 50,000 個成員資格計數。 您無法同步處理具有更多個成員的群組，即使您修改或移除此規則亦然。
* 如果群組是 **通訊群組**，則也必須擁有郵件功能。 請參閱 [連絡人現成可用的規則](#contact-out-of-box-rules) ，以了解強制執行此規則的情形。

下列群組物件 **不會** 同步處理至 Azure AD：

* `IsPresent([isCriticalSystemObject])`。 請確定 Active Directory 中多項現成可用的物件 (例如內建的系統管理員群組) 不會同步處理。
* `[sAMAccountName] = "MSOL_AD_Sync_RichCoexistence"`。 DirSync 所使用的舊版群組。
* `BitAnd([msExchRecipientTypeDetails],&amp;H40000000)`。 角色群組。
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`。 請不要同步處理任何複寫犧牲者物件。

### <a name="foreignsecurityprincipal-out-of-box-rules"></a>ForeignSecurityPrincipal 現成可用的規則
FSP 會聯結至 Metaverse 中的「任何」(\*) 物件。 實際上，此聯結只會針對使用者和安全性群組執行。 此組態可確保跨樹系成員資格會進行解析，並正確地顯示在 Azure AD 中。

### <a name="computer-out-of-box-rules"></a>電腦現成可用的規則
電腦物件必須符合下列條件，才會進行同步處理：

* `userCertificate ISNOTNULL`。 只有 Windows 10 電腦會填入此屬性。 所有具有此屬性值的電腦物件都會進行同步處理。

## <a name="understanding-the-out-of-box-rules-scenario"></a>了解現成可用的規則案例
在此範例中，我們會使用具有一個帳戶樹系 (A)、一個資源樹系 (R) 和一個 Azure AD 目錄的部署。

![含案例說明的圖片](./media/concept-azure-ad-connect-sync-default-configuration/scenario.png)

在此組態中，假設帳戶樹系中有已啟用的帳戶，而具有連結信箱的資源樹系中有已停用的帳戶。

我們使用預設組態的目的是：

* 登入的相關屬性會從具有已啟用帳戶的樹系進行同步處理。
* 可以在 GAL (全域通訊清單) 中找到的屬性，會從具有信箱的樹系進行同步處理。 如果找不到信箱，則會使用任何其他樹系。
* 如果找到連結的信箱，則要匯出至 Azure AD 的物件必須有已連結並啟用的帳戶。

### <a name="synchronization-rule-editor"></a>同步處理規則編輯器
您可以使用同步處理規則編輯器 (SRE) 這項工具來檢視及變更組態，並且可在 [開始] 功能表中找到其捷徑。

![同步處理規則編輯器圖示](./media/concept-azure-ad-connect-sync-default-configuration/sre.png)

SRE 是一項資源套件工具，會隨 Azure AD Connect Sync 一起安裝。您必須是 ADSyncAdmins 群組的成員，才能夠加以啟動。 在它啟動時，您會看到如下的畫面：

![同步處理規則 (輸入)](./media/concept-azure-ad-connect-sync-default-configuration/syncrulesinbound.png)

在此窗格中，您會看到所有為您的組態建立的同步處理規則。 表格中的每一行都是一個同步處理規則。 在 [規則類型] 的左下方會列出兩種不同類型：[輸入] 和 [輸出]。 [輸入] 和 [輸出] 來自 Metaverse 的檢視。 您主要會將重點放在本概觀中的輸入規則。 實際的同步處理規則清單取決於在 AD 中偵測到的結構描述。 在上圖中，帳戶樹系 (fabrikamonline.com) 沒有任何服務 (例如 Exchange 和 Lync)，而且也沒有為這些服務建立任何同步處理規則。 不過，在資源樹系 (res.fabrikamonline.com) 中，您可找到這些服務的同步處理規則。 規則的內容會隨著偵測到的版本而有所不同。 比方說，在 Exchange 2013 的部署中，屬性流程比在 Exchange 2010/2007 中設定的多。

### <a name="synchronization-rule"></a>同步處理規則
同步處理規則是一個組態物件，當滿足條件時會有一組屬性進行流動。 此規則也會用來說明連接器空間中物件與 Metaverse 中物件的關係 (稱為**聯結**或**相符項目**)。 同步處理規則具有優先順序值，指出它們彼此之間的關係。 具有較低數值的同步處理規則具有較高的優先順序，而在發生屬性流程衝突時，較高的優先順序會在衝突解決過程中勝出。

舉例來說，我們將了解同步處理規則 **In from AD – User AccountEnabled**。 在 SRE 中標示這一行並選取 [編輯] 。

由於此規則是現成可用的規則，因此您會在開啟規則時收到警告。 您不應 [變更現成可用的規則](how-to-connect-sync-best-practices-changing-default-configuration.md)，而要讓系統詢問您自己的意願為何。 在此案例中，您只想要檢視規則。 請選取 [否] 。

![同步處理規則警告](./media/concept-azure-ad-connect-sync-default-configuration/warningeditrule.png)

同步處理規則具有四個組態區段：說明、範圍篩選器、聯結規則及轉換。

#### <a name="description"></a>說明
第一個區段提供基本資訊，例如名稱和說明。

![同步處理規則編輯器中的說明索引標籤 ](./media/concept-azure-ad-connect-sync-default-configuration/syncruledescription.png)

您也可尋找一些資訊，例如哪個已連線系統與此規則有關、哪個已連線系統的物件類型適用於此規則，以及 Metaverse 物件的類型。 無法來源物件類型為使用者、iNetOrgPerson 或連絡人，Metaverse 物件類型一律為個人。 Metaverse 物件類型永遠不會改變，所以它必須以一般類型建立。 您可以將連結類型設為 Join、StickyJoin 或 Provision。 此設定會與 [聯結規則] 區段共同運作，稍後會討論其運作方式。

您也可以看到此同步處理規則用於密碼同步。如果使用者在此同步處理規則的範圍內，密碼會從內部部署同步處理至雲端 (假設您已啟用密碼同步功能)。

#### <a name="scoping-filter"></a>範圍篩選器
範圍篩選器區段是用來設定同步處理規則套用的時機。 由於您目前看到的同步處理規則名稱表示只應針對已啟用使用者套用，因此您必須設定該範圍，切勿將 AD 屬性 **userAccountControl** 設為位元 2。 當同步處理引擎在 AD 中尋找使用者時，如果 **userAccountControl** 設為十進位值 512 (啟用的一般使用者)，則會套用此同步處理規則。 當使用者的 **userAccountControl** 設為 514 (停用的一般使用者) 時，則不會套用此規則。

![同步處理規則編輯器中的範圍索引標籤 ](./media/concept-azure-ad-connect-sync-default-configuration/syncrulescopingfilter.png)

範圍篩選器具有可為巢狀的群組和子句。 必須滿足群組中的所有子句，才能套用同步處理規則。 如果多個群組已經過定義時，則必須至少滿足一個群組才能套用該規則。 也就是說，系統會在群組間評估邏輯 OR ，而在單一群組中評估邏輯 AND。 您可以在輸出同步處理規則 **Out to AAD – Group Join**中找到此組態的範例。 同步處理篩選器有數個群組，例如，一個適用於安全性群組 (`securityEnabled EQUAL True`) 的群組，和一個適用於通訊群組 (`securityEnabled EQUAL False`) 的群組。

![同步處理規則編輯器中的範圍索引標籤 ](./media/concept-azure-ad-connect-sync-default-configuration/syncrulescopingfilterout.png)

此規則是用來定義哪些群組應該佈建至 Azure AD。 通訊群組必須啟用郵件，才可使用 Azure AD 進行同步處理；但是安全性群組不需要電子郵件，即可進行同步處理。

#### <a name="join-rules"></a>聯結規則
第三個區段是用來設定連接器空間中物件與 Metaverse 中物件的關係。 您先前看過的規則中並沒有任何適用於聯結規則的組態，因此您將繼續了解 **In from AD - User Join**。

![同步處理規則編輯器中的聯結規則索引標籤 ](./media/concept-azure-ad-connect-sync-default-configuration/syncrulejoinrules.png)

聯結規則的內容取決於安裝精靈中選取的比對選項。 針對輸入規則，評估作業將會從來源連接器空間中的物件開始進行，接著再評估聯結規則中的各個群組。 如果來源物件經評估確實符合 Metaverse 中的一個物件，且 Metaverse 使用其中一個聯結規則，物件就會聯結起來。 如果所有規則經評估後沒有任何相符項目，則會使用說明頁面上的連結類型。 如果此組態設定為為 [佈建]，則目標 (Metaverse) 中會建立新的物件。 將新物件佈建至 Metaverse，也等同於將物件**投射**至 Metaverse。

只會對聯結規則評估一次。 當連接器空間物件和 Metaverse 物件聯結起來時，只要仍滿足同步處理規則的範圍，兩者就會維持聯結狀態。

評估同步處理規則時，只有一個具有已定義聯結規則的同步處理規則必須在範圍內。 如果發現多個具有聯結規則的同步處理規則用於單一物件，就會擲回錯誤。 基於這個原因，最佳作法就是在多個同步處理規則都在同一個範圍內用於單一物件時，只有一個具有已定義聯結的同步處理規則。 在 Azure AD Connect 同步處理現成可用的組態中，您可以查看規則名稱來找到這些規則，並發現它們的名稱結尾都有 **Join** 一詞。 如果其他同步處理規則將物件聯結在一起，或在目標中佈建新物件，則未定義任何聯結規則的同步處理規則會套用屬性流程。

在檢視上圖時，您可以看到規則嘗試將 **objectSID** 與 **msExchMasterAccountSid** (Exchange) 和 **msRTCSIP-OriginatorSid** (Lync) 聯結，而這正是我們在帳戶資源樹系拓撲中所預期的。 您會發現所有樹系具有相同的規則。 我們可以假設每個樹系可能是帳戶或資源樹系。 如果您有帳戶存在於單一樹系中，且不需要聯結，也適用此組態。

#### <a name="transformations"></a>轉換
轉換區段會定義當物件呈現聯結狀態並滿足範圍篩選器時，會套用至目標物件的所有屬性流程。 回到 **In from AD - User AccountEnabled** 同步處理規則時，您會看見下列轉換：

![同步處理規則編輯器中的轉換索引標籤 ](./media/concept-azure-ad-connect-sync-default-configuration/syncruletransformations.png)

若要將此組態放在內容中，則在帳戶-資源樹系部署中，預計會在帳戶樹系中找到啟用的帳戶，並在具有 Exchange 和 Lync 設定的資源樹系中找到停用的帳戶。 您目前看到的同步處理規則包含登入所需的屬性，而這些屬性應從具有已啟用帳戶的樹系流出。 這些屬性流程會全部放在一個同步處理規則中。

轉換可具有不同類型：Constant、Direct 和 Expression。

* Constant 流程會一律流送硬式編碼值。 在上述例子中，一律會在名為 **accountEnabled** 的 Metaverse 屬性中設定 **True** 值。
* Direct 流程一律會將來源中的屬性值依現狀流動至目標屬性。
* 第三個流程類型是 Expression，可讓您使用更為進階的組態。

由於運算式語言為 VBA (Visual Basic for Applications)，因此具有 Microsoft Office 或 VBScript 使用經驗的人員就能辨認其格式。 屬性會包在方括號之中，例如 [attributeName]。 屬性名稱與函數名稱有區分大小寫，但是當運算式無效時，同步處理規則編輯器會評估運算式並且發出警告。 所有運算式將會以具有巢狀函式的單一行表示。 若要發揮組態語言的功能，請使用插入其他註解的 pwdLastSet 流程：

```
// If-then-else
IIF(
// (The evaluation for IIF) Is the attribute pwdLastSet present in AD?
IsPresent([pwdLastSet]),
// (The True part of IIF) If it is, then from right to left, convert the AD time format to a .Net datetime, change it to the time format used by Azure AD, and finally convert it to a string.
CStr(FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")),
// (The False part of IIF) Nothing to contribute
NULL
)
```

如需屬性流程的運算式語言詳細資訊，請參閱 [了解宣告式佈建運算式](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)

### <a name="precedence"></a>優先順序
您現在已看過一些個別的同步處理規則，但這些規則會在組態中一起運作。 在某些情況下，屬性值會由相同目標屬性的多個同步處理規則提供。 在此情況下，即可使用屬性優先順序來判斷哪個屬性會勝出。 以 sourceAnchor 屬性為例。 此屬性是能否登入 Azure AD 的重要屬性。 您可以在兩個不同的同步處理規則中看到此屬性的屬性流程：**In from AD – User AccountEnabled** 和 **In from AD – User Common**。 由於有同步處理規則優先順序，如果有數個物件聯結至 Metaverse 物件，則會由先具有已啟用帳戶的樹系提供 sourceAnchor 屬性。 如果沒有已啟用的帳戶，則同步處理引擎會使用全部擷取同步處理規則 **In from AD – User Common**。 此組態可確保即使是已停用的帳戶，仍有 sourceAnchor。

![同步處理規則 (輸入)](./media/concept-azure-ad-connect-sync-default-configuration/syncrulesinbound.png)

同步處理規則的優先順序會由安裝精靈設定在群組中。 群組中的所有規則具有相同的名稱，但是會連接到不同的連接目錄。 安裝精靈會將最高優先順序提供給 **In from AD – User Join** 規則，並逐一查看所有連接的 AD 目錄。 接著，它會以預先定義的順序繼續處理後續的規則群組。 在群組中，會以在精靈中新增連接器的順序來新增規則。 如果透過精靈新增其他連接器，則會將同步處理規則重新排序，而新連接器的規則會插入在每個群組中的結尾處。

### <a name="putting-it-all-together"></a>總整理
我們現在對同步處理規則已有足夠的認識，而能夠了解組態如何在不同的同步處理規則下運作。 如果您觀察某個使用者以及提供給 Metaverse 的屬性，則會以下列順序套用規則：

| 名稱 | 註解 |
|:--- |:--- |
| In from AD - User Join |聯結連接器空間物件與 Metaverse 的規則。 |
| In from AD – UserAccount Enabled |登入 Azure AD 和 Office 365 所需的屬性。 我們想從已啟用的帳戶取得這些屬性。 |
| In from AD – User Common from Exchange |在全域通訊清單中找到的屬性。 我們假設在使用者信箱所在的樹系中，具有最佳的資料品質。 |
| In from AD – User Common |在全域通訊清單中找到的屬性。 如果找不到信箱，則可由任何其他聯結物件提供屬性值。 |
| In from AD – User Exchange |只有在偵測到 Exchange 後才存在。 它會流送所有基礎結構 Exchange 屬性。 |
| In from AD – User Lync |只有在偵測到 Lync 後才存在。 它會流送所有基礎結構 Lync 屬性。 |

## <a name="next-steps"></a>後續步驟
* 如需組態模型的詳細資訊，請參閱 [了解宣告式佈建](concept-azure-ad-connect-sync-declarative-provisioning.md)。
* 如需運算式語言的詳細資訊，請參閱 [了解宣告式佈建運算式](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)。
* 如需了解現成可用組態的運作方式，請繼續閱讀 [了解使用者和連絡人](concept-azure-ad-connect-sync-user-and-contacts.md)
* 如需了解如何使用宣告式佈建進行實際變更，請參閱 [如何變更預設組態](how-to-connect-sync-change-the-configuration.md)。

**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)

