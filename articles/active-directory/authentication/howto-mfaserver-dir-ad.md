---
title: Azure Multi-Factor Authentication 和 Active Directory 之間的目錄整合
description: 如何整合 Azure Multi-Factor Authentication Server 與 Active Directory，讓您可以同步處理目錄。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 818674ba1825a438a5abeb7927bcf0f683506625
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159918"
---
# <a name="directory-integration-between-azure-mfa-server-and-active-directory"></a>Azure MFA Server 與 Active Directory 之間的目錄整合

使用 Azure MFA Server 的 [目錄整合] 區段來與 Active Directory 或另一個 LDAP 目錄整合。 您可以設定相關屬性以比對目錄結構描述，並設定自動使用者同步處理。

## <a name="settings"></a>設定
根據預設，Azure Multi-Factor Authentication (MFA) Server 會設定為從 Active Directory 匯入或同步處理使用者。  [目錄整合] 索引標籤可讓您覆寫預設行為，並繫結至不同的 LDAP 目錄、ADAM 目錄或特定 Active Directory 網域控制站。  它也支援使用 LDAP 驗證來代理 LDAP 或 LDAP 繫結做為 RADIUS 目標、IIS 驗證的預先驗證，或使用者入口網站的主要驗證。  下表描述個別設定。

![設定](./media/howto-mfaserver-dir-ad/dirint.png)

| 功能 | 說明 |
| --- | --- |
| 使用 Active Directory |選取 [使用 Active Directory] 選項，以使用 Active Directory 來匯入和同步處理。  這是預設設定。 <br>注意：為了使 Active Directory 整合能正常運作，請將電腦加入網域並以網域帳戶進行登入。 |
| 包含受信任的網域 |核取 [包含受信任的網域]，讓代理程式嘗試連線至目前網域所信任的網域、樹系中的另一個網域，或涉及樹系信任的網域。  當不從任何受信任的網域匯入或同步處理使用者時，請取消選取此核取方塊，以改善效能。  預設為核取。 |
| 使用特定 LDAP 組態 |選取 [使用 LDAP] 選項，以使用指定的 LDAP 設定來匯入和同步處理。 注意：當選取 [使用 LDAP] 時，使用者介面會將參考從 Active Directory 變更為 LDAP。 |
| [編輯] 按鈕 |[編輯] 按鈕允許修改目前的 LDAP 組態設定。 |
| 使用屬性範圍查詢 |指出是否應該使用屬性範圍查詢。  屬性範圍查詢可以根據另一個記錄屬性中的項目，有效率地在目錄中搜尋合格的記錄。  Azure Multi-Factor Authentication Server 可以使用屬性範圍查詢，有效率地查詢安全性群組成員的使用者。   <br>注意：在某些情況下，雖然支援屬性範圍查詢，但卻不應該使用。  比方說，當安全性群組包含來自多個網域的成員時，Active Directory 的屬性範圍查詢可能會有問題。 在此情況下，請取消選取此核取方塊。 |

下表描述 LDAP 組態設定。

| 功能 | 說明 |
| --- | --- |
| 伺服器 |輸入執行 LDAP 目錄之伺服器的主機名稱或 IP 位址。  也可以指定備份伺服器 (以分號隔開)。 <br>注意：當 [繫結類型] 是 [SSL] 時，需要完整主機名稱。 |
| 基準 DN |輸入作為所有目錄查詢起點之基準目錄物件的辨別名稱。  例如，dc=abc,dc=com。 |
| 繫結類型 - 查詢 |選取繫結時可用的適當繫結類型，以搜尋 LDAP 目錄。  這用於匯入、同步處理和使用者名稱解析。 <br><br>  匿名 - 執行匿名繫結。  不會使用繫結 DN 和繫結密碼。  只有當 LDAP 目錄允許匿名繫結，而且權限允許查詢適當的記錄和屬性時，這才有作用。  <br><br> 簡單 - 以純文字傳遞 [繫結 DN] 和 [繫結密碼] 來繫結至 LDAP 目錄。  這只適用於測試目的，以確認可連上伺服器且繫結帳戶具有適當的存取權。 安裝適當的憑證之後，請改用 SSL。  <br><br> SSL - 使用 SSL 加密 [繫結 DN] 和 [繫結密碼] 以繫結至 LDAP 目錄。  在本機安裝 LDAP 目錄所信任的憑證。  <br><br> Windows - 使用繫結使用者名稱和繫結密碼來安全地連線到 Active Directory 網域控制站或 ADAM 目錄。  如果 [繫結使用者名稱] 空白，則會使用已登入的使用者帳戶進行繫結。 |
| 繫結類型 - 驗證 |選取適當的繫結類型供執行 LDAP 繫結驗證時使用。  請參閱＜繫結類型 - 查詢＞下的繫結類型描述。  比方說，這可將匿名繫結用於查詢，而將 SSL 繫結用於保護 LDAP 繫結驗證。 |
| 繫結 DN 或繫結使用者名稱 |輸入帳戶的使用者記錄辨別名稱，以在繫結至 LDAP 目錄時使用。<br><br>只有在 [繫結類型] 為 [簡單] 或 [SSL] 時，才會使用繫結辨別名稱。  <br><br>當 [繫結類型] 為 [Windows] 時，輸入當繫結至 LDAP 目錄時要使用的 Windows 帳戶的使用者名稱。  如果空白，則會使用已登入的使用者帳戶進行繫結。 |
| 繫結密碼 |輸入用來繫結至 LDAP 目錄之繫結 DN 或使用者名稱的繫結密碼。  若要設定 Multi-Factor Auth Server AdSync 服務的密碼，請啟用同步處理並確保此服務在本機電腦上執行。  此密碼會儲存在用於執行 Multi-Factor Auth Server AdSync Service 的帳戶下的 [Windows 儲存的使用者和密碼] 中。  此密碼也會儲存在用於執行 Multi-Factor Auth Server 使用者介面的帳戶下，以及用於執行 Multi-Factor Auth Server Service 的帳戶下。  <br><br>由於密碼只會儲存在本機伺服器的 [Windows 儲存的使用者和密碼] 中，所以在每個需要存取此密碼的 Multi-Factor Auth Server 上重複此步驟。 |
| 查詢大小限制 |指定目錄搜尋可傳回的使用者數目上限。  此限制應該符合 LDAP 目錄上的設定。  對於不支援分頁的大型搜尋，匯入和同步處理會嘗試以批次方式擷取使用者。  如果此處指定的大小限制大於 LDAP 目錄上設定的限制，可能會遺漏部分使用者。 |
| 測試按鈕 |按一下 [測試] 以測試 LDAP 伺服器繫結。  <br><br>您不需要選取 [使用 LDAP] 選項來測試繫結。 這樣可以在使用 LDAP 組態之前先測試繫結。 |

## <a name="filters"></a>篩選器
篩選可讓您在執行目錄搜尋時，設定準則來限定記錄。  您可以設定篩選以限定想要同步處理的物件範圍。  

![篩選器](./media/howto-mfaserver-dir-ad/dirint2.png)

Azure Multi-Factor Authentication 具有下列三個篩選選項：

* **容器篩選** - 指定執行目錄搜尋時，用來限定容器記錄的篩選準則。  對於 Active Directory 和 ADAM，通常會使用 (|(objectClass=organizationalUnit)(objectClass=container))。  對於其他 LDAP 目錄，請根據目錄結構描述，使用可限定每一種容器物件的篩選準則。  <br>注意：如果空白，預設會使用 ((objectClass=organizationalUnit)(objectClass=container))。
* **安全性群組篩選** - 指定執行目錄搜尋時，用來限定安全性群組記錄的篩選準則。  對於 Active Directory 和 ADAM，通常會使用 (&(objectCategory=group)(groupType:1.2.840.113556.1.4.804:=-2147483648))。  對於其他 LDAP 目錄，請根據目錄結構描述，使用可限定每一種安全性群組物件的篩選準則。  <br>注意：如果空白，預設會使用 (&(objectCategory=group)(groupType:1.2.840.113556.1.4.804:=-2147483648))。
* **使用者篩選** - 指定執行目錄搜尋時，用來限定使用者記錄的篩選準則。  對於 Active Directory 和 ADAM，通常會使用 (&(objectClass=user)(objectCategory=person))。  對於其他 LDAP 目錄，請根據目錄結構描述，使用 (objectClass = inetOrgPerson) 或類似的準則。 <br>注意：如果空白，預設會使用 (&(objectCategory=person)(objectClass=user))。

## <a name="attributes"></a>屬性
您可以視需要自訂特定目錄的屬性。  這可讓您新增自訂屬性，並微調到只同步處理您需要的屬性。 使用目錄結構描述中所定義的屬性名稱作為每個屬性欄位的值。 下表提供每項功能的其他相關資訊。

您可以手動輸入屬性，並不需要符合屬性清單中的屬性。

![屬性](./media/howto-mfaserver-dir-ad/dirint3.png)

| 功能 | 說明 |
| --- | --- |
| 唯一識別碼 |輸入屬性的屬性名稱，做為容器、安全性群組和使用者記錄的唯一識別碼。  在 Active Directory 中，這通常是 objectGUID。 其他 LDAP 實作可能使用 entryUUID 或類似的項目。  預設值是 objectGUID。 |
| 唯一識別碼類型 |選取唯一識別碼屬性的類型。  在 Active Directory 中，objectGUID 屬性為 GUID 類型。 其他 LDAP 實作可能使用 ASCII 位元組陣列或字串類型。  預設值是 GUID。 <br><br>務必正確設定此類型，因為同步處理項目是依照其唯一識別碼來參考。 唯一識別碼類型是用來直接在目錄中尋找物件。  當目錄實際上將值儲存為 ASCII 字元的位元組陣列時，將此類型設定為 [字串] 會導致同步處理無法正確運作。 |
| 辨別名稱 |輸入屬性的屬性名稱，此屬性包含每一筆記錄的辨別名稱。  在 Active Directory 中，這通常是 distinguishedName。 其他 LDAP 實作可能使用 entryDN 或類似的項目。  預設值是 distinguishedName。 <br><br>如果不存在只包含辨別名稱的屬性，可以使用 adspath 屬性。  路徑的 "LDAP://\<server\>/" 部分將會自動去除，只保留物件的辨別名稱。 |
| 容器名稱 |輸入屬性的屬性名稱，此屬性包含容器記錄中的名稱。  從 Active Directory 匯入或新增同步處理項目時，這個屬性的值會顯示在容器階層中。  預設值是 name。 <br><br>如果不同容器使用不同的屬性做為名稱，請使用分號來隔開多個容器名稱屬性。  在容器物件上找到的第一個容器名稱屬性用來顯示其名稱。 |
| 安全性群組名稱 |輸入屬性的屬性名稱，此屬性包含安全性群組記錄中的名稱。  從 Active Directory 匯入或新增同步處理項目時，這個屬性的值會顯示在安全性群組清單中。  預設值是 name。 |
| 使用者名稱 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的使用者名稱。  這個屬性的值用於做為 Multi-Factor Auth Server 使用者名稱。  可以指定第二個屬性做為第一個屬性的備用。  只有當第一個屬性不包含使用者的值時，才會使用第二個屬性。  預設值是 userPrincipalName 和 sAMAccountName。 |
| 名字 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的名字。  預設值是 givenName。 |
| 姓氏 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的姓氏。  預設值是 sn。 |
| 電子郵件地址 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的電子郵件地址。  電子郵件地址用來傳送歡迎及更新電子郵件給使用者。  預設值是 mail。 |
| 使用者群組 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的使用者群組。  使用者群組可在 Multi-Factor Auth Server 管理入口網站中用來篩選代理程式和報告中的使用者。 |
| 說明 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的描述。  描述只用於搜尋。  預設值是 description。 |
| 通話語言 |輸入屬性的屬性名稱，此屬性包含用於使用者語音通話的語言簡稱。 |
| 簡訊語言 |輸入屬性的屬性名稱，此屬性包含用於使用者簡訊的語言簡稱。 |
| 行動應用程式語言 |輸入屬性的屬性名稱，此屬性包含用於使用者電話應用程式簡訊的語言簡稱。 |
| OATH 權杖語言 |輸入屬性的屬性名稱，此屬性包含用於使用者 OATH 權杖簡訊的語言簡稱。 |
| 商務電話 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的商務電話號碼。  預設值是 telephoneNumber。 |
| 住家電話 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的住家電話號碼。  預設值是 homePhone。 |
| 呼叫器 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的呼叫器號碼。  預設值是 pager。 |
| 行動電話 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的行動電話號碼。  預設值是 mobile。 |
| 傳真 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的傳真號碼。  預設值是 facsimileTelephoneNumber。 |
| IP 電話 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的 IP 電話號碼。  預設值是 ipPhone。 |
| 自訂 |輸入屬性的屬性名稱，其中包含使用者記錄中的自訂電話號碼。  預設值為空白。 |
| 分機 |輸入屬性的屬性名稱，此屬性包含使用者記錄中的分機電話號碼。  分機欄位的值只會做為主要電話號碼的分機。  預設值為空白。 <br><br>如果不指定分機屬性，則可以在電話屬性中包含分機。 在此情況下，在分機前面加上 'x'，便可得到正確剖析。  例如，555-123-4567 x890 會形成電話號碼 555-123-4567 和分機 890。 |
| [還原預設值] 按鈕 |按一下 [還原預設值]，可將所有屬性還原為預設值。  在一般 Active Directory 或 ADAM 結構描述中，預設值應該可以正確運作。 |

若要編輯屬性，按一下 [屬性] 索引標籤上的 [編輯] 按鈕。這會帶出您可以在其中編輯屬性的視窗。 選取任何屬性旁邊的 [...] 可開啟視窗供您選擇要顯示的屬性。

![編輯屬性](./media/howto-mfaserver-dir-ad/dirint4.png)

## <a name="synchronization"></a>同步處理
同步處理可將 Azure MFA 使用者資料庫與 Active Directory 或另一個輕量型目錄存取通訊協定 (LDAP) 目錄中的使用者保持同步。 此程序類似於從 Active Directory 手動匯入使用者，但會定期輪詢 Active Directory 使用者和安全性群組是否有需要處理的變更。  它也可停用或移除已從容器、安全性群組或 Active Directory 中移除的使用者。

Multi-Factor Auth ADSync 服務是會定期輪詢 Active Directory 的 Windows 服務。  請勿與 Azure AD Sync 或 Azure AD Connect 搞混。  Multi-Factor Auth ADSync 雖然是根據類似的程式碼基底建置，但為 Azure Multi-Factor Authentication Server 所專用。  它會安裝成「已停止」狀態，在設定為執行時才由 Multi-Factor Auth Server 服務啟動。  如果您有多伺服器 Multi-Factor Auth Server 組態，Multi-Factor Auth ADSync 只能在單一伺服器上執行。

Multi-Factor Auth ADSync 服務使用 Microsoft 所提供的 DirSync LDAP 伺服器擴充功能，有效率地輪詢變更。  此 DirSync 控制呼叫端必須具有「目錄變更」權限和 DS-Replication-Get-Changes 擴充控制存取權限。  根據預設，這些權限指派給網域控制站上的 Administrator 和 LocalSystem 帳戶。  根據預設，Multi-Factor Auth AdSync 服務設定為以 LocalSystem 身分執行。  因此，在網域控制站上執行此服務最簡單。  如果您將此服務設定為一律執行完整同步處理，此服務可以用權限較少的帳戶執行。  這樣比較沒有效率，但需要的帳戶權限較少。

如果 LDAP 目錄支援 DirSync 並為其進行設定，則輪詢使用者和安全性群組的變更時，就如同輪詢 Active Directory 的變更一樣。  如果 LDAP 目錄不支援 DirSync 控制，則會在每個週期執行完整同步處理。

![同步處理](./media/howto-mfaserver-dir-ad/dirint5.png)

下表包含各項 [同步處理] 索引標籤設定的其他資訊。

| 功能 | 說明 |
| --- | --- |
| 啟用與 Active Directory 同步處理 |核取時，Multi-Factor Auth Server 服務會定期輪詢 Active Directory 的變更。 <br><br>注意：必須加入至少一個同步處理項目，且必須執行「立即同步處理」，Multi-Factor Auth Server 服務才會開始處理變更。 |
| 同步處理頻率 |指定 Multi-Factor Auth Server 服務在輪詢和處理變更之間等候的時間間隔。 <br><br> 注意：指定的間隔是每個週期開始之間的時間。  如果處理變更的時間超過間隔，服務將立即再次輪詢。 |
| 移除已不在 Active Directory 中的使用者 |核取時，Multi-Factor Auth Server 服務會處理 Active Directory 刪除的使用者標記，並移除相關的 Multi-Factor Auth Server 使用者。 |
| 永遠執行完整同步處理 |核取時，Multi-Factor Auth Server 服務將一律執行完整同步處理。  取消核取時，Multi-Factor Auth Server 服務只會查詢已變更的使用者來執行增量同步處理。  預設為核取。 <br><br>取消核取時，只有當目錄支援 DirSync 控制，且繫結至目錄的帳戶具有執行 DirSync 增量查詢的權限時，Azure MFA Server 才能執行增量同步處理。  如果帳戶沒有適當的權限，或有多個網域涉及同步處理，則 Azure MFA Server 會執行完整同步處理。 |
| 超過 X 個使用者將停用或移除時，需要系統管理員核准 |同步處理項目可以設定為停用或移除不再是項目的容器或安全性群組成員的使用者。  以防萬一，當停用或移除的使用者數目超過臨界值時，可能需要系統管理員核准。  核取時，超過指定的臨界值就需要核准。  預設值為 5，範圍從 1 到 999。 <br><br> 核准之前會先傳送電子郵件通知給系統管理員。 電子郵件通知提供指示來檢閱及核准停用和移除使用者。  Multi-Factor Auth Server 使用者介面啟動時會提示要求核准。 |

[ **立即同步處理** ] 按鈕可讓您針對指定的同步處理項目，執行完整同步處理。  每當加入、修改、移除或重新排列同步處理項目時，就需要完整同步處理。  也需要如此，Multi-Factor Auth AdSync 服務才能運作，因為它會設定服務輪詢增量變更的起始點。  如果同步處理項目已變更，但尚未執行完整同步處理，系統將會提示您「立即同步處理」。

[ **移除** ] 按鈕可讓系統管理員從 Multi-Factor Auth Server 同步處理項目清單刪除一個或多個同步處理項目。

> [!WARNING]
> 一旦移除同步處理項目記錄就無法復原。 如果不小心刪除，則必須再次新增同步處理項目記錄。

同步處理項目已從 Multi-Factor Auth Server 中移除。  Multi-Factor Auth Server 服務將不再處理這些同步處理項目。

[上移] 和 [下移] 按鈕可讓系統管理員變更同步處理項目的順序。  順序很重要，因為相同的使用者可能是多個同步處理項目 (例如容器和安全性群組) 的成員。  在同步處理期間套用至使用者的設定，將來自使用者相關聯的清單中的第一個同步處理項目。  因此，同步處理項目應該依優先順序放置。

> [!TIP]
> 移除同步處理項目之後應該執行完整同步處理。  排序同步處理項目之後應該執行完整同步處理。  按一下 [立即同步處理] 以執行完整同步處理。

## <a name="multi-factor-auth-servers"></a>Multi-Factor Auth Server
可以設定其他 Multi-Factor Auth Server 做為備份 RADIUS Proxy、LDAP Proxy 或是用於 IIS 驗證。 所有代理程式會共用同步處理組態。 但其中只有一個代理程式會執行 Multi-Factor Auth Server 服務。 此索引標籤可讓您選取應該啟用同步處理的 Multi-Factor Auth Server。

![Multi-Factor-Auth Server](./media/howto-mfaserver-dir-ad/dirint6.png)
