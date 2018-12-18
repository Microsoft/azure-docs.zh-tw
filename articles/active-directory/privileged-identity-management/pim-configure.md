---
title: 什麼是 Azure AD Privileged Identity Management？ | Microsoft Docs
description: 提供 Azure Active Directory Privileged Identity Management (PIM) 的概觀。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: pim
ms.topic: overview
ms.date: 03/07/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: af93ade2a7031aeda5b4108649c59a8d6c1393ce
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465855"
---
# <a name="what-is-azure-ad-privileged-identity-management"></a>什麼是 Azure AD Privileged Identity Management？

使用 Azure Active Directory (Azure AD) Privileged Identity Management 時，您可以管理、控制及監視您組織內的存取。 這包括存取 Azure AD、Azure 資源與其他 Microsoft 線上服務 (如 Office 365 或 Microsoft Intune) 中的資源。

> [!NOTE]
> 當您為租用戶啟用 Privileged Identity Management 時，每個與此服務互動或從此服務受益的使用者都必須具備有效 Azure AD Premium P2 或 Enterprise Mobility + Security E5 的付費或試用授權。 範例包括群組中符合下列條件的使用者：
>
>- 指派給「特殊權限角色管理員」角色 
>- 指派成符合可透過 PIM 管理的其他目錄角色資格 
>- 能夠在 PIM 中核准/拒絕要求 
>- 指派給具有「及時」或「直接」(時間型) 指派的 Azure 資源角色  
>- 指派給存取檢閱
>
>如需詳細資訊，請參閱 [Azure Active Directory 版本](../fundamentals/active-directory-whatis.md)。

組織想要將能夠存取安全資訊或資源的人數降到最低，因為這樣可以降低惡意使用者取得該存取權，或者授權使用者無意中影響到敏感資源的機率。  不過，使用者仍然需要在 Azure AD、Azure、Office 365 或 SaaS 應用程式中執行特殊權限作業。 組織可以提供使用者特殊權限以存取 Azure 資源，如訂用帳戶和 Azure AD。 需要針對這些使用者使用其系統管理權限的方式進行監督。 Azure AD Privileged Identity Management 可協助減少過多、不必要或誤用的存取權限風險。

Azure AD Privileged Identity Management 可協助您的組織：

- 查看哪些使用者已被指派特殊權限角色來管理 Azure 資源，以及哪些使用者在 Azure AD 中已被指派系統管理角色
- 啟用隨選、「即時」的系統管理權限以存取 Microsoft 線上服務，如 Office 365 和 Intune，以及訂用帳戶、資源群組和個別資源 (如虛擬機器) 等 Azure 資源 
- 查看系統管理員啟用的記錄，包括系統管理員對 Azure 資源所做變更
- 取得系統管理員指派變更的警示
- 需要核准才能啟用 Azure AD 特殊權限系統管理角色
- 檢閱系統管理角色的成員資格，並要求使用者提供持續成員資格的證明

在 Azure AD 中，Azure AD Privileged Identity Management 可以管理指派為內建 Azure AD 組織角色 (如全域管理員) 的使用者。 在 Azure 中，Azure AD Privileged Identity Management 可以管理透過 Azure RBAC 角色 (包括擁有者或參與者) 指派的使用者和群組。

## <a name="just-in-time-administrator-access"></a>即時管理員存取權

在過去，您可以透過 Azure 入口網站、其他 Microsoft 線上服務入口網站或在 Windows PowerShell 中的 Azure AD Cmdlet 將使用者指派為系統管理員角色。 因此，該使用者會成為 **永久管理員**，其獲得指派的角色永遠處於作用狀態。 Azure AD Privileged Identity Management 導入了「合格系統管理員」 的概念。合格系統管理員應是指偶爾需要特殊存取權限，而非全天及每天都需要此權限的使用者。 在使用者需要存取權之前，角色會處於非作用中狀態，然後使用者須完成啟用程序，才能在一段預定的時間內成為作用中的系統管理員。 越來越多組織選擇使用此方法來減少或消除特殊權限角色的「永久性系統管理存取權限」。


## <a name="terminology"></a>術語

*合格角色使用者* – 合格角色使用者是您組織內已在符合資格時指派給 Azure AD 角色的使用者 (角色需要啟用)。

*委派核准者* – 委派核准者是您 Azure AD 內的一或多個個人或群組，負責核准啟用角色的要求。

## <a name="scenarios"></a>案例

Privileged Identity Management 支援下列案例：

**身為特殊權限角色管理員，您可以：**

- 啟用特定角色的核准
- 指定核准者使用者和/或群組來核准要求
- 檢視所有特殊權限角色的要求和核准歷程記錄

**身為指定的核准者，您可以：**

- 檢視待決的核准 (要求)
- 核准或拒絕提高角色權限 (單一和/或大量) 的要求
- 提供我的核准/拒絕理由 

**身為合格角色使用者，您可以：**

- 要求啟用需要核准的角色
- 檢視要啟用之要求的狀態
- 如果已核准啟用，在 Azure AD 中完成您的工作

## <a name="enable-privileged-identity-management-for-your-directory"></a>啟用目錄的 Privileged Identity Management

您可以在 [Azure 入口網站](https://portal.azure.com/)中開始使用 Azure AD Privileged Identity Management。

> [!NOTE]
> 您必須是具有組織帳戶 (例如 @yourdomain.com) 而非 Microsoft 帳戶 (例如 @outlook.com) 的全域管理員，才能啟用目錄的 Azure AD Privileged Identity Management。

1. 以目錄的全域系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/) 。
2. 如果貴組織有多個目錄，請選取 Azure 入口網站右上角的使用者名稱。 選取您將在其中使用 Azure AD Privileged Identity Management 的目錄。
3. 選取 [所有服務] 並使用 [篩選器] 文字方塊來搜尋 [Azure AD Privileged Identity Management]。
4. 選取 [釘選到儀表板]，然後按一下 [建立]。 Privileged Identity Management 應用程式隨即開啟。

如果您是在目錄中第一個使用 Azure AD Privileged Identity Management 的人，並且瀏覽至 Azure AD 目錄角色，則[安全性精靈](pim-security-wizard.md)會引導您完成初始指派體驗。 之後，您就會自動成為目錄的第一個「安全性系統管理員」和「特殊權限角色管理員」。

對於 Azure AD 角色，只有具特殊權限角色系統管理員角色的使用者可以管理 Azure AD PIM 中其他系統管理員的指派。 您可以[在 PIM 中為其他使用者提供管理目錄角色的能力](pim-how-to-give-access-to-pim.md)。 全域管理員、安全性系統管理員和安全性讀取者可以在 Azure AD PIM 中檢視 Azure AD 角色指派。
對於 Azure RBAC 角色，只有訂用帳戶管理員、資源擁有者、或資源使用者存取系統管理員可以管理 Azure AD PIM 中其他系統管理員的指派。  作為特殊權限角色系統管理員、安全性系統管理員或安全性讀取者的使用者，依預設沒有存取權限可檢視 Azure AD PIM 中的 Azure RBAC 角色指派。

## <a name="privileged-identity-management-overview-entry-point"></a>Privileged Identity Management 概觀 (進入點)

Azure AD Privileged Identity Management 可支援 Azure AD 目錄角色與 Azure 資源角色的系統管理。 Azure 資源角色與 Azure AD 中系統管理角色的功能不同。 Azure 資源角色提供針對指派的資源以及資源階層中所有從屬資源 (稱為繼承) 的細微權限。 [深入了解 RBAC、資源階層和繼承](../../role-based-access-control/role-assignments-portal.md)。 在 PIM [概觀] 進入點左側導覽功能表的 [管理] 區段下存取相應連結，即可管理適用於 Azure AD 目錄角色與 Azure 資源的 PIM。

PIM 提供方便的存取，以利啟用角色、檢視擱置的啟用/要求、擱置的核准 (適用於 Azure AD 目錄角色)，以及左側導覽功能表 [工作] 區段的待回應檢閱。

從 [概觀] 進入點存取任何 [工作] 功能表項目時，產生的檢視會包含 Azure AD 目錄角色與 Azure 資源角色的結果。

![快速入門](./media/pim-configure/quick-start.png)

[我的角色] 包含一份作用中與合格的角色指派清單，適用於 Azure AD 目錄角色和 Azure 資源角色。 [深入了解啟用合格的角色指派](pim-how-to-activate-role.md)。

啟用 Azure 資源角色帶來了新的體驗，可讓合格的角色成員排程在未來日期/時間進行啟用，並可在系統管理員允許的最大範圍內選取特定的啟用持續時間。

![](./media/pim-configure/activations.png)

在事件中若不再需要排程的啟用，則使用者可以取消擱置的要求，方法是瀏覽至左側導覽功能表中擱置的要求，然後按一下該要求的 [取消] 按鈕。

![擱置的要求](./media/pim-configure/pending-requests.png)

## <a name="privileged-identity-management-admin-dashboard"></a>Privileged Identity Management 管理員儀表板

Azure AD Privileged Identity Manager 有一個管理員儀表板可提供重要資訊，例如：

* 指出有提升安全性機會的警示
* 指派給每個特殊權限角色的使用者數目  
* 合格和永久系統管理員的數目
* 您的目錄中特殊權限角色啟用的圖表
* Azure 資源角色 的 Just-In-Time、時間繫結和永久指派數目
* 過去 30 天內具有新角色指派的使用者與群組 (Azure 資源角色)


![PIM 儀表板 - 螢幕擷取畫面](./media/pim-configure/PIM_Admin_Overview.png)

## <a name="privileged-role-management"></a>特殊權限角色管理

使用 Azure AD Privileged Identity Management，您便可透過新增或移除 Azure AD 目錄角色中每個角色的永久或合格系統管理員來管理這些系統管理員。 有了適用於 Azure 資源的 PIM，擁有者、使用者存取系統管理員，和啟用其租用戶中訂用帳戶管理的全域系統管理員便可以將使用者或群組指派為合格的 (Just-In-Time 存取)，或具開始和結束日期/時間的時間繫結 (不需啟用) 存取，或永久的 (如果在角色設定中已啟用) Azure 資源角色。

![PIM 新增/移除系統管理員 - 螢幕擷取畫面](./media/pim-configure/PIM_AddRemove.png)

## <a name="configure-the-role-activation-settings"></a>設定角色啟用設定

您可以使用[角色設定](pim-how-to-change-default-settings.md)來設定 Azure AD 目錄角色的合格角色啟用屬性，包括：

* 角色啟用期間的持續時間
* 角色啟用通知
* 使用者在角色啟用程序期間所需提供的資訊
* 服務票證或事件數目
* [核准工作流程需求](./azure-ad-pim-approval-workflow.md)

![PIM 設定 - 系統管理員啟動 - 螢幕擷取畫面](./media/pim-configure/PIM_Settings_w_Approval_Disabled.png)

請注意，此影像中已停用 [Multi-Factor Authentication] 的按鈕。 針對某些高特殊權限的角色，我們會要求使用 MFA 來增強防護。

Azure 資源角色的角色設定允許系統管理員設定 Just-In-Time 及直接指派設定，包括：

- 能夠將使用者或群組指派為沒有結束日期/時間 (永久指派) 的角色
- 預設的指派持續時間 (非永久時)
- 啟用持續時間上限 (合格的角色成員啟用時)
- 使用者需要在角色啟用 (Just-In-Time 指派) 或指派程序 (直接指派) 期間提供的資訊

![](./media/pim-configure/role-settings-details.png)

## <a name="role-activation"></a>角色啟用

為了 [啟用角色](pim-how-to-activate-role.md)，合格系統管理員會為角色要求一個有時效性的「啟用」。 使用 Azure AD 特殊權限身分識別管理中的 [啟用我的角色] 選項，即可要求啟用。

想要啟用角色的管理員必須在 Azure 入口網站中初始化 Azure AD Privileged Identity Management。

角色啟用是可自訂的。 在 [PIM] 設定中，您可以決定啟用的長度，以及管理員必須提供才能啟用角色的資訊。

![PIM 系統管理員要求角色啟用 - 螢幕擷取畫面](./media/pim-configure/PIM_RequestActivation.png)

## <a name="review-role-activity"></a>檢閱角色活動

有兩種方式可以追蹤您的員工和系統管理員使用特殊權限角色的情況。 第一個選項是使用[目錄角色稽核歷程](pim-how-to-use-audit-log.md)。 稽核歷程記錄會追蹤特殊權限角色指派、角色啟用記錄的變更，以及 Azure 資源角色設定的變更。 

![PIM 啟動歷程記錄 - 螢幕擷取畫面](./media/pim-configure/PIM_ActivationHistory.png)

第二個選項是設定標準[存取檢閱](pim-how-to-start-security-review.md)。 這些存取檢閱可以由指派的檢閱者 (例如團隊經理) 來執行或者員工可以檢閱自己。 如此來監視誰仍需要或不再需要存取是最佳的方式。

## <a name="azure-ad-pim-at-subscription-expiration"></a>訂用帳戶過期時的 Azure AD PIM

租用戶必須有 Azure AD Premium P2 (或 EMS E5) 試用版或在租用戶中有付費訂用帳戶，才能使用 Azure AD PIM。  此外，必須指派授權給租用戶的系統管理員。  具體來說，必須將授權指派給透過 Azure AD PIM 管理的 Azure AD 角色中的系統管理員、透過 Azure AD PIM 管理的 Azure RBAC 角色中的系統管理員，以及執行存取權檢閱的任何非系統管理員使用者。
如果您的組織未更新 Azure AD Premium P2 或試用版到期，則租用戶中的 Azure AD PIM 功能將無法再使用、合格的角色指派將會移除，且使用者將不再能夠啟用角色。 您可以深入了解 [Azure AD PIM 訂用帳戶要求](./subscription-requirements.md)

## <a name="next-steps"></a>後續步驟

- [使用 PIM 的訂用帳戶需求](subscription-requirements.md)
- [您可以在 PIM 中管理的 Azure AD 目錄角色](pim-roles.md)
- [在 Azure AD 中保護混合式部署和雲端部署的特殊權限存取](../users-groups-roles/directory-admin-roles-secure.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json)
