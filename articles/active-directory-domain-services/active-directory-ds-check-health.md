---
title: Azure AD Domain Services：檢查受控網域的健康情況 | Microsoft Docs
description: 使用 Azure 入口網站中的健康情況頁面來檢查受控網域的健康情況。
services: active-directory-ds
documentationcenter: ''
author: eringreenlee
manager: mtillman
editor: curtand
ms.assetid: 8999eec3-f9da-40b3-997a-7a2587911e96
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/12/2018
ms.author: ergreenl
ms.openlocfilehash: a9248e2310e509bbc4f527608246b47543a03fc4
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502172"
---
# <a name="check-the-health-of-an-azure-ad-domain-services-managed-domain"></a>檢查 Azure AD Domain Services 受控網域的健康情況

## <a name="overview-of-the-health-page"></a>健康情況頁面概觀
使用 Azure 入口網站上的健康情況頁面時，您可以了解受控網域的最新情況。 本文將逐步解說健康情況頁面的元素。

### <a name="how-to-view-the-health-of-your-managed-domain"></a>如何檢視受控網域的健康情況
1. 在 Azure 入口網站中，瀏覽至 [Azure AD Domain Services 頁面](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.AAD%2FdomainServices)。
2. 按一下您想要檢視健康情況的網域。
3. 在左側導覽窗格中，按一下 [健康情況]。

下圖顯示一個範例健康情況頁面：![範例健康情況頁面](.\media\active-directory-domain-services-alerts\health-page.png)

>[!NOTE]
> 受控網域的評估頻率為每小時一次。 對您的受控網域進行變更之後，必須等到下一個評估週期，才能檢視受控網域的已更新健康情況。 右上角的 [上次評估時間] 時間戳記會顯示上次評估您受控網域健康情況的時間。
>

### <a name="status-of-your-managed-domain"></a>受控網域的狀態
健康情況頁面右上角的狀態會指出您受控網域的整體健康情況。 此狀態會將您網域上所有現有的警示都納入為考慮因素。 您也可以在 Azure AD Domain Services 的概觀頁面上檢視網域的狀態。

| 狀態 | 圖示 | 說明 |
| --- | :----: | --- |
| 執行中 | <img src= ".\media\active-directory-domain-services-alerts\running-icon.png" width = "15"> | 您的受控網域執行順暢，沒有任何嚴重或警告警示。 此網域可能有資訊警示。 |
| 需注意 (警告) | <img src= ".\media\active-directory-domain-services-alerts\warning-icon.png" width = "15"> | 您的受控網域上沒有任何嚴重警示，但有一或多個需要解決的警告警示。 |
| 需注意 (嚴重) | <img src= ".\media\active-directory-domain-services-alerts\critical-icon.png" width = "15"> | 您的受控網域上有一或多個嚴重警示。 您可能也有警告和/或資訊警示。 |
| 正在部署 | <img src= ".\media\active-directory-domain-services-alerts\deploying-icon.png" width = "15"> | 您的網域正在部署中。 |

## <a name="monitors"></a>監視器
監視器係指 Azure AD Domain Services 定期監視的受控網域層面。 讓您監視器保持在健全狀態的最佳方法，就是解決受控網域的所有作用中警示。

Azure AD Domain Services 目前會監視下列項目：
 - Backup 
 - 與 Azure AD 的同步處理

### <a name="the-backup-monitor"></a>「備份」監視器
這會監視受控網域的定期備份是否有在執行。 下表說明備份監視器的詳細資料資料行中應有的內容：

| 詳細資料值 | 說明 |
| --- | --- |
|「從未備份」 | 此狀態對於新建立的受控網域來說是正常狀態。 一般而言，會在佈建受控網域的 24 小時後才建立第一次備份。 如果您的受控網域不是新建立的，或是您看到此狀態的持續時間異常的長，請[連絡支援人員](active-directory-ds-contact-us.md)。 |
| 上次備份時間是在 1 到 14 天之前 | 這通常是備份監視器應有的值。 |
| 上次備份時間已超過 14 天之前。 | 任何自上次備份後超過兩週的時間都是異常的長時間。 作用中的嚴重警示可能會導致無法定期備份您的受控網域。 請先解決受控網域的所有作用中警示，然後如果問題仍持續存在，則請[連絡支援人員](active-directory-ds-contact-us.md)。 |


### <a name="the-synchronization-with-azure-ad-monitor"></a>「與 Azure AD 的同步處理」監視器
Microsoft 會監視受控網域與 Azure Active Directory 進行同步處理的頻率。 物件 (使用者和群組) 數目及自上次同步處理後在您 Azure AD 目錄中進行的變更數目，都會影響同步處理期間所花費的時間。 如果您受控網域上次進行同步處理的時間已超過三天之前，請[連絡支援人員](active-directory-ds-contact-us.md)。

## <a name="alerts"></a>警示
系統會針對您受控網域上必須解決才能讓 Azure AD Domain Services 執行的問題產生警示。 每個警示都會說明問題並提供解決方式 URL，以概要說明解決問題的特定步驟。 若要檢視所有警示及其解決方式，請瀏覽[疑難排解警示](active-directory-ds-troubleshoot-alerts.md)一文。

### <a name="alert-severity"></a>警示嚴重性
警示分成三種不同的嚴重性層級：嚴重、警告及資訊。

 * **嚴重警示**係指嚴重影響您受控網域的問題。 這些是應該立即解決的警示，因為 Microsoft 會無法監視、管理、修補及同步處理您的受控網域。 
 * **警告警示**會通知您未來可能影響您受控網域的問題。 這些警示會提供建議來保護您的受控網域。
 * **資訊警示**係指不會對您網域造成負面影響的通知。 資訊警示的設計目的是要讓您隨時掌握網域及 Azure AD Domain Services 中的最新情況。

## <a name="next-steps"></a>後續步驟
- [解決受控網域上的警示](active-directory-ds-troubleshoot-alerts.md)
- [深入了解 Azure AD Domain Services](active-directory-ds-overview.md)
- [連絡產品小組](active-directory-ds-contact-us.md)
