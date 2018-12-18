---
title: Azure Active Directory 中的大型組織使用者、群組、授權和角色 | Microsoft Docs
description: Azure Active Directory 中使用者與指派的授權、系統管理員角色、群組成員資格之間的關聯性
keywords: ''
author: curtand
manager: mtillman
ms.author: curtand
ms.reviewer: vincesm
ms.date: 10/01/2018
ms.topic: overview
ms.service: active-directory
ms.workload: identity
services: active-directory
ms.custom: it-pro
ms.openlocfilehash: 7f588766ead19f2cbca22aab596e6dc9d0210791
ms.sourcegitcommit: 7bc4a872c170e3416052c87287391bc7adbf84ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48019911"
---
# <a name="users-groups-licensing-and-roles-for-large-organizations"></a>大型組織的使用者、群組、授權和角色

本文將在重要的使用者[身分識別管理](/azure/active-directory/fundamentals/identity-fundamentals?context=azure/active-directory/users-groups-roles/context/ugr-context)工作關係中 (就使用者的群組、授權、已部署企業應用程式及系統管理員角色而言)，引入 Azure AD 系統管理員。 隨著貴組織的成長，您可以使用 Azure AD 群組和系統管理員角色來：

* 將授權指派給群組而非個人
* 委派權限以將 Azure AD 管理工作分配給較低權限的角色
* 將企業應用程式存取權指派給群組

## <a name="assign-users-to-groups"></a>將使用者指派給群組

您可以在 Azure AD 中使用群組，將授權指派給大量使用者，或者將使用者存取權指派給已部署的企業應用程式。 您可以在 Azure AD 中使用群組指派系統管理員角色，或者您可以將存取權授與外部資源，例如 SaaS 應用程式或 SharePoint 網站。

如需額外彈性及減少管理群組成員資格的工作，您可以使用 Azure AD 中的[動態群組](groups-create-rule.md)來自動擴充及縮減群組成員資格。 屬於一或多個動態群組成員的每個唯一使用者都需要 Azure AD Premium P1 授權。

## <a name="assign-licenses-to-groups"></a>將授權指派給群組

個別指派或移除使用者的授權需要時間和心力。 相反地，如果您[將授權指派給群組](/azure/active-directory/fundamentals/license-users-groups?context=azure/active-directory/users-groups-roles/context/ugr-context)，可以讓大規模授權管理更容易。

在 Azure AD 中，當使用者加入授權的群組時，他們會自動獲派適當的授權。 當使用者離開群組時，Azure AD 會移除其授權指派。 如果沒有 Azure AD 群組，您就必須撰寫 PowerShell 指令碼或使用圖形 API，針對加入或離開組織的使用者大量新增或移除使用者授權。

如果沒有足夠的可用授權，或發生像是無法同時指派服務方案的問題，您可以在 Azure 入口網站中看到任何群組授權問題的狀態。

>[!NOTE]
>群組型授權功能目前為公開預覽狀態。 在預覽期間，此功能可用於任何付費 Azure Active Directory (Azure AD) 授權方案或試用版。

## <a name="delegate-administrator-roles"></a>委派系統管理員角色

許多大型組織希望可以選擇讓其使用者取得適用於工作的足夠權限，而不是指派強大的全域管理員角色給他們，例如，必須註冊應用程式的使用者。 以下是新 Azure AD 系統管理員角色的範例，可協助您更細微地分配應用程式管理工作：

 角色名稱 | 權限摘要
 --------- | -------------------
 **應用程式系統管理員** | 可以新增和管理企業應用程式和應用程式註冊，並設定 Proxy 應用程式設定。 應用程式系統管理員可以檢視條件式存取原則和裝置，但是不能管理它們。
 **雲端應用程式系統管理員** | 可以新增和管理企業應用程式和企業應用程式註冊。 除了無法管理應用程式 Proxy 設定以外，這個角色具有應用程式系統管理員的所有權限。
**應用程式開發人員** | 可以新增和更新應用程式註冊，但是無法管理企業應用程式或設定應用程式 Proxy。

我們已新增新的 Azure AD 系統管理員角色。 請查看 Azure 入口網站或[系統管理員角色權限參考](directory-assign-admin-roles.md)以了解目前可用的角色。

## <a name="assign-app-access"></a>指派應用程式存取權

您可以使用 Azure AD 將群組存取權指派給[部署在 Azure AD 租用戶中的企業應用程式](/azure/active-directory/manage-apps/methods-for-assigning-users-and-groups#assign-a-group-directly-to-an-application-as-an-administrator?context=azure/active-directory/users-groups-roles/context/ugr-context)。 如果您將具有群組指派的動態群組合併至應用程式，您就可以在組織成長時，自動化使用者應用程式的存取權指派。 您需要 Azure Active Directory Premium P1 或 Premium P2 授權，才能將存取權指派給企業應用程式。

Azure AD 也可讓您更精細地控制在應用程式與群組 (您對其指派存取權) 之間流動的資料。 在[企業應用程式](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)中，開啟應用程式，然後選取 [佈建] 來：

* 為應用程式設定自動佈建 (若有支援)
* 提供認證以連線至應用程式的使用者管理 API
* 設定對應，控制在使用者帳戶佈建或更新時，Azure AD 與應用程式之間有哪些使用者屬性流動
* 針對應用程式啟動和停止 Azure AD 佈建服務、清除佈建快取，或重新啟動服務
* 檢視**佈建活動報告**，該報告提供 Azure AD 與應用程式之間已建立、更新及移除的所有使用者和群組記錄，以及檢視**佈建錯誤報告**，該報告提供更詳細的錯誤訊息

## <a name="next-steps"></a>後續步驟

如果您剛開始成為 Azure AD 系統管理員，請在 [Azure Active Directory 基礎](https://docs.microsoft.com/azure/active-directory/fundamentals/index)中取得基本概念。

或者您可以開始[建立群組](/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal?context=azure/active-directory/users-groups-roles/context/ugr-context)、[指派授權](/azure/active-directory/fundamentals/license-users-groups?context=azure/active-directory/users-groups-roles/context/ugr-context)、[指派應用程式存取權](/azure/active-directory/manage-apps/methods-for-assigning-users-and-groups#assign-a-group-directly-to-an-application-as-an-administrator?context=azure/active-directory/users-groups-roles/context/ugr-context)或[指派系統管理員角色](directory-assign-admin-roles.md)。