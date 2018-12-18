---
title: 在 Azure Active Directory 中啟用 LinkedIn 連線整合 | Microsoft Docs
description: 說明如何在 Azure Active Directory 中為 Microsoft 應用程式啟用或停用 LinkedIn 帳戶連線
services: active-directory
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 09/11/2018
ms.author: curtand
ms.reviewer: beengen
ms.custom: it-pro
ms.openlocfilehash: 0eaa2656313ecd9b64503051265dc16285f0a1f3
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44492816"
---
# <a name="linkedin-account-connections"></a>LinkedIn 帳戶連絡人

在本文中，您可以了解如何在 Azure Active Directory (Azure AD) 系統管理中心啟用或停用您租用戶的 LinkedIn 帳戶連線。

> [!IMPORTANT]
> LinkedIn 帳戶連線設定即將推出，以供 Azure AD 租用戶使用。 當它推出以供您的租用戶使用時，預設會加以啟用。 
> 
> 例外狀況：
> * 此設定不適用於使用 Microsoft Cloud for US Government、Microsoft Cloud Germany 或中國 21Vianet 管理之 Azure 和 Office 365 的客戶。
> * 對於佈建於德國的租用戶，此設定預設為關閉。 請注意，此設定不適用於使用 Microsoft Cloud Germany 的客戶。
> * 對於佈建於法國的租用戶，此設定預設為關閉。

> 只有在啟用整合*且*如果您允許使用者同意應用程式存取公司資料時，整合才會生效。 如需同意設定的相關資訊，請參閱[如何移除使用者的應用程式存取](https://docs.microsoft.com/azure/active-directory/application-access-assignment-how-to-remove-assignment)。

## <a name="enable-or-disable-linkedin-account-connections-for-your-tenant-in-the-azure-portal"></a>在 Azure 入口網站中為租用戶啟用或停用 LinkedIn 帳戶連線

您可以以啟用或停用整個租用戶的 LinkedIn 帳戶連線，或是僅針對您租用戶中選取的使用者。

1. 使用具有 Azure AD 租用戶全域管理員身分的帳戶來登入 [Azure Active Directory 系統管理中心](https://aad.portal.azure.com/)。
2. 選取 [使用者]。
3. 在 [使用者] 刀鋒視窗上，選取 [使用者設定]。
4. 在 [LinkedIn 帳戶連線] 之下：
  * 選取 [是] 可啟用您租用戶中所有使用者的 LinkedIn 帳戶連線
  * 選取 [已選取] 只啟用所選租用戶使用者的 LinkedIn 帳戶連線
  * 選取 [否] 可停用所有使用者的 LinkedIn 帳戶連線 ![啟用 LinkedIn 帳戶連線](./media/linkedin-integration/linkedin-integration.png)
5. 當您完成時，選取 [儲存] 會儲存您的設定。

## <a name="enable-or-disable-linkedin-account-connections-for-your-tenant-using-group-policy"></a>使用群組原則，啟用或停用租用戶的 LinkedIn 帳戶連線

1. 下載 [Office 2016 系統管理範本檔案 (ADMX/ADML)](https://www.microsoft.com/download/details.aspx?id=49030)
2. 解壓縮 **ADMX** 檔案，並將其複製到中央存放區。
3. 開啟群組原則管理。
4. 使用下列設定建立群組原則物件：[使用者設定] > [系統管理範本] > [Microsoft Office 2016] > [其他] > [在 Office 應用程式中顯示 LinkedIn 功能]。
5. 選取 [啟用] 或 [停用]。
  
 State | 效果
------ | ------
**已啟用** | Office 2016 選項中的 [在 Office 應用程式中顯示 LinkedIn 功能] 設定已啟用。 組織中的使用者可在其 Office 應用程式中使用 LinkedIn 功能。
 **已停用** | Office 2016 選項中的 [在 Office 應用程式中顯示 LinkedIn 功能] 設定已停用，而且終端使用者無法變更此設定。 組織中的使用者無法在其 Office 2016 應用程式中使用 LinkedIn 功能。

這個群組原則只會影響本機電腦的 Office 2016 應用程式。 即使使用者在其 Office 2016 應用程式中停用 LinkedIn，他們仍會在 Office 365 的設定檔卡中看到 LinkedIn 功能。

## <a name="learn-more"></a>深入了解

* [在您的組織中整合 LinkedIn](linkedin-user-consent.md)

* [Microsoft 應用程式中的 LinkedIn 資訊和功能](https://go.microsoft.com/fwlink/?linkid=850740)

* [LinkedIn 說明中心](https://www.linkedin.com/help/linkedin)

## <a name="next-steps"></a>後續步驟
使用下列連結，在 Azure 入口網站中查看您目前的 LinkedIn 帳戶連線設定：

[設定 LinkedIn 帳戶連線](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/UserSettings) 