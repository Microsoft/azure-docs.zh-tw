---
title: 管理 Azure Active Directory 租用戶 | Microsoft Docs
description: 了解什麼是 Azure AD 租用戶，以及如何使用 Azure Active Directory 管理 Azure。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: lizross
ms.reviewer: jeffsta
custom: it-pro
ms.openlocfilehash: eb9cf7c21613ab91cc0b8e41a4454835d18f2c34
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297923"
---
# <a name="manage-your-azure-active-directory-tenant"></a>管理 Azure Active Directory 租用戶

## <a name="what-is-an-azure-ad-tenant"></a>什麼是 Azure AD 租用戶？
在 Azure Active Directory (Azure AD) 中，租用戶是您組織在註冊 Microsoft 雲端服務 (例如 Azure 或 Office 365) 時所收到的專屬 Azure AD 目錄執行個體。 每個 Azure AD 目錄都不同，並與其他 Azure AD 目錄分開。 就像公司辦公大樓是您組織特有的安全資產，Azure AD 目錄也是設計成僅供您組織使用的安全資產。 Azure AD 架構會隔離客戶資料和身分識別資訊，某個 Azure AD 目錄的使用者和系統管理員便無法意外或惡意存取另一個目錄中的資料。

![管理 Azure Active Directory](./media/active-directory-administer/aad_portals.png)

## <a name="how-can-i-get-an-azure-ad-directory"></a>如何取得 Azure AD 目錄？
Azure AD 提供大部分 Microsoft 雲端服務的核心目錄和身分識別管理功能，包括：

* Azure
* Microsoft Office 365
* Microsoft Dynamics CRM Online
* Microsoft Intune

註冊上述任何 Microsoft 雲端服務時，會收到 Azure AD 目錄。 您可以視需要建立額外的目錄。 例如，您可能會將第一個目錄維護為生產目錄，然後建立另一個目錄來進行測試或預備。

### <a name="using-the-azure-ad-directory-that-comes-with-a-new-azure-subscription"></a>使用新 Azure 訂用帳戶隨附的 Azure AD 目錄

建議您在註冊其他 Microsoft 服務時使用您用於第一項服務的系統管理員帳戶。 您第一次註冊 Microsoft 服務時提供的資訊於為您的組織建立新的 Azure AD 目錄執行個體。 如果您在訂閱其他 Microsoft 服務時使用該目錄來驗證登入嘗試，這些服務可以使用您在預設目錄中設定的現有使用者帳戶、原則、設定或內部部署目錄整合。

例如，如果您註冊 Microsoft Intune 訂用帳戶，然後進一步同步處理內部部署 Active Directory 與 Azure AD 目錄，則可以註冊另一項 Microsoft 服務 (例如 Office 365) 並輕鬆達成與 Microsoft Intune 相同的目錄整合優勢。

如需整合內部部署目錄與 Azure AD 的詳細資訊，請參閱[透過 Azure AD Connect 進行目錄整合](../hybrid/whatis-hybrid-identity.md)。

### <a name="associate-an-existing-azure-ad-directory-with-a-new-azure-subscription"></a>關聯現有的 Azure AD 目錄與新的 Azure 訂用帳戶
您可以關聯新的 Azure 訂用帳戶與驗證現有 Office 365 或 Microsoft Intune 訂用帳戶登入的相同目錄。 如需該案例的詳細資訊，請參閱[將 Azure 訂用帳戶的擁有權轉移給另一個帳戶](../../billing/billing-subscription-transfer.md)

### <a name="create-an-azure-ad-directory-by-signing-up-for-a-microsoft-cloud-service-as-an-organization"></a>以組織身分註冊 Microsoft 雲端服務來建立 Azure AD 目錄
如果您還沒有 Microsoft 雲端服務的訂用帳戶，您可使用下列其中一個連結進行註冊。 註冊第一項服務可自動建立 Azure AD 目錄。

* [Microsoft Azure](https://account.azure.com/organization)
* [Office 365](http://products.office.com/business/compare-office-365-for-business-plans/)
* [Microsoft Intune](https://portal.office.com/Signup/Signup.aspx?OfferId=40BE278A-DFD1-470a-9EF7-9F2596EA7FF9&dl=INTUNE_A&ali=1#0%20)

### <a name="how-to-change-the-default-directory-for-a-subscription"></a>如何變更訂用帳戶的預設目錄

1. 以訂用帳戶之帳戶管理員的帳戶登入 [Azure 帳戶中心](https://account.azure.com/Subscriptions)，以便移轉訂用帳戶擁有權。
2. 確定您希望成為訂用帳戶擁有者的使用者位於目標目錄中。
3. 按一下 [移轉訂用帳戶]。
4. 指定接受者。 接受者會自動收到含有接受連結的電子郵件。
5. 接受者按一下連結並遵循指示進行，包括輸入他們的付款資訊。 當接受者成功時，訂用帳戶就轉移完成。 
6. 若訂用帳戶擁有權移轉成功，則訂用帳戶的預設目錄會變更為使用者所在的目錄。

若要深入了解，請參閱[將 Azure 訂用帳戶擁有權轉移給另一個帳戶](../../billing/billing-subscription-transfer.md)

### <a name="manage-the-default-directory-in-azure"></a>管理 Azure 中的預設目錄
當您註冊 Azure 時，預設 Azure AD 目錄會與您的訂用帳戶相關聯。 使用 Azure AD 不需要成本，而您的目錄為免費資源。 付費 Azure AD 服務會分開授權並提供其他功能 (例如登入時的公司商標和自助式密碼重設)。 您也可以使用您擁有的 DNS 名稱建立自訂網域，而非使用預設 *.onmicrosoft.com 網域。

## <a name="how-can-i-manage-directory-data"></a>如何管理目錄資料？
若要管理一或多個 Microsoft 雲端服務訂用帳戶，您可以使用 [Azure AD 系統管理中心](https://aad.portal.azure.com)、Microsoft Intune 帳戶入口網站或 [Office 365 系統管理中心](https://portal.office.com/)來管理組織的目錄資料。 您也可以使用 [Azure Active Directory PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/active-directory)，協助您管理 Azure AD 中所儲存的資料。

從上述其中一個入口網站 (或 Cmdlet)，您可以：

* 建立和管理使用者和群組帳戶
* 管理您組織之訂用帳戶的相關雲端服務
* 設定與 Azure AD 身分識別和驗證服務的內部部署整合

Azure AD 系統管理中心、Office 365 系統管理中心、Microsoft Intune 帳戶入口網站和 Azure AD Cmdlet 都會讀取和寫入與您組織的目錄相關聯的單一 Azure AD 共用執行個體。 上述每個工具可做為提取或變更目錄資料的前端介面。
如果利用其中一個服務身分登入時使用任何入口網站或 Cmdlet 來變更組織的資料，則變更也會在您下次登入時顯示在其他入口網站中。 這項資料會在您所訂閱的 Microsoft 雲端服務之間共用。

例如，如果您使用 Office 365 系統管理中心封鎖使用者登入，該動作會封鎖使用者登入您組織目前所訂閱的任何其他服務。 如果您在 Microsoft Intune 帳戶入口網站檢視相同的使用者帳戶，您也會看到該使用者遭到封鎖。

## <a name="how-can-i-add-and-manage-multiple-directories"></a>如何新增和管理多個目錄？
您可以在 [Azure 入口網站中新增 Azure AD 目錄](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory)。 填妥資訊並選取 [建立]。

您可以管理每個目錄當做完全獨立的資源：每個目錄都是對等、全功能且邏輯上與您所管理的其他目錄無關；目錄之間沒有任何父子關聯性。 目錄之間的這項獨立性包括資源獨立性、系統管理獨立性和同步處理獨立性。

* **資源獨立性**。 如果您在某個目錄中建立或刪除資源，則不會影響另一個目錄中的任何資源 (但外部使用者有部分例外狀況)。 如果您搭配使用自訂網域 'contoso.com' 與某個目錄，則它不能與任何其他目錄搭配使用。
* **系統管理獨立性**。  如果不是 'Contoso' 目錄之系統管理員的使用者建立 'Test' 測試目錄，則：
  
  * 'Contoso' 目錄的系統管理員沒有 'Test' 目錄的直接系統管理權限，除非 'Test' 的系統管理員特別將這些權限授與他們。 'Contoso' 的系統管理員憑藉著其對已建立 'Test' 之使用者帳戶的控制，可以控制 'Test' 目錄的存取權。
    
  * 如果您指派或移除某個目錄中使用者的系統管理員角色，則變更不會影響另一個目錄中該使用者可能有的任何系統管理員角色。
* **同步處理獨立性**。 您可以單獨設定每個 Azure AD 租用戶，以從 Azure AD Connect 目錄同步工具的單一執行個體同步處理資料。

與其他 Azure 資源不同，您的目錄不是 Azure 訂用帳戶的子資源。 因此，如果您取消或允許 Azure 訂用帳戶到期，則還是可以使用 Azure AD PowerShell、Azure Graph API 或其他介面 (例如 Office 365 系統管理中心) 存取目錄資料。 您也可以關聯另一個訂用帳戶與目錄。

## <a name="how-to-prepare-to-delete-an-azure-ad-directory"></a>如何準備刪除 Azure AD 目錄
全域管理員可以從入口網站刪除 Azure AD 目錄。 刪除目錄時，也會一併刪除目錄中內含的所有資源。 確認您不需要該目錄之後，再予以刪除。

> [!NOTE]
> 如果使用者以工作或學校帳戶登入，則使用者不得嘗試刪除其主目錄。 例如，如果使用者以 joe@contoso.onmicrosoft.com 身分登入，則該使用者無法刪除預設網域為 contoso.onmicrosoft.com 的目錄。

Azure AD 需要符合特定條件才能刪除目錄。 這可降低刪除目錄對使用者或應用程式造成負面影響的風險 (例如使用者登入 Office 365 或存取 Azure 中資源的能力)。 例如，如果不小心刪除訂用帳戶的目錄，則使用者無法存取該訂用帳戶的 Azure 資源。

會檢查下列條件：

* 目錄中的唯一使用者應該是將要刪除目錄的全域管理員。 必須先刪除任何其他使用者，才能刪除目錄。 如果使用者是從內部部署進行同步處理，則必須關閉同步處理，而且必須使用 Azure 入口網站或 Azure PowerShell Cmdlet 來刪除雲端目錄中的使用者。 不需要刪除群組或連絡人 (例如從 Office 365 系統管理中心新增的連絡人)。
* 目錄中可能沒有任何應用程式。 必須先刪除任何應用程式，才能刪除目錄。
* 任何 Multi-Factor Authentication 提供者都無法連線至目錄。
* 任何 Microsoft Online Service 都沒有任何訂用帳戶 (例如與目錄相關聯 Microsoft Azure、Office 365 或 Azure AD Premium)。 例如，在 Azure 中建立預設目錄時，如果您的 Azure 訂用帳戶仍然依賴此目錄來進行驗證，則無法刪除此目錄。 同樣地，如果另一位使用者擁有與目錄相關聯的訂用帳戶，您無法刪除目錄。 


## <a name="next-steps"></a>後續步驟
* [Azure AD 論壇](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=windowsazuread)
* [Azure Multi-Factor Authentication 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=windowsazureactiveauthentication)
* [StackOverflow for Azure 問題](https://stackoverflow.com/questions/tagged/azure-active-directory)
* [Azure Active Directory PowerShell](https://docs.microsoft.com/powershell/azure/active-directory)
* [在 Azure AD 中指派系統管理員角色](../users-groups-roles/directory-assign-admin-roles.md)
