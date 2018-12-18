---
title: Azure Active Directory Domain Services：管理指南 | Microsoft Docs
description: 在 Azure AD 網域服務受控網域上建立組織單位 (OU)
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 52602ad8-2b93-4082-8487-427bdcfa8126
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: 31fe241a94cedb04e1f8c50819eef7ebf675d2fb
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39504833"
---
# <a name="create-an-organizational-unit-ou-on-an-azure-ad-domain-services-managed-domain"></a>在 Azure AD 網域服務的受控網域上建立組織單位 (OU)
Azure AD 網域服務的受控網域包含兩個內建的容器，分別稱為「AADDC 電腦」和「AADDC 使用者」。 「AADDC 電腦」容器有已加入受控網域中全部電腦的電腦物件。 「AADDC 使用者」容器包含 Azure AD 租用戶中的使用者和群組。 有時候，可能需要在受控網域上建立服務帳戶，才能部署工作負載。 為此目的，您可以在受控網域上建立自訂的組織單位 (OU)，並在此 OU 內建立服務帳戶。 本文將示範如何在受控網域中建立 OU。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. 已加入網域的虛擬機器，您可在其中管理 AD Domain Services 受控網域。 如果您沒有這類虛擬機器，請依照名為[將 Windows 虛擬機器加入受控網域](active-directory-ds-admin-guide-join-windows-vm.md)一文所述的所有工作進行操作。
5. 您需要目錄中**屬於「AAD DC 系統管理員」群組之使用者帳戶**的認證，才能在受控網域上建立自訂 OU。

## <a name="install-ad-administration-tools-on-a-domain-joined-virtual-machine-for-remote-administration"></a>在已加入網域的虛擬機器上安裝 AD 系統管理工具進行遠端管理
使用 Active Directory 管理中心 (ADAC) 或 AD PowerShell 等熟悉的 Active Directory 系統管理工具，可以從遠端管理 Azure AD 網域服務受控網域。 租用戶系統管理員沒有權限，不能透過遠端桌面連接受控網域上的網域控制站。 若要管理受控網域，請在加入受控網域的虛擬機器上安裝 AD 系統管理工具功能。 如需相關指示，請參閱 [dminister an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md) 一文。

## <a name="create-an-organizational-unit-on-the-managed-domain"></a>在受控網域上建立組織單位
既然加入網域的虛擬機器上已安裝了 AD 系統管理工具，我們就可以使用這些工具在受控網域上建立組織單位。 執行下列步驟：

> [!NOTE]
> 只有「AAD DC 管理員」群組的成員具有建立自訂 OU 的必要權限。 請確保以這個群組成員的使用者身分執行下列步驟。
>
>

1. 從 [開始] 畫面中，按一下 [系統管理工具] 。 您應該會看到安裝在虛擬機器上的 AD 系統管理工具。

    ![安裝在伺服器上的系統管理工具](./media/active-directory-domain-services-admin-guide/install-rsat-admin-tools-installed.png)
2. 按一下 [Active Directory 管理中心] 。

    ![Active Directory 管理中心](./media/active-directory-domain-services-admin-guide/adac-overview.png)
3. 若要檢視網域，請按一下左窗格中的網域名稱 (例如，'contoso100.com')。

    ![ADAC - 檢視網域](./media/active-directory-domain-services-admin-guide/create-ou-adac-overview.png)
4. 在右側的 [工作] 窗格中，按一下網域名稱節點下的 [新增]。 在此範例中，會在右側的 [工作] 窗格中，按一下 'contoso100(local)' 節點下的 [新增]。

    ![ADAC - 新的 OU](./media/active-directory-domain-services-admin-guide/create-ou-adac-new-ou.png)
5. 您應該會看到建立組織單位的選項。 按一下 [組織單位] 啟動 [建立組織單位] 對話方塊。
6. 在 [建立組織單位] 對話方塊中，指定新 OU 的 [名稱]。 提供 OU 的簡短描述。 您也可以設定 OU 的 [受控者] \(管理者)  欄位。 若要建立自訂映像，請按一下 [確定] 。

    ![ADAC - 建立 OU 對話方塊](./media/active-directory-domain-services-admin-guide/create-ou-dialog.png)
7. 新建立的 OU 現在應該會出現在 AD 管理中心 (ADAC) 中。

    ![ADAC - OU 已建立](./media/active-directory-domain-services-admin-guide/create-ou-done.png)

## <a name="permissionssecurity-for-newly-created-ous"></a>新建 OU 的權限/安全性
建立自訂 OU 的使用者 (「AAD DC 系統管理員」群組的成員) 預設會被授與 OU 的系統管理權限 (完全控制)。 這個使用者接著可以繼續將權限授與其他使用者，或視需要授與「AAD DC 系統管理員」群組。 如下面的螢幕擷取畫面所示，建立新 'MyCustomOU' 組織單位的使用者 'bob@domainservicespreview.onmicrosoft.com' 獲授與該組織單位的完整控制權。

 ![ADAC - 新的 OU 安全性](./media/active-directory-domain-services-admin-guide/create-ou-permissions.png)

## <a name="notes-on-administering-custom-ous"></a>管理自訂 OU 的注意事項
既然您已建立了自訂 OU，就可以繼續在這個 OU 中建立使用者、群組、電腦和服務帳戶。 您無法將使用者或群組從「AAD DC 使用者」OU 移至自訂 OU。

> [!WARNING]
> 您在自訂 OU 下建立的使用者帳戶、群組、服務帳戶和電腦物件無法在 Azure AD 租用戶中使用。 換句話說，使用 Azure AD Graph API 或 Azure AD UI 中都不會顯示這些物件。 這些物件只可用於 Azure Active Directory Domain Services 受控網域中。
>
>

## <a name="related-content"></a>相關內容
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [設定受控網域的群組原則](active-directory-ds-admin-guide-administer-group-policy.md)
* [Active Directory 管理中心：入門](https://technet.microsoft.com/library/dd560651.aspx)
* [服務帳戶的逐步指南](https://technet.microsoft.com/library/dd548356.aspx)
