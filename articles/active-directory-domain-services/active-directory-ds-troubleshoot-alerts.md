---
title: Azure Active Directory Domain Services：針對警示進行疑難排解 | Microsoft Docs
description: 針對 Azure AD Domain Services 的警示進行疑難排解
services: active-directory-ds
documentationcenter: ''
author: eringreenlee
manager: ''
editor: ''
ms.assetid: 54319292-6aa0-4a08-846b-e3c53ecca483
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/28/2018
ms.author: ergreenl
ms.openlocfilehash: 6be67a92c95ccf1161ffeeb636ee4f998c65fa05
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503684"
---
# <a name="azure-ad-domain-services---troubleshoot-alerts"></a>Azure AD Domain Services - 針對警示進行疑難排解
本文提供的疑難排解指引，適用於受控網域上可能會遇到的任何警示。


選擇與警示中的識別碼或訊息相對應的疑難排解步驟。

| **警示識別碼** | **警示訊息** | **解決方案** |
| --- | --- | :--- |
| AADDS001 | *受控網域已啟用透過網際網路的安全 LDAP。不過，卻未使用網路安全性群組鎖定連接埠 636 的存取。這可能會使受控網域上的使用者帳戶暴露於暴力密碼破解攻擊的威脅之下。* | [安全 LDAP 設定不正確](active-directory-ds-troubleshoot-ldaps.md) |
| AADDS100 | *與受控網域相關聯的 Azure AD 目錄可能已遭刪除。受控網域已不在支援的設定中。Microsoft 無法監視、管理、修補及同步處理受控網域。* | [遺失目錄](#aadds100-missing-directory) |
| AADDS101 | 無法在 Azure AD B2C 目錄中啟用 Azure AD Domain Services。 | [Azure AD B2C 在此目錄中執行](#aadds101-azure-ad-b2c-is-running-in-this-directory) |
| AADDS102 | *Azure AD 目錄中已刪除要讓 Azure AD Domain Services 正常運作所需的服務主體。此設定會影響 Microsoft 監視、管理、修補及同步處理受控網域的能力。* | [遺失服務主體](active-directory-ds-troubleshoot-service-principals.md) |
| AADDS103 | *您於其中啟用 Azure AD Domain Services 之虛擬網路的 IP 位址範圍位於公用 IP 範圍中。Azure AD Domain Services 必須在具有私人 IP 位址範圍的虛擬網路中啟用。此設定會影響 Microsoft 的監視、管理、修補及同步處理受控網域等功能。* | [位址位於公用 IP 範圍中](#aadds103-address-is-in-a-public-ip-range) |
| AADDS104 | *Microsoft 無法觸達此受控網域的網域控制站。如果虛擬網路上設定的網路安全性群組 (NSG) 封鎖受控網域的存取，就可能發生這種情況。另一個可能的原因，是使用者定義的路由封鎖了來自網際網路的連入流量。* | [網路錯誤](active-directory-ds-troubleshoot-nsg.md) |
| AADDS105 | *應用程式識別碼為 “d87dcbc6-a371-462e-88e3-28ad15ec4e64” 的服務主體已刪除，然後重新建立。重新建立會在為您受控網域提供服務所需的 Azure AD Domain Services 資源上留下不一致的權限。受控網域上的密碼同步處理可能會受到影響。* | [密碼同步處理應用程式已過期](active-directory-ds-troubleshoot-service-principals.md#alert-aadds105-password-synchronization-application-is-out-of-date) |
| AADDS500 | *受控網域前次是在 [date] 與 Azure AD 同步處理。* 使用者可能無法登入受控網域，或者群組成員資格可能無法與 Azure AD 同步。 | [有一陣子未發生同步處理](#aadds500-synchronization-has-not-completed-in-a-while) |
| AADDS501 | *受控網域前次是在 [date] 進行備份。* | [有一陣子未進行備份](#aadds501-a-backup-has-not-been-taken-in-a-while) |
| AADDS502 | *受控網域的安全 LDAP 憑證將於 [date] 到期。* | [安全 LDAP 憑證即將到期](active-directory-ds-troubleshoot-ldaps.md#aadds502-secure-ldap-certificate-expiring) |
| AADDS503 | *受控網域已擱置，因為與此網域相關聯的 Azure 訂用帳戶不在作用中。* | [因為停用的訂用帳戶而造成擱置](#aadds503-suspension-due-to-disabled-subscription) |
| AADDS504 | 受控網域因為無效的組態而造成擱置。此服務已無法針對受控網域管理、修補或更新網域控制站很長一段時間。 | [因為無效的組態而造成擱置](#aadds504-suspension-due-to-an-invalid-configuration) |


## <a name="aadds100-missing-directory"></a>AADDS100：遺失目錄
**警示訊息：**

*與受控網域相關聯的 Azure AD 目錄可能已遭刪除。受控網域已不在支援的設定中。Microsoft 無法監視、管理、修補及同步處理受控網域。*

**解決方案：**

造成這個錯誤的原因，通常是因為您錯誤地將 Azure 訂用帳戶移到新的 Azure AD 目錄，並將仍與 Azure AD Domain Services 相關聯的舊 Azure AD 目錄刪除。

此錯誤無法復原。 若要加以解決，您必須[刪除現有受控網域](active-directory-ds-disable-aadds.md)，然後在新目錄中予以重新建立。 如果您在刪除時遇到問題，請連絡 Azure Active Directory Domain Services 產品小組以[尋求支援](active-directory-ds-contact-us.md)。

## <a name="aadds101-azure-ad-b2c-is-running-in-this-directory"></a>AADDS101：Azure AD B2C 在此目錄中執行
**警示訊息：**

無法在 Azure AD B2C 目錄中啟用 Azure AD Domain Services。

**解決方案：**

>[!NOTE]
>若要繼續使用 Azure AD Domain Services，您必須在非 Azure AD B2C 目錄中重新建立 Azure AD Domain Services 執行個體。

若要將服務還原，請遵循下列步驟：

1. 從現有 Azure AD 目錄中[刪除受控網域](active-directory-ds-disable-aadds.md)。
2. 建立非 Azure AD B2C 目錄的新目錄。
3. 遵循[使用者入門](active-directory-ds-getting-started.md)指南，以重新建立受控網域。

## <a name="aadds103-address-is-in-a-public-ip-range"></a>AADDS103：位址位於公用 IP 範圍中

**警示訊息：**

*您於其中啟用 Azure AD Domain Services 之虛擬網路的 IP 位址範圍位於公用 IP 範圍中。Azure AD Domain Services 必須在具有私人 IP 位址範圍的虛擬網路中啟用。此設定會影響 Microsoft 的監視、管理、修補及同步處理受控網域等功能。*

**解決方案：**

> [!NOTE]
> 若要解決此問題，您必須刪除現有受控網域，然後在具有私人 IP 位址範圍的虛擬網路中加以重新建立。 此流程會造成干擾。

開始之前，請先閱讀[這篇文章](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces)中的**私人 IP v4 位址空間**一節。

在虛擬網路內，電腦可以針對所在 IP 位址範圍與為子網路設定之 IP 位址範圍相同的 Azure 資源提出要求。 不過，由於虛擬網路已針對此範圍做設定，因此將會在虛擬網路內路由傳送這些要求，而不會抵達所要的 Web 資源。 此組太可能導致 Azure AD Domain Services 發生無法預設測的錯誤。

**如果您擁有虛擬網路中所設定的網際網路 IP 位址範圍，便可以忽略此警示。不過，在有此設定的情況下，Azure AD Domain Services 無法承諾達成 [SLA](https://azure.microsoft.com/support/legal/sla/active-directory-ds/v1_0/)]，因為這會導致無法預測的錯誤。**


1. 從目錄中[刪除受控網域](active-directory-ds-disable-aadds.md)。
2. 修正子網路的 IP 位址範圍
  1. 瀏覽至 [Azure 入口網站上的虛擬網路頁面](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_AAD_DomainServices=preview#blade/HubsExtension/Resources/resourceType/Microsoft.Network%2FvirtualNetworks)。
  2. 選取您打算用於 Azure AD Domain Services 的虛擬網路。
  3. 按一下 [設定] 下的 [位址空間]
  4. 按一下現有位址範圍並加以編輯，或是新增其他位址範圍，來更新位址範圍。 請確定新的位址範圍位於私人 IP 範圍中。 儲存您的變更。
  5. 在左側導覽中按一下 [子網路]。
  6. 在資料表中按一下您要編輯的子網路。
  7. 更新位址範圍，並儲存變更。
3. 遵循[開始使用 Azure AD Domain Services 指南](active-directory-ds-getting-started.md)，以重新建立受控網域。 請確定您挑選的是具有私人 IP 位址範圍的虛擬網路。
4. 若要將虛擬機器的網域加入新的網域，請遵循[本指南](active-directory-ds-admin-guide-join-windows-vm-portal.md)。
8. 若要確保解決警示，請在兩小時內檢查您網域的健康情況。

## <a name="aadds500-synchronization-has-not-completed-in-a-while"></a>AADDS500：同步處理有一陣子未完成

**警示訊息：**

*受控網域前次是在 [date] 與 Azure AD 同步處理。* 使用者可能無法登入受控網域，或者群組成員資格可能無法與 Azure AD 同步。

**解決方案：**

[檢查您的網域健康狀態](active-directory-ds-check-health.md)是否有任何警示，其可能表示受控網域的組態有問題。 有時候，您的組態問題可能會妨礙 Microsoft 同步處理受控網域的能力。 如果您能夠解決任何警示，請等候兩小時並回頭查看是否已完成同步處理。

以下是受控網域上同步停止的一些常見原因：
- 受控網域上的網路連線已遭封鎖。 若要深入了解檢查網路是否存在問題的相關資訊，請參閱[針對受控網域的無效網路設定進行移難排解](active-directory-ds-troubleshoot-nsg.md)，以及 [Azure AD 網域服務的網路考量](active-directory-ds-networking.md)。
-  從未設定密碼同步化，或已完成同步。 若要設定密碼同步化，請參閱[這篇文章](active-directory-ds-getting-started-password-sync.md)。

## <a name="aadds501-a-backup-has-not-been-taken-in-a-while"></a>AADDS501：有一陣子未進行備份

**警示訊息：**

*受控網域前次是在 [date] 進行備份。*

**解決方案：**

[檢查您的網域健康狀態](active-directory-ds-check-health.md)是否有任何警示，其可能表示受控網域的組態有問題。 有時候，您的組態問題可能會妨礙 Microsoft 同步處理受控網域的能力。 如果您能夠解決任何警示，請等候兩小時並回頭查看是否已完成同步處理。


## <a name="aadds503-suspension-due-to-disabled-subscription"></a>AADDS503：因為停用的訂用帳戶而造成擱置

**警示訊息：**

*受控網域已擱置，因為與此網域相關聯的 Azure 訂用帳戶不在作用中。*

**解決方案：**

> [!WARNING]
> 如果您的受控網域已擱置一段時間，可能會遭到刪除。 請盡快處理擱置問題。 若要深入了解，請造訪[這篇文章](active-directory-ds-suspension.md)。

若要還原服務，[請更新與受控網域相關聯的 Azure 訂用帳戶](https://docs.microsoft.com/azure/billing/billing-subscription-become-disable)。

## <a name="aadds504-suspension-due-to-an-invalid-configuration"></a>AADDS504：因為無效的組態而造成擱置

**警示訊息：**

受控網域因為無效的組態而造成擱置。此服務已無法針對受控網域管理、修補或更新網域控制站很長一段時間。

**解決方案：**

> [!WARNING]
> 如果您的受控網域已擱置一段時間，可能會遭到刪除。 請盡快處理擱置問題。 若要深入了解，請造訪[這篇文章](active-directory-ds-suspension.md)。

[檢查您的網域健康狀態](active-directory-ds-check-health.md)是否有任何警示，其可能表示受控網域的組態有問題。 如果您可以解決任何警示，請這麼做。 之後，請連絡支援人員以重新啟用您的訂用帳戶。


## <a name="contact-us"></a>與我們連絡
請連絡 Azure Active Directory Domain Services 產品小組， [分享意見或尋求支援](active-directory-ds-contact-us.md)。
