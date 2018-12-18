---
title: 如何使用 Azure Active Directory 建立基本群組並新增成員 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 建立基本群組。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: quickstart
ms.date: 08/22/2018
ms.author: lizross
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 36bd0ca06859c1952a75b1aa8397fba88dc22d40
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574907"
---
# <a name="how-to-create-a-basic-group-and-add-members-using-azure-active-directory"></a>如何：使用 Azure Active Directory 建立基本群組並新增成員

您可以使用 Azure Active Directory (Azure AD) 入口網站建立基本群組。 基於此文章的目的，一個基本的群組會由資源擁有者 (系統管理員) 新增到單一資源，並包含需要存取該資源的特定成員 (員工)。 如需更複雜的案例，包括動態成員資格和規則建立，請參閱[Azure Active Directory 使用者管理文件](../users-groups-roles/index.yml)。

## <a name="create-a-basic-group-and-add-members"></a>建立基本群組並新增成員
您可以同時建立基本群組並新增成員。

### <a name="to-create-a-basic-group-and-add-members"></a>建立基本群組並新增成員
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。

2. 選取 [Azure Active Directory]、[群組]，然後選取 [新增群組]。

    ![顯示 [群組] 的 Azure AD](media/active-directory-groups-create-azure-portal/group-full-screen.png)

3. 在 [群組] 頁面中，填妥必要資訊。

    ![已填入範例資訊的 [新增群組] 頁面](media/active-directory-groups-create-azure-portal/new-group-blade.png)

    - **群組類型 (必要)**： 選取預先定義的群組類型。 其中包括：
        
        - **安全性**： 用來管理成員和電腦對使用者群組所共用之資源的存取權。 例如，您可以針對特定安全性原則建立安全性群組。 透過這麼做，您可以將一組權限同時授與所有成員，而不必個別為每個成員新增權限。 如需管理資源存取權的詳細資訊，請參閱[使用 Azure Active Directory 群組來管理資源的存取權](active-directory-manage-groups.md)。
        
        - **Office 365**： 透過將共用信箱、行事曆、檔案、SharePoint 網站等的存取權授與成員，來提供共同作業的機會。 此選項也可讓您將群組的存取權授與組織外的人員。 如需 Office 365 群組的詳細資訊，請參閱[了解 Office 365 群組](https://support.office.com/article/learn-about-office-365-groups-b565caa1-5c40-40ef-9915-60fdb2d97fa2) \(機器翻譯\)。

    - **群組名稱 (必要)**： 為群組新增名稱，請使用易記且合理的名稱。

    - **群組描述**： 為群組新增選擇性的描述。

    - **成員資格類型 (必要)**： 選取預先定義的成員資格類型。 其中包括：

        - **已指派**： 讓您將特定使用者新增為此群組的成員並具有唯一權限。 基於此文章的目的，我們會使用此選項。

        - **動態使用者**： 讓您使用動態群組規則來自動新增及移除成員。 如果成員的屬性出現變化，系統會查看您針對目錄所設定的動態群組規則，以判斷該成員是否仍符合規則需求 (並新增該成員)，或已不再符合規則需求 (並移除該成員)。

        - **動態裝置**： 讓您使用動態群組規則來自動新增及移除裝置。 如果裝置的屬性出現變化，系統會查看您針對目錄所設定的動態群組規則，以判斷該裝置是否仍符合規則需求 (並新增該裝置)，或已不再符合規則需求 (並移除該裝置)。

        >[!Important]
        >您可以針對裝置或使用者建立動態群組，但不能同時建立。 您也無法根據裝置擁有者的屬性來建立裝置群組。 裝置成員資格規則只能參考裝置屬性。 如需針對使用者及裝置建立動態群組的詳細資訊，請參閱[建立動態群組並檢查狀態](../users-groups-roles/groups-create-rule.md)。

4. 選取 [建立] 。

    您的群組已建立，並已可以新增成員。

5. 選取 [群組] 頁面中的 [成員] 區域，然後從 [選取成員] 頁面開始搜尋要新增至群組的成員。

    ![在群組建立程序期間選取群組成員](media/active-directory-groups-create-azure-portal/select-members-create-group.png)

6. 完成新增成員之後，請選擇 [選取]。

    [群組概觀] 頁面會更新，以顯示已新增至群組的成員數目。

    ![醒目提示成員數目的 [群組概觀] 頁面](media/active-directory-groups-create-azure-portal/group-overview-blade-number-highlight.png)

## <a name="next-steps"></a>後續步驟
在建立群組並具有至少一個使用者之後，您便可以：

- [檢視群組和成員](active-directory-groups-view-azure-portal.md)

- [管理群組成員資格](active-directory-groups-membership-azure-portal.md)

- [管理群組中使用者的動態規則](../users-groups-roles/groups-create-rule.md)

- [編輯群組設定](active-directory-groups-settings-azure-portal.md)

- [使用群組管理對資源的存取](active-directory-manage-groups.md)

- [使用群組管理對 SaaS 應用程式的存取](../users-groups-roles/groups-saasapps.md)

- [使用 PowerShell 命令管理群組](../users-groups-roles/groups-settings-v2-cmdlets.md)

- [將 Azure 訂用帳戶關聯或新增至 Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)