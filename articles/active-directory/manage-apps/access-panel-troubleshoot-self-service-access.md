---
title: 使用自助應用程式存取時遇到問題 | Microsoft Docs
description: 針對自助應用程式存取的相關問題進行疑難排解
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
ms.topic: article
ms.date: 07/11/2017
ms.author: barbkess
ms.reviewer: japere,asteen
ms.openlocfilehash: 3a8e0ca54766f36ba3343f420d751f2e40a8033b
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355038"
---
# <a name="problem-using-self-service-application-access"></a>使用自助應用程式存取時遇到問題

自助應用程式存取是讓使用者自行探索應用程式的絕佳方式，還可讓商務群組核准對那些應用程式的存取。 您可以讓商務群組直接從其存取面板，管理指派給「密碼單一登入應用程式」使用者的認證。

對於您想要允許使用者自行探索和要求存取的任何應用程式，您必須啟用**自助應用程式存取**，使用者才能從存取面板自行探索應用程式。

## <a name="general-issues-to-check-first"></a>首先檢查的一般問題

-   請確定已正確設定自助應用程式存取。 請參閱＜如何設定自助應用程式存取＞。

-   請確定已允許使用者或群組要求自助應用程式存取。

-   請確定使用者造訪正確的位置來要求自助應用程式存取。 使用者可以瀏覽至其[應用程式存取面板](https://myapps.microsoft.com/)，並按一下 [+新增] 按鈕，以尋找您已啟用自助存取的應用程式。

-   如果最近才剛設定自助應用程式存取，請在幾分鐘之後嘗試登入並再次登出使用者的存取面板，查明自助存取變更是否出現。

## <a name="how-to-configure-self-service-application-access"></a>如何設定自助應用程式存取

若要啟用對應用程式的自助存取，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  從 Azure Active Directory 左邊瀏覽功能表，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  從該清單選取您要啟用自助存取的應用程式。

7.  應用程式載入之後，按一下應用程式左邊瀏覽功能表中的 [自助]。

8.  若要啟用此應用程式的自助式應用程式存取，請將 [要允許使用者要求此應用程式的存取權嗎?] 切換開關切換為 [是]。

9.  接著，若要為要求存取此應用程式的使用者指派群組，請按一下 [要將指派的使用者新增至哪個群組呢?] 標籤旁的選取控制項，然後選取一個群組。

10. **選擇性：** 若要將使用者設定為必須經過商務核准才能存取應用程式，請將 [需要核准才能授予此應用程式的存取權嗎?] 切換開關設定為 [是]。

11. **選擇性：對於只使用密碼單一登入的應用程式，** 若要讓那些商務核准者為核准的使用者指定傳送給此應用程式的密碼，請將 [要允許核准者為此應用程式設定使用者的密碼嗎?] 切換開關設定為 [是]。

12. **選擇性：** 若要指定商務核准者以核准此應用程式存取權，請按一下 [哪些人員可核准此應用程式的存取權?] 標籤旁的選取控制項以選取最多 10 個商務核准者。

 >[!NOTE]
 > 不支援群組。
 >
 >

13. **選擇性：** **對於公開角色的應用程式**，若要將已獲得自助存取核准的使用者指派給角色，請按一下 [在此應用程式中應為使用者指派的角色為?] 旁的選取控制項，以選取要為這些使用者指派的角色。

14. 按一下刀鋒視窗頂端的 [儲存] 按鈕以完成此動作。

完成自助應用程式設定之後，使用者可以瀏覽到其[應用程式存取面板](https://myapps.microsoft.com/)並按一下 [+新增] 按鈕以尋找您已啟用自助存取的應用程式。 商務核准者在其[應用程式存取面板](https://myapps.microsoft.com/)中也會看到通知。 您可以啟用電子郵件，通知他們有使用者已要求存取應用程式，需要他們核准。 

這些核准只支援單一核准工作流程，這表示若您指定多個核准者，任何核准者都可以核准應用程式存取。

## <a name="if-these-troubleshooting-steps-do-not-resolve-the-issue"></a>如果這些疑難排解步驟無法解決問題 

使用下列資訊 (若有的話) 開啟支援票證︰

-   相互關聯錯誤 ID

-   UPN (使用者電子郵件地址)

-   TenantID

-   瀏覽器類型

-   錯誤發生期間的時區和時間/時間範圍

-   Fiddler 追蹤

## <a name="next-steps"></a>後續步驟
[設定 Azure Active Directory 進行自助服務群組管理](../users-groups-roles/groups-self-service-management.md)
