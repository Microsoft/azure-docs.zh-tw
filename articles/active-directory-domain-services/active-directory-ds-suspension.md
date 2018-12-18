---
title: Azure Active Directory Domain Services：暫時停權的網域 | Microsoft Docs
description: 受控網域的暫時停權與刪除
services: active-directory-ds
documentationcenter: ''
author: eringreenlee
manager: mtillman
editor: curtand
ms.assetid: 95e1d8da-60c7-4fc1-987d-f48fde56a8cb
ms.service: active-directory
ms.component: domains
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: ergreenl
ms.openlocfilehash: 72e22fbe61b4e30191bbac553709241e1b13f1f5
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502244"
---
# <a name="suspended-domains"></a>暫時停權的網域
當 Azure Active Directory Domain Services (Azure AD DS) 有很長的一段時間無法服務受控網域時，其會使該受控網域進入暫時停權狀態。 本文會說明受控網域被暫時停權的原因，以及如何修復被暫時停權的網域。


## <a name="states-your-managed-domain-can-be-in"></a>受控網域可能會處於的狀態

![暫時停權的網域時間軸](media\active-directory-domain-services-suspension\suspension-timeline.PNG)

上圖概述 Azure AD DS 受控網域可能會處於的狀態。

### <a name="running-state"></a>「執行中」狀態
已正確設定且定期運作的受控網域，會處於**執行中**狀態。

**有何影響**
* Microsoft 可以定期監視您受控網域的健康情況。
* 您受控網域的網域控制站會被定期修補並更新。
* 來自 Azure Active Directory 的變更會定期同步處理到您的受控網域。
* 會針對您的受控網域進行定期備份。


### <a name="needs-attention-state"></a>「需要注意」狀態
如果有一或多個問題需要系統管理員採取動作，受控網域即會處於**需要注意**狀態。 您受控網域的健康情況頁面會在此狀態下列出一或多個警示。 

例如，如果您已針對虛擬網路設定受限的 NSG，Microsoft 可能無法更新和監視您的受控網域。 這個無效的設定會觸發警示，使您的受控網域進入「需要注意」狀態。

每個警示都有一組解決步驟。 某些警示是暫時性，並會由服務自動解決。 針對其他警示，您可以藉由遵循該警示對應解決步驟中的指示來加以解決。 針對某些重大警示，您需要連絡 Microsoft 支援服務才能加以解決。

如需詳細資訊，請參閱[如何對受控網域上的警示進行疑難排解](active-directory-ds-troubleshoot-alerts.md)。

**有何影響**

在某些情況下 (例如，如果您的網路設定無效)，可能就無法連線到您受控網域的網域控制站。 當您的受控網域處於「需要注意」狀態時，Microsoft 無法保證它能被定期監視、修補、更新或備份。

* 您的受控網域會處於健康情況不良的狀態，而持續進行的健康情況監視可能會停止，直到警示解決為止。
* 您受控網域的網域控制站可能不會被修補或更新。
* 來自 Azure Active Directory 的變更可能無法同步處理到您的受控網域。
* 如果可行，可能會對您的受控網域進行備份。
* 如果您解決了影響受控網域的警示，可能會將它還原為「執行中」狀態。
* 針對 Microsoft 無法連線到您網域控制站的情況，系統會針對設定問題觸發重大警示。 如果這類警示未在 15 天內解決，您的受控網域就會進入「暫時停權」狀態。


### <a name="the-suspended-state"></a>「暫時停權」狀態
受控網域會基於下列因素而進入**暫時停權**狀態：

* 有一或多個重大警示未在 15 天內解決。 重大警示可能是因為設定錯誤而使 Azure AD DS 所需資源的存取遭到封鎖而導致的。
    * 例如，受控網域中的 [AADDS104：網路錯誤](active-directory-ds-troubleshoot-nsg.md)警示並未在 15 天內解決。
* 您的 Azure 訂用帳戶有計費問題，或者您的 Azure 訂用帳戶已過期。

當 Microsoft 無法持續管理、監視、修補或備份網域時，即會將受控網域暫時停權。

**有何影響**
* 您受控網域的網域控制站會被解除佈建，而且無法在虛擬網路中連線。
* 透過網際網路對受控網域進行的安全 LDAP 存取 (如果啟用) 會停止運作。
* 您注意到在對受控網域進行驗證、登入已加入網域的虛擬機器，或透過 LDAP/LDAPS 連線時會失敗。
* 不會再為您的受控網域進行備份。
* 與 Azure AD 的同步處理會停止。

解決警示之後，您的受控網域會進入「暫時停權」狀態。 接著，您需要連絡支援人員。
只有在少於 30 天的備份存在時，支援服務才有可能還原您的受控網域。

受控網域只會保持暫時停權狀態 15 天。 若要復原您的受控網域，Microsoft 建議您立即解決重大警示。


### <a name="deleted-state"></a>「已刪除」狀態
處於「暫時停權」狀態 15 天的受控網域會遭到**刪除**。

**有何影響**
* 該受控網域的所有資源和備份都會被刪除。
* 您無法還原該受控網域，並需要建立新的受控網域以使用 Azure AD DS。
* 將它刪除之後，您就不需繼續支付該受控網域的費用。


## <a name="how-do-you-know-if-your-managed-domain-is-suspended"></a>如何得知您的受控網域已被暫時停權？
您會在 Azure 入口網站中的 Azure AD DS [健康情況] 頁面上看到[警示](active-directory-ds-troubleshoot-alerts.md)，宣告該網域已被暫時停權。 網域的狀態也會顯示「暫時停權」。


## <a name="restore-a-suspended-domain"></a>還原暫時停權的網域
若要還原處於「暫時停權」狀態的網域，請採取下列步驟：

1. 前往 Azure 入口網站中的 [Azure Active Directory Domain Services 頁面](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.AAD%2FdomainServices)。
2. 選取受控網域。
3. 在左窗格中，選取 [健康情況]。
4. 選取警示。 依據暫時停權的原因而定，該警示識別碼將會是 AADDS503 或 AADDS504。
5. 選取警示中提供的解決連結。 接著，遵循下列步驟來解決警示。

您的受控網域只能還原到上次備份的日期。 上次備份的日期會顯示在您受控網域的 [健康情況] 頁面上。 在上次備份之後發生的任何變更都將無法還原。 受控網域的備份最多儲存 30 天。 超過 30 天的備份會被刪除。


## <a name="next-steps"></a>後續步驟
- [解決受控網域的警示](active-directory-ds-troubleshoot-alerts.md)
- [深入了解 Azure Active Directory Domain Services](active-directory-ds-overview.md)
- [連絡產品小組](active-directory-ds-contact-us.md)

## <a name="contact-us"></a>與我們連絡
請連絡 Azure Active Directory Domain Services 產品小組， [分享意見或尋求支援](active-directory-ds-contact-us.md)。
