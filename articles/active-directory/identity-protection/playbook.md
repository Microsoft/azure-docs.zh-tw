---
title: Azure Active Directory Identity Protection 腳本 | Microsoft Docs
description: 了解 Azure AD Identity Protection 如何讓您限制攻擊者利用遭入侵的身分識別或裝置的能力，以及保護先前疑似或已知遭到入侵的身分識別或裝置。
services: active-directory
keywords: azure active directory identity protection, cloud discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: 60836abf-f0e9-459d-b344-8e06b8341d25
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 02d402b7de82631ce459c60cb42e5335c7e7cfe3
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40003295"
---
# <a name="azure-active-directory-identity-protection-playbook"></a>Azure Active Directory Identity Protection 腳本

這個腳本可協助您︰

* 藉由模擬風險事件和弱點，在 Identity Protection 環境中填入資料
* 設定以風險為基礎的條件式存取原則，並測試這些原則的影響


## <a name="simulating-risk-events"></a>模擬風險事件

本節將為您提供模擬下列風險事件類型的步驟：

* 從匿名 IP 位址登入 (容易)
* 從不熟悉的位置登入 (適中)
* 不可能到達非典型位置的移動 (困難)

無法以安全的方式模擬其他風險事件。

### <a name="sign-ins-from-anonymous-ip-addresses"></a>從匿名 IP 位址登入

如需此風險事件的詳細資訊，請參閱[從匿名 IP 位址登入](../reports-monitoring/concept-risk-events.md#sign-ins-from-anonymous-ip-addresses)。 

要完成下列程序，您必須使用：

- [Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en)，用以模擬匿名 IP 位址。 如果您的組織對 Tor 瀏覽器的使用有所限制，您可能需要使用虛擬機器。
- 尚未註冊多重要素驗證的測試帳戶。

**若要模擬從匿名 IP 登入，請執行下列步驟**：

1. 使用 [Tor 瀏覽器](https://www.torproject.org/projects/torbrowser.html.en)，瀏覽至 [https://myapps.microsoft.com](https://myapps.microsoft.com)。   
2. 輸入您要在 [從匿名 IP 位址登入]  報告中顯示之帳戶的認證。

登入將會在 10 - 15 分鐘內顯示於 [身分識別保護] 儀表板上。 

### <a name="sign-ins-from-unfamiliar-locations"></a>從不熟悉的位置登入

如需此風險事件的詳細資訊，請參閱[從不熟悉的位置登入](../reports-monitoring/concept-risk-events.md#sign-in-from-unfamiliar-locations)。 

若要模擬不熟悉的位置，您必須從測試帳戶未曾用來登入的位置和裝置進行登入。

下列程序會使用新建立的：

- VPN 連線，用以模擬新位置。

- 虛擬機器，用以模擬新裝置。

要完成下列程序，您必須使用符合下列條件的使用者帳戶：

- 至少有 30 天的登入歷程記錄。
- 已啟用多重要素驗證。


**若要模擬從不熟悉的位置登入，請執行下列步驟**：

1. 以您的測試帳戶登入時，藉由不傳遞 MFA 挑戰而使 MFA 挑戰失敗。
2. 使用新的 VPN 導覽至 [https://myapps.microsoft.com](https://myapps.microsoft.com)，然後輸入您測試帳戶的認證。
   

登入將會在 10 - 15 分鐘內顯示於 [身分識別保護] 儀表板上。

### <a name="impossible-travel-to-atypical-location"></a>不可能到達非典型位置的移動

如需此風險事件的詳細資訊，請參閱[不可能到達非慣用位置](../reports-monitoring/concept-risk-events.md#impossible-travel-to-atypical-locations)。 

模擬不可能的移動情況相當困難，因為此演算法會使用機器學習服務來剔除誤判，例如，不可能來自熟悉裝置的移動，或從目錄中其他使用者所用的 VPN 登入。 此外，使用者必須要有 14 天的登入歷程記錄，且登入 10 次，此演算法才會開始產生風險事件。 由於機器學習模型的複雜性和前述規則，下列步驟有可能不會導致風險事件。 您可以為多個 Azure AD 帳戶複寫這些步驟，以發佈此風險事件。


**若要模擬不可能到達非典型位置的移動，請執行下列步驟**：

1. 使用標準瀏覽器，瀏覽至 [https://myapps.microsoft.com](https://myapps.microsoft.com)。  
2. 輸入您想要對其產生不可能移動風險事件之帳戶的認證。
3. 變更您的使用者代理程式。 您可以在 Internet Explorer 中從 [開發人員工具] 變更使用者代理程式，或在 Firefox 或 Chrome 中使用使用者代理程式切換器附加元件來變更您的使用者代理程式。
4. 變更您的 IP 位址。 使用 VPN、Tor 附加元件，或在不同資料中心於 Azure 中啟動新機器，即可變更您的 IP 位址。
5. 在前次登入之後的幾分鐘內，使用與之前相同的認證登入 [https://myapps.microsoft.com](https://myapps.microsoft.com)。

登入會在 2-4 小時內顯示於 [身分識別保護] 儀表板上。

## <a name="simulating-vulnerabilities"></a>模擬弱點
弱點是 Azure AD 環境中不良執行者可以利用的弱點。 Azure AD Identity Protection 中目前顯示 3 種會運用其他 Azure AD 功能的弱點。 一旦設定好這些功能，這些弱點就會自動顯示在 Identity Protection 儀表板上。

* Azure AD [多重要素驗證](../authentication/multi-factor-authentication.md)
* Azure AD [Cloud Discovery](https://docs.microsoft.com/cloud-app-security/).
* Azure AD [Privileged Identity Management](../privileged-identity-management/pim-configure.md)。 


## <a name="testing-security-policies"></a>測試安全性原則

本節為您提供測試使用者風險和登入風險安全性原則的步驟。


### <a name="user-risk-security-policy"></a>使用者風險安全性原則

如需詳細資訊，請參閱[使用者風險安全性原則](overview.md#user-risk-security-policy)。

![使用者風險](./media/playbook/02.png "腳本")


**若要測試使用者風險安全性原則，請執行下列步驟**：

1. 使用租用戶的全域管理員認證來登入 [https://portal.azure.com](https://portal.azure.com)。
2. 瀏覽至 [Identity Protection] 。 
3. 在 [Azure AD Identity Protection] 頁面上，按一下 [使用者風險原則]。
4. 在 [指派] 區段中，選取所需的使用者 (與群組) 和使用者風險層級。

    ![使用者風險](./media/playbook/03.png "腳本")

5. 在 [控制] 區段中，選取所需的存取控制 (例如，需要變更密碼)。
5. 針對 [強制執行原則]，選取 [關閉]。
6. 提高測試帳戶的使用者風險，例如，藉由模擬其中一個風險事件多次來執行。
7. 等候幾分鐘，然後確認使用者的使用者層級為 [中]。 如果不是，請為使用者模擬更多風險事件。
8. 針對 [強制執行原則]，選取 [開啟]。
9. 您現在可以使用風險層級已提高的使用者進行登入，以測試以使用者風險為基礎的條件式存取。
    
    

### <a name="sign-in-risk-security-policy"></a>登入風險安全性原則

如需詳細資訊，請參閱[使用者風險安全性原則](overview.md#user-risk-security-policy)。

![登入風險](./media/playbook/01.png "腳本")


**若要測試登入風險原則，請執行下列步驟：**

1. 使用租用戶的全域管理員認證來登入 [https://portal.azure.com](https://portal.azure.com)。

2. 導覽至 **Azure AD Identity Protection**。

3. 在 [Azure AD Identity Protection] 主頁面上，按一下 [登入風險原則]。 

4. 在 [指派] 區段中，選取所需的使用者 (與群組) 和登入風險層級。

    ![登入風險](./media/playbook/04.png "腳本")


5. 在 [控制] 區段中，選取所需的存取控制 (例如，**需要多重要素驗證**)。 

6. 針對 [強制執行原則]，選取 [開啟]。

7. 按一下 [檔案] 。

8. 您現在可以使用具風險的工作階段 (例如，藉由使用 Tor 瀏覽器) 進行登入，以測試以登入風險為基礎的條件式存取。 

 




## <a name="see-also"></a>另請參閱

- [Azure Active Directory Identity Protection](../active-directory-identityprotection.md)

