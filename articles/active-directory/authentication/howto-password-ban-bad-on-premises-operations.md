---
title: Azure AD 密碼保護預覽作業和報告
description: Azure AD 密碼保護預覽部署後作業和報告
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: jsimmons
ms.openlocfilehash: 14aa52b6d424423f4863efa63f3e2e66b84dac70
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39163495"
---
# <a name="preview-azure-ad-password-protection-post-deployment"></a>預覽：Azure AD 密碼保護後續部署

|     |
| --- |
| Azure AD 密碼保護和自訂的禁用密碼清單是 Azure Active Directory 的公開預覽版功能。 如需有關預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

完成在內部部署環境[安裝 Azure AD 密碼保護](howto-password-ban-bad-on-premises.md)後，您必須在 Azure 入口網站中設定一些項目。

## <a name="configure-the-custom-banned-password-list"></a>設定自訂禁用密碼清單

請遵循[設定自訂禁用密碼清單](howto-password-ban-bad.md)一文中的指導方針，以了解為組織自訂禁用密碼清單的步驟。

## <a name="enable-password-protection"></a>啟用密碼保護

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory]、[驗證方法] 和 [密碼保護 (預覽)]。
1. 將 [啟用 Windows Server Active Directory 上的密碼保護] 設定為 [是]。
1. 如[部署指南](howto-password-ban-bad-on-premises.md#deployment-strategy)所述，建議您一開始就將 [模式] 設定為 [稽核]
   * 對功能感到滿意後，您可以將 [模式] 切換為 [已強制]
1. 按一下 [儲存] 

![在 Azure 入口網站中啟用 Azure AD 密碼保護元件](./media/howto-password-ban-bad-on-premises-operations/authentication-methods-password-protection-on-prem.png)

## <a name="audit-mode"></a>稽核模式

稽核模式的目的是為了在「What If」模式中執行軟體。 每個 DC 代理程式服務都會根據目前作用中的原則來評估傳入的密碼。 如果目前的原則設定為要在稽核模式中執行，「不良」密碼就會產生事件記錄訊息，但系統會接受該密碼。 這是稽核和強制執行模式的唯一差別；其他所有作業的執行方式則相同。

> [!NOTE]
> Microsoft 建議您一律讓初始部署和測試在稽核模式中開始進行。 然後，請監視事件記錄中的事件，以嘗試預期在啟用強制執行模式後，是否會干擾任何現有的作業程序。

## <a name="enforce-mode"></a>強制執行模式

強制執行模式的目的是為了作為最終設定。 和上述在稽核模式時一樣，每個 DC 代理程式服務都會根據目前作用中的原則來評估傳入的密碼。 但如果啟用強制執行模式，則系統會拒絕根據原則而視為不安全的密碼。

當 Azure AD 密碼保護 DC 代理程式在強制執行模式中拒絕密碼時，終端使用者所見到的影響，會和傳統內部部署密碼複雜度強制執行功能拒絕其密碼時所見到的影響相同。 例如，使用者可能會在 logon\change password 畫面看到下列傳統錯誤訊息：

「無法更新密碼。 為新密碼提供的值不符合長度、複雜度或網域的歷史需求。」

這則訊息只是數個可能結果的其中一個範例。 特定錯誤訊息會隨著嘗試設定不安全密碼的實際軟體或案例而有所不同。

受影響的終端使用者可能需要與其 IT 人員合作，以了解新的需求，並更加能夠選擇安全的密碼。

## <a name="usage-reporting"></a>使用量回報

`Get-AzureADPasswordProtectionSummaryReport` Cmdlet 可用來產生活動的摘要檢視。 此 Cmdlet 的範例輸出如下：

```
Get-AzureADPasswordProtectionSummaryReport -DomainController bplrootdc2
DomainController                : bplrootdc2
PasswordChangesValidated        : 6677
PasswordSetsValidated           : 9
PasswordChangesRejected         : 10868
PasswordSetsRejected            : 34
PasswordChangeAuditOnlyFailures : 213
PasswordSetAuditOnlyFailures    : 3
PasswordChangeErrors            : 0
PasswordSetErrors               : 1
```

此 Cmdlet 的報告範圍可使用 –Forest、-Domain 或 –DomainController 參數其中之一來加以影響。 未指定參數意指 –Forest。

> [!NOTE]
> 從遠端查詢每個 DC 代理程式服務的系統管理事件記錄即可讓此 Cmdlet 運作。 如果事件記錄包含大量事件，此 Cmdlet 可能需要很長的時間才能完成。 此外，透過網路大量查詢大型資料集可能會影響網域控制站的效能。 因此，在生產環境中請小心使用這個 Cmdlet。

## <a name="next-steps"></a>後續步驟

[針對 Azure AD 密碼保護進行疑難排解並記錄資訊](howto-password-ban-bad-on-premises-troubleshoot.md)
