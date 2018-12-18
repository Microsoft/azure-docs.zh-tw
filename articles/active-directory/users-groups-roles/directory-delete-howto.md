---
title: 刪除 Azure Active Directory 租用戶目錄 | Microsoft Docs
description: 說明如何讓 Azure AD 租用戶目錄準備好刪除
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 06/13/2018
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.openlocfilehash: b1d3439412e324c71687c43aa9e47c520cb72262
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141305"
---
# <a name="delete-an-azure-active-directory-tenant"></a>刪除 Azure Active Directory 租用戶
刪除租用戶時，也會一併刪除租用戶中包含的所有資源。 在刪除之前，您必須盡可能縮減租用戶的相關資源，使其就緒。 只有 Azure Active Directory (Azure AD) 全域管理員可從入口網站刪除 Azure AD 租用戶。

## <a name="prepare-the-tenant-for-deletion"></a>讓租用戶準備好刪除

在 Azure AD 中的租用戶通過幾項檢查之前，您無法予以刪除。 這些檢查可降低刪除租用戶對使用者存取產生負面影響的風險，例如無法登入 Office 365 或存取 Azure 中資源。 例如，如果不小心刪除與訂用帳戶建立關聯的租用戶，使用者就無法存取該訂用帳戶的 Azure 資源。 以下說明檢查的條件：

* 除了一個要刪除租用戶的全域管理員以外，租用戶中不能有任何使用者。 其他任何使用者都必須先刪除，才能刪除目錄。 如果使用者會從內部部署同步，則必須關閉同步，而且必須使用 Azure 入口網站或 Azure PowerShell Cmdlet 來刪除雲端租用戶中的使用者。 
* 租用戶中不能有任何應用程式。 任何應用程式都必須先移除，才能刪除租用戶。
* 不能有任何多重要素驗證提供者連結到租用戶。
* 不能有任何 Microsoft Online Services (Microsoft Azure、Office 365 或 Azure AD Premium) 的訂用帳戶與租用戶建立關聯。 例如，在 Azure 中建立預設租用戶時，如果您的 Azure 訂用帳戶仍然仰賴這個租用戶進行驗證，就無法刪除此租用戶。 同樣地，如果另一位使用者讓訂用帳戶與租用戶建立關聯，您也會無法予以刪除。 

## <a name="delete-an-azure-ad-tenant"></a>刪除 Azure AD 租用戶

1. 使用身分為租用戶全域管理員的帳戶登入 [Azure AD 系統管理中心](https://aad.portal.azure.com)。

2. 選取 **Azure Active Directory**。

3. 切換到您想要刪除的租用戶。
  
  ![[刪除目錄] 按鈕](./media/directory-delete-howto/delete-directory-command.png)

4. 選取 [刪除目錄]。
  
  ![[刪除目錄] 按鈕](./media/directory-delete-howto/delete-directory-list.png)

5. 若您的租用戶未通過一或多項檢查，系統會提供您一個連結，其中有如何通過的詳細資訊。 在您通過所有檢查之後，選取 [刪除] 即可完成流程。

## <a name="i-have-an-expired-subscription-but-i-cant-delete-the-tenant"></a>我有已過期的訂用帳戶，但無法刪除租用戶

在您當初設定 Azure Active Directory 租用戶時，可能也為組織啟用了以授權為基礎的訂用帳戶，像是 Azure Active Directory Premium P2、Office 365 商務進階版或企業行動力 + 安全性 E5。 這些訂用帳戶在完全刪除之前，會使目錄無法刪除，以避免意外遺失資料。 訂用帳戶的狀態必須為**取消佈建**，租用戶才能刪除。 **過期**或**取消**的訂用帳戶會進入**停用**狀態，而最終階段為**取消佈建**狀態。 

至於試用 Office 365 訂用帳戶過期時會發生的情況 (不含付費合作夥伴/CSP、Enterprise 合約或大量授權)，請見下表。 如需 Office 365 資料保留期與訂用帳戶生命週期的詳細資訊，請參閱 [What happens to my data and access when my Office 365 for business subscription ends?](https://support.office.com/article/what-happens-to-my-data-and-access-when-my-office-365-for-business-subscription-ends-4436582f-211a-45ec-b72e-33647f97d8a3) (當商務用 Office 365 訂閱結束時，我的資料和存取權會如何？)。 

訂用帳戶狀態 | 資料 | 資料存取權
----- | ----- | -----
有效 (試用 30 天)  | 資料可供全體存取    | <li>使用者可以正常存取 Office 365 檔案或應用程式<li>管理員可以正常存取 Office 365 系統管理中心及資源 
過期 (30 天)   | 資料可供全體存取    | <li>使用者可以正常存取 Office 365 檔案或應用程式<li>管理員可以正常存取 Office 365 系統管理中心及資源
停用 (30 天) | 資料僅供管理員存取  | <li>使用者無法存取 Office 365 檔案或應用程式<li>管理員可以存取 Office 365 系統管理中心，但無法指派授權或更新使用者
取消佈建 (「停用」後 30 天) | 資料已刪除 (若沒有其他使用中的服務，就會自動刪除) | <li>使用者無法存取 Office 365 檔案或應用程式<li>管理員可以存取 Office 365 系統管理中心以購買及管理其他訂用帳戶 

您可以使用商務用 Microsoft Store 系統管理中心讓訂用帳戶進入**取消佈建**狀態，以在 3 天內刪除。 這項功能即將在 Office 365 系統管理中心推出。

1. 使用在租用戶中身分為全域管理員的帳戶登入[商務用 Microsoft Store 系統管理中心](https://businessstore.microsoft.com/manage/)。 若您嘗試刪除初始預設網域為 contoso.onmicrosoft.com 的 “Contoso” 租用戶，請使用 admin@contoso.onmicrosoft.com這樣的 UPN 登入。

2. 前往 [管理] 索引標籤，選取 [產品與服務]，然後選擇您要取消的訂用帳戶。 在按一下 [取消] 後，重新整理頁面。
  
  ![用於刪除訂用帳戶的 [刪除] 連結](./media/directory-delete-howto/delete-command.png)
  
3. 選取 [刪除] 以刪除訂用帳戶並接受條款及條件。 所有資料都會在三天內永久刪除。 若您改變了心意，在這三天內都可以重新啟用訂用帳戶。
  
  ![條款及條件](./media/directory-delete-howto/delete-terms.png)

4. 訂用帳戶狀態現已變更，而訂用帳戶會標示為待刪除。 訂用帳戶會在 72 小時候進入**取消佈建**狀態。

5. 在您刪除了租用戶上的訂用帳戶，並經過 72 小時之後，您就可以再次登入 Azure AD 系統管理中心，這時理應沒有要採取的動作，也不會有訂用帳戶使您無法刪除租用戶。 您應能夠成功刪除 Azure AD 租用戶。
  
  ![刪除畫面上的通過訂用帳戶檢查](./media/directory-delete-howto/delete-checks-passed.png)

## <a name="next-steps"></a>後續步驟
[Azure Active Directory 文件](https://docs.microsoft.com/azure/active-directory/)
