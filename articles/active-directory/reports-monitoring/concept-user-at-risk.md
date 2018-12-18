---
title: Azure Active Directory 入口網站中標幟為有風險的使用者安全性報告 | Microsoft Docs
description: 了解 Azure Active Directory 入口網站中標幟為有風險的使用者安全性報告
services: active-directory
author: priyamohanram
manager: mtillman
ms.assetid: addd60fe-d5ac-4b8b-983c-0736c80ace02
ms.service: active-directory
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 11/14/2017
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: fde063cb593ca1f610dc35cd044fe41e34ab9202
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45578349"
---
# <a name="users-flagged-for-risk-security-report-in-the-azure-active-directory-portal"></a>Azure Active Directory 入口網站中標幟為有風險的使用者安全性報告

利用 Azure Active Directory (Azure AD) 中的安全性報告，您可以深入了解環境中使用者帳戶被盜用的可能性。 

Azure Active Directory 會偵測使用者帳戶相關的可疑動作。 針對每個偵測到的動作，將會建立一筆稱為「風險事件」的記錄。 如需詳細資訊，請參閱 [Azure Active Directory 風險事件](concept-risk-events.md)。 

偵測到的風險事件用來計算︰

- **有風險的登入** - 有風險的登入表示非使用者帳戶合法擁有者的某人嘗試登入。 如需詳細資訊，請參閱[如何設定登入風險原則](../identity-protection/howto-sign-in-risk-policy.md)。 

- **標幟為有風險的使用者** - 有風險的使用者表示可能被盜用的使用者帳戶。 如需詳細資訊，請參閱[如何設定使用者風險原則](../identity-protection/howto-user-risk-policy.md)。  

在 Azure 入口網站中，您可以在 [Azure Active Directory] 刀鋒視窗的 [安全性] 區段中找到安全性報告。  

![有風險的登入](./media/concept-user-at-risk/10.png)



## <a name="what-azure-ad-license-do-you-need-to-access-a-security-report"></a>您需要哪項 Azure AD 授權才能存取安全性報告？  

所有 Azure Active Directory 版本都會為您提供標幟為有風險的使用者報告。  
不過，報告細微性層級因版本而異： 

- 在 [Azure Active Directory Free 和 Basic 版本] 中，您已取得標幟為有風險的使用者清單。 

- **Azure Active Directory Premium 1** 版本也可讓您檢查每份報告部分已偵測到的基礎風險事件，藉此擴充此模型。 

- **Azure Active Directory Premium 2** 版本可提供有關所有基礎風險事件的最詳細資訊，可讓您設定安全性原則，自動回應已設定的風險層級。



## <a name="azure-active-directory-free-and-basic-edition"></a>Azure Active Directory 免費和基本版本

Azure Active Directory 免費和基本版本中標幟為有風險的使用者報告，會提供可能遭到盜用的使用者帳戶清單。 


![有風險的登入](./media/concept-user-at-risk/03.png)

選取使用者，即會開啟相關的使用者資料刀鋒視窗。
針對有風險的使用者，您可以檢閱使用者的登入記錄，如有必要，請重設密碼。

![有風險的登入](./media/concept-user-at-risk/46.png)


此對話方塊會提供選項以便：

- 下載報告

- 搜尋使用者

![有風險的登入](./media/concept-user-at-risk/16.png)


## <a name="azure-active-directory-premium-editions"></a>Azure Active Directory Premium Edition

Azure Active Directory Premium Edition 中標幟為有風險的使用者報告可提供：

- 可能已遭盜用的使用者帳戶清單 

- 關於已偵測到之[風險事件類型](concept-risk-events.md)的彙總資訊

- 下載報告的選項

- 選擇設定[使用者風險補救原則](../identity-protection/howto-user-risk-policy.md)  


![有風險的登入](./media/concept-user-at-risk/71.png)

當您選取使用者時，即會取得這位使用者的詳細報告檢視，讓您能夠：

- 開啟 [所有登入] 檢視

- 重設使用者的密碼

- 關閉所有事件

- 調查針對該使用者報告的風險事件。 


![有風險的登入](./media/concept-user-at-risk/324.png)


若要調查風險事件，請從清單中選取一個，以開啟此風險事件的 [詳細資料] 刀鋒視窗。 在 [詳細資料] 刀鋒視窗上，您可以選擇手動關閉風險事件或重新啟動已手動關閉的風險事件。 


![有風險的登入](./media/concept-user-at-risk/325.png)



## <a name="next-steps"></a>後續步驟

- 如需 Azure Active Directory Identity Protection 的詳細資訊，請參閱 [Azure Active Directory Identity Protection](../active-directory-identityprotection.md)。

