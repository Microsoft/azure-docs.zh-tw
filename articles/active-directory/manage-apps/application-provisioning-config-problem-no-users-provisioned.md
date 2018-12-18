---
title: 未將任何使用者佈建至 Azure AD 資源庫應用程式 | Microsoft Docs
description: 如何對當您沒有看到 Azure AD 資源庫應用程式中顯示任何使用者時所面臨的常見問題進行疑難排解，而該應用程式是您已設定來搭配 Azure AD 進行使用者佈建
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/20/2018
ms.author: barbkess
ms.reviewer: asteen
ms.openlocfilehash: 3874e6ff6586726577a2c89e3cf45bbd3343b821
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040681"
---
# <a name="no-users-are-being-provisioned-to-an-azure-ad-gallery-application"></a>未將任何使用者佈建至 Azure AD 資源庫應用程式
為應用程式設定自動佈建 (包括驗證供給 Azure AD 以連接至應用程式的應用程式認證提是有效的) 之後，使用者及/或群組就會佈建至應用程式。 佈建是由下列事項決定：

-   已將哪些使用者和群組**指派**給應用程式。 如需指派的詳細資訊，請參閱[在 Azure Active Directory 中將使用者或群組指派給企業應用程式](assign-user-or-group-access-portal.md)。
-   是否已啟用**屬性對應**，並設定為將來自 Azure AD 的有效屬性同步處理至應用程式。 如需詳細資訊，請參閱[在 Azure Active Directory 中自訂 SaaS 應用程式的使用者佈建屬性對應](customize-application-attributes.md)。
-   是否存在**範圍設定篩選**，這會根據根據特定的屬性值來篩選使用者。 如需範圍設定篩選條件的詳細資訊，請參閱[含範圍篩選器的屬性型應用程式佈建](define-conditional-rules-for-provisioning-user-accounts.md)。
  
若觀察到尚未佈建使用者，請參閱 Azure AD 中的稽核記錄。 搜尋特定使用者的記錄項目。

您可以在 Azure 入口網站的 [Azure Active Directory]&gt;[企業應用程式]&gt;[\[應用程式名稱\]]&gt;[稽核記錄] 索引標籤中存取佈建稽核記錄。基於 [帳戶佈建] 類別來篩選記錄，只顯示該應用程式的佈建事件。 您可以根據「比對識別碼」來搜尋使用者，此識別碼是在屬性對應中針對使用者所設定。 例如，如果您設定了「使用者主體名稱」或「電子郵件地址」做為 Azure AD 端的比對屬性，而且未佈建的使用者具有 "audrey@contoso.com" 的值，接著搜尋 “audrey@contoso.com” 的稽核記錄，然後檢閱傳回的項目。

佈建稽核記錄會記錄佈建服務所執行的所有作業，包括針對位於佈建範圍內的已指派使用者查詢 Azure AD、查詢目標應用程式以確認那些使用者是否存在、比較系統之間的使用者物件。 接著，根據比較，在目標系統中新增、更新或停用使用者帳戶。

## <a name="general-problem-areas-with-provisioning-to-consider"></a>關於佈建要考慮的一般問題領域
如果您已知道要從何處著手，則以下是您可以向下切入的一般問題領域清單。

- [佈建服務似乎未啟動](#provisioning-service-does-not-appear-to-start)
- [稽核記錄表示「已略過」且未佈建使用者，即使已指派它們](#audit-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned)

## <a name="provisioning-service-does-not-appear-to-start"></a>佈建服務似乎未啟動
如果您在 Azure 入口網站的 [Azure Active Directory]&gt;[企業應用程式]&gt;[\[應用程式名稱\]]&gt;[佈建] 區段中，將 [佈建狀態] 設為 [啟用]。 但是，在後續重新載入之後，該頁面上並未顯示任何其他狀態詳細資料，很可能是服務正在執行，但尚未完成初始同步處理。 請檢查上述的**稽核記錄**，以判斷服務正在執行哪些作業，以及是否發生任何錯誤。

>[!NOTE]
>根據 Azure AD 目錄大小和佈建範圍中的使用者數目而定，初始同步處理可能會花費 20 分鐘到數小時。 初始同步處理之後的後續同步處理會更快，因為佈建服務會儲存浮水印，代表兩個系統在初始同步處理之後的狀態。初始同步會改善後續同步處理的效能。
>


## <a name="audit-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned"></a>稽核記錄表示「已略過」且未佈建使用者，即使已指派它們

當使用者在稽核記錄中顯示為「已略過」時，請務必讀取記錄訊息中延伸的詳細資料來判斷原因。 下面是常見原因和解決方式：

- **已設定範圍設定篩選**，**這會根據屬性值篩選出使用者**。 如需有關範圍設定篩選的詳細資訊，請參閱[範圍篩選器](define-conditional-rules-for-provisioning-user-accounts.md)。
- **使用者「不具有效權限」。** 如果您看到此特定錯誤訊息，這是因為儲存在 Azure AD 中的使用者指派記錄發生問題。 若要修正此問題，請從應用程式解除指派使用者 (或群組)，然後重新指派一次。 如需有關指派的詳細資訊，請參閱[指派使用者或群組存取](assign-user-or-group-access-portal.md)。
- **必要屬性已遺失或未針對使用者填入。** 設定佈建時必須考量的重點是，檢視和設定屬性 (Attribute) 對應，以及定義哪些使用者 (或群組) 屬性 (Property) 會從 Azure AD 流向應用程式的工作流程。 此組態包括設定「比對屬性」，此屬性可用於唯一識別並比對兩個系統之間的使用者/群組。 如需這個重要程序的詳細資訊，請參閱[在 Azure Active Directory 中自訂 SaaS 應用程式的使用者佈建屬性對應](customize-application-attributes.md)。
- **群組的屬性對應：** 除了成員，還有群組名稱和群組詳細資訊的佈建 (如果某些應用程式有支援)。 您可以對 [佈建] 索引標籤中顯示的群組物件啟用或停用 [對應]，以啟用或停用此功能。如果已啟用佈建群組，請務必檢閱屬性對應，以確保「比對識別碼」使用適當的欄位。 此比對識別碼可以是顯示名稱或電子郵件別名。 如果 Azure AD 中某個群組的比對屬性空白或未填入，則不會佈建群組及其成員。

## <a name="next-steps"></a>後續步驟

[Azure AD Connect 同步處理：了解宣告式佈建](../hybrid/concept-azure-ad-connect-sync-declarative-provisioning.md)
