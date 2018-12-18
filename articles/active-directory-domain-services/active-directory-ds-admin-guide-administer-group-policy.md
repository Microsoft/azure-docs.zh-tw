---
title: Azure Active Directory Domain Services：管理受控網域上的群組原則 | Microsoft Docs
description: 管理 Azure Active Directory Domain Services 受控網域上的群組原則
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: acdba45bef5407af4b96d8e5f805a828e10d2d61
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502211"
---
# <a name="administer-group-policy-on-an-azure-ad-domain-services-managed-domain"></a>管理 Azure AD Domain Services 受控網域上的群組原則
Azure Active Directory Domain Services 有「AADDC 使用者」和「AADDC 電腦」容器專用的內建群組原則物件 (GPO)。 您可以自訂這些內建 GPO，來設定受控網域上的群組原則。 此外，「AAD DC 系統管理員」群組的成員可以在受控網域內建立自己的自定組織單位 (OU)。 他們也可以建立自訂 GPO，並將它們連結至這些自訂的 OU。 屬於「AAD DC 系統管理員」群組的使用者會獲得受控網域上的「群組原則」系統管理權限。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. **已加入網域的虛擬機器** ，您可在其中管理 Azure AD 網域服務受控網域。 如果您沒有這類虛擬機器，請依照名為[將 Windows 虛擬機器加入受控網域](active-directory-ds-admin-guide-join-windows-vm.md)一文所述的所有工作進行操作。
5. 您需要目錄中**屬於「AAD DC 系統管理員」群組之使用者帳戶**的認證，才能管理受控網域的群組原則。

<br>

## <a name="task-1---provision-a-domain-joined-virtual-machine-to-remotely-administer-group-policy-for-the-managed-domain"></a>工作 1 - 佈建已加入網域的虛擬機器以從遠端管理受控網域的群組原則
使用 Active Directory 管理中心 (ADAC) 或 AD PowerShell 等熟悉的 Active Directory 系統管理工具，可以從遠端管理 Azure AD 網域服務受控網域。 同樣地，使用群組原則伺服器系統管理工具，可以從遠端管理受控網域的群組原則。

Azure AD 目錄中的系統管理員沒有權限，不能透過遠端桌面連接受控網域上的網域控制站。 「AAD DC 系統管理員」群組的成員可以遠端管理受控網域的群組原則。 他們可以使用已加入受控網域的 Windows Server/用戶端電腦上的群組原則工具。 可以將群組原則工具安裝為 Windows Server/用戶端電腦 (已加入受控網域) 上群組原則管理選用功能的一部分。

第一個工作是佈建已加入受控網域的 Windows Server 虛擬機器。 如需相關指示，請參閱標題為 [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)的文章。

## <a name="task-2---install-group-policy-tools-on-the-virtual-machine"></a>工作 2 - 在虛擬機器上安裝群組原則工具
執行下列步驟，在已加入網域的虛擬機器上安裝群組原則管理工具。

1. 瀏覽至 Azure 入口網站。 在左側面板上按一下 [所有資源]。 找出並按一下您在工作 1 中建立的虛擬機器。
2. 在 [概觀] 索引標籤上按一下 [連線] 按鈕。隨即建立並下載遠端桌面通訊協定 (.rdp) 檔案。

    ![連線至 Windows 虛擬機器](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)
3. 若要連線至您的 VM，請開啟下載的 RDP 檔案。 出現提示時，按一下 [連線]。 在登入提示中，使用屬於「AAD DC 系統管理員」群組之使用者的認證。 例如，在我們的案例中，我們會使用 'bob@domainservicespreview.onmicrosoft.com'。 您可能會在登入過程中收到憑證警告。 按一下 [是] 或 [繼續] 以繼續進行連線。
4. 在 [開始] 畫面中開啟 [伺服器管理員] 。 按一下 [伺服器管理員] 視窗中央窗格內的 [新增角色及功能]  。

    ![啟動虛擬機器上的伺服器管理員](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager.png)
5. 在 [新增角色及功能精靈] 的 [開始之前] 頁面上，按 [下一步]。

    ![[開始之前] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-begin.png)
6. 在 [安裝類型] 頁面上，保持勾選 [角色型或功能型安裝] 選項，然後按 [下一步]。

    ![[安裝類型] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-type.png)
7. 在 [伺服器選擇] 頁面上，從伺服器集區中選取目前的虛擬機器，然後按 [下一步]。

    ![[伺服器選擇] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-server.png)
8. 在 [伺服器角色] 頁面上，按 [下一步]。 我們會略過此頁面，因為我們沒有要在伺服器上安裝任何角色。
9. 在 [功能] 頁面上，選取 [群組原則管理] 功能。

    ![[功能] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-gp-management.png)
10. 在 [確認] 頁面上，按一下 [安裝] 在虛擬機器上安裝群組原則工具的功能。 順利完成功能安裝時，按一下 [關閉] 以結束 [新增角色及功能] 精靈。

    ![確認電子郵件](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-gp-management-confirmation.png)

## <a name="task-3---launch-the-group-policy-management-console-to-administer-group-policy"></a>工作 3 - 啟動群組原則管理主控台來管理群組原則
您可以使用已加入網域的虛擬機器上的「群組原則」管理主控台來管理受控網域上的群組原則。

> [!NOTE]
> 您必須是「AAD DC 系統管理員」群組的成員，才能管理受控網域的群組原則。
>
>

1. 從 [開始] 畫面中，按一下 [系統管理工具] 。 您應該會看到安裝在虛擬機器上的 [群組原則管理] 主控台。

    ![啟動群組原則管理](./media/active-directory-domain-services-admin-guide/gp-management-installed.png)
2. 按一下 [群組原則管理] 啟動群組原則管理主控台。

    ![群組原則主控台](./media/active-directory-domain-services-admin-guide/gp-management-console.png)

## <a name="task-4---customize-built-in-group-policy-objects"></a>工作 4 - 自訂內建群組原則物件
有兩個內建群組原則物件 (GPO)，各自用於受控網域中的「AADDC 電腦」和「AADDC 使用者」容器。 您可以自訂這些 GPO，來設定受控網域上的群組原則。

1. 在 [群組原則管理] 主控台中，按一下 [Forest: contoso100.com] 和 [Domains] 節點加以展開，查看您的受控網域的群組原則。

    ![內建 GPO](./media/active-directory-domain-services-admin-guide/builtin-gpos.png)
2. 您可以自訂這些內建 GPO，來設定受控網域上的群組原則。 以滑鼠右鍵按一下 GPO，然後按一下 [編輯...] 以自訂內建 GPO。 [群組原則設定編輯器] 工具可讓您自訂 GPO。

    ![編輯內建 GPO](./media/active-directory-domain-services-admin-guide/edit-builtin-gpo.png)
3. 現在您可以使用 [群組原則管理編輯器] 主控台來編輯內建的 GPO。 例如，下列螢幕擷取畫面示範如何自訂內建「AADDC 電腦」的 GPO。

    ![自訂 GPO](./media/active-directory-domain-services-admin-guide/gp-editor.png)

## <a name="task-5---create-a-custom-group-policy-object-gpo"></a>工作 5 - 建立自訂群組原則物件 (GPO)
您可以建立或匯入自己的自訂群組原則物件。 您也可以將自訂 GPO 連結到您在受控網域中建立的自訂 OU。 如需建立自訂組織單位的詳細資訊，請參閱[在受控網域上建立自訂 OU](active-directory-ds-admin-guide-create-ou.md)。

> [!NOTE]
> 您必須是「AAD DC 系統管理員」群組的成員，才能管理受控網域的群組原則。
>
>

1. 在 [群組原則管理] 主控台中，按一下以選取您的自訂組織單位 (OU)。 以滑鼠右鍵按一下 OU，然後按一下 [在這個網域中建立 GPO 並連結到...]。

    ![建立自訂 GPO](./media/active-directory-domain-services-admin-guide/gp-create-gpo.png)
2. 指定新 GPO 的名稱，然後按一下 [確定]。

    ![指定 GPO 的名稱](./media/active-directory-domain-services-admin-guide/gp-specify-gpo-name.png)
3. 系統將會建立新的 GPO 並連結至您的自訂 OU。 以滑鼠右鍵按一下 GPO，然後按一下功能表上的 [編輯...]。

    ![新建立的 GPO](./media/active-directory-domain-services-admin-guide/gp-gpo-created.png)
4. 您可以使用 [群組原則管理編輯器] 自訂新建立的 GPO。

    ![自訂新 GPO](./media/active-directory-domain-services-admin-guide/gp-customize-gpo.png)


可在 Technet 上取得有關使用[群組原則管理主控台](https://technet.microsoft.com/library/cc753298.aspx)的詳細資訊。

## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [群組原則管理主控台](https://technet.microsoft.com/library/cc753298.aspx)
