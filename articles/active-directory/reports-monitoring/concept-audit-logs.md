---
title: Azure Active Directory 入口網站中的稽核活動報告 | Microsoft Docs
description: 介紹 Azure Active Directory 入口網站中的稽核活動報告
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 04/19/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: b6fa26cb7947658af77496831d7239b4331aa1f2
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42141054"
---
# <a name="audit-activity-reports-in-the-azure-active-directory-portal"></a>Azure Active Directory 入口網站中的稽核活動報告 

透過 Azure Active Directory 中的報告，您可以取得判斷您的環境執行狀況所需的資訊。

Azure AD 中的報告架構包含下列元件：

- **活動** 
    - **登入活動** – 受控應用程式和使用者登入活動的使用情況相關資訊
    - **稽核記錄** - 可針對各種功能在 Azure AD 內進行的所有變更，提供記錄追蹤功能。 稽核記錄的範例包括使用者、應用程式、群組、角色、原則、驗證等任何資源在 Azure AD 中所做的變更。
- **安全性** 
    - **有風險的登入** - 有風險的登入表示非使用者帳戶合法擁有者的某人嘗試登入。 如需詳細資訊，請參閱＜有風險的登入＞。
    - **標幟為有風險的使用者** - 有風險的使用者表示可能被盜用的使用者帳戶。 如需詳細資訊，請參閱＜標幟為有風險的使用者＞。

本主題提供稽核活動的概觀。
 
## <a name="who-can-access-the-data"></a>誰可以存取資料？
* 具有安全性系統管理員或安全性讀取器角色的使用者
* 全域管理員
* 個別使用者 (非系統管理員) 可以看到自己的活動


## <a name="audit-logs"></a>稽核記錄

Azure Active Directory 中的稽核記錄會提供符合規範的系統活動記錄。  
所有稽核資料的第一個進入點是 [Azure Active Directory] 的 [活動] 區段中的 [稽核記錄]。

![稽核記錄檔](./media/concept-audit-logs/61.png "稽核記錄檔")

稽核記錄的預設清單檢視顯示︰

- 發生時間與日期
- 活動的啟動器/動作項目 (對象) 
- 活動 (項目) 
- 目標

![稽核記錄檔](./media/concept-audit-logs/18.png "稽核記錄檔")

您可以按一下工具列中的 [資料行] 來自訂清單檢視。

![稽核記錄檔](./media/concept-audit-logs/19.png "稽核記錄檔")

這可讓您顯示其他欄位，或移除已顯示的欄位。

![稽核記錄檔](./media/concept-audit-logs/21.png "稽核記錄檔")


按一下清單檢視中的項目，即可取得所有可用的詳細資料。

![稽核記錄檔](./media/concept-audit-logs/22.png "稽核記錄檔")


## <a name="filtering-audit-logs"></a>篩選稽核記錄檔

若要將報告的資料縮小至您適用的層級，您可以使用下列欄位篩選稽核資料︰

- 日期範圍
- 啟動者 (執行者)
- 類別
- 活動資源類型
- 活動

![稽核記錄檔](./media/concept-audit-logs/23.png "稽核記錄檔")


**日期範圍**篩選條件可讓您定義傳回資料的時間範圍。  
可能的值包括：

- 1 個月
- 7 天
- 24 小時
- 自訂

當您選取自訂時間範圍時，可以設定開始時間和結束時間。

**啟動者**篩選條件可讓您定義執行者的名稱或其萬用主體名稱 (UPN)。

**類別**篩選條件可讓您選取下列其中一個篩選條件︰

- 全部
- 核心類別
- 核心目錄
- 自助式密碼管理
- 自助式群組管理
- 帳戶佈建 - 自動密碼變換
- 受邀的使用者
- MIM 服務
- 身分識別保護
- B2C

**活動資源類型**篩選條件可讓您選取下列其中一個篩選條件︰

- 全部 
- 群組
- 目錄
- 使用者
- Application
- 原則
- 裝置
- 其他

當您選取 [群組] 作為 [活動資源類型] 時，會取得可讓您也提供**來源**的額外篩選條件類別：

- Azure AD
- O365


**活動**篩選條件是根據您類別和您選取的活動資源類型。 您可以選取您想要查看的特定活動或選擇全部。 

您可以使用圖形 API https://graph.windows.net/$tenantdomain/activities/auditActivityTypes?api-version=beta 來取得所有稽核活動的清單 (其中，$tenantdomain 是網域名稱)，或請參閱[稽核報告事件](concept-audit-logs.md)一文。


## <a name="audit-logs-shortcuts"></a>稽核記錄快速鍵

除了 **Azure Active Directory** 之外，Azure 入口網站可提供您稽核資料的兩個額外進入點︰

- 使用者和群組
- 企業應用程式

### <a name="users-and-groups-audit-logs"></a>使用者和群組稽核記錄檔

透過以使用者和群組為基礎的稽核報告，可以取得下列問題的解答︰

- 使用者已套用哪些類型的更新？

- 有多少使用者已變更？

- 有多少密碼已變更？

- 系統管理員已在目錄中執行哪些作業？

- 已新增的群組為何？

- 群組有成員資格變更嗎？

- 群組的擁有者已變更嗎？

- 指派給群組或使用者的授權為何？

如果您只想檢閱使用者和群組相關的稽核資料，您可以在 [使用者和群組] 的 [活動] 區段中的 [稽核記錄] 之下找到篩選過的檢視。 此進入點將 [使用者和群組] 作為預先選取的 [活動資源類型]。

![稽核記錄檔](./media/concept-audit-logs/93.png "稽核記錄檔")

### <a name="enterprise-applications-audit-logs"></a>企業應用程式稽核記錄

透過以應用程式為基礎的稽核報告，可以取得下列問題的解答︰

* 已新增或更新的應用程式為何？
* 已移除的應用程式為何？
* 應用程式的服務原則已變更嗎？
* 應用程式的名稱已變更嗎？
* 誰已同意應用程式？

如果您只想檢閱應用程式相關的稽核資料，您可以在 [企業應用程式] 刀鋒視窗的 [活動] 區段中的 [稽核記錄] 之下找到篩選過的檢視。 此進入點將 [企業應用程式] 當作預先選取的 [活動資源類型]。

![稽核記錄檔](./media/concept-audit-logs/134.png "稽核記錄檔")

您可以將此檢視進一步向下篩選為只有 [群組] 或只有 [使用者]。

![稽核記錄檔](./media/concept-audit-logs/25.png "稽核記錄檔")



## <a name="next-steps"></a>後續步驟

- 如需報告概觀，請參閱 [Azure Active Directory 報告](overview-reports.md)。

- 如需完整的稽核活動清單，請參閱 [Azure AD 稽核活動參考](reference-audit-activities.md)

