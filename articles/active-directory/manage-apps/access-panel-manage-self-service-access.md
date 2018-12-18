---
title: 如何使用自助應用程式存取 | Microsoft Docs
description: 啟用自助應用程式存取以允許使用者尋找自己的應用程式
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
ms.openlocfilehash: 4936c6a0c7323ff5b607519c6d86c2428d7003bb
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355014"
---
# <a name="how-to-use-self-service-application-access"></a>如何使用自助應用程式存取

對於您想要允許使用者自行探索和要求存取的任何應用程式，您必須啟用**自助應用程式存取**，使用者才能從存取面板自行探索應用程式。

此功能極有助於節省 IT 群組的時間和成本，非常建議在搭配 Azure Active Directory 進行現代化應用程式部署時使用。

您可以使用這項功能︰

-   讓使用者不需要 IT 群組的協助，即可從[應用程式存取面板](https://myapps.microsoft.com/)自行探索應用程式。

-   將那些使用者新增至預先設定的群組，以便您可以查看哪些人要求存取權、移除存取權，以及管理指派給他們的角色。

-   可選擇讓商務核准者核准應用程式存取要求，以分擔 IT 群組的工作。

-   可選擇設定最多 10 人可核准對此應用程式的存取。

-   可選擇讓商務核准者直接從商務核准者的[應用程式存取面板](https://myapps.microsoft.com/)設定密碼，供那些使用者用來登入應用程式。

-   可選擇自動將自助式指派的使用者直接指派至應用程式角色。

## <a name="enable-self-service-application-access-to-allow-users-to-find-their-own-applications"></a>啟用自助應用程式存取以允許使用者尋找自己的應用程式

自助應用程式存取是讓使用者自行探索應用程式的絕佳方式，還可讓商務群組核准對那些應用程式的存取。 您可以讓商務群組直接從其存取面板，管理指派給「密碼單一登入應用程式」使用者的認證。

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

   * 不支援群組。

13. **選擇性：****對於公開角色的應用程式，** 若要將已獲得自助存取核准的使用者指派給角色，請按一下 [在此應用程式中應為使用者指派的角色為?] 旁的選取控制項，以選取要為這些使用者指派的角色。

14. 按一下刀鋒視窗頂端的 [儲存] 按鈕以完成此動作。

完成自助應用程式設定之後，使用者可以瀏覽到其[應用程式存取面板](https://myapps.microsoft.com/)並按一下 [+新增] 按鈕以尋找您已啟用自助存取的應用程式。 商務核准者在其[應用程式存取面板](https://myapps.microsoft.com/)中也會看到通知。 您可以啟用電子郵件，通知他們有使用者已要求存取應用程式，需要他們核准。 

這些核准只支援單一核准工作流程，這表示若您指定多個核准者，任何核准者都可以核准應用程式存取。

## <a name="next-steps"></a>後續步驟
[設定 Azure Active Directory 進行自助服務群組管理](../users-groups-roles/groups-self-service-management.md)
