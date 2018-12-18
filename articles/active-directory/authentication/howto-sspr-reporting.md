---
title: 自助式密碼重設報告 - Azure Active Directory
description: 報告 Azure AD 自助式密碼重設事件
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 5931958d17ea119e58c066a78a44169be2b9f451
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627520"
---
# <a name="reporting-options-for-azure-ad-password-management"></a>Azure AD 密碼管理的報告選項

部署之後，許多組織會想要知道如何或是否實際上正在使用自助式密碼重設 (SSPR)。 Azure Active Directory (Azure AD) 提供的報告功能可協助您使用預先建立的報告來回答問題。 如果您已適當地取得授權，則也可以建立自訂查詢。

![報告][Reporting]

存在於 [Azure 入口網站] 中的報告可以回答下列問題 (https://portal.azure.com/):)

> [!NOTE]
> 您必須是[全域管理員](../users-groups-roles/directory-assign-admin-roles.md)且至少必須已瀏覽一次報告索引標籤或稽核記錄。 若要選擇加入，您必須瀏覽 [報告] 索引標籤或稽核記錄至少一次。 在此之前，不會為您的組織收集資料。
>

* 有多少人已註冊密碼重設？
* 有誰已註冊密碼重設？
* 什麼資料是人員註冊？
* 有多少人在過去 7 天內重設密碼？
* 使用者或系統管理員用來重設其密碼最常見的方法是什麼？
* 當使用者或系統管理員嘗試使用密碼重設時所面臨的常見問題是什麼？
* 哪些系統管理員經常重設自己的密碼？
* 密碼重設時是否有任何可疑的活動？

## <a name="power-bi-content-pack"></a>Power BI 內容套件

如果您是 Power BI 使用者，可使用 Azure AD 的內容套件，其中包含 SSPR 的易用報告。 如需如何使用和部署內容套件的詳細資訊，請參閱[如何使用 Azure Active Directory Power BI 內容套件](../reports-monitoring/howto-power-bi-content-pack.md)。 利用內容套件，您可以建立自己的儀表板，並與貴組織中的其他人共用。

## <a name="how-to-view-password-management-reports-in-the-azure-portal"></a>如何在 Azure 入口網站中檢視密碼管理報告

在 Azure 入口網站體驗中，我們找到更好的方式來檢視密碼重設和密碼重設註冊活動。 使用下列步驟，尋找密碼重設和密碼重設註冊事件：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 在左側窗格中選取 [所有服務]。
3. 在服務清單中搜尋並選取 **Azure Active Directory**。
4. 選取 [使用者和群組]。
5. 從 [使用者和群組] 功能表選取 [稽核記錄]。 這會顯示目錄中所有使用者發生的稽核事件。 您可以篩選此檢視，查看密碼相關的所有事件。
6. 若要篩選此檢視以只查看密碼重設的相關事件，請按一下窗格頂端的 [篩選] 按鈕。
7. 從 [篩選] 功能表中，選取 [類別] 下拉式清單，然後將它變更為 [自助式密碼管理] 類別類型。
8. 另可選擇您有興趣的特定 [活動]，以進一步篩選清單。

### <a name="converged-registration-preview"></a>聚合式註冊 (預覽)

如果您參與聚合式註冊的公開預覽，則可在**驗證方法**的類別下方找到稽核記錄中有關使用者活動的資訊。

## <a name="description-of-the-report-columns-in-the-azure-portal"></a>Azure 入口網站中報告資料行的說明

下列清單詳細說明 Azure 入口網站中的每個報告資料行：

* **使用者**：嘗試進行密碼重設註冊作業的使用者。
* **角色**：使用者在目錄中的角色。
* **日期和時間**：嘗試的日期和時間。
* **已註冊資料**：使用者在密碼重設註冊期間提供哪些驗證資料。

## <a name="description-of-the-report-values-in-the-azure-portal"></a>Azure 入口網站中報告值的說明

下表描述您可以在 Azure 入口網站中為每個資料行設定的不同值：

| 欄 | 允許的值及其意義 |
| --- | --- |
| 已註冊資料 |**備用電子郵件**：使用者用來驗證的備用電子郵件或驗證電子郵件。<p><p>**辦公室電話**：使用者用來驗證的辦公室電話。<p>**行動電話**：使用者用來驗證的行動電話或驗證電話。<p>**安全性問題**：使用者使用安全性問題來驗證。<p>**前述方法的任何組合，例如替代電子郵件 + 行動電話**：當指定兩個閘道原則時會發生這種情況，且會顯示使用者用來驗證其密碼重設要求的兩種方法。 |

## <a name="self-service-password-management-activity-types"></a>自助式密碼管理活動類型

下列活動類型會出現在 [自助式密碼管理] 稽核事件類別中：

* [封鎖自助式密碼重設](#activity-type-blocked-from-self-service-password-reset)：指出使用者在 24 小時內嘗試重設密碼、使用特定的閘道或驗證電話號碼，總計五次以上。
* [變更密碼 (自助式)](#activity-type-change-password-self-service)：指出使用者自願或被迫 (因為到期) 執行密碼變更。
* [重設密碼 (由管理員)](#activity-type-reset-password-by-admin)：指出系統管理員從 Azure 入口網站代表使用者執行密碼重設。
* [重設密碼 (自助式)](#activity-type-reset-password-self-service)：指出使用者已從 [Azure AD 密碼重設入口網站](https://passwordreset.microsoftonline.com)成功重設其密碼。
* [自助式密碼重設流程活動進度](#activity-type-self-serve-password-reset-flow-activity-progress)：指出使用者在密碼重設過程中經過的每個特定步驟，例如，通過特定的密碼重設驗證關卡。
* [解除鎖定使用者帳戶 (自助式)](#activity-type-unlock-user-account-self-service)：指出使用者已使用 Active Directory 的帳戶解除鎖定而不重設功能，成功解除鎖定其 Active Directory 帳戶，而沒有從 [Azure AD 密碼重設入口網站](https://passwordreset.microsoftonline.com)重設其密碼。
* [使用者已註冊自助式密碼重設](#activity-type-user-registered-for-self-service-password-reset)：指出使用者已根據目前指定的租用戶密碼重設原則，註冊所有必要的資訊，以便能夠重設其密碼。

### <a name="activity-type-blocked-from-self-service-password-reset"></a>活動類型︰封鎖自助式密碼重設

下列清單詳細說明此活動︰

* **活動描述**：指出使用者在 24 小時內嘗試重設密碼、使用特定的閘道或驗證電話號碼，總計五次以上。
* **活動執行者**：已受節流控制而無法執行其他重設作業的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：已受節流控制而無法執行其他重設作業的使用者。 使用者可以是一般使用者或系統管理員。
* **活動狀態**：
  * _成功_：指出使用者已受節流控制，在接下來 24 小時內無法執行任何額外的重設、嘗試任何額外的驗證方法，或驗證任何額外的電話號碼。
* **活動狀態失敗原因**：不適用。

### <a name="activity-type-change-password-self-service"></a>活動類型：變更密碼 (自助式)

下列清單詳細說明此活動︰

* **活動描述**：指出使用者自願或被迫 (因為到期) 執行密碼變更。
* **活動執行者**：已變更其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：已變更其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **活動狀態**：
  * _成功_：指出使用者已成功變更其密碼。
  * _失敗_：指出使用者變更其密碼失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。
* **活動狀態失敗原因**： 
  * _FuzzyPolicyViolationInvalidPassword_：使用者選取的密碼自動被禁用，因為 Microsoft 的禁用密碼偵測功能發現此密碼太普通或太弱。

### <a name="activity-type-reset-password-by-admin"></a>活動類型：重設密碼 (由系統管理員)

下列清單詳細說明此活動︰

* **活動描述**：指出系統管理員從 Azure 入口網站代表使用者執行密碼重設。
* **活動執行者**：代表另一個使用者或系統管理員來執行密碼重設的系統管理員。 必須是全域管理員、密碼管理員、使用者管理員或技術服務管理員。
* **活動目標**：已重設密碼的使用者。 使用者可以是一般使用者或不同的系統管理員。
* **活動狀態**：
  * _成功_：指出系統管理員已成功重設使用者的密碼。
  * _失敗_：指出管理員變更使用者的密碼失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。

### <a name="activity-type-reset-password-self-service"></a>活動類型：重設密碼 (自助式)

下列清單詳細說明此活動︰

* **活動描述**：指出使用者已從 [Azure AD 密碼重設入口網站](https://passwordreset.microsoftonline.com)成功重設其密碼。
* **活動執行者**：已重設其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：已重設其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **活動狀態**：
  * _成功_：指出使用者已成功重設其密碼。
  * _失敗_：指出使用者重設其密碼失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。
* **活動狀態失敗原因**： 
  * _FuzzyPolicyViolationInvalidPassword_：系統管理員選取的密碼自動被禁用，因為 Microsoft 的禁用密碼偵測功能發現此密碼太普通或太弱。

### <a name="activity-type-self-serve-password-reset-flow-activity-progress"></a>活動類型：自助式密碼重設流程活動進度

下列清單詳細說明此活動︰

* **活動描述**：指出使用者在密碼重設過程中經過的每個特定步驟 (例如，通過特定的密碼重設驗證關卡)。
* **活動執行者**：執行部分密碼重設流程的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：執行部分密碼重設流程的使用者。 使用者可以是一般使用者或系統管理員。
* **活動狀態**：
  * _成功_：指出使用者已成功完成密碼重設流程的特定步驟。
  * _失敗_：指出密碼重設流程的特定步驟失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。
* **活動狀態原因**：請參閱下的表以取得[所有允許的重設活動狀態原因](#allowed-values-for-details-column)。

### <a name="activity-type-unlock-a-user-account-self-service"></a>活動類型：解除鎖定使用者帳戶 (自助式)

下列清單詳細說明此活動︰

* **活動描述**：指出使用者已使用 Active Directory 的帳戶解除鎖定而不重設功能，成功解除鎖定其 Active Directory 帳戶，而沒有從 [Azure AD 密碼重設入口網站](https://passwordreset.microsoftonline.com)重設其密碼。
* **活動執行者**：已解除鎖定帳戶但未重設其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：已解除鎖定帳戶但未重設其密碼的使用者。 使用者可以是一般使用者或系統管理員。
* **允許的活動狀態**：
  * _成功_：指出使用者已成功解除鎖定其帳戶。
  * _失敗_：指出使用者解除鎖定帳戶失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。

### <a name="activity-type-user-registered-for-self-service-password-reset"></a>活動類型︰使用者已註冊自助式密碼重設

下列清單詳細說明此活動︰

* **活動描述**：指出使用者已根據目前指定的租用戶密碼重設原則，註冊所有必要的資訊，以便能夠重設其密碼。 
* **活動執行者**：已註冊密碼重設的使用者。 使用者可以是一般使用者或系統管理員。
* **活動目標**：已註冊密碼重設的使用者。 使用者可以是一般使用者或系統管理員。
* **允許的活動狀態**：
  * _成功_：指出使用者已根據目前的原則成功註冊密碼重設。 
  * _失敗_：指出使用者註冊密碼重設失敗。 您可以選取該資料列，以查看 [活動狀態原因] 類別，深入了解失敗發生的原因。 

     >[!NOTE]
     >失敗並不代表使用者無法重設自己的密碼。 這表示它們未完成註冊程序。 如果帳戶上有未驗證但正確的資料 (例如未驗證的電話號碼)，即使此電話號碼尚未驗證，仍然可用來重設密碼。 如需詳細資訊，請參閱[當使用者註冊時會發生什麼事？](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-learn-more#what-happens-when-a-user-registers)。
     >

## <a name="next-steps"></a>後續步驟

* [如何完成 SSPR 成功首度發行？](howto-sspr-deployment.md)
* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)。
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)。
* [您有授權問題嗎？](concept-sspr-licensing.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [哪些驗證方法可供使用者使用？](concept-sspr-howitworks.md#authentication-methods)
* [使用 SSPR 的原則選項有哪些？](concept-sspr-policy.md)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [我認為有中斷。如何針對 SSPR 進行疑難排解？](active-directory-passwords-troubleshoot.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)

[Reporting]: ./media/howto-sspr-reporting/sspr-reporting.png "Azure AD 中 SSPR 活動稽核記錄的範例"
