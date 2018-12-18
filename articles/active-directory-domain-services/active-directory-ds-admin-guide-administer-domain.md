---
title: Azure Active Directory Domain Services：管理受控網域 | Microsoft Docs
description: 管理 Azure Active Directory Domain Services 受控網域
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: d4fdbc75-3e6b-4e20-8494-5dcc3bf2220a
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: c2258b9d26a62fd7c8e50c1ed7b3583c06e1c353
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503429"
---
# <a name="administer-an-azure-active-directory-domain-services-managed-domain"></a>管理 Azure Active Directory Domain Services 受控網域
本文說明如何管理 Azure Active Directory (AD) 網域服務受控網域。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要完成本文中所列的工作，您需要：

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. **已加入網域的虛擬機器** ，您可在其中管理 Azure AD 網域服務受控網域。 如果您沒有這類虛擬機器，請依照名為[將 Windows 虛擬機器加入受控網域](active-directory-ds-admin-guide-join-windows-vm.md)一文所述的所有工作進行操作。
5. 您需要目錄中 **屬於「AAD DC 系統管理員」群組之使用者帳戶** 的認證，才能管理受控網域。

<br>

## <a name="administrative-tasks-you-can-perform-on-a-managed-domain"></a>可在受控網域中執行的系統管理工作
「AAD DC 系統管理員」群組的成員已獲授與受控網域的權限，這些權限讓他們可以進行如下工作：

* 將機器加入受控網域。
* 為受控網域中的「AADDC 電腦」和「AADDC 使用者」容器設定內建 GPO。
* 管理受控網域上的 DNS。
* 建立及管理受控網域上的自訂組織單位 (OU)。
* 取得已加入受控網域之電腦的系統管理存取權。

## <a name="administrative-privileges-you-do-not-have-on-a-managed-domain"></a>未擁有的受控網域系統管理權限
網域是由 Microsoft 負責管理，包括修補、監視及建立備份等活動。 網域已被鎖定，而您沒有在網域上進行特定系統管理工作的權限。 您無法進行之工作的部分範例如下。

* 您不會擁有受控網域的網域系統管理員或企業系統管理員權限。
* 您無法延伸受控網域的結構描述。
* 您無法使用遠端桌面連線到受控網域的網域控制站。
* 您無法在受控網域中新增網域控制站。

## <a name="task-1---create-a-domain-joined-windows-server-virtual-machine-to-remotely-administer-the-managed-domain"></a>工作 1 - 建立已加入網域的 Windows Server 虛擬機器以從遠端管理受控網域
您可以使用 Active Directory 管理中心 (ADAC) 或 AD PowerShell 等熟悉的 Active Directory 系統管理工具來管理 Azure AD 網域服務受控網域。 租用戶系統管理員沒有權限，不能透過遠端桌面連接受控網域上的網域控制站。 「AAD DC 系統管理員」群組的成員可以使用 AD 系統管理工具，從已加入受控網域的 Windows Server/用戶端電腦遠端管理受控網域。 在已加入受控網域的 Windows Server 和用戶端電腦上，可以將 AD 系統管理工具安裝做為遠端伺服器管理工具 (RSAT) 選擇性功能的一部分。

第一個步驟是設定已加入受控網域的 Windows Server 虛擬機器。 如需相關指示，請參閱標題為 [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)的文章。

### <a name="remotely-administer-the-managed-domain-from-a-client-computer-for-example-windows-10"></a>從用戶端電腦 (例如 Windows 10) 遠端管理受控網域
本文中的指示使用 Windows Server 虛擬機器來管理 AAD-DS 受控網域。 不過，您也可以選擇使用 Windows 用戶端 (例如 Windows 10) 虛擬機器來執行這項操作。

您可以按照 TechNet 上的指示，在 Windows 用戶端虛擬機器上 [安裝遠端伺服器管理工具 (RSAT)](http://social.technet.microsoft.com/wiki/contents/articles/2202.remote-server-administration-tools-rsat-for-windows-client-and-windows-server-dsforum2wiki.aspx) 。

## <a name="task-2---install-active-directory-administration-tools-on-the-virtual-machine"></a>工作 2 - 在虛擬機器上安裝 Active Directory 系統管理工具
完成下列步驟，在已加入網域的虛擬機器上安裝 Active Directory 系統管理工具。 如需有關[安裝和使用遠端伺服器管理工具的詳細資訊](https://technet.microsoft.com/library/hh831501.aspx)，請參閱 TechNet。

1. 瀏覽至 Azure 入口網站。 在左側面板上按一下 [所有資源]。 找出並按一下您在工作 1 中建立的虛擬機器。
2. 在 [概觀] 索引標籤上按一下 [連線] 按鈕。隨即建立並下載遠端桌面通訊協定 (.rdp) 檔案。

    ![連線至 Windows 虛擬機器](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)
3. 若要連線至您的 VM，請開啟下載的 RDP 檔案。 出現提示時，按一下 [連線]。 使用屬於「AAD DC 系統管理員」群組之使用者的認證。 例如，'bob@domainservicespreview.onmicrosoft.com'。 您可能會在登入過程中收到憑證警告。 按一下 [是] 或 [繼續] 以繼續進行連線。
4. 在 [開始] 畫面中開啟 [伺服器管理員] 。 按一下 [伺服器管理員] 視窗中央窗格內的 [新增角色及功能]  。

    ![啟動虛擬機器上的伺服器管理員](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager.png)
5. 在 [新增角色及功能精靈] 的 [開始之前] 頁面上，按 [下一步]。

    ![[開始之前] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-begin.png)
6. 在 [安裝類型] 頁面上，保持勾選 [角色型或功能型安裝] 選項，然後按 [下一步]。

    ![[安裝類型] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-type.png)
7. 在 [伺服器選擇] 頁面上，從伺服器集區中選取目前的虛擬機器，然後按 [下一步]。

    ![[伺服器選擇] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-server.png)
8. 在 [伺服器角色] 頁面上，按 [下一步]。
9. 在 [功能] 頁面上，按一下以展開 [遠端伺服器管理工具] 節點，然後按一下以展開 [角色管理工具] 節點。 從角色管理工具的清單中選取 [AD DS 及 AD LDS 工具]  功能。

    ![[功能] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-ad-tools.png)
10. 在 [確認] 頁面上，按一下 [安裝] 以在虛擬機器上安裝 AD 與 AD LDS 工具功能。 順利完成功能安裝時，按一下 [關閉] 以結束 [新增角色及功能] 精靈。

    ![確認電子郵件](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-confirmation.png)

## <a name="task-3---connect-to-and-explore-the-managed-domain"></a>工作 3 - 連線到受控網域並進行瀏覽
現在，您可以使用 Windows Server AD 系統管理工具來探索和管理受控網域。

> [!NOTE]
> 您必須是「AAD DC 系統管理員」群組的成員，才能管理受控網域。
>
>

1. 從 [開始] 畫面中，按一下 [系統管理工具] 。 您應該會看到安裝在虛擬機器上的 AD 系統管理工具。

    ![安裝在伺服器上的系統管理工具](./media/active-directory-domain-services-admin-guide/install-rsat-admin-tools-installed.png)
2. 按一下 [Active Directory 管理中心] 。

    ![Active Directory 管理中心](./media/active-directory-domain-services-admin-guide/adac-overview.png)
3. 若要探索網域，請按一下左窗格中的網域名稱 (例如，'contoso100.com')。 請注意分別稱為「AADDC 電腦」和「AADDC 使用者」的兩個容器。

    ![ADAC - 檢視網域](./media/active-directory-domain-services-admin-guide/adac-domain-view.png)
4. 在稱為 [AADDC 使用者]  的容器上按一下，以查看所有屬於受控網域的使用者和群組。 在此容器中，您應該會看到您的 Azure AD 租用戶中的使用者帳戶和群組顯示出來。 在這個範例中，請注意此容器中有 'bob' 使用者的使用者帳戶和稱為「AAD DC 系統管理員」的群組。

    ![ADAC - 網域使用者](./media/active-directory-domain-services-admin-guide/adac-aaddc-users.png)
5. 在稱為 [AADDC 電腦]  的容器上按一下，以查看已新增此受控網域的電腦。 您應該會看到已加入網域之目前虛擬機器的項目。 已新增 Azure AD 網域服務受控網域之所有電腦的電腦帳戶會儲存在此「AADDC 電腦」容器中。

    ![ADAC - 已加入網域的電腦](./media/active-directory-domain-services-admin-guide/adac-aaddc-computers.png)

<br>

## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [部署遠端伺服器管理工具](https://technet.microsoft.com/library/hh831501.aspx)
