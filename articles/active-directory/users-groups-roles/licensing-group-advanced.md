---
title: Azure Active Directory 群組型授權其他案例 | Microsoft Docs
description: Azure Active Directory 群組型授權的更多案例
services: active-directory
keywords: Azure AD 授權
documentationcenter: ''
author: curtand
manager: mtillman
editor: piotrci
ms.service: active-directory
ms.topic: article
ms.workload: identity
ms.component: users-groups-roles
ms.date: 06/02/2017
ms.author: curtand
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 15b52920774a878cd386ced5966d507768a8af70
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627384"
---
# <a name="scenarios-limitations-and-known-issues-using-groups-to-manage-licensing-in-azure-active-directory"></a>使用群組來管理 Azure Active Directory 授權的案例、限制及已知問題

使用下列資訊和範例，以更深入了解 Azure Active Directory (Azure AD) 群組型授權。

## <a name="usage-location"></a>使用位置

並非所有位置都可使用某些 Microsoft 服務。 系統管理員必須指定使用者的 [使用位置]屬性，才能將授權指派給使用者。 您可以在 [Azure 入口網站](https://portal.azure.com)的 [使用者] &gt; [設定檔] &gt; [設定] 中指定。

如果是群組授權指派，未指定使用位置的任何使用者會繼承目錄的位置。 如果您在多個位置有使用者，請確定先在使用者物件中正確反映，然後再將使用者新增至具有授權的群組。

> [!NOTE]
> 群組授權指派永遠不會修改使用者現有的使用位置值。 建議您一律將使用位置設為 Azure AD 中使用者建立流程的一部分 (例如透過 AAD Connect 設定) - 確保授權指派的結果一律是正確的，且使用者不會在不允許的位置收到服務。

## <a name="use-group-based-licensing-with-dynamic-groups"></a>對動態群組使用群組型授權

您可以對任何安全性群組使用群組型授權，這表示可與 Azure AD 動態群組結合。 動態群組會針對使用者物件屬性執行規則，以自動新增和移除群組的使用者。

例如，您可以針對想要指派給使用者的某組產品建立動態群組。 每個群組會填入一項規則，依使用者的屬性新增使用者，而且每個群組會指派您希望他們收到的授權。 您可以在內部部署指派屬性，再將它與 Azure AD 同步，或者，您也可以直接在雲端管理屬性。

使用者新增至群組之後，會很快獲派授權。 變更屬性時，使用者會離開群組，且會移除授權。

### <a name="example"></a>範例

假設有一個內部部署身分識別管理解決方案，可決定哪些使用者可存取 Microsoft Web 服務。 它使用 **extensionAttribute1** 來儲存字串值，代表使用者應該擁有的授權。 Azure AD Connect 會將它與 Azure AD 同步。

使用者可能需要其中一個授權，也可能需要兩個授權。 在下列範例中，您會將 Office 365 企業版 E5 和 Enterprise Mobility + Security (EMS) 授權散發給群組中的使用者：

#### <a name="office-365-enterprise-e5-base-services"></a>Office 365 企業版 E5︰基本服務

![Office 365 企業版 E5 基本服務的螢幕擷取畫面](./media/licensing-group-advanced/o365-e5-base-services.png)

#### <a name="enterprise-mobility--security-licensed-users"></a>Enterprise Mobility + Security︰授權的使用者

![Enterprise Mobility + Security 授權的使用者螢幕擷取畫面](./media/licensing-group-advanced/o365-e5-licensed-users.png)

在此範例中，如果您想要讓使用者有兩個授權，請修改一個使用者，並將其 extensionAttribute1 的值設為 `EMS;E5_baseservices;`。 您可以在內部部署做此修改。 在變更與雲端同步之後，使用者會自動新增至這兩個群組，並且指派授權。

![顯示如何設定使用者的 extensionAttribute1 的螢幕擷取畫面](./media/licensing-group-advanced/user-set-extensionAttribute1.png)

> [!WARNING]
> 修改現有群組的成員資格規則時請小心。 變更規則時，將會重新評估群組的成員資格，並移除不再符合新規則的使用者 (在此程序期間不會影響仍符合新規則的使用者)。 這些使用者的授權會在程序期間移除，而導致服務遺失，或在某些情況下導致資料遺失。

> 如果需要授權指派的動態群組很大，請考慮在較小的測試群組上驗證任何重大變更，再將這些變更套用至主要群組。

## <a name="multiple-groups-and-multiple-licenses"></a>多個群組和多個授權

使用者可以是多個具有授權之群組的成員。 以下是要考量的一些事項：

- 相同產品的多個授權可能重疊，導致所有已啟用的服務套用至使用者。 下列範例顯示兩個授權群組︰「E3 基本服務」包含最先部署給所有使用者的基礎服務。 「E3 擴充服務」包含只部署給某些使用者的其他服務 (Sway 和 Planner)。 在此範例中，已將使用者新增至這兩個群組︰

  ![已啟用服務的螢幕擷取畫面](./media/licensing-group-advanced/view-enabled-services.png)

  因此，使用者在產品的 12 項服務中已啟用 7 項，但只對此產品使用一個授權。

- 選取 E3 授權會顯示更多詳細資訊，包括哪些群組造成對使用者啟用什麼服務的相關資訊。

  ![依群組啟用服務的螢幕擷取畫面](./media/licensing-group-advanced/view-enabled-service-by-group.png)

## <a name="direct-licenses-coexist-with-group-licenses"></a>直接授權與群組授權共存

當使用者從群組繼承授權時，您無法直接在使用者屬性中移除或修改該授權指派。 您必須在群組中進行變更，再傳播給所有使用者。

不過，除了繼承的授權，還可以將相同產品授權直接指派給使用者。 您可以僅針對一個使用者啟用產品中的其他服務，而不會影響其他使用者。

可以移除直接指派的授權，而不會影響繼承的授權。 假設使用者從群組繼承 Office 365 企業版 E3 授權。

1. 一開始，使用者只從「E3 基本服務」群組繼承授權，以啟用四個服務方案，如下所示：

  ![E3 群組啟用的服務的螢幕擷取畫面](./media/licensing-group-advanced/e3-group-enabled-services.png)

2. 您可以選取 [指派]，直接將 E3 授權指派給使用者。 在此案例中，您準備停用 Yammer Enterprise 以外的所有服務方案：

  ![如何直接將授權指派給使用者的螢幕擷取畫面](./media/licensing-group-advanced/assign-license-to-user.png)

3. 因此，使用者仍然只使用 E3 產品的一個授權。 但是，直接指派只會針對該使用者啟用 Yammer Enterprise 服務。 您可以看到群組成員資格所啟用的服務與直接指派：

  ![繼承和直接指派的螢幕擷取畫面](./media/licensing-group-advanced/direct-vs-inherited-assignment.png)

4. 使用直接指派時允許下列作業︰

  - 可以直接在使用者物件上關閉 Yammer Enterprise。 在圖中，這項服務已啟用 [開啟/關閉] 切換，不同於其他服務切換。 因為此服務直接在使用者上啟用，所以可以修改。
  - 其他服務也可啟用，做為直接指派授權的一部分。
  - [移除] 按鈕可用來移除使用者的直接授權。 您可以看到使用者現在只有繼承的群組授權，且只有原始服務保持啟用：

    ![顯示如何移除直接指派的螢幕擷取畫面](./media/licensing-group-advanced/remove-direct-license.png)

## <a name="managing-new-services-added-to-products"></a>管理新增至產品的新服務
當 Microsoft 新增服務至某項產品時，預設會在您指派產品授權的所有群組中啟用這項服務。 您租用戶中訂閱產品變更相關通知的使用者會事先收到電子郵件，通知他們即將新增服務。

身為系統管理員，您可以檢閱受變更影響的所有群組，並採取動作，例如停用每個群組中的新服務。 例如，如果您建立的群組只要以用於部署的特定服務為目標，您可以返回這些群組，並確定已停用所有新增的服務。

此程序可能如下所示：

1. 您一開始將 *Office 365 企業版 E5* 產品指派給數個群組。 其中一個群組稱為「O365 E5 - 僅 Exchange」，設計成只會為其成員啟用 [Exchange Online (方案 2)] 服務。

2. 您收到來自 Microsoft 的通知，指出 E5 產品將透過新服務 *Microsoft Stream* 擴充。 當此服務在您的租用戶中可供使用時，您可以執行下列動作：

3. 移至 [[Azure Active Directory] > [授權] > [所有產品]](https://portal.azure.com/#blade/Microsoft_AAD_IAM/LicensesMenuBlade/Products) 刀鋒視窗並選取 [Office 365 企業版 E5]，然後選取 [經過授權的群組] 以檢視該產品的所有群組清單。

4. 按一下您要檢閱的群組 (在本例中為「O365 E5 - 僅 Exchange」)。 這會開啟 [授權] 索引標籤。按一下 E5 授權會開啟刀鋒視窗，其中列出所有已啟用的服務。
> [!NOTE]
> 在此群組中，除了 *Exchange Online* 服務，還會自動新增並啟用 *Microsoft Stream* 服務：

  ![新增至群組授權的新服務螢幕擷取畫面](./media/licensing-group-advanced/manage-new-services.png)

5. 如果您想要停用此群組中的新服務，請按一下服務旁的 [開啟/關閉] 切換，然後按一下 [儲存] 按鈕以確認變更。 Azure AD 現在會處理群組中的所有使用者以套用變更；新增至群組的任何新使用者都不會啟用*Microsoft Stream*服務。

  > [!NOTE]
  > 使用者仍然可以透過其他授權指派 (使用者所屬的另一個群組或直接授權指派) 啟用服務。

6. 如有需要，請對指派此產品的其他群組執行相同步驟。

## <a name="use-powershell-to-see-who-has-inherited-and-direct-licenses"></a>使用 PowerShell 查看誰具有繼承和直接授權
您可以使用 PowerShell 指令碼，來檢查使用者是否有直接指派或繼承自群組的授權。

1. 執行 `connect-msolservice` Cmdlet 來驗證並連線至您的租用戶。

2. `Get-MsolAccountSku` 可用於探索租用戶中佈建的所有產品授權。

  ![Get-Msolaccountsku Cmdlet 的螢幕擷取畫面](./media/licensing-group-advanced/get-msolaccountsku-cmdlet.png)

3. 透過[此 PowerShell 指令碼](licensing-ps-examples.md#check-if-user-license-is-assigned-directly-or-inherited-from-a-group)對您感興趣的授權使用 *AccountSkuId* 值。 這會產生具有此授權的使用者清單，並提供授權指派方式的相關資訊。

## <a name="use-audit-logs-to-monitor-group-based-licensing-activity"></a>使用稽核記錄來監視群組型授權活動

您可以使用 [Azure AD 稽核記錄](../reports-monitoring/concept-audit-logs.md#audit-logs)來查看與群組型授權相關的所有活動，包括：
- 哪些人變更了群組的授權
- 系統何時開始處理群組授權變更，以及何時完成
- 由於群組授權指派而對使用者做了哪些授權變更。

>[!NOTE]
> 入口網站之 [Azure Active Directory] 區段中的大部分刀鋒視窗都有提供稽核記錄。 根據您存取記錄的位置，可能會預先套用篩選，僅顯示與刀鋒視窗內容相關的活動。 如果您沒有看到預期的結果，請檢查[篩選選項](../reports-monitoring/concept-audit-logs.md#filtering-audit-logs)，或在 [[Azure Active Directory] > [活動] > [稽核記錄]](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Audit) 下存取未篩選的稽核記錄。

### <a name="find-out-who-modified-a-group-license"></a>了解哪些人修改了群組授權

1. 將 [活動] 篩選設定為 [設定群組授權]，然後按一下 [套用]。
2. 結果會包含在群組上設定或修改的所有授權案例。
>[!TIP]
> 您也可以在 [目標] 篩選中鍵入群組名稱來限定結果範圍。

3. 按一下清單檢視中的項目來查看已變更內容的詳細資料。 在 [已修改的屬性] 下會同時列出授權指派的新舊值。

下列範例顯示最近的群組授權變更，並提供詳細資料：

![群組授權變更的螢幕擷取畫面](./media/licensing-group-advanced/audit-group-license-change.png)

### <a name="find-out-when-group-changes-started-and-finished-processing"></a>了解群組變更何時開始和完成處理

當群組變更授權時，Azure AD 會開始將變更套用至所有使用者。

1. 若要查看群組何時開始處理，請將 [活動] 篩選設定為 [開始將群組型授權套用至使用者]。 請注意，此作業的動作項目是「Microsoft Azure AD 群組型授權」，這是用來執行所有群組授權變更的系統帳戶。
>[!TIP]
> 按一下清單中的項目可查看 [已修改的屬性] 欄位，該欄位會顯示為處理所選取的授權變更。 如果您對群組進行多項變更，而且不確定哪項變更已經過處理，這會很有用。

2. 同樣地，若要查看群組何時完成處理，請使用篩選值 [完成將群組型授權套用至使用者]。
>[!TIP]
> 在此情況下，[已修改的屬性] 欄位會包含結果的摘要，這項資訊有助於快速檢查處理是否導致任何錯誤。 範例輸出：
> ```
Modified Properties
...
Name : Result
Old Value : []
New Value : [Users successfully assigned licenses: 6, Users for whom license assignment failed: 0.];
> ```

3. 若要查看群組處理方式的完整記錄 (包括所有使用者變更)，請設定下列篩選：
  - **啟動者 (執行者)**：「Microsoft Azure AD 群組型授權」
  - **日期範圍** (選用)：特定群組開始和完成處理時間的自訂範圍

此範例輸出會顯示開始處理的時間、所有產生的使用者變更，以及完成處理的時間。

![群組授權變更的螢幕擷取畫面](./media/licensing-group-advanced/audit-group-processing-log.png)

>[!TIP]
> 按一下與「變更使用者授權」相關的項目，會顯示套用至每個使用者之授權變更的詳細資料。

## <a name="deleting-a-group-with-an-assigned-license"></a>刪除含指派授權的群組

您無法刪除被指派使用中授權的群組。 系統管理員可能會刪除群組，但他不知道這會從使用者移除授權；基於這個理由，我們需要先從群組移除任何授權，之後才能刪除它。

嘗試在 Azure 入口網站中刪除群組時，您可能會看到如下的錯誤通知：![螢幕擷取畫面群組刪除失敗](./media/licensing-group-advanced/groupdeletionfailed.png)

移至群組 [授權] 索引標籤，並查看是否有任何指派的授權。 如果是，移除這些授權，並試著再次刪除群組。

嘗試透過 PowerShell 或圖形 API 刪除群組時，您可能會看到類似的錯誤。 如果您使用從內部部署同步的群組，當 Azure AD Connect 無法在 Azure AD 中刪除群組時，可能也會報告錯誤。 在所有情況下，請務必檢查是否有指派給群組的任何授權，並先移除它們。

## <a name="limitations-and-known-issues"></a>限制與已知問題

如果您使用群組型授權，最好先熟悉下列的限制和已知問題清單。

- 群組型授權目前不支援包含其他群組的群組 (巢狀群組)。 如果您將授權套用至巢狀群組，則只有群組的直接第一層級使用者成員會套用授權。

- 此功能只能用於安全性群組。 目前不支援 Office 群組，因此您將無法在授權指派程序中使用。

- [Office 365 系統管理入口網站](https://portal.office.com )目前不支援群組型授權。 如果使用者從群組繼承授權，此授權在 Office 系統管理入口網站中會顯示為一般使用者授權。 如果您嘗試修改該授權或嘗試移除授權，入口網站會傳回錯誤訊息。 無法直接修改使用者繼承的群組授權。

- 當使用者從群組移除且失去授權時，來自該授權的服務方案 (例如 SharePoint Online) 會設定為「暫停」狀態。 服務方案不會設定為最終的已停用狀態。 此預防措施可避免系統管理員在群組成員資格管理時犯錯，而意外移除使用者資料。

- 當指派或修改大型群組 (例如 100,000 個使用者) 的授權時，可能會影響效能。 具體來說，Azure AD 自動化所產生的大量變更，可能會降低 Azure AD 和內部部署系統之間目錄同步作業的效能。

- 在某些高負載情況下，授權處理可能會延遲，導致諸如新增/移除群組授權或新增/移除群組使用者等變更，可能需要很長的時間來處理。 如果您發現變更需要 24 小時以上才能處理完畢，請[開啟支援票證](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/supportRequest)讓我們調查。 我們會在這項功能「正式運作」之前改善它的效能特性。

- 授權管理自動化並不會自動對環境中所有類型的變更做出回應。 例如，您可能已用盡授權，導致某些使用者處於錯誤狀態。 若要釋出可用的授權數量，您可以移除其他使用者的一些直接指派授權。 不過，系統不會自動回應這項變更，也不會修正處於該錯誤狀態的使用者。

  若要解決這幾種限制，您可以前往 Azure AD 中的 [群組] 刀鋒視窗，然後按一下 [重新處理]。 此命令會處理該群組中的所有使用者，並解決錯誤狀態 (如果可能)。

- 如果由於 Exchange Online 中的重複 Proxy 位址設定而無法對使用者指派授權，群組型授權不會記錄錯誤，而且在授權指派期間會略過這類使用者。 如需如何識別及解決此問題的詳細資訊，請參閱[本節](licensing-groups-resolve-problems.md#license-assignment-fails-silently-for-a-user-due-to-duplicate-proxy-addresses-in-exchange-online)。

## <a name="next-steps"></a>後續步驟

若要深入了解透過群組型授權來管理授權的其他案例，請參閱：

* [什麼是 Azure Active Directory 中以群組為基礎的授權？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [將授權指派給 Azure Active Directory 中的群組](licensing-groups-assign.md)
* [識別及解決 Azure Active Directory 中群組的授權問題](licensing-groups-resolve-problems.md)
* [如何將個別授權使用者移轉至 Azure Active Directory 中以群組為基礎的授權](licensing-groups-migrate-users.md)
