---
title: Azure Active Directory Identity Protection | Microsoft Docs
description: 了解 Azure AD Identity Protection 如何讓您限制攻擊者利用遭入侵的身分識別或裝置的能力，以及保護先前疑似或已知遭到入侵的身分識別或裝置。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: e7434eeb-4e98-4b6b-a895-b5598a6cccf1
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/08/2017
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 06e3a596b60bf96319071fff68b0bf1655869559
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40003792"
---
# <a name="azure-active-directory-identity-protection"></a>Azure Active Directory Identity Protection

Azure Active Directory Identity Protection 是 Azure AD Premium P2 版本的一向功能，可讓您：

- 偵測會影響組織的身分識別的潛在弱點

- 針對偵測到的與您組織的身分識別有關的可疑動作，設定自動回應  

- 調查可疑事件並採取適當動作以解決它們   


## <a name="getting-started"></a>開始使用

Microsoft 保護雲端身分識別已經超過十多年。 在您的環境中使用 Azure Active Directory Identity Protection，即可使用與 Microsoft 用來保護身份識別一樣的保護系統。

大部分的安全性缺口出現於當攻擊者藉由竊取使用者的身分識別來取得環境的存取權時。 這些年來，攻擊者變得越來越擅於利用第三方缺口，以及使用複雜的網路釣魚攻擊。 只要攻擊者取得更低權限的使用者帳戶的存取權，他們即可相對容易地透過橫向移動存取重要的公司資源。

如此一來，您必須：

- 保護所有的識別身分，不論其權限層級

- 主動防止遭入侵的身分識別被濫用

探索遭入侵的身分識別並不容易。 Azure Active Directory 使用調適性機器學習運算法和啟發學習法來偵測表示可能遭入侵身份識別的異常與可疑事件。 Identity Protection 會使用此資料來產生報告和警示，讓您評估偵測到的問題並採取適當的緩和或補救動作。

Azure Active Directory Identity Protection 不只是監視和報告工具而已。 若要保護您組織的身分識別，您可以設定以風險為基礎的原則，當達到指定風險層級時自動回應偵測到的問題。 除了 Azure Active Directory 與 EMS 所提供的其他條件式存取控制以外，這些的原則可以自動封鎖或起始調適性補救動作，包括重設密碼以及強制 Multi-Factor Authentication。


#### <a name="identity-protection-capabilities"></a>Identity Protection 功能

**偵測弱點和風險帳戶：**  

* 提供自訂建議，藉由反白顯示弱點來改善整體安全性狀態
* 計算登入風險層級
* 計算使用者風險層級


**調查風險事件：**

* 傳送風險事件的通知
* 使用相關和內容資訊來調查風險事件
* 提供基本工作流程來追蹤調查
* 讓您輕鬆存取補救動作，例如重設密碼

**以風險為基礎的條件式存取原則：**

* 此原則可藉由封鎖登入或要求多重要素驗證挑戰，以儘量阻止高風險登入
* 此原則會封鎖或保護有風險的使用者帳戶
* 此原則會要求使用者註冊以便進行 Multi-Factor Authentication



## <a name="identity-protection-roles"></a>Identity Protection 角色

若要讓 Identity Protection 實作方面的管理活動達到負載平衡，您可以指派數個角色。 Azure AD Identity Protection 支援 3 種目錄角色：

| 角色                         | 可以執行                          | 無法執行
| :--                          | ---                                |  ---   |
| 全域管理員         | 完整存取 Identity Protection、將 Identity Protection 上架| |
| 安全性系統管理員       | 完整存取 Identity Protection | 將 Identity Protection 上架、重設使用者密碼 |
| 安全性讀取者              | 唯讀存取 Identity Protection | 讓 Identity Protection 上線、修復使用者、設定原則、重設密碼 |




如需詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](../users-groups-roles/directory-assign-admin-roles.md)



## <a name="detection"></a>偵測

### <a name="vulnerabilities"></a>弱點

Azure Active Directory Identity Protection 會分析您的組態，並偵測可能影響您使用者身份識別的弱點。 如需詳細資訊，請參閱 [Azure Active Directory Identity Protection 偵測到的弱點](vulnerabilities.md)。

### <a name="risk-events"></a>風險事件

Azure Active Directory 使用調適性機器學習運算法和啟發學習法來偵測與您使用者身份識別有關的可疑動作。 系統會針對每個偵測到的可疑動作建立記錄。 這些記錄又名風險事件。  
如需詳細資訊，請參閱 [Azure Active Directory風險事件](../active-directory-identity-protection-risk-events.md)。


## <a name="investigation"></a>調查
您通常會從 Identity Protection 儀表板開始使用 Identity Protection。

![補救](./media/overview/1000.png "補救")

儀表板可讓您存取：

* 報告，例如 [標示有風險的使用者]、[風險事件] 和 [弱點]
* 設定，例如 [安全性原則]、[通知] 和 [Multi-Factor Authentication 註冊] 的組態

這通常是調查的起點，而在調查過程中會檢閱風險事件相關活動、記錄檔和其他相關資訊，以決定是否需要採取補救或緩和步驟，了解身分識別如何遭到入侵，以及了解遭到入侵的身分識別如何被利用。

您可以將調查活動繫結至 Azure Active Directory Protection 傳送的每封電子郵件 [通知](notifications.md) 。

下列各節提供更多詳細資料以及調查的相關步驟。  


## <a name="risky-sign-ins"></a>有風險的登入

Azure Active Directory 會以即時和離線方式偵測[風險事件類型](../reports-monitoring/concept-risk-events.md#risk-event-types)。 使用者登入時偵測到的每個風險事件，構成了名為有風險的登入的邏輯概念。 有風險的登入表示可能不是由使用者帳戶合法擁有者執行的嘗試登入。


### <a name="sign-in-risk-level"></a>登入風險層級

登入風險層級指的是，不是由使用者帳戶合法擁有者執行的登入的可能性指示 (高、中或低)。

### <a name="mitigating-sign-in-risk-events"></a>緩和登入風險事件

緩和動作可限制攻擊者利用遭到入侵的身分識別或裝置的能力，但不需將身分識別或裝置還原至安全的狀態。 緩和並未解決先前與身分識別或裝置相關聯的登入風險事件。

若要自動降低有風險的登入，您可以設定登入風險安全性原則。 使用這些原則時，請考慮使用者或登入的風險層級，以封鎖有風險的登入或要求使用者執行 Multi-Factor Authentication。 這些動作可防止攻擊者利用遭竊的身分識別而造成損害，而且會讓您有一些時間可保護身分識別。

### <a name="sign-in-risk-security-policy"></a>登入風險安全性原則
登入風險原則是條件式存取原則，可評估特定登入的風險，並根據預先定義的條件和規則來套用緩和動作。

![登入風險原則](./media/overview/1014.png "登入風險原則")

Azure AD Identity Protection 可讓您執行下列作業，以協助您管理有風險登入的緩和動作：

* 設定要套用原則的使用者和群組：

    ![登入風險原則](./media/overview/1015.png "登入風險原則")
* 設定可觸發原則的登入風險層級臨界值 (低、中或高)：

    ![登入風險原則](./media/overview/1016.png "登入風險原則")
* 設定原則觸發時要強制執行的控制項︰  

    ![登入風險原則](./media/overview/1017.png "登入風險原則")
* 切換原則的狀態：

    ![MFA 註冊](./media/overview/403.png "MFA 註冊")
* 在啟用變更前檢閱和評估其影響：

    ![登入風險原則](./media/overview/1018.png "登入風險原則")

#### <a name="what-you-need-to-know"></a>您所需了解的事情
您可以設定登入風險安全性原則以要求 Multi-Factor Authentication︰

![登入風險原則](./media/overview/1017.png "登入風險原則")

不過，基於安全性理由，這項設定只適用於已註冊 Multi-Factor Authentication 的使用者。 如果尚未註冊 Multi-Factor Authentication 的使用者很滿意要求 Multi-Factor Authentication 的條件，則會封鎖該使用者。

最佳作法是，如果您想對有風險的登入要求 Multi-Factor Authentication，您應該︰

1. 為受影響的使用者啟用 [Multi-Factor Authentication 註冊原則](#multi-factor-authentication-registration-policy)。
2. 要求受影響的使用者登入沒有危險的工作階段以執行 MFA 註冊

完成這些步驟可確保有風險的登入一定需要 Multi-Factor Authentication。

#### <a name="best-practices"></a>最佳作法
選擇 [高]  臨界值可減少觸發原則的次數，並將對使用者的影響降至最低。  

不過，它會從原則中排除標示 [低] 和 [中] 度風險的登入，而無法阻止攻擊者利用遭到入侵的身分識別。

設定原則時，

* 排除不會 / 不能有 Multi-Factor Authentication 的使用者
* 在啟用原則並不實用 (例如無法存取技術服務人員) 的地區設定中排除使用者
* 排除可能會產生大量誤判的使用者 (開發人員、安全性分析人員)
* 在原則推出初期，或如果您必須盡量減少使用者所看到的挑戰，請使用 [高]  臨界值。
* 如果您的組織需要更高的安全性，請使用 [低]  臨界值。 選取 [低]  臨界值會帶來額外的使用者登入挑戰，並提高安全性。

大部分組織的建議預設值是設定 [中]  臨界值的規則，以取得可用性與安全性之間的平衡。

登入風險原則：

* 會套用至所有使用新式驗證的瀏覽器流量和登入。
* 不會套用至使用較舊安全性通訊協定的應用程式，其做法是停用位於同盟 IDP (例如 ADFS) 的 WS-Trust 端點。

Identity Protection 主控台中的 [風險事件]  頁面會列出所有事件：

* 此原則已套用至
* 您可以檢閱活動並判斷動作是否適當

如需相關的使用者經驗概觀，請參閱︰

* [有風險的登入復原](flows.md#risky-sign-in-recovery)
* [已封鎖有風險的登入](flows.md#risky-sign-in-blocked)  
* [使用 Azure AD Identity Protection 時的登入體驗](flows.md)  

**開啟相關的組態對話方塊**：

- 在 [Azure AD Identity Protection] 刀鋒視窗的 [設定] 區段中，按一下 [登入風險原則]。

    ![使用者風險原則](./media/overview/1014.png "使用者風險原則")



## <a name="users-flagged-for-risk"></a>標示有風險的使用者

由 Azure Active Directory 針對某個使用者所偵測到的所有[作用中風險事件](../reports-monitoring/concept-risk-events.md)，構成了名為使用者風險的邏輯概念。 標幟為有風險的使用者表示可能被盜用的使用者帳戶。

![標示有風險的使用者](./media/overview/1200.png)


### <a name="user-risk-level"></a>使用者風險層級

使用者風險層級可指出使用者的身分識別遭到入侵的可能性 (高、中或低)。 它會根據與使用者的身分識別相關聯的使用者風險事件進行計算。

風險事件的狀態為 [作用中] 或 [已關閉]。 只有 [作用中] 的風險事件會納入使用者風險層級計算。

使用下列輸入來計算使用者風險層級：

* 影響使用者的作用中風險事件
* 這些事件的風險層級
* 是否已採取任何補救動作

![使用者風險](./media/overview/1031.png "使用者風險")

您可以使用使用者風險層級來建立條件式存取原則，以阻止有風險的使用者進行登入，或迫使他們安全地變更其密碼。

### <a name="closing-risk-events-manually"></a>手動關閉風險事件

在大部分情況下，您將採取補救動作 (例如重設安全的密碼) 來自動關閉風險事件。 但是，這不一定都可行。  
比方說，在下列情況下即是如此︰

* 已刪除具有作用中風險事件的使用者
* 調查顯示合法的使用者已執行報告的風險事件

因為 [作用中]  的風險事件會納入使用者風險計算，所以您可能必須以手動方式降低風險層級，但不要手動關閉風險事件。  
在調查過程中，您可以選擇採取下列任何動作，以變更風險事件的狀態：

![動作](./media/overview/34.png "動作")

* **解決** - 如果在調查風險事件之後，您在 Identity Protection 外部採取適當的補救動作，而且您認為應該將風險事件視為已關閉，請將事件標示為 [已解決]。 解決的事件會將風險事件的狀態設定為 [已關閉]，而此風險事件便不再算是使用者風險。
* **標示為誤判** - 在某些情況下，您可能會調查某個風險事件並發現該事件被誤標為有風險。 您可以將風險事件標示為誤判，以減少發生這種情況。 這可協助機器學習演算法未來改善類似事件的分類。 誤判事件的狀態為 [已關閉]  ，而且不再算是使用者風險。
* **忽略** - 如果您尚未採取任何補救動作，但希望風險事件從作用中清單中移除，您可忽略風險事件，且事件狀態將會是 [已關閉]。 忽略的事件不算是使用者風險。 這個選項只能用於不尋常的情況下。
* **重新啟用** - 可以重新啟用已手動關閉的風險事件 (藉由選擇 [解決]、[誤判] 或 [略過])，並將事件狀態設回 [作用中]。 重新啟用的風險事件會納入使用者風險層級計算。 無法重新啟用透過補救 (例如重設安全的密碼) 關閉的風險事件。

**開啟相關的組態對話方塊**：

1. 在 [Azure AD Identity Protection] 刀鋒視窗的 [調查] 底下，按一下 [風險事件]。

    ![手動密碼重設](./media/overview/1002.png "手動密碼重設")
2. 在 [風險事件]  清單中，點選一個風險。

    ![手動密碼重設](./media/overview/1003.png "手動密碼重設")
3. 在 [風險] 刀鋒視窗中，按右鍵點選一個使用者。

    ![手動密碼重設](./media/overview/1004.png "手動密碼重設")

### <a name="closing-all-risk-events-for-a-user-manually"></a>手動關閉使用者的所有風險事件
Azure Active Directory Identity Protection 也提供按一下即可為使用者關閉所有事件的方法，而不需個別手動關閉使用者的風險事件。

![動作](./media/overview/2222.png "動作")

當您按一下 [關閉所有事件] ，所有事件都已關閉，且受影響的使用者不再有風險。

### <a name="remediating-user-risk-events"></a>補救使用者風險事件

補救就是用來保護先前疑似或已知遭到入侵的身分識別或裝置的動作。 補救動作可讓身分識別或裝置還原到安全的狀態，以及解決先前與身分識別或裝置相關聯的風險事件。

若要補救使用者風險事件，您可以：

* 手動重設安全密碼，以補救使用者風險事件
* 設定使用者風險安全性原則，以自動緩和或補救使用者風險事件
* 重新製作受感染裝置的映像  

#### <a name="manual-secure-password-reset"></a>手動重設安全密碼
重設安全的密碼是許多風險事件的有效補救動作，一旦執行，就會自動關閉這些風險事件並重新計算使用者風險層級。 您可以使用 Identity Protection 儀表板，為有風險的使用者起始密碼重設。

相關的對話方塊提供兩個不同的方法，可以將密碼重設為︰

**重設密碼** - 選取 [要求使用者重設密碼] 以允許使用者執行自助復原 (若該使用者已針對多重要素驗證註冊)。 在使用者下次登入期間，使用者必須成功解決 Multi-Factor Authentication 挑戰，且被迫變更密碼。 如果使用者帳戶尚未註冊 Multi-Factor Authentication，則無法使用此選項。

**暫時密碼** - 選取 [產生暫時密碼]，立即讓現有的密碼失效，並且為使用者建立新的暫時密碼。 將新的暫時密碼傳送到使用者的備用電子郵件地址，或傳送給使用者的經理。 因為此密碼是暫時的，所以會提示使用者在登入時變更密碼。

![原則](./media/overview/1005.png "原則")

**開啟相關的組態對話方塊**：

1. 在 [Azure AD Identity Protection] 刀鋒視窗上，按一下 [標示為危險的使用者]。

    ![手動密碼重設](./media/overview/1006.png "手動密碼重設")
2. 從使用者清單中，選取具有至少一個風險事件的使用者。

    ![手動密碼重設](./media/overview/1007.png "手動密碼重設")
3. 在 [使用者] 刀鋒視窗中，按一下 [重設密碼] 。

    ![手動密碼重設](./media/overview/1008.png "手動密碼重設")

### <a name="user-risk-security-policy"></a>使用者風險安全性原則
使用者風險安全性原則是條件式存取原則，可評估特定使用者的風險層級，並根據預先定義的條件和規則來套用補救和緩和動作。

![使用者風險原則](./media/overview/1009.png "使用者風險原則")

Azure AD Identity Protection 可讓您執行下列作業，以協助您管理標示有風險的使用者的緩和與補救動作：

* 設定要套用原則的使用者和群組：

    ![使用者風險原則](./media/overview/1010.png "使用者風險原則")
* 設定可觸發原則的使用者風險層級臨界值 (低、中或高)：

    ![使用者風險原則](./media/overview/1011.png "使用者風險原則")
* 設定原則觸發時要強制執行的控制項︰

    ![使用者風險原則](./media/overview/1012.png "使用者風險原則")
* 切換原則的狀態：

    ![使用者風險原則](./media/overview/403.png "MFA 註冊")
* 在啟用變更前檢閱和評估其影響：

    ![使用者風險原則](./media/overview/1013.png "使用者風險原則")

選擇 [高]  臨界值可減少觸發原則的次數，並將對使用者的影響降至最低。
不過，這會從原則中排除標示 [低] 和 [中] 度風險的使用者，而無法保護先前疑似或已知遭到入侵的身分識別或裝置。

設定原則時，

* 排除可能會產生大量誤判的使用者 (開發人員、安全性分析人員)
* 在啟用原則並不實用 (例如無法存取技術服務人員) 的地區設定中排除使用者
* 在原則推出初期，或如果您必須盡量減少使用者所看到的挑戰，請使用 [高]  臨界值。
* 如果您的組織需要更高的安全性，請使用 [低]  臨界值。 選取 [低]  臨界值會帶來額外的使用者登入挑戰，並提高安全性。

大部分組織的建議預設值是設定 [中]  臨界值的規則，以取得可用性與安全性之間的平衡。

如需相關的使用者經驗概觀，請參閱︰

* [遭到入侵的帳戶復原流程](flows.md#compromised-account-recovery)。  
* [遭到入侵的帳戶封鎖流程](flows.md#compromised-account-blocked)。  

**開啟相關的組態對話方塊**：

- 在 [Azure AD Identity Protection] 刀鋒視窗的 [設定] 區段中，按一下 [使用者風險原則]。

    ![使用者風險原則](./media/overview/1009.png "使用者風險原則")

### <a name="mitigating-user-risk-events"></a>緩和使用者風險事件
系統管理員可以設定使用者風險安全性原則，以根據風險層級防止使用者登入。

封鎖登入：

* 避免對受影響的使用者產生新的使用者風險事件
* 可讓系統管理員手動地補救會影響使用者身分識別的風險事件，並將它還原到安全的狀態



## <a name="multi-factor-authentication-registration-policy"></a>Multi-Factor Authentication 註冊原則
Azure Multi-Factor Authentication 是除了使用使用者名稱與密碼之外，需要再利用其他方法驗證身份的驗證方法。 它可以為使用者登入和交易提供第二層安全性。  
建議您要求對使用者登入進行 Azure Multi-Factor Authentication，因為它：

* 提供增強式驗證與一系列簡單的驗證選項
* 扮演著舉足輕重的角色，可讓您的組織防護帳戶遭竊及從中復原

![使用者風險原則](./media/overview/1019.png "使用者風險原則")

如需詳細資訊，請參閱 [什麼是 Azure Multi-Factor Authentication？](../authentication/multi-factor-authentication.md)

Azure AD Identity Protection 可讓您設定原則來執行下列作業，以協助您管理首次 Multi-Factor Authentication 註冊：

* 設定要套用原則的使用者和群組：

    ![MFA 原則](./media/overview/1020.png "MFA 原則")
* 設定原則觸發時要強制執行的控制項︰：  

    ![MFA 原則](./media/overview/1021.png "MFA 原則")
* 切換原則的狀態：

    ![MFA 原則](./media/overview/403.png "MFA 原則")
* 檢視目前註冊狀態：

    ![MFA 原則](./media/overview/1022.png "MFA 原則")

如需相關的使用者經驗概觀，請參閱︰

* [Multi-Factor Authentication 註冊流程](flows.md#multi-factor-authentication-registration)。  
* [使用 Azure AD Identity Protection 時的登入體驗](flows.md)。  

**開啟相關的組態對話方塊**：

- 在 [Azure AD Identity Protection] 刀鋒視窗的 [設定] 區段中，按一下 [Multi-Factor Authentication 註冊]。

    ![MFA 原則](./media/overview/1019.png "MFA 原則")

## <a name="next-steps"></a>後續步驟
* [第 9 頻道：Azure AD 和身分識別展示：Identity Protection 預覽](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

* [啟用 Azure Active Directory Identity Protection](enable.md)

* [Azure Active Directory Identity Protection 偵測到的弱點](vulnerabilities.md)

* [Azure Active Directory 風險事件](../active-directory-identity-protection-risk-events.md)

* [Azure Active Directory Identity Protection 通知](notifications.md)

* [Azure Active Directory Identity Protection 腳本](playbook.md)

* [Azure Active Directory Identity Protection 詞彙](glossary.md)

* [使用 Azure AD Identity Protection 時的登入體驗](flows.md)

* [Azure Active Directory Identity Protection - 如何解鎖使用者](howto-unblock-user.md)

* [開始使用 Azure Active Directory Identity Protection 和 Microsoft Graph](graph-get-started.md)
