---
title: Azure AD Connect 同步：設定篩選 | Microsoft Docs
description: 說明如何在 Azure AD Connect 同步處理中設定篩選。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 880facf6-1192-40e9-8181-544c0759d506
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 9ec136b418e78f82486d9d38f361e411c3d00c31
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306357"
---
# <a name="azure-ad-connect-sync-configure-filtering"></a>Azure AD Connect 同步處理：設定篩選
使用篩選功能可讓您控制內部部署目錄中的哪些物件應該出現在 Azure Active Directory 中。 預設組態會擷取所設定樹系中所有網域內的所有物件。 一般會建議使用者使用這個組態。 完整的全域通訊清單對於使用 Exchange Online 和商務用 Skype 等 Office 365 工作負載的使用者來說十分方便，因為如此一來，他們就可以傳送電子郵件和呼叫每個人。 使用預設設定時，所獲得的體驗與使用 Exchange 或 Lync 的內部部署實作相同。

但是，您有時必須對預設組態進行一些變更。 這裡有一些範例：

* 您打算使用[多重 Azure AD 目錄拓撲](plan-connect-topologies.md#each-object-only-once-in-an-azure-ad-tenant)。 然後，您需要套用篩選器，以控制要將哪些物件同步至特定 Azure AD 目錄。
* 您要執行 Azure 或 Office 365 試驗，因此只想要使用 Azure AD 中的部分使用者。 在進行小規模試驗時，並不需要用到完整的全域通訊清單來展示功能。
* Azure AD 中有很多您不需要的服務帳戶和其他非個人帳戶。
* 為了符合法規，您不能刪除任何內部部署的使用者帳戶。 你只能停用它們。 但您只想要顯示 Azure AD 內的使用中帳戶。

本文介紹如何設定不同的篩選方法。

> [!IMPORTANT]
> Microsoft 不支援在正式記載的動作以外修改和操作 Azure AD Connect 同步處理。 任何這些動作都可能會導致 Azure AD Connect 同步處理的不一致或不受支援狀態。如此一來，Microsoft 無法提供這類部署的技術支援人員。

## <a name="basics-and-important-notes"></a>基本概念和重要事項
在 Azure AD Connect 同步處理中，您隨時都能啟用篩選功能。 如果您一開始是使用目錄同步作業的預設組態，接著設定了篩選，則篩選出的物件就不會再同步處理至 Azure AD。 因為這項變更，系統會在 Azure AD 中，刪除 Azure AD 中先前已同步處理但接著篩選出的所有物件。

在開始變更篩選之前，請確定您已 [停用排程的工作](#disable-the-scheduled-task) ，如此才不會意外匯出尚未確認是否正確的變更。

因為篩選後會同時移除許多的物件，您必須先確定新的篩選器正確無誤，然後再開始將變更匯出至 Azure AD。 在完成組態設定步驟後，建議您一定要先按照 [驗證步驟](#apply-and-verify-changes) 中的指示執行過一次，然後才對 Azure AD 進行匯出和變更作業。

為了避免您意外刪除許多物件，預設會開啟「[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)」功能。 如果因為進行篩選而刪除了許多物件 (預設是 500 個)，您需要遵循本文中的步驟，允許將刪除結果傳播至 Azure AD。

如果您使用 2015 年 11 月 ([1.0.9125](reference-connect-version-history.md#1091250)) 之前的組建、變更篩選組態並使用密碼雜湊同步處理，則在完成組態設定之後，必須觸發所有密碼的完整同步處理。 如需如何觸發密碼的完整同步處理的步驟，請參閱 [觸發所有密碼的完整同步處理](tshoot-connect-password-hash-synchronization.md#trigger-a-full-sync-of-all-passwords)。 如果您使用組建 1.0.9125 或更新版本，則一般的 **完整同步處理** 動作也會計算是否應同步處理密碼，以及是否不再需要進行這個額外步驟。

如果在 Azure AD 中， **使用者** 物件因為篩選錯誤而遭到意外刪除，您可以在 Azure AD 中重新建立使用者物件，方法是移除您的篩選組態。 然後您可以再次同步處理您的目錄。 這個動作會還原 Azure AD 資源回收筒中的使用者。 不過，您無法取消刪除其他物件類型。 例如，如果您意外刪除安全性群組，而該群組是用來對資源進行 ACL，則無法復原群組和其 ACL。

Azure AD Connect 只會刪除其曾經認為是在範圍內的物件。 如果 Azure AD 中有物件是由另一個同步處理引擎所建立且不在範圍內，則新增篩選並不會移除這些物件。 例如，如果您一開始是使用建立了整個 Azure AD 目錄完整複本的 DirSync 伺服器，然後您在從一開始便啟用篩選的情況下平行安裝新的 Azure AD Connect 同步處理伺服器，Azure AD Connect 將不會移除 DirSync 所建立的額外物件。

當您安裝或升級至較新版本的 Azure AD Connect 時，篩選組態將會保留。 在升級至較新版本之後且在首次執行同步處理循環之前，最好一律先確認設定並未遭到意外變更。

如果您有多個樹系，則您必須套用本主題中所說的篩選組態至每個樹系 (假設您想要讓所有樹系使用相同組態)。

### <a name="disable-the-scheduled-task"></a>停用排程的工作
若要停用每 30 分鐘觸發一次同步處理週期的內建排程器，請遵循下列步驟：

1. 移至 PowerShell 提示字元。
2. 執行 `Set-ADSyncScheduler -SyncCycleEnabled $False` 停用排程器。
3. 依本文章中所述進行變更。
4. 執行 `Set-ADSyncScheduler -SyncCycleEnabled $True` 重新啟用排程器。

**如果您使用 1.1.105.0 之前的 Azure AD Connect 組建**  
若要停用每三個小時觸發一次同步處理作業的排定工作，請遵循下列步驟：

1. 從 [開始] 功能表啟動 [工作排程器]。
2. 在 [工作排程器程式庫] 正下方尋找名稱為 [Azure AD 同步排程器] 的工作，在該工作上按一下滑鼠右鍵，然後選取 [停用]。  
   ![](./media/how-to-connect-sync-configure-filtering/taskscheduler.png)  
3. 您現在可以從 **Synchronization Service Manager** 主控台進行組態變更和手動執行同步處理引擎。

完成所有篩選變更之後，別忘了返回重新 [啟用] 工作。

## <a name="filtering-options"></a>篩選選項
您可以將以下篩選組態類型套用至目錄同步處理工具：

* [**群組型**](#group-based-filtering)：您只能在使用安裝精靈進行初始安裝時，設定以單一群組為基礎的篩選。
* [**網域型**](#domain-based-filtering)：使用此選項可讓您選取要將哪些網域同步到 Azure AD。 當您在安裝 Azure AD Connect 同步處理之後對內部部署基礎結構進行變更，您也可以從同步處理引擎組態新增和移除網域。
* [**組織單位 (OU) 型**](#organizational-unitbased-filtering)：使用此選項可讓您選取要將哪些 OU 同步到 Azure AD。 此選項會套用在所選組織單位中的所有物件類型上。
* [**屬性型**](#attribute-based-filtering)：使用此選項可讓您根據物件上的屬性值來篩選物件。 您也可以讓不同物件類型透用不同篩選器。

您可以同時使用多個篩選選項。 例如，您可以使用組織單位型篩選僅將物件包含在一個 OU 中。 在此同時，您可以使用屬性型篩選來篩選物件。 當您使用多個篩選方法時，篩選器之間會使用邏輯 "AND"。

## <a name="domain-based-filtering"></a>網域型篩選
本節提供您設定網域篩選的步驟。 如果您在安裝 Azure AD Connect 之後新增或移除樹系中的網域，也必須更新篩選組態。

變更網域型篩選的慣用方法是執行安裝精靈，然後變更[網域與 OU 篩選](how-to-connect-install-custom.md#domain-and-ou-filtering)。 安裝精靈將會自動執行本主題中記載的所有工作。

您應該只有在因某些原因而無法執行安裝精靈時，才依照下列步驟進行操作。

網域型篩選組態包含下列步驟：

1. [選取網域](#select-domains-to-be-synchronized)，這些是您要納入同步處理作業的網域。
2. 針對所新增和移除的每個網域，調整 [執行設定檔](#update-run-profiles)。
3. [套用並驗證變更](#apply-and-verify-changes)。

### <a name="select-the-domains-to-be-synchronized"></a>選取要同步處理的網域
若要設定網域篩選，請執行下列步驟：

1. 使用隸屬於 **ADSyncAdmins** 安全性群組的帳戶，登入執行 Azure AD Connect 同步處理的伺服器。
2. 從 [開始] 功能表啟動 [同步處理服務]。
3. 選取 [連接器]，然後在 [連接器] 清單中，選取類型為 [Active Directory Domain Services] 的連接器。 在 [動作] 中選取 [屬性]。  
   ![連接器屬性](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. 按一下 [設定目錄分割] 。
5. 在 [選取目錄分割] 清單中，視需要選取和取消選取網域。 確認只選取了您想要同步處理的分割。  
   ![分割數](./media/how-to-connect-sync-configure-filtering/connectorpartitions.png)  
   如果您變更了內部部署 Active Directory 基礎結構並在樹系中新增或移除網域，則請按一下 [重新整理] 按鈕以取得更新後的清單。 當您重新整理時，系統會要求您提供認證。 請提供具有內部部署 Windows Server Active Directory 讀取權限的任何認證。 您不一定要使用對話方塊中預先填入的使用者。  
   ![需要重新整理](./media/how-to-connect-sync-configure-filtering/refreshneeded.png)  
6. 當您完成時，請按一下 [確定] 來關閉 [屬性] 對話方塊。 如果您已移除樹系中的網域，畫面上將會出現快顯訊息，指出已移除網域且將會清除組態。
7. 繼續調整 [執行設定檔](#update-run-profiles)。

### <a name="update-the-run-profiles"></a>更新執行設定檔
如果您已更新網域篩選，則也需更新執行設定檔。

1. 在 [連接器] 清單中，確定已選取在上一個步驟中所變更的連接器。 在 [動作] 中選取 [設定執行設定檔]。  
   ![連接器執行設定檔 1](./media/how-to-connect-sync-configure-filtering/connectorrunprofiles1.png)  
2. 尋找及識別下列設定檔：
    * 完整匯入
    * 完整同步處理
    * 差異匯入
    * 差異同步處理
    * 匯出
3. 對於每個設定檔，調整**已新增**與**已移除**的網域。
    1. 針對上述五個設定檔，請為每個**新增的**網域執行下列步驟：
        1. 選取執行設定檔，然後按一下 [新增步驟] 。
        2. 在 [設定步驟] 頁面上的 [類型] 下拉式選單中，選取與所要設定之設定檔同名的步驟類型。 然後按 [下一步] 。  
        ![連接器執行設定檔 2](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep1.png)  
        3. 在 [連接器組態] 頁面上的 [分割] 下拉式選單中，選取您已新增至網域篩選的網域名稱。  
        ![連接器執行設定檔 3](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep2.png)  
        4. 若要關閉 [設定執行設定檔] 對話方塊，請按一下 [完成]。
    2. 針對上述五個設定檔，請為每個**移除的**網域執行下列步驟：
        1. 選取執行設定檔。
        2. 如果 [分割] 屬性的 [值] 是 GUID，請選取執行步驟，然後按一下 [刪除步驟]。  
        ![連接器執行設定檔 4](./media/how-to-connect-sync-configure-filtering/runprofilesdeletestep.png)  
    3. 驗證您的變更。 想要同步處理的每個網域應該皆已列為每個執行設定檔中的步驟。
4. 若要關閉 [設定執行設定檔] 對話方塊，請按一下 [確定]。
5.  若要完成組態，您必須執行「完整匯入」與「差異同步處理」。繼續閱讀[套用並驗證變更](#apply-and-verify-changes)一節。

## <a name="organizational-unitbased-filtering"></a>組織單位型篩選
變更 OU 型篩選的慣用方法是執行安裝精靈，然後變更[網域與 OU 篩選](how-to-connect-install-custom.md#domain-and-ou-filtering)。 安裝精靈將會自動執行本主題中記載的所有工作。

您應該只有在因某些原因而無法執行安裝精靈時，才依照下列步驟進行操作。

若要設定組織單位型篩選，請執行下列步驟：

1. 使用隸屬於 **ADSyncAdmins** 安全性群組的帳戶，登入執行 Azure AD Connect 同步處理的伺服器。
2. 從 [開始] 功能表啟動 [同步處理服務]。
3. 選取 [連接器]，然後在 [連接器] 清單中，選取類型為 [Active Directory Domain Services] 的連接器。 在 [動作] 中選取 [屬性]。  
   ![連接器屬性](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. 按一下 [設定目錄分割]、選取要設定的網域，然後按一下 [容器]。
5. 出現提示時，請提供具有內部部署 Active Directory 讀取權限的任何認證。 您不一定要使用對話方塊中預先填入的使用者。
6. 在 [選取容器] 對話方塊中，取消選取您不想與雲端目錄同步處理的 OU，然後按一下 [確定]。  
   ![在 [選取容器] 對話方塊中的 OU](./media/how-to-connect-sync-configure-filtering/ou.png)  
   * 應該選取 [電腦]  容器，這樣您的 Windows 10 電腦才能順利同步處理至 Azure AD。 如果加入網域的電腦位於其他組織單位，請確定已選取這些電腦。
   * 如果您有多個信任的樹系，則應該選取 [ForeignSecurityPrincipals]  容器。 這個容器允許解析跨樹系安全性群組成員資格。
   * 如果您已啟用裝置回寫功能，則應該選取 [RegisteredDevices] OU。 如果您使用另一個回寫功能，例如群組回寫，請確定已選取這些位置。
   * 選取使用者、iNetOrgPersons、群組、連絡人和電腦所在位置的其他 OU。 在圖中，這些 OU 全都位於 ManagedObjects OU。
   * 如果您使用群組型篩選，則必須包含群組所屬的 OU。
   * 請注意，在篩選設定完成後，您可以設定要不要同步處理新增的新 OU。 如需詳細資料，請參閱下一節。
7. 當您完成時，請按一下 [確定] 來關閉 [屬性] 對話方塊。
8. 若要完成組態，您必須執行「完整匯入」與「差異同步處理」。繼續閱讀[套用並驗證變更](#apply-and-verify-changes)一節。

### <a name="synchronize-new-ous"></a>同步處理新 OU
依預設，設定篩選之後，會同步處理建立的新 OU。 此狀態是以所選取的核取方塊表示。 您也可以取消選取某些子 OU。 做法是按一下方塊，直到它變成白色方塊且其中有藍色打勾記號 (其預設狀態)。 然後取消選取任何您不想同步處理的子 OU。

如果所有子 OU 皆同步處理，該方塊會呈白色且其中有藍色打勾記號。  
![所有方塊皆選取的 OU](./media/how-to-connect-sync-configure-filtering/ousyncnewall.png)

如果取消選取部分子 OU，則該方塊會呈現有白色打勾記號的灰色方塊。  
![部分子 OU 未選取的 OU](./media/how-to-connect-sync-configure-filtering/ousyncnew.png)

使用此設定時，在 ManagedObjects 之下建立的新 OU 會同步處理。

Azure AD Connect 安裝精靈一律會建立此設定。

### <a name="dont-synchronize-new-ous"></a>不要同步處理新 OU
在完成篩選設定後，您可以設定同步引擎不要同步處理新 OU。 在使用者介面中，此狀態是以無打勾記號的灰色實心方塊表示。 做法是按一下方塊，直到它變成無打勾記號的白色方塊。 然後選取任何您想要同步處理的子 OU。

![根 OU 未選取](./media/how-to-connect-sync-configure-filtering/oudonotsyncnew.png)

使用此設定時，在 ManagedObjects 之下建立的新 OU 不會同步處理。

## <a name="attribute-based-filtering"></a>屬性型篩選
請確定您是使用 2015 年 11 月 ([1.0.9125](reference-connect-version-history.md#1091250)) 或更新版本的組建，這些步驟才有作用。

屬性型篩選是最具彈性的物件篩選方式。 您可以使用[宣告式佈建](concept-azure-ad-connect-sync-declarative-provisioning.md)的強大功能來控制物件同步處理至 Azure AD 時的大多數層面。

您可以從 Active Directory 將[輸入](#inbound-filtering)篩選套用到 Metaverse，也可以從 Metaverse 將[輸出](#outbound-filtering)篩選套用到 Azure AD。 我們建議您套用輸入篩選，因為這是最容易的維護方式。 只有在必須先加入多個樹系中的物件再進行評估時，才應該使用輸出篩選。

### <a name="inbound-filtering"></a>輸入篩選
輸入篩選會使用預設組態，亦即，即將輸入 Azure AD 的物件必須未將 Metaverse 屬性 cloudFiltered 設定為要同步處理的值。 如果這個屬性的值設定為 **True**，則不會同步處理物件。 就設計而言，此值不應設為 **False**。 為了確保其他規則能夠提供值，這個屬性的值應該只能是 **True** 或 **NULL** (不存在)。

在輸入篩選中，我們將利用「範圍」的強大功能來決定哪些物件應該或不應該同步處理。 您要在這裡進行調整以符合貴組織的需求。 範圍模組包含「群組」和「子句」，可用來決定何時要將某個同步規則納入範圍中。 「群組」會包含一個或多個「子句」。 多個子句之間會有邏輯 "AND"，而多個群組之間會有邏輯 "OR"。

讓我們看看以下範例：  
![範圍](./media/how-to-connect-sync-configure-filtering/scope.png)  
這應該解讀為 **(department = IT) OR (department = Sales AND c = US)**。

在下列範例和步驟中，您將以使用者物件做為例子，但您可以將此例子套用到所有物件類型。

在下列範例中，優先順序值以 50 為開頭。 這可以是任何未使用的數字，但是應該小於 100。

#### <a name="negative-filtering-do-not-sync-these"></a>負面篩選：「不同步處理這些項目」
在下列範例中，您會篩除 (不同步處理) **extensionAttribute15** 的值為 **NoSync** 的所有使用者。

1. 使用隸屬於 **ADSyncAdmins** 安全性群組的帳戶，登入執行 Azure AD Connect 同步處理的伺服器。
2. 從 [開始] 功能表啟動 [同步處理規則編輯器]。
3. 確定已選取 [輸入]，然後按一下 [新增規則]。
4. 為規則提供一個描述性名稱，例如 "*In from AD – User DoNotSyncFilter*"。 選取正確的樹系，亦即選取 [使用者] 作為 [CS 物件類型]，以及選取 [人員] 作為 [MV 物件類型]。 在 [連結類型] 中，選取 [聯結]。 在 [優先順序] 中，輸入目前沒有被其他「同步化規則」使用的值 (例如 50)，然後按 [下一步]。  
   ![輸入 1 描述](./media/how-to-connect-sync-configure-filtering/inbound1.png)  
5. 在 [範圍設定篩選] 中，按一下 [新增群組]，然後按一下 [新增子句]。 在 [屬性] 中，選取 [ExtensionAttribute15]。 確定已將 [運算子] 設為 [EQUAL]，然後在 [值] 方塊中輸入值 **NoSync**。 按 [下一步] 。  
   ![輸入 2 範圍](./media/how-to-connect-sync-configure-filtering/inbound2.png)  
6. 將 [聯結] 規則保留空白，然後按 [下一步]。
7. 按一下 [新增轉換]、選取 [FlowType] 做為 [常數]，選取 [cloudFiltered] 做為 [目標屬性]。 在 [來源] 文字方塊中，輸入 **True**。 按一下 [新增]  以儲存規則。  
   ![輸入 3 轉換](./media/how-to-connect-sync-configure-filtering/inbound3.png)
8. 若要完成組態，您必須執行「完整同步處理」。繼續閱讀[套用並驗證變更](#apply-and-verify-changes)一節。

#### <a name="positive-filtering-only-sync-these"></a>正面篩選：「只同步處理這些項目」
表述正面篩選的程序比較困難，因為您必須同時考慮不是明顯需要同步處理的物件，例如會議室。 您也將覆寫全新規則 [In from AD - User Join] 中的預設篩選器。 當您建立自訂篩選時，請確定不包含 Azure AD connect 的關鍵性系統物件、複寫衝突物件、特殊信箱和服務帳戶。

正面篩選選項需要兩個同步處理規則。 您需要一個要同步處理之物件的正確範圍規則 (或多個)。 您也需要篩選出尚未被識別為應進行同步處理物件之所有物件的第二個全面涵蓋同步處理規則。

在下列範例中，您只會同步處理部門屬性值為 **Sales**的使用者物件。

1. 使用隸屬於 **ADSyncAdmins** 安全性群組的帳戶，登入執行 Azure AD Connect 同步處理的伺服器。
2. 從 [開始] 功能表啟動 [同步處理規則編輯器]。
3. 確定已選取 [輸入]，然後按一下 [新增規則]。
4. 為規則提供一個描述性名稱，例如 "*In from AD – User Sales sync*"。 選取正確的樹系，亦即選取 [使用者] 作為 [CS 物件類型]，以及選取 [人員] 作為 [MV 物件類型]。 在 [連結類型] 中，選取 [聯結]。 在 [優先順序] 中，輸入目前沒有被其他「同步化規則」使用的值 (例如 51)，然後按 [下一步]。  
   ![輸入 4 描述](./media/how-to-connect-sync-configure-filtering/inbound4.png)  
5. 在 [範圍設定篩選] 中，按一下 [新增群組]，然後按一下 [新增子句]。 在 [屬性] 中，選取 [部門]。 確定已將 [運算子] 設為 [EQUAL]，然後在 [值] 方塊中輸入值 **Sales**。 按 [下一步] 。  
   ![輸入 5 範圍](./media/how-to-connect-sync-configure-filtering/inbound5.png)  
6. 將 [聯結] 規則保留空白，然後按 [下一步]。
7. 按一下 [新增轉換]、選取 [常數] 做為 [FlowType]，選取 [cloudFiltered] 做為 [目標屬性]。 在 [來源] 方塊中，輸入 **False**。 按一下 [新增]  以儲存規則。  
   ![輸入 6 轉換](./media/how-to-connect-sync-configure-filtering/inbound6.png)  
   這是一個特殊案例，在此您會將 cloudFiltered 明確設定為 **False**。
8. 我們現在必須建立全面涵蓋同步處理規則。 為規則提供一個描述性名稱，例如 "*In from AD – User Catch-all filter*"。 選取正確的樹系，亦即選取 [使用者] 作為 [CS 物件類型]，以及選取 [人員] 作為 [MV 物件類型]。 在 [連結類型] 中，選取 [聯結]。 在 [優先順序] 中，輸入目前沒有被其他「同步化規則」使用的值 (例如 99)。 您已選取高於 (較低優先順序) 先前同步處理規則的優先順序值。 但是，您也留一些空間，這樣當您稍後要開始同步處理其他部門時，可以新增更多篩選同步處理規則。 按 [下一步] 。  
   ![輸入 7 描述](./media/how-to-connect-sync-configure-filtering/inbound7.png)  
9. 將 [範圍設定篩選] 保留空白，然後按 [下一步]。 空白篩選器表示規則會套用至所有物件。
10. 將 [聯結] 規則保留空白，然後按 [下一步]。
11. 按一下 [新增轉換]、選取 [常數] 做為 [FlowType]，選取 [cloudFiltered] 做為 [目標屬性]。 在 [來源] 方塊中，輸入 **True**。 按一下 [新增]  以儲存規則。  
    ![輸入 3 轉換](./media/how-to-connect-sync-configure-filtering/inbound3.png)  
12. 若要完成組態，您必須執行「完整同步處理」。繼續閱讀[套用並驗證變更](#apply-and-verify-changes)一節。

如有需要，您可以建立更多第一種類型的規則，以在同步處理作業中納入更多物件。

### <a name="outbound-filtering"></a>輸出篩選
在某些情況下，只有在 Metaverse 中聯結物件之後，才需進行篩選。 例如，可能需要從資源樹系的 mail 屬性以及帳戶樹系的 userPrincipalName 屬性，來判斷是否應同步處理物件。 在這些情況下，您將在輸出規則上建立篩選。

在此範例中，您將變更篩選，讓系統只同步處理 mail 和 userPrincipalName 的結尾都是 @contoso.com 的使用者：

1. 使用隸屬於 **ADSyncAdmins** 安全性群組的帳戶，登入執行 Azure AD Connect 同步處理的伺服器。
2. 從 [開始] 功能表啟動 [同步處理規則編輯器]。
3. 在 [規則類型] 下方按一下 [輸出]。
4. 根據您使用的 Connect 版本，請尋找名為 **Out to AAD – User Join** 或 **Out to AAD-User Join SOAInAD** 的規則，然後按一下 [編輯]。
5. 在快顯視窗中，回答 [是]  來建立規則的複本。
6. 在 [描述] 頁面上，將 [優先順序] 變更為一個未使用的值，例如 50。
7. 按一下左邊導覽列上的 [範圍設定篩選]，然後按一下 [新增子句]。 在 [屬性] 中，選取 [郵件]。 在 [運算子] 中，選取 [ENDSWITH]。 在 [值] 中，輸入 **@contoso.com**，然後按一下 [新增子句]。 在 [屬性] 中，選取 [userPrincipalName]。 在 [運算子] 中，選取 [ENDSWITH]。 在 [值] 中，輸入 **@contoso.com**。
8. 按一下 [檔案] 。
9. 若要完成組態，您必須執行「完整同步處理」。繼續閱讀[套用並驗證變更](#apply-and-verify-changes)一節。

## <a name="apply-and-verify-changes"></a>套用並驗證變更
在變更組態後，必須將這些變更套用至系統中已有的物件。 情況也可能是目前不在同步處理引擎中的物件應受到處理，因此同步處理引擎需要再次讀取來源系統，以確認其內容。

如果您已使用「網域」或「組織單位」篩選來變更組態，您就必須在執行「完整匯入」之後執行「差異同步處理」。

如果您已使用「屬性」篩選來變更組態，則必須執行「完整同步處理」。

請執行下列步驟：

1. 從 [開始] 功能表啟動 [同步處理服務]。
2. 選取 [連接器]。 在 [連接器] 清單中，選取稍早進行過組態變更的連接器。 在 [動作] 中，選取 [執行]。  
   ![連接器執行](./media/how-to-connect-sync-configure-filtering/connectorrun.png)  
3. 在 [執行設定檔] 中，選取上一節中提到的作業。 如果您需要執行兩個動作，請於第一個完成之後再執行第二個。 (所選取連接器的 [狀態] 資料行是 [閒置]。)

在進行過同步處理後，所有變更會進入匯出階段。 實際在 Azure AD 中進行變更之前，您會想要先驗證所有變更是否正確。

1. 啟動命令提示字元，並移至 `%Program Files%\Microsoft Azure AD Sync\bin`。
2. 執行 `csexport "Name of Connector" %temp%\export.xml /f:x`。  
   連接器名稱在同步處理服務中。 它的名稱類似 Azure AD 的 "contoso.com – AAD"。
3. 執行 `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv`。
4. 現在您在 %temp% 中已經有名稱為 export.csv 的檔案，可在 Microsoft Excel 中加以檢查。 此檔案包含即將匯出的所有變更。
5. 對資料或組態進行必要的變更並再次執行這些步驟 (匯入、同步處理、驗證)，直到要匯出的變更皆如您所預期進行。

在感到滿意後，將變更匯出至 Azure AD。

1. 選取 [連接器]。 在 [連接器] 清單中，選取 [Azure AD 連接器]。 在 [動作] 中，選取 [執行]。
2. 在 [執行設定檔] 中，選取 [匯出]。
3. 如果您的組態變更將會刪除許多物件，且數目超過設定的臨界值 (預設值為 500)，則會在匯出時看到錯誤。 如果看到這個錯誤，您必須先暫時停用「[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)」功能。

現在該重新啟用排程器了。

1. 從 [開始] 功能表啟動 [工作排程器]。
2. 在 [工作排程器程式庫] 正下方尋找名稱為 [Azure AD 同步排程器] 的工作，在該工作上按一下滑鼠右鍵，然後選取 [啟用]。

## <a name="group-based-filtering"></a>群組型篩選
您可以在首次安裝 Azure AD Connect 時，使用[自訂安裝](how-to-connect-install-custom.md#sync-filtering-based-on-groups)來設定群組型篩選。 它適用於試驗部署，其中您只要同步小型物件集合。 當您停用群組型篩選時，無法將它重新啟用。 不支援在自訂設定中使用群組型篩選。 只支援使用安裝精靈來設定此功能。 完成試驗時，使用此主題中的其他篩選選項之一。 當將 OU 型篩選搭配群組型篩選使用時，必須包含群組及其成員所在的 OU。

同步處理多個 AD 樹系時，您可藉由為每個 AD 連接器指定不同的群組，設定以群組為基礎的篩選。 如果您想要同步處理某個 AD 樹系中的使用者，而相同的使用者在其他 AD 樹系中有一或多個對應的物件，您必須確定該使用者物件及其對應的所有物件，都位於以群組為基礎的篩選範圍內。 例如：

* 您有位於某個樹系，並在另一個樹系中具有相對應 FSP (外部安全性主體) 物件的使用者。 這兩個物件都必須位於以群組為基礎的篩選範圍內。 否則，系統將不會將該使用者同步處理至 Azure AD。

* 您有位於某個樹系，並在另一個樹系中具有相對應資源帳戶 (例如連結的信箱) 的使用者。 此外，您已將 Azure AD Connect 設定為與具有該資源帳戶的使用者連結。 這兩個物件都必須位於以群組為基礎的篩選範圍內。 否則，系統將不會將該使用者同步處理至 Azure AD。

* 您有位於某個樹系，並在另一個樹系中具有相對應郵件連絡人的使用者。 此外，您已將 Azure AD Connect 設定為與具有該郵件連絡人的使用者連結。 這兩個物件都必須位於以群組為基礎的篩選範圍內。 否則，系統將不會將該使用者同步處理至 Azure AD。


## <a name="next-steps"></a>後續步驟
- 深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。
- 深入了解[整合內部部署身分識別與 Azure AD](whatis-hybrid-identity.md)。
